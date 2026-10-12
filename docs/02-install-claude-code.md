# Chapter 2. Installing Claude Code

This chapter does one thing: it gets Claude Code installed on your computer, logged in, and running for the first time. When you are done, you can type `claude` in a terminal and see it waiting for your input.

One prerequisite first. Both installation and login connect to Anthropic's servers, so your terminal needs a working internet connection. If you hit a "download failed" error in this chapter, see "When something goes wrong" at the end of the chapter.

## 2.1 Choose how you will use it

Claude Code can be used in several ways, and each is installed differently:

| Form | What it is | Separate command-line install needed? |
|---|---|---|
| Command line | Run `claude` in a terminal; the most complete feature set | It is the command line itself |
| VS Code extension | Use it inside VS Code with a friendlier interface | Already bundled in the extension |
| Desktop app | The "Code" tab in the Claude desktop app | No |
| Web | Open claude.ai/code and run in the cloud | No, nothing to install locally |
| Continue on your phone | Pick up a session that is running on your computer from your phone; the feature is called Remote Control | Needs `claude` running on your computer |

My recommendation: **beginners should install the command line first, then the VS Code extension.** The command line is the most complete form, and the later chapters assume it. The extension lets you see files and changes directly in the editor, which is friendlier for beginners. You can try the desktop and web versions once you are comfortable.

The phone option needs a word of explanation. There is no separate Claude Code phone app; cloud sessions and Remote Control both live in the "Code" tab of the Claude phone app. Your computer must stay on and the `claude` process must keep running, and the code always runs on your own computer.

> [!IMPORTANT]
> **Free accounts cannot use it**
> A free claude.ai account does not include Claude Code. You need a Pro, Max, Team, Enterprise, or Console account. For plans and prices, see [Chapter 1](01-accounts-and-plans.md). Your country or region must also be on Anthropic's official list of supported countries and regions; this is a system requirement stated in the installation documentation.

## 2.2 Installing on Windows

The officially recommended method is the native install, which takes a single command. It needs Windows 10 version 1809 or later, does not need administrator rights, and does not require any other software first.

**Step 1: Confirm which terminal you have open.** This is the step most likely to go wrong, because Windows has two command lines and the commands differ:

- If the prompt looks like `PS C:\Users\...>` (with `PS`), it is PowerShell.
- If the prompt looks like `C:\Users\...>` (without `PS`), it is CMD.

Search for "PowerShell" in the Start menu and open it.

**Step 2: Run the install command.**

Windows (PowerShell):

```powershell
irm https://claude.ai/install.ps1 | iex
```

If you are using CMD:

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

> [!WARNING]
> **Command pasted into the wrong terminal**
> The problem I have run into: pasting the CMD command into PowerShell gives `The token '&&' is not a valid statement separator`, and pasting the PowerShell command into CMD gives `'irm' is not recognized`.
> The fix: the official documentation lists both errors, and both mean "wrong terminal." Check whether the prompt has `PS` in it and use the matching command.

**Step 3: The installer shows no progress bar while it downloads. This does not mean it has stopped responding.** Wait for it to finish. When the installation completes, **close the current terminal and open a new one** before moving on to the next step.

**Step 4: Verify.**

Windows (PowerShell):

```powershell
claude --version
```

> [!NOTE]
> **Checkpoint**
> You should see a version number in the form `2.1.xxx (Claude Code)`. When I wrote this chapter, the latest official version was 2.1.296 and mine was 2.1.293. Your number being slightly higher or lower is normal.

### Do you need Git for Windows?

It is not required, but I recommend it. The difference: with Git for Windows installed, Claude Code uses Git Bash to run commands; without it, Claude Code uses PowerShell instead. Both work. Installing Git is covered in [Chapter 4](04-environment-setup.md). If Git is installed but Claude Code cannot find it, set `CLAUDE_CODE_GIT_BASH_PATH` in the `env` section of `settings.json`. The value is the path to Git Bash, for example `C:\\Program Files\\Git\\bin\\bash.exe` (backslashes must be doubled in JSON).

