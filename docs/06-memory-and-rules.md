# Chapter 6. Making it remember you: CLAUDE.md, AGENTS.md and memory

Claude and Codex share one trait: every time you start a new session, they know nothing about you. You told it last time, "don't modify my config files," and this time it modifies them anyway.

The fix is to write your habits and project rules into a file that it reads automatically at every startup. This chapter covers three things: where that file lives, how to write it, and how to let the tool build up memory on its own. At the end there are two starter templates you can use as they are.

## 6.1 What CLAUDE.md does and where it lives

`CLAUDE.md` is a plain Markdown file that Claude Code reads every time it starts. Where you put it determines its scope:

| Level | Location | Applies to |
|---|---|---|
| User | `~/.claude/CLAUDE.md` | All of your projects, and only you |
| Project | `CLAUDE.md` in the project root, or `.claude/CLAUDE.md` | This project, shared along with the code |
| Local | `CLAUDE.local.md` in the project root | Only you, and only this project; remember to add it to `.gitignore` |

On Windows, `~` means `C:\Users\<your-username>`, so the full path of the user-level file is `C:\Users\<your-username>\.claude\CLAUDE.md`. There is also an "organization-level" file that a company can distribute centrally; individual users don't need to deal with it, and the official docs are authoritative.

**Loading rules**: At startup, it reads the `CLAUDE.md` and `CLAUDE.local.md` files in the current directory and in all parent directories, concatenating them from the root downward. They don't override each other; they stack, and the ones closest to the current directory are read last. A `CLAUDE.md` in a subdirectory is not read at the start; it is loaded only when Claude accesses files in that subdirectory.

Two more practical features:

- A file can import other files with `@path`, up to four levels deep. For example, put settings that are specific to one computer in a separate file and import it.
- The official recommendation is to keep each file under 200 lines. When a file gets too long, the important parts get buried.

### Let it generate the first version

Start Claude Code in your project directory and enter:

```text
/init
```

It scans the whole project and generates a draft `CLAUDE.md`. If you want an interactive flow (which also asks about Skills and hooks), set the environment variable `CLAUDE_CODE_NEW_INIT=1` first; check the official docs for details.

> [!NOTE]
> **Checkpoint**
> A `CLAUDE.md` now exists in the project directory. Open it and you should see a summary of the project, such as how to run it and how the directories are organized.

The generated draft tends to be long and generic. What I do is keep the "how to build, how to test" parts it read from the code, delete content that looks right but isn't useful to you, and then add your own habits by hand.

### Editing with /memory

Enter the following command to choose which CLAUDE.md file to edit, to switch auto memory on or off, and to see what it has already recorded:

```text
/memory
```

> [!TIP]
> Older tutorials often say that typing a sentence that starts with `#` writes it into memory. I checked the current official table of shortcut prefixes: it lists only `/`, `!` and `@`, with no `#`. So treat that usage as unconfirmed, check the official docs, and don't rely on it. When you want it to remember something, the most reliable way is to tell it in plain words: "write this into memory."

## 6.2 AGENTS.md for Codex

Codex reads `AGENTS.md`, which does the same job as `CLAUDE.md`; only the location rules differ slightly:

- **Global level**: It lives in Codex's home directory, which defaults to `~/.codex` (on Windows, `C:\Users\<your-username>\.codex`). If `AGENTS.override.md` exists, it reads that; otherwise it reads `AGENTS.md`. Only the first non-empty file is used.
- **Project level**: Starting at the project root (usually the directory containing `.git`) and going down, level by level, to the current directory, it looks in each level for `AGENTS.override.md` and then `AGENTS.md`, taking at most one file per level.
- **Merging**: Files are concatenated from the root downward; content closer to the current directory comes later and takes higher priority. Empty files are skipped. Once the total exceeds 32 KiB (the default `project_doc_max_bytes`), reading stops and anything after that is discarded.
- It is rebuilt at every startup, so you don't need to clear any cache after editing.

Entering `/init` in Codex also generates a draft `AGENTS.md` in the current directory.

> [!IMPORTANT]
> The 32 KiB limit in Codex is a default (`project_doc_max_bytes`). If your rules are too long, it cannot read the part that goes over.

### Don't copy Claude's rules into Codex wholesale

This is a lesson I learned the hard way. I copied the rules I had written for Claude straight into `AGENTS.md`. One of them said, "when a skill matches, ask me first." That is fine for Claude, which talks with you in front of the screen. Codex runs tasks unattended in the background; as soon as it asks a question, it stops and waits for an answer that never comes. The result was two tasks in a row with no output at all.

