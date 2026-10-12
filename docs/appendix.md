# Appendix. Command reference and config templates

This appendix is for looking things up; you don't need to read it from the start. Every command and template in the tables appears in an earlier chapter, written exactly as it is here. If you want to know why something is written that way, see the chapter in the right-hand column. Anything inside `<angle brackets>` is a placeholder for you to replace with your own value; the main text explains where to get each one.

## A. Command reference

### A1. Claude Code

| Command | Purpose | See |
|---|---|---|
| `irm https://claude.ai/install.ps1 \| iex` | Install on Windows (PowerShell) | [Ch. 2](02-install-claude-code.md) |
| `curl -fsSL https://claude.ai/install.sh \| bash` | Install on macOS / Linux | [Ch. 2](02-install-claude-code.md) |
| `winget install Anthropic.ClaudeCode` | Install on Windows with WinGet | [Ch. 2](02-install-claude-code.md) |
| `brew install --cask claude-code` | Install on macOS with Homebrew | [Ch. 2](02-install-claude-code.md) |
| `npm install -g @anthropic-ai/claude-code` | Install with npm | [Ch. 2](02-install-claude-code.md) |
| `claude --version` | Show the version; confirms the install worked | [Ch. 2](02-install-claude-code.md) |
| `claude doctor` | Diagnose the installation and settings | [Ch. 2](02-install-claude-code.md) |
| `claude update` | Update manually | [Ch. 2](02-install-claude-code.md) |
| `where.exe claude` | On Windows, show where claude is installed | [Ch. 2](02-install-claude-code.md) |
| `code --install-extension anthropic.claude-code` | Install the VS Code extension | [Ch. 2](02-install-claude-code.md) |
| `claude` | Start in the current folder | [Ch. 5](05-first-steps.md) |
| `claude -c` | Continue the most recent conversation in the current folder | [Ch. 5](05-first-steps.md) |
| `claude -r` | Open the picker and choose a conversation to resume | [Ch. 5](05-first-steps.md) |
| `claude -p --model sonnet "<prompt>"` | Run non-interactively and exit when done | [Ch. 2](02-install-claude-code.md) |
| `claude --permission-mode <mode-name>` | Set the starting permission mode for this run | [Ch. 5](05-first-steps.md) |
| `claude mcp add --transport http <name> <url>` | Add a remote MCP server | [Ch. 10](10-extensions.md) |
| `claude mcp list` | List the MCP servers you have added | [Ch. 10](10-extensions.md) |
| `claude mcp get <name>` | Show the details of one MCP server | [Ch. 10](10-extensions.md) |

Slash commands inside a conversation:

| Command | Purpose | See |
|---|---|---|
| `/init` | Generate the project's `CLAUDE.md` | [Ch. 6](06-memory-and-rules.md) |
| `/memory` | Edit `CLAUDE.md` and toggle auto memory | [Ch. 6](06-memory-and-rules.md) |
| `/clear` | Start a new conversation with an empty context | [Ch. 5](05-first-steps.md) |
| `/compact` | Compress the earlier conversation into a summary | [Ch. 7](07-saving-usage.md) |
| `/autocompact 600k` | Set the auto-compaction window | [Ch. 7](07-saving-usage.md) |
| `/model` | Switch models | [Ch. 7](07-saving-usage.md) |
| `/effort medium` | Set the reasoning effort | [Ch. 7](07-saving-usage.md) |
| `/resume` | Resume an earlier conversation | [Ch. 5](05-first-steps.md) |
| `/rewind` | Roll back the conversation and the code | [Ch. 5](05-first-steps.md) |
| `/permissions` | Manage allowed, ask-first and denied actions | [Ch. 5](05-first-steps.md) |
| `/context` | Show how much of the context is used | [Ch. 5](05-first-steps.md) |
| `/usage` | Show this session's cost and your plan usage | [Ch. 5](05-first-steps.md) |
| `/status` | Show the version, model, account and connections | [Ch. 5](05-first-steps.md) |
| `/plan` | Go straight into plan mode | [Ch. 5](05-first-steps.md) |
| `/hooks` | Show the hooks in effect | [Ch. 10](10-extensions.md) |
| `/plugin install <plugin-name>@<marketplace-name>` | Install a plugin | [Ch. 10](10-extensions.md) |
| `/help` | Help | [Ch. 5](05-first-steps.md) |