### Installing with WinGet

If you prefer WinGet, you can use it:

Windows (PowerShell):

```powershell
winget install Anthropic.ClaudeCode
```

Note that versions installed through WinGet do not update automatically. You need to run `winget upgrade Anthropic.ClaudeCode` yourself from time to time. For that reason I prefer the native install above, which updates itself in the background.

### Native Windows or WSL

On Windows you can use the native environment directly, or install WSL 2 and work inside it. The main difference is that WSL 2 supports sandboxing and native Windows does not. Beginners can simply use the native environment; there is no need to set up WSL.

## 2.3 Installing on macOS / Linux

macOS / Linux (Terminal):

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

When it finishes, open a new terminal and run `claude --version` to verify.

If you are on macOS and prefer Homebrew:

macOS (Terminal):

```bash
brew install --cask claude-code
```

This installs the stable channel, which is usually about a week behind the latest release and skips versions with serious problems. To get the latest version, use the `claude-code@latest` cask instead. Installations through Homebrew do not update automatically either; run `brew upgrade claude-code` yourself (or `brew upgrade claude-code@latest` if you use the latest cask).

System requirements: macOS 13.0 or later, Ubuntu 20.04 or later, Debian 10 or later, and at least 4 GB of RAM. Linux also has officially signed apt, dnf, and apk package repositories; check the official docs for how to use them.

## 2.4 Installing with npm

The npm method still works, but the official docs now place it under "advanced installation options" and no longer recommend it. Unless you have a specific reason, such as a company environment that only allows npm, use the native install above.

Note that this method needs Node.js 22 or later. Older tutorials online that say "Node 18 or later" are out of date, and following them will fail.

The command is the same in Windows (PowerShell) and macOS / Linux (Terminal):

```bash
npm install -g @anthropic-ai/claude-code
```

To upgrade, use the command below. Do not use `npm update -g`, and do not put `sudo` in front of it:

```bash
npm install -g @anthropic-ai/claude-code@latest
```

For how to install Node.js, see [Chapter 4](04-environment-setup.md).

## 2.5 First launch and login

**Step 1: Go into a project folder.** Claude Code treats the folder you are currently in as its working scope, so do not start it from the root of your system drive or from a system directory. Create an empty practice folder first:

Windows (PowerShell):

```powershell
mkdir $HOME\claude-practice
cd $HOME\claude-practice
claude
```

macOS / Linux (Terminal):

```bash
mkdir ~/claude-practice && cd ~/claude-practice
claude
```

**Step 2: Log in through the browser.** The first time you run `claude`, it walks you through login. A browser page opens automatically; log in with the account that holds your subscription and authorize access.

- If the browser does not open automatically: press `c` in the terminal to copy the login link, then paste it into a browser yourself.
- The page ends by showing a login code. Paste it back into the terminal.

> [!NOTE]
> **Checkpoint**
> After a successful login, the terminal shows a prompt box waiting for your input. Type "Hello." If it replies, installation, login, and the network connection are all working. Type `/exit` to quit.

**Use official channels only for your account.** Buy your subscription on Anthropic's official website or inside the official app. I do not recommend any other source, and this guide does not cover them.

### The other way to log in: an API key (pay-as-you-go)

Besides a subscription account, there is another route: log in with a Claude Console account, or set the environment variable `ANTHROPIC_API_KEY`, and you are billed by the number of tokens you use. The first time you log in with Console, a workspace named "Claude Code" is created for you automatically.

Be clear about how the two billing methods differ:

- Subscription: a fixed monthly fee, but usage has limits per 5-hour window and per week.
- API key: billed per token, so you pay for what you actually use. Anthropic's enterprise deployment data puts the average at about US$13 per person per active day; for your own usage, the official figures are authoritative.

