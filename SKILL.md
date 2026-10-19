---
name: "pencil-crayon-remake"
description: "Frame-by-frame anime remake pipeline in pencil-sketch + crayon-coloring style: one locked verbatim style prompt, independent per-frame img2img (no chaining), one locked title card and one locked background per scene, shot classification with a 3-drawings-per-second mode for static shots, 12fps production, honest gap handling, independent frame-level QC, faststart delivery. Use when remaking an anime OP/ED/clip into a hand-drawn pencil & crayon look while keeping the original composition and acting intact."
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
- Establish the aspect ratio before anything else: distinguish the file's canvas
  ratio from the picture's content ratio (pillarbox/letterbox), and state the
  delivery ratio and any fixed crop window in the production log.
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

- Describe a character trait (hairstyle, hair color, costume detail, jersey number)
  only after viewing the matching source frame — never from memory or impression.
- When the user names a standard frame for a trait, cite that frame number in
  every report and fix instruction that relies on it.

### 3. Per-Frame Independent img2img — No Chaining (Iron Rule)
Each output frame is transformed **from its own source frame only**.
Never feed frame N's output as the reference for frame N+1.

Chaining accumulates drift: small characters melt, poses warp, faces slide off-model
(a chained run once turned a slight head-tilt into a deep bow with wild hair).
Independent frames keep every frame anchored to the original.

**Complex motion (rotations, multi-subject moves):** never reconstruct the motion
from measured angles/curves or from imagination — two such reconstructions were
rejected on likeness and pose. Run probe frames first (start / middle / end of the
segment), get the user's go-ahead, then direct-img2img every source frame.
Measurements are for QC and candidate selection only, never for rebuilding action.

### 3b. Lock the Title Card and the Backgrounds Before the Batch
- Build exactly **one finished title card** for the whole film before batch
  production: text, font size, position, and decoration fixed from the source.
  Reuse that same card on every frame where it appears; never redraw it per frame.
- The lock covers the artwork, not the source's animation: if the source title
  assembles, wipes, or drifts, composite the one locked card along the source's
  own trajectory/timing — do not freeze a moving title, and do not redraw it.
- Group frames into scenes/shots before production; name one **anchor frame**
  per scene. Where frames share a background, use **one locked background**
  for the whole span instead of redrawing it per frame, and lock the scene's
  colors to the anchor (hair / costume / machine / scene tone — no large jumps
  between neighboring frames).
- A locked background is a locked *design*, not a static-plate shortcut: if the
  source camera moves, the background content and camera path must still follow
  the source (a user-cancelled single-image pan is the cautionary case).
- When a background is wrong, fix the background layer/plate itself and
  recomposite only the affected frames instead of redrawing the characters.

### 3c. Shot Book & Shot Classification (before the batch)
Build a shot book first; most expensive rework traces back to a missing line here.
Per shot, record:
- Characters present (verified per segment — a misidentified character once
  poisoned a whole segment's hair color), the user/QC standard frame, and
  **measured key colors sampled from this shot's own frames** (scene lighting
  shifts apparent color; never copy a character's colors from another scene).
- Background plate/anchor, motion class (below), treatment (color / black-and-white /
  silhouette / glare-washed), and transition type (cut / dissolve / wipe).
- **Restricted motifs**: logos, emblems, characters, or props that may appear
  only in specific segments (e.g. an emblem that belongs only at the ending).
  Any occurrence outside its source segment is an automatic FAIL.
- Diegetic text (signs, plaques, in-frame writing that stays): source size,
  position, and density — it is redrawn at source scale, never enlarged.

Motion classes:
- **Dynamic** — full per-frame independent img2img (§3).
- **Slow camera move** — per-frame img2img; locked background design following
  the source camera path.
- **Static / near-static** — do NOT draw 12 unique frames per second (per-frame
  coloring is the flicker source). Use **1 second = 3 drawings**: 3 anchor
  drawings per second, each held 4 output frames. Choose anchor positions from
  the source's actual drift and cut points, not mechanically (frames 1/5/9 is
  only the default for a truly still second). Slow pushes are NOT static.
