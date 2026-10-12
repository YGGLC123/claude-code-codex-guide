# Chapter 13. Troubleshooting quick reference

This chapter is for lookup; you do not need to read it from the start. When you hit an error, find the matching group, then work through the "What to do" column step by step.

One convention first: English text in quotation marks in the tables is either the original wording from the official documentation or wording I have seen on screen myself. Entries without quotation marks, described as "roughly says...", are my paraphrases. The wording on your screen may differ, so do not fixate on the exact text.

## 13.1 Work out which kind of problem it is

| What you see | Which group to read first |
|---|---|
| Install fails, download stalls, cannot connect | 13.2, 13.3 |
| Command not found after you type it | 13.3 |
| Cannot log in, constant 401, sudden pay-as-you-go billing | 13.4 |
| Garbled text, or a script did not run but reported no error | 13.5 |
| Codex cannot resume, model at capacity | 13.6 |
| Tools all failing, cannot stop a background task | 13.7 |
| Errors when using other model providers | 13.8 |
| Just restarted after a power cut or blue screen | 13.9 |

## 13.2 Network and region

| Symptom | Most likely cause | What to do |
|---|---|---|
| The command line cannot reach the official servers | The network is down, or the official service is having a temporary outage | First confirm that your browser can open the official site, then retry; use the command below to check whether the API endpoint is reachable |
| Login or use fails with a message that your region is not supported (roughly: your location is outside the service area) | Your region is not on the official supported list | See [Chapter 1](01-accounts-and-plans.md). OpenAI states explicitly that using the service from outside the listed regions may lead to a ban or suspension, so do not try to force it |
| A certificate error on a company network (roughly: the certificate is not trusted) | The company network replaces certificates | For Claude Code, install the root certificate into the system certificate store, or set `NODE_EXTRA_CA_CERTS`; for Codex, set `CODEX_CA_CERTIFICATE` |

First check for yourself whether the official API endpoint is actually reachable. Windows (PowerShell):

```powershell
curl.exe -o NUL -w "%{http_code}" https://api.anthropic.com/v1/messages
```

macOS / Linux (Terminal):

```bash
curl -o /dev/null -w "%{http_code}" https://api.anthropic.com/v1/messages
```

A 405 response is normal. It means the network is working and the request simply carried no key. Likewise, a 401 from `https://api.openai.com/v1/models` also means the connection works.

> [!WARNING]
> **The curl in PowerShell is not the real curl**
> The problem I ran into: typing `curl` in PowerShell actually runs an alias of `Invoke-WebRequest`, and the arguments do not match at all.
> The fix: always write `curl.exe`.

## 13.3 Installation and "command not found"

| Symptom | Most likely cause | What to do |
|---|---|---|
| The install script fails to download | Cannot reach the official servers | Check the network as in 13.2, then reinstall |
| After installing, it says `claude` or `codex` is not found (roughly: not a recognized command) | The terminal has not refreshed PATH | Close the terminal and open a new one. The official docs say to open a new terminal after installation |
| Still not found in a new terminal | The install directory is not on PATH | Use `where.exe claude` to see where it is installed. The native Claude Code install goes to `C:\Users\<your-username>\.local\bin` |
| You see `The token '&&' is not a valid statement separator` | You typed a CMD command into PowerShell | Use the PowerShell install command instead |
| You see `'irm' is not recognized` | You typed a PowerShell command into CMD | Use the CMD install command instead |
| The script is blocked from running (roughly: the execution policy does not allow it) | PowerShell execution policy | The official Codex command already includes `-ExecutionPolicy ByPass`, so use it as written; for Claude, see [Chapter 2](02-install-claude-code.md) |
| Claude Code says it cannot find Git Bash | Git for Windows is not installed | It works without Git for Windows and falls back to the PowerShell tool; if you want Git Bash, install it, and if it still cannot be found, set `CLAUDE_CODE_GIT_BASH_PATH` in `env` |
| After an npm install it will not run, or reports a wrong version | Node version is too old | Installing Claude Code with npm requires Node 22 or later. Do not use `sudo npm install -g` |
| The version is old and new models are missing | Not updated | Check the version first, then run `claude update` or `codex update` |

> [!TIP]
> When you cannot tell what is wrong, run the health-check commands first: `claude doctor` and `codex doctor`. `claude doctor` is read-only and does not start a session; it lists validation errors in your settings files along with suggested fixes.

## 13.4 Login and billing

