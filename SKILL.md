---
name: "pencil-crayon-remake"
description: "Frame-by-frame anime remake pipeline in pencil-sketch + crayon-coloring style: one locked verbatim style prompt, independent per-frame img2img (no chaining), shot classification with a 3-drawings-per-second mode for static shots, 12fps production, honest gap handling, independent frame-level QC, faststart delivery. Use when remaking an anime OP/ED/clip into a hand-drawn pencil & crayon look while keeping the original composition and acting intact."
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
- Record the source's maximum resolution and the quality ceiling it sets:
  a 360p-only source upscaled to 1920×1080 is capped by the source, and no
  redraw can invent detail the source never had.
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

### 3b. No Locked Title Cards, No Locked Backgrounds — Background-First Board Is User-Gated (user orders, 2026-10-19 and 2026-11-03)
- **Title cards are drawn per frame** from each frame's own source frame, like
  everything else. There is no locked or composited title card — a whole-frame
  title lock in Gundam W v3 froze a standing character's pose along with the
  card, and the user ordered the technique removed. Title fidelity is enforced
  by QC instead (§7.5): every frame of a title's appearance is checked at full
  size against the source, including the title's own animation (assembly,
  wipe, drift) exactly as the source performs it.
- **Backgrounds are drawn per frame too.** Never composite a shared background
  plate across frames — the plate pass produced the artifacts the user
  rejected in Gundam X v4. Background consistency is an outcome, not a
  technique: QC measures adjacent-frame background differences and each
  frame's color difference against the scene anchor (§7.8), and frames that
  jump are redrawn from their own source frames.
