---
name: weekly-digest-archive-update
description: Step 4 of the weekly AI digests. Adds this week's three digests (newsletters, AI labs research, LLM innovations) as the newest week in the dark-mode archive page index.html, then pushes to main so the live GitHub Pages site (https://acmay3187.github.io/ai-digests/) updates. Reads digests/<weekId>/*.html and falls back to Gmail. Safe to re-run; it replaces the week instead of duplicating it. Use when asked to update the digest archive, refresh the archive page, or run step 4.
---

## Objective

Add a "week" entry to the AI digest archive at `index.html` in the root of this repo. It is served live by GitHub Pages at https://acmay3187.github.io/ai-digests/.

The archive is a sleek dark-mode HTML page that groups three weekly digests by week, newest first. Skills 1–3 (`ai-newsletter-digest`, `ai-labs-research-digest`, `ai-innovations-digest`) ran earlier today. Your job is to take their output and make it the top entry of the archive.

## Setup
- Run `git pull --rebase origin main` first, so you have the latest digests and archive. If skills 1–3 pushed through `claude/…` branches, their files reach `main` about a minute after each push.
- **Dates:** cloud machines run in UTC, and this skill is scheduled for 10:30pm New York time, which is already Saturday in UTC. Always compute dates in New York time with `TZ=America/New_York date` (see CLAUDE.md for a python fallback).

## Source data (in priority order)

For each of the three digest types, `newsletters`, `research` and `innovations`:

1. **Repo file (preferred):** `digests/<weekId>/<type>.html`, written by skills 1–3. It is the full HTML email body.
2. **Gmail fallback:** use it only if the repo file is missing. Use the claude.ai Gmail connector's `search_threads`, and take the most recent matching thread or draft:
   - newsletters: `subject:"Weekly AI Newsletter Digest" newer_than:2d`
   - research: `subject:"AI Labs Research Digest" newer_than:2d`
   - innovations: `subject:"LLM Innovations Digest" newer_than:2d`

   The Gmail connector leaves drafts out of search results unless the query includes `in:draft`. So if the plain query returns nothing, run it again with `in:draft` added, because the digests start life as drafts. Then call `get_thread` with full content to get the body. Use the HTML body if there is one, since it keeps all links and structure. Otherwise use the plaintext body.

If both sources come up empty for a digest, it didn't run this week. Carry on with the others and note the gap (see Step 5).

## Step 1 — Determine the week

Compute the week as Monday through Friday of the current week in New York time:
- `weekEnd` = this week's Friday (today, if today is Friday in New York; otherwise the most recent Friday)
- `weekStart` = `weekEnd` minus 4 days (the Monday)
- `weekId` = `weekEnd` formatted as `YYYY-MM-DD` (used in HTML IDs, `data-week`, and the `digests/` folder name)
- `weekTitle` = e.g. "April 27 – May 1, 2026" (full month names on both ends if the month changes, otherwise abbreviated like "May 4 – 8, 2026")

If today isn't Friday in New York time, still use the most recent Friday, and log the anomaly.

## Step 2 — Read the existing archive

Read `index.html`. It is large (more than 600 KB), so use Grep and Read with offsets instead of loading it all at once. Find:
- The first `<section class="week"` block inside `<main class="main">`. This is the newest existing week and your structural template.
- The first `<li>` inside `<ul class="week-list" id="weekList">` in the sidebar.
- The first occurrence of `data-week-active="true"` and of `class="week-item active"`.

Study the existing structure closely. The new week section must match it exactly: same CSS classes, same nesting, same element types. Don't invent new classes or change the visual conventions.

**Re-run check:** if a `<section class="week" data-week="<weekId>"` already exists, this week was archived before. Replace that section and its sidebar `<li>` (the one with `data-week-target="<weekId>"`) in place. Don't insert a second copy.

## Step 3 — Extract content from each digest

For each digest's HTML body (repo file or Gmail):
- Pull every `<a href="...">` link with its surrounding title and date.
- Pull synopsis paragraphs verbatim.
- Preserve sub-section groupings:
  - **Newsletters:** grouped by publication name (e.g. "The Argument", "Interconnects", "One Useful Thing", "Understanding AI", "WIRED AI Lab"). Each publication is a `<div class="pub-name">PublicationName</div>` chip followed by that publication's items.
  - **Research:** grouped by lab (e.g. "Anthropic", "OpenAI", "Google DeepMind"). Each is an `<h3>LabName</h3>` followed by its papers.
  - **Innovations:** grouped by the four current categories, in this order, each an `<h3>` with an HTML-entity emoji and the title:
    `<h3>&#129504; Knowledge &amp; Memory Management</h3>`, `<h3>&#129302; Agentic &amp; Multi-Agent Systems</h3>`, `<h3>&#128161; Prompting, System Prompts &amp; Reasoning</h3>`, `<h3>&#127963;&#65039; AI Policy &amp; Governance Innovations</h3>`. Omit a category with no items. If the source uses a different category, keep its title and encode its emoji as HTML entities. Never paste raw emoji; copy the entity codes from existing weeks as a reference.