So when you write rules for Codex, replace sentences like "ask me first" and "wait for my confirmation" with something like: "When you are unsure, pick a reasonable approach, finish the task, and state in your final summary what you chose."

## 6.3 Auto memory: what it remembers on its own

Besides the files you write yourself, local Claude Code sessions have "auto memory" turned on by default. It automatically records your preferences, corrections you have given it, project background, and where external resources are stored.

- Location: `~/.claude/projects/<project>/memory/`.
- Structure: an index, `MEMORY.md`, plus a number of topic files.
- In each session, it automatically reads only the **first 200 lines or first 25KB** of `MEMORY.md` (whichever limit is reached first); topic files are read only when needed.
- This content exists only on this computer and is not synced to other machines.
- Switch: toggle it in `/memory`; you can also change `autoMemoryEnabled` in settings, or set the environment variable `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

### After a mistake, have it write a memory entry

What it records by itself is not necessarily what you most want remembered. My habit: every time it makes a mistake, I have it write the lesson as a memory entry in a fixed three-part format:

```markdown
Rule: Before changing a config file, make a backup in the same directory with .bak added to the file name.
Why: On 2026-10-03 it overwrote a config I had written by hand, with no backup, and I couldn't get it back.
How to apply: Whenever an existing config file needs to change, copy a backup first, then make the change, then tell me where the backup is.
```

You can say to it: "Write what just happened into memory, using the three parts Rule, Why and How to apply, and put today's date on it."

What "Why" does: with a reason, it can make a judgment call in edge cases; without one, it applies the rule mechanically. What the date does: over time, entries start to contradict each other, and the date lets you tell which one is newer and which one has been superseded.

Also, if you want a practice to hold for the long term, say explicitly that it applies "from now on." If you don't, it will most likely treat it as a request for this one occasion only.

### When the index gets too long, the newest entries are cut off

As mentioned above, `MEMORY.md` is read only up to 200 lines or 25KB. Entries pile up, and once the limit is exceeded, **what gets cut off is exactly the newest content at the end**, which is what you taught it most recently.

Here is what I do:

- Each entry in the index keeps just one sentence that points to a topic file; the details all go in the topic files.
- Periodically (for me, every month), I move outdated or rarely used entries into an "archive" file, and leave only a single line in the index that points to the archive.

## 6.4 How to write rules

There are three bad habits in writing rules, and I have had all of them.

### Use a normal tone, and give the reason

Don't write emphatic phrases such as "must," "absolutely," "highest priority," or "even if there is only a 1% chance." This is a habit left over from the era of older models: back then, models tended to overlook things, so people raised the volume to get their attention. Newer models follow the text strictly, and an over-emphatic tone leads them to over-apply the rule.

Here is a comparison:

```markdown
Bad:
You MUST NEVER, under any circumstances, modify settings.json. This is the highest priority!

Good:
Don't edit settings.json directly. It is strict JSON, a single wrong comma causes a Settings Error, and I have no backup. When a change is needed, tell me what to change and I'll make it myself.
```

The second version spells out the reason, so it can judge similar cases by itself, for example whether it should touch `settings.local.json`.

### Don't write assertive progress pointers

Examples are "this session is working on X" and "after compaction, you must continue X." I have a lesson here: a rule saying "after compaction, continue task X" stayed in the file after the task had long been called off, and from then on every compacted session went off to take over a task that was no longer valid.

Progress changes, so it doesn't belong in a rules file. Have it write progress into a separate handoff document (see the compaction section in [Chapter 7](07-saving-usage.md)); at most, the rules file can say "read the handoff document before starting."

### For each rule, ask first: which failure called for it?

Before adding a rule, ask yourself: which failure of which model made me need this one? If you can't answer, don't add it.

When you switch models, review the rules as a whole. An outdated rule doesn't raise an error; it quietly pulls the new model away from what you expect. In one cleanup, I deleted the "Use proactively" phrase from the descriptions of a dozen or so agents, along with the thinking-length environment variable I had set for an older model, because the newer models recognize only the reasoning effort setting (effort) (see [Chapter 7](07-saving-usage.md)).

## 6.5 Dividing global rules and project rules

- **Global files** (`~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`): these hold practices that apply across projects, such as "reply to me in English," "read a file before modifying it," and "write long output to a file first."
- **Project files**: these hold only that project's own rules, such as how to run it, what the test command is, what the directory conventions are, and which files not to touch.

### Several projects sharing one startup folder

I keep several project sessions open at the same time, all started from the same folder. After compaction or a restart, a session forgets which project it is working on. In the `CLAUDE.md` of this folder, I put a routing table:

```markdown
# About this directory

