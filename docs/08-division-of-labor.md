# Chapter 8. Division of labor: Claude coordinates, Codex writes code, Sonnet writes prose, Pro takes on hard problems

The previous chapters covered installing Claude Code and Codex. This chapter explains how to make them work together. My approach is not to let one model handle everything, but to manage them like a small team, with each role responsible for what it does best.

## 8.1 Why divide the work

The reason is practical: usage limits.

My Claude subscription is already on the highest tier, and usage is still tight. On the ChatGPT side, Codex usage has always had room to spare. I also worked out the cost of pay-as-you-go API billing and found it poor value, so I did not use it, and I did not buy a second Claude account either. The conclusion: Claude does the thinking and judging, and Codex handles the execution work.

My current division of labor looks like this:

| Role | Who | Responsibilities |
|---|---|---|
| Main session (the "controller") | Claude (Opus) | Talks to you, aligns on requirements, assigns work, checks results, reports back; the subagents that execute, monitor and verify also run on it |
| Writing code | Codex | Writes scripts, writes code, edits code, diagnoses code problems, produces a second implementation |
| Writing prose | Claude's Sonnet subagent | Copy, explanations, polishing |
| Hard problems | GPT Pro (used manually in the web interface) | Proofs, strict verification, problems that need long thinking |

Claude also does the execution and monitoring itself, because mistakes in that work are costly and it is more reliable there. Usage is the bottleneck, so I outsource only the two kinds of work that suit it best: writing code and writing text.

> [!IMPORTANT]
> This is my own setup, not an official recommendation. Your subscription tier and usage limits differ, so adjust it to your situation. Model names and tiers change; check the official documentation for what is currently available (see Section 7.6 in [Chapter 7](07-saving-usage.md)).

## 8.2 Claude as the main session

The main session has four jobs: aligning on requirements, assigning work, checking results, and reporting.

**Aligning on requirements.** After you state an idea, it first restates it: what is to be done, what counts as finished, and what must not be changed. Do not assign any work until the restatement is clear, otherwise a misunderstanding gets amplified once the work is out.

**Assigning work.** Split the task into pieces that do not depend on each other, write each piece as a standalone task brief, and give it to the right executor. A brief should be short but complete: the current state, the rules, where the output goes, and what situations require a report.

**Checking results.** The main session cannot skip this step. For anything Codex hands back, I trust only what comes from actually running it and from grep, not Codex's own verdict. Having it say "fixed, tests pass" is not enough; have Claude run the tests itself, or check the key numbers.

**Reporting.** The main session absorbs the command output, logs and paths produced during execution and gives you only a few sentences you can understand directly: what was finished, what the result is, whether anything went wrong. An exception report covers three things: what happened, what it affects, and what it plans to do or what it needs you to decide. For the "talk in the foreground, execute in the background" approach, see [Chapter 9](09-subagents-and-background-work.md).

> [!TIP]
> One test: if the main session redoes work the executor has already finished, the delegation was pointless. The main session should look only at the summary and the key evidence.

## 8.3 Assigning coding tasks to Codex

### Basic usage

`codex exec` is Codex's non-interactive mode (short form `codex e`): you give it a task, and it exits when the task is done. Here are a few points I have checked:

- The default is a read-only sandbox. To let it modify files, add `--sandbox workspace-write`.
- Progress goes to stderr; only the final answer goes to stdout.
- `-o <file>` writes the last message to a file, and still prints it as usual.
- If you write the prompt as `-`, or give no prompt argument, it reads from standard input. The official docs do not list a `--prompt-file` option, and it does not appear in `codex exec --help` on my machine either.
- By default it requires a Git repository; outside one, add `--skip-git-repo-check`.
- `-C <directory>` sets its working directory.

For longer tasks, do not put the text directly on the command line. My method is to write the task into a UTF-8 text file and pass it in through standard input. Putting several thousand characters of non-English text on a command line makes quoting and escaping errors very likely; I learned this the hard way: with a prompt of about 7 KB containing Chinese text pasted into the command, all 7 attempts failed with quoting errors.

### A general task-assignment template

Step one: in your project folder, create `task.txt` (UTF-8 encoding) and write the task brief in it. You can use this skeleton as is:

```text
You are an unattended executor. Do not enable Skills, and do not ask me questions; if something is ambiguous, take the most literal reading and note in the report which reading you chose.

[Background]
<One or two sentences on what this project is and its current state>

[Task]
<The specific thing you want done, with the input and the expected output spelled out>

[Output location]
Write all files under this absolute path: <absolute path of the task directory>

[Rules]
- Change only files related to the task.
- Do not delete any existing file, and do not change conclusions.
- If you hit a conflict, get stuck, or the numbers do not add up, stop and say so in the report.

[Report requirements]
Write the report in your last message, in 15 lines or fewer: what you did, the absolute paths of the output files, and whether anything went wrong.
```

Step two: run it.

Windows (PowerShell):

```powershell
cd "<absolute path of the task directory>"
cmd /c 'codex exec --sandbox workspace-write --skip-git-repo-check -C "<absolute path of the task directory>" -o summary.md - < task.txt'
```

