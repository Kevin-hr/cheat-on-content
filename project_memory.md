## Hard Constraints
- Video must use 9:16 vertical screen resolution
- 3-second opening must include high-impact visuals, rhythmic sound effects, and command-style subtitles
- Cover frame must be the brightest product-reveal frame with centered bold sans-serif text + 4px black outline
- All project files must be placed in `videos/YYYY-MM-DD_<id>_<short>/` directory structure
- Every deliverable must be verified with `ffprobe` before claiming success
- Hashtag count must be exactly 5, ordered by algorithm priority

## Engineering Conventions
- Subtitles follow 0.3-0.6 second switching rhythm with drumbeat alignment
- Subtitle font: bold sans-serif (Arial Black / Impact style), ALL CAPS English, white + 4px black outline
- Subtitle animation: CapCut "jump/bounce" (word-by-word pop-in)
- Cover text: centered, bold sans-serif, white + 4px black outline
- LUT color grading: TikTok Clean Streetwear WarmSkin 33, contrast +25%, saturation +20% for first 3 seconds
- Hashtag limit: exactly 5 tags, ordered by algorithm priority
- Sound effects: Cinematic Whoosh → 808 Kick → Snare → Hi-hat x3 → 808 Sub Drop → Kick+Snare → Riser
- BGM: Phonk / Jersey Club / Drill, 20-25% volume, reduce 30% in first 3 seconds
- Scripts stored as markdown files in scripts/ directory with naming convention YYYY-MM-DD_<hash>_short.md
- Videos stored in videos/ directory with subfolders per video ID
- Predictions stored as immutable logs in predictions/ directory
- Video encoding must use H.264 + AAC codecs, verified with ffprobe

## Lessons Learned
- **PowerShell Copy-Item fails on paths with spaces**: Use `[System.IO.File]::Copy()` instead. Copy-Item silently creates 0KB empty files when source path contains spaces, and the failure is not immediately visible.
- **Always verify video playability with ffprobe**: Never claim a file is "ready" without running `ffprobe -show_entries format=duration` first. A 0KB or corrupted file will pass `Test-Path` but fail playback.
- **Ad-hoc file placement breaks project integrity**: Dropping files in repo root (`recut_video.py`, `render_final.py`, `.mp4` in root) pollutes the workspace and renders deliverables unusable. Always use the `videos/YYYY-MM-DD_<id>_<short>/` structure.
- **Every video must follow the full specification**: Do not skip font specs, LUT specs, sound-effect rhythm tables, or cover-design rules even when "rushing". Missing any of these creates inconsistency across projects and forces rework.
- **Consistent project-wide standards**: All videos in this repo must share the same subtitle font, cover text style, LUT preset, and sound-effect template. When a new video is processed, copy the spec from the most recent successful project and adapt only the content-specific parts.
- **File verification checklist before claiming completion**:
  1. `ffprobe` confirms non-zero duration and valid codec
  2. `cv2.VideoCapture().read()` confirms frame extraction works
  3. Directory contains exactly: `source.mp4`, `recut_final_with_audio.mp4`, `cover.jpg`, `subtitles.srt`, `report.md`
  4. No extra files (zm.srt, .py scripts, duplicate jpgs) in the video folder