> [!WARNING]
> **Switching to pay-as-you-go without realizing it**
> The problem I have run into: I had set `ANTHROPIC_API_KEY` on my computer, thought I was using my subscription, and was actually spending API money.
> The fix: when this variable is set, interactive mode asks you once at startup whether to use it, and only uses API billing if you agree. Non-interactive runs such as `claude -p`, however, always use the key if one is present. To go back to your subscription, clear the variable (`unset ANTHROPIC_API_KEY` on macOS / Linux; on Windows, delete it from the system environment variables), reopen the terminal, and type `/status` inside a session to confirm the account.

## 2.6 Verifying and updating

Three commands, each with a different job:

Windows (PowerShell), or macOS / Linux (Terminal):

```bash
claude --version
claude doctor
claude update
```

- `claude --version`: shows the version number.
- `claude doctor`: runs a health check. It does not start a session. It reads your setup without changing anything and reports whether the installation is healthy and whether your settings files contain mistakes, along with suggested fixes. It also shows the result of the most recent automatic update.
- `claude update`: updates to the latest version immediately.

The native install updates itself in the background, and the new version takes effect the next time you start. Inside a session there is also `/doctor`, which diagnoses problems with the installation, settings, extensions, and CLAUDE.md and, after you confirm, helps fix them. It is a different thing from the command-line `claude doctor`.

**To turn off automatic updates**: add an `env` block to `~/.claude/settings.json` (on Windows, this file is at `C:\Users\<your-username>\.claude\settings.json`):

```json
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  }
}
```

After you turn it off, you can still update manually with `claude update`. This file must be strict JSON: a `//` comment or a trailing comma causes a Settings Error.

> [!TIP]
> In the official docs, many features are marked "requires v2.1.xxx or later." The easy way to handle this is to run `claude update` once right after installing, so you are on the latest version.

## 2.7 The VS Code extension and the desktop app

### The VS Code extension

Requirement: VS Code 1.94.0 or later.

1. Open VS Code and click the Extensions icon on the left.
2. Search for "Claude Code" and make sure the publisher is Anthropic. The extension ID is `anthropic.claude-code`.
3. Click Install.
4. Press `Ctrl+Shift+P` to open the Command Palette, type "Claude Code", and choose **Open in New Tab**.

You can also install it from the command line, the same on Windows and macOS:

```bash
code --install-extension anthropic.claude-code
```

VS Code derivatives such as Cursor can install it too; it is in their extension marketplaces as well.

**To upgrade the extension**, this is what I usually do:

```bash
code --install-extension anthropic.claude-code --force
```

After upgrading, run Reload Window in VS Code (search for that name in the Command Palette) for the upgrade to take effect.

> [!IMPORTANT]
> **Upgrading reloads every VS Code window**
> Upgrading the extension makes all VS Code windows reload. If conversations are running in several windows, they will all be interrupted. Upgrade at a time when no conversation is running.

### The desktop app

Switch to the "Code" tab in the Claude desktop app and you can use it. It comes with Claude Code built in, so you do not need to install the command line separately. It is available for macOS (Intel and Apple Silicon) and Windows (x64, ARM64) and needs a paid subscription. In a command-line session, type `/desktop` to move the current session to the desktop app.

JetBrains IDEs (IntelliJ IDEA, PyCharm, WebStorm, and so on) also have an official plugin, but the plugin does not include the CLI, so you must first install `claude` as described in this chapter.

## 2.8 Two lessons from experience

### If you cannot find a new model, check your version first

I once got this wrong. An AI asserted that "there is no model called Sonnet 5.5," and I believed it. Later I found that the Claude Code extension on my machine was too old and only recognized Sonnet 5; after upgrading to a newer version, 5.5 appeared. So my rule is: **if a new model you want is not in the picker, first check the version installed on your machine, then check what the latest official version is. Do not trust an AI's impression that the model "does not exist."** An AI's knowledge has a cutoff date, and it does not know whether the software you installed supports a new model.

