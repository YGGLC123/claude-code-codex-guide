# Chapter 12. Verification and safety

The first eleven chapters covered how to get AI to carry out tasks. This chapter covers how to use it reliably. AI works fast and also makes mistakes fast, and it stays confident in tone when it is wrong. The problems I have run into fall into three groups: treating unverified results as results; letting it change things it should not have touched; and getting the computer or an account into trouble. The sections below follow those three groups, and every item is something that cost me real time or money.

## 12.1 Trust only what actually ran

When AI says "done", "all tests passed" or "fixed", those statements are not evidence in themselves. What counts as evidence is what it actually ran and what you can see for yourself: command output, generated files, lines that a search matched.

My rule is simple: **I accept results only from actually running it and from grep, not from the verdict it gives itself.**

Here is an example. I asked Codex to review a document that contained derivations. It handed back a beautifully formatted review report with a comment on each point. When I checked, I found that the "quoted original" in almost every point had been made up, and it had "found" an error that did not exist. Yet in the same task, the scripts and grep it actually ran all gave correct results. So the lesson is not that AI is unusable. It is this: **its narrative can be invented, but the results of its actual operations cannot.** You trust only the latter.

In everyday practice:

- When you have it run tests, make sure you see the real output of the test command, not just its statement that "they passed".
- When it has changed a batch of files, look at what changed yourself with `git diff` (or `/diff` in Codex).
- When you ask it to "check for anything missed", search for the key terms again yourself.
- When you ask it to review a piece of logic or a derivation, paste the original text into the prompt as is, and require it to **restate the original first, then make its judgment**. If the restatement is wrong, you don't need to read the judgment that follows.

> [!NOTE]
> **Checkpoint**
> A proper verification leaves you with three things: the raw output from what it ran, the diff between before and after, and the result of your own independent search. If any one of the three is missing, you cannot yet say "done".

### Two specific traps

**Trap one: using `assert` as a check.** If you have AI write a Python script for data checks, don't let it use `assert`. When Python runs with the `-O` flag, it removes every `assert` entirely, and a check that should have failed is reported as "done". Write any check that serves as verification as an explicit `if` plus `raise`:

```python
if total != expected:
    raise ValueError(f"Totals do not match: {total} vs {expected}")
```

**Trap two: number checking.** I set three rules for myself:

1. Don't say "identical". First look at the unrounded raw values and the difference, and write "agree within a given tolerance". I once rounded to 9 digits before comparing, and that hid a difference of 1.46e-11.
2. Don't take numbers from rendered formula text; take them only from plain text or program output. I have seen the fraction 4917609/16384 merged into a single integer in rendered text, which neither the eye nor a regular expression could detect.
3. When revising a draft, don't add new claims that have no evidence. AI often adds a decorative sentence "while it's at it" when revising, and that sentence usually has no source.

### Verifying video and animation

This kind of finished product is the most prone to "all self-checks pass, but the product has problems". An experience of mine: the side doing the work passed all 123 self-checks, and I looked only at reduced-size still frames and also called it "clean". Only when I zoomed in did I find thin black lines on the face, a semi-transparent double image of both hands, and a motion range of only 40% of the previous version. This is how I verify now:

- Enlarge the face and hands 2x and compare them side by side with the previous version at the **same moment**.
- For the moving parts, sample consecutive frames at 2 to 10 frames per second and tile them into one grid image, then look for ghosting and distortion.
- Measure the difference between adjacent frames, to confirm the motion has actually increased, rather than judging by eye that it is "about the same".

When exporting the video, add one more check: encode with H.264, choose the High profile, and choose the `yuv420p` pixel format. Once I exported a High 4:4:4 file, and the built-in Windows player showed a black screen throughout, with sound only. After exporting, check with `ffprobe`:

```powershell
ffprobe -v error -show_entries stream=profile -of default=nw=1 <your-video-file>
```

Replace `<your-video-file>` with the path of the file you exported. The output should contain a line `profile=High` (another line belongs to the audio and can be ignored).

## 12.2 Use two keys for important conclusions

