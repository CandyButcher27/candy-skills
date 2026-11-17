---
name: scrappy
description: Web and social lookups through the scrappy tools (search, fetch, crawl) without flooding the main context. Every lookup goes to the scout agent, which reads the raw results and returns only a short digest with source URLs. Use when the user says "/scrappy", asks to search, look up, research or check what the web, Reddit, X, HN, GitHub, arXiv, YouTube or another site says, or when a task needs outside facts. Raw mode only when exact URLs, image links or full page text are needed.
---
# Scrappy

Never call `mcp__scrappy__search`, `mcp__scrappy__fetch` or `mcp__scrappy__crawl` from the main thread. Their results are raw page dumps. Send every lookup to an agent.

## Default: digest
Dispatch the `scout` agent (`scrappy:scout` when installed as a plugin). Its brief states:
- the goal, in one sentence;
- the time window ("last 30 days", "any date");
- what a useful answer looks like: how many bullets, and what each must hold (a price, a name, a quote, a date);
- the sources to prefer, when the user named them (`on: "reddit"`, `on: "hn"`, a URL to fetch).

Scout returns at most 10 bullets, each one finding plus its URL, and one line on what it could not get. Pass that on. Do not re-fetch its sources to check it unless a finding decides something the user will act on; then fetch only that one URL, through scout.

Independent questions go to separate scout agents in one message, in parallel.

## Raw mode
Only when the caller needs the material itself: exact URLs, image links (for example design references for pixi), or full page text to process later. Dispatch the `scrappy` agent (`scrappy:scrappy` as a plugin) with an output file path; it writes the full result there and returns the path plus a count. Read only the parts you need from that file.

## Rules
- A blocked page, empty result or login wall is reported as exactly that. No retries around bot checks.
- Paid fallbacks (Firecrawl, Bright Data, Apify) are used only when the free sources fail; scrappy decides this itself. Mention it in the answer when a result came from a paid source.
- Never put the user's name, email, project or other personal details into a query.
