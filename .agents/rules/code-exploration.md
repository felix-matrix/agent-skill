# Code Exploration — Token-Efficient Code Understanding

## Rule: Prefer codebase-memory-mcp FIRST, fall back to GitNexus

Both `codebase-memory-mcp` and `gitnexus` return structured, concise results (~150-500 tokens)
vs traditional approaches (grep/find/reading files) that can consume thousands of tokens.
**Always reach for codebase-memory-mcp FIRST**, then GitNexus for cross-service/repo needs.

## Tool Selection Matrix (Priority-Ordered)

| Task | codebase-memory-mcp (PREFERRED) | GitNexus (fallback) | Instead Of |
|------|-------------------------------|---------------------|------------|
| Finding functions/classes/routes | `search_graph(name_pattern/label/qn_pattern)` | `context()` | grep/find + reading files |
| Understanding how X works | `search_graph(query="...")` → `get_code_snippet()` | `query()` | Reading multiple files |
| Architecture overview | `get_architecture(aspects=[...])` | `READ gitnexus://repo/{name}/context` | Reading directory trees |
| Call chain tracing | `trace_path(func, mode=calls\|data_flow)` | `query()` + process trace | Manual tracing through files |
| What calls a function? | `trace_path(func, direction=inbound)` | `context()` | grep for function name |
| What breaks if I change X? | `impact()` or `trace_path()` | `impact()` | Manual tracing through files |
| Text/code search | `search_code(pattern)` (graph-augmented grep) | — | Raw grep |
| Complex graph queries | `query_graph(cypher)` (Cypher) | `cypher()` | Manual analysis |
| Git diff impact | `detect_changes()` | `detect_changes()` | Reading diff + manual analysis |
| Security/taint analysis | — | `explain()` | Manual data-flow tracing |
| API route mapping | `search_graph(label="Route")` | `route_map()` | grep for route patterns |
| Circular dependency check | `check(cycles:true)` | `check(cycles:true)` | Manual import tracing |
| Cross-service/repo analysis | — | `query()` in group mode | Manual cross-repo grep |
| Symbol source code | `get_code_snippet(qualified_name)` | `READ gitnexus://...` | Reading full files |
| PR review risk assessment | `detect_changes()` | `detect_changes()` + `impact()` | Manual file-by-file review |

---

## PART 1: codebase-memory-mcp (PRIMARY)

**GitHub**: https://github.com/DeusData/codebase-memory-mcp

### Mandatory Workflow

**Before reading ANY source file** to understand code:

```
1. list_projects()                                   → Confirm project is indexed
2. get_architecture(aspects=["overview","clusters"])  → Get high-level structure (~200 tokens)
3. search_graph({query: "<what you need>"})           → Find relevant symbols (~300 tokens)
4. get_code_snippet(qualified_name)                   → Read ONLY the relevant function (~200 tokens)
5. Only THEN Read source files if needed              → Targeted reads of specific files
```

### Key Tools Reference

#### Discovery
| Tool | Use | Example |
|------|-----|---------|
| `search_graph` | BM25 full-text search for symbols | `search_graph({query: "handle auth"})` |
| `search_graph` | Regex name matching | `search_graph({name_pattern: ".*handler.*", label: "Function"})` |
| `search_code` | Graph-augmented grep (text → enriched functions) | `search_code({pattern: "TODO", mode: "compact"})` |
| `query_graph` | Complex Cypher queries | `query_graph({query: "MATCH (f:Function) WHERE f.complexity > 10 RETURN f"})` |

#### Understanding
| Tool | Use | Example |
|------|-----|---------|
| `get_architecture` | High-level project structure, clusters, dependencies | `get_architecture({aspects: ["overview", "clusters", "dependencies"]})` |
| `get_code_snippet` | Read exact symbol source (precise ranges) | `get_code_snippet({qualified_name: "pkg.Service.Method"})` |
| `get_graph_schema` | View node labels and edge types | `get_graph_schema()` |

#### Tracing
| Tool | Use | Example |
|------|-----|---------|
| `trace_path` | Call chains (inbound/outbound) | `trace_path({function_name: "login", direction: "both"})` |
| `trace_path` | Data flow through parameters | `trace_path({function_name: "validate", mode: "data_flow"})` |
| `trace_path` | Cross-service HTTP/async flows | `trace_path({function_name: "createOrder", mode: "cross_service"})` |

#### Change Impact
| Tool | Use | Example |
|------|-----|---------|
| `detect_changes` | Analyze uncommitted diff impact | `detect_changes({scope: "unstaged"})` |
| `detect_changes` | Compare branch impact | `detect_changes({scope: "compare", base_branch: "main"})` |

#### Maintenance
| Tool | Use |
|------|------|
| `index_repository` | Index a new/updated repo |
| `index_status` | Check indexing status |
| `manage_adr` | Create/update Architecture Decision Records |

### Token Comparison

| Approach | Typical Token Cost |
|----------|-------------------|
| `search_graph()` | ~200-400 tokens |
| `get_code_snippet()` | ~100-300 tokens |
| `get_architecture()` | ~150-250 tokens |
| `trace_path()` | ~200-500 tokens |
| `search_code()` | ~300-600 tokens |
| grep + reading 5 files | ~2,000-10,000+ tokens |
| Reading a large file to understand it | ~1,000-5,000+ tokens |

**Bottom line:** codebase-memory-mcp calls are 5-50x more token-efficient than the equivalent
file-reading approach.

### Never Do This

- ❌ `grep` for a function name then read 10 files → use `search_graph()` → `get_code_snippet()`
- ❌ Read directory trees to understand structure → use `get_architecture()`
- ❌ Manually trace call chains through files → use `trace_path()`
- ❌ grep for patterns across the codebase → use `search_code()` (graph-augmented)
- ❌ Read large files to find a function → use `get_code_snippet(qualified_name)`

---

## PART 2: GitNexus (SECONDARY — Cross-Service/Repo Needs)

**GitHub**: https://github.com/abhigyanpatwari/GitNexus

Use GitNexus when:
- You need **cross-service or cross-repository** analysis (group mode)
- You need **API route shape checking** (`api_impact`, `shape_check`)
- You need **security/taint analysis** (`explain`)
- You need **PDG-level program dependence graph** queries (`pdg_query`)
- codebase-memory-mcp doesn't have the specific capability you need

### Key Tools Reference

| Tool | Use |
|------|-----|
| `query` | Natural-language execution flow search |
| `context` | 360° view of a symbol (callers, callees, refs) |
| `impact` | Blast radius of changing a symbol |
| `trace` | Shortest path between two symbols |
| `detect_changes` | Git diff → affected processes |
| `route_map` / `api_impact` / `shape_check` | API route analysis |
| `explain` | Security taint findings |
| `pdg_query` | Program dependence graph (control/data flow) |
| `rename` | Multi-file coordinated rename (graph + text) |
| `check` | Circular dependency detection |

### Group Mode (Cross-Repo)

```bash
# Configure a group in group.yaml first, then:
mcp__gitnexus__query({search_query: "payment flow", repo: "@myGroup"})
```

---

## Index Freshness

### codebase-memory-mcp
```bash
codebase-memory-mcp config set auto_index true
# Or manually:
# Re-index via the tool or CLI
```

### GitNexus
```bash
npx gitnexus analyze
# Or if node .gitnexus/run.cjs exists:
node .gitnexus/run.cjs analyze
```