- **The background-first board is user-gated, not a default method.**
  The board method draws one background plate first and then adds characters
  frame by frame on top of it. Do NOT use it on your own initiative, even
  when the background looks static. Use it only when the user explicitly
  orders it for a named place or segment, and the user judges, place by
  place, whether it suits that place (final user order, 2026-11-03, which
  supersedes the same day's earlier outright-ban wording). Evidence: in
  November 2026 the board was tried on two 90-second OPs — it held the
  background, but per-draw characters and light beams still varied in
  brightness, so flicker remained; one OP's independent QC still failed
  with flicker steps only falling from 379 to 245, and a segment redone
  under the board (arena beams, 15 frames) still had not healed. The user
  then ordered the board-made frames redone from their own source frames.
  A board the user orders for one named segment is not a license to use it
  elsewhere, and it does not revive the abolished background lock of
  2026-10-19.
- **Dark-scene black paper is not a board.** Black paper means each dark
  frame is drawn on black paper from its own source frame, with no shared
  plate reused across frames. Black-paper dark frames are not redone under
  the board rule, and the locked-camera one-moving-element exception in
  §3c (named shots only) is a separate exception, not a general board
  license.
- **Paper color follows the source background (user order, 2026-11-15).**
  Rotate paper colors by the source background; black is not the default.
  Black paper is used only when the background itself is black or dark.
  Colored-paper prompts are derived from the frozen black-paper prompt by
  changing only the paper-color description (gray, light gray, warm tan,
  teal, and similar, chosen against the source background). Each paper
  color has its own luminance distribution, and a target window can fall
  in the gap between two papers' distributions — a structural miss, not a
  sampling miss (see §5b).
- **Flicker repair, replacement method (user order, 2026-11-03):** redraw
  each frame from its own source frame only, generate multiple candidates
  per frame (at least 4, up to 8; stop drawing on a category block), and
  select first on subject likeness and pose, then on luminance closeness
  to the source, then on same-scene neighboring-frame background color
  (background step target under 12 in the November 2026 redo order).
  Diagnose what is actually jumping first — beam, person, or both — before
  rerunning anything; rerunning the same method on an undiagnosed segment
  is how the 15-frame beam segment above stayed unhealed. A candidate
  worse than the current frame is not hard-swapped in: keep the current
  frame and disclose the residual. A candidate that paints only background
  with no face or subject — even if its mean luminance looks close — is
  not a pass; visually confirm the subject exists.
- Group frames into scenes/shots before production and name one **anchor
  frame** per scene as the color reference (hair / costume / machine / scene
  tone). The anchor is a reference for drawing and QC — never a plate or a
  frozen layer.

### 3c. Shot Book & Shot Classification (before the batch)
Build a shot book first; most expensive rework traces back to a missing line here.
Per shot, record:
- Characters present (verified per segment — a misidentified character once
  poisoned a whole segment's hair color), the user/QC standard frame, and
  **measured key colors sampled from this shot's own frames** (scene lighting
  shifts apparent color; never copy a character's colors from another scene).
- Background reference/anchor, motion class (below), treatment (color / black-and-white /
  silhouette / glare-washed), and transition type (cut / dissolve / wipe).
  Record each shot's exact frame boundaries in the shot book: flicker is
  judged shot by shot (§7.8), and a wrong boundary misjudges a cut as a
  flicker.
- **Restricted motifs**: logos, emblems, characters, or props that may appear
  only in specific segments (e.g. an emblem that belongs only at the ending).
  Any occurrence outside its source segment is an automatic FAIL.
- Diegetic text (signs, plaques, in-frame writing that stays): source size,
  position, and density — it is redrawn at source scale, never enlarged.

Motion classes:
- **Dynamic** — full per-frame independent img2img (§3).
- **Slow camera move** — per-frame img2img following the source camera path;
  the background is drawn in every frame like everything else.
- **Static / near-static** — default is still per-frame independent img2img
  like everything else. The **1 second = 3 drawings** mode (3 anchor drawings
  per second, each held 4 output frames; anchor positions chosen from the
  source's actual drift and cut points, frames 1/5/9 only the default for a
  truly still second; slow pushes are NOT static) is **opt-in only**: use it
  solely when the user explicitly asks for it on a named production — never
  on your own initiative (user rule, 2026-10-19).
- **Locked camera, one moving element** (user-approved exception, named shots
  only): take one already-passing generated frame as the base; lock characters,
  composition, and set; vary only the moving element (e.g. water) per frame.

### 3d. Resume Gate — Audit the Base Before Building on It
Before resuming a paused line, assembling from existing frames, or
delivering from them, audit the base and write the audit result into the
production log before new work starts:
- Check the base for methods the user has since abolished — above all
  locked background plates, locked or composited title cards, frozen or
  composited segments carried in from before the method change, and
  background-first boards used without a user order for that segment.
- Count duplicate and frozen frame pairs across the whole base and
  reconcile every pair against logged gap fills, because an unexplained
  duplicate pair is a failed audit.
- Replace suspect segments by per-frame redraw from their own source
  frames instead of shipping them for the user to catch. The QC verdict
  must state the resume-gate result, with duplicate/frozen pair counts
  and how each pair was reconciled.

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
- Close out a multi-worker batch only from a per-group reconciliation table
  listing every worker group, the frames that group claimed, and the
  decode-valid frames actually landed on disk for that group reconciled
  against its own log. Treat a group as incomplete when its landed count
  falls short of its claimed count. End a coordinator run only after
  verified assembly — the assembled file exists, its frame count matches
  the expected count, and its faststart state is correct. Keep the run
  open, or hand the unreconciled list explicitly to a closing coordinator,
  when any group is unreconciled or assembly is missing. Close-out is a
  disk check, never a status summary: a coordinator once reported a group
  as still running when it had already landed 4/4, and the work sat
  unmerged for hours until the user asked.
- Before dispatching a cross-segment rework, verify the segment boundaries
  on disk (landed directories and manifests), never from memory — a
  dispatch once wrote the generation directory wrong in four places
  because the boundaries were recalled, not checked. Check the task brief
  against the line's standing rules (removing names, paper choice, and
  similar) before dispatch: a brief that omits a line rule wastes a whole
  wave of candidates on frames that must be discarded.

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
- **"Use X" means copy X.** When the user says a frame should use another
  frame, copy that frame as the placeholder (byte-identical), never redraw
  it. The same holds for "use the previous frame" fallbacks: disclose
  what the copy gains and what it loses (a copy can fix a wall color while
  losing the text), and offer the revert.
- **When it cannot land, leave the placeholder (user rule, 2026-11-15).**
  After the capped waves below, a frame that still cannot land stays a
  placeholder: no more redraws, no bridge, no filter, unless the user
  orders otherwise. A worse candidate is never hard-swapped in.
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

### 5a. Placeholder Anchors Roll Forward (user order, 2026-11-09)
- Fill every placeholder from the previous frame, uniformly across a
  segment, unless the user explicitly orders a different anchor for a
  named frame and that exception is logged with its frame number.
- Audit every placeholder anchor against the previous frame before any
  retry round starts, and correct wrong anchors before drawing anything
  in that round.
- When a placeholder landing produces a new image, use that new image as
  the anchor for the following placeholders in the same segment, so the
  anchor rolls forward with each landing instead of staying on the old
  frame.
- Report the audit in the retry report with the count of anchors checked,
  the count corrected, and any frame the user explicitly ordered kept on
  a different anchor.

### 5b. Bridge Frames and the Marginal Band
- **Bridge frames are user-approved only, for a user-named range.**
  A bridge is a frame that does not pass the gate, placed as a stepping
  stone so later frames can pass off it. Its over-gate deviation is
  disclosed at approval time (for example a bridge at −20.6, or a second
  bridge at +25.6 after the first), and the bridge is logged as a
  disclosed residual, never as a pass.
- **Paper-distribution gaps are structural.** When the target window falls
  between two papers' distributions (for example black paper tops out
  near 63 while gray paper starts near 129, with the window at 88–118),
  no paper reaches it and more draws of the same papers will not help.
  The options are: expand the redraw range so the ramp can be rebuilt,
  place a user-approved bridge, or accept the placeholder (§5). Do not
  keep drawing the same papers.
