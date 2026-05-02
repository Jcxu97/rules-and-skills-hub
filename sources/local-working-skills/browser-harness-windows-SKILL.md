---
name: browser-harness-windows
description: Windows-specific browser-harness setup and usage patterns. Covers dedicated Chrome auto-launch (Chrome 147+ workaround), 4K/HiDPI coordinate handling, per-site zoom compatibility, and common pitfalls on Windows.
---

# Browser Harness — Windows Dedicated Chrome

## When to use

Any time the agent needs to control a browser on Windows without disturbing the user's active Chrome session.

## Architecture

```
Dedicated Chrome (--user-data-dir + --remote-debugging-port)
    ↓ stderr: "DevTools listening on ws://..."
browser_harness.daemon (parses WS URL at launch)
    ↓ IPC (TCP loopback on Windows)
browser-harness CLI / MCP server
```

## Setup (one-time)

1. Clone `browser-use/browser-harness` to a stable path (e.g. `D:\browser-harness`).
2. `uv tool install -e .` — makes `browser-harness` globally available.
3. Create `.env` in repo root:

```env
BU_CHROME_PROFILE=D:\browser-harness\chrome-profile
BU_CHROME_PORT=9222
```

4. First `browser-harness -c "..."` call auto-launches dedicated Chrome. No manual steps.

## Chrome 147+ Discovery Workaround

Chrome 147 removed the `/json/version` HTTP API for all profiles. The `DevToolsActivePort` file is also not written with `--remote-debugging-port`.

The daemon now captures Chrome's stderr at launch to extract the WebSocket URL directly:

```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/<uuid>
```

Required launch flags:
- `--remote-debugging-port=9222`
- `--remote-allow-origins=*` (or Chrome rejects WS upgrade)
- `--user-data-dir=<path>` (isolates from user's main profile)

## 4K / HiDPI Handling

- Windows 200% scaling → CSS viewport 1920×1080, DPR = 2.
- `click_at_xy(x, y)` takes **CSS pixels** — no DPR conversion needed.
- `capture_screenshot()` returns **device pixels** (double on 4K).
- When estimating coordinates from a screenshot, divide by DPR.
- Prefer `js("el.getBoundingClientRect()")` over visual estimation.
- Use `max_dim=1800` on `capture_screenshot()` to stay under LLM image limits.

## Per-site Zoom

Chrome per-site zoom (125%, 150% etc.) changes `devicePixelRatio` dynamically. CDP `Input.dispatchMouseEvent` still operates in CSS pixels, so clicks remain accurate. No special handling.

## Relative Path Screenshots

On Windows the default terminal cwd is often `C:\Windows\System32` (no write access). The patched `capture_screenshot()` redirects relative paths to `<repo>/screenshots/`.

## Claude Desktop Integration (MCP)

A minimal MCP server at `D:\browser-harness\mcp_server.py` exposes a `browser` tool that shells out to `browser-harness -c <code>`. Register in `claude_desktop_config.json`:

```json
"mcpServers": {
  "browser-harness": {
    "command": "python",
    "args": ["D:\\browser-harness\\mcp_server.py"]
  }
}
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `DevToolsActivePort not found` | Set `BU_CHROME_PROFILE` in `.env` |
| `BU_CDP_URL unreachable 404` | Chrome 147 killed HTTP API; use `BU_CHROME_PROFILE` |
| Screenshot permission denied | Use absolute path or rely on `screenshots/` redirect |
| Clicks miss on zoomed pages | Use DOM `getBoundingClientRect()` not screenshot pixels |
| Chrome won't accept WS | Add `--remote-allow-origins=*` |
| Typed text repeats | Clear input field with JS before `type_text()` |

## Key API Reference

```python
new_tab(url)          # Open URL in new tab (don't clobber user's active tab)
goto_url(url)         # Navigate current tab
click_at_xy(x, y)    # CSS pixel coordinates
type_text(text)       # Type into focused element
press_key("Enter")    # Keyboard input
js("return ...")      # Execute JS, return value
capture_screenshot(path, max_dim=1800)
page_info()           # Quick health check
ensure_real_tab()     # Re-attach if session is stale
```
