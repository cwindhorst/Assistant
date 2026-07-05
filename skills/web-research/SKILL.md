---
name: web-research
description: >
  Playbook for conducting targeted web research on behalf of the user — looking up facts,
  comparing options, summarising documentation, or answering questions that require current
  information the model may not have baked in. Use this skill whenever the user asks "look
  this up", "research X", "what's the latest on…", or otherwise needs information retrieved
  from the web rather than recalled from memory.
version: 1.0.0
metadata:
  hermes:
    tags: [web, search, research]
    category: assistant
    requires_toolsets: [web_search]
---

# Web Research

## When to Use
- User asks to look something up or research a topic ("what's the current Grok model id?",
  "research home-lab NAS options under $500").
- User needs information that changes over time (prices, docs versions, news, release notes).
- User wants a summary or comparison of several sources rather than a raw link dump.
- User asks "what's the latest on X?" or "is X still the recommended way to do Y?"

## Procedure

1. **Clarify scope before searching** if the request is broad or ambiguous — one targeted
   search beats three vague ones. Ask at most one clarifying question; don't interrogate.
2. **Search with specific queries**: use the `web_search` tool with precise, keyword-rich
   queries. Prefer date-bounded queries (append the current year) when recency matters.
3. **Read primary sources**: if a search result points to official docs, a changelog, or an
   authoritative page, fetch that page for the relevant section rather than relying solely
   on the search snippet.
4. **Synthesise, don't just paste**: distil findings into a concise answer. Lead with the
   direct answer or recommendation, follow with supporting detail and source citations.
5. **Cite sources inline**: include at least one URL per key claim so the user can verify
   and read further. Format as `[description](url)`.
6. **Flag staleness**: note when a source's last-updated date is more than a few months old
   and the topic is fast-moving, so the user knows to double-check before acting.
7. **Offer to go deeper**: after the summary, ask "Want me to dig into any of these further?"
   rather than guessing how much detail is needed.

## Good defaults for this project
- When researching xAI/Grok model IDs or Hermes Agent features, prefer the official
  [xAI docs](https://docs.x.ai) and
  [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs) over third-party summaries.
- Keep research summaries under ~300 words unless the user explicitly asks for depth.
- Do not store raw search results in `MEMORY.md` — store only the distilled fact or
  recommendation, with a source URL, so memory stays compact and accurate.

## Pitfalls
- Don't treat search snippets as ground truth — always verify against the linked page for
  anything consequential (config values, API endpoints, security guidance).
- Don't return a wall of links — curate to the two or three most relevant sources.
- Don't skip the synthesis step; a list of URLs with no explanation forces the user to do
  the research themselves.
- Don't hallucinate citations — only link to URLs you have actually fetched and verified
  exist.

## Verification
- The answer directly addresses what the user asked.
- Every key claim has a source URL that opens to a page supporting the claim.
- The summary is concise enough to read in under a minute for a typical request.
