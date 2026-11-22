---
name: vox-explainer
description: End-to-end pipeline for Vox-style explainer videos inside Subclip, from one topic prompt to a finished, narrated, captioned and scored film. Script by the agent; voiceover, keyframes, animation and music with Subclip's own AI models (credits, no outside API keys); captions, music mix and export in the Subclip editor (no ffmpeg). Use whenever the creator asks for an explainer video, a Vox-style video, a documentary short, an educational video essay, a "one prompt to video" film, or wants to turn a topic, article or report into a narrated video with subtitles and music.
---

# Vox-Style Explainer Video Pipeline (Subclip edition)

Produce a complete, narrated, captioned, scored explainer video from a single topic, entirely inside Subclip. Every generation uses the creator's Subclip AI credits through Subclip's MCP tools. No outside API key, no Python, no ffmpeg.

Voiceover is generated BEFORE any visuals because narration length drives every timing decision.

```
TOPIC → 1.Script → 2.Voiceover → 3.Keyframes → 4.Animation → 5.Music → 6.Assembly
        (agent)    (Gemini TTS)  (Seedream 5)  (P-Video 2)    (Lyria 3)  (Subclip editor)
```

## Before you start

- **Creator's own generators first.** If the Codex CLI (images) or Grok CLI (images, video) is signed in and working on this computer, ask once whether to use it for keyframes and animation without Subclip credits, unless the creator already chose. See `references/subclip-models.md`.
- Call `list_ai_models` (kinds `image`, `video`, `audio`) and `get_ai_credit_status`. Model ids, durations and prices below were checked in November 2026; trust the live catalog if it differs.
- Quote the credit budget (see `references/subclip-models.md`) and get the creator's go-ahead before Stage 2. A direct request like "make me a 60-second Vox explainer about X" counts as approval for the quoted plan; a much larger bill does not.
- Create one landscape project: `create_project` with `canvasPreset: "landscape"`. Use its `projectId` in every later call.
- Generation tools return an operation id. Check progress with `get_ai_operation` (in-app agents wait for the completion event instead of polling). Never loop tight polls; check every 15 to 30 seconds.
- Run one editing call at a time on the project. Generations may run in parallel; timeline edits may not.

## Stage 1: Script

You write this yourself. No tool call. This stage decides quality more than any other.

Read `references/vox-style-guide.md` (Part 1, Writing) before drafting. Core rules:

1. **Beats.** A beat = one narration paragraph (1 to 3 sentences, 6 to 12 seconds spoken) + one visual. 45 to 60 seconds = 6 to 8 beats.
2. **Vox arc:** cold-open hook → the turn to context ("to understand why, you have to go back to...") → escalating explanation with one throughline → complication → resolution that reframes the hook.
3. **Voice:** concrete numbers, present tense for history, short declarative sentences. No "in this video we will". Write numbers as words.
4. **One idea per beat.**
5. **Facts must be true.** Only state numbers you are confident of; soften or cut the rest.

Keep the plan in this shape (save it with `update_project_director_brief` so later sessions can resume):

```json
{
  "title": "The Box That Shrank the World",
  "style_seed": "mid-century archival photo cutouts, blueprint grid, stencil lettering",
  "accent_color": "#E4572E",
  "voice": { "modelId": "gemini-3.1-flash-tts", "voice": "Charon", "style": "measured, warm, slightly wry documentary narrator" },
  "beats": [
    {
      "id": 1,
      "narration": "In nineteen fifty-six, an old oil tanker leaves Newark with fifty-eight metal boxes on deck.",
      "visual": "Paper cutout of a 1950s tanker with stacked boxes, harbor cranes behind, blueprint grid sky",
      "caption_text": "1956",
      "motion": "slow push-in on the ship, water ripples, gulls drift right"
    }
  ]
}
```

`caption_text` is an optional on-frame title (under 5 words), drawn into the keyframe. Narration captions come from Stage 6.

## Stage 2: Voiceover

One `generate_speech` call per beat, same model and voice for the whole film.

- Default: `modelId: "gemini-3.1-flash-tts"`, `voiceId: "gemini-3.1-flash-tts:Charon"` (or Iapetus, Kore, Sadaltager), `styleInstruction`: "Confident, brisk documentary narrator at about 155 words per minute. Warm and slightly wry. Short natural pauses only." Without a pace instruction Gemini reads at about 100 words a minute and the film runs a third too long. Avoid trailer voices.
- Free option (and the default for test runs): `generate_local_speech` with an on-device voice such as `jean`, then `import_generated_audio_to_timeline`.
- If the creator has not chosen a voice, offer free samples first (`get_voice_samples`) unless they told you to pick, or the project has an approved voice. Alternatives: `elevenlabs-v3`, `speech-2.8-turbo`.
- When each operation completes, place it with `import_speech_to_timeline`. Beat 1 starts at 0.3 s; each next beat starts at the previous beat's end + 0.5 s. Read real durations from the imported audio items (`get_project` or `read_project_section`). These start and end times are the master clock.
- If the total runs long, tighten that beat's text and regenerate only that beat.

## Stage 3: Keyframes (style anchor pattern)

One 16:9 keyframe per beat, all in the same look. Read `references/vox-style-guide.md` (Part 2, Visual Grammar) first.