- **Marginal band (user order, 2026-11-08, and 2026-11-15):** a luminance
  step over the gate by 5 or less is a marginal residual — accept it,
  disclose it, do not count it as a FAIL driver, and do not ask the user.
  Over the gate by more than 5, the frame does not land. Likeness must
  still look like the source frame; small color or line deviations are
  acceptable and disclosed, a wrong likeness is not.

### 6. Assembly
- 12fps master: `ffmpeg -framerate 12 -i out/f_%04d.png -i audio.m4a -map 0:v:0 -map 1:a:0 -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags faststart final.mp4`
- **Explicit stream mapping is required, not optional.** An audio file can
  carry a hidden video stream that default stream selection picks, shipping
  the wrong picture. Map video stream 0 and audio stream 1 explicitly, and
  verify output dimensions with ffprobe against the remake frame
  dimensions, not the source dimensions.
- **Assembly writes are remove-before-copy.** Remove the destination file
  before copying a frame into an assembly directory, and never overwrite a
  file in place when that file may be hard linked to an earlier assembly.
  After assembly, check earlier assemblies to confirm no earlier assembly
  changed: in November 2026 a copy without remove-first wrote through hard
  links and polluted two earlier assemblies' frames, which had to be
  recovered from the fix workers' own directories. The same rule applies
  when landing a replacement frame into a landed directory: a landed file
  may share an inode with an already-delivered assembly, and an in-place
  copy writes through it and changes the delivered version. Remove first,
  then copy, and verify delivered assemblies are unchanged afterwards.
- After any rework landing, refresh every dependent gap fill before
  assembly, because a gap fill built from an older anchor ships the wrong
  shot. Verify every manifest anchor claim against the actual frame
  before assembly, by content and by md5, and never trust the manifest
  text alone. Back up a manifest before patching it, and check the patch
  maps to the right column (landed luminance, not source luminance) with
  Unix line endings. Keep a fixed frame fixed: when the landed copy of a
  frame is older than the delivered fixed version, keep the fixed version
  in the assembly and sync the landed copy and the manifest row to it, so
  assembly never regresses a fix.
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
   Frames whose background or colors jump visibly or past threshold against
   the source and the scene anchor are redrawn from their source frames; a
   scene that keeps failing this check is redone as a whole scene.
   **Judge flicker shot by shot (user order, 2026-11-16):** the luminance
   step gate applies only inside one shot. At a cut, do not judge the
   luminance step — check only the content (early/late cut, wrong shot)
   and each frame's likeness. White flashes, fades in and out, and
   dissolves inside a shot are still judged by the gate. Compute each
   step from the delivered frames themselves, so a frame that matches its
   own source frame closely can still fail on the jump against its
   delivered neighbors.
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

### 7a. QC Gate Before Delivery (user order, 2026-11-01)
- **Independent QC PASS is required before delivery.** The producer's own
  check does not count as a pass, and a FAIL film is not delivered or
  published, however far the numbers improved.
