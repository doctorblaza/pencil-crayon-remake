# Rework Experience — Casebook

Casebook from pencil-crayon B-track productions (12fps, per-frame independent img2img)
that went through multiple user QC rounds. The fixes below are the expensive lessons;
read this before starting a new remake, and again before any rework round.

- **Slam Dunk OP** (103.25s, 1239 frames): v2 deleted → v3 QC FAIL → v4 PASS;
  4:3 line (1440×1080) 43 → 43b → 43c → 43d → **43e final**, published.
- **Fire Force OP** (89s, 1068 frames, 1920×1080): v1 → v2/v3/v4 (title-card
  continuity, three rounds) → v5–v8 (skin-tone & scale chain).
- **IRIS OUT** (MV 153s/1838 frames; OP 149s/1788 frames): blocked-frame
  three-step siege, letterbox normalization, candidate-elimination fixes.
- **October 16–19 batch** (Yanineko, KICK BACK, Chuuka Ichiban, Gurren Lagann,
  Frieren OP2, Fire Force ED1, Higurashi OP, Gundam W/X, FX, Witch from Mercury):
  dynamic fills, static-shot mode, locked plates, direct-source motion — see the
  dated casebook section below.

---

## Watermark & Burned-in Text Handling (Slam Dunk)

The Slam Dunk sources were the hard case: every findable copy had burned-in text —
Bilibili 960×540 with hardcoded Chinese subtitles + Japanese staff credits,
YouTube 1080P likewise. No clean source existed.

### What failed: a separate de-watermark pass
v2 first ran de-subtitling (neighbor-frame inpainting + AI repair) as its own step,
then stylized. The user verdict: "去水印没去好" (not clean) — v2 was deleted outright.

### What works: fold removal into the redraw itself
From v3 on, there is **no separate de-watermark step**. Each frame is redrawn from
scene content only — lyrics, subtitles, credits, watermarks, and title logos are
simply **never drawn**. The redraw's fidelity target is the clean scene; text
overlays are not part of the scene, so they never enter the output.

### Caveats learned the hard way
- **Hidden content has no source pixels.** Whatever the burned-in text covered is
  reconstructed plausibly by the redraw, not recovered. Check those regions
  (usually the bottom third) extra carefully in QC.
- **The model invents glyphs on blank surfaces.** In an early build the AI painted
  a readable 「彦」 character onto a blank backboard (39.5–40.5s; 38 frames had to be
  redone). Large blank surfaces — backboards, walls, skies — get an explicit
  residue scan in QC; "looks like text" anywhere is an automatic FAIL.
- **Process rule:** say upfront that removal is hard, present options, and only
  start after the user authorizes it ("你自己想办法把字幕去掉").
- **Contrast case (Fire Force):** a clean no-credit source existed — use it and
  skip watermark work entirely. Always hunt for the clean source first.

### Verification that passed
- Full-sample text scan: 207 frames checked, **zero** text residue.
- All 13 backboard frames blank; corners / title cards / bottom third spot-checked.

---

## Slam Dunk — Rework Log

**Hair/character locks (set before any rework, never improvised):**
Sakuragi = bright-red **airplane head** (not a flat top), Miyagi = brown hair,
Haruko = dark-brown hair. Describe a character's features only after looking at
the source frame — never from impression (the "flat top" mislabel caused a redo).

**v3 → v4 (199 frames redrawn, QC FAIL → PASS).** v3's hard failures:
- Street-ball segment: a defender girl's leg drawn as a **black mechanical
  prosthetic** (f_0468) — and the gap-fill copy spread it to 2 more frames.
  Lesson: neighbor-filled gap frames inherit the neighbor's defects; QC them too.
- Crosswalk segment: Sakuragi's hair uncolored/color-shifting for ~14 frames.
- Ending hero shot: jersey flickered a readable fake word ("OHOK") + number,
  jacket jumped blue↔black, Sakuragi's hair went brown and his face aged.
- Miyagi drawn red-haired; Haruko's hair changed color three times.
Fix: redraw all 199 flagged frames with the original earliest-version method +
the hair locks above; zero gaps; independent QC PASS.

