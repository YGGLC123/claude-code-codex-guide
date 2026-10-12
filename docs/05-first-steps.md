# Chapter 5. First steps

The first four chapters covered installing the tools you need. This chapter walks you through a complete first session: how to start, how to phrase a request, which commands to learn first, how to keep the tool from changing files you didn't expect it to change, and how to adjust it to the way you work. Claude Code and Codex are used in much the same way. This chapter focuses on Claude Code and points out where Codex differs.

## 5.1 Go into a project folder first, then start

Claude Code and Codex both work in the folder you are currently in, and by default they can read and write only within the folder where you started them. So the first step is to give the tool a dedicated folder. Don't start it from the desktop or from your whole user directory.

For your first practice session, create a new empty folder.

Windows (PowerShell):

```powershell
mkdir "$HOME\Documents\first-project"
Set-Location "$HOME\Documents\first-project"
claude
```

macOS / Linux (Terminal):

```bash
mkdir -p ~/Documents/first-project
cd ~/Documents/first-project
claude
```

If you use Codex, replace `claude` on the last line with `codex`.

The first time you start in a folder, the tool asks whether you trust it. Trusting the folder means allowing the tool to read files, modify files and run commands there. Before you agree, confirm that you created the folder yourself and that it contains nothing of unknown origin. The exact wording of the prompt and its options may differ between versions; the official documentation is authoritative.

> [!NOTE]
> **Checkpoint**
> After this step you should see an input box in the terminal waiting for your input. In Claude Code, the interface shows the current model and the folder you are in. If it doesn't appear, run `claude doctor` (`codex doctor` for Codex) to check the installation. If the network connection fails, see [Chapter 13](13-troubleshooting.md).

> [!IMPORTANT]
> **Don't start in your whole user directory**
> If you start in `C:\Users\<your-username>` or at the root of a drive, the tool can see far too much. Reading files becomes both slow and expensive (see [Chapter 7](07-saving-usage.md)), and it is easier for the tool to change something it shouldn't. Use one folder per task.

## 5.2 How to phrase your first request

The most common beginner mistake is a request that is too short, such as "make me a web page". The tool then guesses at many details, and when it guesses wrong you can only correct it round by round. My habit is to write three things:

1. **Goal**: what you want to end up with.
2. **Scope**: which files it may touch, and what it must not touch.
3. **Definition of done**: how you will confirm that it did the job correctly.

Then add one more: **explain the plan before acting**. Ask it to say how it intends to proceed, and let it modify files only after you confirm. Here is a complete example that you can adapt with your own content:

```text
I want to build a single-file web page in this folder: a simple to-do list where I can add items, check them off and delete them, and the items are still there after I refresh the page.

Scope: create only one new file called todo.html. Don't install any dependencies, and don't touch anything else in the folder.

Definition of done:
1. Double-clicking todo.html opens it in a browser;
2. After I add three items, check one of them off and refresh the page, all three items are still there and the checked state is preserved;
3. The layout doesn't break at phone width (when the window is made very narrow).

Before you start, tell me in no more than 10 lines how you plan to do it, and wait for me to say "go" before you write any code.
```

After you send this, it replies first with a plan. If the plan looks right, reply "go". When it finishes, it should explain how to verify the result; check each item of the definition of done yourself.

> [!NOTE]
> **Checkpoint**
> After this step you should see a new `todo.html` in the folder, and when you open it in a browser you can add and delete items. If it starts working without reporting a plan first, next time repeat the instruction in the last sentence of your prompt, or press Shift+Tab first to switch to plan mode (see Section 5.4).

> [!TIP]
> **When it gets something wrong, say so directly**
> There is no need to be polite, and no need to start over. Write something like "the text doesn't turn gray after I check an item; make it gray with a strikethrough". A small, specific change is quick for it to make. Asking for one or two changes at a time is more reliable than listing ten at once.

## 5.3 Common slash commands

Typing something that starts with `/` in the input box gives a command to the tool itself, not a message to the AI. The commands below are enough for everyday use.

### Claude Code

