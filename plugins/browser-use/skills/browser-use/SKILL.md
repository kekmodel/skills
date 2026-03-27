---
name: browser-use
description: Use when needing to browse the web interactively, automate browser actions, fill forms, click elements, take screenshots, extract page content, or perform any task requiring a real browser. Also use as a fallback when accessing websites or services where official APIs are unavailable, paid, or restricted (e.g. X/Twitter, Instagram, LinkedIn), or when WebFetch fails or returns incomplete results due to authentication walls, anti-bot protections, or JS-rendered content. Triggers on web scraping, browser automation, form submission, page interaction, visual inspection of websites, or accessing authenticated/restricted web services via the user's logged-in Chrome profile.
---

# Browser Use CLI

Browser automation CLI with persistent daemon. Binary at `~/.browser-use-env/bin/browser-use`.

**This skill covers common commands only.** For unfamiliar commands or flags, run `--help`:
```bash
browser-use --help                # all commands
browser-use <command> --help      # command-specific (e.g. browser-use cookies --help)
```

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

```bash
browser-use --session <name> close                 # 0. Clean up any prior session
browser-use --session <name> open <url>            # 1. Navigate
browser-use --session <name> state                 # 2. See page elements (indexed)
browser-use --session <name> click <index>         # 3. Interact by element index
browser-use --session <name> input <index> "text"  # 4. Type into fields
browser-use --session <name> screenshot [path]     # 5. Capture result
browser-use --session <name> close                 # 6. Clean up when done
```

## Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `open <url>` | Navigate to URL | `browser-use open https://example.com` |
| `state` | Get URL, title, clickable elements (indexed) | `browser-use state` |
| `click <index>` | Click element by index | `browser-use click 5` |
| `click <x> <y>` | Click by coordinates | `browser-use click 100 200` |
| `type "text"` | Type text (active element) | `browser-use type "hello"` |
| `input <index> "text"` | Type into specific element | `browser-use input 3 "search query"` |
| `keys <combo>` | Send keyboard keys | `browser-use keys "Enter"` |
| `select <index> <value>` | Select dropdown option | `browser-use select 4 "option1"` |
| `scroll up/down` | Scroll page | `browser-use scroll down` |
| `back` | Go back in history | `browser-use back` |
| `screenshot [path]` | Take screenshot (base64 if no path) | `browser-use screenshot /tmp/page.png` |
| `screenshot --full [path]` | Full page screenshot | `browser-use screenshot --full /tmp/full.png` |
| `extract "query"` | Extract data using LLM | `browser-use extract "list all prices"` |
| `get html/text/value/attr` | Get page content | `browser-use get text` |
| `eval "js code"` | Execute JavaScript | `browser-use eval "document.title"` |
| `hover <index>` | Hover over element | `browser-use hover 3` |
| `dblclick <index>` | Double-click element | `browser-use dblclick 5` |
| `rightclick <index>` | Right-click element | `browser-use rightclick 5` |
| `upload <index> <path>` | Upload file | `browser-use upload 2 ./file.pdf` |
| `wait text "string"` | Wait for text to appear | `browser-use wait text "Success"` |
| `wait element "selector"` | Wait for element | `browser-use wait element "#result"` |
| `switch <tab_index>` | Switch browser tab | `browser-use switch 1` |
| `close-tab` | Close current tab | `browser-use close-tab` |
| `cookies get/set/clear` | Cookie operations | `browser-use cookies get` |
| `python "code"` | Execute Python (persistent vars) | `browser-use python "print(1+1)"` |
| `sessions` | List active sessions | `browser-use sessions` |
| `close` | Close browser & stop daemon | `browser-use close` |

## Global Options

| Flag | Purpose |
|------|---------|
| `--headed` | Show browser window (visible) |
| `--profile [NAME]` | Use real Chrome with profile (default: "Default") |
| `--session NAME` | Named session (parallel browsers) |
| `--json` | JSON output |
| `--cdp-url URL` | Connect to existing browser via CDP |
| `--connect` | Auto-discover running Chrome via CDP |

## Common Patterns

### Search and extract
```bash
browser-use --session search open "https://google.com"
browser-use --session search state                    # find search input index
browser-use --session search input 1 "search query"
browser-use --session search keys "Enter"
browser-use --session search extract "list all search result titles and URLs"
browser-use --session search close
```

### Fill a form
```bash
browser-use --session form open "https://example.com/form"
browser-use --session form state                    # see all form fields
browser-use --session form input 1 "John"          # name field
browser-use --session form input 2 "john@test.com" # email field
browser-use --session form select 3 "option_a"     # dropdown
browser-use --session form click 4                  # submit button
browser-use --session form wait text "Thank you"
browser-use --session form close
```

### Screenshot for visual inspection
```bash
browser-use --session inspect open "https://example.com"
browser-use --session inspect screenshot /tmp/page.png  # then Read the image
browser-use --session inspect close
```

### Use real Chrome profile (logged-in sessions)
```bash
browser-use --profile --session gmail open "https://gmail.com"  # uses Default Chrome profile
browser-use --profile "Work" --session work open "https://app.com"  # specific profile
```

## Tips

- **`--session <name>` is mandatory** — always name your session for tracking and parallel safety
- **Always `close` before `open` and after finishing** — prevents memory leaks from orphaned daemons
- `state` returns indexed elements — use indices for `click`, `input`, `select`
- Screenshots saved to file can be read with the Read tool for visual inspection
- `extract` uses LLM to parse page content — good for unstructured data
- `--profile` reuses real Chrome cookies/sessions (already logged in)
- Daemon persists between commands — no need to re-open browser each time
- Multiple `--session` names allow parallel browsing without conflicts