This directory is the startup directory shared by several sessions. After compaction or a restart, work out from the summary which line of work you are on, then read the handoff document for that line.

| Line of work | Read first |
|---|---|
| Website redesign | `<path to project A's handoff document>` |
| Data cleanup | `<path to project B's handoff document>` |
| Weekly report automation | `<path to project C's handoff document>` |
```

When a session starts, it reads this table, decides which row it belongs to, and then reads the matching document, so it doesn't read the wrong one.

> [!TIP]
> Every computer has some settings of its own (for example, this one is the main machine and that one is a laptop). You can put them in a separate file and import it with `@path` at the end of the shared `CLAUDE.md`. The shared file doesn't need to change, and each of the two machines reads its own settings.

## 6.6 Starter templates you can copy

The two templates below are generic; copy them and adjust them to your situation. Note that the templates contain no emphatic wording, and every item comes with a reason.

### CLAUDE.md (put it at ~/.claude/CLAUDE.md)

```markdown
# How I work

## Communication
- Reply in English. I'm a beginner, so explain each technical term in one sentence the first time it appears.
- Give the conclusion first, then the process. If the process is long, write it to a file and put only the key points in the reply, because I read on my phone and can't get through long paragraphs.

## Editing files
- Read an existing file before changing it. Changing it without reading it makes it easy to edit the wrong place.
- Before changing config-type files (settings, environment variables, startup scripts), make a backup first so a bad edit can be restored.
- Change only the parts I asked for. If you "improve" other things along the way, I can't check them one by one.

## Long tasks
- For a task that will take more than a few minutes, tell me in two or three sentences how you plan to do it before you start, and begin when I say "go."
- When you finish, tell me: which files changed, how you verified it, and what is left undone.

## When you're not sure
- If you aren't sure whether a command or parameter exists, check the official docs or verify by actually running it. Don't write from memory. A wrong command costs me more time than hearing "I'm not sure."

## Progress
- Write progress and next steps in `handoff.md`, not in this file. Read it before you start work.
```

### AGENTS.md (put it at ~/.codex/AGENTS.md, or in the project root)

```markdown
# Working instructions for Codex

This is an unattended task environment; nobody will answer questions in real time. When you are unsure, pick a reasonable approach, finish the work, and state in the final summary what you chose and why.

## How to work
- Before making changes, read the relevant files, understand how things are currently written, and follow the existing style.
- Change only the parts named in the task. If you see problems elsewhere, write them in the summary; don't fix them along the way.
- After changing, verify by actually running it; if tests can run, run them. In the summary, state what you ran and what the result was.

## Delivery
- Put the summary in the final message, within 15 lines: what you did, which files changed, the verification results, and what is left undone.
- Make the summary factual. For anything you didn't verify, write "not verified"; don't present it as done.

## Notes
- Don't write keys, passwords or tokens into any file.
- The `.git` and `.codex` directories in the project are read-only; don't try to modify them.
```

> [!NOTE]
> **Checkpoint**
> After saving, open a new session and ask it: "Which rules have you read right now?" It should be able to restate the key points you wrote. If it can't, check that the file is in the right place, that the file name is spelled correctly, and that you started it in a directory where the file will be read.

## 6.7 When something goes wrong

- **It doesn't seem to have read my rules**: First check the capitalization of the file name (`CLAUDE.md`, `AGENTS.md`) and where it is stored, then confirm that you started in the matching directory. In Claude, `/memory` shows which files it loaded.
- **Rules contradict each other**: The layers of rules stack, so if the global and project files say conflicting things, it cannot tell which one to follow. Keep only one and delete the other.
- **Rules keep piling up and behavior gets stranger**: Go back and ask of each rule, "which failure called for it?" Delete the ones you can't answer for.

### References

- Claude Code memory and CLAUDE.md: https://code.claude.com/docs/en/memory (Checked in October 2026.)
- Claude Code slash commands: https://code.claude.com/docs/en/commands (Checked in October 2026.)
- Claude Code interactive mode and shortcut prefixes: https://code.claude.com/docs/en/interactive-mode (Checked in October 2026.)
- Codex AGENTS.md: https://learn.chatgpt.com/docs/agent-configuration/agents-md (Checked in October 2026.)
- Codex configuration and commands: https://learn.chatgpt.com/docs/config-file/config-advanced , https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)

---
[← Chapter 5. First steps](05-first-steps.md) · [Contents](../README.md#contents) · [Chapter 7. Saving usage: context, caching and compaction →](07-saving-usage.md)
