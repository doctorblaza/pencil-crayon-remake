# Style Prompt Template (use verbatim on every frame)

Fill in the bracketed parts once, then freeze the prompt. Every frame of the
batch uses the exact same text.

```
Redraw this anime frame in pencil-sketch and crayon-coloring style:
visible graphite pencil linework with hand-shaded hatching for shadows,
flat textured crayon fills with visible paper grain, [warm/cool] palette
kept close to the source colors.

STRICT FIDELITY — the single most important rule:
keep the exact composition, character poses, facial expressions, body
proportions, costumes, background elements, camera angle, and every visible
detail identical to the source frame. Change ONLY the rendering style into
pencil and crayon. Do NOT add, remove, or invent any detail, however small.
```

## Notes
- Keep it in English; image pipelines follow English style descriptors best.
- If the source has a distinctive look worth preserving (e.g. sunset-orange hair
  that must NOT read as fire), add one line naming it explicitly — then freeze.
- Record the final frozen prompt at the top of the batch log so any rerun or
  QC agent can verify byte-identical reuse.