- **Locked camera, one moving element** (user-approved exception, named shots
  only): take one already-passing generated frame as the base; lock characters,
  composition, and set; vary only the moving element (e.g. water) per frame.

### 4. Parallel Batch Production
- Split the frame range into contiguous chunks across 2–4 workers.
- Naming: `f_0001.png`, `f_0002.png`, … (zero-padded, sortable).
- Each worker records its own refusals/gaps with frame numbers and reasons.
- **All state lives on disk** (batch log + frame files). If a worker dies mid-run,
  the next one resumes from the on-disk state — never restart from zero.
- Report progress only from files verified on disk. After any interruption,
  validate that frame files actually decode and match the expected dimensions —
  a truncated PNG still counts in a file listing (two frames were once silently
  cut in half by a restart). Correct a wrong progress report in the same thread
  as soon as a disk check disagrees with it.

### 5. Gap Policy — Honest, Never Hidden
The image pipeline will refuse some frames (content filters) or return black/bad frames.
- **Default: never reword-retry a category-blocked frame.** A refusal is a refusal.
- Grade every gap by run length:
  - **1–2 frames**: fill with the nearest good neighbor frame; leave as is.
  - **3+ consecutive frames**: a static freeze is NOT acceptable. Rebuild the run
    as moving frames — each frame slightly different, the differences stringing
    into the source action (transition generation, §5 siege step ③, or the
    locked-base method of §3c). If every attempt fails or is QC-rejected, keep
    the neighbor fill but report the run as an unresolved gap, never as fixed.
- Log every gap: frame number + reason. Report gaps to the user as 缺口, never
  silently. Always report the **longest continuous static stretch** too, not just
  the total count (one production hid a 42-frame ≈ 3.5s freeze inside a
  "13% gaps" total), plus the per-step recovery counts.
- **The three-step siege — exception, gated:** only on the user's explicit order,
  only for user-named segments: ① retry with the original prompt, up to 2 attempts;
  ② retry with a slightly adjusted prompt, up to 2 attempts (log the exact wording
  change); ③ generate the frame from its neighbors' transition. Disclose every
  step-③ deviation from the source (wrong facing, shifted palette, missing
  elements) — a recovered frame is not automatically a faithful frame.
- **Hard stops that override the siege:**
  - If the pipeline says a request must not be repeated, do not repeat it;
    only a non-repeating fallback the user has approved may be used.
  - Sensitive category blocks (bathing/shower/nudity-class content): prompt
    tweaks and transition generation would be filter evasion — do not execute
    steps ②/③ against them. Close such runs only with already-passing neighbor
    AI frames under the user's closing instruction, each fill labeled as a fill.
  - A user-set stop condition ("last attempt", a retry budget, a schedule) is
    final: when it triggers, stop, leave the film unchanged if nothing passed,
    and close any retry job.

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
- If the user changes the aspect ratio, redo from the source frames at the new
  ratio — never crop or stretch the previous version's finished frames to fit.

### 7. QC — Independent Agent, "Does It Look Right"
A separate QC agent (not the producer) checks the assembled video:
1. **Character consistency**: extract every 0.5s, compare faces against the source frame.
2. **No artifacts**: no foreign objects, no melted details, no extra limbs.
3. **Object/scene logic**: props, costumes, and background anchors consistent across cuts.
4. **Natural expressions**: restrained, on-model — never generic pretty faces.
5. **Title cards by continuity, at full size**: check every frame of a title's
   appearance — never from thumbnail contact sheets (a thumbnail audit once
   reported 3 bad frames where a full-frame quantitative audit found 22).
   Report the title-card check as its own line in the verdict, including the
   card's background/full-frame state, not just the lettering.
6. **Skin-tone & brightness consistency**: within a shot, flag frames whose skin
   tones are outliers (too pale / too dark) against their neighbors.
