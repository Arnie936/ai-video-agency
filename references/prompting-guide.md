# Prompting Guide

Battle-tested prompt patterns for each phase of the pipeline.

## Character sheet (GPT Image 2)

Template:

```
Character reference sheet of the person from the photo. Show the same person with
identical facial features from multiple angles: front view, three-quarter view,
profile view, plus a close-up of the face showing their distinct features
(<hair style/color, facial hair, eye color, notable features>). Also include one
full-body version of them as <story character with costume description>. Neutral
grey studio background, clean character sheet layout, photorealistic, consistent
identity across all views.
```

Why it works:
- Naming concrete facial features forces the model to actually encode them instead
  of averaging toward a generic face.
- The in-costume full-body panel means video scenes get face + costume from a
  single reference image.

## Transformation target image (GPT Image 2)

The magic phrase is **"keeping their exact same pose, facial structure, expression
and camera framing"**. Everything you want changed must be stated explicitly
(clothes, props, hair, background); everything unstated tends to stay.

## Morph videos (Seedance 2.0, start+end image)

Structure: current state → trigger → enumerated changes → style.

```
A man in a white t-shirt with headphones sits in front of a black curtain, holding
up his index finger. He snaps his fingers and a magical swirling transformation
ripples over him: his clothes morph into rough animal furs, his hair grows wild and
long, the black curtain dissolves into a rocky torch-lit cave, and his raised hand
now grips a burning wooden torch. Smooth cinematic morph transition, warm firelight
growing, mystical particles, seamless transformation.
```

- The trigger (finger snap) motivates the morph — unmotivated dissolves look cheap.
- Enumerate each element that changes (clothes → X, background → Y, prop appears).
- For the reverse morph, mirror the same list backwards and end with
  "ending exactly on the reference photo pose".

## Story scenes (Seedance 2.0, character sheet reference)

Every scene prompt starts with the identical reference clause:

```
Use the person from the character reference sheet as <character> (their exact face,
<costume>). Scene: <action beats in order>, <camera work>, <lighting/mood keywords>.
```

Scene-writing rules:
- 10 seconds fits 2–3 action beats. Don't cram five.
- Name the camera work (handheld chase, slow push-in, wide-then-close) — Seedance
  follows it well.
- Keep one lighting motif across all scenes (e.g. "warm torchlight against
  darkness") so hard cuts still feel like one film.
- The final scene's last beat: character turns to camera and hits the anchor pose.

## Story structure (3-scene arc)

| Scene | Function | Content pattern |
|---|---|---|
| 1 | Hook & threat | Establish setting, danger appears within seconds, flight/reaction begins |
| 2 | Struggle & turning point | Escape/lowest moment, emotional decision beat |
| 3 | Triumph & full circle | Payoff, celebration, character faces camera in anchor pose |

The hook must raise a question in the first 5 seconds (rumble → what's coming?).
Payoff follows immediately in the same scene — don't defer it to scene 2.

## Title / text transition cards (Seedance 2.0, text-to-video)

Structure: silent empty opening → text flies in with ONE sound accent → silent hold/fade.

```
Cinematic title card on a near-black background with faint drifting embers and
subtle smoke, matching a prehistoric fire theme. The first second is completely
silent and empty. Then the title text 'DER FUNKE' flies in dynamically as glowing
ember-orange letters, assembling from swirling sparks, accompanied by a single deep
cinematic whoosh-impact sound exactly when the text lands. The text holds, gently
flickering like firelight. The final second: total silence again, text slowly
fading. No music, no other sounds — only the one whoosh-impact. Elegant, epic,
minimalist title sequence, 16:9.
```

Why the silence bookends matter: the card sits between two story clips with their
own audio — starting and ending in silence makes the hard cuts inaudible.

Rules:
- Put the exact title in quotes; keep it short (1–3 words render most reliably).
- Match the particle/lighting motif to the film's theme.
- Always verify the rendered spelling on a mid-clip frame before delivering.
