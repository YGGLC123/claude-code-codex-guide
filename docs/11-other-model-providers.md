# Chapter 11. Using other model providers

Claude Code and Codex can also be used with models from other providers. You install the tools as usual and change only the endpoint they connect to, pointing it at a compatible endpoint that the provider officially offers. This chapter walks through the steps.

First, set your expectations: this way you are using the provider's model, not Claude or GPT. The results are not on par with the originals, and I treat this setup as suitable for learning, for writing in a language the provider's model is strong in, and for small everyday tasks. For heavy or difficult work, use the original models.

Model names and prices below are what the providers' official pages showed on 2026-10-12; they change quickly, so check each provider's integration page.

> [!IMPORTANT]
> Providers update model names, plans and prices quickly, and different pages from the same provider can disagree because some are newer than others. Everything in this chapter was written from what I found in October 2026, and **the provider's official integration page is authoritative**. The link to each provider's page is listed after the table and in the references at the end of the chapter.

## 11.1 How it works: the client is only a shell

Claude Code and Codex are only clients installed on your computer. The model that does the work is whatever endpoint they connect to.

- Claude Code connects to Anthropic by default. Many providers offer an "Anthropic-compatible endpoint": replace the address and key with the provider's, and Claude Code talks to the provider's model.
- Codex connects to OpenAI by default. Providers offer an "OpenAI-compatible endpoint": add a model provider to Codex's configuration file and point it at that provider.

With this setup, your requests go directly to the provider you chose.

> [!IMPORTANT]
> **Codex now supports only the Responses API**
> In the current Codex configuration, `wire_api` accepts only `responses`; `wire_api = "chat"` is no longer supported. The official configuration reference says: "Protocol used by the provider. `responses` is the only supported value, and it is the default when omitted." In other words, an endpoint that offers only Chat Completions cannot be connected, and a new Codex cannot be used with a provider that has no Responses endpoint.
> Some providers that offer only Chat Completions tell you to install an old Codex release (0.80.0) instead. I don't recommend downgrading for this; pick a provider that has a Responses endpoint.
> Claude Code has no such limit: every provider below gives an Anthropic-compatible address.

## 11.2 Provider comparison

The table contains only what I checked in the providers' official documentation. Prices are in US dollars as shown on the official pages on 2026-10-12.

| Provider | Anthropic-compatible base URL (for Claude Code) | Codex (Responses API) | Coding plan / pricing (USD) |
|---|---|---|---|
| DeepSeek | `https://api.deepseek.com/anthropic` | Supported | No monthly plan; pay-as-you-go only, priced in USD |
| Kimi Open Platform (international) | `https://api.moonshot.ai/anthropic` | Supported | No subscription; pay-as-you-go only, priced in USD |
| Kimi Code (international) | `https://api.kimi.ai/coding/` | Supported | Part of the Kimi membership (Plus and above): Plus $19 / Pro $39 / Max $99 / Ultra $199 per month, or $180 / $372 / $948 / $1,908 per year; the Go tier has no coding quota |
| Z.ai GLM Coding Plan | `https://api.z.ai/api/anthropic` | Supported (requires the dedicated Responses endpoint) | Lite $18 / Pro $80 / Max $168 per month; quarterly billing is 20% off and yearly billing 30% off |

A few notes:

- **Allowed tools**: the plans from Z.ai and Kimi Code state that the subscription is for use inside officially supported tools (Z.ai) or for interactive use only (Kimi Code). Claude Code and Codex are on Z.ai's list of supported tools. See each provider's plan description for the exact rules.
- **Kimi has two separate products**: Kimi Code (part of the Kimi membership) and the Kimi Open Platform (pay-as-you-go). Their addresses and keys are not interchangeable.
- **Z.ai changed its plan on 2026-07-30**: it moved to a credit-based system. Existing subscribers keep their old price until the end of their current term. Check the Z.ai pages for the current rules.

Each provider's integration page (model names and the latest instructions are authoritative there):

