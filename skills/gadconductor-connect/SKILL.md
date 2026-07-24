---
name: gadconductor-connect
description: >-
  Connect Claude Code or the Codex CLI directly to a Google Ads account through
  the Google Ads API — no Sheets, no CSVs, no manual uploads. This one self-
  contained skill builds the whole project in a folder the user chooses, then
  guides them through: a Manager Account (MCC), the developer token + Basic
  Access application (it interviews the user and generates their copy-paste
  answers AND builds their Design Doc PDF automatically), a Google Cloud project,
  OAuth credentials, and a refresh token — ending with a verified live
  connection and a .env holding the 6 credentials. Use whenever the user wants
  to set up Google Ads API access, obtain or approve a developer token, generate
  a refresh token, complete the Basic Access form, or troubleshoot a Google Ads
  API connection on Claude or Codex.
---

# Connect Claude / Codex to Google Ads

> A skill by **Mohamed Hamouda** — https://mhamouda.com
> Digital Marketing Manager · Performance & Growth Marketing Consultant · Marketing Automation & AI Trainer · SaaS Builder

**Part 1 — the connection.** Building, optimizing, and reporting on campaigns come
in the follow-up skills. By the end of this one the user has a `.env` with 6 values
and their assistant can read + write Google Ads directly.

---

## How this skill is used

The user drops this single file into a skill folder and invokes it. It works on both:

- **Claude Code** — folder `~/.claude/skills/gadconductor-connect/SKILL.md` (or per-project
  `<project>/.claude/skills/…`). It triggers on `/gadconductor-connect` or a request like
  *"connect me to Google Ads."*
- **Codex CLI** — the user says *"read SKILL.md and connect me to Google Ads."* Codex
  reads this file and follows it. The flow is identical; only the runner differs.

---

## Operating contract (read first)

- **You are the guide. The user only does browser sign-ins and approvals.** You do
  everything else: create the project files, run the scripts, interview the user,
  generate their Basic Access answers, build their Design Doc PDF, and verify.
- Move **one step at a time** and wait for the user before continuing.
- **Security, non-negotiable:**
  - Never ask for passwords or 2-step codes — those only happen in the user's browser.
  - Put every secret in `.env` only. Create `.gitignore` with `.env` in it **before**
    writing any secret.
  - The refresh token is a master key. Never print it in chat, never commit it.
- If any browser page shows a "Confirm it's you" / security check, that is the user's
  to complete — pause and let them.

---

## Step 0 — Build the project (you do this now)

Ask the user which folder to use (default: `./google-ads`). In that folder:

1. Write `.gitignore`:
   ```
   .env
   *.json
   venv/
   __pycache__/
   ```
2. Write `.env` (empty template):
   ```
   GOOGLE_ADS_DEVELOPER_TOKEN=
   GOOGLE_ADS_CLIENT_ID=
   GOOGLE_ADS_CLIENT_SECRET=
   GOOGLE_ADS_REFRESH_TOKEN=
   GOOGLE_ADS_LOGIN_CUSTOMER_ID=
   GOOGLE_ADS_CUSTOMER_ID=
   ```
3. Write the two scripts from **Appendix A** (`get_refresh_token.py`, `test_connection.py`).
4. Create a virtualenv and install deps:
   ```bash
   python3 -m venv venv
   ./venv/bin/pip install -q --upgrade pip
   ./venv/bin/pip install -q google-ads google-auth-oauthlib python-dotenv
   ```
   (Windows: `python -m venv venv` then `venv\Scripts\pip install …`.)

Tell the user Step 0 is done and move on.

---

## Step 1 — Manager Account (MCC)

User opens <https://ads.google.com/home/tools/manager-accounts/> → **Create a manager
account** → name, country, currency → Submit. The **Customer ID** (`123-456-7890`) is in
the top-right header next to the avatar. Ask them to paste it; strip the dashes and save
as `GOOGLE_ADS_LOGIN_CUSTOMER_ID`.
> ⚠️ Google won't issue a developer token to a regular account — an MCC is required.

## Step 2 — Link the real Ads account

Inside the MCC → **Manage accounts → Sub-account settings → Link existing account** →
enter the real account's Customer ID → **Send request**. The user accepts via the email
from `ads-account-noreply@google.com` (or the bell icon inside the destination account),
signed in as that account. Save the real ID (dashes stripped) as `GOOGLE_ADS_CUSTOMER_ID`.

## Step 3 — Developer token + Basic Access  ← the hard part; you do the heavy lifting