| Symptom | Most likely cause | What to do |
|---|---|---|
| The browser does not open automatically | The terminal could not launch a browser | In Claude Code, press `c` to copy the login link and paste it into a browser manually; if the browser gives you a login code, paste it back into the terminal |
| After logging in to Claude, it says you cannot use Claude Code | You are on a free account | The free plan does not include Claude Code; you need Pro or higher, see [Chapter 1](01-accounts-and-plans.md) |
| The interactive UI asks whether to use an API key | A leftover `ANTHROPIC_API_KEY` in the environment | Once you choose it, you are billed through the API instead of your subscription. To return to the subscription, clear the variable and confirm with `/status` |
| Your subscription is active, but you keep being charged by the API | Same cause: in non-interactive `-p` mode, a key is always used whenever one exists | Clear the variable |
| It says your usage is used up | The 5-hour window or the weekly limit has been reached | Wait for the reset, or turn on usage credits to keep going at pay-as-you-go rates, see [Chapter 7](07-saving-usage.md) |
| `settings.json` reports a Settings Error | This file is strict JSON | Do not write `//` comments, and do not leave trailing commas |
| Codex login hangs on the browser | There is no browser on the machine | `codex login --device-auth` (beta; you must first enable it in the ChatGPT security settings), or log in on a machine with a browser and copy `~/.codex/auth.json` over |
| Codex says cloud features are unavailable | You logged in with an API key | Codex Cloud works only with a ChatGPT account login |
| A Free or Go account cannot use the command line | The official docs do not say | Check the official docs |

> [!TIP]
> If you are not sure which account is currently logged in, type `codex login status` in Codex, or `/status` in a Claude Code session.

## 13.5 Windows encoding and paths

This group is the hardest to spot: some of these problems produce no error at all, only wrong results.

| Symptom | Most likely cause | What to do |
|---|---|---|
| Non-ASCII text (for example Chinese) in a script turns into garbage | PowerShell 5.1 reads a UTF-8 script without a BOM as ANSI | Put the non-ASCII text in a `.txt` file and read it with `[System.IO.File]::ReadAllText($f,[System.Text.Encoding]::UTF8)` |
| It finishes with no error, but the output is stale | `$j.tex` was not quoted, so it was treated as a property lookup and evaluated to null | Always write file names as `"$j.tex"`, and check the output's modification time |
| Backslashes are dropped, `\b` becomes a control character, and `D:\t` becomes a Tab | Python was passed through a heredoc in the Bash tool | Write the `.py` file to disk first, then run it, and set `PYTHONUTF8=1` |
| After editing a file, the whole file is flagged as modified | Default text mode writes line endings as `\r\n` | Read and write in Python with `encoding="utf-8", newline=""` |
| Changing into a path with non-ASCII characters fails in Git Bash | The path gets mangled when another program passes it along | Use forward-slash paths of the form `/c/Users/...` directly |

This has caught me out. After I appended a section to a 1,133-line file, the whole file became CRLF, and the commit diff showed the entire file as rewritten. On another occasion pdflatex did not compile one of the files, so the log and the PDF were both stale, and I nearly checked against the old result. So my rule is: after a run, check the modification time of the output, and do not rely on the log alone.

## 13.6 Codex-specific problems

| Symptom | Most likely cause | What to do |
|---|---|---|
| Resuming fails with "Not inside a trusted directory" | The current directory is not the original working directory | First `cd` back to the original directory, then run `codex exec resume <session-id> --skip-git-repo-check` |
| "Selected model is at capacity", then it exits immediately | The selected model was at capacity at that moment | Resume by session ID in the same way. Note that the exit code is 0 and there is no summary, so do not treat it as a success |
| 401, although you are already logged in | The Codex sandbox does not have your GitHub login | Run queries that need a login outside the sandbox |
| `codex exec` will not run in a non-Git directory | By default it requires a Git repository | Add `--skip-git-repo-check` |
| Codex will not start after you set `approval_policy = "untrusted"` in the config | That value is no longer supported | Use only the officially documented values such as `on-request` and `never`, see [Chapter 3](03-install-codex.md) |

To find the session ID you need for resuming, open the picker with `codex resume`, or use `codex exec resume --last` to continue the most recent session in the current directory.

To check whether your GitHub login has been carried into the sandbox, use `gh api user`. `gh api rate_limit` works even anonymously, so it proves nothing. This has caught me out: one run of 120 queries all returned 401, and Codex also left a background process of its own that kept slowly resuming the job. So when dispatching a job, write "do not leave any background processes" in the instructions.

## 13.7 Sessions and tools

