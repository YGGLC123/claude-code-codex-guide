# Chapter 4. Environment setup: Git, Node.js, Python and common Windows problems

This chapter has two parts. First you install Git, Node.js and Python, which the rest of the guide relies on. Then I cover several common problems I ran into on Windows.

Some of the installation methods in Chapters 2 and 3 require these tools first. For example, installing Claude Code or Codex with npm requires Node.js. If you got stuck earlier because a tool was missing, finish this chapter and then go back and try again.


## 4.1 Install Git, Node.js and Python

These three are the basic tools used in later chapters. Download each installer from its official website and follow the installation wizard. I don't give download links here because web addresses change; search for each tool's official site and confirm that the domain is the official one.

- **Git**: On Windows, install Git for Windows, which comes with a terminal called Git Bash. Claude Code does not require Git on Windows; without it, Claude Code runs commands through PowerShell instead. Installing Git is still the better choice, because many open-source projects and tutorials assume you have it. If Git is installed but Claude Code can't find it, set `CLAUDE_CODE_GIT_BASH_PATH` in the `env` section of settings.json to the path of `bash.exe`; the official documentation has an example path.
- **Node.js**: Download the LTS (long-term support) version. If you plan to install Claude Code through npm, you need Node 22 or later (as explained in [Chapter 2](02-install-claude-code.md)).
- **Python**: Download Python 3. On the first page of the installer, check "Add python.exe to PATH".

When the installation finishes, **open a new terminal** and verify each tool:

Windows (PowerShell):

```powershell
git --version
node --version
npm --version
py --version
```

macOS / Linux (Terminal):

```bash
git --version
node --version
npm --version
python3 --version
```

> [!NOTE]
> **Checkpoint**
> Each command prints a version number. On my machine the results are `git version 2.53.0.windows.2`, `v22.19.0`, npm `10.9.3` and `Python 3.13.5`. Your version numbers may differ; what matters is that you don't get a "command not found" message.

> [!TIP]
> On Windows, start Python with `py`, not `python`. Windows ships a `python` placeholder that redirects to the Microsoft Store, and many readers get stuck there. `py` is the launcher that comes with the Python installer and is more reliable.

If you still get a message that the command is not recognized (for example `'node' is not recognized as an internal or external command, operable program or batch file.`) after installing, the most likely cause is that you didn't reopen the terminal, so PATH has not been refreshed. Close the terminal and open it again. If that doesn't help, restart the computer.

I don't cover package managers such as winget here, because I haven't verified their package names one by one. The installers from the official websites are the most reliable option.


## 4.2 Common problems on Windows

Each of the problems below is one I actually ran into on Windows while having an AI carry out tasks, and each comes with the reason. If you use macOS, you can skip most of this section and take only the idea from the last item.

### PowerShell 5.1 does not read UTF-8 without a BOM correctly

Windows 11 ships with PowerShell 5.1. When it reads a script file that is UTF-8 encoded but has no BOM (a marker at the start of the file), it interprets the file using the system's ANSI code page. As a result, any non-ASCII literal written directly in the script (Chinese characters, for example) is already garbled by the time it reaches the clipboard, and any search that uses it returns nothing.

My approach is to keep non-ASCII text out of the script. I put it in a separate .txt file and read it from the script like this:

```powershell
$text = [System.IO.File]::ReadAllText("<path-to-text-file>", [System.Text.Encoding]::UTF8)
```

Replace `<path-to-text-file>` with the full path of that .txt file. When you ask an AI to write scripts, you can give it this rule directly.

### Quote non-ASCII paths and paths with spaces, and be careful with a file extension after a variable

When a path contains non-ASCII characters (for example, folder names in Chinese) or spaces, always put it in quotes. There is also a subtler mistake: in PowerShell, `$j.tex` does not mean "the variable j followed by .tex". It means "the tex property of the variable j", and the result is an empty value.

I ran into this when compiling LaTeX in batch with a script. I wrote `pdflatex $j.tex`, and nothing was compiled. The logs and PDFs were all old, and I almost checked against the old results.

The fix is to write the file name as `"$j.tex"`, in quotes, when you build it. Also, after a run finishes, don't rely on the log alone. Check the modification time of the output to confirm it was actually updated.

### PowerShell and Git Bash use different syntax

On Windows, Claude Code may run commands in either PowerShell or Git Bash, and the two syntaxes are not interchangeable. Most of my errors came from heredocs in Git Bash (a way of passing several lines of text to a command):

The problem I hit: I had an AI use a heredoc to pass a piece of Python to the interpreter, and anything with non-ASCII text or backslashes broke. Backslashes were swallowed, `\b` became a control character, and the `\t` in `D:\t` became a real Tab. Non-ASCII text such as Chinese was decoded with the system's legacy code page (GBK on my machine), so the whole block came out garbled. Quoting the heredoc does not prevent this.

The fix is to save the Python script as a .py file first, by writing it out as a file, and then run it in UTF-8 mode. In PowerShell:

```powershell
$env:PYTHONUTF8 = "1"
py <script-name>.py
```

In Git Bash, write paths that contain non-ASCII characters literally, with forward slashes, in the form `/c/Users/...`. Don't route them through `python -c`, or the characters turn into garbage and `cd` fails.

### Specify encoding and newline when Python reads and writes files

When you ask Python to modify an existing file, specify both parameters for reading and for writing:

```python
with open("<file-path>", "r", encoding="utf-8", newline="") as f:
    content = f.read()
```

The reason: the default text mode writes `\n` as `\r\n`. I had a 1,133-line file where I only appended one passage, but the whole file switched to CRLF line endings. The commit diff showed the entire file as rewritten, and I couldn't tell what had actually changed.

### Don't use CopyFromScreen for screenshots

If you ask an AI to capture a desktop window, tell it not to use `Graphics.CopyFromScreen`. It should use `PrintWindow` instead, with `PW_RENDERFULLCONTENT` (the value is 2) in the parameters.

The reason: many applications draw their windows through GPU compositing, and `CopyFromScreen` returns a stale frame, which makes it look as if your last action had no effect. A minimized window yields only a few dozen pixels, so restore the window before capturing it.

### Turn off automatic restart for system updates on a machine that runs long tasks

If you need a computer to run tasks for a long time, turn off the automatic restart for Windows updates. My own approach is to pick a quiet time once a month and install updates manually.

The reason: twice, an automatic restart after a system update wiped out a running computation and every conversation at once. All caches were invalidated, and I had to recover from scratch.

Here is how I set it up. I used Group Policy to change updates to "download but notify me to install", and I enabled "no auto-restart with logged-on users". For the detailed Group Policy path, and for differences between Windows versions, Microsoft's official documentation is authoritative. Windows Home has no Group Policy editor. There you can pause updates or set active hours on the update page in Settings. The idea is the same: **don't let the system choose its own time to restart**.

> [!IMPORTANT]
> The cost of turning off automatic restart is that you have to remember to install updates yourself, and you can't leave security patches uninstalled for long. I remind myself once a month.

### References

- Claude Code environment variables: https://code.claude.com/docs/en/env-vars (Checked in October 2026.)
- Claude Code settings: https://code.claude.com/docs/en/settings (Checked in October 2026.)
- Claude Code setup and Windows notes: https://code.claude.com/docs/en/setup (Checked in October 2026.)

---
[← Chapter 3. Installing Codex](03-install-codex.md) · [Contents](../README.md#contents) · [Chapter 5. First steps →](05-first-steps.md)
