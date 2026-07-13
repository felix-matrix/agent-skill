# Agent-Browser + Obscura — Lightweight Browser Automation

## Rule: ALWAYS use agent-browser with Obscura for ANY browser task

agent-browser is a fast Rust CLI for browser automation via CDP. Obscura is a
lightweight Rust headless browser (~30MB memory, ~85ms page loads vs Chrome's
200MB+ and 500ms). Together they replace headless Chrome/Puppeteer/Playwright
for all AI agent browser interactions — lighter, faster, more token-efficient.

## Why This Combo

| Metric | Obscura + agent-browser | Headless Chrome + Puppeteer |
|--------|------------------------|---------------------------|
| Memory | ~30 MB | 200-500+ MB |
| Page load | ~85 ms | ~500 ms |
| Binary size | ~70 MB | 300+ MB |
| Snapshot tokens | ~200-400 tokens (a11y tree) | 2,000-10,000+ tokens (raw HTML) |
| Stealth mode | Built-in anti-fingerprinting | Requires plugins |

## Installation

```bash
# agent-browser (already installed: v0.27.3+)
npm i -g agent-browser && agent-browser install

# Obscura headless browser
cargo install obscura
# Or: git clone https://github.com/h4ckf0r0day/obscura && cd obscura && cargo build --release
```

## Mandatory Startup Workflow

**Before ANY browser automation task**, start the backend then connect:

```bash
# 1. Start Obscura as the CDP backend (keep running in background)
obscura serve --port 9222 &

# 2. Connect agent-browser to Obscura via CDP
agent-browser connect 9222

# 3. Now use agent-browser normally — all commands route through Obscura
```

Or in one line:

```bash
obscura serve --port 9222 & sleep 1 && agent-browser connect 9222
```

## Connection Patterns

```bash
# ✅ PREFERRED — connect once, reuse
agent-browser connect 9222
agent-browser open https://example.com
agent-browser snapshot -i
agent-browser click @e3
agent-browser close

# ✅ Direct CDP flag per command
agent-browser --cdp 9222 snapshot
agent-browser --cdp "ws://127.0.0.1:9222/devtools/browser" open https://example.com

# ✅ Auto-discover (falls back to port probing including 9222)
agent-browser --auto-connect open https://example.com

# ❌ NEVER — launch full Chrome when Obscura is available
# Don't use: google-chrome --remote-debugging-port=9222
# Don't use: agent-browser open (without connecting to obscura first)
```

## The Core Loop (Token-Efficient)

```bash
agent-browser open <url>            # Navigate
agent-browser snapshot -i           # Interactive elements only (~200-400 tokens)
agent-browser click @e3             # Act on element refs
agent-browser snapshot -i           # Re-snapshot after any page change
```

Snapshots use accessibility-tree format with compact `@eN` refs, costing
~200-400 tokens instead of thousands for raw HTML. Always use `-i` flag
(interactive elements only) unless you need full page structure.

## Reading Pages (prefer minimal output)

```bash
agent-browser snapshot -i           # Interactive elements only (DEFAULT)
agent-browser snapshot -i -c        # Compact, skip empty nodes
agent-browser snapshot -i -d 3      # Cap depth at 3 levels
agent-browser snapshot -s "#main"   # Scope to CSS selector
agent-browser get text @e1          # Get text of specific element
agent-browser get title             # Just the page title
agent-browser get url               # Just the current URL
agent-browser get count ".result"   # Count matching elements
```

## Interacting

```bash
agent-browser click @e1             # Click element
agent-browser fill @e2 "hello"      # Clear then type
agent-browser type @e2 " world"     # Type without clearing
agent-browser press Enter           # Key press
agent-browser press Control+a       # Key combo
agent-browser select @e4 "option"   # Dropdown select
agent-browser check @e3             # Check checkbox
agent-browser upload @e5 file.pdf   # File upload
agent-browser scroll down 500       # Scroll
agent-browser hover @e1             # Hover
```

## Screenshots & Media

```bash
agent-browser screenshot output.png
agent-browser screenshot --full output.png   # Full page
agent-browser screenshot --css output.png    # CSS-styled (no a11y marks)
agent-browser pdf output.pdf                 # Save as PDF
```

## Tabs & Sessions

```bash
agent-browser tab                    # List tabs
agent-browser tab --new              # Open new tab
agent-browser tab --switch 1         # Switch to tab index 1
agent-browser close                  # Close current tab
agent-browser close --all            # Close all tabs
```

## Extraction

```bash
agent-browser get text @e1           # Visible text
agent-browser get html @e1           # innerHTML of element
agent-browser get attr @e1 href      # Specific attribute
agent-browser get value @e1          # Input value
agent-browser eval "document.title"  # Arbitrary JS evaluation
```

## Waiting

```bash
agent-browser wait --load networkidle   # Wait for network idle
agent-browser wait --selector ".results" # Wait for element to appear
agent-browser wait --timeout 5000        # Wait 5 seconds
```

## When to Use Specialized Skills

```bash
agent-browser skills get electron       # Electron desktop apps
agent-browser skills get slack          # Slack workspace automation
agent-browser skills get dogfood        # Exploratory testing / QA
agent-browser skills get vercel-sandbox # Vercel Sandbox microVMs
agent-browser skills get agentcore      # AWS Bedrock AgentCore cloud browsers
```

## Obscura Stealth Mode (Anti-Detection)

```bash
obscura serve --port 9222 --stealth     # Anti-fingerprinting + tracker blocking
```

Stealth mode provides: randomized GPU/screen/canvas/audio/battery fingerprints,
blocklist of 3,520 tracker domains, realistic user-agent data. Use this for
scraping or automation on sites with bot detection.

## Never Do This

- ❌ Launch full Chrome/Chromium for browser automation → use Obscura
- ❌ Use Puppeteer or Playwright directly → use agent-browser CLI
- ❌ Read raw HTML with `agent-browser get html` on full page → use `snapshot -i`
- ❌ Forget to `close` browser sessions when done → always clean up
- ❌ Keep re-connecting per command → use `agent-browser connect` once per session
