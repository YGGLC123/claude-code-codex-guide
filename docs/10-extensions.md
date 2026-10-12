# Chapter 10. Extensions: Skills, MCP, hooks and plugins

The earlier chapters covered how to get Claude to carry out tasks. This chapter covers how to extend it: Skills teach it your way of working, MCP gives it new tools, hooks make certain actions happen automatically, and plugins bundle these into one thing you install.

First, my principle: **more extensions is not better.** Every extra one can take up more context and start more background processes. Each section below explains how I keep the count down.

## 10.1 Skills: write your way of working as a manual

### What it is

A Skill is a folder whose core is a `SKILL.md` file. You write down "how to handle this kind of task", and Claude decides from the description whether to use it. You can also call it manually by typing `/skill-name`.

### Where it lives

| Scope | Location |
|---|---|
| Personal (available in all projects) | `~/.claude/skills/<skill-name>/SKILL.md` |
| Project (this project only) | `.claude/skills/<skill-name>/SKILL.md` |
| Bundled with a plugin | `<plugin>/skills/<skill-name>/SKILL.md`; the call name is `/plugin-name:skill-name` |

On Windows, `~` is `C:\Users\<your-username>`. When skills with the same name conflict, organization-level wins over personal, and personal wins over project.

> [!TIP]
> You may have seen the older "custom commands" (`.claude/commands/deploy.md`). They have now been merged into skills: both `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` produce `/deploy`. Old files still work, so there is no need to migrate in a hurry.

### Writing the simplest skill

At the top of `SKILL.md`, the part wrapped between two `---` lines is the front matter. The most important field in it is `description`: it decides when Claude thinks of this skill. `name` can be left out and defaults to the folder name. After that comes ordinary Markdown instructions.

For example, create a "commit message" skill. Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\commit-style"
notepad "$env:USERPROFILE\.claude\skills\commit-style\SKILL.md"
```

macOS / Linux (Terminal):

```bash
mkdir -p ~/.claude/skills/commit-style
nano ~/.claude/skills/commit-style/SKILL.md
```

Paste in the following and save:

```markdown
---
description: Use when the user asks for a Git commit message; defines the commit message format
---

When writing a commit message, follow these rules:
1. The first line is at most 50 characters and says what changed.
2. Leave one blank line, then use two or three sentences to say why it changed.
3. Do not write empty phrases such as "optimize code".
```

> [!NOTE]
> **Checkpoint**
> Start a new Claude session and ask it to "write a commit message for the changes just made". It should respond in the format you wrote, and sometimes it will mention in its reply that it used the skill. You can also type `/commit-style` to call it manually. If nothing happens, first check that the file is named exactly `SKILL.md` and that it sits at the right folder level.

`SKILL.md` has some optional fields too, such as restricting which tools it may use (`allowed-tools`) and choosing a model (`model`). Check the official docs when you need them.

### When you have many skills: write a "task to preferred skill" table

I have installed quite a few skills, and many of them overlap: there are several for writing papers and several for drawing figures. With that much overlap, Claude picks the wrong one or misuses one. My fix is a table in `CLAUDE.md` (see [Chapter 6](06-memory-and-rules.md)):

```markdown
## Skill selection table
| Task | Preferred skill |
|---|---|
| Writing and polishing papers | Skill A |
| Scientific figures | Skill B |
| Creating Word and Excel files | Skill C |

Rule: when a task clearly matches an item in the table above, tell me in one sentence at the start of your reply, "this could use XX, want to use it?", and wait for my decision. If I name a skill directly, just use it. Do not mention this for casual chat or small questions.
```

The benefit: it won't choose skills you don't need on its own, and when you name one, it won't ask again.

## 10.2 MCP: connect Claude to new tools

### What it is

MCP is a unified interface that lets Claude call external tools, such as a browser, a database, note-taking software or design software. Each MCP server provides a set of tools, and Claude calls them when needed.

### Three ways to add one

**Remote service (HTTP):**

```bash
claude mcp add --transport http <name> <url>
```

The official example is `claude mcp add --transport http notion https://mcp.notion.com/mcp`.