- Preserve any "newsletters with no posts" or "sourcing caveats" notes from the source as a `<p class="note">` at the end of the relevant digest.
- Preserve any intro paragraph as a `<p class="intro-block">` at the top of the digest body.

## Step 4 — Build the new week section

Construct exactly this structure (substitute `{{weekId}}`, `{{weekTitle}}`, content, and counts):

```html
    <section class="week" data-week="{{weekId}}" data-week-active="true" id="week-{{weekId}}">

      <header class="week-header">
        <div class="week-eyebrow">Week of</div>
        <h2 class="week-title">{{weekTitle}}</h2>
        <p class="week-summary">{{a 1-3 sentence overview synthesizing the major themes across all three digests this week}}</p>

        <div class="jump-row">
          <a class="jump-link newsletters" href="#digest-newsletters-{{weekId}}">
            <div class="jump-icon">&#9993;&#65039;</div>
            <div class="jump-content">
              <div class="jump-name">Newsletters</div>
              <div class="jump-meta">{{N}} posts · {{M}} publications</div>
            </div>
            <div class="jump-arrow">↗</div>
          </a>
          <a class="jump-link research" href="#digest-research-{{weekId}}">
            <div class="jump-icon">&#128218;</div>
            <div class="jump-content">
              <div class="jump-name">AI Labs Research</div>
              <div class="jump-meta">{{N}} papers · {{M}} labs</div>
            </div>
            <div class="jump-arrow">↗</div>
          </a>
          <a class="jump-link innovations" href="#digest-innovations-{{weekId}}">
            <div class="jump-icon">&#9889;</div>
            <div class="jump-content">
              <div class="jump-name">LLM Innovations</div>
              <div class="jump-meta">{{N}} items · {{M}} categories</div>
            </div>
            <div class="jump-arrow">↗</div>
          </a>
        </div>
      </header>

      <!-- Newsletter digest -->
      <section class="digest newsletters" id="digest-newsletters-{{weekId}}">
        <header class="digest-header">
          <div class="digest-icon">&#9993;&#65039;</div>
          <div class="digest-titles">
            <h3 class="digest-title">Weekly AI Newsletter Digest</h3>
            <div class="digest-meta">{{M}} publications · {{N}} posts &nbsp;·&nbsp; Friday, {{Mon Day}} · 4:00 PM</div>
          </div>
        </header>
        <div class="digest-body">
          <p class="intro-block">{{intro from email}}</p>
          {{publication chips + items}}
          {{optional .note for "no posts" caveats}}
        </div>
      </section>

      <!-- Research digest -->
      <section class="digest research" id="digest-research-{{weekId}}">
        <header class="digest-header">
          <div class="digest-icon">&#128218;</div>
          <div class="digest-titles">
            <h3 class="digest-title">AI Labs Research Digest</h3>
            <div class="digest-meta">{{labs joined by " · "}} &nbsp;·&nbsp; Friday, {{Mon Day}} · 4:30 PM</div>
          </div>
        </header>
        <div class="digest-body">
          <p class="intro-block">{{intro}}</p>
          {{<h3>Lab</h3> + items per lab}}
          {{optional .note}}
        </div>
      </section>

      <!-- Innovations digest -->
      <section class="digest innovations" id="digest-innovations-{{weekId}}">
        <header class="digest-header">
          <div class="digest-icon">&#9889;</div>
          <div class="digest-titles">
            <h3 class="digest-title">LLM Innovations Digest</h3>
            <div class="digest-meta">Community-resonant LLM &amp; agent systems &nbsp;·&nbsp; Friday, {{Mon Day}} · 5:00 PM</div>
          </div>
        </header>
        <div class="digest-body">
          <p class="intro-block">{{intro}}</p>
          {{<h3>Category</h3> + items per category}}
          {{optional .note for sourcing caveats}}
        </div>
      </section>

    </section>
```

Each item inside a digest body uses this format:

