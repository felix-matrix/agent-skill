# GitNexus — Token-Efficient Code Exploration

## Rule: Prefer GitNexus for ALL code understanding tasks

GitNexus MCP tools return structured, concise results (~150-500 tokens) vs traditional
approaches (grep/find/reading files) that can consume thousands of tokens. Always
reach for GitNexus FIRST before falling back to raw file reads or grep.

## When GitNexus is REQUIRED (not optional)

| Task | Use This | Instead Of |
|------|----------|------------|
| Understanding how X works | `mcp__gitnexus__query` | grep/find + reading multiple files |
| Finding what calls a function | `mcp__gitnexus__context` | grep for function name |
| Codebase overview | `READ gitnexus://repo/{name}/context` | Reading directory trees |
| What breaks if I change X? | `mcp__gitnexus__impact` | Manual tracing through files |
| Understanding git diff impact | `mcp__gitnexus__detect_changes` | Reading diff + manual analysis |
| Debugging / tracing errors | `mcp__gitnexus__query` + `context` | grep for error strings |
| PR review risk assessment | `mcp__gitnexus__detect_changes` + `impact` | Manual file-by-file review |
| Refactoring / renaming | `mcp__gitnexus__impact` → `rename` | Manual find-and-replace |
| Finding API routes | `mcp__gitnexus__route_map` | grep for route patterns |
| Tracing taint/security flows | `mcp__gitnexus__explain` | Manual data-flow tracing |
| Checking for circular deps | `mcp__gitnexus__check` with `cycles:true` | Manual import tracing |

## Mandatory Workflow

**Before reading ANY source file** to understand code:

```
1. READ gitnexus://repo/{name}/context          → Check index freshness, get overview (~150 tokens)
2. query({search_query: "<what you need>"})      → Find relevant execution flows (~300 tokens)
3. context({name: "<key symbol>"})               → 360° view of specific symbols (~300 tokens)
4. Only THEN Read source files if needed         → Targeted reads of specific files
```

If `gitnexus://repo/{name}/context` reports the index is stale, remind the user to run:
```bash
node .gitnexus/run.cjs analyze
```

## Token Comparison

| Approach | Typical Token Cost |
|----------|-------------------|
| `query()` | ~200-500 tokens |
| `context()` | ~150-400 tokens |
| `READ gitnexus://repo/{name}/context` | ~150 tokens |
| `READ gitnexus://repo/{name}/process/{name}` | ~200 tokens |
| `impact()` | ~300-800 tokens |
| grep + reading 5 files | ~2,000-10,000+ tokens |
| Reading a large file to understand it | ~1,000-5,000+ tokens |

**Bottom line:** GitNexus calls are 5-50x more token-efficient than the equivalent
file-reading approach.

## Never Do This

- ❌ `grep` for a function name then read 10 files → use `context()` then targeted reads
- ❌ Read directory trees to understand structure → use `READ gitnexus://repo/{name}/context`
- ❌ Manually trace call chains through files → use `query()` + `READ .../process/{name}`
- ❌ grep for error messages across the codebase → use `query({search_query: "<error>"})`

## Index Freshness

If GitNexus is not installed or the index is missing:
```bash
npx gitnexus analyze
```
Or if `node .gitnexus/run.cjs` exists:
```bash
node .gitnexus/run.cjs analyze
```
