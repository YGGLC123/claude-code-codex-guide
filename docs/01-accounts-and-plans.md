# Chapter 1. Getting ready: accounts and subscriptions

This chapter doesn't install any software. It covers three things to think through and settle: what the risks are, how to register an account, and which plan to buy. Installation starts in [Chapter 2](02-install-claude-code.md).

The prices and rules in this chapter are what I found on the official pages in October 2026. They change quickly, so check the official pages before you place an order.

## 1.1 The risks, stated plainly

First, something many tutorials leave out: both Claude and ChatGPT serve only "supported regions," and the list of supported regions is published in plain text.

Before you sign up, check that your country or region is on each company's official supported-regions list. The lists are separate: a region can be on one and missing from the other, so check both.

Anthropic's definition of an unsupported region is broader than "where you are physically located." The official page says that any country or region not on the list is unsupported, and that this covers three cases:

1. You use the service while located in an unsupported region, even if your organization is in a supported region.
2. The user is an entity incorporated or headquartered in an unsupported region, or a person working for such an entity, regardless of where the individual is.
3. The user is an entity directly or indirectly majority-owned or controlled by individuals or entities in an unsupported region, or a person working for such an entity, regardless of where the individual is.

OpenAI's two supported-countries pages (one for ChatGPT, one for the API) carry the same sentence. The original reads: "Accessing or offering access to our services outside of the countries and territories listed below may result in your account being blocked or suspended." In short: accessing or offering access from outside the list may get the account blocked or suspended.

Section 3 of Anthropic's consumer terms also requires you to use the service in compliance with the terms, the usage policy, and the supported regions policy. Its usage policy additionally prohibits accessing the service in violation of the supported regions policy. The usage policy page shows an effective date of 2026-11-12, which is later than the date I checked it, so it may be a newly published version that has not yet taken effect. Check the official page for the exact wording.

> [!IMPORTANT]
> **Regional restrictions and account risk**
> - Both companies state clearly that using the services outside a supported region violates their terms. OpenAI's wording says the account may be blocked or suspended; Anthropic puts this in its consumer terms and usage policy.
> - If an account is banned, the subscription fees you paid and the conversation history in the account may be affected. Whether a refund is possible is up to the company.
> - You have to judge this yourself and bear the responsibility. This guide does not make that decision for you.
> - [Chapter 11](11-other-model-providers.md) covers connecting the tools to other model providers.

This guide assumes you are in a supported region.

## 1.2 Registering accounts

Register only through official channels, with no middleman.

**Claude account:** Register at claude.ai. Follow the prompts: enter your email and the verification email you receive, or complete phone verification if the page asks for it, then sign in.

**ChatGPT account:** Register at chatgpt.com. Again, follow the page's prompts to complete email or phone verification.

I recommend registering both accounts with the same everyday email address, which makes them easier to remember and to recover later. Store the passwords in a password manager, and don't reuse passwords from other sites.

> [!NOTE]
> **Checkpoint**
> After registering, open claude.ai and chatgpt.com in your browser. You should be able to reach the chat interface, and see your account name at the lower left or upper right. At this point both accounts are on the free tier. The next section covers whether to subscribe.

> [!IMPORTANT]
> **Get accounts only through official channels**
> If you don't receive a verification code, or the page won't let you register, check the prompts in the official help center first. Don't use any third party to receive verification codes, set up accounts, or pay on your behalf. That is an easy way to get scammed, it also violates the official terms, and the account can be banned at any time.

## 1.3 Choosing a subscription and plan

The conclusion first: **buy the entry-level plan from one company, use it for two weeks, and upgrade only if you confirm the usage really isn't enough.** Don't start with the top tier.

Subscribe only on the official website or in the official app. Which payment methods are supported is whatever the official subscription page lists; I won't go into it here.

### Claude

| Plan | Monthly price (US$, excluding tax) | Includes Claude Code | Notes |
|---|---|---|---|
| Free | $0 | No | You can chat, but cannot use Claude Code |
| Pro | $20/month; $17/month if paid annually ($200 in one payment) | Yes | The best entry-level choice |
| Max 5x | $100/month | Yes | 5 times the usage of Pro per session |
| Max 20x | $200/month | Yes | 20 times the usage of Pro per session |

A few details that are easy to miss:

- Max can only be paid monthly. There is no annual option.
- On the pricing page, the Max card says only "From $100." The price of the $200 tier is in the official help center.
- A Free claude.ai account does not include Claude Code. Even if you install it, you cannot sign in and use it. You need a paid plan such as Pro or Max (or a Console account billed through the API, covered in [Chapter 2](02-install-claude-code.md)).

### ChatGPT (includes Codex)

| Plan | Monthly price (US$) | Notes |
|---|---|---|
| Free | $0 | The official statement is that Codex is available across all plans, but the page doesn't make clear how much you can do on Free or whether the command line is included; check the official page |
| Go | $8 | Same as above; check the official page |
| Plus | $20 | The best entry-level choice |
| Pro | $100, $200 and $500 tiers | For the differences between the three tiers, check the official page |

Codex's cloud features (assigning a task from the web and letting it run in the cloud) are available only on Plus, the Pro tiers, and the team and enterprise plans. Free and Go don't have them. Prices are listed in US dollars. Whether prices are set in local currencies in some regions isn't stated on the page, so go by what the checkout page shows.

### Usage rules: the 5-hour window and the weekly limit

Neither service is "unlimited once you pay." Both meter usage:

