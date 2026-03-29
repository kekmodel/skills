---
name: browser-use
description: Use when needing to browse the web interactively, automate browser actions, fill forms, click elements, take screenshots, extract page content, or perform any task requiring a real browser. Also use as a fallback when accessing websites or services where official APIs are unavailable, paid, or restricted (e.g. X/Twitter, Instagram, LinkedIn), or when WebFetch fails or returns incomplete results due to authentication walls, anti-bot protections, or JS-rendered content. Triggers on web scraping, browser automation, form submission, page interaction, visual inspection of websites, or accessing authenticated/restricted web services via the user's logged-in Chrome profile.
allowed-tools: Bash(browser-use:*)
---

# Browser Automation with browser-use CLI

The `browser-use` command provides fast, persistent browser automation. A background daemon keeps the browser open across commands, giving ~50ms latency per call.

## Prerequisites

```bash
browser-use doctor    # Verify installation
```

For setup details, see https://github.com/browser-use/browser-use/blob/main/browser_use/skill_cli/README.md

## Session Lifecycle (MUST follow)

> ⚠️ **`--session <name>` is REQUIRED on every command.** Never run browser-use without a session name. This ensures parallel safety, explicit cleanup, and no orphaned processes.

**Before starting any browser task**, clean up the target session:

```bash
browser-use --session <name> close    # 1. Close if exists (prevents memory leak)
```

**After completing a browser task**, always close:

```bash
browser-use --session <name> close    # Always close when done
```

> **Never skip cleanup.** Each `open` without prior `close` spawns a new daemon/browser process that leaks memory. Always close before opening and after finishing.

## Core Workflow

1. **Navigate**: `browser-use --session s open <url>` — starts browser if needed
2. **Inspect**: `browser-use --session s state` — returns clickable elements with indices
3. **Interact**: use indices from state (`browser-use --session s click 5`, `browser-use --session s input 3 "text"`)
4. **Verify**: `browser-use --session s state` or `browser-use --session s screenshot` to confirm
5. **Repeat**: browser stays open between commands
6. **Cleanup**: `browser-use --session s close` when done

## Browser Modes

```bash
browser-use --session s open <url>                         # Default: headless Chromium
browser-use --headed --session s open <url>                # Visible window
browser-use --profile "Default" --session s open <url>      # Real Chrome with Default profile (existing logins/cookies)
browser-use --profile "Profile 1" --session s open <url>   # Real Chrome with named profile
browser-use --connect --session s open <url>               # Auto-discover running Chrome via CDP
browser-use --cdp-url ws://localhost:9222/... --session s open <url>  # Connect via CDP URL
```

`--connect`, `--cdp-url`, and `--profile` are mutually exclusive.

## Commands

```bash
# Navigation
browser-use open <url>                    # Navigate to URL
browser-use back                          # Go back in history
browser-use scroll down                   # Scroll down (--amount N for pixels)
browser-use scroll up                     # Scroll up
browser-use switch <tab>                  # Switch to tab by index
browser-use close-tab [tab]              # Close tab (current if no index)

# Page State — always run state first to get element indices
browser-use state                         # URL, title, clickable elements with indices
browser-use screenshot [path.png]         # Screenshot (base64 if no path, --full for full page)

# Interactions — use indices from state
browser-use click <index>                 # Click element by index
browser-use click <x> <y>                 # Click at pixel coordinates
browser-use type "text"                   # Type into focused element
browser-use input <index> "text"          # Click element, then type
browser-use keys "Enter"                  # Send keyboard keys (also "Control+a", etc.)
browser-use select <index> "option"       # Select dropdown option
browser-use upload <index> <path>         # Upload file to file input
browser-use hover <index>                 # Hover over element
browser-use dblclick <index>              # Double-click element
browser-use rightclick <index>            # Right-click element

# Data Extraction
browser-use eval "js code"                # Execute JavaScript, return result
browser-use get title                     # Page title
browser-use get html [--selector "h1"]    # Page HTML (or scoped to selector)
browser-use get text <index>              # Element text content
browser-use get value <index>             # Input/textarea value
browser-use get attributes <index>        # Element attributes
browser-use get bbox <index>              # Bounding box (x, y, width, height)

# Wait
browser-use wait selector "css"           # Wait for element (--state visible|hidden|attached|detached, --timeout ms)
browser-use wait text "text"              # Wait for text to appear

# Cookies
browser-use cookies get [--url <url>]     # Get cookies (optionally filtered)
browser-use cookies set <name> <value>    # Set cookie (--domain, --secure, --http-only, --same-site, --expires)
browser-use cookies clear [--url <url>]   # Clear cookies
browser-use cookies export <file>         # Export to JSON
browser-use cookies import <file>         # Import from JSON

# Python — persistent session with browser access
browser-use python "code"                 # Execute Python (variables persist across calls)
browser-use python --file script.py       # Run file
browser-use python --vars                 # Show defined variables
browser-use python --reset                # Clear namespace

# Session
browser-use close                         # Close browser and stop daemon
browser-use sessions                      # List active sessions
browser-use close --all                   # Close all sessions
```

