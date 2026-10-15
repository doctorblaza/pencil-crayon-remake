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
  Always hunt for a clean (e.g. no-credit) source first.
- If only dirty sources exist, say so upfront, present options, and proceed only with
  the user's go-ahead. Do NOT run a separate de-watermark pass — it doesn't survive QC.
  Removal is folded into the redraw itself (§3): overlays are simply never drawn.
  After production, residue-check corners, title cards, and bottom thirds at multiple
  timestamps (see `references/rework-experience.md`).
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
- **All state lives on disk** (batch log + frame files). If a worker dies mid-run,
  the next one resumes from the on-disk state — never restart from zero.

### 5. Gap Policy — Honest, Never Hidden
The image pipeline will refuse some frames (content filters) or return black/bad frames.
- Fill every gap with the nearest good neighbor frame.
- Log every gap: frame number + reason. Report gaps to the user as 缺口, never silently.
  Always report the **longest continuous static stretch** too, not just the total
  count (one production hid a 42-frame ≈ 3.5s freeze inside a "13% gaps" total).
- **Default: never reword-retry a category-blocked frame.** A refusal is a refusal;
  the neighbor-frame fill is the correct action.
- **Exception — the three-step siege (only on the user's explicit order, only for
  user-named segments):** ① retry with the original prompt, up to 2 attempts;
  ② retry with a slightly adjusted prompt, up to 2 attempts; ③ generate the frame
  from its neighbors' transition. Disclose every step-③ deviation from the source
  (wrong facing, shifted palette, missing elements) — a recovered frame is not
  automatically a faithful frame. Frames that still fail stay neighbor-filled.

### 6. Assembly
- 12fps master: `ffmpeg -framerate 12 -i out/f_%04d.png -i audio.m4a -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags faststart final.mp4`
- If 24fps delivery is needed, duplicate each frame once at assembly time
  (12fps of real AI frames → 24fps output), don't invent in-between frames.
- 1920×1080, H.264 + AAC, `faststart` (moov atom at file head) — required for web/GitHub playback.
- **Aspect ratio & letterbox (iron rule, applies to reworks too):** one aspect ratio
  for the whole clip. If the source has letterbox bars, they are fixed-size,
  pure-black bars from first frame to last — never resize them mid-clip, never give
  them crayon texture. Verify by measuring bar height on sampled frames
  (a passing production held 137px/137px constant, content unscaled).

### 7. QC — Independent Agent, "Does It Look Right"
A separate QC agent (not the producer) checks the assembled video:
1. **Character consistency**: extract every 0.5s, compare faces against the source frame.
2. **No artifacts**: no foreign objects, no melted details, no extra limbs.
3. **Object/scene logic**: props, costumes, and background anchors consistent across cuts.
4. **Natural expressions**: restrained, on-model — never generic pretty faces.
5. **Title cards by continuity, at full size**: check every frame of a title's
   appearance — never from thumbnail contact sheets (a thumbnail audit once
   reported 3 bad frames where a full-frame quantitative audit found 22).
6. **Skin-tone & brightness consistency**: within a shot, flag frames whose skin
   tones are outliers (too pale / too dark) against their neighbors.
7. **Subject scale consistency**: within a continuous shot, the subject's size
   must not breathe (one 6-frame run drifted ~6.5%).

Verdict per batch: PASS / FAIL with frame numbers. FAIL frames are redone individually
(per §3, from their own source frames), then the whole clip is **re-encoded under a
new filename** — never reuse a filename the user may have cached.

### 7b. Targeted Frame Fixes (user-named frames)
- Confirm the frame-sheet label ↔ production-file mapping by pixel comparison
  before touching anything: some productions label sheets 0-based while files
  are 1-based — an off-by-one here fixes the wrong frames.
- Generate 4–5 candidates per frame. Reject any candidate that violates ANY
  locked attribute (hair color, costume color, composition), even if the named
  defect is fixed. Ship only the candidate matching its neighbors on everything.
- If you spot a worse defect on a neighboring frame while fixing, report it and
  wait for the user's call — do not expand the scope on your own.
- Skin-tone fixes are matched to the neighbors' brightness/warmth and may need
  more than one single-frame round; re-check after each round.

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
5. **Gaps are logged and neighbor-filled**, never hidden. Retries only via the
   user-ordered three-step siege (§5).
6. **Independent QC** with 0.5s extraction vs source; verdict in writing.
7. **Fix only flagged frames.** When the user names frame numbers, touch exactly those.
8. **New filename on every re-encode.** Cached old versions must never resurface.
9. **faststart + ffprobe** before every delivery. Verify, don't assume.
10. **Aspect ratio constant;** letterbox bars fixed-size and pure black, head to tail.
11. **State on disk, always.** A dead run is resumed from disk, never restarted.
