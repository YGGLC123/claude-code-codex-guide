# Chapter 3. Installing Codex

Codex is a coding assistant from OpenAI. It is the same kind of tool as Claude Code in [Chapter 2](02-install-claude-code.md): it installs on your computer, can read and write files in the folders you point it at, and can run commands. This is how I divide the work: Claude handles project coordination and judgment, and Codex writes the code, so I install both.

This chapter covers installing and logging in to Codex, running a first command, and a short introduction to its sandbox and approvals. More detailed usage is in [Chapter 5](05-first-steps.md).

> [!IMPORTANT]
> The official documentation has moved. The old `developers.openai.com/codex` now redirects to `learn.chatgpt.com/docs`. Many older tutorials online contain links, commands, and settings that are out of date. What follows is based on the official docs as I checked them in October 2026; when installing, follow the latest official instructions.

## 3.1 Ways to use Codex

The common entry points are:

| Form | What it is | Who it suits |
|---|---|---|
| CLI (command line) | Run `codex` in a terminal | People who want every feature and are comfortable typing commands |
| IDE extension | VS Code, Cursor, and Windsurf share one extension, with ID `openai.chatgpt` | People who like working in an editor |
| ChatGPT desktop app | Formerly called the Codex desktop app; the official name is now "ChatGPT desktop app," and you switch to Codex inside it | Beginners who prefer not to use the command line |
| Cloud (Codex Cloud) | The web page `chatgpt.com/codex`; tasks run in the cloud and continue even while your computer sleeps | People who want to hand tasks off to the cloud |
| Phone | Use the ChatGPT phone app to run cloud tasks, or to remotely control a connected computer (approve commands, view changes) | People who want to check progress while away |
| Xcode, JetBrains | Integrations built by those vendors; choose Codex in their AI chat, with no OpenAI extension installed | People who use those two editors |

My recommendation is the same as for Claude Code: **beginners should install the command line first, then the VS Code extension.** This chapter mainly covers the command line, and later chapters use it for examples too.

A few things to know about the phone option in advance: the connection must be started from "Settings > Connections" in the desktop app, and neither the CLI nor an IDE can start it. Whether Android is supported is described inconsistently across the official pages, so check the official docs.

## 3.2 Installing on Windows

Windows now has official native support, so you do not need to install WSL2. I use it in the native Windows environment, running commands through PowerShell.

> [!IMPORTANT]
> The official docs contradict each other in two places. The documentation says the CLI, the IDE extension, and the desktop app all run natively on Windows. But the system requirements table in `docs/install.md` on GitHub still says Windows 11 requires WSL2, which may be an old table that was not updated. Treat the official docs as authoritative on this point. My approach is to use the native environment directly and consider WSL2 only if I run into sandbox-related problems.

### Method 1: the PowerShell install script (the official first choice)

First **open a new** PowerShell window and run:

Windows (PowerShell):

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

By default the script installs to `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`. When the installation finishes, **close that window and open a new one** so the system recognizes the new command.

### Method 2: npm

If you already have Node.js installed (see [Chapter 4](04-environment-setup.md) for how to install it), you can use npm:

Windows (PowerShell):

```powershell
npm install -g @openai/codex
```

The `package.json` of the npm package states a Node requirement of version 16 or later. My machine has Node 22, and there are no problems. My own Codex was installed through npm, and it ends up at `C:\Users\<your-username>\AppData\Roaming\npm\codex.cmd`.

> [!TIP]
> Pick one of the two methods and do not mix them. Otherwise you end up with two copies of Codex and cannot tell which one an upgrade is upgrading. After installing, run `where.exe codex` to see the path; only one should appear.

> [!NOTE]
> **Checkpoint**
> Open a new PowerShell and run `codex --version`. You should see a version number like `codex-cli 0.159.2` (your number will differ, because versions change quickly).

> [!WARNING]
> **"codex" is not found**
> The problem I have run into: right after installing, I typed `codex` in the same window and got a command-not-found error.
> The fix: close the window and open a new one so PATH refreshes. If that does not help, run `where.exe codex` to find where it was installed, then check whether that directory is on your PATH.

> [!WARNING]
> **The install script fails to download**
> If the download hangs or reports a connection error, most likely your network cannot reach the official servers. First confirm that your browser can open the official website and try again; if it still fails, see [Chapter 13](13-troubleshooting.md).

## 3.3 Installing on macOS / Linux

macOS / Linux (Terminal):

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

If you use Homebrew, note that it is a cask, not an ordinary formula:

macOS (Terminal):

```bash
brew install --cask codex
```

The requirements are macOS 12 or later and Ubuntu 20.04 or later; these numbers come from the system requirements table in the official GitHub repository. Memory is 4 GB at minimum, with 8 GB recommended. This table differs from what the official docs say elsewhere, so check the official docs.

## 3.4 Logging in