```html
          <div class="item">
            <div class="item-title-line">
              <span class="item-title"><a href="{{url}}" target="_blank" rel="noopener">{{title}}</a></span>
              <span class="item-date">{{Mon Day, Year}}</span>
            </div>
            <p class="item-synopsis">{{synopsis verbatim from email}}</p>
          </div>
```

If an item has no URL, drop the `<a>` and just use the plain title. If an item has an author or sub-attribution (e.g. "— Forrest Chang"), use `<span class="item-author">— {{author}}</span>` after the title. If an item references an external source link separately (like Simon Willison's writeup for Qwen3.6-27B), use `<a class="source-link" href="..." target="_blank" rel="noopener">Source description</a>` after the synopsis.

## Step 5 — Handle missing digests gracefully

If a digest has neither a repo file nor a Gmail result, replace that digest's `<section class="digest [type]">` block with:

```html
      <section class="digest {{type}}" id="digest-{{type}}-{{weekId}}">
        <header class="digest-header">
          <div class="digest-icon">{{icon entity}}</div>
          <div class="digest-titles">
            <h3 class="digest-title">{{Digest name}}</h3>
            <div class="digest-meta">Not produced this week</div>
          </div>
        </header>
        <div class="digest-body">
          <p style="color: var(--text-muted); font-style: italic; padding: 12px 0;">No {{type}} digest was produced this week.</p>
        </div>
      </section>
```

Update that digest's jump-link card meta line to read "Not produced this week".

## Step 6 — Insert into the file

Make these targeted edits to `index.html`. For a re-run, see the re-run check in Step 2.

a) **Sidebar:** inside `<ul class="week-list" id="weekList">`, add a new `<li>` as the first child:
```html
      <li>
        <a class="week-item active" data-week-target="{{weekId}}" href="#week-{{weekId}}">
          <span class="week-item-date">{{weekTitle}}</span>
          <span class="week-item-meta">3 digests · {{N}} newsletters · {{M}} papers</span>
        </a>
      </li>
```
Then remove the `active` class from the anchor in the previous first `<li>` (change `class="week-item active"` to `class="week-item"`).

b) **Main pane:** immediately after `<main class="main">`, before the existing `<section class="week" ...>` block, insert the new week section from Step 4. Then change the previous first week section's `data-week-active="true"` to `data-week-active="false"`, so only the newest week is active by default.

c) Use the Edit tool for these surgical changes. Don't rewrite the whole file. Leave the CSS, the JS and all earlier weeks exactly as they are.

## Step 7 — Verify

Check the edited file with Grep or a short python script:
- The new `<li>` is the first sidebar week, with `class="week-item active"`.
- The new `<section class="week">` is the first one in `<main>`, with `data-week-active="true"`.
- Every other week has `data-week-active="false"`, and no other sidebar item has `active`. Exactly one of each should be active.
- `data-week="{{weekId}}"` appears exactly once.
- Every URL from the source digests appears as an `<a href>` somewhere in the new week section.
- The three digests appear in this order: newsletters → research → innovations.
- The HTML is well formed, with no unclosed tags. Quick check: `grep -c '<section class="digest' index.html` should equal 3 × `grep -c '<section class="week"' index.html`.

## Step 8 — Publish and report

```bash
git add index.html && git commit -m "Archive: week of {{weekTitle}}"
```
Then publish using the **Publishing** commands in CLAUDE.md. If the direct push to `main` is refused, the fallback `claude/…` branch is merged into `main` by the repo's Action. Either way, the live page redeploys within a minute or two.

Finish with one line like:
"Appended week of {{weekTitle}}: {{N}} newsletter posts ({{M}} publications), {{N}} research papers ({{M}} labs), {{N}} innovations ({{M}} categories). Sources: repo/Gmail per digest. Live at https://acmay3187.github.io/ai-digests/"

## Style invariants to never violate

- Dark mode only — never introduce light backgrounds.
- Color coding stays consistent: `newsletters` → green, `research` → blue, `innovations` → purple. Never swap them.
- Order in jump nav and digest sections is always: Newsletters, AI Labs Research, LLM Innovations.
- Digest names in headers and jump cards are exactly: "Weekly AI Newsletter Digest", "AI Labs Research Digest", "LLM Innovations Digest" (in their card title slots) and "Newsletters", "AI Labs Research", "LLM Innovations" (in jump-link `.jump-name` slots).
- Preserve every link from the source emails. Do not fabricate URLs.
- All external `<a>` tags get `target="_blank" rel="noopener"`.