Two prefixes work in the input box: `!` at the start of a line runs a command directly (for example `!git status`), and `@` completes a file path.

### A2. Codex

| Command | Purpose | See |
|---|---|---|
| `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 \| iex"` | Install on Windows | [Ch. 3](03-install-codex.md) |
| `curl -fsSL https://chatgpt.com/codex/install.sh \| sh` | Install on macOS / Linux | [Ch. 3](03-install-codex.md) |
| `brew install --cask codex` | Install on macOS with Homebrew | [Ch. 3](03-install-codex.md) |
| `npm install -g @openai/codex` | Install with npm | [Ch. 3](03-install-codex.md) |
| `codex login` | Sign in with your ChatGPT account | [Ch. 3](03-install-codex.md) |
| `codex --version` | Show the version | [Ch. 3](03-install-codex.md) |
| `codex doctor` | Diagnose | [Ch. 3](03-install-codex.md) |
| `codex update` | Update | [Ch. 3](03-install-codex.md) |
| `codex` | Start in the current folder | [Ch. 3](03-install-codex.md) |
| `codex --search` | Start with web search turned on | [Ch. 3](03-install-codex.md) |
| `codex exec --skip-git-repo-check "Reply with exactly: OK"` | Smoke test to confirm the connection works | [Ch. 3](03-install-codex.md) |
| `codex exec -m gpt-6.1-sol "<your task>"` | Run non-interactively with a specified model | [Ch. 7](07-saving-usage.md) |
| `codex exec --sandbox workspace-write ...` | Non-interactive, restricted to writing in the workspace | [Ch. 3](03-install-codex.md), [Ch. 8](08-division-of-labor.md) |
| `codex exec resume --last "<follow-up instruction>"` | Continue the most recent non-interactive session in this directory | [Ch. 8](08-division-of-labor.md) |
| `codex exec resume <session-id> "<instruction>"` | Continue a specific session | [Ch. 9](09-subagents-and-background-work.md) |
| `codex resume --last` | Continue the most recent session in the current folder | [Ch. 5](05-first-steps.md) |
| `codex resume` | Open the session picker | [Ch. 5](05-first-steps.md) |
| `codex mcp list` | List the MCP servers you have added | [Ch. 10](10-extensions.md) |

Slash commands inside a Codex conversation:

| Command | Purpose | See |
|---|---|---|
| `/model` | Choose the model and reasoning effort | [Ch. 5](05-first-steps.md) |
| `/permissions` | Switch between presets such as Auto and Read Only | [Ch. 5](05-first-steps.md) |
| `/init` | Generate a draft `AGENTS.md` | [Ch. 6](06-memory-and-rules.md) |
| `/compact` | Compress the earlier conversation | [Ch. 5](05-first-steps.md) |
| `/new` | Start a new conversation | [Ch. 5](05-first-steps.md) |
| `/resume` | Pick an old session to continue | [Ch. 5](05-first-steps.md) |
| `/status` | Show the configuration and token usage | [Ch. 5](05-first-steps.md) |
| `/diff` | Show the Git changes | [Ch. 5](05-first-steps.md) |
| `/review` | Have it review the changes in the workspace | [Ch. 5](05-first-steps.md) |
| `/plan` | Switch to plan mode | [Ch. 5](05-first-steps.md) |

There is no `/approvals`; use `/permissions` to change permissions. `--full-auto` is deprecated, so new scripts should use `--sandbox workspace-write`.

