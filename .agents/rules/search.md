# Search — Context7-First Research with agent-browser + Obscura Fallback

## Rule: Context7 Primary → agent-browser+Obscura Secondary

ALWAYS research with current documentation — never rely on training data. Use
Context7 as the primary source because it's **5-10x more token-efficient** than
browser-based search. Fall back to agent-browser + Obscura ONLY when Context7
doesn't cover the library/tool or returns insufficient information.

```
Context7 (primary, always first) ──→ Found & sufficient? ──→ Use it
         │
         └── Not found / too sparse / niche tool? ──→ agent-browser + Obscura
```

## Phase 1: Context7 (ALWAYS FIRST)

For EVERY library, framework, API, CLI tool, or cloud service:

```bash
# Step 1 — Resolve the library ID
resolve-library-id({libraryName: "<name>", query: "<user's goal>"})

# Step 2 — Query docs (one concept per call)
query-docs({libraryId: "<resolved-id>", query: "<specific concept>"})
```

### When Context7 Applies

- "How do I use X?" (X is a known library/framework)
- "What's the current API for Y v2?"
- "Is there a library that does Z?"
- "What's the best practice for..."
- "What are the tradeoffs between A and B?"
- Evaluating architecture options / dependencies
- "Show me a code example of..."

### Context7 Best Practices

- **One concept per query** — split "routing + auth + caching" into 3 calls
- **Resolve once, query many** — resolve the library ID once, then query per concept
- **Prefer official** — when multiples match, pick the primary/official package
- **Prefer high benchmark scores** — better documentation quality
- **Version-aware** — user specifies a version? Use it if available

### Token Cost

| Approach | Typical Tokens |
|---|---|
| Context7 resolve + 1 query | ~200-500 |
| Context7 resolve + 3 queries | ~600-1,500 |
| agent-browser fallback search | ~1,000-3,000 |
| Reading raw docs pages in full | ~5,000-15,000+ |

## Phase 2: agent-browser + Obscura (FALLBACK ONLY)

Use ONLY when Context7:
- Returns no matching library
- Returns docs that are too sparse or irrelevant
- Doesn't cover the specific tool (niche/new libraries, GitHub repos)
- The question is about community discussions, blog posts, or real-world case studies

### Startup (once per session)

```bash
obscura serve --port 9222 &       # Start CDP backend (Docker: docker run -d --name obscura -p 9222:9222 h4ckf0r0day/obscura)
agent-browser connect 9222        # Connect agent-browser
```

For anti-detection on protected sites:

```bash
obscura serve --port 9222 --stealth
```

### Fallback Search Workflow

```bash
# 1. Search
agent-browser open https://duckduckgo.com
agent-browser snapshot -i
agent-browser fill @e<search-box> "<library> documentation <topic>"
agent-browser press Enter
agent-browser wait --load networkidle
agent-browser snapshot -i

# 2. Open most relevant result
agent-browser click @e<result-ref>
agent-browser wait --load networkidle

# 3. Extract content (token-efficient)
agent-browser snapshot -i -c        # Compact interactive snapshot
agent-browser get text @e<content>  # Targeted text extraction
```

### Web Search Shortcuts

```bash
agent-browser open "https://github.com/search?q=<library>&type=repositories"
agent-browser open "https://www.npmjs.com/package/<package>"
agent-browser open "https://developer.mozilla.org/en-US/search?q=<api>"
agent-browser open "https://<library>.dev/docs"
```

### Token-Efficient Snapshot Patterns

```bash
agent-browser snapshot -i           # Interactive elements only (~200-400 tokens)
agent-browser snapshot -i -c        # Compact, skip empty nodes
agent-browser snapshot -i -d 3      # Cap depth at 3 levels
agent-browser snapshot -s "#main"   # Scope to CSS selector
agent-browser get text @e1          # Just the text of one element
agent-browser get title             # Just the page title
```

## Decision Matrix

| Question Type | Tool |
|---|---|
| "How do I use X?" (X is a known library) | **Context7** |
| "What's the current API for Y v2?" | **Context7** |
| "Is there a library that does Z?" | **Context7** (resolve) → agent-browser if no match |
| "What does this niche GitHub repo do?" | **agent-browser** (Context7 won't have it) |
| "Show me real-world examples of X in production" | **agent-browser** (blogs, case studies) |
| "What's the community saying about A vs B?" | **agent-browser** (forums, discussions) |
| "How do I configure this SaaS tool?" | **Context7** first, then agent-browser for their docs site |

## Never Do This

- ❌ Answer library questions from training data → always fetch current docs
- ❌ Use agent-browser when Context7 has the library → waste of tokens
- ❌ Skip research and assume APIs haven't changed → always verify
- ❌ Combine 4+ concepts in one query-docs call → split them up
- ❌ Launch full Chrome/Chromium → use Obscura (30MB vs 200MB+)
- ❌ Read raw HTML with `agent-browser get html` on full page → use `snapshot -i`
- ❌ Keep re-connecting agent-browser per command → connect once per session
- ❌ Forget to close browser sessions → `agent-browser close --all`

## Why agent-browser + Obscura (Not Puppeteer/Chrome)

| Metric | Obscura + agent-browser | Headless Chrome + Puppeteer |
|---|---|---|
| Memory | ~30 MB | 200-500+ MB |
| Page load | ~85 ms | ~500 ms |
| Snapshot tokens | ~200-400 (a11y tree) | 2,000-10,000+ (raw HTML) |
| Stealth mode | Built-in (3,520 trackers blocked) | Requires plugins |
