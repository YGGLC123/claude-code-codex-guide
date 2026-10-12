# Chapter 9. Subagents and background work: getting AI to follow long-running tasks

In the earlier chapters, you were always one person talking to one AI. Once tasks get longer, that setup is no longer enough: when you run a job that takes several hours, you cannot sit in front of the screen the whole time. This chapter describes my current approach: **the foreground is for conversation only; execution work goes to background "subagents," which keep following the job and notify me when there is a result.**

First, a note on terms. A subagent is another AI that the main conversation sends off to handle one thing. It has its own context and hands the result back when it finishes.
## 9.1 What a subagent is and how to create one

A subagent is a Markdown file with YAML front matter. Two fields are required: `name` and `description`. Optional fields include `tools` (which tools it may use), `model`, `permissionMode` and others. Where you put the file decides who can use it:

| Location | Who can use it |
|---|---|
| `.claude/agents/` in a project | Only that project |
| Personal directory `~/.claude/agents/` | All of your projects |

On Windows, `~` means `C:\Users\<your-username>`. The order of precedence is organization > the `--agents` startup flag > project > personal > plugins; when names collide, the earlier one overrides the later ones.

Below is a minimal example. Save it as `~/.claude/agents/log-watcher.md`:

```markdown
---
name: log-watcher
description: Watches a long-running job and gives a one-line conclusion when the job finishes or fails
tools: Read, Bash
---
You are responsible for watching one job. Read only the log and the process status, and do not modify any file.
When the job finishes, fails, gets stuck or the process disappears, report in one sentence: what happened and where the result is.
Do not write commentary along the way; write only the report at the end.
```

> [!IMPORTANT]
> **`/agents` is no longer a management interface**
> The older tutorial advice to "type `/agents` to open the management interface" is out of date. Since v2.1.198, it only prints a reminder suggesting that you tell Claude directly "help me create a subagent," or edit the files in the two directories above yourself.

The easiest way to create one is to describe the subagent you need to Claude and let it write the file for you.

### Sending work to the background; background sessions

In the interactive interface, press `Ctrl+B` to move a running command or agent to the background, which frees the input box right away. The command line also has a `claude agents` subcommand, described as "background sessions" in `claude --help` on my machine. For the exact arguments to list and reattach to a background session, the official documentation is authoritative; run `claude agents --help` before using it.

> [!NOTE]
> **Checkpoint**
> After you press `Ctrl+B`, the command is still running, but you can keep typing in the input box. This is the smallest form of "talk only in the foreground."

## 9.2 Foreground for conversation, background for execution

I set myself a rule: **the input box holds only things I can read**, that is, discussion, decisions I need to make, and results. Running commands, reading logs, checking and committing all go to background subagents.

The reason is practical. I cannot read server output, process status or logs, and reading them gains me nothing. Worse, once tool calls start flooding the screen, the few sentences of conclusion that actually need reading get buried.

When handing a task to a subagent, I follow three practices:

1. **Keep the task brief short but complete.** Spell out four things: what the current state is; what the rules are; how to handle each item in flight; and what situations call for a report. If the brief is incomplete, it will backtrack and waste a round.
2. **Report only when there is a result or an exception.** Reporting routine progress every day just wastes your attention. An exception report states three things: what happened, what it affects, and what it plans to do or what it needs you to decide.
3. **No commentary between tool calls; write only the report at the end.** Commentary shows up in your panel and floods it line by line.

> [!TIP]
> Add one sentence to the task brief: "Do not write commentary along the way; write only one report at the end." That single sentence makes the panel noticeably cleaner.

There is one more rule, aimed at you: for anything you need to decide, have the AI explain it fully in its reply in plain language, including the mechanism, the numbers, the options, its recommendation and the consequences, and never write "see such-and-such file." I never open the files it writes. Once it put the decision content in a file and replied with only a file name, and I could not make a decision from that.

## 9.3 One background lead agent per line of work, plus a ledger

Too many subagents cause problems. At first I opened one subagent per task; then I tried opening a separate "execution session," which I scrapped the same day because switching back and forth was too inconvenient. The approach I settled on is:

- **For each line of work, keep exactly one background "lead agent" behind the main conversation**, dedicated to the long, multi-step work on that line.
- Beyond that, the main conversation opens only **one-shot, short subagents that end when done**, for example to check a number or write a passage. Once finished, they end, and I do not give them any other tasks.
- The lead agent's progress is **written to a ledger file** (a progress log); the main conversation receives only conclusions and exceptions and does not watch the process.

This way the main conversation can keep a long context and receive only conclusions, so it does not drift from the main thread.

It started with a bill: I compared my spending over several consecutive days and found that **long-lived subagents carried about 370,000 tokens of context on average at each step, and more than 80% of the cost went to them**. The longer a subagent grows, the more it must carry all the earlier content again at every step, so the cost rises naturally. That is where the rule "when a task is done, wrap it up; never let a single subagent keep growing" comes from, along with the cap and rotation below.

> [!IMPORTANT]
> **What to tell a subagent when it stops to wait for a job**
> Have it write only one line as its last message: "(waiting) what it is waiting for, and the expected time," and the main conversation takes no action on it. To check progress, read the ledger; do not send a message to ask. Each time a background subagent finishes a turn, the platform notifies and wakes the main conversation once, and this cannot be avoided; if you then send a message to ask, it wakes it one more time.

### The standard for a ledger

I set only one standard for the ledger: **a brand-new agent should be able to pick up the work by reading this ledger alone.** Therefore:

- Update it at every milestone: a job starts or stops, a conclusion is settled, an output is saved to disk;
- Before the main conversation is compacted or restarted, have the lead agent "wrap up" and write the ledger first;
- Otherwise neither rotation nor compaction can carry the work forward, and nobody will know how to take over the jobs in flight.

The template is in Section 9.6.

## 9.4 Context cap and rotation

A long-running lead agent keeps growing, so set a limit for it. Mine is **about 270,000 tokens**: when it reaches the limit, it first writes the ledger, then ends the turn with a fixed marker (I use "[ROTATE]"); the main conversation then opens a new lead agent from the ledger to carry on.

### Reminding at the cap with a hook (the principle)

The difficulty is that a subagent shares its compaction threshold with the session it runs in; there is no setting specific to subagents. So the cap has to be implemented separately. My approach uses a hook (for what hooks are, see [Chapter 10](10-extensions.md)):

1. Use the `PostToolUse` event, which fires after every tool call;
2. The hook script reads the latest usage figures from this subagent's own transcript and adds up the input, cache-read and cache-write counts; the sum is the current context size;
3. When the limit is reached, it injects a reminder into this subagent's context: "Time to wrap up: write the ledger and end with [ROTATE]"; after that it reminds again every 10 tool calls;
4. It identifies its targets by a fixed marker at the start of the prompt. I begin every lead agent's prompt with "[LEAD]", and the hook applies only to subagents like that;
5. In the subagent's role description, state in advance: "You will receive this kind of reminder. It is legitimate; please follow it."

The last item cannot be skipped. If a subagent is not told beforehand, it may treat the reminder as a suspicious instruction: I tested this twice, and once it complied and once it refused.

> [!WARNING]
> **Do not use `py` in the hook command**
> The problem I ran into: using the `py` launcher in the hook command made startup twice as slow, and `PostToolUse` runs on every tool call. Another problem was that after a wrong script path, Python exited with code 2, and **every** tool call afterward reported an error.
> The fix: call `python.exe` directly and add the `-S -E` flags; to disable the hook, first delete the relevant block from settings, and only then edit the script.

I also observed several other things in testing: a hook newly added to the project `settings.json` is hot-loaded; when a subagent makes the call, the hook input carries `agent_id` and `agent_type`, while the main thread's does not; injected content goes only to the subagent that made that call; and an agent definition added midway through a session cannot be used in that session. For the exact fields, the official documentation is authoritative.

## 9.5 Background commands and polling scripts

One of the main reasons a subagent's context grows is that it queries the status over and over, or keeps sleeping to wait. Every query carries one more round of context.

The right approach is to **write a polling script and let the script do the looping**; the subagent only needs to "start the command, end its turn, and be woken up." Three key points:

1. **Cover every terminal state.** Whether the job finishes, fails, gets stuck or the process disappears, print a one-line conclusion first and then exit; do not wait only for a "success marker." If the script waits only for success, it will wait forever once an error occurs.
2. **A background command can run for at most 2 hours.** For longer jobs, have the script print "still running" and exit at about 110 minutes, then start it again after the wake-up.
3. **Only a background subagent can work this way.** When a foreground subagent ends, its result goes straight back to the main conversation, so there is no "being woken up" step.