7. **Subject scale consistency**: within a continuous shot, the subject's size
   must not breathe (one 6-frame run drifted ~6.5%).
8. **Continuity spans**: check every frame inside any span where one element
   persists (title card, recurring prop, background), not only the 0.5s samples.
   A recurring prop must stay one design across front/side/bottom views, appear
   once per frame, and gain no set elements the source doesn't have. Measure the
   background difference between neighboring frames and each frame's color
   difference against the scene anchor; report the numbers in the verdict.
   A locked background passes only at measured difference zero, or with the
   residual explained frame by frame; a scene whose locked background differs
   or whose colors jump past threshold is redone as a whole scene.
9. **Dark scenes & black frames (dedicated pass)**: sample every dark shot.
   Black frames stay pure black — any invented paper edge, border, or glow is a
   FAIL (dark space backgrounds repeatedly came back beige-paper; a black frame
   once came back white). Check generator-systematic shapes (stars, glow discs)
   against the source one by one.
10. **Text audit**: inventory every readable glyph against the source. Invented
    or corrupted text anywhere is a FAIL; preserved diegetic text must match
    the source's size and position (oversized in-frame text was a user catch).
11. **Transitions**: dissolves, wipes, and fades are temporal objects — verify
    the effect survives across its whole span (a source dissolve once vanished
    into a hard background jump). Verify the true boundary of a continuous
    effect in the source before fixing a named range: a title wipe the user
    ended at one frame actually ran 7 frames longer, and stopping at the named
    frame would have left a visible break.
12. **Transition-generated frames (siege step ③) get their own gate**: compare
    source | established same-character anchor | candidate side by side, face
    zoom included. Reject distorted likeness, changed costume patterns, missing
    hand-held props, panel/layout changes, or compositions unrelated to the
    source — a moving but wrong frame is worse than an honest fill.

Verdict per batch: PASS / FAIL with frame numbers. FAIL frames are redone individually
(per §3, from their own source frames), then the whole clip is **re-encoded under a
new filename** — never reuse a filename the user may have cached.

### 7b. Targeted Frame Fixes (user-named frames)
- Confirm the frame-sheet label ↔ production-file mapping by pixel comparison
  before touching anything: some productions label sheets 0-based while files
  are 1-based — an off-by-one here fixes the wrong frames.
- Generate 4–5 candidates per frame (up to 10 for a pure color fix). Reject any
  candidate that violates ANY locked attribute (hair color, costume color,
  composition), even if the named defect is fixed.
- **Select quantitatively where a quantity exists**: color fixes are chosen by
  measured color family / luminance / saturation against the source and the
  neighbors (a shirt was fixed by picking the one candidate whose measured
  color left the black range into the source's grey-purple family; an eye color
  by saturation 0.655 → 0.557 to match its neighbors).
- **Consecutive frames with the same defect are fixed as a group**: one shared
  geometry/color target across the run and a check against the frame after the
  run, so the fix itself doesn't introduce a jump.
- If you spot a worse defect on a neighboring frame while fixing, report it and
  wait for the user's call — do not expand the scope on your own.
- Skin-tone fixes are matched to the neighbors' brightness/warmth and may need
  more than one single-frame round; re-check after each round.

### 8. Delivery Checks
- `ffprobe`: duration, fps, total frame count match expectations (verify the
  real stream frame count — a film once shipped assembled at 24fps/2160 frames
  by mistake and had to be re-assembled at true 12fps/1080).
- Spot-check frames at ~10s / 45s / 80s against the source.
- Confirm `faststart` (moov before mdat).
- Play the full clip once, end to end, before announcing completion.

### 9. Publishing — Verify the Title Against the Source
- Build the Bilibili title only after inspecting the source clip itself. Name the
  segment for what the source frames show it is (OP, ED, MV, main-footage clip),
  never from impression or from the file name alone. Use the series' formal title
  exactly as the source gives it.
