---
name: ai-labs-research-digest
description: Step 2 of the weekly AI digests. Finds notable AI research published in the past 7 days by major labs (Anthropic, OpenAI, Google DeepMind, Google Research, Meta AI, others) using arXiv, Hugging Face Daily Papers, lab feeds and pages, then saves an HTML digest grouped by lab to digests/<weekId>/research.html and creates a Gmail draft. Use when asked to run the research digest, "AI labs research digest", or step 2 of the weekly digests.
---

You are compiling the weekly AI Labs Research Digest.

## Objective
Find every notable AI research publication released in the past 7 days by the major AI labs. Save a well-formatted digest to the repo and as a Gmail draft.

## Setup
- **Recipient:** `DIGEST_RECIPIENT` from the invoking prompt. If it's missing, ask. Never send email, only create a draft.
- **Dates:** cloud machines run in UTC, so use New York time. Get today with `TZ=America/New_York date '+%Y-%m-%d %a'` (see CLAUDE.md for a python fallback).
  - `weekId` is this week's Friday as `YYYY-MM-DD`.
  - The search window is the 7 days ending today.
- **Gmail:** use the claude.ai Gmail connector's `create_draft` tool. Its exact name depends on how the connector is mounted.

## Efficiency principles
- Prefer structured data sources (JSON APIs, RSS/Atom feeds) over scraping rendered HTML pages.
- Never use browser tools.
- Fetch only the cheapest endpoint that answers the question.
- Issue independent fetches in parallel, in a single tool-call message.
- Fall back to HTML pages only when no structured feed exists, and only for labs that got no hits from the structured sources.

## Tooling notes (cloud environment)

- **Use `WebFetch` for every URL.** Bash `curl` may be blocked by the environment's network policy. Try it only as a last resort for a raw response, such as when `WebFetch` summarizes away the XML or JSON you need. If neither can reach a URL, drop that source for this run and continue.
- **Ask `WebFetch` for raw, complete data.** For feeds and APIs, prompt it with something like "Return every entry's title, published date, authors/affiliations, link and abstract verbatim as a list; do not summarize or omit entries." Otherwise it may condense the result.
- **Large responses.** HF Daily Papers at `limit=50` (~102 KB), the OpenAI RSS (~100 KB) and the DeepMind RSS (~64 KB) are big. If a tool saves a large response to a file instead of returning it inline, parse that file with bash and python. Don't re-fetch hoping for a shorter response.
- **Truncated JSON repair.** If raw HF Daily Papers JSON is cut off mid-string, don't retry. Walk the brace depth in python to find the last complete top-level object, then close the array:
  ```python
  depth=0; in_str=False; esc=False; last=0
  for i,c in enumerate(s):
      if esc: esc=False; continue
      if c=='\\': esc=True; continue
      if c=='"': in_str=not in_str; continue
      if in_str: continue
      if c=='{': depth+=1
      elif c=='}':
          depth-=1
          if depth==0: last=i+1
  data = json.loads(s[:last] + ']')
  ```
- **Long arXiv queries.** Keep arXiv as two separate queries (Group A and Group B) rather than one bundled query, which can exceed URL-length limits.

## Data sources (work top-down; stop when you have enough)

### Tier 1 — Structured APIs (always use these, in one parallel batch)

1. **arXiv API.** Run two queries in the same parallel batch:
   - Group A (closed labs): `http://export.arxiv.org/api/query?search_query=abs:Anthropic+OR+abs:%22OpenAI%22+OR+abs:DeepMind&sortBy=submittedDate&sortOrder=descending&max_results=40`
   - Group B (Google + Meta): `http://export.arxiv.org/api/query?search_query=abs:%22Google+Research%22+OR+abs:%22Meta+AI%22+OR+abs:FAIR&sortBy=submittedDate&sortOrder=descending&max_results=40`

   Keep `<entry>` elements whose `<published>` date falls in the last 7 days. Each entry already has the title, abstract, authors, date and a direct link, so no follow-up fetch is needed.

2. **Hugging Face Daily Papers API.** Fetch `https://huggingface.co/api/daily_papers?limit=25` (JSON). Use `limit=25`, not `limit=50`, which is too large and often gets truncated. Keep items whose `publishedAt` falls in the last 7 days. This source often surfaces high-signal papers before they reach the lab blogs.

   **NOTE:** HF author objects don't include affiliations, so HF data alone can't tell you which lab a paper came from. Treat HF as a candidate list. Confirm the lab against the Tier 2/3 feeds, or by reading the linked arXiv abstract.

### Tier 2 — RSS/Atom feeds (only for labs that Tier 1 didn't cover well)

3. **OpenAI:** `https://openai.com/news/rss.xml`. **NOTE:** this feed is mostly product, Codex and customer-story posts, so filter hard.
   - Drop items whose titles contain "Codex", customer or company names ("Sea", "AutoScout24", "NVIDIA", etc.), "Campus Network", "how X uses", "scaling AI", "AI adoption", and similar.
   - If no research-flavored items appear in the window, leave out the OpenAI section rather than padding it with product news.