**Remote service (SSE, the older way):**

```bash
claude mcp add --transport sse <name> <url>
```

**Local program (stdio):**

```bash
claude mcp add --transport stdio <name> -- <start command> <arguments>
```

Everything after `--` is passed to the server program as is. When you need environment variables, put `--env KEY=value` before it. Official example: `claude mcp add --env KEY=value --transport stdio myserver -- python server.py --port 8080`.

To supply a piece of JSON in one go, use `claude mcp add-json <name> '<json>'`. To see what you already have:

```bash
claude mcp list
claude mcp get <name>
```

You can choose any `<name>`. For `<url>` and `<start command>`, look in the official instructions of the MCP service you want to connect.

### Scope: who it is installed for

Add `-s` (or `--scope`) to choose:

| Scope | Who can use it | Where it is stored |
|---|---|---|
| `local` (default) | Only you, only in the current project | Under that project's name in `~/.claude.json` |
| `project` | Shared with the team | `.mcp.json` in the project root; needs approval on first use |
| `user` | Only you, in all projects | `~/.claude.json` |

When unsure, use the default `local`; it affects your other projects the least.

### An example: have it test a web page you built

You built a web page with Claude and want it to open the page itself and check for problems. You can connect a Playwright MCP. It starts a browser and lets Claude click, fill in forms and take screenshots.

I recommend only this one use here: **testing a web page you built yourself and verifying it with screenshots.** For the install command, follow the official Playwright MCP instructions. The format is the stdio form shown above:

```bash
claude mcp add --transport stdio playwright -- <official start command>
```

Once it is connected, tell Claude:

```text
Open http://localhost:<port> in the browser, click through every button on the page, take a screenshot at each step, and tell me what is wrong.
```

`<port>` is the number your local web server shows.

> [!NOTE]
> **Checkpoint**
> You should see Claude call a browser tool, a browser window pop up (or run in the background), and finally hand you a set of check results with screenshots. If it says it doesn't have this tool, first run `claude mcp list` to see whether `playwright` is listed, then reopen the session.

> [!IMPORTANT]
> Do not let browser automation operate your accounts on websites, chat sites or social platforms. Platforms treat it as abnormal automation: at best your account is rate-limited, at worst it is banned. [Chapter 12](12-verification-and-safety.md) covers this in detail.

### Enable on demand; avoid a bloated global config

I learned this one the hard way. **Every session, when it starts, launches all the MCP servers in your config.** For convenience I once configured ten browser channels, then opened ten windows at once. The result was about 500 processes using 18 GB of memory. I later cut it down to two, and I only connect them when I actually need them.

What I do now:

- Only the servers I use constantly and that every project needs go in the `user` scope.
- A server that only one project needs goes in `local` or `project`.
- A server needed temporarily is removed when I'm done (check the official docs for the removal command, or just ask Claude to edit the config).

> [!WARNING]
> **Common mistake: the number of MCP servers keeps growing**
> The problem I ran into: the machine got slower and slower, and Task Manager showed a large number of browser processes.
> The fix: run `claude mcp list` to see what is attached and remove everything you don't use. Attach fewer servers especially when several windows are open at the same time.

## 10.3 Hooks: make certain things happen automatically

### What it is

Hooks are small commands that run automatically at a certain point in Claude's work, for example "check formatting every time a file has been changed". A hook is a setting in settings.json and is executed by Claude Code itself, **not by relying on Claude to remember**, so it is far more reliable than writing "remember to check every time" in a prompt.

### Which events exist

There are many events. The ones beginners use most are these:

| Event | When it fires |
|---|---|
| `SessionStart` | A session starts |
| `UserPromptSubmit` | When you send a message |
| `PreToolUse` | Before it uses a tool |
| `PostToolUse` | After it has used a tool |
| `Stop` | When it finishes answering a turn |
| `PreCompact` / `PostCompact` | Before / after context compaction |
| `Notification` | When it needs your attention |

