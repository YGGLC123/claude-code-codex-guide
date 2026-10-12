# Chapter 7. Saving usage: context, caching and compaction

A subscription is billed monthly, but usage is not unlimited. After several months with Claude, my biggest takeaway is this: **what actually consumes your usage is not the number of questions you ask, but the length of the conversation that every question carries along with it.** This chapter explains how usage is calculated, why longer conversations cost more, how to use caching and compaction, and which habits to build for everyday work.

After reading this chapter, you will know how to check your remaining usage, when to start a new conversation, why "coming back after a while" is sometimes especially expensive, how to set up automatic compaction, and which model and reasoning effort to choose for Claude and for Codex.

## 7.1 How usage is calculated

Start with Claude. This is what the official documentation says:

- Every plan has a usage limit that resets on a **rolling 5-hour window**.
- Paid plans also have a **weekly limit**, which resets once a week at a fixed time assigned to your account.
- Claude on the web, desktop and mobile, and Claude Code, **share one usage pool**. For example, a long conversation on your phone reduces the usage available to Claude Code on your computer.
- Consumption does not correspond to a fixed number of messages. It depends on the length and complexity of the conversation, the model used, the features used, and the reasoning effort. The Max plan also has a separate weekly limit for the Fable model.

In Claude Code, enter `/usage` to see this session's cost, your plan usage bars, and activity statistics. `/cost` is now just an alias for `/usage`; either one works.

> [!NOTE]
> **Checkpoint**
> Enter `/usage` in Claude Code and press Enter. You should see a progress bar for plan usage and the cost of this session. If you don't, update to the latest version first: `claude update`.

When you exceed the limit, you can turn on "usage credits" and keep going, paying as you use; enter `/usage-credits` inside a session to open the settings. You can also wait for the window to reset. Check the official pricing page for prices.

> [!IMPORTANT]
> As soon as you start consuming usage credits, the cache lifetime gets shorter (see 7.3). So going over the limit costs more than the extra charge itself: the same operations also become more expensive.

Now Codex. When you sign in to Codex with a ChatGPT account, it uses your subscription usage, shared between local and cloud. There may also be a weekly limit, and the number of messages per 5 hours that the official docs give is only an estimate, not a fixed cap. In the Codex command line, use `/status` to see remaining usage and the token usage of the current session, and `/usage` to see account token usage; on the web you can check `https://chatgpt.com/codex/settings/usage`.

> [!WARNING]
> **The "remaining usage" in the ChatGPT sidebar**
> The problem I ran into: the ChatGPT web sidebar shows "X% usage remaining." For a while I assumed it was the usage for the chat models on the web, and on that basis I wrongly concluded that I had to wait for usage to refresh before I could continue.
> The fix: from what I observed, that number corresponds to Codex usage and has nothing to do with the model used for web chats. This is my own observation, and the interface is subject to what the official docs say; to check Codex usage, use `/status` above or the usage web page.

## 7.2 Why longer conversations cost more

The model itself has no "memory." Every time you send a message, Claude Code has to send the **entire conversation** to the model again, including the earlier questions, the answers, the files it has read, and the command output, plus the current sentence. So:

- Message 1 carries a thousand words; message 100 may carry several hundred thousand.
- The same "OK, continue," sent in a long conversation, costs far more than in a new one.
- Large files you read in, long logs you paste in, and screenshots you take: as long as they stay in the conversation, **every** later message has to carry them.

The context window is the maximum amount of content that fits in the conversation. According to the official figures, Fable 5.1, Opus 5.5, Sonnet 5.5 and Haiku 5.5 all have 1M tokens (about 555,000 English words) and a maximum output of 128K. A big window doesn't mean you should fill it. The fuller the window, the more each step costs, and the longer the conversation, the easier it is for details to be missed.

In a session, enter `/context` to see, as a colored grid, how much of the context is in use; it also gives optimization suggestions. I like to check it midway through a long task.

To start over completely, use `/clear`. It opens a new conversation with an empty context, costs nothing, and you can still find the old conversation with `/resume`.

> [!TIP]
> A simple rule of thumb: when the current matter is finished and the next one is unrelated, use `/clear` to start a new conversation instead of continuing in the same session indefinitely.

## 7.3 Caching: how long you're away makes a big difference

### What it is

Because every message resends the whole conversation, Anthropic built "prompt caching": the part at the start of the conversation that hasn't changed is billed at a much lower price if it is sent again within a certain time. According to the official figures, reading from the cache generally costs 10% of the base input price, 5% for Opus 5.5 and Sonnet 5.5, and 2.5% for Fable 5.1.