Do these three moves **before** sending the user to the form:

### 3.1 — Interview the user (ask all six)
1. Company / brand name (exactly as on the website)?
2. Website URL (must be live)?
3. Contact email? (a `you@yourdomain.com` address is stronger than Gmail; Gmail is a fine fallback)
4. Country (principal place of business)?
5. Do they advertise **their own** business or **clients'** accounts?
6. Which campaign types will they actually run? (Search / Display / Performance Max / Video / Shopping / Demand Gen)

Enforce the **consistency rule**: email domain + Company URL + Company name must agree.
Pick **Company type = Advertiser** for own-business (highest approval rate) or **Agency/SEM**
only if they manage multiple clients and the site shows a client roster.

### 3.2 — Generate their copy-paste answers
Fill the templates in **Appendix B** with their answers and hand them each field ready to
paste — the Company Info form and the 12-field Basic Access form, including the *Intended
use* paragraph and *Field 7* paragraph.
> ⚠️ Never mention AI, LLMs, Claude, or any third-party/"processing" service anywhere in the
> application or the PDF. Describe the tool at the data-flow level only. Reviewers reject on this.

### 3.3 — Build their Design Doc PDF (Basic Access field 8)
Take the HTML in **Appendix C**, replace every `{{placeholder}}` with their details, save it
as `design-doc.html` in the project, and render to PDF:
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf="design-doc.pdf" --print-to-pdf-no-header \
  "file://$(pwd)/design-doc.html"
```
(No Chrome path? Open `design-doc.html` in any Chrome → Print → Save as PDF.) Give the user
`design-doc.pdf` to upload. Keep it to the 6 sections + one wireframe (~400 words).

### 3.4 — Walk the submission
Inside the MCC → **Admin → API Center** → fill Company Info → **Apply for Basic Access** →
paste the generated answers → upload the PDF → submit. Save the token string now as
`GOOGLE_ADS_DEVELOPER_TOKEN`. Offer **brand verification** (in Google Cloud) to speed review.
> ⚠️ API Center only appears **inside an MCC**, not a regular account. The token works on test
> accounts immediately and on real accounts once Google emails approval (~1–3 business days).

## Step 4 — Google Cloud project

<https://console.cloud.google.com/projectcreate> → name it (e.g. `google-ads-dashboard`) →
Create → select it → search **Google Ads API** → **Enable**.
> ⚠️ The Cloud **project ID** Google assigns can differ from what you typed — that's fine.

## Step 5 — OAuth consent screen (now "Google Auth Platform")

**APIs & Services → OAuth consent screen** (or search "Google Auth Platform") → **Get
started** → App name + support email → **External** → contact email → agree → Create. Then
**Audience → Test users → + Add users** → add the same Google email that owns the Ads account.
> ⚠️ The **App name cannot contain the word "Google"** — Google rejects it. Use e.g. `Ads Dashboard`.

## Step 6 — OAuth credentials

**APIs & Services → Credentials → + Create credentials → OAuth client ID → Desktop app** →
name it → Create → **Download JSON**. Save `client_id` → `GOOGLE_ADS_CLIENT_ID` and
`client_secret` → `GOOGLE_ADS_CLIENT_SECRET`.
> ⚠️ Google no longer re-displays the client secret. If lost, open the client → **Add secret**
> and read the new one from the JSON. The digits before the dash in the client_id are your
> **Cloud project number** (Basic Access field 2).

## Step 7 — Refresh token

Run `./venv/bin/python3 get_refresh_token.py`. The user's browser opens → they pick the
**same** Google account → **Advanced → Go to … (unsafe) → Allow** (their own unverified app).
Capture the printed token into `GOOGLE_ADS_REFRESH_TOKEN`. **Do not echo it in chat.**

## Step 8 — Verify

Run `./venv/bin/python3 test_connection.py`.
- ✓ Lists their campaigns → **fully connected. Done.**
- ✗ `only approved for test accounts` → Basic Access still pending; the same token starts
  working the moment the approval email lands — nothing to re-run.

## Step 9 — Make it reusable for every future session (do this after Step 8 passes)

This connection is **local code, not an MCP connector** — so a *new* chat won't magically
"see" Google Ads; it must run the local Python. Leave three files in the project folder so any
future Claude Code / Cowork / Codex session that opens this folder knows the connection exists
and how to use it, without re-connecting:

1. Write **`gads.py`** — the reporting helper from **Appendix D**.
2. Write **`CLAUDE.md`** — the project instructions from **Appendix D**, filling the account map
   from the user's real accounts (run `./venv/bin/python3 gads.py accounts`, then ask the user
   which sub-account is which).
3. Write **`AGENTS.md`** — an identical copy of `CLAUDE.md` (Codex/Cowork read this filename).

Then tell the user: *in any new chat, point it at this folder and ask for your campaigns — it
runs `gads.py` instead of hunting for a connector.* Quick check:
```bash
./venv/bin/python3 gads.py accounts
./venv/bin/python3 gads.py perf <ACCOUNT_ID>
```
> ⚠️ This works only where the assistant can **run local code** (Claude Code, Cowork with a
> terminal, Codex). It does **not** make Google Ads appear as a plug-and-play MCP "connector"
> like Meta — that would need a dedicated Google Ads MCP server (a separate project).

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `developer token is only approved for use with test accounts` | Basic Access pending. Same token works on real accounts after approval. |
| `invalid_grant` on refresh token | Re-run Step 7 — token revoked or a character lost on paste. |
| `PERMISSION_DENIED / login-customer-id not set` | `GOOGLE_ADS_LOGIN_CUSTOMER_ID` must be the **MCC** ID, not the real account. Both go in `.env`. |
| `not authorized to access customer` | MCC not linked / invite not accepted. Redo Step 2. |
| OAuth "app name does not comply" | App name contains "Google" — rename it. |
| Can't see the client secret again | Open the OAuth client → **Add secret** → read the new one from the JSON. |
| API Center missing | You're in a regular account — switch into the MCC. |
| `Unrecognized field` in a GAQL query | Field names change across API versions — drop/rename it. |
| Creating a **Video** (YouTube) campaign → `MUTATE_NOT_ALLOWED` | Expected: the API can't create/edit Video campaigns — use the Ads UI. Search / Display / Performance Max / Demand Gen work via API. |
| Suspected leaked refresh token | <https://myaccount.google.com/permissions> → revoke the app → re-run Step 7. |

---

## Appendix A — scripts to write in Step 0

**`get_refresh_token.py`**
```python
"""Generates a Google Ads refresh token. Reads CLIENT_ID/SECRET from .env."""
import os
from dotenv import load_dotenv
from google_auth_oauthlib.flow import InstalledAppFlow