macOS / Linux (Terminal):

```bash
cd "<absolute path of the task directory>"
codex exec --sandbox workspace-write --skip-git-repo-check -C "<absolute path of the task directory>" -o summary.md - < task.txt
```

Replace `<absolute path of the task directory>` with the folder that holds the task file, for example `C:\Users\<your-username>\projects\demo`. On Windows I deliberately let `cmd` handle the `<` redirection, because it passes the file to Codex unchanged and does not alter the text encoding.

> [!NOTE]
> **Checkpoint**
> When it finishes, a `summary.md` should appear in the same directory, containing Codex's last message; the output files you asked for are in the directory you specified. Read only this summary; you do not need to go through Codex's full log.

> [!IMPORTANT]
> `workspace-write` allows it to change files in the working directory. The first time, try it in an unimportant folder rather than running it directly against your only copy of the data. Flags such as `--dangerously-bypass-approvals-and-sandbox`, which skip all confirmations, are described in the official docs as extremely dangerous; beginners should not touch them.

### Running in the background and resuming

I wrapped Codex in a small script of my own: the task runs in the background, I read only the final summary in the results directory, and a status file records the state, exit code and session ID. You do not necessarily need to write a script; you can simply ask Claude to run the command above in the background. The key is to read only the summary, not the full log.

After a disconnection, you can resume with the session ID: `codex exec resume --last "<instruction for what to continue>"` picks up the most recent session in the current directory, or use `codex exec resume <session ID>`. For more on following long-running tasks, see [Chapter 9](09-subagents-and-background-work.md).

### Five lessons

**1. In the task prompt, write the report as "your last message", not "write it to the file given by -o".** `-o` is handled by the Codex command line itself, which writes the file; Codex does not see that argument. If you tell it to write to that file, it creates a different new file.

**2. Give output locations as absolute paths, or point `-C` directly at the task directory.** I once wrote "put it in this directory", and it interpreted "this directory" as the root given by `-C`, so five files ended up in the repository root.

**3. Dispatch all independent pieces at once, in parallel.** Start several `codex exec` runs at the same time, each reading its own task file. I tested this: with three started together, each finished within 7.5 to 9.5 seconds. Parallel work has two prerequisites: the pieces must connect through a shared interface description, and two runs must never modify the same file at the same time.

In a macOS / Linux terminal, add `&` at the end of a line to send it to the background. On Windows, the easiest way is to open several PowerShell windows and run one command in each, or simply tell Claude "these three pieces are independent, dispatch them to Codex in parallel" and let it start them.

**4. Manage Codex the way you would manage a team member.** Ask it to hand back only a summary of 15 lines or fewer, and look only at the summary and the progress. Look deeper only when it reports a conflict, gets stuck, the numbers do not add up, or it is about to overstep (deleting files, changing conclusions). Work in rounds with a stopping point at the end of each. I once tried letting it run to the end unattended, and only noticed long after that it had drifted off course.

**5. Do not dispatch small jobs separately.** Even an empty run loads several files such as AGENTS.md, which alone costs 10,000 to 20,000 tokens. For changes that take a few lines, it is cheaper to have Claude make them directly.

### Fix tasks: rerun only the affected checks

When you assign a "fix the bug" task, state in it that only the checks whose inputs changed should be rerun. I have seen one session fix a bug, write tests and run the entire regression suite in a row, and even rerun a full twenty-plus-minute acceptance run from scratch. Work such as testing, diagnosis and producing tables belongs in a separate directory, in a separate session.

### Do not write "skip it if it takes more than 30 minutes"

This one cost me the most. I once wrote "stop on timeout" into a task, and as a result several computations were skipped every time, and the final report contained only extrapolated values.

There are two better approaches:

- First consider whether the algorithm can be improved; do not set compute caps casually. One of my tasks had a 1-hour cap. After the algorithm was improved, it finished on my own machine in 4.5 minutes, and the median gap dropped from 119 to 4.
- If the task really is heavy, have the executor package it as a job bundle (code, small data, resumable checkpoints, per-item saves to disk, and a time estimate) and then hand it to a more powerful machine.

## 8.4 Handing writing to Sonnet

I hand writing work to a Sonnet subagent. In Claude Code, just tell the main session, for example:

```text
Please start a Sonnet subagent to write a 300-word product introduction from the facts and numbers below. I will check it when it is done.
Facts: <the facts you have fixed>
Numbers: <the numbers you have fixed>
```

Claude Code's model aliases include `sonnet`; a subagent's file can also set a model with the `model` field (for how to create subagents, see [Chapter 9](09-subagents-and-background-work.md)).

My process has three steps:

1. **The main session fixes the facts and numbers first.** List clearly in the task brief what must not be changed.
2. **Sonnet produces the draft.** Its prose is concise and fluent. For a single article, try to keep using the same Sonnet session so the tone stays consistent throughout.
3. **The main session checks item by item.** Check the numbers, check the citations, and check the wording you have already settled on.

The third step cannot be skipped. I have seen Sonnet change my settled phrase "researchers and investors" to "readers" on its own. It was not an error; it had "optimized" the text by itself. So after every draft, check the wording you have settled on, one item at a time.