But the cache has a **lifetime**. Once it expires, it is gone, and the next message has to write the whole conversation into the cache again (which costs more). The official rules:

- At the API level, the default is 5 minutes, renewed on every hit; you can also choose 1 hour, where writing to the cache costs 2 times the base input price (the 5-minute tier is 1.25 times).
- In Claude Code, for subscribers using usage within their plan, **the main conversation uses a 1-hour cache**; as soon as you start consuming usage credits, the main conversation drops to 5 minutes; with an API key or on a cloud platform, the default is 5 minutes.
- **Subagents default to 5 minutes.**
- You can choose between `5m` and `1h` with the `promptCacheTtl` setting or the environment variable `CLAUDE_CODE_PROMPT_CACHE_TTL` (requires v2.1.242 or later).

The first message you send after being away longer than the cache lifetime recomputes the whole conversation, and the cost rises noticeably.

### What I measured

I went through 40,571 requests, and every cache write was on the 1-hour tier. I also tested the borderline case: with two requests 58 to 60 minutes apart, the cache still hit; at 62 minutes or more, the whole thing was rewritten. For my large sessions, each rewrite is 500,000 to 700,000 tokens, which is no small expense.

So I set a few rules:

1. **When you need to come back and check on something at intervals, keep the interval under 1 hour.** For example, while waiting for a long task to produce a result, I check at least every 50 minutes, rather than every 70.
2. **When you are waiting for a result, poll with a script**, and wake the AI only when the result appears or something goes wrong (see [Chapter 9](09-subagents-and-background-work.md)). Having the AI wake itself every few minutes to look costs usage and is unnecessary.
3. **Don't slip into overage billing without knowing it.** After you go over, the main conversation's cache drops to 5 minutes, and every short break can trigger a rewrite.

### Keep-warm messages

Some sessions I keep open for a long time, with a large context (several hundred thousand tokens) that holds all the background of a project. One cold start costs about as much as forty "keep-warm" messages. What I do: **about 55 minutes after the last request, I send a very short message**, for example one that starts with "[Keep-warm]", and I agree on it in advance in the rules file: on receiving a message that starts with "[Keep-warm]", reply with a single word, "Got it," and call no tools. This renews the cache at very low cost. When nobody has used the session for about a day, I stop the keep-warm messages.

I do this because I learned the lesson: over a few days, there were more than a hundred full rewrites caused by "coming back after more than an hour."

> [!IMPORTANT]
> Keep-warm messages suit only large sessions that you really need to use repeatedly over a long period. Small and one-off sessions don't need them; let the cache expire naturally and start a new one next time, which saves more.

### Don't send non-urgent messages to a session whose cache has expired

A session whose last request was more than 1 hour ago has already "gone cold." Any message you send to it now makes it rewrite the whole context.

> [!WARNING]
> **One broadcast, three sessions with expired caches**
> The problem I ran into: I sent a notification to several sessions at once. Three of them had gone cold, and each rewrote nearly 500,000 tokens, about US$11.6 in total, all for a notification that wasn't urgent.
> The fix: don't proactively send non-urgent things to cold sessions. Write them into a rules file that the session reads when it wakes up, and let it pick them up the next time it is used.

### Caching for subagents

Subagents (covered in [Chapter 9](09-subagents-and-background-work.md)) have only a 5-minute cache by default. If a subagent has to wait for an external result, such as a script that is still running, the cache expires during a long wait, and after it wakes up the whole context has to be rewritten. I looked at three days of data, and about two thirds of the subagents' cache writes were this kind of rewrite.

The official docs describe the `promptCacheTtl` setting and the environment variable `CLAUDE_CODE_PROMPT_CACHE_TTL`, with the value `5m` or `1h`. In my own `~/.claude/settings.json` (on Windows, `C:\Users\<your-username>\.claude\settings.json`), I also set a key, `subagentPromptCacheTtl`:

```json
{
  "subagentPromptCacheTtl": "1h"
}
```

I could find this key name in the Claude Code 2.1.292 program on my machine, so it does exist, but the official docs don't mention it. So **check the official docs**: try the official `promptCacheTtl` first, and see whether `claude doctor` reports an error in your settings. I estimate that changing to 1 hour saves about US$100 a day; this is my own estimate, and the number will differ with how you use it.

> [!TIP]
> The write price for the 1-hour tier is about 1.6 times that of the 5-minute tier (2 times versus 1.25 times). It pays off only when you often come back after more than 5 minutes; for short tasks with rapid back-and-forth, the default 5 minutes is actually cheaper.

## 7.4 Compaction: what to do when the conversation gets too long

### Two kinds of compaction