The official list has many more events, such as subagents, tasks, file changes and model switching; check the official docs for the full list. There are five handler types: `command` (run a command), `http`, `mcp_tool`, `prompt` and `agent`. As a beginner, `command` is all you need.

### A simple example: run a format check after files change

The idea: on the `PostToolUse` event, after a tool that edits files has been used, run your project's format-check command.

I won't write the hooks JSON from memory here; a single misplaced bracket makes the whole config stop working. The most reliable approach is to have Claude write it for you:

```text
Add a PostToolUse hook to the project's .claude/settings.json: every time you finish editing a file, automatically run <your format-check command>. Before writing it, check the official hooks docs to confirm the format, and when you're done, show it to me with /hooks.
```

Replace `<your format-check command>` with whatever your project actually uses. For a frontend project it is commonly `npx prettier --check .`; for a Python project it can be whichever checking tool you installed.

When it is written, type `/hooks` to see the hooks currently in effect.

> [!NOTE]
> **Checkpoint**
> Type `/hooks`; the `PostToolUse` hook you just added should appear in the list. Then ask Claude to edit any file, and you should see the check command run.

### Common mistakes with hook commands

A hook may run on every tool call, so **the cost of being slow or failing is multiplied**. I ran into two problems:

> [!WARNING]
> **Common mistake: do not use the py launcher in hooks**
> The problem I ran into: I wrote the hook command as `py script.py`, and it was slow every time. Once I got the script path wrong, Python returned exit code 2, and after that every tool call reported an error. A `PostToolUse` hook runs on every tool call, and the `py` launcher is roughly twice as slow as calling `python.exe` directly.
> The fix: in a hook, write the full path to `python.exe` and add the two flags `-S -E` (skip site packages and environment variables, so it starts faster). When you want to disable a hook, **first delete the hook entry in settings, and only then touch the script**; the other way around leaves the hook unable to find the script and erroring continuously.

Two more points that I verified by testing: a hook newly added to the project settings.json is hot-loaded, so you don't need to reopen the session; and a subagent definition added in the middle of a session can't be used by the current session.

### Reading context usage in the status line

Claude has no tool of its own to know how much context has been used. The way to get it is the status line: type `/statusline` to configure it. You can describe the look you want, or run it without arguments and let it generate one from your shell prompt. The data passed to the status line script includes a context usage percentage (the field I use is `context_window.used_percentage`; check the official docs for the exact name). I have my status line script also write that number to a file, so other scripts can read the real usage and remind me to write a handoff note before compacting when it is nearly full.

In addition, a `SessionStart` hook with the compaction trigger condition can write key information back in after compaction, because the compaction summary loses detail. Check the official docs for the exact matching syntax.

## 10.4 Plugins: install a whole set at once

### What it is

A plugin can bundle skills, subagents, hooks, MCP servers and more. Installing one plugin installs the whole set at once.

### How to install

The official marketplace is called `claude-plugins-official`, and it is added automatically the first time you enter an interactive session. Inside a session:

```text
/plugin install <plugin-name>@<marketplace-name>
```

Official example:

```text
/plugin install commit-commands@claude-plugins-official
```

To add someone else's marketplace:

```text
/plugin marketplace add <author-name>/<repo-name>
```

It also accepts git URLs, local paths and `marketplace.json` URLs. In a regular terminal, the matching commands are `claude plugin install`, `claude plugin marketplace add` and `claude plugin list`.

> [!NOTE]
> **Checkpoint**
> After installation, run `claude plugin list`. You should see the plugin you just installed. A skill bundled with a plugin is called with `/plugin-name:skill-name`.

> [!IMPORTANT]
> A plugin is code and prompts written by someone else, and it runs on your computer. Install only plugins whose source is clear, and don't add marketplaces of unknown origin.

### Turn off plugins that stuff large blocks of text into every turn

I only discovered this after an audit: one skill plugin pack **injects a large block of text into the context on every turn, along the lines of "extremely important, you must use this"**. The statistics showed it came with 14 skills, of which I had actually used 3 times in total. That is context taken up for nothing, and it burns usage on every turn.

