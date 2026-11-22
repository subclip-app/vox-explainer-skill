# vox-explainer (Subclip edition)

A skill for AI agents working in [Subclip](https://subclip.app) that turns one topic into a finished Vox-style explainer: script, voiceover, art-directed paper-collage keyframes, subtle animation, music, captions and export, all inside the Subclip editor.

**One topic in, finished film out. No outside API keys, no ffmpeg.**

```
TOPIC → Script → Voiceover → Keyframes → Animation → Music → Assembly
        (agent)  (Gemini TTS  (Seedream 5  (P-Video 2)  (Lyria 3,   (Subclip
                  or local)    Lite)                     MiniMax)    timeline)
```

Generation runs through Subclip's MCP tools on the creator's Subclip AI credits. If the creator has the Codex CLI or the Grok CLI signed in, the agent offers those first for images (and Grok for video), so they cost no Subclip credits.

## Install

In Subclip: **Skills > Add skill**, paste `https://github.com/subclip-app/vox-explainer-skill`. Or ask the agent: "install the vox-explainer skill from github.com/subclip-app/vox-explainer-skill".

## Use

> Make me a 60-second Vox-style explainer about how the shipping container shrank the world.

The agent writes the script in beats, voices it first (narration length drives all timing), locks the look with one anchor keyframe, makes the other frames from it, animates the hook, the key idea and the ending, scores it with Lyria (MiniMax if Lyria misses), captions it and assembles everything on an editable Subclip timeline.

## What's inside

| File | What it covers |
|---|---|
| `SKILL.md` | The six-stage pipeline in Subclip |
| `references/vox-style-guide.md` | Writing arc, collage visual grammar, prompt templates |
| `references/subclip-models.md` | Which Subclip model per stage, settings, prices, budget formula, Codex and Grok |
| `references/subclip-assembly.md` | Timeline, titles, captions, music mix, review and export calls |

## Cost

A 7-beat, 60-second film with 3 animated beats is about 100 Subclip credits (frames about 21, animation about 69, music about 7, cloud voice about 4 or free with local voice). Animating every beat is about 150.

## Credits

Based on [vox-explainer](https://github.com/CK42BB/vox-explainer-skill) by Kingsley (MIT). The original runs on the Atlas Cloud API with local ffmpeg; this fork moves every step into Subclip.

## License

MIT. See `LICENSE`.