### A3. Environment checks

| Command | Purpose | See |
|---|---|---|
| `curl.exe -o NUL -w "%{http_code}" https://api.anthropic.com/v1/messages` | On Windows, test whether the API endpoint is reachable; a 405 response means it is | [Ch. 13](13-troubleshooting.md) |
| `git --version` | Check Git | [Ch. 4](04-environment-setup.md) |
| `node --version` | Check Node.js | [Ch. 4](04-environment-setup.md) |
| `npm --version` | Check npm | [Ch. 4](04-environment-setup.md) |
| `py --version` | Check Python on Windows | [Ch. 4](04-environment-setup.md) |
| `python3 --version` | Check Python on macOS / Linux | [Ch. 4](04-environment-setup.md) |
| `$env:PYTHONUTF8 = "1"` | Make Python use UTF-8 in PowerShell | [Ch. 4](04-environment-setup.md) |

Note: in PowerShell, `curl` is not the real curl; write `curl.exe` (see [Chapter 13](13-troubleshooting.md)).

## B. Config file templates

### B1. Claude Code settings.json

It lives at `~/.claude/settings.json` (on Windows, `C:\Users\<your-username>\.claude\settings.json`). It is strict JSON: no comments, and a single wrong comma causes an error, so back it up before you edit it. The template below combines every field that appears in the main text. Delete whole lines you don't need, and don't leave a stray comma behind.

```json
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  },
  "model": "sonnet",
  "permissions": {
    "defaultMode": "acceptEdits"
  },
  "autoCompactWindow": 600000,
  "subagentPromptCacheTtl": "1h"
}
```

Keep `DISABLE_AUTOUPDATER` only if you want to turn off automatic updates ([Chapter 2](02-install-claude-code.md)). If you want `defaultMode` to be `auto` or `bypassPermissions`, it must go in this user-level file; it has no effect in a project file ([Chapter 5](05-first-steps.md)). Don't lower `autoCompactWindow` casually ([Chapter 7](07-saving-usage.md)).

### B2. Starter CLAUDE.md

It lives at `~/.claude/CLAUDE.md` and applies to all projects. Rules for a single project go in the `CLAUDE.md` at that project's root ([Chapter 6](06-memory-and-rules.md)).

```markdown
# How I work

## Communication
- Reply in English. I'm a beginner, so explain each technical term in one sentence the first time it appears.
- Give the conclusion first, then the process. If the process is long, write it to a file and put only the key points in the reply, because I read on my phone and long paragraphs are hard to get through.

## Editing files
- Read an existing file before changing it. Editing without reading tends to hit the wrong place.
- Before changing a config-type file (settings, environment variables, startup scripts), make a backup so it can be restored if something breaks.
- Change only what I asked for. I can't check every "improvement" made on the side.

## Long tasks
- For a task expected to take more than a few minutes, tell me in two or three sentences how you plan to do it before starting, and wait until I say "go".
- When done, tell me: which files changed, how you verified it, and what is left unfinished.

## When you're not sure
- If you're not sure a command or flag exists, check the official docs or verify by actually running it. Don't write it from memory. A wrong command costs me more time than "I'm not sure".

## Progress
- Write progress and next steps in `handoff.md`, not in this file. Read it before starting work.
```

### B3. Codex config.toml

It lives at `~/.codex/config.toml` (on Windows, `C:\Users\<your-username>\.codex\config.toml`). You can also put a `.codex/config.toml` in a project, but it loads only after the project is trusted, and the one closest to the current directory takes priority.

