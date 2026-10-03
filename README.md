# AI Digests

Live archive: **https://acmay3187.github.io/ai-digests/**

Four Claude Code skills build a weekly set of AI digests. Each one runs as a scheduled cloud routine on claude.ai, and any of them can also be started from the Claude app on a phone.

| # | Skill | What it does | Schedule (ET) |
|---|---|---|---|
| 1 | `ai-newsletter-digest` | AI newsletter posts from Gmail → digest draft | Fri 4:00pm |
| 2 | `ai-labs-research-digest` | Research from major AI labs → digest draft | Fri 4:30pm |
| 3 | `ai-innovations-digest` | LLM/agent/policy innovations → digest draft | Fri 5:00pm |
| 4 | `weekly-digest-archive-update` | Adds the week to `index.html`, which redeploys the live page | Fri 10:30pm |
| – | `run-weekly-digests` | Runs 1 → 4 in one session | on demand |

Skills 1–3 save their email HTML to `digests/<weekId>/{newsletters,research,innovations}.html`. Skill 4 builds the archive week from those files and falls back to Gmail when a file is missing.

## Running from your phone
- **Routine:** in the Claude app, open Code → Routines, pick a routine, and tap **Run now**.
- **Ad hoc:** start a new cloud session on this repo and type `/ai-newsletter-digest` (or another skill name). Include `DIGEST_RECIPIENT=<your address>` in the message.