Here is a script skeleton, Windows (PowerShell); adapt it to your job:

```powershell
$logFile = "<path to the log file>"
$deadline = (Get-Date).AddMinutes(110)
while ((Get-Date) -lt $deadline) {
    if (Select-String -Path $logFile -Pattern "<success marker>" -Quiet) { "Done"; exit 0 }
    if (Select-String -Path $logFile -Pattern "<error marker>" -Quiet) { "Failed, see the end of the log"; exit 1 }
    if (-not (Get-Process -Id <process ID> -ErrorAction SilentlyContinue)) { "Process is gone and there is no success marker"; exit 2 }
    Start-Sleep -Seconds 60
}
"Still running, 110 minutes reached, restart the watch"
exit 3
```

`<path to the log file>` is where the job writes its log; `<process ID>` can be found on the Details tab of Task Manager; replace the two markers with lines that will actually appear in your job's log. This way, when the script exits, the subagent receives only a one-line conclusion and does not need to read the log itself.

### Set up a watchdog for Codex too

When running several Codex tasks in parallel in non-interactive mode (`codex exec`), I have had two lessons: in one night, a run silently stopped responding 4 times; and two runs sat idle for 80 minutes after disconnecting, with nobody noticing. My rules now are:

- Set up a watchdog for each run: **if the log has no new writes for 30 to 45 minutes, wake me up**;
- After a disconnection, resume the original session with `codex exec resume <session ID>` and do not start over;
- Put a cap on the number of retries to avoid endless loops.

The syntax of `codex exec resume` is `codex exec resume <session ID>`, or use `codex exec resume --last` to pick up the most recent one in the current directory; add `--all` to search across directories.

```bash
codex exec resume <session ID> "Continue the part you did not finish"
```

`<session ID>` is the session ID that Codex prints on its first run.

## 9.6 A ledger template you can copy directly

Save the template below as `ledger.md` in the folder for that line of work, and have the lead agent update it at every milestone.

```markdown
# Ledger: <line name>

Last updated: <date and time>    Updated by: <which lead agent in the sequence>

## Current state
One or two sentences: what stage this line is in now, and whether everything is normal overall.

## Jobs in flight
| Item | Status | How to pick it up | Expected |
|---|---|---|---|
| <Job A> | Running | Log at <path>; watch the end of the log for <success marker> | <expected completion time> |
| <Job B> | Paused | Resume command: <command> | Waiting for me to say go |

## Next steps
1. <First thing to do next>
2. <Second thing to do next>

## Where outputs are
- <Result file>: <path>
- <Intermediate file>: <path> (deletable / not deletable)

## To report
- What to report when there is a result: <for example, "After A finishes, report the key numbers to me">
- What to report when there is an exception: what happened, what it affects, what it plans to do or what it needs me to decide

## Rules (read before taking over)
- <One or two hard rules for this line>
- When the context reaches about 270,000 tokens: write the ledger and end this turn with [ROTATE]
```

> [!NOTE]
> **Checkpoint**
> Do a dry run before relying on it: start a brand-new session, give it only this ledger, and ask "what should you do next?" If it can answer, the ledger is good enough; if it cannot, something is missing, and you should fill it in before using it.

### References

- Claude Code subagents: https://code.claude.com/docs/en/sub-agents (checked in October 2026)
- Claude Code interactive mode and keyboard shortcuts: https://code.claude.com/docs/en/interactive-mode (checked in October 2026)
- Claude Code slash commands: https://code.claude.com/docs/en/commands (checked in October 2026)
- Claude Code hooks: https://code.claude.com/docs/en/hooks (checked in October 2026)
- Codex non-interactive mode: https://learn.chatgpt.com/docs/non-interactive-mode (checked in October 2026)
- Codex command-line reference: https://learn.chatgpt.com/docs/developer-commands?surface=cli (checked in October 2026)

---
[← Chapter 8. Division of labor: Claude coordinates, Codex writes code, Sonnet writes prose, Pro takes on hard problems](08-division-of-labor.md) · [Contents](../README.md#contents) · [Chapter 10. Extensions: Skills, MCP, hooks and plugins →](10-extensions.md)