When a conversation gets too long, you can have Claude compress the earlier history into a summary to free up space:

- **Manual:** In a session, enter `/compact`, optionally followed by a note on what to keep, for example `/compact keep the current task's goal and the list of files already changed`.
- **Automatic:** When the context approaches the "auto-compact window," old history is summarized automatically. The official docs say that for native 1M models, compaction happens by default at about 967K tokens.

After compaction, some details are lost; a summary is lossy. So my principle is: if you can avoid compaction, start a new conversation, and when you do need to compact, pick the timing.

### How to set the auto-compact window

There are three ways to write it; pick any one.

In `settings.json` (`~/.claude/settings.json`):

```json
{
  "autoCompactWindow": 600000
}
```

In a session:

```text
/autocompact 600k
```

With an environment variable (highest priority, and it **accepts plain integers only**; written as `600k` it is read as 600 and then raised to the lower bound). Windows (PowerShell):

```powershell
$env:CLAUDE_CODE_AUTO_COMPACT_WINDOW = "600000"
```

macOS / Linux (Terminal):

```bash
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=600000
```

The official range is 100,000 to 1,000,000. A few related switches: `DISABLE_AUTO_COMPACT=1` turns off automatic compaction but keeps `/compact`; `DISABLE_COMPACT=1` turns off `/compact` as well; `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` sets the percentage of the window at which compaction triggers, and it can only be lowered, not raised.

What I measured: the actual trigger point is roughly the value you set minus 25,000 to 33,000, which is space reserved for the summary. You can also set it per folder.

### Don't shrink the window just to save money

Many people see that "the longer the context, the more it costs" and set the compaction window very small, for example 100,000. I don't recommend this. I do long research tasks, where long context matters a lot for the quality of the results, so I set the window between 600,000 and 750,000. With a smaller window, compaction happens more often, every compaction loses details, and you end up re-explaining things and re-reading files, which costs more than it saves.

How to choose: for short tasks such as writing code or fixing small problems, the window size makes little difference because you will soon `/clear` anyway; for long tasks that read large amounts of material in a row, the window can be larger, but don't let one session grow without limit (see 7.5).

### Compact at the end of a phase

What I do now: **instead of waiting for automatic compaction, I compact on purpose when a phase comes to a natural end.**

1. When a phase is basically done and nothing is in flight (no running tasks), stop first.
2. Have Claude write a compaction document covering: what was done, what was decided, the current state, the next steps, and where the relevant files are.
3. After I review it and agree, I enter `/compact`.
4. After compaction, the first thing is to have it read that compaction document.

I do it this way because the AI cannot read how large its own context is and cannot trigger `/compact` by itself, so the timing has to be your call; and the compaction document is a second layer of insurance, so that if the summary misses something key, the document still has it on record.

For extra protection, you can use a PreCompact hook (with the matcher set to `auto`; check the official docs for the exact syntax) to refresh a handoff file before automatic compaction triggers (for hooks, see [Chapter 10](10-extensions.md)).

> [!TIP]
> It is safest to compact when nothing is in flight. If you compact while background tasks are running, the summary can easily lose the state of those tasks.

## 7.5 Habits that reduce unnecessary cost

Behind each of the habits below is an experience of mine where I incurred unnecessary cost.

### Locate first when reading files, then read the relevant part

Have Claude use search to find the relevant location first, and then read only those few dozen lines, instead of reading the whole file. Once a file of thousands of lines is read in whole, every later message has to carry it.

Logs, tables, and long answers from other models should all be **written to a file first**, so that you look only at a summary or the last few lines. By my estimate, a 20,000-character answer read straight into the context takes about 6k tokens; a log read in may take more.

The same goes for images: look at it once, note the key points in words, and work from the words afterward without rereading the old image. I have a lesson here: rereading an old image once triggered an "image could not be processed" error and wasted tokens too.

### One subagent per task, closed when done

Don't let a subagent grow longer and longer. When a task is done, wrap it up, and open a new one for the next task.

I counted the spending of two sessions: 88% and 80% respectively went to "long-lived subagents," each of which carried about 370,000 tokens of context on average at every step. That number changed how I work; for the specific changes, see [Chapter 9](09-subagents-and-background-work.md).

### Leave routine checks to scripts and wake the AI only on exceptions

For example, "check every 20 minutes whether the task is still running" can be done with the system's scheduled tasks and a small script, which wakes the AI to handle things only when something goes wrong. There is no need to set an alarm on the hour, and don't have the AI send "everything is normal" reports.

The reason: every time a background subagent finishes a round, it notifies the main session, and the main session has hundreds of thousands of tokens of context, so each notification makes it reread all of that. An "everything is normal" notification costs no less than a real problem.