SCOPES = ["https://www.googleapis.com/auth/adwords"]
load_dotenv(".env")
client_id = os.getenv("GOOGLE_ADS_CLIENT_ID") or input("CLIENT_ID: ").strip()
client_secret = os.getenv("GOOGLE_ADS_CLIENT_SECRET") or input("CLIENT_SECRET: ").strip()

config = {"installed": {
    "client_id": client_id, "client_secret": client_secret,
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "redirect_uris": ["http://localhost"]}}

flow = InstalledAppFlow.from_client_config(config, SCOPES)
creds = flow.run_local_server(port=0, prompt="consent", access_type="offline")
print("\n=== REFRESH TOKEN ===\n" + creds.refresh_token + "\n=====================")
print("Paste this into .env as GOOGLE_ADS_REFRESH_TOKEN")
```

**`test_connection.py`**
```python
"""Verifies the Google Ads connection by listing up to 5 campaigns."""
import os
from dotenv import load_dotenv
from google.ads.googleads.client import GoogleAdsClient

load_dotenv(".env")
req = ["GOOGLE_ADS_DEVELOPER_TOKEN","GOOGLE_ADS_CLIENT_ID","GOOGLE_ADS_CLIENT_SECRET",
       "GOOGLE_ADS_REFRESH_TOKEN","GOOGLE_ADS_LOGIN_CUSTOMER_ID","GOOGLE_ADS_CUSTOMER_ID"]
missing = [k for k in req if not os.getenv(k)]
if missing:
    raise SystemExit("Missing in .env: " + ", ".join(missing))

client = GoogleAdsClient.load_from_dict({
    "developer_token": os.getenv("GOOGLE_ADS_DEVELOPER_TOKEN"),
    "client_id": os.getenv("GOOGLE_ADS_CLIENT_ID"),
    "client_secret": os.getenv("GOOGLE_ADS_CLIENT_SECRET"),
    "refresh_token": os.getenv("GOOGLE_ADS_REFRESH_TOKEN"),
    "login_customer_id": os.getenv("GOOGLE_ADS_LOGIN_CUSTOMER_ID"),
    "use_proto_plus": True})