1. **Anchor.** `generate_ai_media` with `kind: "image"`, `modelId: "seedream-5-lite"`, `aspectRatio: "16:9"`, `resolution: "2K"`. Prompt = beat 1 visual + composition + style suffix with the film's `style_seed` and `accent_color`.
2. **Review the anchor** (`preview_asset` or import and `view_frame`). Regenerate until the look is right. Every other frame inherits it.
3. **Import the anchor** into the project with `import_ai_operation_to_timeline` at beat 1's start time.
4. **Other frames.** Same model, with the anchor item as `sourceAssets: [{ sourceItemId: <anchor item>, role: "reference" }]`. Prompt: "Keep the exact art style, palette, paper-collage treatment and border framing of the reference image. New scene: {beat.visual}. {composition}. {style suffix}". Import each at its beat's start time.
5. Alternative for stronger reference consistency: `nano-banana-2` (up to 14 references).
6. **Check every word in every frame.** Image models misspell and change numbers. A title that must be exact (a number, a name) is safer as a native text layer: generate that frame with no text and add the title in Stage 6.

## Stage 4: Animation

Turn keyframes into short silent clips, then swap each in place. Animation is the biggest cost: for a cheap draft animate 3 consecutive beats from the start and give the rest a slow push-in (free); for the finished film animate every beat. Ask which the creator wants.

- Default (budget): `generate_ai_media` with `kind: "video"`, `modelId: "p-video-2"`, `resolution: "720p"`, `aspectRatio: "16:9"`, `durationSeconds` = beat length + 0.5 s rounded up (1 to 20 s), `generateAudio: false`, `settings: { "save_audio": false }`, `sourceAssets: [{ sourceItemId: <keyframe item>, role: "start_frame" }]`.
- Premium hero beats: `gemini-omni-1.1` (image-to-video, automatic length up to 10 s, about 5x the price).
- Free option: skip AI animation and give each still a slow push-in with `add_media_keyframe` (scale 1.0 → 1.05 across the beat). Good for drafts or tight budgets.
- Motion prompts must be SUBTLE: "Subtle motion graphics animation of this collage: {beat.motion}. All elements stay flat paper cutouts. Slow steady 2D camera. No 3D rotation, no face animation, no text warping, no style change, no camera shake."
- When a clip is ready, `import_ai_operation_to_timeline` with `replaceItemId: <keyframe item>` so it keeps the beat's slot. Then trim or hold it to the beat window.
- Review each clip; regenerate any where text warps or the style breaks. Budget about 15% for retries.

## Stage 5: Music

One instrumental bed for the whole film.

1. **Try Google Lyria first:** `generate_music` with `modelId: "lyria-3-pro"`, `settings: { "mode": "instrumental" }`. Lyria also accepts up to 10 project images as references; pass the anchor keyframe so the score matches the look. Lyria tracks run about 3 minutes.
2. **Judge it** (`preview_asset`): no vocals, steady mood, no big drop in the middle, the instrumentation suits the subject.
3. **If Lyria is not good enough, use MiniMax:** `modelId: "minimax-music-2.6"`, `settings: { "mode": "instrumental", "audio_format": "mp3", "sample_rate": 44100, "bitrate": 256000 }`.

Prompt formula: `{mood} {texture}, {tempo}, {instrumentation}, {arc}, instrumental documentary underscore, no vocals`. Match instruments to the subject.

## Stage 6: Assembly in Subclip

Read `references/subclip-assembly.md` for exact calls. The sequence:

1. **Titles:** add any on-frame title that is not drawn into a keyframe with `add_text_overlay` (see the reference).
2. **Check the timeline:** narration items at their beat times, one clip per beat covering its window with no gaps; `set_project_duration` to the last beat end + 1.5 s.
3. **Captions before music:** `transcribe_project` with `mode: "local"`, `allTimelineClips: true`, `captionPreset: "authority"` (or `bold-clean`), `highlight: "none"`. Whisper hears only the narration at this point, so timing is word-accurate. Fix misheard names with `update_caption_words`.
4. **Music:** `import_ai_operation_to_timeline` at 0, then `update_audio_item` with `volume` 0.18 to 0.22 (about -14 dB under narration), `endTime` = project end, `fadeIn` 1, `fadeOut` 2.5.
5. **Review:** `get_timeline_contact_sheet` with `at: "auto"`, plus `view_frame` on anything doubtful. Check caption sync, music level, style drift, visible loops.
6. **Export** only when the creator asked for the finished film: `export_video` (`mp4`, `1080p`), then `get_export_status` for the file path.

Fix problems at the stage they started (re-voice a beat, regenerate a frame), not with assembly hacks.

## Orchestration notes

- Stages run in order the first time. Afterwards any beat can be redone alone: its voice, frame or clip.
- Start all Stage 4 clips together, then collect them; video takes 1 to 4 minutes each.
- Report credits spent at the end (`get_ai_credit_status`).

## References

- `references/vox-style-guide.md`: writing arc, visual grammar, prompt templates. Read before Stage 1 and Stage 3.
- `references/subclip-models.md`: which Subclip model for each stage, settings, prices, budget formula.
- `references/subclip-assembly.md`: timeline, captions, music mix and export calls.

Based on vox-explainer by Kingsley (MIT), adapted for Subclip.