- Keep AI out of every new Bilibili title; use the form 《XXX》OP铅笔蜡笔重制版
  (substituting the verified segment for OP when it isn't one). Declare the AI
  generation in the **creation statement** instead (select the AI-generated
  option) and keep the AI explanation in the description. Leave existing posts
  that already carry AI in the title unchanged.
- After posting, open the live post and confirm the title shown matches what was
  verified and that the creation statement took effect.
- **Recheck the version at upload time**: list the output folder again in the
  moment right before uploading and upload only the newest landed version —
  never a snapshot taken when the user picked a cover or approved an earlier
  version (a v2 once went up while v3/v4 landed during cover selection).

### 10. Preserve the Production Assets
The workspace is disposable; the method assets are not.
- Archive outside the deletable workspace: the frozen prompt (full text + hash),
  the locked title card and background plates, the batch log, the assembly/gap
  map, and the shot book. A cleanup once destroyed a frozen prompt original —
  only its hash survived.
- If the prompt is lost, reconstruct it from the skill template + surviving
  specs/logs, validate the reconstruction on a standard frame, freeze it, save
  it back, and label it **reconstructed** in every report — never present it as
  the original.
- A repair base rebuilt from a finished film (frames extracted back out) is a
  repair base, not the production master: label it as recovered, with what was
  permanently lost (candidate pools, per-round logs) stated once.

### 11. Review Frame Sheets (how QC material is shown to the user)
- One sheet per second, 12 frames per sheet (the 12fps grid), frame numbers in
  red. No 25-frame mega-sheets, no one-image-per-frame spam.
- Keep sheets in a persistent workspace directory, never /tmp (links died there).

## Tooling
- Image pipeline `img2img`: per-frame stylization (source frame + locked prompt)
- `muse.exec` + `ffmpeg`/`ffprobe`: extraction, assembly, verification
- Subagents: parallel frame chunks (§4), independent QC agent (§7)

## Operating Rules
1. **One prompt, verbatim, every frame.** No mid-batch edits, except the logged
   step-② wording change inside a user-ordered siege (§5).
2. **No chaining.** Each frame is independent img2img from its own source frame.
3. **Strict fidelity.** Change the rendering style only; never invent details.
4. **12fps production standard.** Duplicate to 24fps at assembly if needed.
   Static shots use 1s = 3 drawings (§3c) inside that standard.
5. **Gaps are graded by run length**: 1–2 frames neighbor-filled; 3+ frames
   rebuilt as moving frames or reported unresolved. Retries only via the
   user-ordered three-step siege, under its hard stops (§5).
6. **Independent QC** with 0.5s extraction vs source; verdict in writing.
7. **Fix only flagged frames.** When the user names frame numbers, touch exactly those.
8. **New filename on every re-encode.** Cached old versions must never resurface.
9. **faststart + ffprobe** before every delivery. Verify, don't assume.
10. **Aspect ratio constant;** letterbox bars fixed-size and pure black, head to tail.
    An aspect change means a full redo from source, never a crop/stretch of old frames.
11. **State on disk, always.** A dead run is resumed from disk, never restarted;
    progress is reported from disk-verified, decode-validated files only.
12. **View the source before naming a trait.** Never describe a hairstyle, color
    or costume detail from memory or impression; cite the user's standard frame.
13. **One title card per film.** One locked card reused on every frame it appears;
    source title animation is followed with that same card, never redrawn.
14. **One background per shared scene.** One locked background + one anchor color
    set per scene; QC reports measured background/color differences per scene.
15. **Check continuity spans frame by frame.** A persistent title card or prop
    must stay present, single, and on-model in every frame of its span.
16. **Shot book before batch.** Characters, measured colors, plates, treatments,
    restricted motifs, and motion class are written down before production.
17. **Verify the title against the source** before any post; no AI in the title,
    AI declared in the creation statement; existing AI-titled posts stay as they are.
18. **Recheck the version right before upload.** Upload only the newest landed version.
19. **Preserve the method assets** (prompt, plates, logs, maps) outside the
    deletable workspace; a lost prompt is reconstructed, validated, and labeled.