**v5 — the glare exception.** 1:04–1:06 (f_0776–f_0784): hair read wrong because
the **source frames are blown out by sunlight glare — there is no red information
in the source to recover**. Root cause stated to the user; user authorized a
one-off exception (saturated "grade-A bright red") for exactly those 9 frames.
QC then flagged color jumps in neighbors and silver-gray hair on f_0756 —
user ruled **"不扩，就这样"** (don't expand): the exception scope is a hard
boundary; accepted defects are recorded as accepted, not silently fixed later.

**4:3 line (43 → 43e).** Source is 4:3 inside a 16:9 pillarbox: crop the black
bars, deliver 1440×1080.
- Ending pose: jersey number must be **10 in every frame** — 79 frames checked
  one by one, 28 missing-number frames redrawn.
- 58–59s: user named 6 frames from a frame sheet (f_0696/697/699/700/702/703);
  hairstyle, trouser color, shirt, and the one-hand backpack action had to match
  adjacent frames; 12 more frames checked backward, 10 redrawn total (43b).
- Opening 0–6s: **pure black, no characters**; exactly **one** central light for
  the whole segment, drifting slowly upward as in the source; no roof, no extra
  lights, no high seats, no ceiling. 70 light-segment frames redrawn against
  standard frame f_0010 (43e).
- Hairstyle segment f_0775–f_0782 standardized against user-named standard frame
  f_0784. Back numbers re-checked across the whole 22–26s dunk segment.
- **Progress-reporting lesson (43d):** the build was reported "stuck" while the
  finished file had already landed on disk. Always verify on-disk artifacts
  before reporting status — disk truth beats pipeline summaries.

---

## Fire Force — Rework Log

Source: no-credit OP (YouTube), 0:00–1:29 only. User's standing requirement:
**backgrounds must not flicker or jump** between consecutive frames of the same shot.

**Base delivery:** 1068/1068 frames drawn; an anti-flicker pass replaced **84
tonally abnormal frames** (per-frame independent coloring makes long shots
shimmer — scan same-background runs for tonal outliers and replace them);
4 pipeline-rejected frames (f_0166, f_0735, f_0800, f_0950) neighbor-filled.

**Title cards are temporal objects (v2, v3).** The user caught what frame-level
QC missed: within a title's appearance, some frames had the title and their
immediate neighbors didn't — a visible pop.
- v2: ending title f_1038–f_1064 — 3 frames missing or mis-sized; fixed and aligned.
- v3: opening title — f_0042–f_0052 had **no title at all**, then the full title
  popped in at f_0053. Deeper check found f_0053–f_0057 and f_0059 also missing
  (only f_0058 had it): **17 frames replaced total, nothing else touched.**
- The original opening title **assembles piece by piece and drifts downward**
  while doing so — the redraw must reproduce both the assembly and the drift,
  not just paste a static title.
Rule: QC title/credit overlays by **continuity across the whole appearance**,
never by single frames; a title may never pop in or out mid-sequence.

**v4 — thumbnails lie.** The user reported the ending title still leaking at
1:26–1:28. A full-frame quantitative audit redefined the segment as
f_1036–f_1064: **22 frames** missing or incomplete (incl. f_1055, which needed
a second repair). v2's "3 frames" figure came from checking a thumbnail
contact sheet. Title-card verification is full-size, frame by frame, plus a
quantitative audit.

**Skin-tone chain (v5–v8).** Per-frame coloring also throws off skin tones
within a shot. v5 fixed 11 user-named frames (6 too pale: f_0556/0558/0562/
0565/0574/0575; 5 too dark: f_0566/0573/0961/0962/0963) by matching neighbor
brightness; f_0573 still read dark on the user's re-watch → one more
single-frame round → v6. v7: f_0449/f_0450 (corner figure distorted, palette
pulled toward f_0447/f_0448), f_0577/f_0578 brightened toward f_0576/f_0579.
v8: f_0960/f_0962 faces restored to the neighbors' warm tone, and f_0963
**enlarged to sit between f_0962 and f_0964** — subject scale is a sequence
property, not a frame property. Off-by-one hazard on this line: frame sheets
were labeled 0-based, production files 1-based (label N = file f_{N+1});
pixel-compare to confirm the mapping before assembling any named-frame fix.

---

## IRIS OUT — Blocked-Frame Siege & Letterbox

Two versions of the same song (official MV, 153s/1838 frames; no-subtitle OP,
149s/1788 frames) stress-tested the gap policy.

**MV version.** 239 frames (13%) refused (chainsaw/gore/bed scenes); the
longest neighbor-filled stretch was f_1070–f_1111 = 42 frames ≈ 3.5s static —
**report the longest stretch, not just the percentage**. An anti-flicker pass
unified the ending static card and repaired 8 of 12 single-frame tone jumps.
The production agent died at 171/1838; a replacement resumed from on-disk
state across repeated service restarts and finished the run. MV v2 (cut to
1:26 per the user) also set the letterbox precedent: bars normalized to fixed
137/137 pure black, verified constant on 7 sampled frames, content unscaled.

**OP version — the three-step siege.** On the user's explicit order, the 160
refused frames were attacked in steps: ① same-prompt retries recovered 8;
② slight prompt adjustments recovered 14; ③ transition-generation from
neighbors recovered 136. Two frames (f_0450/f_0451, a near-static yawn
close-up) never passed and stayed neighbor-filled (~0.17s). Step ③ is **not**
free fidelity — known deviations, disclosed to the user: f_0443 generated as
a back view (source: front); a bedroom close-up run darker and teal-er than
source; a clock run unified warm, with a cat emblem where the source has an
eye; f_1460/f_1465 missing a foreground creature; a splash burst omitted.
A siege recovery is always reported **with its deviations** — silent
substitution is how trust dies.
Scale consistency: the 28–32s newspaper figure breathed ~6.5% across 6
frames (f_0345, f_0358, f_0362, f_0367, f_0370, f_0373) — located
quantitatively, redrawn, re-measured back to baseline.

**Single-frame fix by candidate elimination (stubble frames f_0074/f_0075).**
Generate 4–5 candidates per frame and eliminate: f_0075's first batch was
rejected wholesale — candidates turned black pants into blue jeans and
blonde hair brown. Only the candidate matching its neighbors on face, hair,
costume **and** composition shipped. And f_0075 itself was spotted by the
fixer, worse than the user-named f_0074: reported first, fixed only after
the user's explicit go.

---

## October 16–19 Casebook — Fills, Static Shots, Locked Plates

### Yanineko & KICK BACK — dynamic fills and the likeness gate
The user set the run-length rule: neighbor-filled runs of **3+ frames must be
rebuilt as moving frames** (each frame slightly different, differences stringing
into action); 1–2 frame fills stay. Yanineko v3 rebuilt 3 runs (35 frames) this
way — and the user still rejected it: **the filled frames' characters didn't
look like the characters**. Motion is not enough. Every transition-generated
frame with a person now passes a three-way check (source | established
same-character anchor | candidate, face zoom included); distorted candidates
are discarded in production, never shipped for the user to catch (KICK BACK v3
discarded runs whose suits shifted blue frame-to-frame, whose chain froze a
face and invented a fist, and one that drew the wrong shot entirely).
Known step-③ deviation types to audit explicitly: costume pattern invented
(plain dirty dress → red patterned), scene tone shifted a grade, small props
sanitized away (a lighter's flame vanished although both endpoint frames had
one). Also: verify segment boundaries against the source itself — one ledger
had a cut point wrong by a frame.
Color fixes are measured, not eyeballed: KICK BACK f_0379 still read black in
v3; v4 drew 10 candidates and picked the one whose measured color (191,166,155)
sat in the source's grey-purple family, clear of the black range — the rest of
the film stayed byte-identical. A face/body fix spanning f_0105–f_0108 was
selected as a group (same narrowed face/neck grade across all four, checked
against f_0109), not frame by frame.

### Chuuka Ichiban & Gurren Lagann — restricted motifs, transitions, trajectories
- The emblem 「特」 belongs **only** to the ending card; the model kept inventing
  it mid-film (dragon tail, pre-ending frames, a whole 3–9s stretch). Rule:
  inventory where a motif may appear; scan the whole film for out-of-segment
  occurrences. Same for a prop (a booklet) that exists only at the ending.
- Treatment is per-segment: one frame (f_0209) had to be pure black-and-white
  while its neighbors are color; near-black transition frames follow their
  standard frame, not the color pipeline's default.
- A source **dissolve** (f_0930–f_0938) was lost — the background jumped.
  Transitions are checked as spans, not frames.
- Gurren's ending moon: 72 frames redrawn from 454 candidates, selection gated
  on the moon's position against the source trajectory (mean deviation 5.1px,
  max 14.3px; over-limit candidates rejected). And a correction with a lesson:
  the "anchor moon shifted 55px" figure reported for v2/v3 was a measurement
  artifact (a bright nebula skewed the reading). **Use one measurement method
  across versions, robust against bright distractors, before reporting a drift.**

