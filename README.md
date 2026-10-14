# pencil-crayon-remake

A reproducible skill for remaking anime clips (OPs, EDs, scenes) into
**pencil-sketch + crayon-coloring** style, frame by frame, with strict fidelity
to the original.

## Method in one paragraph

Extract the source at 12fps. Transform every frame with the **same frozen
style prompt** via independent img2img — never chain frames, or drift
accumulates. Fill pipeline-refused frames with neighbors and log them honestly.
Assemble at 12fps (duplicate to 24fps only if needed), mux the original audio,
`faststart`, then have an **independent QC agent** compare 0.5s extractions
against the source before delivery.

## Provenance

Distilled from production runs remaking anime OPs at 1920×1080 / 12fps /
~1080 frames per 90-second clip, including multi-round user QC
(character consistency, artifact checks, frame-number-targeted fixes).

## Contents

- `SKILL.md` — the full workflow and operating rules
- `references/style-prompt-template.md` — the frozen prompt template
- `references/pipeline.md` — ffmpeg commands for extract / assemble / verify / QC
- `references/rework-experience.md` — casebook: watermark handling, title-card continuity, color locks, and rework discipline from the Slam Dunk & Fire Force productions

## The iron rules

1. One prompt, verbatim, every frame.
2. No chaining — each frame from its own source frame.
3. Strict fidelity — style changes, details don't.
4. Gaps logged and neighbor-filled, never hidden.
5. Independent QC; fix only flagged frames; new filename on re-encode.