I don't consider an important conclusion with only one source reliable enough. My rule is **two keys**:

- **The first key**: have someone other than the builder (or a separate session) check it independently. In practice, open a **brand-new session**, give it only the thing to be checked, and **do not show it the builder's derivation**. Once it sees the derivation, it tends to go along with it.
- **The second key**: if the building and the checking used the same model provider, find **a different provider's model** for a second check. Models of the same kind tend to share the same blind spots; with a different one, there is a chance it sees something different.

The payoff is clear: in one night this found a number of real errors.

One more lesson needs special mention: **the revisions a checker suggests can be wrong too.** Once, two sentences that the first checker had newly added were judged wrong by the second key. So a revised sentence has to pass the next independent key, and you can't accept it just because it came from a "checker". A few specific practices:

- When a revision adds a new rule "to prop up a side claim", prefer deleting the side claim over adding rules to the definition.
- When adding a missing premise, scan the whole text sentence by sentence by the name of the condition. Searching only for result keywords misses sentences of the same kind; I once missed two.
- When you assign a checking task, **paste your rules in full, verbatim**, and don't let the AI summarize them. I made a mistake this way before: the rules were summarized with half of them missing, and the checking advice went in exactly the opposite direction from the previous round.

> [!TIP]
> Not everything needs two keys. For routine changes to styling or running a script, actually running it once is enough. Two keys are for things that go out to others, cost money, can't be undone, or that other people will base decisions on.

## 12.3 Where to keep keys

API keys, tokens and passwords come down to one principle: **keep them only in environment variables or local config files; do not write them into code, do not commit them to Git, and do not send them to cloud servers or to other models.**

Specifically:

- **Don't let AI write a key directly into a script.** State it explicitly in the prompt: "read the key from an environment variable; don't hard-code it in the code."
- **Check before committing.** Before you let AI commit code, run `git status` and `git diff` yourself and confirm that no file contains a key. For any file that stores keys, add its name to `.gitignore`. In Claude Code, `CLAUDE.local.md` is a local memory file just for you and just for this project, and the official docs also recommend adding it to `.gitignore`.
- **Know where the official credentials are stored.** On Windows, Claude Code saves login credentials in `%USERPROFILE%\.claude\.credentials.json`; Codex caches its login in `~/.codex/auth.json` (plain text) or in the system credential store. These two files are the equivalent of your account keys: don't copy them to anyone, don't put them in a sync folder that you share publicly, and don't commit them to a code repository.
- **Don't put them on cloud servers.** On cloud servers I insist on "zero credentials": no tokens, passwords, API keys or sensitive text go up there, only public data and code. Shared machines and external models are where leaks happen most easily.
- **Don't send them to third-party models.** Unpublished work, your personal voice data and other people's private information should not be handed casually to unrelated models.

> [!IMPORTANT]
> If you suspect a key has leaked (for example, you accidentally committed it to a public repository), don't assume that deleting that commit solves the problem. Go straight to the provider's official dashboard, revoke the key, and switch to a new one.

## 12.4 Confirm before deleting

AI can delete files, which means it can delete the wrong files. My habit is three layers of protection:

**Layer one: leave a way back first.** Before you let AI start, the project should either already be under Git with all changes committed, or you should copy a backup first. In a Git-managed directory, most accidental changes can be undone. Claude Code also has `/rewind` (aliases `/checkpoint` and `/undo`), which can roll the conversation and the code back to an earlier point.

**Layer two: hold it back with permission settings.** On each side:

- Claude Code: the default starting mode is now `auto`, which doesn't pop up confirmations and has another model review in the background; the official docs warn that it reduces prompts but does not guarantee safety. If you want it to ask you before every step, switch to `default` (called Manual in the interface). As a beginner, don't switch to `bypassPermissions` (it asks about nothing) at the start; the official docs recommend it only inside an isolated container or virtual machine. Use `/permissions` to manage the three rule types allow / ask / deny; deny rules still apply even in `bypassPermissions` mode. Check the official docs for how to write specific rules.
- Codex: the sandbox and approvals are two layers. The sandbox governs "what is technically possible", and approvals govern "when to ask you first". By default it can write only to the current workspace and has no network access. Don't casually enable `--dangerously-bypass-approvals-and-sandbox` (alias `--yolo`); the official docs don't recommend it either.