### Frieren OP2 — the named range is not the effect's boundary
The user named the title segment to f_0337; full-frame measurement showed the
source title wipe actually runs to **f_0344**. The fix covered f_0284–f_0344
(61 frames) — stopping at the named frame would have left a visible break at
f_0338. Rule: for a continuous effect, verify its true boundary in the source;
the fix covers the effect, and the extension is reported, not silently made.

### Fire Force ED1 — static shots, a waterfall, and a lost prompt
- **Static-shot mode born here**: per-frame coloring made still shots shimmer.
  v5 converted 9 still segments (242 frames) to 1s = 3 drawings; v10 extended
  it to the whole first 13 seconds, choosing anchors by source analysis
  (still/slow-drift seconds take frames 1/5/9; a smoke-transition second
  anchors on the transition frame). Report unique drawings separately from
  output frames; when a siege-blocked anchor is missing, that second honestly
  has 2 unique drawings held longer.
- **Anti-flicker by candidate fit**: four still light-pillar shots (94 frames)
  were fixed by generating multiple candidates per frame and picking by fit
  with the neighbors — no programmatic blending. Cathedral median adjacent
  difference 44.5 → 20.6; the fastest pan pairs stayed the film's highest and
  were disclosed, not hidden.
- **The waterfall (27–30s)**: 33 frames in a bathing-class scene. A silhouette
  version was rejected by the user. What worked: the segment already had ONE
  passing generated frame (f_0335, static camera, only the water moves) — it
  became the locked base; characters/composition/set frozen, only the water
  varies per frame. Final accounting: 20 true generations, 3 dynamic fills
  derived from passing neighbors, 10 static fills — every fill labeled.
