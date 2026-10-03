---
name: ai-innovations-digest
description: Step 3 of the weekly AI digests. Compiles noteworthy, primary-sourced LLM and agent-system innovations from the past 7 days across four areas (knowledge & memory management, agentic & multi-agent systems, prompting & system prompts, AI policy & governance), saves the HTML digest to digests/<weekId>/innovations.html, and creates a Gmail draft. Use when asked to run the innovations digest, "LLM innovations digest", or step 3 of the weekly digests.
---

Compile a weekly digest of noteworthy developments in LLM and agent systems across four focus areas: knowledge/memory management, agentic systems, prompting and system-prompt technique, and AI policy and governance. Save it to the repo and as a Gmail draft.

## Setup (cloud environment)
- **Recipient:** `DIGEST_RECIPIENT` from the invoking prompt. If it's missing, ask. Never send email, only create a draft.
- **Dates:** cloud machines run in UTC, so use New York time. Get today with `TZ=America/New_York date '+%Y-%m-%d %a'` (see CLAUDE.md for a python fallback). `weekId` is this week's Friday as `YYYY-MM-DD`.
- **Tools:** use `WebSearch` to find items and `WebFetch` to open primary sources. Bash `curl` may be blocked by the network policy, so use it only as a last resort. Use the claude.ai Gmail connector's `create_draft` for the draft.
- **Previously covered items:** to check whether an item was already covered, look in earlier weeks' `digests/*/innovations.html` files and in `index.html`.

## Search window

Cover the 7-day period ending today (last Friday through this Friday, inclusive). If an item falls outside the window but is genuinely important and hasn't been covered before, include it and label the date explicitly rather than omitting it. Never include anything older than 14 days.

## Sourcing rules (read before searching — these govern what makes the cut)

**Every item must resolve to a primary source.** The link on each item must point to one of:

- a GitHub repository, pull request, release page, or specific commit
- an arXiv abstract page or other direct paper link
- a specification document, RFC, or protocol changelog
- official policy text — a regulation, bill, agency publication, consultation document, or standard
- a specific blog post, engineering writeup, or docs page from the lab, company, or author who did the work

**Never link to a generic landing page** for an AI news site, an aggregator, a "trending" index, or a company homepage. `github.com/trending`, a newsletter archive page, and `openai.com/news` are all disqualifying as the primary link for an item.

**Drop anything you cannot primary-source.** If an item is interesting but you can only reach secondary coverage of it, leave it out. A shorter digest of things the reader can actually open beats a longer one full of dead ends. It is fine for a section to run short or be omitted entirely.

**Newsletters are for discovery only, never for citation.** Newsletters and aggregators may be scanned to find candidates, but every item must then be traced back to its original source, and the newsletter must not appear as the link. If the trail goes cold, drop the item.

A secondary link (HN thread, community discussion) may be added *in addition to* the primary link when it carries the community-signal evidence — but never *instead of* it.

## Sections

Use exactly these four sections, in this order, as H2 headers. Omit any section with no qualifying items rather than padding it.

### 🧠 Knowledge & Memory Management

Emerging approaches to knowledge management and memory management in LLM systems. Knowledge graphs and graph RAG remain in scope but are one approach among several, not the center of gravity. Cover: agent memory architectures (working, episodic, long-term, procedural), context engineering and context compaction, memory consolidation and forgetting, retrieval strategies beyond naive vector search, filesystem- and document-based context stores, structured/ontological knowledge representation, memory persistence and portability across sessions and tools, and evaluation of memory systems.

Look for: arXiv work on memory and context management; repos and design docs from memory/context frameworks (Letta/MemGPT, mem0, Zep/Graphiti, cognee, LlamaIndex, and newer entrants); vendor engineering posts on context management (Anthropic, OpenAI, Google, LangChain); memory-related additions to agent SDKs and their release notes.

### 🤖 Agentic & Multi-Agent Systems

Emerging trends, implementation patterns, security, and governance for agentic systems — **not** a roundup of products and tools. A new framework release only qualifies if the writeup is about the *pattern or problem* it addresses. Cover: architectural patterns for orchestration, delegation, and handoff; failure modes and what long-horizon evaluations reveal; agent security (prompt injection, tool-chain taint, confused-deputy problems, sandboxing, capability confinement, credential and session handling); agent identity, authorization, and permissioning; audit trails, observability, and interpretability of agent runs; human oversight and intervention design; and organizational governance of agents in production.

Look for: arXiv work on multi-agent coordination, agent safety, and agent evaluation; protocol specifications and their changelogs (MCP, agent-to-agent protocols) in their own repos; security research and advisories (OWASP GenAI/LLM guidance, published incident writeups, vulnerability disclosures); post-mortems and evaluation results published by labs and independent evaluators; production-experience writeups from engineering teams.

### 💡 Prompting, System Prompts & Reasoning

