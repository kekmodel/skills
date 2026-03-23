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

## Core Workflow

```bash
browser-use open <url>        # 1. Navigate
browser-use state             # 2. See page elements (indexed)
browser-use click <index>     # 3. Interact by element index
browser-use input <index> "text"  # 4. Type into fields
browser-use screenshot [path] # 5. Capture result
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
browser-use open "https://google.com"
browser-use state                    # find search input index
browser-use input 1 "search query"
browser-use keys "Enter"
browser-use extract "list all search result titles and URLs"
```

### Fill a form
```bash
browser-use open "https://example.com/form"
browser-use state                    # see all form fields
browser-use input 1 "John"          # name field
browser-use input 2 "john@test.com" # email field
browser-use select 3 "option_a"     # dropdown
browser-use click 4                  # submit button
browser-use wait text "Thank you"
```

### Screenshot for visual inspection
```bash
browser-use open "https://example.com"
browser-use screenshot /tmp/page.png  # then Read the image
```

### Use real Chrome profile (logged-in sessions)
```bash
browser-use --profile open "https://gmail.com"  # uses Default Chrome profile
browser-use --profile "Work" open "https://app.com"  # specific profile
```

## Tips

- `state` returns indexed elements — use indices for `click`, `input`, `select`
- Screenshots saved to file can be read with the Read tool for visual inspection
- `extract` uses LLM to parse page content — good for unstructured data
- `--profile` reuses real Chrome cookies/sessions (already logged in)
- Daemon persists between commands — no need to re-open browser each time
- Use `close` when done to clean up the daemon
- For parallel browsing, use `--session name` to create separate sessions
