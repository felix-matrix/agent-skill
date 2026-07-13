# Context7-First Research for Brainstorming (with agent-browser Fallback)

## Rule: Context7 → agent-browser+Obscura pipeline for ALL brainstorming research

During brainstorming, ALWAYS research with current documentation — never rely on
training data. Use Context7 as the primary source, with agent-browser+Obscura as
the web-search fallback when Context7 doesn't cover the needed library/tool.

## The Research Pipeline

```
Context7 (primary) ──→ Found? ──→ Use docs in brainstorming design
         │
         └── Not found / insufficient? ──→ agent-browser + Obscura (fallback)
                                              │
                                              └── Search web, fetch docs, return results
```

## Phase 1: Context7 (ALWAYS FIRST)

For EVERY library, framework, API, or tool mentioned during brainstorming:

```bash
# Step 1: Resolve the library
resolve-library-id({libraryName: "<name>", query: "<user's question>"})

# Step 2: Fetch docs (one call per concept)
query-docs({libraryId: "<resolved-id>", query: "<specific concept>"})
```

### When to Trigger Context7

| Brainstorming Step | Context7 Usage |
|-------------------|----------------|
| "What library should we use for X?" | Resolve candidates, compare their docs |
| "How does framework Y handle Z?" | query-docs on Y for concept Z |
| "What's the best practice for..." | Fetch current docs, not training data |
| "Is this API still supported?" | Check latest version docs |
| "Show me a code example of..." | query-docs with the specific pattern |
| "What are the tradeoffs between A and B?" | Fetch both libraries' docs, compare |
| Evaluating architecture options | Research each dependency's current API |

### Context7 Best Practices

- **One concept per query** — "routing" and "auth" and "caching" → 3 separate calls
- **Version-aware** — user says "React 19"? Use version-specific ID if available
- **Prefer official** — when multiple matches, pick the official/primary package
- **Prefer high benchmark scores** — better documentation quality
- **Resolve once, query many** — resolve the library ID once, then query per concept

## Phase 2: agent-browser + Obscura (FALLBACK)

Use ONLY when Context7:

- Returns no matching library for the tool/framework
- Returns docs that are too sparse or irrelevant
- Doesn't cover the specific tool (e.g., niche/new libraries)
- The user asks about something not in any package registry

### Fallback Workflow

```bash
# 1. Ensure Obscura is running (start if needed)
obscura serve --port 9222 &

# 2. Connect agent-browser
agent-browser connect 9222

# 3. Search for the docs
agent-browser open https://duckduckgo.com
agent-browser snapshot -i
agent-browser fill @e<search-box> "<library> documentation <topic>"
agent-browser press Enter
agent-browser wait --load networkidle
agent-browser snapshot -i

# 4. Open the most relevant result
agent-browser click @e<result-ref>
agent-browser wait --load networkidle

# 5. Extract the relevant content
agent-browser snapshot -i -c
agent-browser get text @e<content-area>

# 6. If needed, follow links for deeper docs
agent-browser click @e<link-to-api-docs>
```

### Web Search Shortcuts

```bash
# Search GitHub for the library
agent-browser open "https://github.com/search?q=<library>&type=repositories"

# Search official docs site
agent-browser open "https://<library>.dev/docs"

# Search npm/PyPI for package details
agent-browser open "https://www.npmjs.com/package/<package>"

# Search MDN for web APIs
agent-browser open "https://developer.mozilla.org/en-US/search?q=<api>"
```

## Token Budget Awareness

| Approach | Typical Token Cost |
|----------|-------------------|
| Context7 resolve + query (1 concept) | ~200-500 tokens |
| Context7 resolve + 3 queries (3 concepts) | ~600-1,500 tokens |
| agent-browser fallback search | ~1,000-3,000 tokens |
| Reading raw docs pages in full | ~5,000-15,000+ tokens |

**Always exhaust Context7 first** — it's 5-10x more token-efficient than
browser-based research.

## Integration with Brainstorming Checklist

When the brainstorming skill is active, inject research at these points:

1. **Explore project context** — Context7 for existing deps' current APIs
2. **Ask clarifying questions** — Context7 to validate technical feasibility of options
3. **Propose 2-3 approaches** — Context7 for each candidate library/framework's docs
4. **Present design** — Context7 for API references, code examples in the design
5. **Write design doc** — Include Context7 citations (library, version, key APIs used)

## Decision Matrix

| Question Type | Research Tool |
|--------------|---------------|
| "How do I use X?" (X is a known library) | Context7 |
| "What's the current API for Y v2?" | Context7 |
| "Is there a library that does Z?" | Context7 (resolve) → agent-browser if no match |
| "What does this niche GitHub repo do?" | agent-browser (Context7 likely won't have it) |
| "Show me real-world examples of X in production" | agent-browser (blog posts, case studies) |
| "What's the community saying about A vs B?" | agent-browser (forums, discussions) |
| "How do I configure this SaaS tool?" | Context7 first, then agent-browser for their docs site |

## Never Do This

- ❌ Answer library questions from training data → always fetch current docs
- ❌ Use agent-browser when Context7 has the library → waste of tokens
- ❌ Skip research and assume APIs haven't changed → always verify
- ❌ Combine 4+ concepts in one query-docs call → split them up
- ❌ Forget to close browser sessions after fallback research → `agent-browser close --all`
