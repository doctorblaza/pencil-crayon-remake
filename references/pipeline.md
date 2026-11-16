# Pipeline Commands

## 1. Extract frames (12fps) and audio
```bash
mkdir -p frames
ffmpeg -i src.mp4 -vf "fps=12,scale=1920:1080" frames/f_%04d.png
ffmpeg -i src.mp4 -vn -c:a aac audio.m4a
# frame count sanity: duration_sec * 12
```

## 2. Batch log (keep per run)
```
batch: shibuya-op-b2
frames: 0001-1080 (1080)
style_prompt_sha256: <hash of frozen prompt>
gaps: f_0465 integrity_check_failed -> filled with f_0463
      f_0467 integrity_check_failed -> filled with f_0463
output: out/f_%04d.png
```

## 3. Assemble 12fps master + mux audio + faststart
```bash
# Remove the destination before copying into an assembly directory:
# never overwrite a file in place that may be hard linked to an earlier
# assembly (November 2026: a copy without remove-first polluted earlier
# assemblies). Map streams explicitly: an audio file can carry a hidden
# video stream that default selection picks.
ffmpeg -framerate 12 -i out/f_%04d.png -i audio.m4a \
  -map 0:v:0 -map 1:a:0 \
  -c:v libx264 -pix_fmt yuv420p -crf 18 -c:a aac -movflags faststart \
  final-12fps.mp4
# After assembly, verify earlier assemblies are unchanged, and verify
# output dimensions against the remake frame dimensions, not the source.
# Back up a manifest before patching it; patch the landed-luminance
# column (not source luminance), with Unix line endings.
```

## 4. Optional 24fps delivery (frame duplication, no invented frames)
```bash
ffmpeg -i final-12fps.mp4 -vf "fps=24" -c:v libx264 -pix_fmt yuv420p \
  -c:a copy -movflags faststart final-24fps.mp4
# NOTE: fps=24 on a 12fps source duplicates frames; verify with ffprobe
# that no motion interpolation was applied (check with select='eq(n\,0)' spot frames).
```

## 5. Verify before delivery
```bash
ffprobe -v error -show_entries stream=codec_name,width,height,avg_frame_rate,nb_frames \
  -show_entries format=duration -of default=noprint_wrappers=1 final-12fps.mp4
# faststart check: moov before mdat
python3 -c "
d=open('final-12fps.mp4','rb').read()
print('moov at', d.find(b'moov'), 'mdat at', d.find(b'mdat'))"
# spot frames
ffmpeg -ss 10 -i final-12fps.mp4 -frames:v 1 check-10s.png
ffmpeg -ss 45 -i final-12fps.mp4 -frames:v 1 check-45s.png
ffmpeg -ss 80 -i final-12fps.mp4 -frames:v 1 check-80s.png
```

## 6. QC extraction (every 0.5s vs source)
```bash
ffmpeg -i final-12fps.mp4 -vf fps=2 qc/q_%04d.png
# compare each qc frame side-by-side with the matching source frame
```
