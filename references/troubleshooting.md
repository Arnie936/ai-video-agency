# Troubleshooting

## Identity drift (character stops looking like the user)

Symptoms: after several scenes the face is more muscular / older / generic.

Causes & fixes:
- **Frame chaining** (last frame of scene N as start image of N+1) compounds drift.
  Fix: independent scenes, each referencing the character sheet via `--image`.
- Weak reference: regenerate the character sheet with concrete facial features
  spelled out in the prompt.
- Too many scenes: prefer 3×10s over 6×5s.
- If one scene drifted badly, regenerate just that scene — never chain a drifted
  frame forward into a morph.

## CLI errors

| Error | Fix |
|---|---|
| `Invalid values: resolution=1080p (allowed: 1k,2k,4k)` | GPT Image 2 uses `1k/2k/4k`, not video-style resolutions |
| `Session expired` / `Not authenticated` | User runs `higgsfield auth login` |
| `Unknown params: <flag>` | Run `higgsfield model get <model>` and use only declared params |
| `Missing required params: prompt` | Ask the user what they want |
| Timeout on `--wait` | Add `--wait-timeout 20m`; 10s Seedance clips can take several minutes |

## Morph looks bad / jumps

- Start and end image must share pose and framing. If the target image changed the
  pose, regenerate it with "keeping their exact same pose ... and camera framing".
- The final story scene must end near the anchor pose; extract the real last frame
  with `ffmpeg -sseof -0.1` (not a random mid-frame) as the morph start.
- Enumerate transformations in the prompt; vague "transforms into X" produces
  cross-dissolves instead of morphs.

## Assembly issues

Clips from different jobs can differ in resolution/fps. Always re-encode on concat
(`-c:v libx264 -crf 18`) instead of `-c copy`.

## Frame extraction

```bash
# true last frame
ffmpeg -y -sseof -0.1 -i clip.mp4 -frames:v 1 -q:v 2 last.png
# first frame
ffmpeg -y -i clip.mp4 -frames:v 1 first.png
```