- DeepSeek: `https://api-docs.deepseek.com/quick_start/agent_integrations/claude_code`
- Kimi Open Platform: `https://platform.kimi.ai/docs/guide/claude-code-kimi`
- Kimi Code: `https://www.kimi.com/code/docs/en/third-party-tools/claude-code.html`
- Z.ai: `https://docs.z.ai/devpack/tool/claude`

> [!TIP]
> How to choose: if you only want to try it and keep costs lowest, use DeepSeek with pay-as-you-go billing. Top up a small amount first; you can see the actual spend in the console. If you want a monthly plan with a defined allowance, consider Z.ai or Kimi Code. If you also want to use Codex, first confirm that the provider has a Responses endpoint (the third column of the table).

## 11.3 Connecting Claude Code

### First, use a separate configuration directory

If you edit `~/.claude/settings.json` directly, it interferes with your existing Claude login. My approach is a separate configuration directory used only for the other provider, so that plain `claude` keeps using the original account. This relies on the official environment variable `CLAUDE_CONFIG_DIR` (it changes the configuration directory, which defaults to `~/.claude`); each directory has its own settings, session history and login.

I call this directory `.claude-alt`; you can name it anything.

Windows: create a small launcher file, `claude-alt.cmd`. First find which folder `claude` is installed in:

```powershell
where.exe claude
```

Create a new file in Notepad containing the three lines below, and save it in the folder you just found, with the file name `claude-alt.cmd` (set "Save as type" to "All files"):

```cmd
@echo off
set "CLAUDE_CONFIG_DIR=%USERPROFILE%\.claude-alt"
claude %*
```

From then on, typing `claude-alt` in any terminal starts the alternate-provider version, and typing `claude` starts the original one. The variable takes effect only at launch and does not stay set in the terminal.

> [!TIP]
> **Why not a PowerShell profile**
> The default PowerShell execution policy on Windows blocks scripts in the profile, so even if you write the change there, it does not take effect. A `.cmd` file is not subject to this restriction, so beginners can copy it as is.

macOS / Linux (Terminal): add one alias line to `~/.zshrc` (or `~/.bashrc`), then open a new terminal:

```bash
alias claude-alt='CLAUDE_CONFIG_DIR=$HOME/.claude-alt claude'
```

### Writing settings.json in that directory

The file is at `C:\Users\<your-username>\.claude-alt\settings.json` on Windows and `~/.claude-alt/settings.json` on macOS / Linux. Create the directory first if it does not exist. The example below uses DeepSeek; for other providers, replace the values in `env` with those on the provider's integration page:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "<your API key>",
    "ANTHROPIC_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-flash",
    "CLAUDE_CODE_AUTO_COMPACT_WINDOW": "786432"
  }
}
```

Create `<your API key>` in the provider's developer platform console. It is usually shown only once at creation, so save it somewhere safe right away. The model name (here `deepseek-flash`) is whatever DeepSeek's official integration page says; if it changes, change it too.

Item by item:

- The three `DEFAULT_*_MODEL` variables correspond to the three tiers in Claude Code (Opus, Sonnet, Haiku). Some providers require all three to be set to the same model.
- `CLAUDE_CODE_SUBAGENT_MODEL` is the model used for subagent tasks. If you leave it unset, subagent tasks request a model name the provider does not recognize. Providers' documentation generally asks you to set it to the provider's model as well.
- The `[1m]` suffix on a model name turns on the 1M context window, and `CLAUDE_CODE_AUTO_COMPACT_WINDOW` is the matching auto-compaction window. If Claude Code says the model does not exist after you add `[1m]`, upgrade Claude Code first.
- Use `ANTHROPIC_AUTH_TOKEN` for the key: it is sent in the `Authorization` request header. If you use `ANTHROPIC_API_KEY` instead, then once it is set it takes precedence over the subscription you are logged in with. Kimi Code's official example does use `ANTHROPIC_API_KEY`; follow its page and do not change it yourself.
- Put these `ANTHROPIC_*` settings only in `.claude-alt`. **Do not** set them as system environment variables, or they change the original `claude` as well.

After saving, launch it:

```powershell
claude-alt
```

> [!NOTE]
> **Checkpoint**
> After launch you go straight into the chat interface. Type "Hello, which model are you?" If it answers normally, the connection works. Note that the model's own statement of its name is not always accurate; the call log in the provider's console is the reliable record.

> [!WARNING]
> **Launch asks you to log in to Anthropic**
> Some providers' setup steps write the line `"hasCompletedOnboarding": true` into `~/.claude.json` (on Windows, `C:\Users\<your-username>\.claude.json`) to skip the login prompt; Kimi Code's official setup script does this, for example. Whether it is needed with `CLAUDE_CONFIG_DIR` set, and whether that file moves into the new directory, is not documented clearly, so check the official docs. I suggest trying without it first; if the login prompt still appears, put a `.claude.json` containing that line in the `.claude-alt` directory as well, then launch again.

> [!WARNING]
> **Conflicting old variables**
> If you previously set `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` or `ANTHROPIC_BASE_URL`, clear them before configuring, or they conflict with the new settings. Kimi's documentation also warns that if you configure only some of the tier variables, the scenarios that use the missing ones fail silently, so set all three tiers.

## 11.4 Connecting Codex

Codex reads its configuration from `~/.codex/config.toml`. As with Claude, I use a separate directory, pointed to by the environment variable `CODEX_HOME`, containing its own `config.toml`, so the original Codex configuration is untouched.

Windows (PowerShell): create the directory and store the key (it exists only on your own computer):

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex-alt"
setx KIMI_API_KEY "<your API key>"
```