- Claude: Every plan has a usage limit that resets on a rolling **5-hour session window**. Paid plans also have a **weekly limit**. Claude on the web, desktop, and phone and Claude Code share the same pool of usage. How much usage a request consumes depends on the length of the conversation, the model, the features used, and the reasoning effort. There is no fixed "so many messages per day."
- Codex: The official figures are estimates, not fixed limits. For example, on Plus, the number of local messages you can send in each 5 hours ranges from a handful to thousands depending on the model, which is a very wide gap. Local and cloud use draw on the same allowance, and there may also be a weekly limit. ChatGPT Pro currently has no 5-hour limit.

These numbers are adjusted often, so I don't list specific message counts here. Go by the official pricing page.

### When you run out of usage

- Claude: You can turn on usage credits, which are billed pay-as-you-go for usage beyond the limit. In a session, type `/usage-credits` to open the settings, and `/usage` to see the current usage bar. You can also just wait for the window to reset.
- Codex: On Plus and Pro, once the allowance is used up you can buy credits to keep going, or switch to an API key with pay-as-you-go billing. In the CLI, `/status` shows the remaining allowance. On the web, see chatgpt.com/codex/settings/usage.

### My subscription setup

I subscribe to both Claude and ChatGPT: Claude handles project coordination and judgment, and Codex writes the code. The reason is practical: even on Claude's top tier, usage is still tight, while the Codex allowance is comparatively generous. I also worked out the numbers, and paying through the API pay-as-you-go turned out to be poor value, so for long-term use a subscription is the better deal.

Here are three ways to start:

1. **Start with one entry-level plan.** If you just want to try it, subscribe to either Claude Pro or ChatGPT Plus, $20/month. Choose Claude if you mainly write documents and organize material, and ChatGPT if you mainly write code.
2. **One entry-level plan from each.** Claude Pro plus ChatGPT Plus comes to $40/month in total. This is enough for most people. The two allowances back each other up: when one runs out, switch to the other.
3. **Heavy use.** If you keep several project sessions open at once and run them for long stretches, then consider upgrading to Claude Max 5x or the ChatGPT Pro $100 tier. Make that decision after you've used the tools for two weeks.

> [!TIP]
> **Confirm these three things before you buy**
> - Prices and supported regions on the subscription page are whatever the official page says that day. The numbers in this guide are from October 2026.
> - Annual billing is cheaper than monthly, but don't choose annual the first time. Start with monthly and try it out.
> - Claude Max is monthly only, so there is no annual option to consider.

> [!NOTE]
> **Checkpoint**
> After you subscribe, the account settings on claude.ai or chatgpt.com should show your current plan name (Pro, Plus, and so on). To see your usage, wait until Claude Code is installed and then use `/usage` (explained in detail in [Chapter 7](07-saving-usage.md)).

## 1.4 Preparing your computer

**System requirements:**

- Claude Code: macOS 13.0 or later; Windows 10 1809 or later; Ubuntu 20.04 or later; at least 4 GB of RAM; an internet connection is required.
- Codex: The official system requirements table lists macOS 12 or later and Ubuntu 20.04 or later, with a minimum of 4 GB of RAM and 8 GB recommended. For Windows, the two official documents disagree: the table in the installation document says it runs through WSL2, while another document says it can run natively on Windows. Check the official docs. I use it natively on Windows, and [Chapter 3](03-install-codex.md) explains.

For disk space, the official docs give no clear figure; check the official docs. From my experience, leave a good amount of free space, because the projects you ask it to work on, the materials you download, and the files it generates keep piling up.

**If you have a discrete graphics card:** make sure the AI uses it. Speech synthesis, transcription, and training can use CUDA, and video encoding can use `h264_nvenc`. When you hand out a task, put this in the task description. I got this wrong once: 4 to 5 speech processes all ran on the CPU, CPU usage reached 81% while the graphics card was at 4%, and the computer lagged so badly I couldn't do anything else.

**Large computations can go on a cloud server.** For a computation expected to take more than 30 minutes, or that needs more than 8 processes at the same time, I set up a cloud server ahead of time and run it there, leaving my own machine for the browser and everyday editing. Otherwise a long computation takes over the local machine, and the computer lags while you use it. How to use a cloud server, and the rules for it, are covered in [Chapter 12](12-verification-and-safety.md). For now you only need to know the option exists.

### References

- Anthropic supported regions: https://www.anthropic.com/supported-countries (Checked in October 2026.)
- Claude Help Center, supported regions: https://support.claude.com/en/articles/8461763-where-can-i-access-claude (Checked in October 2026.)
- Claude pricing: https://claude.com/pricing (Checked in October 2026.)
- About the Claude Pro plan: https://support.claude.com/en/articles/8325606-what-is-the-pro-plan (Checked in October 2026.)
- About the Claude Max plan: https://support.claude.com/en/articles/11049741-what-is-the-max-plan (Checked in October 2026.)
- Claude usage rules: https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work (Checked in October 2026.)
- Anthropic consumer terms: https://www.anthropic.com/legal/consumer-terms (Checked in October 2026.)
- Anthropic usage policy: https://www.anthropic.com/legal/aup (Checked in October 2026.)
- Claude Code setup and system requirements: https://code.claude.com/docs/en/setup (Checked in October 2026.)
- ChatGPT supported countries: https://help.openai.com/en/articles/7947663-chatgpt-supported-countries (Checked in October 2026.)
- OpenAI API supported countries: https://developers.openai.com/api/docs/supported-countries (Checked in October 2026.)
- Codex pricing and usage: https://learn.chatgpt.com/docs/pricing (Checked in October 2026.)
- Codex sign-in and plans: https://learn.chatgpt.com/docs/auth (Checked in October 2026.)
- About ChatGPT Pro plans: https://help.openai.com/en/articles/9793128-about-chatgpt-pro-plans (Checked in October 2026.)

---
[← Chapter 0. Before you start](00-introduction.md) · [Contents](../README.md#contents) · [Chapter 2. Installing Claude Code →](02-install-claude-code.md)