4. **Google DeepMind blog:** `https://deepmind.google/blog/rss.xml`. The blog often has nothing new in a strict 7-day window. If it's empty, fetch `https://deepmind.google/research/publications/` (HTML) as a Tier 2.5 fallback; its dated publication list tends to be more current than the blog feed.
5. **Google Research blog:** `https://research.google/blog/rss/`. If the response is binary or unreadable, skip it. Most Google Research papers also appear on arXiv and will surface through Tier 1.

### Tier 3 — Raw HTML (last resort, one fetch each)

6. **Anthropic:** `https://www.anthropic.com/research`. There is no known feed, and Anthropic rarely posts to arXiv, so HTML is required. Look for dated items in the publications list. This is the **primary** source for Anthropic; don't expect Tier 1 to find their papers.
7. **Meta AI:** `https://ai.meta.com/results/?content_types%5B0%5D=publication`. Use it if arXiv Group B didn't already catch FAIR/Meta papers. Dated entries are listed near the top.
8. **DeepMind publications:** `https://deepmind.google/research/publications/`. Use it when the DeepMind blog RSS has nothing recent.

### What to skip
- Don't run open-ended web searches by default. Skip Step 4 entirely unless Tiers 1–3 returned fewer than 3 items in total.
- Don't use any browser tools.
- Don't fetch individual paper pages just to confirm a date that's already in the feed.

## Process

### Step 1 — Tier 1 in parallel
In a single message, fetch the **two** arXiv URLs and the Hugging Face Daily Papers URL in parallel (3 fetches). Parse each one.
- For arXiv, sort items into labs using the abstract and author affiliation strings.
- For HF, build a candidate list keyed by arXiv id and date.

### Step 2 — Targeted Tier 2/3 fill-in
1. List which priority labs (Anthropic, OpenAI, DeepMind, Google Research, Meta) got no hits from Tier 1. Anthropic almost always needs Tier 3, so plan on fetching it.
2. Fetch the relevant Tier 2 feeds and Tier 3 pages in **one parallel batch**, with at most 4 fetches.
3. If a Tier 2 feed has nothing, go straight to the Tier 2.5/3 HTML page instead of retrying the feed.

### Step 3 — Compile entries
For each publication, collect:
- title;
- lab or author affiliation;
- publication date;
- a 2–3 sentence plain-language synopsis, written from the arXiv abstract or feed summary (don't fetch the paper);
- a direct link (prefer the arXiv abs URL or the lab page over the HF mirror).

Aim for 5–12 items. Favor breadth across labs and significance. Skip minor product or marketing posts. Remove items that appear in more than one source.

**Date-window flexibility.** The strict window is the last 7 days (last Friday through this Friday). A priority lab may have no items in that window but a research post 1–3 days outside it that hasn't been covered before. In that case, include it and mark the date clearly; a digest representing only one lab is worse than slight calendar drift. Never include items older than 14 days. To check what earlier digests covered, look in the `digests/*/research.html` files from previous weeks.

### Step 4 — Fallback search (only if fewer than 3 items)
Run a single `WebSearch` query like `AI research paper [current month year] site:arxiv.org`. Don't run variant searches.

## Email format
Use HTML with inline styles.

Subject: `AI Labs Research Digest — Week of [Monday's date]`. The prefix must stay exactly `AI Labs Research Digest`, because the archive skill searches for it.

Body:
- Intro: "Here's what the major AI labs published this week."
- Sections grouped by lab in this priority order: Anthropic, OpenAI, Google DeepMind, Google Research, Meta AI, then "Other Labs" for everything else.
- Each section: a header with the lab name, then one entry per paper with a bold linked title, the date in muted text, and a 2–3 sentence synopsis.
- Closing: "Have a great weekend!"
- Signature: "Your Weekly AI Research Digest"

## Delivery
1. **Save to the repo.** Write the complete HTML email (a full `<html>` document) to `digests/<weekId>/research.html`, overwriting any earlier version. Then:
   ```bash
   git add digests && git commit -m "Research digest: <weekId>" && git pull --rebase origin main && git push origin main
   ```
   Retry the pull and push once if the push is rejected. If the push still fails, report it and carry on.
2. **Gmail draft.** Create a Gmail draft to `DIGEST_RECIPIENT` with the subject and HTML body. Do NOT send.

## Success criteria
- Tier 1 (two arXiv queries plus HF Daily Papers) fetched in parallel (3 fetches).
- Tier 2/3 fetched only for labs Tier 1 didn't cover, also in parallel.
- At least 3 research items, grouped by lab, with working links.
- Ideally 6 or fewer HTTP fetches for the whole run, and never more than 8.

## Notes
- Focus on substantive research, not product launches or press releases.
- Write synopses for a technically literate non-specialist.
- If a fetch fails, log it briefly and continue. Retry at most once.
- End with one line: items per lab, file path, push status, draft status.