- **A PASS film is delivered at once, without asking again (user order,
  2026-11-04).** Do not hold a PASS film for a separate delivery-approval
  question: upload the newest version, share it with the standing share
  recipient, move the previous delivered version to trash, and send the
  user a copy in the conversation. A FAIL film stays undelivered as a
  final version unless the user explicitly orders a review copy, and then
  label that copy as a review copy, not a final version. Two exceptions:
  a user hold ("wait for X") suspends even a PASS direct delivery until
  the hold lifts, and a user order to skip QC or to upload in parallel
  for review is followed — but that upload is labeled a review copy, not
  a PASS delivery, and the FAIL conclusion (if any) stays on record.
  Sending frame sheets for the user to look at does not pause PASS
  delivery, unless the user says to look first.
- **Re-verify the verdict numbers.** The coordinator independently
  re-checks the verdict's key numbers against disk before acting on them
  — a verdict once claimed two frames had different md5s when they were
  identical, and the defect itself was real but the number was not.
- Besides the 13 items above, QC must check the standing user-opinion
  checklist and write each check into the verdict: subject-scale trajectory
  (height / centroid step of roughly >10px inside a continuous segment is
  a one-frame pulse and a flicker FAIL); neighbor-frame face/subject
  brightness outliers; the same defect swept across the whole segment and
  visually similar segments, not only the named frames; a whole-film
  ghosting / double-image sweep; anchor-frame character alignment (each
  frame still redrawn from its own source frame, never a wholesale copy of
  the anchor); title-card segments checked to the end of their shot, as
  their own verdict line; a placeholder list kept separate from real
  frames, with freeze fills and neighbor fills labeled differently; dark
  and black frames; costume and hair color against neighbors and source;
  scope discipline — changed frames reconciled against the authorized
  list, neighbor defects reported, never self-expanded; and measurement
  standards applied literally as given.
- **Disclosure residuals are listed separately and do not count as FAIL;
  pending user decisions are reported, never decided by QC.** An accepted
  residual band is not open-ended: a frame past the accepted band needs a
  new user call, not a silent acceptance. A candidate worse than the
  current frame is not swapped in; the current frame stays and the
  residual is disclosed.
- Check a defect class at full size. When a defect is found, sweep the
  same defect across the whole segment and visually similar segments
  frame by frame at full size, never at frame-sheet scale alone, because
  thin artifacts are invisible at contact-sheet scale. When a repair
  wave notices the same defect on a sibling frame outside the named
  range, list it in the verdict as a disclosed residual with its frame
  numbers when the user has not ordered it fixed, and never leave it
  unmentioned.
- Check unification in landed space. When a segment is unified to one
  base frame, check ghosting and double images by comparing each landed
  frame against the base and the source, frame by frame, never by source
  frame difference alone or by spot-checking two frames, because a ghost
  can exist only in the generated frames while the source shows no
  difference.
- Measure every inter-frame step in delivered space. For each filter
  fallback frame and each reworked frame, report its step deviation
  against the adjacent delivered frames, never its deviation against its
  own source frame. A source-relative number is not a step, because a
  filter frame can sit close to its source while jumping far against its
  AI neighbors. A frame whose delivered step jumps visibly against its
  neighbors is a FAIL driver, even when its source-relative deviation is
  small. A filter fallback is a user-approved exception for a named line
  only (brightness stitching with a linearly decreasing offset, as used
  on two lines), never a default method.
- Re-verify even a one-frame change. A version that changes only one
  frame still needs an independent re-verification scoped to that frame
  and its neighbors before delivery. A direct upload order from the user
  is not a scoped re-verification unless the user explicitly orders
  skipping QC, in which case label the delivery as a review copy with no
  independent QC verdict, not as a final version.

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
- **Fix waves are capped at 2 waves × 4 draws per frame.** Only frames
  that failed the previous wave enter the next wave. For a text fix, the
  gate judges the text only (not costume color); after 2 waves, stop and
  close with the closest previous text-correct frame copy, or keep the
  current frame — do not keep drawing blindly.
- **Fix instructions outrank the source.** When the user says to change
  only the text, change only the text on the current version of the
  frame, without checking the source text; the source is the auxiliary
  reference, the instruction is the authority. When the user names a
  frame to look at first ("make it and show me"), that frame goes
  look-first and is not assembled until the user approves it.
