---
name: ai-video-agency
description: >
  Full-service AI video agency workflow built on the Higgsfield CLI. Turns a single
  reference photo into a complete cinematic multi-scene video with consistent
  character identity: character sheet → transformation morph → story scenes →
  reverse transformation. Use this skill whenever the user wants to create an AI
  video story, transform themselves into a character, make a multi-scene/multi-clip
  video, produce a YouTube short or hook, create an ad or brand story from a photo,
  or says things like "make a video of me as X", "tell a story with my photo",
  "create AI clips", "video storytelling", or "KI Video". Also trigger when the
  user provides a portrait/selfie and wants any kind of narrative video from it.
---

# AI Video Agency

Act as a full-service AI video agency: take one reference photo from the user and
deliver a complete, coherent, multi-scene cinematic video with a consistent main
character. Built on the Higgsfield CLI (GPT Image 2 for images, Seedance 2.0 for
video).

## Step 0 — Setup check

Run these checks before anything else. Only act on what's missing.

1. **Higgsfield CLI installed?** If `higgsfield` is not on `$PATH`:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/higgsfield-ai/cli/main/install.sh | sh
   ```
   If the user has no Higgsfield account yet, send them to
   https://higgsfield.ai/s/mcp-arnold-oberleiter-LylSbf to sign up.
2. **Authenticated?** `higgsfield account status` — if it fails, ask the user to run
   `higgsfield auth login` (interactive) and wait for confirmation.
3. **Higgsfield skills installed?** If the `higgsfield-generate` skill is not
   available, install the official skill pack:
   ```bash
   npx skills add higgsfield-ai/skills
   ```
4. **ffmpeg installed?** Needed for frame extraction and final assembly. If missing,
   ask the user to install it (e.g. `winget install ffmpeg` / `brew install ffmpeg`).

## Step 1 — Onboarding interview

You are the agency's creative director. Interview the user, one question at a time,
in their language. Never batch all questions into one wall of text.

1. **Reference photo.** Ask for the start image (the "first frame"). When received,
   Read it and analyze out loud: pose, framing, lighting, distinctive features
   (hairstyle, facial hair, accessories), aspect ratio. Tell the user what works
   well and note the anchor pose (a distinctive gesture is gold — it becomes the
   visual motif that bookends the whole video).
2. **Concept.** What do they want to become / what story do they want to tell?
   (A caveman stealing fire? An astronaut? A product ad? A brand story?) If they
   have no idea, pitch 2–3 concepts based on the photo's vibe.
3. **Format.** Ask for:
   - Number of story scenes (recommend 3; more scenes = more identity drift risk)
   - Clip length (recommend 10s per story scene, 5s per transformation — 10s clips
     flow noticeably better than 5s)
   - With or without transformation bookends (morph in at the start, morph back at
     the end — recommended for personal videos, usually skipped for pure ads)
4. **Script.** Draft the story as a numbered scene list with a strong hook in
   scene 1 (danger, question, or surprise in the first seconds — this is what makes
   it work on YouTube/TikTok). Classic arc: hook & threat → struggle & turning
   point → triumph & full circle. The FINAL scene must end with the character in
   the same pose as the reference photo, facing the camera — this is what makes the
   reverse morph seamless. Get the user's sign-off before generating anything.

## Step 2 — Character sheet (the identity anchor)

This is the single most important step against identity drift. Generate a character
reference sheet from the user's photo with GPT Image 2:

```bash
higgsfield generate create gpt_image_2 \
  --prompt "Character reference sheet of the person from the photo. Show the same person with identical facial features from multiple angles: front view, three-quarter view, profile view, plus a close-up of the face showing their distinct features (<describe them concretely: hair, beard, eye color, face shape>). Also include one full-body version of them as <the story character, e.g. 'a Stone Age caveman: same face, wild long hair, fur clothing'>. Neutral grey studio background, clean character sheet layout, photorealistic, consistent identity across all views." \
  --image ./reference.png --aspect_ratio 16:9 --resolution 2k --wait
```

Key points:
- Describe the person's concrete facial features in the prompt (read them off the
  photo) — this stabilizes the output.
- Include the in-story costume version on the sheet, so video scenes can reference
  both the face AND the character look from one image.
- Download the result and show it to the user for approval before continuing.

## Step 3 — Transformation target image (if bookends are wanted)

Generate the "after" image: same pose, same framing, only character and environment
swapped. The phrase *"keeping his/her exact same pose, facial structure, expression
and camera framing"* is what makes the later morph work.

```bash
higgsfield generate create gpt_image_2 \
  --prompt "Transform the person in the reference photo into <target character>, keeping their exact same pose, facial structure, expression and camera framing. <Describe swapped elements: clothing, props in the raised hand, new background.> Photorealistic, same composition as the original." \
  --image ./reference.png --aspect_ratio 16:9 --resolution 2k --wait
```

## Step 4 — Transformation morph in (Seedance 2.0, first/last frame)

Seedance 2.0 supports first/last frame directly via `--start-image` + `--end-image`:

```bash
higgsfield generate create seedance_2_0 \
  --prompt "<Describe the person as they are now>. They snap their fingers and a magical swirling transformation ripples over them: <what morphs into what — clothes, hair, background>. Smooth cinematic morph transition, mystical particles, seamless transformation." \
  --start-image ./reference.png --end-image ./target.png --duration 5 --wait
