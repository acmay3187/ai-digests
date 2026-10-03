---
name: ai-newsletter-digest
description: Step 1 of the weekly AI digests. Compiles the AI newsletter posts Alex received in Gmail over the past 7 days into an HTML digest grouped by newsletter (bold titled+dated posts, oldest first, with web-view links), saves it to digests/<weekId>/newsletters.html, and creates a Gmail draft. Use when asked to run the newsletter digest, "AI newsletter digest", or step 1 of the weekly digests.
---

Compile a weekly digest of the AI newsletters Alex received in Gmail over the past 7 days. Save it to the repo and as a Gmail draft he can send to himself.

## Setup
- **Recipient:** `DIGEST_RECIPIENT` from the invoking prompt. If it's missing, ask. Never send email, only create a draft.
- **Dates:** cloud machines run in UTC, so use New York time. Get today with `TZ=America/New_York date '+%Y-%m-%d %a'` (see CLAUDE.md for a python fallback).
  - `weekId` is this week's Friday as `YYYY-MM-DD`.
  - "7 days ago" for the Gmail filter is also computed in New York time.
- **Gmail:** use the claude.ai Gmail connector's tools (`search_threads`, `get_thread`, `create_draft`). Their exact names depend on how the connector is mounted, for example `mcp__Gmail__search_threads`.

## Steps
1. Search Gmail for all AI-related newsletters received in the past 7 days. Use `search_threads` with a date filter like `after:YYYY/MM/DD` for 7 days ago. These AI newsletters *must* be searched for, and included if they posted in the last 7 days:

* Understanding AI (sender understandingai@substack.com; canonical URLs on understandingai.org)
* AI Snake Oil (sender aisnakeoil@substack.com; canonical URLs on aisnakeoil.com)
* AI as Normal Technology (canonical URLs on normaltech.ai). IMPORTANT: Arvind Narayanan runs both AI Snake Oil and AI as Normal Technology, and AI as Normal Technology posts are often sent through the AI Snake Oil address (aisnakeoil@substack.com). Do NOT assume an email from that address is an AI Snake Oil post. Always check the post's canonical "View this post on the web" URL. If it's on normaltech.ai, the post belongs to AI as Normal Technology. If it's on aisnakeoil.com, it belongs to AI Snake Oil. Treat these as two separate newsletters so neither is missed or mislabeled.
* Rising Tide
* Dean Ball (Hyperdimensional)
* Jasmine Sun
* Interconnects/Nathan Lambert (sometimes sent from "robotic@substack.com", which resolves to Interconnects (Nathan Lambert))
* One Useful Thing
* Will Knight (WIRED)
* Alisar Mustafa (The AI Policy Newsletter)
* Nathan Witkin (Arachne)

Also include AI-related posts from similar newsletters Alex subscribes to.

2. For each newsletter, read the messages (`get_thread`, full content) and extract each post or article inside:
   - its title;
   - its publication date;
   - a 2–4 sentence synopsis;
   - the link to that post's web view. This is the canonical URL on the publisher's site, such as the Substack post URL, not the email itself.

   Decide which newsletter a post belongs to by the domain of its canonical URL, not just the Gmail sender address (see the AI Snake Oil / AI as Normal Technology note above). The canonical URL appears near the top of the plaintext body as "View this post on the web at <URL>".
3. Compose the digest as an HTML email grouped by newsletter, with inline styles. Formatting requirements:
   - Start with a short intro paragraph (1–2 sentences on the week's themes).
   - Put each newsletter's posts under its own header, such as "Understanding AI" as an `<h2>`.
   - Within each newsletter's section, list posts in chronological order, oldest first.
   - Bold each post's title and include its publication date, e.g. `<strong>Post Title — Apr 8, 2026</strong>`.
   - Follow the title with the 2–4 sentence synopsis.
   - Put a "Read full post" link (the web view URL) under each synopsis, so Alex can click through if the synopsis interests him.
   - Order the newsletters sensibly: alphabetically, or by number of posts.
   - End with a short note listing any must-include newsletters that had no posts this week.
4. **Save to the repo.** Write the complete HTML email (a full `<html>` document) to `digests/<weekId>/newsletters.html`, overwriting any earlier version. Then:
   ```bash
   git add digests && git commit -m "Newsletter digest: <weekId>"
   ```
   Then publish using the **Publishing** commands in CLAUDE.md (push to `main`, or fall back to a `claude/…` branch that the repo's Action merges). If publishing fails, report it and carry on.
5. **Gmail draft.** Create a Gmail draft (do not send) addressed to `DIGEST_RECIPIENT`. Use the subject `Weekly AI Newsletter Digest — <week ending date, e.g. Oct 2, 2026>` and the HTML body. The subject prefix must stay exactly `Weekly AI Newsletter Digest`, because the archive skill searches for it.
   **Re-runs:** before creating the draft, use the Gmail connector's `list_drafts` to look for an existing draft with the same subject. If one exists, update it with `update_draft` instead of creating a second one.
6. If no AI newsletters arrived in the past 7 days, save and draft a short note saying so instead of an empty digest. Use the same file path and subject.

## Report
End with one line: the number of posts and newsletters, the file path, whether the push succeeded, and whether the draft was created.
