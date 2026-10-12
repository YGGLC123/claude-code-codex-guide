# Claude Code & Codex: A Practical Guide

From installation to a day-to-day workflow, for beginners and heavy users alike.

This is a step-by-step guide to Claude Code and Codex, written by a heavy user of both. It starts with accounts and installation, then moves on to memory files, saving usage, dividing work between the two tools, subagents, extensions, verification and troubleshooting. This is the English edition of a Chinese guide.

This guide reflects the versions available in October 2026. These tools change quickly; the official documentation is authoritative.

## Who this is for

- Beginners who want to use AI to get work done. You do not need to know how to program. If you can type and install software, that is enough.
- People who already use these tools but find that their usage limits run out too fast, or that things get confusing once they have several sessions open. The later chapters describe practices built up after making mistakes of my own.

Every step follows the pattern "what to do, what you should see when it is done, what to do if something goes wrong."

## Contents

<!-- toc -->
- [Chapter 0. Before you start](docs/00-introduction.md)
- [Chapter 1. Getting ready: accounts and subscriptions](docs/01-accounts-and-plans.md)
- [Chapter 2. Installing Claude Code](docs/02-install-claude-code.md)
- [Chapter 3. Installing Codex](docs/03-install-codex.md)
- [Chapter 4. Environment setup: Git, Node.js, Python and common Windows problems](docs/04-environment-setup.md)
- [Chapter 5. First steps](docs/05-first-steps.md)
- [Chapter 6. Making it remember you: CLAUDE.md, AGENTS.md and memory](docs/06-memory-and-rules.md)
- [Chapter 7. Saving usage: context, caching and compaction](docs/07-saving-usage.md)
- [Chapter 8. Division of labor: Claude coordinates, Codex writes code, Sonnet writes prose, Pro takes on hard problems](docs/08-division-of-labor.md)
- [Chapter 9. Subagents and background work: getting AI to follow long-running tasks](docs/09-subagents-and-background-work.md)
- [Chapter 10. Extensions: Skills, MCP, hooks and plugins](docs/10-extensions.md)
- [Chapter 11. Using other model providers](docs/11-other-model-providers.md)
- [Chapter 12. Verification and safety](docs/12-verification-and-safety.md)
- [Chapter 13. Troubleshooting quick reference](docs/13-troubleshooting.md)
- [Chapter 14. Worked examples: from an idea to delivery](docs/14-worked-examples.md)
- [Appendix. Command reference and config templates](docs/appendix.md)
<!-- /toc -->

## How to read it

You do not have to read from beginning to end. Choose based on your situation:

- Complete beginner: read from the start through Chapter 5, then you can begin using the tools.
- Already using them: go straight to Chapters 6 through 10 (memory, saving usage, division of labor, subagents, extensions).
- Want to use other model providers: start with Chapter 11.
- Hit an error: see Chapter 13.
- Want quick-reference tables and templates: go to the appendix.

Whichever route you take, read all of Chapter 12 before you let an AI modify any file that matters to you.

## Disclaimer

This is an independent guide. It is not affiliated with or endorsed by Anthropic or OpenAI. Product names belong to their respective owners.

Both services are offered only in supported regions. Check the official lists and terms before you sign up. Prices and limits change, so check the official pages.

## License

The text is licensed under CC BY-NC-SA 4.0 (see [LICENSE](LICENSE) and https://creativecommons.org/licenses/by-nc-sa/4.0/). In short, you may share and adapt it for non-commercial purposes, with attribution, under the same license.

Feel free to copy the commands and configuration snippets into your own setup.

## Author

Written by [YGGLC123](https://github.com/YGGLC123). The guide grew out of my Xiaohongshu (RED) channel 超大果粒橙云课堂.

## Follow on Xiaohongshu / 小红书

You can find me on Xiaohongshu (RED) under two accounts.

| Account | About | Xiaohongshu ID (小红书号) |
|---|---|---|
| AI account 「超大果粒橙云课堂」 | AI tutorials and animations | `ygglc_123` |
| Main account 「阳光果力橙（科研版）」 | My main account, research and study | `ygglc123` |

Search the ID in the Xiaohongshu app to find the account.
