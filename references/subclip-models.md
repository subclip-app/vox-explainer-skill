# Subclip models for each stage

Everything runs through Subclip's MCP tools and the creator's Subclip AI credits. No outside API key. Prices are credits as listed by `list_ai_models` in November 2026; always re-check with `list_ai_models` and `estimate_ai_generation`, the live catalog wins.

## Before any paid stage: the creator's own generators

If the Codex CLI (images) or the Grok CLI (images and video) is signed in and working on this computer, ask once whether to use it instead of Subclip credits, unless the creator already chose. Subclip's official `subclip-ai-generation` skill (`references/creator-cli-generators.md`) has the check, the exact commands and the fallback. Bring each file in with `import_external_media`. Everything else in this file applies when Subclip generates.

## Stage table

| Stage | Default | Alternatives | Price |
|---|---|---|---|
| Voiceover | `generate_speech`, `gemini-3.1-flash-tts`, voice `gemini-3.1-flash-tts:Charon` | Free on-device: `generate_local_speech` (voices `jean`, `marius`, `javert`; `subclip-local-supertonic` for 31 languages). Cloud: `elevenlabs-v3`, `speech-2.8-turbo` | 4.25 / 1,000 characters (a 60 s script is about 4 credits); local is free |
| Keyframes | `generate_ai_media` image, `seedream-5-lite`, 16:9, `2K` | `nano-banana-2` (up to 14 references, 2.5), `ideogram-4` for text-heavy frames (5) | 2.98 per image |
| Animation | `generate_ai_media` video, `p-video-2`, 720p, 1 to 20 s, start frame | `gemini-omni-1.1` (closest to the original Omni Flash, automatic length up to 10 s, 12.75/s), `wan-3` (4.25/s), `hailuo-2-3` (6 or 10 s, 3/s), or free `add_media_keyframe` push-ins | 2.55 per second |
| Music | `generate_music`, `lyria-3-pro`, instrumental, about 3 minutes | `minimax-music-2.6` (12.75), `elevenlabs-music` (exact length, 0.71/s) | 6.8 per track |
| Captions | `transcribe_project`, `mode: "local"` | `mode: "cloud"` | free locally |

## Voiceover notes

- Gemini TTS reads slowly by default (about 100 words a minute). Ask for pace in `styleInstruction`: "Confident, brisk documentary narrator at about 155 words per minute. Warm and slightly wry. Short natural pauses only." That took a 7-beat script from 87 s to 64 s.
- `get_ai_operation` returns `outputAssets[0].durationSeconds`; use it as the beat length. For local speech, the WAV path is returned; its length is in the imported audio item.

## Keyframe notes

- Anchor first, review it, then pass it as `sourceAssets: [{ sourceItemId, role: "reference" }]` for every other frame. The anchor must be on the timeline (or in the project media) to be referenced.
- Image models garble words and change numbers (a test frame read "Don't unxack", another printed -80% where the script said ninety). Check every title against the script. For anything exact, generate the frame with "no text, no letters or numbers; leave clean paper for a title" and add the title as a native text layer (`add_text_overlay`, then `update_text_item` for font, colour and background).

## Animation notes

- Pass `generateAudio: false` and `settings: { "save_audio": false }`; narration and music come later.
- Animating every beat is the biggest cost. For drafts or tight budgets animate the hook, the key idea and the ending (3 beats) and give the rest a slow `add_media_keyframe` push-in (scale 1.0 to 1.05). A 60 s film: 3 clips x 9 s x 2.55 = about 69 credits, against about 150 for all 7.
- Import finished clips with `import_ai_operation_to_timeline` and `replaceItemId` set to the beat's still, so the clip keeps the slot.

## Music notes

- Try Lyria first. Pass the anchor keyframe as an image reference so the score matches the look. Judge it before use: steady arc, no vocals, no mid-track drop. If it is not good, generate with MiniMax.
- You cannot listen to music as an agent. At least check the loudness curve (for example with the exported audio) for sudden drops, and tell the creator to listen.

## Operations

- Generation calls return `generation.id`. Check with `get_ai_operation` every 15 to 30 s; images take about a minute, clips 1 to 4 minutes.
- If an operation shows `processing` with no progress change for more than 5 minutes, start the same generation again and use whichever finishes. Stuck jobs charge nothing.
- If a call fails with "fetch failed" or "Failed to create AI generation", retry once after a few seconds.

## Budget formula

`beats x 3 (frames) + animated seconds x 2.55 + 7 (music) + 4 (cloud voice, 0 if local)`

A 7-beat, 60 s film with 3 animated beats: about 21 + 69 + 7 + 4 = 100 credits. Quote this before the first paid stage.