| Command | Purpose |
|---|---|
| `/init` | Generate a `CLAUDE.md` for the current project (project instructions, covered in [Chapter 6](06-memory-and-rules.md)) |
| `/clear` | Start a new conversation with empty context (aliases `/reset`, `/new`); you can get the old conversation back with `/resume` |
| `/compact` | Compress the earlier conversation into a summary to free up context, covered in [Chapter 7](07-saving-usage.md) |
| `/model` | Switch the model |
| `/effort` | Set the reasoning effort |
| `/resume` | Resume an earlier conversation by name or number, or open the picker |
| `/rewind` | Roll the conversation and code back to an earlier point (aliases `/checkpoint`, `/undo`) |
| `/memory` | Edit `CLAUDE.md` and turn automatic memory on or off |
| `/permissions` | Manage which actions are allowed, require asking, or are forbidden |
| `/config` | Open the settings interface |
| `/context` | Show current context usage as a colored grid |
| `/usage` | Show the spending for this session and your plan usage; `/cost` is now just an alias for it |
| `/status` | Show the version, model, account and connection status |
| `/plan` | Enter plan mode directly |
| `/help` | Help |
| `/exit` | Exit |

Some older tutorials are out of date on these points: `/cost` is only an alias of `/usage`; since v2.1.198, `/agents` only prints a reminder and is no longer a management interface; `/vim` has been removed.

Two other symbols at the start of the input line are very useful:

- Type `!` at the start of the line to enter shell mode. What follows is run directly as a command, and the output is added to the conversation. For example, `!git status`.
- Type `@` to autocomplete a file path and point the tool at a specific file. For example, `@todo.html change the title to "My To-Dos"`.

Some older tutorials say that starting a line with `#` saves it to memory. I couldn't find that in the official table of shortcut prefixes, so treat the official documentation as authoritative and don't rely on it. To make the tool remember something, see [Chapter 6](06-memory-and-rules.md).

### Codex

| Command | Purpose |
|---|---|
| `/model` | Choose the model (and reasoning effort) |
| `/permissions` | Switch between presets such as Auto and Read Only |
| `/init` | Generate a draft `AGENTS.md` in the current directory |
| `/compact` | Compress the earlier conversation |
| `/new` | Start a new conversation in the same interface |
| `/clear` | Clear the screen and start a new conversation |
| `/resume` | Pick a saved session to continue |
| `/status` | Show the current configuration and token usage |
| `/usage` | Show the account's token usage |
| `/diff` | Show Git changes, including untracked files |
| `/review` | Ask it to review the changes in the working tree |
| `/plan` | Switch to plan mode |
| `/mention` | Attach a file to the conversation |

Codex's official command table has no `/approvals`; to change permissions, use `/permissions`.

> [!TIP]
> **If you forget a command, ask the tool**
> In Claude Code, type `/help`, or simply ask "how do I go back to the previous step". Claude Code can also answer questions about how to use itself.

## 5.4 Permission modes: how much it does on its own

When the AI needs to modify files or run commands, the setting that decides whether it asks you first is the permission mode.

### Claude Code

Press Shift+Tab to cycle through several modes (on Windows, if nothing happens, use Alt+M). The official documentation lists six; as a beginner, start with these four:

| Mode | Behavior | Good for |
|---|---|---|
| `default` (called Manual in the interface) | Only reading needs no approval; it stops to ask before editing files, running commands or accessing the network | Sensitive work, or when you want to review every step |
| `acceptEdits` | Reading, editing files and common file commands (creating folders, copying, moving) need no approval | Everyday work where you review as you go |
| `plan` | Read-only exploration; it proposes a plan and changes no files until you approve | Looking at the plan before it acts, or working in an unfamiliar project |
| `auto` | No confirmation prompts for any action; a separate classifier model reviews actions in the background and blocks anything beyond what you asked for | A convenient setting once you are familiar with the tool |

Since v2.1.283, `auto` is the default starting mode for interactive terminals and VS Code sessions. Note the official warning: it reduces prompts, but it does not guarantee safety. There are also `dontAsk` (allows only reads inside the directory and pre-approved tools, and refuses everything else; suited to scripts) and `bypassPermissions` (never asks; the official documentation recommends it only inside an isolated container or virtual machine, and the command-line form is `--dangerously-skip-permissions`).

To fix a starting mode, use `claude --permission-mode <mode-name>` for a single launch. For long-term use, set `permissions.defaultMode` in `~/.claude/settings.json`. Setting `auto` or `bypassPermissions` in the project's `.claude/settings.json` has no effect; it must go in the user-level file.

### Codex

Codex splits "what it can do" and "when it asks you" into two layers:

- The **sandbox** controls what is technically possible: `read-only`, `workspace-write` (can write only in the workspace) and `danger-full-access` (unrestricted). On the command line, use `--sandbox` or `-s`.
- **Approval** controls when it asks you first. Currently only two values are commonly used: `on-request` and `never`. `untrusted` is no longer supported, and putting it in the configuration can make startup fail; `on-failure` is deprecated; `--full-auto` is also deprecated, so new scripts should use `--sandbox workspace-write` instead.

