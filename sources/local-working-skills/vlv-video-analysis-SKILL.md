---
name: vlv
description: Extract and analyze video content (Bilibili/YouTube/Douyin) from a URL — fetches subtitles, danmaku, and optionally frames, then Claude analyzes the transcript directly in conversation.
---

# VLV — Video Listen View (In-Conversation)

Invoke with `/vlv <url>` or `/vlv <url> --vision` for frame-level analysis.

## Prerequisites

- Project location: `C:\Users\82077\Desktop\STLA\bilibili-transcript-oneclick-vision`
- Embedded Python: `python_embed\python.exe`
- B站 login state must exist (`browser_state.json`) for Bilibili URLs; if missing, inform the user to run `vlv login` via the GUI first.

## Workflow

### Step 1: Parse arguments

From the user's message, extract:
- `url`: the video URL (Bilibili, YouTube, Douyin, or any yt-dlp supported site)
- `--vision` flag: if present, download video and run frame extraction + OCR
- `--asr`: if present, force local Whisper transcription even if subtitles exist

### Step 2: Run extraction

Execute the pipeline via PowerShell:

```powershell
$proj = "C:\Users\82077\Desktop\STLA\bilibili-transcript-oneclick-vision"
$py = "$proj\python_embed\python.exe"
$env:PYTHONPATH = "$proj\src"
& $py -m bilibili_vision.bilibili_pipeline extract --no-analyze --no-download-video --no-playlist "<URL>"
```

Flags:
- Always use `--no-analyze` (Claude does the analysis, not an external LLM)
- Use `--no-download-video` unless `--vision` was requested
- Always use `--no-playlist` (single video only)
- Add `--asr-force` if user specified `--asr`

Timeout: 120 seconds for text-only, 300 seconds if downloading video.

### Step 3: Locate output

After the command completes, find the newest session directory:

```powershell
$latest = Get-ChildItem "$proj\out" -Directory -Recurse -Depth 1 | Sort-Object LastWriteTime -Descending | Select-Object -First 1
```

The key file is `transcript_merged.txt` inside that directory.

### Step 4: Read transcript

Use the Read tool to read `transcript_merged.txt` from the session directory. If it's very large (>2000 lines), read in chunks.

### Step 5: Analyze and respond

Provide the user with a structured analysis of the video content:

1. **一句话总结** — what the video is about
2. **核心内容** — key points, arguments, or information presented (bulleted)
3. **观众反应** — what the danmaku/comments reveal about audience sentiment (if danmaku present)
4. **亮点/金句** — notable quotes or moments
5. **个人评价** — brief quality/usefulness assessment

Adapt the depth and structure based on video type:
- Tutorial/educational → focus on actionable takeaways
- News/commentary → focus on claims and evidence
- Entertainment → focus on highlights and audience reaction
- Technical → focus on architecture/implementation details

### Step 6 (Optional): Vision analysis

If `--vision` was requested:
1. Run the vision pipeline after extraction:
   ```powershell
   & $py -m bilibili_vision.vision_deep_pipeline "$latest"
   ```
2. Read `video_analysis_deep.json` or frame captions
3. Incorporate visual context into the analysis

## Error Handling

- If extraction fails with login error → tell user to run the GUI once to refresh B站 login
- If no subtitles found and no `--asr` flag → suggest re-running with `--asr`
- If timeout → the video might be very long; suggest trying again or using the GUI

## Example Usage

User: `/vlv https://www.bilibili.com/video/BV1xx411c7mD`
→ Extract subs + danmaku → Read transcript → Provide structured analysis

User: `/vlv https://youtu.be/dQw4w9WgXcQ --vision`
→ Extract subs + download video + frame analysis → Rich multimodal summary