```toml
model = "gpt-6.1-sol"
model_reasoning_effort = "medium"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false

[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

For `approval_policy`, use only `on-request` or `never`; `untrusted` may stop Codex from starting ([Chapter 3](03-install-codex.md)). To allow network access, change `network_access` to `true` ([Chapter 3](03-install-codex.md)). Which levels `model_reasoning_effort` accepts depends on your model and client ([Chapter 7](07-saving-usage.md)). The MCP section is only an example; delete it if you don't use it ([Chapter 10](10-extensions.md)).

### B4. Starter AGENTS.md

It lives at `~/.codex/AGENTS.md` or at the root of a project. Don't copy your whole CLAUDE.md over; the Codex file should contain only what Codex needs to do its work ([Chapter 6](06-memory-and-rules.md)).

```markdown
# Working instructions for Codex

This is an unattended task environment; nobody will answer questions in real time. When something is unclear, pick a reasonable approach, finish the work, and state in the final summary what you chose and why.

## How to work
- Read the relevant files before making changes, understand how things are currently written, and follow the existing style.
- Change only what the task names. If you notice problems elsewhere, note them in the summary and leave them alone.
- Verify by actually running it after you change something; if tests can run, run them. State in the summary what you ran and what the result was.

## Delivery
- Put the summary in your last message, within 15 lines: what you did, which files you changed, the verification result, and anything left unfinished.
- State facts plainly in the summary. If something was not verified, say "not verified" instead of writing it as done.

## Cautions
- Never write keys, passwords or tokens into any file.
- The `.git` and `.codex` directories in the project are read-only; don't try to change them.
```

### B5. Task file for handing work to Codex

Save it as `task.txt` in the task folder. `<absolute path of the task folder>` is the full path of that folder ([Chapter 8](08-division-of-labor.md)).

```text
You are an unattended executor. Do not enable Skills and do not ask me questions; when something is ambiguous, take the most literal reading and note in your report which reading you chose.

[Background]
<One or two sentences on what this project is and where it stands>

[Task]
<The specific thing you want done; state the input and the expected output clearly>

[Output location]
Write all files under this absolute path: <absolute path of the task folder>

[Rules]
- Change only files related to the task.
- Do not delete any existing file and do not alter conclusions.
- If you hit a conflict, get stuck, or the numbers don't add up, stop and say so in the report.

[Report requirements]
Write the report in your last message, within 15 lines: what you did, the absolute paths of the files you produced, and whether anything went wrong.
```

Windows (PowerShell):

```powershell
cd "<absolute path of the task folder>"
cmd /c 'codex exec --sandbox workspace-write --skip-git-repo-check -C "<absolute path of the task folder>" -o summary.md - < task.txt'
```

macOS / Linux (Terminal):

```bash
cd "<absolute path of the task folder>"
codex exec --sandbox workspace-write --skip-git-repo-check -C "<absolute path of the task folder>" -o summary.md - < task.txt
```

When the run finishes, read only `summary.md`; there is no need to go through the whole transcript.

### B6. Ledger template

Keep one per long-running line of work, in a location you choose. The background agent updates it each round, and whoever takes over uses it to pick up the work ([Chapter 9](09-subagents-and-background-work.md)).

```markdown
# Ledger: <line name>

Last updated: <date and time>  Updated by: <which lead agent in sequence>

## Status
One or two sentences: what stage this line is in now, and whether everything is normal overall.

## In flight
| Item | Status | How to pick it up | Expected |
|---|---|---|---|
| <Job A> | Running | Log is at <path>; watch the end of the log for <success marker> | <expected completion time> |
| <Job B> | Paused | Resume command: <command> | Wait for me to say go |

## Next steps
1. <first thing to do next>
2. <second thing to do next>

## Where outputs go
- <result file>: <path>
- <intermediate file>: <path> (deletable / not deletable)

## To report
- What to report when a result arrives: <for example, "when A finishes, report the key numbers to me">
- What to report when something goes wrong: what happened, what it affects, what you plan to do or what you need me to decide