A variable set with `setx` takes effect only in a **newly opened terminal**. Then open `C:\Users\<your-username>\.codex-alt\config.toml` in Notepad and write the following (this example uses the Kimi Open Platform, which supports the Responses API):

```toml
model = "kimi-k3"
model_provider = "kimi"
model_context_window = 1048576

[model_providers.kimi]
name = "Kimi"
base_url = "https://api.moonshot.ai/v1"
env_key = "KIMI_API_KEY"
wire_api = "responses"
```

To launch with that directory, do the same as for `claude-alt.cmd` above: use `where.exe codex` to find the folder `codex` is in, then put a `codex-alt.cmd` there:

```cmd
@echo off
set "CODEX_HOME=%USERPROFILE%\.codex-alt"
codex %*
```

From then on, typing `codex-alt` uses the alternate provider, and `codex` still uses the original configuration.

macOS / Linux (Terminal):

```bash
mkdir -p ~/.codex-alt
export KIMI_API_KEY="<your API key>"
CODEX_HOME=$HOME/.codex-alt codex
```

On macOS, create `~/.codex-alt/config.toml` the same way, with the same content as above.

Notes:

- `env_key` holds the **name of the environment variable** that stores the key, not the key itself. Kimi's documentation recommends not writing the key directly into config.toml.
- The line `wire_api = "responses"` is essential. If a provider has no Responses endpoint, there is no value to put here, and the provider cannot be connected.
- For other providers' `base_url` and model names, see their Codex integration pages: DeepSeek, Kimi Open Platform, Kimi Code and Z.ai each have a dedicated page. Some providers also ask you to write a `models.json` first to declare the models (DeepSeek, Z.ai and Kimi Code do); follow the page.
- If you keep `model_catalog_json = "~/.codex/..."` in a different directory, then after you change `CODEX_HOME` the path must be changed to the new directory as well.
- If a message such as "Model metadata for kimi-k3 not found" appears at startup, Kimi's documentation says this is normal.

> [!NOTE]
> **Checkpoint**
> After you run it you enter the Codex interface, and sending one message gets a reply. Codex shows session history separately for each login method: sessions created through a third-party endpoint and sessions created with a ChatGPT login are not in the same list. Nothing has been lost.

## 11.5 Notes

