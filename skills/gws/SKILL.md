---
name: gws
description: Use when interacting with Google Workspace services — Gmail, Drive, Calendar, Sheets, Docs, Slides, Tasks, Chat, Forms, Meet, People, Keep. Replaces MCP Gmail/Calendar tools. Triggers on email, calendar, spreadsheet, document, file upload, or Google Workspace tasks.
---

# Google Workspace CLI (gws)

One CLI for all Google Workspace. Already authenticated via OAuth. Prefer `gws` over MCP Gmail/Calendar tools.

**This skill covers common commands only.** For unfamiliar commands, flags, or services, run `--help` to discover options:
```bash
gws <service> --help              # service-level help
gws <service> <command> --help    # command-level help (e.g. gws gmail +send --help)
gws schema <service.resource.method>  # full API schema for raw calls
```

## Quick Reference — Helper Commands (+)

| Service | Command | Purpose |
|---------|---------|---------|
| **Gmail** | `gws gmail +triage` | Unread inbox summary |
| | `gws gmail +send --to EMAIL --subject S --body B` | Send email |
| | `gws gmail +read --id ID` | Read message body |
| | `gws gmail +reply --id ID --body B` | Reply to message |
| | `gws gmail +reply-all --id ID --body B` | Reply-all |
| | `gws gmail +forward --id ID --to EMAIL` | Forward message |
| | `gws gmail +watch` | Stream new emails (NDJSON) |
| **Calendar** | `gws calendar +agenda` | Upcoming events |
| | `gws calendar +agenda --today` | Today's events |
| | `gws calendar +agenda --week` | This week's events |
| | `gws calendar +insert --summary S --start T --end T` | Create event |
| **Drive** | `gws drive +upload ./file.pdf` | Upload file |
| | `gws drive +upload ./file.pdf --parent FOLDER_ID` | Upload to folder |
| **Sheets** | `gws sheets +read --spreadsheet ID --range "Sheet1!A1:D10"` | Read cells |
| | `gws sheets +append --spreadsheet ID --values 'a,b,c'` | Append row |
| | `gws sheets +append --spreadsheet ID --json-values '[["a","b"]]'` | Append rows (JSON) |
| **Docs** | `gws docs +write --document ID --text 'Hello'` | Append text |
| **Workflow** | `gws workflow +standup-report` | Today's meetings + open tasks |
| | `gws workflow +meeting-prep` | Next meeting prep |
| | `gws workflow +email-to-task` | Gmail → Tasks |
| | `gws workflow +weekly-digest` | Weekly summary |
| | `gws workflow +file-announce` | Announce file in Chat |

## Raw API Access

For anything beyond helpers, use the raw API pattern:

```bash
gws <service> <resource> <method> [--params JSON] [--json JSON]
```

### Common Raw API Examples

```bash
# List Drive files
gws drive files list --params '{"pageSize": 10}'

# Search Drive
gws drive files list --params '{"q": "name contains '\''report'\''", "pageSize": 5}'

# Get file metadata
gws drive files get --params '{"fileId": "FILE_ID"}'

# Download file
gws drive files get --params '{"fileId": "FILE_ID", "alt": "media"}' --output ./downloaded.pdf

# List Gmail messages
gws gmail users messages list --params '{"userId": "me", "q": "from:someone@example.com"}'

# Create spreadsheet
gws sheets spreadsheets create --json '{"properties": {"title": "My Sheet"}}'

# List calendar events
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-23T00:00:00Z", "maxResults": 10}'

# Create task
gws tasks tasks insert --params '{"tasklist": "@default"}' --json '{"title": "Review PR"}'

# List task lists
gws tasks tasklists list

# Send Chat message
gws chat spaces messages create --params '{"parent": "spaces/SPACE_ID"}' --json '{"text": "Hello"}'
```

## Global Flags

| Flag | Purpose |
|------|---------|
| `--format table/json/yaml/csv` | Output format (default: json) |
| `--page-all` | Auto-paginate (NDJSON output) |
| `--page-limit N` | Max pages (default: 10) |
| `--dry-run` | Validate without sending |
| `--upload PATH` | Upload file (multipart) |
| `--output PATH` | Save binary response to file |

## Gmail Helpers Detail

**+send flags:** `--to`, `--subject`, `--body`, `--cc`, `--bcc`, `--html`, `--from` (alias), `-a/--attach PATH` (repeatable, max 25MB total)

**+triage flags:** `--max N` (default 20), `--query 'gmail query'`, `--labels`, `--format table`

**+read flags:** `--id ID`, `--headers`, `--html`, `--format text/json`

## Calendar Helpers Detail

**+agenda flags:** `--today`, `--tomorrow`, `--week`, `--days N`, `--calendar NAME`, `--timezone TZ`

**+insert flags:** `--summary`, `--start` (ISO 8601), `--end` (ISO 8601), `--location`, `--description`, `--attendee EMAIL` (repeatable), `--meet`

## Schema Discovery

```bash
gws schema drive.files.list          # See all params for any API method
gws schema drive.files.list --resolve-refs  # Expanded schema
```

## Tips

- All output is structured JSON by default — pipe to `jq` for filtering
- Use `--format table` for human-readable output
- Helper commands (+) handle encoding/MIME/pagination automatically
- For bulk operations, use `--page-all` for auto-pagination
- `gws gmail +triage --format table` is the fastest way to check inbox
