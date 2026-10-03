---
name: run-weekly-digests
description: Runs the whole weekly AI digest pipeline in one session, in order. 1) ai-newsletter-digest, 2) ai-labs-research-digest, 3) ai-innovations-digest, 4) weekly-digest-archive-update. Use when asked to "run all the digests", "run the weekly digests", or to produce this week's digests and update the archive in one go.
---

Run the four weekly digest skills in this exact order, one after another, in this session:

1. `ai-newsletter-digest`: `.claude/skills/ai-newsletter-digest/SKILL.md`
2. `ai-labs-research-digest`: `.claude/skills/ai-labs-research-digest/SKILL.md`
3. `ai-innovations-digest`: `.claude/skills/ai-innovations-digest/SKILL.md`
4. `weekly-digest-archive-update`: `.claude/skills/weekly-digest-archive-update/SKILL.md`

For each step:
- Read its SKILL.md in full and follow it exactly, as if it had been invoked on its own. You can also invoke the skill by name with the Skill tool.
- Pass along `DIGEST_RECIPIENT` from the invoking prompt. If it's missing, ask once at the start and reuse the answer for steps 1–3.
- Finish each step before starting the next, including its commit and push. Step 4 reads the `digests/<weekId>/*.html` files that steps 1–3 push.
- If a step fails, note the reason and continue with the next one. Step 4 already handles a missing digest.
- Compute `weekId` once at the start, in New York time (see CLAUDE.md), and use the same value for every step.

Finish with a short status block, one line per step: ✅ or ⚠️, item counts, push and draft status. End with the live page link: https://acmay3187.github.io/ai-digests/
