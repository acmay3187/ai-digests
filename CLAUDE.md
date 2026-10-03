# ai-digests

Weekly AI digests, produced by Claude Code skills that run as cloud routines. The archive page (`index.html`) is served by GitHub Pages at https://acmay3187.github.io/ai-digests/.

## Rules for every session in this repo

- **Timezone.** Cloud machines run in UTC, but the digests use the user's local week (America/New_York). Compute dates with `TZ=America/New_York date ...` and check that the output says EDT or EST. If tzdata is missing, use `python3 -c "from datetime import datetime; from zoneinfo import ZoneInfo; print(datetime.now(ZoneInfo('America/New_York')).strftime('%Y-%m-%d %a %H:%M %Z'))"`.
- **weekId** is the Friday of the current week in New York time, as `YYYY-MM-DD`. A run on Friday, or in the early hours of Saturday UTC, uses that Friday. A run on another day uses the most recent Friday.
- **Email.** Only ever create Gmail **drafts**. Never send. The recipient comes from `DIGEST_RECIPIENT` in the invoking prompt. If it's missing, ask the user. Never write the address into this repo, because the repo is public.
- **Publishing** (every skill uses this after committing):
  ```bash
  git pull --rebase origin main
  git push origin HEAD:main || git push origin "HEAD:claude/digests-$(date -u +%Y%m%d-%H%M%S)"
  ```
  - Routines may not be allowed to push to `main`. In that case the second push sends a `claude/…` branch instead.
  - The repo's GitHub Action (`.github/workflows/publish.yml`) merges any `claude/…` branch into `main` within about a minute, then redeploys the live page. Either path publishes, so report which one was used.
  - If both pushes fail, run the pull and push once more. If they still fail, report it.
- **Skills** live in `.claude/skills/`. Run order: `ai-newsletter-digest` → `ai-labs-research-digest` → `ai-innovations-digest` → `weekly-digest-archive-update`. `run-weekly-digests` runs all four in order.
