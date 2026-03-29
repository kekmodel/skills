---
name: gemini-generate
description: Use when generating images, videos, or music with Gemini Pro web interface via browser-use CLI. Triggers on "generate image/video/music with Gemini", "제미나이로 만들어줘", "Gemini로 그림/영상/음악", "make me a picture/video/song", or when needing AI-generated media from Google Gemini.
---

# Gemini Generate

Generate images, videos, or music via Gemini Pro web, download originals, and optionally send via Telegram.

## Quick Reference

| Media | Prompt prefix | Generation time | Element | Download |
|-------|--------------|-----------------|---------|----------|
| Image | `Generate an image:` | 30-120s | `img[src*="blob"]` | Expand → click download btn |
| Video | `Generate a video:` | 60-180s | `video` (count increases) | JS fetch → blob → anchor |
| Music | `Generate music:` | 30-90s | `video` (count increases) | JS fetch → blob → anchor |

## Core Workflow

```bash
# 1. Open Gemini (MUST use --profile --headed)
browser-use --profile --headed --session gemini open "https://gemini.google.com"

# 2. Set CDP download path (REQUIRED before any download)
browser-use --session gemini python "
loop = browser._loop
import asyncio
fut = asyncio.run_coroutine_threadsafe(
    browser._session.cdp_client.send_raw('Browser.setDownloadBehavior',
        {'behavior': 'allow', 'downloadPath': '/tmp/gemini_output', 'eventsEnabled': True}),
    loop)
print(fut.result(timeout=10))
"

# 3. Verify Pro & logged in
browser-use --session gemini eval "document.body.innerText.substring(0, 100)"
# Must contain "PRO" and user name. If "로그인" visible → not authenticated.

# 4. Submit prompt
browser-use --session gemini click <rich-textarea-index>
browser-use --session gemini type "<prompt>"
browser-use --session gemini keys "Enter"

# 5. Poll for completion (see Polling section)
# 6. Download (see Download section)
# 7. Close
browser-use --session gemini close
```

## Polling

### Image
```bash
# Poll for blob image appearance
browser-use --session gemini eval "document.querySelectorAll('img[src*=\"blob\"]').length"
# Returns > 0 when ready. May need scroll down first.
```

### Video / Music
```bash
# Track video element count (baseline=0 or previous count)
browser-use --session gemini eval "document.querySelectorAll('video').length"
# When count increases → new media ready
# Also check: "동영상 생성 중" text disappears
```

## Download Methods

### Image (best: download button via CDP)
```bash
# 1. Click image to expand
browser-use --session gemini click <single-image-index>
# 2. Click download button (aria-label contains "다운로드")
browser-use --session gemini click <download-button-index>
# 3. File appears in CDP downloadPath within ~5s
```

### Image fallback / Video / Music (JS fetch → blob → anchor)
**IMPORTANT:** eval doesn't await async results. Use callback pattern + poll for completion.
```bash
# Step 1: Trigger async download
browser-use --session gemini eval "
window.__dlReady = false; window.__dlError = null;
const el = document.querySelectorAll('video');
const target = el[el.length - 1];
fetch(target.src, {credentials: 'include'})
  .then(r => r.blob())
  .then(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'gemini_output.mp4';
    document.body.appendChild(a);
    a.click();
    a.remove();
    window.__dlReady = true;
  })
  .catch(e => { window.__dlError = e.message; window.__dlReady = true; });
'started'
"
# Step 2: Poll for completion (wait 10-15s then check)
browser-use --session gemini eval "window.__dlReady + '|' + window.__dlError"
# Expected: "true|null"
# File appears in CDP downloadPath
```

## Telegram Delivery

```bash
# Images > 10MB: resize first
sips -Z 2048 /tmp/gemini_output/file.png --out /tmp/resized.jpg -s format jpeg -s formatOptions 85

# Send via Telegram MCP
mcp__plugin_telegram_telegram__reply(chat_id, text, files=["/tmp/gemini_output/file.mp4"])
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No `--headed` flag | CDP downloads only work in headed mode |
| No `--profile` flag | Gemini requires Google auth from Chrome profile |
| Forgot CDP setDownloadBehavior | Must set BEFORE clicking download |
| Checking downloads too fast | Wait 5-10s after trigger |
| Image not visible | Scroll down after generation completes |
| Using chunk extraction | Use CDP download instead — 10-60x faster |
| Telegram photo > 10MB | Resize with sips before sending |
| Music element type | Music uses `video` element, not `audio` |