Codex has two ways to log in:

- **ChatGPT account**: uses your subscription's usage allowance. Most people should choose this.
- **API key**: billed separately by usage on the OpenAI platform. It does not draw on your subscription allowance and has no cloud features.

### Logging in with a ChatGPT account

Run this in the terminal:

Windows (PowerShell) or macOS / Linux (Terminal):

```text
codex login
```

It opens a browser and asks you to authorize with your ChatGPT account. After you authorize, return to the terminal and you are logged in.

> [!NOTE]
> **Checkpoint**
> Run `codex login status`; it should show that you are logged in. The login information is cached in `~/.codex/auth.json` (on Windows, `C:\Users\<your-username>\.codex\auth.json`) or in the system credential store. **This file is plain text. Do not send it to anyone or upload it anywhere.** The CLI and the IDE extension share this one login.

If you are on a machine without a browser (a remote server, for example), the officially most recommended method is device-code login: `codex login --device-auth`. This feature is still in beta, and you first need to turn it on in ChatGPT's security settings.

### Logging in with an API key

Windows (PowerShell):

```powershell
$env:OPENAI_API_KEY | codex login --with-api-key
```

macOS / Linux (Terminal):

```bash
printenv OPENAI_API_KEY | codex login --with-api-key
```

Before running this, you need to put your API key into an environment variable named `OPENAI_API_KEY`. You generate the API key in the OpenAI developer platform dashboard. It is the equivalent of your wallet password, so do not paste it into a chat, a screenshot, or any public place. To log out, use `codex logout`.

### Which plans include Codex

According to the official pricing page and help center, Free, Go, Plus, Pro, Business, Edu, and Enterprise all include Codex. Monthly prices for individual plans (in US dollars): Free 0, Go 8, Plus 20, and Pro in three tiers at 100, 200, and 500.

Two more details to note:

- **Codex Cloud**: offered only on Plus, every Pro tier, Business, Enterprise, and Edu. Free and Go do not have it.
- **Whether Free and Go can use the command line**: the official pages are not clear on this, so check the official docs. If you plan to use the command line long term, my suggestion is to buy at least Plus.

On usage, Plus is estimated per 5-hour window. The number of messages varies a great deal between models, local and cloud usage share the same allowance, and there may also be a weekly limit. These are estimates, not fixed caps. To see how much allowance remains, type `/status` in the CLI or open `https://chatgpt.com/codex/settings/usage`. A full comparison of plans is in [Chapter 1](01-accounts-and-plans.md).

## 3.5 Verifying and updating

Once installed, run three commands first:

In Windows (PowerShell) or macOS / Linux (Terminal):

```text
codex --version
codex doctor
codex update
```

- `codex --version`: shows the version number.
- `codex doctor`: generates a local diagnostic report covering the installation, configuration, authentication, runtime environment, Git, and terminal. Whenever you run into a problem later, run this first.
- `codex update`: checks for and installs an upgrade. You can also rerun the install command, or use the npm or brew update commands (for npm, rerun `npm install -g @openai/codex`; for brew, `brew upgrade --cask codex`).

> [!WARNING]
> **A new model in your config, but the CLI is too old**
> The problem I have run into: I switched to a new model in my configuration, but the CLI on my machine was too old. It failed after 7 seconds with a message such as "requires a newer version" or "not supported when using Codex with a ChatGPT account".
> The fix: run `codex --version` to see the version, then run `codex update` to upgrade. In my case, the model gpt-6-astra needed CLI 0.153 or later.

> [!IMPORTANT]
> **The extension and the desktop app need separate updates**
> The VS Code Codex extension and the desktop app each ship with their own copy of the codex program. Upgrading the command line with npm does not update them. So after an upgrade you can end up with a new command line and an old version inside the extension, and each has to be updated separately.

## 3.6 First run

First create an empty folder. Do not try it in a folder that holds important files:

Windows (PowerShell):

```powershell
mkdir "$env:USERPROFILE\codex-test"
cd "$env:USERPROFILE\codex-test"
codex
```

The first time you enter, it asks whether you trust this directory. Choose to trust it, then type a greeting, for example "Hello, please introduce yourself." If you get a reply, installation, login, and the network connection are all working. To exit, type `/quit` (or `/exit`).

### Smoke test: codex exec

`codex exec` is the non-interactive mode: you give it one sentence, it does the work and exits, which makes it suitable for scripts. After every installation I run this one first:

Windows (PowerShell) or macOS / Linux (Terminal):

```text
codex exec --skip-git-repo-check "Reply with exactly: OK"
```

`--skip-git-repo-check` lets it run in a folder that is not a Git repository; by default it requires you to be inside a Git repository.

> [!NOTE]
> **Checkpoint**
> The last thing on the terminal should be `OK`. Progress messages are printed on screen, and only the final answer is the last line. Seeing OK means Codex can log in, reach the network, and receive the model's answer, so the whole chain works.