Focus on system-prompt technique, research, and approaches. Cover: system prompt design patterns and architecture; how user prompts and system prompts interact, conflict, and are resolved in the model's instruction hierarchy; role assignment and persona techniques and what the evidence says about them; instruction-following and prompt-injection robustness at the prompt layer; writing skills, tools, and instructions *for agents* to consume (skill and tool description authoring, agent-readable documentation); prompt evaluation and regression testing; and reasoning-elicitation techniques with published results.

Look for: arXiv work on system prompts, instruction hierarchy, role prompting, and prompt robustness; published system prompts, model specs, and constitutional/behavioral documents from labs; official prompting guides, cookbooks, and skill-authoring specs; repos of prompts, skills, and agent instructions with real adoption; empirical writeups that test a prompting claim rather than assert it.

### 🏛️ AI Policy & Governance Innovations

Innovations in how AI is governed, worldwide — covering both government and industry, balanced globally rather than centered on any one jurisdiction. Prioritize what is *new in approach*: a novel regulatory mechanism, a first-of-its-kind enforcement action, a new oversight institution, a standard that changes what compliance means in practice. Routine political commentary and speculation about pending legislation do not qualify.

Government sources to check: EU (AI Act implementation, AI Office publications, codes of practice, EUR-Lex texts); US federal (Federal Register notices, NIST AI publications and frameworks, executive actions, agency guidance, bills on congress.gov); US states (enacted AI legislation and its text on state legislature sites); UK (AI Security Institute publications, DSIT policy papers); China (CAC regulations and TC260 standards); and international bodies (OECD.AI, Council of Europe, G7/UN processes, ISO/IEC standards such as 42001).

Industry sources to check: frontier safety and preparedness frameworks and their revisions (Anthropic, OpenAI, Google DeepMind, Meta, and others); model specs and behavioral policy documents; transparency reports, system cards, and safety cases; multi-company bodies (Frontier Model Forum, Partnership on AI, MLCommons); and third-party evaluation and auditing regimes.

Judge these items by consequence and novelty of mechanism, not by community upvotes.

## Search process

1. **Targeted web search per section** — run searches scoped to the window for each of the four focus areas, using the specific concept vocabulary above rather than generic "AI news this week" queries. Use search to *locate* items, then follow through to the primary source.
2. **arXiv** — search recent submissions for each focus area's terms; the abstract page is the link.
3. **GitHub** — where a search surfaces a project, go to the repository itself for the release, changelog, or design document. Use trending or star counts as evidence of traction inside the item text, never as the link.
4. **Official policy and standards sources** — check the government and industry sources listed under the policy section directly; regulatory developments rarely surface through the same channels as technical news.
5. **Practitioner blogs and lab engineering posts** — link the specific post, never the blog's index.
6. **Community discussion** (Hacker News, r/LocalLLaMA, r/MachineLearning, X) — use to gauge traction and to discover candidates. A discussion thread may be cited as a secondary link alongside the primary source.

Aim for **2–3 substantive items per section, 8–12 total**. Depth beats breadth: an item worth including is worth 3–4 sentences explaining the actual mechanism or argument.

## Email format

Produce a well-structured **HTML email** with inline styles.

**Subject line:** `LLM Innovations Digest — Week of [Monday's date]`

**Body:**

- **Intro paragraph** — 2–3 sentences naming the week's dominant theme or the connective tissue between items. Make an argument about the week, don't just announce that items follow.
- **The four sections above**, as H2 headers, in the order given.
- **Each item:**
  - **Bold title, linked to the primary source**
  - 3–4 sentences covering what it is, what's actually new about the approach, and — where relevant — the community signal it has drawn (stars, discussion volume, adoption), with figures stated as point-in-time approximations
  - Any secondary/context link inline in the text, clearly identified
  - An explicit date label on anything outside the coverage window
- **Footer** — brief sourcing notes and caveats: which sources were unreachable, which items were dropped for lack of a primary source, and any figures that are approximations.

Tone: informative and practitioner-focused, for someone building with and closely following LLM technology. Assume the reader knows the fundamentals; spend the words on what's new.

The subject prefix must stay exactly `LLM Innovations Digest`, because the archive skill searches for it.

## Delivery

1. **Save to the repo.** Write the complete HTML email (a full `<html>` document) to `digests/<weekId>/innovations.html`, overwriting any earlier version. Then:
   ```bash
   git add digests && git commit -m "Innovations digest: <weekId>"
   ```
   Then publish using the **Publishing** commands in CLAUDE.md (push to `main`, or fall back to a `claude/…` branch that the repo's Action merges). If publishing fails, report it and carry on.
2. **Gmail draft.** Create a Gmail draft (do not send) addressed to `DIGEST_RECIPIENT`. Note any sourcing limitations at the end of the run summary.
3. End with one line: items per section, file path, push status, draft status.