## Rules (read before taking over)
- <one or two hard rules for this line>
- When the context reaches about 270k: write the ledger and end this round with [ROTATE]
```

## C. Official links

Prices, plans and supported regions change; the official pages below are authoritative. All were checked in October 2026.

**Accounts, plans, regions**

| Topic | URL |
|---|---|
| Claude pricing and plans | https://claude.com/pricing |
| About the Claude Pro plan | https://support.claude.com/en/articles/8325606-what-is-the-pro-plan |
| About the Claude Max plan | https://support.claude.com/en/articles/11049741-what-is-the-max-plan |
| Claude usage rules | https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work |
| Anthropic supported regions | https://www.anthropic.com/supported-countries |
| Claude Help Center: supported regions | https://support.claude.com/en/articles/8461763-where-can-i-access-claude |
| Codex pricing and plans | https://learn.chatgpt.com/docs/pricing |
| Codex sign-in and plans | https://learn.chatgpt.com/docs/auth |
| ChatGPT Pro tiers | https://help.openai.com/en/articles/9793128-about-chatgpt-pro-plans |
| ChatGPT supported regions | https://help.openai.com/en/articles/7947663-chatgpt-supported-countries |
| OpenAI API supported regions | https://developers.openai.com/api/docs/supported-countries |

**Claude Code documentation**

| Topic | URL |
|---|---|
| Installation and system requirements | https://code.claude.com/docs/en/setup |
| Overview | https://code.claude.com/docs/en/overview |
| Slash commands | https://code.claude.com/docs/en/commands |
| Interactive mode and shortcuts | https://code.claude.com/docs/en/interactive-mode |
| Permission modes | https://code.claude.com/docs/en/permission-modes |
| Settings (settings.json) | https://code.claude.com/docs/en/settings |
| Environment variables | https://code.claude.com/docs/en/env-vars |
| Enterprise networking and certificates | https://code.claude.com/docs/en/network-config |
| Sign-in and authentication | https://code.claude.com/docs/en/authentication |
| Model configuration | https://code.claude.com/docs/en/model-config |
| Costs | https://code.claude.com/docs/en/costs |
| Memory and CLAUDE.md | https://code.claude.com/docs/en/memory |
| Subagents | https://code.claude.com/docs/en/sub-agents |
| Skills | https://code.claude.com/docs/en/skills |
| MCP | https://code.claude.com/docs/en/mcp |
| Hooks | https://code.claude.com/docs/en/hooks |
| Plugins | https://code.claude.com/docs/en/discover-plugins |
| VS Code extension | https://code.claude.com/docs/en/vs-code |

**Codex documentation**

| Topic | URL |
|---|---|
| CLI installation and updates | https://learn.chatgpt.com/docs/codex/cli |
| Commands and slash commands | https://learn.chatgpt.com/docs/developer-commands?surface=cli |
| Non-interactive mode (`codex exec`) | https://learn.chatgpt.com/docs/non-interactive-mode |
| Approvals and sandbox | https://learn.chatgpt.com/docs/agent-approvals-security |
| Configuration reference | https://learn.chatgpt.com/docs/config-file/config-reference |
| Advanced configuration | https://learn.chatgpt.com/docs/config-file/config-advanced |
| Environment variables | https://learn.chatgpt.com/docs/config-file/environment-variables |
| AGENTS.md | https://learn.chatgpt.com/docs/agent-configuration/agents-md |
| MCP | https://learn.chatgpt.com/docs/extend/mcp |
| Hooks | https://learn.chatgpt.com/docs/hooks |
| Windows sandbox | https://learn.chatgpt.com/docs/windows/windows-sandbox |
| Installation and system requirements (GitHub) | https://github.com/openai/codex/blob/main/docs/install.md |

Provider sign-up and plan pages for other model providers: see References in [Chapter 11](11-other-model-providers.md).

### References

Every command and template in this appendix comes from Chapters 2 through 13, and the fields were checked against the official documentation. The official URLs are in Section C above (checked in October 2026).

---
[← Chapter 14. Worked examples: from an idea to delivery](14-worked-examples.md) · [Contents](../README.md#contents)