## 7.6 Choosing a model and reasoning effort

### On the Claude side

The current latest models are Claude Fable 5.1 (for hard reasoning and long-horizon agent tasks), Opus 5.5, Sonnet 5.5 and Haiku 5.5, all with a 1M-token context. The official recommendation is to start most work with Opus 5.5. The default model for subscribers is also Opus 5.5 (requires v2.1.280 or later; in earlier versions the Pro default is Sonnet).

To switch models in Claude Code:

```text
/model
```

With no argument, it opens a picker; with an argument, for example `/model sonnet`, it switches directly and saves that as the default for new sessions. In the picker, pressing `s` applies only to the current session. You can also use `claude --model opus` at startup. The aliases include `fable`, `opus`, `sonnet`, `haiku`, and `opusplan` (opus when planning, sonnet when executing).

**Reasoning effort (effort)** has the levels `low`, `medium`, `high`, `xhigh` and `max`, plus `auto` to restore the model's default. In Claude Code, the default for Opus 5.5, Sonnet 5.5 and Haiku 5.5 is `medium`. To set it:

```text
/effort medium
```

Use `/effort status` to see the current level. `max` applies only to the current session. Opus 5.5, Sonnet 5.5 and Haiku 5.5 cannot turn thinking off; they decide how much to think according to effort.

> [!IMPORTANT]
> In the official API documentation, the default effort for Sonnet 5.5 is `high`, while the Claude Code documentation says `medium`. The two sources differ because they describe different things: the API and Claude Code. If you use Claude Code, go by the latter, and you can always check the level actually in effect with `/effort status`.

### On the Codex side

Codex currently recommends: GPT-6.1 Sol (`gpt-6.1-sol`) first for complex coding and agent tasks; GPT-6 Luna (`gpt-6-luna`) for well-defined, repetitive tasks; and GPT-6 Astra (`gpt-6-astra`) as the most capable. The official docs don't give a fixed default model; the actual default varies by plan and client, so check the official docs.

**GPT-5.5 will be retired from ChatGPT and Codex (ChatGPT sign-in) on 2026-10-14**; the API is not affected. If a script or config has GPT-5.5 hard-coded, replace it promptly.

To switch: in the interactive interface, use `/model` to choose the model and reasoning effort; non-interactively:

```bash
codex exec -m gpt-6.1-sol "your task"
```

In the config file, use `model_reasoning_effort`, which can be set to `low`, `medium`, `high`, `xhigh`, `max` or `ultra`; which levels are available depends on the model and client. The official suggested starting points: High for Luna, Light for Astra; Max gives a single task more thinking time.

One more thing to note: Codex's speed tiers deduct extra usage. The Fast tier deducts subscription usage at 2.5 times, and the Ultrafast tier at 8 times. I don't recommend turning them on when you're in no hurry.

### My suggested combination

- **Everyday conversation and execution:** Use Claude's default model (Opus 5.5), keep effort at the default `medium`, and raise it temporarily to `high` or higher only for problems that are truly hard.
- **Writing prose:** Switch to Sonnet 5.5, which is usually enough and also saves more.
- **Writing and changing code:** Hand it to Codex, with a recommended model. That leaves Claude's usage for conversation and judgment (see [Chapter 8](08-division-of-labor.md) for the division of labor).
- **Higher reasoning effort is not always better:** The higher the level, the longer it thinks and the more it consumes. My habit is to start from the default level, raise it only when the answer is clearly not deep enough, and set it back afterward.

### References

- Claude Code documentation: https://code.claude.com/docs/en/costs , https://code.claude.com/docs/en/model-config , https://code.claude.com/docs/en/prompt-caching , https://code.claude.com/docs/en/commands , https://code.claude.com/docs/en/env-vars (Checked in October 2026.)
- Claude model overview and prompt caching (API documentation): https://platform.claude.com/docs/en/about-claude/models/overview , https://platform.claude.com/docs/en/build-with-claude/prompt-caching (Checked in October 2026.)
- Claude usage and plans: https://claude.com/pricing , https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work (Checked in October 2026.)
- Codex pricing, models and configuration: https://learn.chatgpt.com/docs/pricing , https://learn.chatgpt.com/docs/models , https://learn.chatgpt.com/docs/config-file/config-reference (Checked in October 2026.)

---
[← Chapter 6. Making it remember you: CLAUDE.md, AGENTS.md and memory](06-memory-and-rules.md) · [Contents](../README.md#contents) · [Chapter 8. Division of labor: Claude coordinates, Codex writes code, Sonnet writes prose, Pro takes on hard problems →](08-division-of-labor.md)