- **Sensitive-category carve-out**: of 59 category refusals in this film, only
  one frame (a pale rain silhouette) was recovered, via a step-② brightness
  wording tweak. The other 58 were verified bathing/shower frames: steps ②/③
  would be filter evasion and were **not executed**, despite a standing siege
  order elsewhere. Classify the content before applying any siege.
- **Hard stop honored**: f_0467–f_0471 went through scheduled hourly retries
  until the user said "last attempt, then stop; if all fail, don't change the
  film." Final round: 0/5 (one generated frame QC-killed for a black-block
  face). The film stayed untouched and the retry job was closed.
- **Prompt loss**: the workspace was cleaned with the production and the frozen
  prompt original died with it (hash only). v9's prompt was reconstructed from
  the skill template + surviving spec fragments, validated on standard frame
  f_0609, frozen, saved back, and reported as a reconstruction. Archive method
  assets outside the workspace (§10 of the SKILL).
- **Background unification is measured**: v9 unified the 35th second to anchor
  f_0430 (background diff mean 50.2 → 25.4, residual traced to leaf-hatching
  detail, visually stable); v10 unified the whole 33–39s scene, not just the
  one named second (anchor diff mean 23.96 → 16.17), characters untouched.

### Higurashi — complex motion: direct source or nothing
The underwater-twins rotation (117 frames) failed twice: v6 rebuilt it with
locked uniforms but wrong composition; v7 measured the source (rotation curve,
hair color ≈ RGB 99,145,188 steel-blue) and still drifted too far in pose —
the user: follow the original film, not an approximation. v8 changed method:
3 probe frames (f_0609/f_0660/f_0725) approved by the user, then **direct
img2img of every source frame** — pose, composition, and rotation locked by
the source frame itself; only the style and the uniforms changed. 117/117,
no neighbor copies.
Color locks are **shot-local and measured**: the twins read steel-blue under
water light while Satoko elsewhere is gold — and one segment had been colored
purple because the character was misidentified (thought to be Hanyuu). Verify
who is in the segment before locking its colors.
Trajectory locks work when the object is simple: the 1:28–1:30 sun was redrawn
across 24 frames to the source path (center x ≈ 0.913 constant, y 0.103 →
0.080); the remake had it wandering x 0.72–0.96.
Object **state** is part of fidelity: the flower occluding Rika (f_0426) was
first missing (v8), then restored open and front-facing (v9) — user: it must
be the **unopened bud, back-facing**, as in f_0427. v10 matched that spec
(closed red bud, green star calyx) and the user *still* rejected the frame;
the case was reopened with full 34/35/36-second sheets for the user to point
at what's wrong. Lesson recorded honestly: matching the written spec is not
the same as matching the frame — when a user says "still not right" twice,
stop guessing, put the full-size neighboring seconds in front of them, and let
them point.