How to check:

- Look at how much context is taken up at the start of a session ([Chapter 7](07-saving-usage.md) explains how to see this).
- Go through the plugins one by one with `claude plugin list` and ask yourself, "Have I used this in the past month?"
- Disable the ones you haven't used, and reinstall them when you really need them.

## 10.5 The equivalents in Codex

Codex has the same four things, with almost the same names; only the locations and syntax differ.

**MCP.** Two kinds are supported: local processes (STDIO) and Streamable HTTP. The desktop app, the command line and the IDE share one configuration.

```bash
codex mcp add <name> --env VAR1=VALUE1 -- <start command>
codex mcp list
```

Official example: `codex mcp add context7 -- npx -y @upstash/context7-mcp`. For servers that need login authorization, use `codex mcp login <name>`; in a session, type `/mcp` to view them. You can also write them straight into `config.toml`:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]

[mcp_servers.context7.env]
MY_ENV_VAR = "MY_ENV_VALUE"
```

Remote services use `url`. Keep the key in an environment variable and reference it with `bearer_token_env_var`; don't write the key straight into the file. There are also fields you can tune, such as `startup_timeout_sec` (default 10 seconds) and `tool_timeout_sec` (default 60 seconds).

**Skills.** Again a directory plus a `SKILL.md`, but both `name` and `description` are required. The locations differ:

| Scope | Location |
|---|---|
| Repository | `.agents/skills` at every level from the current directory up to the repository root |
| Personal | `$HOME/.agents/skills` |

Usage: in the command line or IDE, type `/skills`; or type `$` followed by the skill name to call one directly. It also matches skills automatically by their description. If you want help writing a skill, use the built-in `$skill-creator`. The official docs mark the older "custom prompts" as deprecated in favor of skills.

**Hooks.** On by default. To turn them off, write this in `config.toml`:

```toml
[features]
hooks = false
```

The configuration goes in `~/.codex/hooks.json`, `~/.codex/config.toml`, or the project's `.codex/hooks.json` and `.codex/config.toml` (the project layer must be trusted before it loads). The official example is a check before running a Bash command:

```toml
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = '/usr/bin/python3 "$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py"'
timeout = 30
statusMessage = "Checking Bash command"
```

This is the macOS / Linux path form; on Windows, replace the command with the path to your own script.

> [!IMPORTANT]
> **Codex hooks must be trusted first**
> Hooks that are not managed take effect only after you review and trust them by typing `/hooks` in a session. Trust is recorded by content, so if you change a hook's content you have to trust it again.

**Plugins.** The commands are `codex plugin add`, `codex plugin list` and `codex plugin remove`. To add a plugin marketplace, use `codex plugin marketplace add <author-name>/<repo-name>`; in a session, type `/plugins`.

> [!TIP]
> The thinking is the same on both sides: skills teach methods, MCP provides tools, hooks handle automation, and plugins handle packaging. The habit you build with Claude of attaching only on demand and cleaning up regularly applies equally to Codex.

### References

- https://code.claude.com/docs/en/skills (Checked in October 2026.)
- https://code.claude.com/docs/en/sub-agents (Checked in October 2026.)
- https://code.claude.com/docs/en/hooks (Checked in October 2026.)
- https://code.claude.com/docs/en/mcp (Checked in October 2026.)
- https://code.claude.com/docs/en/discover-plugins (Checked in October 2026.)
- https://code.claude.com/docs/en/commands (Checked in October 2026.)
- https://learn.chatgpt.com/docs/extend/mcp (Checked in October 2026.)
- https://learn.chatgpt.com/docs/build-skills (Checked in October 2026.)
- https://learn.chatgpt.com/docs/hooks (Checked in October 2026.)
- https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)

---
[← Chapter 9. Subagents and background work: getting AI to follow long-running tasks](09-subagents-and-background-work.md) · [Contents](../README.md#contents) · [Chapter 11. Using other model providers →](11-other-model-providers.md)
