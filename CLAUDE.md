# ai-digests

Weekly AI digests, produced by Claude Code skills that run as cloud routines. The archive page (`index.html`) is served by GitHub Pages at https://acmay3187.github.io/ai-digests/.

## Rules for every session in this repo

- **Timezone.** Cloud machines run in UTC, but the digests use the user's local week (America/New_York). Compute dates with `TZ=America/New_York date ...` and check that the output says EDT or EST. If tzdata is missing, use `python3 -c "from datetime import datetime; from zoneinfo import ZoneInfo; print(datetime.now(ZoneInfo('America/New_York')).strftime('%Y-%m-%d %a %H:%M %Z'))"`.
- **weekId** is the Friday of the current week in New York time, as `YYYY-MM-DD`. A run on Friday, or in the early hours of Saturday UTC, uses that Friday. A run on another day uses the most recent Friday.
- **Email.** Only ever create Gmail **drafts**. Never send. The recipient comes from `DIGEST_RECIPIENT` in the invoking prompt. If it's missing, ask the user. Never write the address into this repo, because the repo is public.
- **Committing.** Commit straight to `main`, then `git pull --rebase origin main && git push origin main`. Retry once on a non-fast-forward. Pushing to `main` redeploys the live page.
- **Skills** live in `.claude/skills/`. Run order: `ai-newsletter-digest` → `ai-labs-research-digest` → `ai-innovations-digest` → `weekly-digest-archive-update`. `run-weekly-digests` runs all four in order.
