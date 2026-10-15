# Rework Experience — Slam Dunk & Fire Force

Casebook from two pencil-crayon B-track productions (12fps, per-frame independent img2img)
that went through multiple user QC rounds. The fixes below are the expensive lessons;
read this before starting a new remake, and again before any rework round.

- **Slam Dunk OP** (103.25s, 1239 frames): v2 deleted → v3 QC FAIL → v4 PASS;
  4:3 line (1440×1080) 43 → 43b → 43c → 43d → **43e final**, published.
- **Fire Force OP** (89s, 1068 frames, 1920×1080): v1 → v2/v3/v4 (title-card
  continuity, three rounds) → v5–v8 (skin-tone & scale chain).
- **IRIS OUT** (MV 153s/1838 frames; OP 149s/1788 frames): blocked-frame
  three-step siege, letterbox normalization, candidate-elimination fixes.

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