The Python `browser` object provides: `browser.url`, `browser.title`, `browser.html`, `browser.goto(url)`, `browser.back()`, `browser.click(index)`, `browser.type(text)`, `browser.input(index, text)`, `browser.keys(keys)`, `browser.upload(index, path)`, `browser.screenshot(path)`, `browser.scroll(direction, amount)`, `browser.wait(seconds)`.

## Cloud API

```bash
browser-use cloud connect                 # Provision cloud browser and connect
browser-use cloud connect --timeout 120 --proxy-country US  # With options
browser-use cloud login <api-key>         # Save API key (or set BROWSER_USE_API_KEY)
browser-use cloud logout                  # Remove API key
browser-use cloud v2 GET /browsers        # REST passthrough (v2 or v3)
browser-use cloud v2 POST /tasks '{"task":"...","url":"..."}'
browser-use cloud v2 poll <task-id>       # Poll task until done
browser-use cloud v2 --help               # Show API endpoints
```

`cloud connect` provisions a cloud browser, connects via CDP, and prints a live URL. `browser-use close` disconnects AND stops the cloud browser.

## Tunnels

```bash
browser-use tunnel <port>                 # Start Cloudflare tunnel (idempotent)
browser-use tunnel list                   # Show active tunnels
browser-use tunnel stop <port>            # Stop tunnel
browser-use tunnel stop --all             # Stop all tunnels
```

## Profile Management

```bash
browser-use profile list                  # List detected browsers and profiles
browser-use profile sync --all            # Sync profiles to cloud
browser-use profile update                # Download/update profile-use binary
```

## Command Chaining

Commands can be chained with `&&`. The browser persists via the daemon, so chaining is safe and efficient.

```bash
browser-use --session s open https://example.com && browser-use --session s state
browser-use --session s input 5 "user@example.com" && browser-use --session s input 6 "password" && browser-use --session s click 7
```

## Common Workflows

### Authenticated Browsing

When a task requires an authenticated site (Gmail, GitHub, internal tools), use Chrome profiles:

```bash
browser-use profile list                           # Check available profiles
browser-use --profile "Default" --session auth open https://github.com  # Already logged in
```

### Connecting to Existing Chrome

```bash
browser-use --connect --session s open https://example.com     # Auto-discovers Chrome's CDP endpoint
```

### Exposing Local Dev Servers

```bash
browser-use tunnel 3000                            # → https://abc.trycloudflare.com
browser-use --session s open https://abc.trycloudflare.com
```

## Global Options

| Option | Description |
|--------|-------------|
| `--headed` | Show browser window |
| `--profile [NAME]` | Use real Chrome (bare `--profile` uses "Default") |
| `--connect` | Auto-discover running Chrome via CDP |
| `--cdp-url <url>` | Connect via CDP URL (`http://` or `ws://`) |
| `--session NAME` | Target a named session (**REQUIRED**) |
| `--json` | Output as JSON |
| `--mcp` | Run as MCP server via stdin/stdout |

## Tips

1. **Always run `state` first** to see available elements and their indices
2. **Use `--headed` for debugging** to see what the browser is doing
3. **Sessions persist** — browser stays open between commands
4. **CLI aliases**: `bu`, `browser`, and `browseruse` all work

## Troubleshooting

- **Browser won't start?** `browser-use --session s close` then `browser-use --headed --session s open <url>`
- **Element not found?** `browser-use --session s scroll down` then `browser-use --session s state`
- **Run diagnostics:** `browser-use doctor`

## Cleanup

```bash
browser-use --session s close             # Close browser session
browser-use tunnel stop --all             # Stop tunnels (if any)
```
