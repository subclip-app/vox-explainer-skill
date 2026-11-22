# Assembly in the Subclip editor

The original skill assembled with ffmpeg. In Subclip the timeline is the assembly: every step below is an editable timeline change, and the creator can open the project and adjust anything.

All calls take the film's `projectId`. Make one editing call at a time.

## 1. Narration (the master clock)

- Cloud voice: `import_speech_to_timeline` with `operationId` and `startTime`.
- Local voice: `generate_local_speech` with `projectId`, then `import_generated_audio_to_timeline` with the returned `projectFilePath` and `startTime`.
- Beat 1 starts at 0.3 s. Each next beat starts at the previous end + 0.5 s. Keep a table of beat start and end times.

## 2. Pictures

- Beat window: from 0.25 s before the beat starts to 0.25 s before the next beat starts (beat 1 from 0, the last beat to the film end).
- `import_ai_operation_to_timeline` (or `import_external_media` for Codex or Grok images) at the window start, then `update_media_item` with `startTime` and `endTime` = the window.
- Animated beats: `import_ai_operation_to_timeline` with `replaceItemId` = the beat's still.
- Still beats: `add_media_keyframe` at the window start (`scale: 1`) and window end (`scale: 1.05`), `easing: "linear"`, for a slow push-in.
- `set_project_duration` = last narration end + 1.5 s.

## 3. On-frame titles (when not drawn into the keyframe)

Look first: `get_timeline_contact_sheet` with `times` across the beat, find the empty paper away from the subject and away from the caption strip at the bottom, and place the title there. Render the same times again after placing. Titles and captions must never overlap or cover the key element.


- `add_text_overlay` with the exact words, `startTime` = beat start + 0.4, `duration` = until the beat window ends, placed in the empty paper area (`x`, `y` in percent).
- `update_text_item`: a bold condensed font, cream or white text, the accent colour as background (like a strip of tape), slight rotation (-3 to 3 degrees). Keep under 5 words.

## 4. Captions

- Run before adding music, so Whisper hears only narration: `transcribe_project` with `mode: "local"`, `language`, `allTimelineClips: true`, `captionPreset: "authority"` (or `bold-clean`), `highlight: "none"`.
- External MCP clients poll `get_transcription_status` with the returned `jobId`.
- Fix misheard names with `update_caption_words`. Vox look: white, bold, bottom centre, at most 2 lines.
- Do not caption all narration in one run: local Whisper can squeeze words at its 20 s chunk boundaries. Caption each narration item with `audioItemId`.
- Set style after the preset: apply the preset first, then `update_caption_style` with `positionY` about 90 and a dark backing (`backgroundColor`, opacity 85) so white text reads on cream paper. Re-applying a preset resets the position.
- Check `at: "caption-pages"` on a contact sheet: captions must sit clear of titles and subjects.

## 5. Music

- `import_ai_operation_to_timeline` at 0.
- `update_audio_item`: `volume` 0.18 to 0.22 (about -14 dB under the voice), `endTime` = film end, `fadeIn` 1, `fadeOut` 2.5.

## 6. Review

- `get_timeline_contact_sheet` with `at: "auto"` (or one time per beat). Check: every beat has a picture, titles match the script, captions sit clear of titles, nothing blank at the end.
- `view_frame` at any doubtful time for a full-size look.

## 7. Export (only when the creator asked for the finished film)

- `list_render_queue` first, so you do not queue the same project twice.
- `export_video` with `format: "mp4"`, `quality: "1080p"`; `get_export_status` for progress and the file path.

## Wrap up

`update_project_director_brief` with the topic, style seed, accent colour, voice and models; `write_session_summary` with what was made, credits spent and any beats worth redoing.