> [!TIP]
> When a prompt is too long to put on the command line, put it in a file and pass it in with `cat prompt.txt | codex exec -` (the trailing `-` means read from standard input). There is no official `--prompt-file` option, so do not try to use one. On Windows, do not pipe prompts containing non-ASCII text (Chinese, for example) through PowerShell; they easily turn into garbled characters. For the correct way to do it, see [Chapter 8](08-division-of-labor.md).

## 3.7 Sandbox and approvals: the basics

Codex has two layers of safety control. [Chapter 5](05-first-steps.md) covers them in detail; here is the basic picture:

- **The sandbox mode** governs "what is technically possible," meaning where it can write and whether it can use the network. There are three:
  - `read-only`: can read, cannot change anything.
  - `workspace-write`: can write to the current workspace.
  - `danger-full-access`: no restrictions; recommended only in an isolated environment.
- **The approval policy** governs "when it asks you first." Only two common settings remain now: `on-request` (asks you when needed) and `never` (never asks).

In a Git-managed directory, the official default recommendation is workspace-write with on-request. By default there is **no network access**, and it can only write to the current workspace.

> [!IMPORTANT]
> **Some older advice that has changed**
> - `approval_policy = "untrusted"` is no longer supported; putting it in the config may prevent Codex from starting.
> - `--full-auto` is deprecated; the official docs ask that new scripts use `--sandbox workspace-write` instead.
> - To adjust permissions during a session, use `/permissions`. There is no `/approvals`.
> - `--dangerously-bypass-approvals-and-sandbox` (alias `--yolo`) means no sandbox and no approvals, and the official docs do not recommend it. Beginners should not use it.

### Network access has to be turned on separately

The sandbox has no network access by default. I have tested this: writing to the working directory, running Python, and compiling LaTeX all work fine inside the sandbox; only network access has to be enabled separately. To enable it for a single run, add one option:

Windows (PowerShell) or macOS / Linux (Terminal):

```text
codex exec --sandbox workspace-write -c sandbox_workspace_write.network_access=true "<what you want it to do>"
```

To let it search the web, use the top-level option `--search`:

```text
codex --search
```

To turn it on permanently, put this in `~/.codex/config.toml`:

```toml
[sandbox_workspace_write]
network_access = true
```

### Open only the smallest writable directory

When you need it to write to one more directory, you can use `--add-dir <path>`. But in my experience: **give it only the directory it actually needs to write to, and do not hand over your whole user directory.** Once, for convenience, I granted it my whole user directory, and the sandbox stopped responding and both tasks failed. The official docs also recommend preferring `--add-dir` with a narrow scope over using `danger-full-access` directly.

Also note that even under workspace-write, `.git`, `.agents`, and `.codex` inside a writable directory stay read-only.

## Chapter summary

By now you should have done all of these: Codex is installed, `codex --version` prints a version, `codex login status` shows you are logged in, and the `codex exec` smoke test printed `OK`. The next chapter ([Chapter 4](04-environment-setup.md)) covers installing basic tools such as Git, Node.js, and Python, and common problems on Windows. If any step above cannot reach the official servers, first confirm that your browser can open the official website and try again; if it still fails, see [Chapter 13](13-troubleshooting.md).

### References

- Codex CLI setup and updates: https://learn.chatgpt.com/docs/codex/cli (Checked in October 2026.)
- Codex forms, desktop app, and IDE: https://learn.chatgpt.com/docs/app ; https://learn.chatgpt.com/docs/codex/ide (Checked in October 2026.)
- Mobile and remote connections: https://learn.chatgpt.com/docs/mobile ; https://learn.chatgpt.com/docs/remote-connections (Checked in October 2026.)
- Native Windows support and the sandbox: https://learn.chatgpt.com/docs/windows/windows-sandbox ; https://learn.chatgpt.com/docs/windows/windows-app (Checked in October 2026.)
- System requirements table: https://github.com/openai/codex/blob/main/docs/install.md (Checked in October 2026.)
- Login and credentials: https://learn.chatgpt.com/docs/auth (Checked in October 2026.)
- Plans and usage: https://learn.chatgpt.com/docs/pricing ; https://help.openai.com/en/articles/11369540-codex-in-chatgpt (Checked in October 2026.)
- Approvals and sandbox: https://learn.chatgpt.com/docs/agent-approvals-security (Checked in October 2026.)
- Non-interactive mode: https://learn.chatgpt.com/docs/non-interactive-mode (Checked in October 2026.)
- Command-line command list: https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)

---
[← Chapter 2. Installing Claude Code](02-install-claude-code.md) · [Contents](../README.md#contents) · [Chapter 4. Environment setup: Git, Node.js, Python and common Windows problems →](04-environment-setup.md)