1. **Plan allowances can only be used in the officially allowed tools.** Z.ai and Kimi Code state this explicitly: the plan is for use inside supported coding tools (Kimi Code: interactive use only, not non-interactive automation). Do not use a plan key to write scripts that call the API directly, since that goes against their terms. My own practice: plan keys go only into tools such as Claude Code and Codex; when I need a script to make batch calls, I use a separate pay-as-you-go key.
2. **The endpoint and the key type have to match, or you get a 401.** Plan endpoints, coding endpoints and pay-as-you-go endpoints use different paths, and plan keys and pay-as-you-go keys are not interchangeable. A wrong path shows up as an "invalid key" 401 error. When I tried one provider's plan, I had the path wrong and kept getting "invalid key"; the key was fine, the address just did not match it.
3. **A 401 is retried for a long time by default.** Claude Code retries several times by default on a 401, so you wait a while. I have reduced the retry count. If you need it, look up the exact variable name and value in the official Claude Code environment variable documentation; the official docs are authoritative.
4. **For pay-as-you-go billing, set a budget first.** For pay-as-you-go models such as video generation, you must enable the model in the console first and confirm that the key's permissions include it, or you get a "not enabled" or "access denied" error. Put a budget cap in your script and record the cost of every call. My first trial had a cap of CNY 20 (about US$3), and the actual spend was CNY 5.44 (under US$1); the cap keeps costs under control.
5. **Your code and files are sent to that provider's servers.** Whoever's model you use receives your content. Do not send sensitive material.
6. **I did not test each provider with a real key for this chapter.** I read the official integration documentation and reproduced what it says. If something goes wrong when you follow it, check the provider's integration page for the latest wording first, then see [Chapter 13](13-troubleshooting.md).

## 11.6 A common pattern: using a second model to polish copy

When I write for social media in a language I don't write natively in, or for a Chinese-speaking audience, drafting directly with Claude or Sonnet can come out with a translated feel that does not read naturally. My process has three steps:

1. I (or Claude) settle the facts, the structure and the key points first, producing a first draft.
2. I hand the draft to a model that is strong in that language and ask it to polish the text into natural, fluent prose. If the first pass changes too little, I raise the request one level, for example "rewrite freely".
3. When the draft comes back, **check every number, fact and link** to see whether anything was changed. Numbers and proper nouns are what polishing most easily alters without notice, so this step cannot be skipped.

The division of labor is clear: we own the facts, and the model that is good at the language owns the feel.

### References

Checked in October 2026.

- Codex configuration reference (`wire_api`): `https://learn.chatgpt.com/docs/config-file/config-reference`
- Codex advanced configuration (`CODEX_HOME`, model providers): `https://learn.chatgpt.com/docs/config-file/config-advanced`
- Claude Code environment variables: `https://code.claude.com/docs/en/env-vars`
- Claude Code authentication: `https://code.claude.com/docs/en/authentication`
- DeepSeek, Claude Code: `https://api-docs.deepseek.com/quick_start/agent_integrations/claude_code`
- DeepSeek, Codex: `https://api-docs.deepseek.com/quick_start/agent_integrations/codex`
- Kimi Open Platform, Claude Code: `https://platform.kimi.ai/docs/guide/claude-code-kimi`
- Kimi Open Platform, Codex: `https://platform.kimi.ai/docs/guide/codex-kimi`
- Kimi Code, Claude Code: `https://www.kimi.com/code/docs/en/third-party-tools/claude-code.html`
- Kimi Code, Codex: `https://www.kimi.com/code/docs/en/third-party-tools/codex.html`
- Z.ai, Claude Code: `https://docs.z.ai/devpack/tool/claude`
- Z.ai, Codex: `https://docs.z.ai/devpack/tool/codex`
- Z.ai, subscribe page: `https://z.ai/subscribe`
- Z.ai, GLM Coding Plan overview: `https://docs.z.ai/devpack/overview`

---
[← Chapter 10. Extensions: Skills, MCP, hooks and plugins](10-extensions.md) · [Contents](../README.md#contents) · [Chapter 12. Verification and safety →](12-verification-and-safety.md)