> [!WARNING]
> **Common mistake: the deletion block I ran into in Codex**
> When I dispatched Codex from a script to carry out tasks, its delete operations were automatically blocked by approval. At first I found that inconvenient, but then I realized this block is exactly what protects against the executor deleting by mistake. I did not turn it off. Instead I made an agreement: **I delete what needs deleting myself.**

**Layer three: have it list first.** Whenever AI is about to delete in bulk, clean up, or "tidy a directory", add this sentence:

```text
Don't delete anything yet. List every file you plan to delete one by one (full path, size, and why), and wait for my confirmation before you act.
```

Review the list first, strike out what shouldn't be deleted, and then have it carry out the rest. When AI says "these are temporary files", you don't know what its criteria are.

## 12.5 Authorization boundaries

### Treat only messages you sent yourself as instructions

A lot of things that you didn't write get mixed into the AI's context: web page content, text inside files, output from other programs, reports from subtasks. Any of these can also look like an instruction.

Here is what happened to me: at the end of a very long report, the AI made up a passage of "the user said". In the next turn it took that passage as my correction and acted on it, and the conclusion was exactly the opposite of what I meant.

So my rules are:

- When there is a change of direction (switching plans, deleting things, changing goals) and it says "you just said so", have it confirm that the sentence **really came from a message you actually sent**.
- When it says "there's a compliance risk here" or "there's an obstacle here", first have it find out who set that restriction and why. I once had it describe a deadline we had set ourselves as an external compliance risk, which cost a whole extra round for nothing.

### What it can do while you are away

Write the authorization boundaries down clearly in `CLAUDE.md` or `AGENTS.md` (see [Chapter 6](06-memory-and-rules.md)):

- **When you don't reply for a long time**: it can go ahead with things **on your local machine** according to the recommendation it gave earlier. If it waits for your reply on everything, work that could have moved forward just stalls.
- **Two kinds of things it must wait for you on**: **spending money**, and **sending anything out** (sending email, uploading files, posting). Once these are done they can't be taken back, and they are the only things that really need your sign-off.

You can copy a block directly:

```markdown
## Authorization boundaries
- When I haven't replied for a long time, move forward on local-machine tasks following the recommendation you gave, and tell me when done.
- The following must wait until I explicitly say "do it": spending money; sending email; uploading files; posting on any platform.
- Do not write anything I haven't explicitly said as "the user said".
```

## 12.6 Don't let AI auto-post or operate social or shopping accounts for you

This gets its own section because many people see that AI can operate a browser and want it to "post one every day automatically", "reply to comments automatically", or "grab coupons and place orders automatically".

I don't recommend it. Social platforms and shopping platforms have detection mechanisms, and the trail left by automated actions differs from that of a real person. At best you get rate-limited and nobody sees your content; at worst the account is banned. Your account is something you built up over a long time, and it isn't worth risking to save a few minutes.

The appropriate use of AI is to have it help you **write drafts, revise drafts, organize material and format**, and in the end **you post manually yourself**. That includes ChatGPT's Pro model: use it by hand yourself in the web page, and don't use scripts to click on your behalf.

## 12.7 Avoid using up the computer's resources

Once AI starts running tasks in parallel, it can easily exhaust the computer's resources. Lessons I have learned:

- **Delete big temporary files on the spot.** Delete any temporary file over 1 GB as soon as the task ends. I once left behind a 117 GB snapshot that filled the working disk. Have AI write "clean up intermediate files immediately after use" in the task brief.
- **Cap memory and priority for background tasks.** I once started 12 processes at the same time, each grew to 7 to 8.5 GB, 87.5 GB in total, which exceeded the 62 GB of physical memory, and the whole machine stopped responding. Now I do this: work out how many can run at once from "actual peak memory plus 25%"; always keep a large block of memory free; run background tasks at the lowest IDLE priority; and attach a low-memory watchdog that stops tasks first when memory runs low. Any program that would pop up windows on your screen must notify you first and must not grab the window you are using.