| Symptom | Most likely cause | What to do |
|---|---|---|
| Every command in the Bash tool fails with "must be a string without null bytes" | Null bytes got mixed into the session startup script | Use the PowerShell tool for the current session; the Bash tool recovers in the next session |
| The file on disk is fixed but the tool still fails | The session cached the corrupted content at startup | Start a new session |
| Codex is still running after you stopped the background task | On Windows, the child process tree did not stop with it | Find the process by the working directory in its command line, then run `taskkill /PID <pid> /T /F` |

> [!IMPORTANT]
> Do not kill processes by name. Killing by name will also kill Codex in your other sessions.

For how to dispatch and monitor background tasks, see [Chapter 9](09-subagents-and-background-work.md).

## 13.8 Using other model providers

| Symptom | Most likely cause | What to do |
|---|---|---|
| Constant 401 (roughly: invalid key) | The endpoint address does not match the key type | The plan, coding, and pay-as-you-go tiers each use a different endpoint path, and their keys cannot be mixed |
| It keeps going for a long time after the error before stopping | Claude Code retries a 401 several times by default | I lowered the retries from the default of 10 to 2; for the default count and the exact setting name, check the official docs, see [Chapter 11](11-other-model-providers.md) |
| "ModelNotOpen", meaning the model has not been enabled | A pay-as-you-go model has not been enabled in the console | Enable it in the console first |
| 403 "AccessDenied" | The key's permissions do not include this model | Confirm that the model is ticked in the key's permissions |
| Remote Control stops working after you point at a non-official address | `ANTHROPIC_BASE_URL` points to a non-official host | This is expected behavior, as stated in the official docs |

What I do: configure only one plan endpoint and key at a time. Before configuring, first get the simplest possible request to succeed, and only then connect the tool. For pay-as-you-go models, set a budget cap in the script. The cap for my first trial was CNY 20 (about US$3), and the actual spend was CNY 5.44 (under US$1).

## 13.9 After a power cut or blue screen

| Symptom | Most likely cause | What to do |
|---|---|---|
| File size and timestamp look right, but the content is all zeros | Files being written at shutdown were zero-filled | Filter by modification time from about 40 minutes before the shutdown, and open each file to check its content |
| Every `codex exec` fails with a sandbox error that includes "apply deny-read ACLs" | The Codex sandbox state file was zero-filled | Look at the state files under `~/.codex/.sandbox/` (this is where I found them on Windows; go by your own machine), rename them out of the way, and let Codex rebuild them automatically |
| A whole batch of sessions all look freshly started | The machine restarted | Check the system log for Kernel-Power 41: a BugcheckCode of 0 looks like a power loss, and a non-zero value means a blue screen |

This has caught me out: checking only file size and timestamp could not reveal the problem, and one backup batch contained 14 all-zero files. So after a restart, spot-check the contents of important files, and do not rely on the directory listing alone.

## 13.10 General troubleshooting order

Whatever the error, I work through the steps below in this order. Please do not skip any:

1. **Run the health check first.** Run `claude doctor` or `codex doctor` and see whether it reports a clear error.
2. **Then check the network.** Use the command in 13.2 to confirm that the official API endpoint is reachable.
3. **Then check the version.** Run `claude --version` and `codex --version`, and update if the version is out of date.
4. **Finally, check the configuration.** First check `settings.json` or `config.toml` for mistakes, then check for leftover variables such as `ANTHROPIC_API_KEY`.
5. **If it is still unresolved**, try again in a brand-new terminal and a brand-new session, and send the original error text together with the steps you have already taken to an AI to ask about it.

> [!NOTE]
> **Checkpoint**
> After the first three steps, you should be able to state three things clearly: whether the health check reported a problem, whether the official API endpoint is reachable, and whether the version is current. If you cannot, do not go on to change the configuration.

### References

- https://code.claude.com/docs/en/setup (Checked in October 2026.)
- https://code.claude.com/docs/en/network-config (Checked in October 2026.)
- https://code.claude.com/docs/en/authentication (Checked in October 2026.)
- https://code.claude.com/docs/en/env-vars (Checked in October 2026.)
- https://code.claude.com/docs/en/settings (Checked in October 2026.)
- https://learn.chatgpt.com/docs/auth (Checked in October 2026.)
- https://learn.chatgpt.com/docs/non-interactive-mode (Checked in October 2026.)
- https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)
- https://learn.chatgpt.com/docs/config-file/environment-variables (Checked in October 2026.)

---
[← Chapter 12. Verification and safety](12-verification-and-safety.md) · [Contents](../README.md#contents) · [Chapter 14. Worked examples: from an idea to delivery →](14-worked-examples.md)
