# AGENTS.md — google-ads-connect (Codex entry point)

> A skill by **Mohamed Hamouda** — https://mhamouda.com
> Digital Marketing Manager · Performance & Growth Marketing Consultant · Marketing Automation & AI Trainer · SaaS Builder

This file makes the `google-ads-connect` skill work with the **OpenAI Codex CLI**.
Codex reads `AGENTS.md` automatically; Claude Code reads `SKILL.md`. Both describe the
same flow — **`SKILL.md` in this folder is the single source of truth.**

## When to activate
When the user asks to *connect Codex/Claude to Google Ads*, *set up the Google Ads API*,
*get a developer token / refresh token*, *fill the Basic Access form*, or *troubleshoot a
Google Ads API connection* → **open `SKILL.md` in this folder and follow it step by step.**

## Non-negotiables (same as SKILL.md)
- You are the guide; the **user** only does browser sign-ins and approvals. You create the
  project files, run the scripts, interview the user, generate their Basic Access answers,
  build their Design Doc PDF, and verify the connection.
- Never ask for passwords or 2-step codes. Keep every secret in `.env`. Create `.gitignore`
  with `.env` in it **before** writing any secret. Never print the refresh token.
- Build everything in the folder the user chooses (SKILL.md Step 0): `.env`, `.gitignore`,
  the two scripts (Appendix A), a Python venv, and `pip install google-ads
  google-auth-oauthlib python-dotenv`.

Follow `SKILL.md` for the full 8-step flow, the Basic Access interview + answer templates
(Appendix B), and the Design Doc PDF template (Appendix C).