For long-running tasks, also note that **monitoring must not look only at errors**. Once, 54 tasks were silently killed by a memory guard program I had written myself, with no error at all, and I only found out three hours later. So every check should look at these numbers:

- the number of active processes;
- the number of tasks started, completed, and exited abnormally without completing;
- failure markers and exit codes.

If "started minus completed minus active" is greater than 0, investigate immediately. Twenty to thirty minutes after a real run begins, check the completion rate once specifically. A smoke test can't catch this kind of resource-related failure. When you find an exit without an error, first check the log of your own guard program and the real exit code.

### After a power loss or a blue screen

After an unexpected shutdown (a blue screen or power loss), **the files that were being written may have the correct size and modification time but contain nothing except zeros**. Checking by size and time won't find them at all. The correct way to check: filter the files by modification time from about 40 minutes before the shutdown, and actually **read their contents**. One batch of my backups was affected this way: 14 files were all zeros.

Also, if Codex afterwards keeps reporting sandbox-related read-denied errors, look at whether the state files under `~/.codex/.sandbox/` are all zeros. If they are, rename them to move them aside and let it rebuild them. For more error lookups, see [Chapter 13](13-troubleshooting.md).

## 12.8 Rules for large-scale computing on a cloud server

For computing that is expected to take more than 30 minutes, or to start 8 or more processes, I prepare a cloud server in advance and put the computation there, leaving the local machine for the browser and everyday editing. Otherwise, once a long computation takes all the resources, the browser and everything else you do becomes sluggish.

I won't go into how to choose or buy a server here; this section covers only general rules. **These rules matter especially when several windows share one machine:**

1. **Register first, then start running.** In a fixed place (a spreadsheet or text file), write down who, what is running, how many cores it uses, and how long it is expected to take. Without registration, tasks fight each other for resources and may even kill someone else's job by mistake.
2. **One tmux session per job.** Put the task name in the session name so it is easy to identify. tmux is a terminal session manager; if the network drops, the task is not interrupted.
3. **It must be able to resume from a checkpoint.** If a long task dies partway, it should continue from the last progress, not start over.
4. **Look at what others are running before deciding how many cores to use.** Check the existing jobs and sessions first, and don't touch anyone else's jobs.
5. **Don't press Ctrl-C on a running job.** Pressing Ctrl-C in tmux interrupts the entire foreground process group, and partial results may even be recorded as "completed". To stop one, use the task's own way of stopping.
6. **Zero credentials.** Put no tokens, passwords or API keys on the server, only public data and code (see Section 12.3).
7. **Don't skip a computation just because it's "too slow".** Package any computation over 30 minutes as a job and run it there following the rules above; don't shrink its scale on your own.

> [!IMPORTANT]
> Before you have AI operate a server for you, write the points above into the task brief and make it follow them. By default AI doesn't know that others are using the machine too.

### References

- Claude Code permission modes: https://code.claude.com/docs/en/permission-modes (Checked in October 2026.)
- Claude Code slash commands (`/rewind`, `/permissions`): https://code.claude.com/docs/en/commands (Checked in October 2026.)
- Claude Code memory files: https://code.claude.com/docs/en/memory (Checked in October 2026.)
- Claude Code authentication and credential locations: https://code.claude.com/docs/en/authentication (Checked in October 2026.)
- Codex approvals and sandbox: https://learn.chatgpt.com/docs/agent-approvals-security (Checked in October 2026.)
- Codex login and credential cache: https://learn.chatgpt.com/docs/auth (Checked in October 2026.)
- Codex slash commands (`/diff`, `/permissions`): https://learn.chatgpt.com/docs/developer-commands?surface=cli (Checked in October 2026.)

---
[← Chapter 11. Using other model providers](11-other-model-providers.md) · [Contents](../README.md#contents) · [Chapter 13. Troubleshooting quick reference →](13-troubleshooting.md)