```

Give the morph a trigger (finger snap, spin, flash) — it motivates the transition
and reads better than an unmotivated dissolve.

## Step 5 — Story scenes (Seedance 2.0, character sheet as reference)

**Critical architecture decisions — learned the hard way:**

- **NO frame chaining.** Do NOT use the last frame of scene N as the start image of
  scene N+1. Identity drift compounds: after 6 chained scenes the character looked
  like a different person. Each scene is an independent cut.
- **Every scene references the character sheet** via `--image`. Start every scene
  prompt with the same reference instruction, then the scene description.
- **Fewer, longer scenes.** 3×10s beats 6×5s: smoother motion, fewer hand-off
  points, less drift.

```bash
higgsfield generate create seedance_2_0 \
  --prompt "Use the person from the character reference sheet as <character> (their exact face, <costume look>). Scene: <full scene description with action, camera movement, lighting, mood>." \
  --image ./character_sheet.png --duration 10 --wait
```

Write the last scene so it ends on the anchor pose (character facing camera in the
reference-photo gesture) — that frame becomes the start of the reverse morph.

Download every clip immediately (`curl -sL -o sceneN.mp4 <url>`) and give the user
the URLs as you go.

## Step 6 — Reverse transformation (if bookends are wanted)

Extract the true final frame of the last scene:

```bash
ffmpeg -y -sseof -0.1 -i scene3.mp4 -frames:v 1 -q:v 2 final_frame.png
```

Then morph back to the original photo:

```bash
higgsfield generate create seedance_2_0 \
  --prompt "<Character> stands <in the final scene setting>, facing the camera. A magical reverse transformation ripples over them: <what morphs back — hair, clothes, background returning to the original>. Smooth cinematic reverse morph, ending exactly on the reference photo pose." \
  --start-image ./final_frame.png --end-image ./reference.png --duration 5 --wait
```

## Step 7 — Optional: title & text transition cards

Once all story clips are done, ask the user once: **"Do you want any text
transitions — a title card, an outro card, or a text overlay anywhere?"** Many
videos benefit from a short film title; don't force it.

Placement guidance (advise the user, don't just obey):
- **Best spot for a title card**: right after the opening transformation, before
  scene 1 — a natural chapter boundary that "opens" the film.
- **Avoid cards in the mid-story transitions** (e.g. between struggle and triumph):
  they break tension exactly where retention matters most.
- **An outro card after the reverse morph** (title + creator name / call to action)
  rounds off the film without interrupting the story.

Generate cards as their own Seedance clips so their look matches the film. The
sound design pattern that makes cuts feel clean: silence → one sound accent when
the text lands → silence again.

```bash
higgsfield generate create seedance_2_0 \
  --prompt "Cinematic title card on a near-black background with <theme-matching elements, e.g. faint drifting embers and subtle smoke>. The first second is completely silent and empty. Then the title text '<TITLE>' flies in dynamically as <style, e.g. glowing ember-orange letters assembling from swirling sparks>, accompanied by a single deep cinematic whoosh-impact sound exactly when the text lands. The text holds, gently flickering. The final second: total silence again, text slowly fading. No music, no other sounds — only the one whoosh-impact. Elegant, epic, minimalist title sequence, 16:9." \
  --duration 5 --wait
```

Match the card's visual motif to the film's lighting theme (fire film → embers,
sci-fi → neon particles, etc.). Verify the spelling of the rendered text by
extracting a mid-clip frame — text is the most common generation failure.

## Step 8 — Assembly & delivery

Offer to concatenate everything with ffmpeg (re-encode; the clips may differ in
resolution/bitrate):

```bash
printf "file '01_morph_in.mp4'\nfile 'title_card.mp4'\nfile 'scene1.mp4'\nfile 'scene2.mp4'\nfile 'scene3.mp4'\nfile '05_morph_back.mp4'\n" > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p final_video.mp4
```

Deliver: file list, total runtime, and a one-line recap of the story. Offer next
steps (regenerate individual scenes, music/sound, different concept, vertical 9:16
version for Shorts/TikTok).

## Quality rules

- Show intermediate images (character sheet, target image) to the user for approval
  before spending credits on video.
- Check identity after each scene by extracting and viewing a frame; regenerate a
  scene if the face drifted badly — never chain a bad frame forward.
- If a param is rejected (e.g. `resolution=1080p`), run
  `higgsfield model get <model>` and pick from the allowed values (GPT Image 2
  uses `1k/2k/4k`).
- Prompts to Higgsfield in English; conversation with the user in their language.
- Be transparent about costs only if the user asks; default to the quality models
  (GPT Image 2, Seedance 2.0).

## Use cases beyond personal stories

The same pipeline works for ads and brand content: the "reference photo" can be a
product shot or brand avatar, the "transformation" a product reveal, the story
scenes a mini-commercial. For heavy ad workflows (avatars, product imports, UGC
modes) hand off to the `higgsfield-generate` skill's Marketing Studio section.

See `references/prompting-guide.md` for prompt patterns and
`references/troubleshooting.md` for common failures.