- **Fit every hair-color fix to that frame's own hair shape.** Build the
  mask from that frame's own source frame and fit it to the hair outline
  in that frame alone. Never reuse a fixed rectangle, a fixed circle, or
  another frame's mask. Check the mask against the hair before applying
  color. The mask must cover hair only, not face, eyes, forehead, skin,
  or background. If the mask does not fit, do not apply the change: leave
  the frame unchanged and report it as a disclosed residual. A wrong mask
  is worse than no fix. Check the recolored frame at full size against
  the source and neighbors; reject and revert the frame when any colored
  shape sits outside the hair (disc, ellipse, rectangle, or straight
  edge on a face or wall). Fix consecutive frames as a group only when
  each frame still gets its own fitted mask.
- Use the mask to constrain an AI redraw, not to authorize a
  programmatic recolor. The mask limits where the image pipeline redraws
  or inpaints the hair. A mask recolor that changes color on the current
  frame without redrawing it is a base-preserving edit: use it only when
  the user approves or prescribes it for that line. Never use it to
  bypass an AI-redraw-only mandate for that line. A minimal local edit
  (for example darkening only a face) is likewise only for a user-named
  frame, labeled with its method in the log.
- **Consecutive frames with the same defect are fixed as a group**: one shared
  geometry/color target across the run and a check against the frame after the
  run, so the fix itself doesn't introduce a jump.
- If you spot a worse defect on a neighboring frame while fixing, report it and
  wait for the user's call — do not expand the scope on your own.
- Skin-tone fixes are matched to the neighbors' brightness/warmth and may need
  more than one single-frame round; re-check after each round.
- **Rework is AI redraw only, no programmatic pictures.** Every frame,
  including pure-black frames, is redrawn by the image pipeline; PIL/Python
  generated pictures, programmatic blending, and programmatic crossfades are
  forbidden in rework. A rework uses the earliest production method for that
  line (original prompt, method, and parameters as logged — no self-invented
  "optimization"), covers the problem segment plus a few seconds either
  side, redraws at 12fps frame by frame, and carries every user-pointed
  defect into the fix.
- **Subtitles are overlaid, never AI-drawn.** The AI draws a clean picture
  only, with no subtitles; subtitles are then overlaid to the source
  specification (font, size, position, timeline). The AI must not change a
  subtitle's font or size, and must not be asked to draw subtitle text.

### 8. Delivery Checks
- `ffprobe`: duration, fps, total frame count match expectations (verify the
  real stream frame count — a film once shipped assembled at 24fps/2160 frames
  by mistake and had to be re-assembled at true 12fps/1080).
- Spot-check frames at ~10s / 45s / 80s against the source.
- Confirm `faststart` (moov before mdat).
- If an encode is interrupted (for example by a runtime restart), the
  file is partial and has no usable moov: re-encode it, do not ship it.
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
- Do not state the frame rate in the post description.
- **Recheck the version at upload time**: list the output folder again in the
  moment right before uploading and upload only the newest landed version —
  never a snapshot taken when the user picked a cover or approved an earlier
  version (a v2 once went up while v3/v4 landed during cover selection).

### 10. Preserve the Production Assets
The workspace is disposable; the method assets are not.
- Archive outside the deletable workspace: the frozen prompt (full text + hash),
  the batch log, the assembly/gap map, and the shot book. A cleanup once
  destroyed a frozen prompt original — only its hash survived.
- If the prompt is lost, reconstruct it from the skill template + surviving
  specs/logs, validate the reconstruction on a standard frame, freeze it, save
  it back, and label it **reconstructed** in every report — never present it as
  the original.
- A repair base rebuilt from a finished film (frames extracted back out) is a
  repair base, not the production master: label it as recovered, with what was
  permanently lost (candidate pools, per-round logs) stated once.

### 11. Review Frame Sheets (how QC material is shown to the user)
- One sheet per second, 12 frames per sheet (the 12fps grid), frame numbers in
  white for user delivery. No 25-frame mega-sheets, no one-image-per-frame spam.
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
   The static-shot 1s = 3 drawings mode is opt-in only, when the user asks (§3c).
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
13. **No locked title cards, no locked backgrounds.** Both are drawn per frame
    from their own source frames; consistency is a QC measurement against the
    source and the scene anchor, never a composited lock (user order, 2026-10-19).
