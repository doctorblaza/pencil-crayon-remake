---
name: "pencil-crayon-remake"
description: "Frame-by-frame anime remake pipeline in pencil-sketch + crayon-coloring style: one locked verbatim style prompt, independent per-frame img2img (no chaining), 12fps production, honest gap handling, independent frame-level QC, faststart delivery. Use when remaking an anime OP/ED/clip into a hand-drawn pencil & crayon look while keeping the original's composition and acting intact."
---

# Pencil-Crayon Anime Remake

Remake an existing anime clip (OP/ED/scene) into pencil-sketch + crayon-coloring style,
frame by frame, with strict fidelity to the source. Proven on 90-second OPs
(1080×1080→1920×1080, ~1080 frames at 12fps).

This is the "B-track" method: fully programmatic, whole-clip stylization.
(For hero-shot hand retouching — the "A-track" — refine individual frames by hand instead.)

## Workflow

### 1. Source Prep
- Use the cleanest source available: no watermarks, no hardcoded subtitles, no karaoke text.
- If the only source has watermarks/subtitles, de-watermark/de-subtitle first and verify
  corners, title cards, and bottom thirds at multiple timestamps before proceeding.
- Extract frames at 12fps: `ffmpeg -i src.mp4 -vf fps=12 frames/f_%04d.png`
- Extract the original audio separately for the final mux.

### 2. Lock One Style Prompt — Verbatim for Every Frame
Write ONE style prompt (see `references/style-prompt-template.md`) and use it
**word-for-word, unchanged, on all frames**. No per-frame tuning, no "improvements"
mid-batch. The prompt must contain:
- The style definition: visible graphite pencil linework, hand-shaded hatching,
  textured crayon fills with paper grain.
- A strict fidelity clause: composition, poses, expressions, proportions, costumes,
  background elements, camera angle, and every visible detail stay identical to the
  source frame. Only the rendering style changes. Nothing added, nothing invented.

Generate 3–5 sample frames first and eyeball them against the source before
launching the full batch.

### 3. Per-Frame Independent img2img — No Chaining (Iron Rule)
Each output frame is transformed **from its own source frame only**.
Never feed frame N's output as the reference for frame N+1.

Chaining accumulates drift: small characters melt, poses warp, faces slide off-model
(a chained run once turned a slight head-tilt into a deep bow with wild hair).
Independent frames keep every frame anchored to the original.

### 4. Parallel Batch Production
- Split the frame range into contiguous chunks across 2–4 workers.
- Naming: `f_0001.png`, `f_0002.png`, … (zero-padded, sortable).
- Each worker records its own refusals/gaps with frame numbers and reasons.

### 5. Gap Policy — Honest, Never Hidden
The image pipeline will refuse some frames (content filters) or return black/bad frames.
- Fill every gap with the nearest good neighbor frame.
- Log every gap: frame number + reason. Report gaps to the user as 缺口, never silently.
- **Never reword-retry a category-blocked frame.** A refusal is a refusal; the
  neighbor-frame fill is the correct action.

### 6. Assembly
- 12fps master: `ffmpeg -framerate 12 -i out/f_%04d.png -i audio.m4a -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags faststart final.mp4`
- If 24fps delivery is needed, duplicate each frame once at assembly time
  (12fps of real AI frames → 24fps output), don't invent in-between frames.
- 1920×1080, H.264 + AAC, `faststart` (moov atom at file head) — required for web/GitHub playback.

### 7. QC — Independent Agent, "Does It Look Right"
A separate QC agent (not the producer) checks the assembled video:
1. **Character consistency**: extract every 0.5s, compare faces against the source frame.
2. **No artifacts**: no foreign objects, no melted details, no extra limbs.
3. **Object/scene logic**: props, costumes, and background anchors consistent across cuts.
4. **Natural expressions**: restrained, on-model — never generic pretty faces.

Verdict per batch: PASS / FAIL with frame numbers. FAIL frames are redone individually
(per §3, from their own source frames), then the whole clip is **re-encoded under a
new filename** — never reuse a filename the user may have cached.

### 8. Delivery Checks
- `ffprobe`: duration, fps, total frame count match expectations.
- Spot-check frames at ~10s / 45s / 80s against the source.
- Confirm `faststart` (moov before mdat).
- Play the full clip once, end to end, before announcing completion.

## Tooling
- Image pipeline `img2img`: per-frame stylization (source frame + locked prompt)
- `muse.exec` + `ffmpeg`/`ffprobe`: extraction, assembly, verification
- Subagents: parallel frame chunks (§4), independent QC agent (§7)

## Operating Rules
1. **One prompt, verbatim, every frame.** No mid-batch edits.
2. **No chaining.** Each frame is independent img2img from its own source frame.
3. **Strict fidelity.** Change the rendering style only; never invent details.
4. **12fps production standard.** Duplicate to 24fps at assembly if needed.
5. **Gaps are logged and neighbor-filled**, never hidden, never reword-retried.
6. **Independent QC** with 0.5s extraction vs source; verdict in writing.
7. **Fix only flagged frames.** When the user names frame numbers, touch exactly those.
8. **New filename on every re-encode.** Cached old versions must never resurface.
9. **faststart + ffprobe** before every delivery. Verify, don't assume.