> [!TIP]
> Let Sonnet write the draft and Claude check the facts. Separating these two roles is more reliable than letting one model both write and verify.

## 8.5 Handing hard problems to GPT Pro

GPT Pro has to be used manually in the ChatGPT web interface, and whether your ChatGPT plan includes it is something to confirm in the official documentation. This section covers manual use only: you open the page yourself, paste your material and wait for the result.

### It can only act as a text-only oracle

In my experience, GPT Pro offers no interface through which it can call tools. I tried the API, Codex and custom connectors in the web interface, and none of them could call it. So do not spend time trying to make it modify files automatically; those paths are dead ends.

There is only one way to use it: you provide text, it outputs text, and then you (or Claude) take the answer back, check it and put it into practice. This is entirely different from tools like Claude and Codex, which can read and write files themselves.

### How to provide long material

When the material is long, do not type it into the input box character by character. I tried typing 30,000 characters; it was slow and often dropped characters.

The method is to copy the material to the clipboard on your computer, then paste it into the ChatGPT input box. ChatGPT automatically converts very long pasted content into a .txt attachment. The input box then looks empty, which is normal and does not mean the paste failed. Then type a short instruction in the input box that points to the attachment, for example:

```text
Complete the task exactly as specified in the attached file.
```

It can read that attachment.

> [!NOTE]
> **Checkpoint**
> After pasting, a small file attachment card should appear above or inside the input box. If the card is there, the material has been uploaded.

### Waiting takes a long time; do not click Stop

Pro thinks for a long time. In one test with a 5,200-word task, it thought for 48 minutes 07 seconds before it started to output; harder problems can take more than an hour.

While you wait, it is normal for the Stop button to stay on the page or for a message such as "the system is processing further" to appear. **Do not click Stop.** Stopping early discards all the long reasoning already spent.

### Keep tasks from getting too large

Keep a single task within about 90,000 characters, preferably under 60,000. I have a lesson here: a round of 98,830 characters was aborted with no output after 4 hours 09 minutes of thinking, while a round of 89,339 characters returned normally. If you are over the limit, first delete the part that is a "verbatim restatement of the previous round"; it takes up the most space and is the least necessary.

### How you frame the question determines the quality

This is the point I found has the biggest effect on output quality.

If you vaguely ask it to "check every sentence of B against A", it will judge anything in B that A does not mention as an "unsupported addition", even when the source is actually in an appendix, a figure caption or an attachment. In my tests, about half of the output was this kind of misjudgment.

There are two remedies:

- Give it the reference material as well, including appendices, figure captions and attachments.
- Fix the criteria explicitly, for example: "The facts are settled; judge only the writing."

### Do not steer it toward a high score

Do not hint at a target score in the prompt. My wording makes it act as the strictest adversarial reviewer and prover: doubt by default, and when unsure, rule against.

I found that a leading prompt inflates its self-assessment; reviewing again in a fresh session brings the score back down. So I treat its score as a reference only: I have Claude score it independently once more, and I record both scores and the reason for any difference.

### Confirm the highest tier before sending

Before each send, check that the highest tier is selected. The tier can be reset when you start a new conversation; I have seen a new conversation fall back to the "medium" tier and send a lower-tier answer.

Also, the ChatGPT interface has been redesigned several times (I have run into it three times). So do not memorize the text on the buttons; remember the rule: **choose the newest, highest tier currently available.** If you see a message such as "features will be limited", refresh the page before drawing conclusions; in my case it was just leftover state from an old page.

> [!WARNING]
> **Lower-tier answer**
> The problem I ran into: I sent a message in a new conversation without checking the tier, got a lower-tier answer, and waited a whole round for nothing.
> The fix: before each click on Send, check the tier selector and confirm it is on the highest tier.

### A complete mini workflow

1. In Claude, prepare the material and the question to ask, and save them as a text file.
2. Open the ChatGPT web page, start a new conversation, choose the highest tier, paste the material, and type one short instruction.
3. After sending, go and do other things; there is no need to wait around.
4. When the answer arrives, copy it back and give it to Claude to check, then decide how to apply it.

### References

- Codex non-interactive mode (`codex exec`, `-o`, reading the prompt from standard input, `resume`): https://learn.chatgpt.com/docs/non-interactive-mode (checked in October 2026)
- Codex command-line options (`--skip-git-repo-check`, `-C`, etc.): https://learn.chatgpt.com/docs/developer-commands?surface=cli (checked in October 2026)
- Claude Code model aliases: https://code.claude.com/docs/en/model-config (checked in October 2026)
- Claude Code subagents: https://code.claude.com/docs/en/sub-agents (checked in October 2026)
- Tested on my own machine (Claude Code 2.1.293, Codex 0.159.2): run by the author on Windows 11 on 2026-10-11.

---
[← Chapter 7. Saving usage: context, caching and compaction](07-saving-usage.md) · [Contents](../README.md#contents) · [Chapter 9. Subagents and background work: getting AI to follow long-running tasks →](09-subagents-and-background-work.md)