### Gundam W/X, FX, Witch from Mercury — dark scenes and locked plates
- Dark scenes are the generator's weak point, three films running: deep-space
  backgrounds came back beige paper with flicker (Gundam X v1 QC FAIL), black
  frames came back with paper borders or not at all (18 black frames lost on
  X; W's pure-black probe frame grew a paper edge), a black frame in another
  production measured luminance 22 → 196 (white). Dark shots get their own
  QC pass (§7.9); "pure black stays pure black" is a checkable rule.
- **Fix the plate, not the frames**: FX's extra staircase was traced to one
  background plate (plate_030 had painted sparse floating steps as a dense
  thick staircase); the 4 generation frames using it were fine. The plate was
  regenerated from the source median, only 4 frames recomposited, and the
  locked-area difference re-measured at exactly 0. FX's full background audit:
  78 same-background groups, locked-area adjacent difference max = 0.
- Restart-truncated PNGs are real: two FX frames were found corrupt at QC.
  Decode-validate every frame after an interruption (§4).
- Aspect changes are full redos: Gundam W shipped at a user-set 3:4, then the
  user ordered W and X both redone at native 4:3 — from source, no cropping or
  stretching of the old frames. Mercury's redo spec is the standing template
  for anti-flicker: scene-split first, one background per scene, one anchor
  frame locking all colors per shot, per-shot measured adjacent color/background
  differences in QC, whole-scene redo over threshold, one locked title card.
- **Superseded (user order, 2026-10-19, later the same day):** the title-card
  lock and the background lock/plate methods recorded in this casebook were
  removed from the SKILL — the W v3 whole-frame title lock froze a standing
  character's pose, and the X v4 plate pass produced the artifacts the user
  rejected. Historical record only: the current method draws title cards and
  backgrounds per frame from their own source frames and checks consistency
  by QC measurement against the source and the scene anchor.

---

## Cross-Cutting Rework Rules

1. **Frame-sheet loop.** The user reviews frame sheets, names frame numbers and
   one **standard frame**; fix exactly the named frames/range against that
   standard. Nothing more.
2. **Exception scope is a hard boundary.** Prompt changes and rule exceptions
   apply only to user-named frames. QC findings outside the scope are reported
   as options — the user decides; "不扩" ends the discussion, gap recorded.
3. **Root cause before remedy.** If a frame cannot be fixed (glare-washed
   source), say why, offer options, let the user choose. Never force it.
4. **Independent QC, four criteria:** character consistency (0.5s extraction vs
   source), no artifacts, object/scene logic consistency, natural restrained
   expressions. FAIL never ships.
5. **Color locks are per-character and written down** before production, since
   independent per-frame coloring drifts (flicker, wrong hair colors).
6. **Verify disk state before reporting progress.** Assembled ≠ reported;
   reported ≠ on disk. Check the artifact.
7. **Every re-encode gets a new filename** (v3→v4→v5, 43→43e) — cached old
   versions must never resurface on the user's phone.
8. **Thumbnails are not evidence.** Any count or verdict about frames comes
   from full-size frames, plus a quantitative audit for overlays, tones,
   and scale.
9. **State on disk, always.** Batch log + frames on disk; a dead worker's
   run is resumed from disk, never restarted from zero.
10. **Grade gaps by run length.** 1–2 frames: neighbor fill, done. 3+ frames:
    rebuild as moving frames or report the run as unresolved — a static
    freeze over a long run is a defect, not a fill.
11. **Classify before you siege.** Sensitive category blocks are not attacked
    with reworded prompts or transition generation; user stop conditions end
    retry jobs on the spot.
12. **Shot book before batch.** Characters verified per segment, colors
    measured in-shot, background references, treatments, restricted motifs, motion class —
    written down first, argued about never.
13. **Preserve the method assets.** Frozen prompt, logs, maps, and shot books
    live outside the deletable workspace; reconstructions are labeled.
14. **Validate files, not counts.** After any interruption, decode-check
    every frame; a truncated file still shows up in `ls`.
15. **Frame sheets: one second per sheet,** 12 frames, red frame numbers,
    persistent directory — that's the format the user reviews in.