In a folder managed by Git, the default recommendation is Auto (the workspace is writable, and it asks only when needed). In a folder not under version control, the default recommendation is read-only. By default there is no network access, and it can write only to the current workspace.

In the interactive interface, use `/permissions` when you need to switch. The fully open form is `--dangerously-bypass-approvals-and-sandbox` (alias `--yolo`), which the official documentation explicitly does not recommend.

### When to loosen permissions and when not to

My own practice:

- **A new folder, or one with a backup or Git**: use `acceptEdits` or `auto`, so you don't have to click approve at every step.
- **A folder with important original material and no backup**: start with `default` or `plan` so it asks at every step, or copy the folder first and let it work on the copy.
- **Deleting, overwriting or batch renaming**: in any mode, have it list what it will touch first so you can review the list, and act only after you confirm (see [Chapter 12](12-verification-and-safety.md) for details).
- **Never enable `bypassPermissions` or `--yolo` in a folder with your real files**: in these modes it can delete anything.

> [!IMPORTANT]
> Before you loosen permissions, make sure everything in the folder is backed up. However reliable the AI is, it can misread what you mean. For example, it may understand "clean up temporary files" as cleaning up your drafts, and this kind of thing does happen.

## 5.5 Interrupting and rolling back

It is partway through and you see it going in the wrong direction:

- Press **Esc** once: this interrupts its current reply or action, and the work already done is kept. You can then continue with "Stop, change it to ...".
- Press Esc twice when the input box has text: this clears the input. Press Esc twice when the input box is empty: this opens the rewind menu, where you choose which earlier step to return to, and both the conversation and the code can be restored together.
- Type `/rewind` (aliases `/checkpoint`, `/undo`): the same rewind feature, opened by command.
- **Ctrl+C**: interrupts. When idle, the first press clears the input and the second press exits.

Pressing Esc on a permission prompt is the same as refusing that one action.

> [!TIP]
> **Rewind is not a cure-all**
> Rewind mainly covers files the tool modified and the conversation itself. It may not be able to undo external commands it ran, such as content that was already uploaded or files that were deleted but not created by the tool. For important files, you still need backups or Git.

## 5.6 Pick up where you left off

After you close the terminal, the earlier conversation is still saved, and you can continue it.

Claude Code:

```text
claude -c
claude -r
```

`-c` continues the most recent conversation in the current folder. `-r` opens a picker where you choose which conversation to continue, or you can pass a session ID directly. Typing `/resume` inside a conversation has the same effect.

Codex:

```text
codex resume --last
codex resume
```

`--last` continues the most recent session in the current folder. Without arguments, it opens the session picker. The non-interactive mode (`codex exec`, covered in [Chapter 8](08-division-of-labor.md)) has a corresponding `codex exec resume`.

> [!WARNING]
> **Open each conversation in only one place**
> The problem I hit: I opened a conversation on the command line with `--resume`, and then VS Code automatically restored the same conversation after a restart. The two copies wrote to the same record in an interleaved way, and in one case both windows assigned tasks on their own and the tasks conflicted.
> The fix: keep any one conversation open in only one place at a time. Before you open a conversation in VS Code, make sure the copy on the command line has exited.

## 5.7 Using it in VS Code

For installing the VS Code extension, see [Chapter 2](02-install-claude-code.md). Usage in VS Code is largely the same as on the command line, with added conveniences such as clicking and dragging files into the conversation. I also recommend one setting: collapse tool calls.

Without collapsing, every file read, file edit and command takes up a large block of the conversation, and the conclusions you actually need to read get pushed out of view. I turned on the setting `claudeCode.focusView` (you can also toggle it with Ctrl+Alt+F). Tool calls then collapse into a single line, and only the conclusions stay visible. I use this setting name and shortcut myself, but I haven't checked them word for word against the official documentation, so they may differ between versions, and the official documentation is authoritative. If you can't find the setting, your version has probably renamed it. Search for "Claude Code" in the settings to find a similar option.

## 5.8 Continue from your phone

Even when you are away from your computer, you can check progress and give instructions from your phone.

**Claude**: this feature is called **Remote Control**. After you start it on your computer, you can continue the sessions on that computer from claude.ai/code or the Claude phone app (iOS, Android). The code always runs on your computer, so **the computer must stay on and the `claude` process must keep running**. There are three ways to start it:

- `claude remote-control`: server mode.
- `claude --remote-control` (short form `--rc`): opens remote access while keeping the interactive session.
- Type `/remote-control` (short form `/rc`) in a conversation.

It requires a Pro, Max, Team or Enterprise subscription; an API key does not work. There is no separate Claude Code app on the phone; sessions appear in the **Code** tab of the Claude app. Type `/mobile` in a conversation, and the terminal shows a QR-style pattern you can scan to download the phone app.

**Codex**: through the ChatGPT phone app, you can run tasks in the cloud, or remotely control a Mac or Windows computer that is already connected (approve commands, view changes). The connection must be started from Settings > Connections in the desktop app; the command line and IDE extensions can't start it. The official pages are inconsistent about which phone operating systems are supported, so check the official documentation.

> [!IMPORTANT]
> Every "approve" you give on your phone approves an action that actually happens on your computer. Don't tap approve casually on your phone, especially for anything involving deletion or network access.

## 5.9 Tune it to your habits

The defaults work, but I set up the following four things in every new project. [Chapter 6](06-memory-and-rules.md) explains how to write them into `CLAUDE.md`; here I only describe what to do and why.

**1. Reply in English every time.** When the material is in another language, the tool easily drifts into that language, even if you keep asking in English. I had to ask it about this three times in three days. Write it as a rule:

```markdown
Reply in English every time, including the short progress notes between tool calls. This holds even when the material is in another language.
```

**2. Use text, not popup menus, when you need to make a choice.** When a selection menu popped up, I rejected it twice in a row, because I wanted to see the reasons before choosing. Have it list one to three options in text, state the consequence of each, and give one recommendation. For any option you haven't answered, it follows its recommendation:

```markdown
When I need to make a choice, list 1 to 3 options in text and give a recommendation. Don't use popup selection menus. For any option I haven't answered, go with your recommendation.
```

**3. Report only when there is a result or an exception.** Daily reports on routine progress just burn your attention. An exception report needs to cover three things: what happened, what it affects, and what the tool plans to do about it or what it needs you to decide.

```markdown
Report to me only when there is a result or an exception; don't report routine progress. For an exception, state three things: what happened, what it affects, and what you plan to do about it or what you need me to decide.
```

**4. For long tasks, say what you will do and roughly how long it will take.** For anything that will take more than a few minutes, have it say "I plan to do X, which will take about this long" first, and start only after you say "go". I learned this the hard way: I only expressed approval of a direction, and it started a long task in the background that interfered with what I was looking at.

```markdown
For a task expected to take more than a few minutes, first tell me what you plan to do and roughly how long it will take, and wait for me to say go before you start.
```

> [!NOTE]
> **Checkpoint**
> After this step you should see this: start a new conversation and ask any question that involves material in another language. It should still answer in English. If it doesn't, use `/memory` to check whether the rule was written into the right `CLAUDE.md`.

That completes the getting-started route. You can now install the tools, start them, phrase a request, control permissions, roll back, and continue an earlier conversation. The chapters that follow are about using the tools more economically and more reliably: [Chapter 6](06-memory-and-rules.md) covers how to make the tool remember you, [Chapter 7](07-saving-usage.md) covers how to use less of your usage limits, and [Chapter 13](13-troubleshooting.md) is a quick reference for errors.

### References

- Claude Code commands: https://code.claude.com/docs/en/commands (Checked in October 2026.)
- Claude Code interactive mode and keyboard shortcuts: https://code.claude.com/docs/en/interactive-mode (Checked in October 2026.)
- Claude Code permission modes: https://code.claude.com/docs/en/permission-modes (Checked in October 2026.)
- Claude Code Remote Control: https://code.claude.com/docs/en/remote-control (Checked in October 2026.)
- Claude Code on mobile: https://code.claude.com/docs/en/mobile (Checked in October 2026.)
- Codex commands and slash commands: https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)
- Codex approvals and sandbox: https://learn.chatgpt.com/docs/agent-approvals-security (Checked in October 2026.)
- Codex non-interactive mode: https://learn.chatgpt.com/docs/non-interactive-mode (Checked in October 2026.)
- Codex mobile and remote connections: https://learn.chatgpt.com/docs/mobile , https://learn.chatgpt.com/docs/remote-connections (Checked in October 2026.)

---
[← Chapter 4. Environment setup: Git, Node.js, Python and common Windows problems](04-environment-setup.md) · [Contents](../README.md#contents) · [Chapter 6. Making it remember you: CLAUDE.md, AGENTS.md and memory →](06-memory-and-rules.md)
