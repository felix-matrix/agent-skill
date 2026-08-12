# Search — Context7-First Research with Obscura MCP Fallback

## Rule: Context7 Primary → Obscura MCP Secondary

ALWAYS research with current documentation — never rely on training data. Use
Context7 as the primary source because it's **5-10x more token-efficient** than
browser-based search. Fall back to Obscura MCP ONLY when Context7 doesn't cover
the library/tool or returns insufficient information.

```
Context7 (primary, always first) ──→ Found & sufficient? ──→ Use it
         │
         └── Not found / too sparse / niche tool? ──→ Obscura MCP (web search)
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
| Obscura MCP search | ~1,000-3,000 |
| Reading raw docs pages in full | ~5,000-15,000+ |

## Phase 2: Obscura MCP (FALLBACK ONLY)

Use ONLY when Context7:
- Returns no matching library
- Returns docs that are too sparse or irrelevant
- Doesn't cover the specific tool (niche/new libraries, GitHub repos)
- The question is about community discussions, blog posts, or real-world case studies

### Setup (once)

```bash
# Register the MCP server with Claude Code (stdio transport)
claude mcp add obscura /path/to/obscura mcp

# With stealth mode for anti-detection
claude mcp add obscura /path/to/obscura mcp -- --stealth
```

Or via `~/.claude.json`:

```json
{
  "mcpServers": {
    "obscura": {
      "command": "/path/to/obscura",
      "args": ["mcp", "--stealth"]
    }
  }
}
```

The MCP server keeps a **live browser session** — tools operate on the current
page, so navigate first, then read or act. No Docker, no CDP, no separate CLI.

### Web Search Workflow

```bash
# 1. Navigate to a search engine
browser_navigate({url: "https://duckduckgo.com"})

# 2. Snapshot to find the search box
browser_snapshot()                       # URL, title, text, element refs
browser_interactive_elements()           # Actionable elements + fields

# 3. Type the query and submit
browser_fill({selector: "<ref>", value: "<library> documentation <topic>"})
browser_press_key({key: "Enter"})

# 4. Wait for results, then snapshot
browser_wait_for({text: "<expected result text>"})
browser_snapshot({max_chars: 4000})      # Limit output tokens

# 5. Open the most relevant result
browser_click({selector: "<result-ref>"})

# 6. Extract content
browser_markdown()                       # Page as markdown (token-efficient)
browser_extract({type: "structured"})    # Structured content extraction
```

### Useful MCP Tools

| Category | Tools |
|---|---|
| Navigate | `browser_navigate`, `browser_back`, `browser_forward`, `browser_reload`, `browser_close` |
| Read | `browser_snapshot`, `browser_markdown`, `browser_links`, `browser_extract`, `browser_search` |
| Inspect | `browser_interactive_elements`, `browser_detect_forms`, `browser_get_attribute`, `browser_count` |
| Interact | `browser_click`, `browser_fill`, `browser_fill_form`, `browser_type`, `browser_press_key`, `browser_select_option`, `browser_scroll` |
| Wait / JS | `browser_wait_for`, `browser_wait_for_text`, `browser_evaluate` |
| Visual | `browser_screenshot` (image content block), `browser_pdf` |
| State | `browser_get_cookies`, `browser_set_cookie`, `browser_clear_cookies` |
| Tabs | `browser_tab_new`, `browser_tab_list`, `browser_tab_switch`, `browser_tab_close` |
| Debug | `browser_network_requests`, `browser_console_messages` |

### Token-Efficient Patterns

```bash
browser_snapshot({max_chars: 4000})      # Cap output size
browser_markdown()                       # Clean markdown instead of raw HTML
browser_links()                          # Just the link list when scanning results
browser_search({text: "<keyword>"})      # Find text without full snapshot
browser_get_attribute({selector: "<ref>", attribute: "href"})
```

### Web Search Shortcuts

```bash
browser_navigate({url: "https://github.com/search?q=<library>&type=repositories"})
browser_navigate({url: "https://www.npmjs.com/package/<package>"})
browser_navigate({url: "https://developer.mozilla.org/en-US/search?q=<api>"})
browser_navigate({url: "https://<library>.dev/docs"})
```

## Decision Matrix

| Question Type | Tool |
|---|---|
| "How do I use X?" (X is a known library) | **Context7** |
| "What's the current API for Y v2?" | **Context7** |
| "Is there a library that does Z?" | **Context7** (resolve) → Obscura MCP if no match |
| "What does this niche GitHub repo do?" | **Obscura MCP** (Context7 won't have it) |
| "Show me real-world examples of X in production" | **Obscura MCP** (blogs, case studies) |
| "What's the community saying about A vs B?" | **Obscura MCP** (forums, discussions) |
| "How do I configure this SaaS tool?" | **Context7** first, then Obscura MCP for their docs site |

## Never Do This

- ❌ Answer library questions from training data → always fetch current docs
- ❌ Use Obscura MCP when Context7 has the library → waste of tokens
- ❌ Skip research and assume APIs haven't changed → always verify
- ❌ Combine 4+ concepts in one query-docs call → split them up
- ❌ Launch full Chrome/Chromium → use Obscura (30MB vs 200MB+, ~85ms loads)
- ❌ Read full pages without `max_chars` → always cap snapshot output
- ❌ Forget to close browser sessions → `browser_close`

## Why Obscura MCP (Not Docker + CDP)

| Metric | Obscura MCP | Docker + agent-browser/CDP | Headless Chrome |
|---|---|---|---|
| Per-tool-call overhead | ~1-5 ms (in-process) | ~10-50 ms (CLI + CDP + NAT) | ~50-200 ms |
| Cold start | ~100-300 ms | ~1-3 s (container) | ~2 s |
| Memory | ~30 MB | ~30 MB + Docker + CLI | 200-500+ MB |
| Page load | ~85 ms | ~85 ms | ~500 ms |
| Snapshot tokens | Capped via `max_chars` | ~200-400 (a11y tree) | 2,000-10,000+ |
| Stealth mode | `--stealth` MCP arg | `obscura serve --stealth` | Requires plugins |
| Hops per action | 1 (stdio → in-process) | 4 (CLI → CDP → Docker → browser) | 1 |
