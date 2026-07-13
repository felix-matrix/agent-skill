# RTK — Token-Optimized CLI Proxy

## Rule: Route ALL CLI commands through RTK

RTK (Rust Token Killer) is a CLI proxy that strips unnecessary output from dev commands,
saving 12-100% of tokens per command. It is installed and available at `~/.local/bin/rtk`.

## How RTK Works

RTK filters command output to show only what matters, removing noise like:
- ANSI color codes and escape sequences
- Progress spinners and redundant status lines
- Boilerplate compiler/linter output
- Unnecessary verbosity in git, docker, npm, etc.

**The Claude Code hook auto-rewrites commands** — but you should still explicitly
use `rtk` prefix when possible to ensure maximum savings.

## Mandatory Usage

**ALWAYS use `rtk` prefix** for these command categories:

| Category | Commands | Typical Savings |
|----------|----------|----------------|
| Git | `git status`, `git diff`, `git log`, `git show`, `git commit` | 30-60% |
| Docker | `docker ps`, `docker images`, `docker logs` | 70-75% |
| File listing | `ls`, `find`, `tree` | 60-65% |
| Package managers | `npm`, `pnpm`, `yarn`, `pip`, `cargo` | 20-50% |
| Build tools | `go build`, `go test`, `make`, `cmake` | 50-100% |
| System tools | `ps`, `top`, `df`, `du`, `netstat` | 40-80% |
| Grep/ripgrep | `grep`, `rg` | 10-15% |
| Any verbose command | Anything producing >50 lines of output | Variable |

## Correct Usage

```bash
# ✅ CORRECT — explicit rtk prefix
rtk git status
rtk git diff
rtk docker ps
rtk ls -la
rtk npm test
rtk go build ./...

# ✅ CORRECT — using Bash tool (hook auto-rewrites)
# The hook will transparently rewrite: git status → rtk git status

# ❌ WRONG — never use rtk proxy (debugging only, bypasses filter)
rtk proxy <cmd>    # Only for debugging RTK itself
```

## Meta Commands

```bash
rtk gain              # Show token savings analytics for this session
rtk gain --history    # Show command usage history with per-command savings
rtk discover          # Analyze Claude Code history for missed RTK opportunities
rtk --version         # Verify RTK is working (should show: rtk X.Y.Z)
```

## When NOT to Use RTK

- Commands that need raw, unfiltered output for parsing (use `rtk proxy <cmd>` sparingly)
- Interactive commands (RTK is for non-interactive output)
- Commands already producing very short output (<5 lines — overhead not worth it)
- When `rtk` itself is the subject of debugging

## Token Impact

From real-world usage across 627+ commands:
- **3.5% average token savings** across all CLI operations (109K tokens saved)
- **Highest impact**: `go test` (100%), `docker ps` (73%), `ls` (64%), `git commit` (59%)
- **Lowest but still valuable**: `grep` (12%), though it adds up over many calls

## Proactive Behavior

- Before running any CLI command, mentally check: "Would RTK filter this usefully?"
- If the command produces >50 lines of output, RTK is almost certainly beneficial
- Use `rtk gain` at the end of a session to see cumulative savings
