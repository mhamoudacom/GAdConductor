<h1 align="center">🎼 GAdConductor</h1>

<p align="center"><b>Google Ads&nbsp;+&nbsp;Claude</b> · by Mohamed Hamouda</p>

<p align="center">
  Let <b>Claude Code</b> or the <b>Codex CLI</b> conduct your Google Ads account directly.<br>
  No Sheets, no CSVs, no manual uploads — your AI assistant reads and writes Google Ads for you.
</p>

<p align="center">
  Built by <a href="https://mhamouda.com"><b>Mohamed Hamouda</b></a> ·
  Digital Marketing Manager · Performance &amp; Growth Marketing Consultant · Marketing Automation &amp; AI Trainer · SaaS Builder
</p>

---

> ⚠️ **Not official. Not an MCP.**
> GAdConductor is an **independent, community skill** — it is **not** an official Model
> Context Protocol (MCP) server, and it is **not affiliated with, endorsed by, or sponsored
> by Google or Anthropic**. It runs entirely on your own Google Ads API credentials.
> "Google Ads", "Google", and "Claude" are trademarks of their respective owners and are
> used here only to describe compatibility.

---

## What's inside

A collection of **skills** that let your AI assistant conduct the Google Ads API.

| Skill | What it does | Status |
|---|---|---|
| **`google-ads-connect`** | The full connection setup — Manager Account, developer token + Basic Access (auto-filled), OAuth, refresh token, verified `.env`. | ✅ Available |
| `google-ads-campaigns` | Create Search / Display / Performance Max / Demand Gen campaigns, ad groups, keywords, and ads via the API. | 🔜 Coming |
| `google-ads-optimize` | Keyword research, negative keywords, search-term cleanup, and performance analysis. | 🔜 Coming |

> **This release = Part 1: the connection.** Campaign management and optimization ship as
> updates to this same repo — ⭐ star it to get them.

---

## Install

### Claude Code
```bash
# personal (all projects)
cp -r skills/google-ads-connect ~/.claude/skills/
```
Then in Claude Code: type `/google-ads-connect` — or just say **“connect me to Google Ads.”**

### Codex CLI
```bash
# drop the skill anywhere in your project
cp -r skills/google-ads-connect ./
```
Then tell Codex: **“read google-ads-connect/AGENTS.md and connect me to Google Ads.”**
Codex reads `AGENTS.md`, which points it at the same step-by-step flow.

*(One skill, two runners — the flow is identical; only who runs it differs.)*

---

## How it works with Codex

Codex doesn't auto-discover skills the way Claude Code does. Each skill ships an
**`AGENTS.md`** — the file Codex reads by convention. It tells Codex when to activate and
hands it the same source of truth (`SKILL.md`): Codex builds your project files, runs the
scripts, interviews you for the Basic Access form, generates your answers and Design Doc PDF,
and verifies the live connection. You only click **Allow** in your browser once.

## How it works with Google

Google Ads has no simple “API key.” Access is a short chain the skill walks you through:

1. **Manager Account (MCC)** — Google only issues a developer token to a manager account.
2. **Developer token + Basic Access** — a manually-reviewed application. The skill **interviews
   you (6 questions), writes every copy-paste answer, and auto-builds the required Design Doc
   PDF** — the exact part where most people get rejected.
3. **Google Cloud project** — the API runs through Google Cloud; the skill enables it.
4. **OAuth credentials + refresh token** — a one-time “Allow” in your browser mints a refresh
   token (a permission slip your assistant reuses forever).

The result is a `.env` with **6 values**. Your assistant loads them and talks to Google Ads
directly. Google reviews Basic Access in ~1–3 business days; your token works on **test
accounts immediately** and on **real accounts** once approved.

> **Note on Video (YouTube) campaigns:** the Google Ads API cannot create or edit Video
> campaigns — a Google restriction. Search / Display / Performance Max / Demand Gen all work
> via the API; Video is done in the Ads UI.

---

## Security

- The **refresh token is a master key** — anyone with it can manage your account. Never share
  or commit it. The skill git-ignores `.env` automatically before writing any secret.
- Revoke access anytime at <https://myaccount.google.com/permissions>.

---

## License

MIT © Mohamed Hamouda — see [LICENSE](LICENSE).