14. **Check continuity spans frame by frame.** A persistent title card or prop
    must stay present, single, and on-model in every frame of its span.
15. **Shot book before batch.** Characters, measured colors, background
    references, treatments, restricted motifs, and motion class are written
    down before production.
16. **Verify the title against the source** before any post; no AI in the title,
    AI declared in the creation statement; existing AI-titled posts stay as they are.
17. **Recheck the version right before upload.** Upload only the newest landed version.
18. **Preserve the method assets** (prompt, logs, maps, shot book) outside the
    deletable workspace; a lost prompt is reconstructed, validated, and labeled.
19. **Background-first board is user-gated only** (§3b): never on your own
    initiative, only on the user's explicit order for a named place, judged
    place by place by the user. Dark-scene black paper is not a board.
20. **Audit the base before building on it** (§3d): abolished methods,
    duplicate/frozen pairs, and unreconciled fills are replaced by per-frame
    redraw, never shipped for the user to catch.
21. **Assembly safety:** remove-before-copy in assembly directories, never
    overwrite a possibly hard-linked file in place, and map streams
    explicitly with a dimension check against the remake frames (§6).
22. **QC PASS before delivery** (§7a): producer self-check does not count;
    disclosure residuals are listed separately, pending decisions are
    reported, never decided by QC.
23. **Rework is AI redraw only** (§7b): no programmatic pictures or blending;
    the AI draws clean pictures only and subtitles are overlaid afterwards
    to source specification.
24. **Deliver only the latest version:** older delivered versions are cleared,
    so the newest version is the only one in circulation.
25. **PASS delivers at once, without asking again** (§7a): upload the newest
    version, share it with the standing share recipient, move the previous
    delivered version to trash, and send the user a copy in the
    conversation. A FAIL film stays undelivered as a final version unless
    the user explicitly orders a review copy, labeled as a review copy.
    A user hold suspends even a PASS delivery until it lifts.
26. **Hair-color fixes stay inside the hair** (§7b): use a mask fitted to
    each frame's own hair shape to constrain an AI redraw or inpaint,
    check it before applying color, and leave the frame unchanged when
    the mask does not fit. A non-redraw mask recolor is a base-preserving
    edit only when the user approves or prescribes it for that line, and
    never to bypass an AI-redraw-only mandate.
27. **Verify landed frames, not proxies** (§6 and §7a): check defect
    siblings at full size, check unification ghosting in landed space
    frame by frame, refresh gap fills after a rework landing and verify
    manifest claims against actual frame content and md5, and re-verify
    even a one-frame change before delivery.
28. **Placeholder anchors roll forward** (§5a): placeholders use the
    previous frame, every placeholder anchor is audited before a retry
    round, and a landed placeholder becomes the anchor for the following
    placeholders in its segment.
29. **Measure inter-frame steps in delivered space** (§7a): report a
    filter fallback or reworked frame by its deviation against the
    adjacent delivered frames, never by its deviation against its own
    source frame.
30. **Paper color follows the source background** (§3b): rotate paper
    colors, black only when the background is black or dark. A target
    window between two papers' distributions is structural, not a
    sampling miss (§5b).
31. **Judge flicker shot by shot** (§7.8): the luminance step gate applies
    only inside one shot; at a cut, check content and likeness only.
    White flashes, fades, and dissolves inside a shot are still gated.
32. **Marginal band is 5** (§5b): a step over the gate by 5 or less is an
    accepted disclosed residual; over 5, the frame does not land.
33. **When it cannot land, leave the placeholder** (§5): no more redraws,
    no bridge, no filter, unless the user orders otherwise. Bridges are
    user-approved only, for a named range, with the deviation disclosed.
34. **"Use X" means copy X** (§5): a user-named frame copy is
    byte-identical, never a redraw, with gains and losses disclosed.
35. **Fix waves are capped at 2 × 4** (§7b): only failures advance; a text
    gate judges text only; after 2 waves, close with the closest
    text-correct copy or keep the current frame.
36. **Verify boundaries and briefs on disk before dispatch** (§4):
    segment boundaries from disk, never from memory; briefs checked
    against the line's standing rules; close-out from disk, never from a
    status summary.