ga = client.get_service("GoogleAdsService")
rows = ga.search(customer_id=os.getenv("GOOGLE_ADS_CUSTOMER_ID"),
    query="SELECT campaign.name, campaign.status FROM campaign LIMIT 5")
print("\n✓ Connected. First campaigns:")
for r in rows:
    print(f"  · {r.campaign.name} ({r.campaign.status.name})")
```

---

## Appendix B — Basic Access answer templates

Fill each `{{placeholder}}` from the interview, then hand the user the ready text.

**Intended use** (Company Info form):
```
Internal automation tool for managing {{our own / our clients'}} Google Ads accounts via our MCC. The tool pulls reporting data through the Google Ads API — search terms, keyword performance, Quality Score, and campaign metrics — and surfaces optimization recommendations in an internal dashboard used to manage ad spend and improve ROAS. Read-heavy use of GoogleAdsService.search across campaign, ad group, keyword, and search-term resources. Limited, user-approved write operations: occasional keyword and negative-keyword-list updates. The tool is internal — no external customers, no third-party access, no resale. Estimated daily API calls: 5,000.
```

**12-field Basic Access form:** 1 ✓ · 2 = Cloud project **number** · 3 = MCC ID ·
4 = contact email · 5 = No · 6 = Company URL · 7 = paragraph below · 8 = upload the PDF ·
9 = Internal users — employees only · 10 = No · 11 = No · 12 = only the campaign types they
use · both bottom checkboxes ✓.

**Field 7 — business model / tool / audience:**
```
{{company_name}} is a {{one line: what the business does}} based in {{country}}. We plan and optimize paid advertising for {{our own business / our clients}}, including {{campaign types}}. We are building an internal tool used only by our own team to pull Google Ads reporting for the accounts under our manager account and to apply user-approved keyword and negative-keyword optimizations. The intended audience is our internal team only — the tool is not offered to or accessed by external parties.
```

---

## Appendix C — Design Doc HTML (render to PDF in Step 3.3)

Save as `design-doc.html`, replace every `{{placeholder}}`, render with the Chrome command
in Step 3.3. Keep the wireframe plain; no AI/third-party mentions.

```html
<meta charset="utf-8">
<style>
  @page{size:A4;margin:22mm 20mm} *{box-sizing:border-box}
  body{font-family:Arial,Helvetica,sans-serif;color:#111;font-size:11.5pt;line-height:1.5;margin:0}
  h1{font-size:18pt;margin:0 0 2pt} .sub{color:#555;font-size:10.5pt;margin:0 0 20pt}
  h2{font-size:12.5pt;margin:18pt 0 4pt;border-bottom:1px solid #ccc;padding-bottom:3pt}
  code{font-family:"Courier New",monospace;font-size:10.5pt;background:#f2f2f2;padding:0 2px}
  .cap{color:#555;font-size:9.5pt;margin-top:4pt;text-align:center} svg{width:100%;height:auto;border:1px solid #ccc}
</style>
<h1>{{company_name}} — Google Ads API Integration</h1>
<p class="sub">Design Documentation · Basic Access Application</p>
<h2>1. Company Name</h2>
<p>{{company_name}} — <code>{{company_url}}</code></p>
<h2>2. Business Model</h2>
<p>{{company_name}} is a {{one line describing the business}} based in {{country}}. We plan,
launch, and optimize paid advertising campaigns for {{our own business / our clients}},
including {{campaign types}}. All accounts are managed through a single Google Ads manager
account (MCC). We advertise only on accounts we or our clients own and have authorized us to
manage.</p>
<h2>3. Tool Access and Use</h2>
<p>The tool is an internal application used only by {{company_name}} employees. It is not
offered to, sold to, or accessed by any external party. Access is limited to a small number
of authenticated internal users. No public sign-up, no reseller access, no third-party
integration.</p>
<h2>4. Tool Design</h2>
<p>The tool authenticates to the Google Ads API with our manager account credentials and reads
reporting data for the accounts linked under our MCC — campaign, ad group, keyword, and
search-term reports — shown in an internal dashboard used to review performance. Write
operations are limited and always initiated manually by a user: adding or pausing keywords and
updating negative keyword lists to apply optimizations the user has reviewed and approved. No
changes are made automatically without an explicit user action.</p>
<h2>5. API Services Called</h2>
<ul>
  <li><code>GoogleAdsService.Search</code> — reporting across campaign, ad group, keyword, and search-term resources.</li>
  <li><code>CustomerService.ListAccessibleCustomers</code> — enumerate accounts under the manager account.</li>
  <li><code>AdGroupCriterionService.Mutate</code> — add or pause keywords (user-initiated).</li>
  <li><code>CampaignCriterionService.Mutate</code> — update negative keyword lists (user-initiated).</li>
</ul>
<h2>6. Tool Mockup</h2>
<svg viewBox="0 0 720 300" xmlns="http://www.w3.org/2000/svg" font-family="Arial">
  <rect x="0" y="0" width="720" height="300" fill="#fff"/>
  <rect x="0" y="0" width="720" height="42" fill="#f4f4f4" stroke="#ccc"/>
  <text x="16" y="27" font-size="14" font-weight="bold" fill="#333">{{company_name}} — Ads Dashboard</text>
  <g stroke="#ccc" fill="#fafafa"><rect x="16" y="60" width="160" height="60"/><rect x="192" y="60" width="160" height="60"/><rect x="368" y="60" width="160" height="60"/><rect x="544" y="60" width="160" height="60"/></g>
  <g font-size="10" fill="#888"><text x="28" y="82">Impressions</text><text x="204" y="82">Clicks</text><text x="380" y="82">Cost</text><text x="556" y="82">Conversions</text></g>
  <g font-size="16" fill="#bbb" font-weight="bold"><text x="28" y="108">— — —</text><text x="204" y="108">— — —</text><text x="380" y="108">— — —</text><text x="556" y="108">— — —</text></g>
  <text x="16" y="150" font-size="12" font-weight="bold" fill="#444">Campaigns</text>
  <rect x="16" y="160" width="688" height="26" fill="#f0f0f0" stroke="#ccc"/>
  <g stroke="#e2e2e2" fill="#fff"><rect x="16" y="186" width="688" height="24"/><rect x="16" y="210" width="688" height="24"/><rect x="16" y="234" width="688" height="24"/></g>
  <g font-size="11" fill="#ccc" font-family="monospace"><text x="28" y="203">████████████  Enabled</text><text x="28" y="227">██████████  Enabled</text><text x="28" y="251">█████████████  Paused</text></g>
</svg>
<p class="cap">Figure 1 — Internal reporting dashboard used by {{company_name}} (wireframe).</p>
```

---

## Appendix D — reusable files (write in Step 9)

**`gads.py`** — a small reporting helper any future session can call instead of writing code.
```python
#!/usr/bin/env python3
"""gads.py — Google Ads reporting helper for this folder.
Reads .env in the same folder. Run with the local venv, e.g.:
    ./venv/bin/python3 gads.py accounts
    ./venv/bin/python3 gads.py campaigns <ACCOUNT_ID> --filter TEXT
    ./venv/bin/python3 gads.py perf <ACCOUNT_ID> --filter TEXT
    ./venv/bin/python3 gads.py search-terms <ACCOUNT_ID>
<ACCOUNT_ID> = a client customer id (digits, no dashes). login-customer-id (the MCC) comes from .env.
"""
import os, sys, argparse
from dotenv import load_dotenv
from google.ads.googleads.client import GoogleAdsClient
from google.ads.googleads.errors import GoogleAdsException

load_dotenv(os.path.join(os.path.dirname(os.path.abspath(__file__)), ".env"))

def client():
    cfg = {k: os.getenv(v) for k, v in {
        "developer_token": "GOOGLE_ADS_DEVELOPER_TOKEN", "client_id": "GOOGLE_ADS_CLIENT_ID",
        "client_secret": "GOOGLE_ADS_CLIENT_SECRET", "refresh_token": "GOOGLE_ADS_REFRESH_TOKEN",
        "login_customer_id": "GOOGLE_ADS_LOGIN_CUSTOMER_ID"}.items()}
    cfg["use_proto_plus"] = True
    return GoogleAdsClient.load_from_dict(cfg)

def egp(m): return f"{m/1e6:,.2f}"

def cmd_accounts(ga, a):
    print("Accessible customers:")
    for r in ga.get_service("CustomerService").list_accessible_customers().resource_names:
        print("  ", r.split("/")[-1])

def cmd_campaigns(ga, a):
    where = f"WHERE campaign.name LIKE '%{a.filter}%'" if a.filter else ""
    q = f"SELECT campaign.id, campaign.name, campaign.status, campaign.advertising_channel_type FROM campaign {where} ORDER BY campaign.id DESC"
    n = 0
    for r in ga.get_service("GoogleAdsService").search(customer_id=a.account, query=q):
        c = r.campaign
        print(f"  {c.id} | {c.advertising_channel_type.name:12} | {c.status.name:8} | {c.name}"); n += 1
    print(f"({n} campaigns)")

def cmd_perf(ga, a):
    conds = [f"segments.date BETWEEN '{a.since}' AND '{a.until}'"]
    if a.filter: conds.append(f"campaign.name LIKE '%{a.filter}%'")
    q = ("SELECT campaign.name, metrics.impressions, metrics.clicks, metrics.cost_micros, "
         "metrics.interactions FROM campaign WHERE " + " AND ".join(conds))
    ti = tc = tk = tv = 0; per = {}
    for r in ga.get_service("GoogleAdsService").search(customer_id=a.account, query=q):
        m = r.metrics; ti += m.impressions; tk += m.clicks; tc += m.cost_micros; tv += m.interactions
        per[r.campaign.name] = per.get(r.campaign.name, 0) + m.impressions
    print(f"=== Performance ({a.since} → {a.until}) ===")
    print(f"  campaigns: {len(per)} | impressions: {ti:,} | views: {tv:,} | clicks: {tk:,}")
    print(f"  cost: {egp(tc)}  (CPM {egp(tc/ti*1000) if ti else '0'})")
    print("  --- top 5 by impressions ---")
    for name, imp in sorted(per.items(), key=lambda x: -x[1])[:5]:
        print(f"    {imp:>9,}  {name[:55]}")

def cmd_search_terms(ga, a):
    q = (f"SELECT search_term_view.search_term, metrics.impressions, metrics.clicks, metrics.cost_micros "
         f"FROM search_term_view WHERE segments.date BETWEEN '{a.since}' AND '{a.until}' "
         "ORDER BY metrics.cost_micros DESC LIMIT 30")
    print("Search terms (top by cost):")
    for r in ga.get_service("GoogleAdsService").search(customer_id=a.account, query=q):
        s = r.search_term_view; m = r.metrics
        print(f"  {egp(m.cost_micros):>10}  imp {m.impressions:>6}  clk {m.clicks:>4}  | {s.search_term}")

def main():
    p = argparse.ArgumentParser(); sub = p.add_subparsers(dest="cmd", required=True)
    sub.add_parser("accounts")
    for name in ("campaigns", "perf", "search-terms"):
        sp = sub.add_parser(name); sp.add_argument("account")
        sp.add_argument("--filter", default=""); sp.add_argument("--since", default="2023-01-01")
        sp.add_argument("--until", default="2026-12-31")
    a = p.parse_args()
    try:
        ga = client()
        {"accounts": cmd_accounts, "campaigns": cmd_campaigns, "perf": cmd_perf,
         "search-terms": cmd_search_terms}[a.cmd](ga, a)
    except GoogleAdsException as e:
        for er in e.failure.errors: print("ERR:", er.message[:200])
        sys.exit(1)

if __name__ == "__main__":
    main()
```

**`CLAUDE.md`** (and an identical `AGENTS.md`) — fill `{{...}}` from the user's real accounts.
```markdown
# Google Ads is connected in THIS folder — read before acting

This folder is wired to **Google Ads** via `.env` (credentials) + `venv` (the `google-ads`
Python library). It is **local code, not a connector.** When the user asks anything about
Google Ads / their campaigns / a specific ad account, **use this setup — do NOT use
Meta/Facebook tools.**

## How to pull data (use the helper, don't re-write code)
    ./venv/bin/python3 gads.py accounts                 # list accessible accounts
    ./venv/bin/python3 gads.py campaigns <ACCOUNT_ID>   # list campaigns (add --filter TEXT)
    ./venv/bin/python3 gads.py perf <ACCOUNT_ID>        # performance summary
    ./venv/bin/python3 gads.py search-terms <ACCOUNT_ID>

## Account map
| Name | Customer ID | Notes |
|---|---|---|
| MCC (login) | `{{mcc_id}}` | always the login-customer-id |
| {{account_name}} | `{{account_id}}` | {{what runs here}} |

## Rules
- Never print or paste `.env`, the refresh token, or any secret. Never commit `.env`.
- The API can create/edit Search / Display / Performance Max / Demand Gen campaigns. It
  **cannot** create/edit Video (YouTube) campaigns — those are UI-only.
- Video view counts come from `metrics.interactions` (campaign-level `metrics.video_views`
  isn't queryable in the current API version).
```