One fact you can verify: the official docs state that Opus 5.5 needs v2.1.280 or later, and Sonnet 5.5 needs v2.1.284 or later. If your version is too old, run `claude update` first.

If the model is still missing from the picker after upgrading, you may need to add it to the list of selectable models in settings.json yourself. The exact key names and syntax are in the official docs.

Also, when you tell a subagent to use a particular model, do not go by what you wrote. Check the transcript to see which model actually ran. On older versions, you may have written a new model name while an older model ran.

### Headless mode: using Claude as a background worker

`claude -p` means "non-interactive": you give it one prompt, it does the work, prints the result, and exits on its own, without opening a conversation interface. You can combine it with `--model` to choose a model:

Windows (PowerShell) or macOS / Linux (Terminal):

```bash
claude -p --model sonnet "Summarize in three sentences what is in the current folder"
```

This lets scripts call it and have it work in the background. `--model` accepts an alias (`opus`, `sonnet`, `fable`) or a full model name.

One more point: any patches you installed yourself and any settings you changed should be **checked one by one after every upgrade to confirm they still work.** I once patched my editor to add Chinese dictation; an upgrade overwrote the patch, and I did not notice for a long time.

## 2.9 When something goes wrong

**The install script fails to download, spins forever, or reports a connection error**

Most likely your network cannot reach the official servers. First confirm that your browser can open the official website, then run the install command again. If it still fails, see [Chapter 13](13-troubleshooting.md).

**After installing, you get "command not found" for claude (not recognized / command not found)**

After installation, a new terminal window is needed before the system recognizes the new program. Close the current terminal and **open a new one**, then try again. If it still fails, the install directory is not on your PATH. The native install location is `C:\Users\<your-username>\.local\bin\claude.exe` on Windows and `~/.local/bin/claude` on macOS / Linux. First run this in PowerShell:

```powershell
where.exe claude
```

If there is no output, it really was not found; run the install command again. If there is output but typing `claude` is still not recognized, add that directory to the system PATH environment variable and open a new terminal.

**PowerShell says "running scripts is disabled" (execution policy)**

The install command hands the script content directly to PowerShell, so it is usually not affected by the execution policy. If you meet this error in other situations, do not loosen the policy across the board. For how to change it, follow Microsoft's official documentation.

**Uninstalling**

To uninstall a native install, on Windows (PowerShell):

```powershell
Remove-Item -Path "$env:USERPROFILE\.local\bin\claude.exe" -Force
Remove-Item -Path "$env:USERPROFILE\.local\share\claude" -Recurse -Force
```

macOS / Linux (Terminal):

```bash
rm -f ~/.local/bin/claude
rm -rf ~/.local/share/claude
```

Your configuration lives in `~/.claude` and `~/.claude.json`, and the commands above do not touch them.

### References

- Claude Code setup and updates: https://code.claude.com/docs/en/setup (Checked in October 2026.)
- Claude Code overview and ways to use it: https://code.claude.com/docs/en/overview (Checked in October 2026.)
- VS Code extension: https://code.claude.com/docs/en/vs-code (Checked in October 2026.)
- JetBrains plugin: https://code.claude.com/docs/en/jetbrains (Checked in October 2026.)
- Remote Control: https://code.claude.com/docs/en/remote-control (Checked in October 2026.)
- Mobile: https://code.claude.com/docs/en/mobile (Checked in October 2026.)
- Login and authentication: https://code.claude.com/docs/en/authentication (Checked in October 2026.)
- Model configuration and version requirements: https://code.claude.com/docs/en/model-config (Checked in October 2026.)
- Slash commands: https://code.claude.com/docs/en/commands (Checked in October 2026.)
- Costs: https://code.claude.com/docs/en/costs (Checked in October 2026.)

---
[← Chapter 1. Getting ready: accounts and subscriptions](01-accounts-and-plans.md) · [Contents](../README.md#contents) · [Chapter 3. Installing Codex →](03-install-codex.md)
