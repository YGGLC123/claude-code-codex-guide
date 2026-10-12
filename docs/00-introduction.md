# Chapter 0. Before you start

This guide reflects the versions available in October 2026. These tools change quickly; the official documentation is authoritative.

## Who this guide is for

I'm YGGLC123 on GitHub. This guide grew out of my Xiaohongshu (RED) channel 超大果粒橙云课堂, where I post AI tutorials. It is written for two kinds of readers:

- Beginners who want to use AI to get work done. You don't need to know how to program. If you can type and install software, that is enough.
- People who already use these tools but find that their usage limits run out too fast, or that things get confusing once they have several sessions open. The later chapters describe the practices I built up after making mistakes of my own.

Every step in this guide follows the pattern "what to do, what you should see when it's done, what to do if something goes wrong." Work through the steps, then compare your result with the checkpoint. If it matches, continue. If it doesn't, read the "if something goes wrong" part of that step, and if that doesn't solve it, look in the troubleshooting reference in [Chapter 13](13-troubleshooting.md).

## What Claude Code and Codex are

They are AI assistants that you install on your computer. A regular chat page can only give you a block of text. These two can read and write files in a folder you choose, run commands, and present you with the result when they finish.

Many people assume they are only for writing code. In practice they handle anything that ends in a file: writing reports, organizing large piles of material, building web pages and videos. All you need to do is state clearly the goal, the scope, and what counts as done.

- **Claude Code** comes from Anthropic.
- **Codex** comes from OpenAI, and you can sign in with a ChatGPT account.

Both work in the command line, and both also have a VS Code extension and a desktop app. On a phone you can keep following their progress. Installation is covered in [Chapter 2](02-install-claude-code.md) (Claude Code) and [Chapter 3](03-install-codex.md) (Codex).

## How I use them day to day

I'll describe my own habits first, and you can pick the ones that suit you.

**I keep several project sessions open at once.** Each session handles one thing only, so they don't interfere with each other.

**Claude handles project coordination and judgment, and Codex does the code writing.** The reason is practical: my Claude subscription is already a high tier and its usage limits are still tight, while the Codex allowance on the ChatGPT side is comparatively generous. I also worked out the numbers, and pay-as-you-go API billing turned out to be poor value for me, so I use subscriptions. The details of this division of labor are in [Chapter 8](08-division-of-labor.md).

**In the conversation, I only look at results.** I don't need to read server output, process status, or logs line by line. Reading them gives me nothing, and a flood of tool calls scrolling past buries the conclusions I actually need to read. So I hand execution work to subagents running in the background, and keep only two things in the foreground, both in language I can follow: decisions that need me, and finished results. When a decision is needed, I require the reply to explain the mechanism, the numbers, the options, and its recommendation in plain language, and never to substitute "see file X." Once, it wrote a decision into a file and gave me only the file name in its reply, and I couldn't make the decision. This approach runs through several later chapters, and [Chapter 5](05-first-steps.md) and [Chapter 9](09-subagents-and-background-work.md) expand on it.

## Estimating the cost

The prices below are the monthly US dollar prices listed officially in October 2026; the Claude prices exclude tax. The official pages don't clearly say whether prices differ by region, so confirm on the official page before you pay.

| Platform | Plan | Monthly price (US$) | Notes |
|---|---|---|---|
| Claude | Free | 0 | Does not include Claude Code |
| Claude | Pro | 20 | Includes Claude Code; about 17 per month if paid annually |
| Claude | Max 5x | 100 | Includes Claude Code; monthly billing only |
| Claude | Max 20x | 200 | Includes Claude Code; monthly billing only |
| ChatGPT | Free | 0 | What Codex offers on Free is described on the official page; check it |
| ChatGPT | Go | 8 | What Codex offers on Go is described on the official page; check it |
| ChatGPT | Plus | 20 | Includes Codex |
| ChatGPT | Pro | 100 / 200 / 500 | Three tiers; includes Codex |

Both companies meter usage in 5-hour windows. Paid Claude plans also have a weekly limit, and Codex may have a weekly limit too (ChatGPT Pro currently has no 5-hour limit). The specific rules are in [Chapter 1](01-accounts-and-plans.md) and [Chapter 7](07-saving-usage.md).

I suggest three ways to combine them:

1. **Start with one.** Pick either Claude Pro (US$20) or ChatGPT Plus (US$20) and use it for two weeks. For most beginners, this is enough.
2. **One entry-level plan from each.** Claude Pro plus ChatGPT Plus comes to US$40 a month. When one runs out of usage, switch to the other. This is also the starting setup I recommend.
3. **Heavy use.** Only when an entry-level plan on one side runs short should you move up: Claude to Max, ChatGPT to Pro. Don't buy the top tier at the start. First find out where your usage actually runs out.

> [!IMPORTANT]
> Always buy subscriptions on the official website or in the official app. [Chapter 1](01-accounts-and-plans.md) explains how to choose a plan and what the two companies' official pages say about supported regions. Make sure the service is available where you live before you decide to buy.

This guide assumes you are in a supported region. If you cannot use the official services for now, [Chapter 11](11-other-model-providers.md) covers connecting these two tools to other model providers. The results fall short of the originals, but it works for getting started and for small everyday tasks.

## Suggestions for reading

You don't have to read from beginning to end. Choose based on your situation:

| Your situation | Where to read |
|---|---|
| Complete beginner | Read from the start through Chapter 5, then you can begin using the tools |
| Already using them | Go straight to Chapters 6 through 10: memory, saving usage, division of labor, subagents, extensions |
| Want to use other model providers | Start with Chapter 11 |
| Hit an error | See Chapter 13 |
| Want quick-reference tables and templates | Go to the appendix |

[Chapter 12](12-verification-and-safety.md) covers verification and safety, and [Chapter 14](14-worked-examples.md) has three complete worked examples. Whichever route you take, I recommend reading all of Chapter 12 before you let an AI modify any file that matters to you. The appendix is [here](appendix.md).

> [!TIP]
> I checked the commands, setting names, and prices in this guide against the official documentation. Where I couldn't verify something, I marked it "check the official docs" instead of making up an answer. When you see those words, confirm on the official page.

### References

- Claude pricing and plans: https://claude.com/pricing (Checked in October 2026.)
- About the Claude Pro plan: https://support.claude.com/en/articles/8325606-what-is-the-pro-plan (Checked in October 2026.)
- About the Claude Max plan: https://support.claude.com/en/articles/11049741-what-is-the-max-plan (Checked in October 2026.)
- Claude Code setup documentation: https://code.claude.com/docs/en/setup (Checked in October 2026.)
- Codex pricing and plans: https://learn.chatgpt.com/docs/pricing (Checked in October 2026.)
- About ChatGPT Pro tiers: https://help.openai.com/en/articles/9793128-about-chatgpt-pro-plans (Checked in October 2026.)

---
[Contents](../README.md#contents) · [Chapter 1. Getting ready: accounts and subscriptions →](01-accounts-and-plans.md)
