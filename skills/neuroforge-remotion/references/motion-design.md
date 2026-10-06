# Motion Design — The Craft

What separates a studio-grade motion piece from "text fades in". API details live in `remotion-dev/`; this file is about choices.

## Contents

1. The principles that matter on screen
2. Easing vocabulary
3. Timing table
4. Overlap, stagger, follow-through
5. Kinetic typography
6. Layout, grids, safe zones
7. Colour and contrast
8. Depth and texture
9. Rhythm and sound
10. The signature moment
11. 3D in video
12. Anti-patterns

---

## 1. The principles that matter on screen

From the classic animation principles, these carry motion graphics:

- **Easing (slow in / slow out)** — nothing physical starts or stops instantly.
- **Anticipation** — a tiny counter-move (scale 1 → 0.96) before a big move makes it land.
- **Follow-through & overlap** — secondary parts keep moving after the lead stops and settle later.
- **Staging** — one clear point of focus per moment; everything else supports or waits.
- **Arcs** — moves along a slight curve look intentional; perfectly straight moves look mechanical.
- **Exaggeration** — push key moments (overshoot, scale punch) so they read at phone size.
- **Secondary action** — a subtle drift, parallax, or shimmer keeps a held frame alive.

## 2. Easing vocabulary

Pick one **primary** curve and one **accent** curve for the whole piece and name them in `src/theme.ts`. Consistency is what makes motion feel designed.

| Name | `Easing.bezier(...)` | Use |
|---|---|---|
| Expo out (primary entrance) | `0.16, 1, 0.3, 1` | UI and type entering — fast start, long luxurious settle |
| Quint out | `0.22, 1, 0.36, 1` | Slightly softer entrance |
| Expo in-out | `0.87, 0, 0.13, 1` | Camera moves and big position changes between rests |
| Quint in | `0.64, 0, 0.78, 0` | Exits — accelerate away |
| Back out (overshoot) | `0.34, 1.56, 0.64, 1` | Stamps, pops, buttons — use sparingly |
| Standard | `0.4, 0, 0.2, 1` | Neutral UI motion |

Physical feel: `spring()` / `Easing.spring({ damping })` — `damping: 200` is a smooth push with no bounce; ~`12` is a lively settle; under ~`8` is cartoon bounce. Prefer springs for things that *arrive*, beziers for things that *travel*.

Linear is correct for only three things: constant rotation, scrolling tickers/marquees, and progress driven by data.

## 3. Timing table (at any fps — think in seconds)

| Motion | Duration |
|---|---|
| Micro (icon tick, colour change) | 0.15–0.25 s |
| Small element enter/exit | 0.3–0.5 s |
| Headline reveal | 0.5–0.8 s |
| Large layout change / camera move | 0.8–1.4 s |
| Hold after a key line lands | ≥ reading time (see `scripting.md`) |
| Logo lock-up hold | ≥ 1 s |
| Stagger between siblings | 2–6 frames @30 (0.06–0.2 s) |

Exits are ~70% of the entrance duration — the viewer has already read it.

## 4. Overlap, stagger, follow-through

- Stagger children by index; cap total stagger so the last item lands within ~0.6 s of the first.
- Lead element first, supporting elements trail. Labels after their shapes; numbers after their bars.
- Exit in reverse order or all together — never in a random order.
- Overlap scenes' motion: the next scene's first element can begin during the last 3–6 frames of the previous exit.

## 5. Kinetic typography

Type carries most great motion graphics.

- **Reveal units:** per line for statements, per word for punchy copy, per character only for short display words. Never per character on a sentence.
- **Mask reveals** (text rising from behind a clip edge) look premium; a plain fade looks cheap. Combine: y from 100% → 0 inside an `overflow: hidden` line + slight opacity.
- **Emphasis by change, not decoration:** a weight shift, a colour swap on the key word, a scale punch, an underline that draws on, a highlighter wipe (`remotion-dev/remotion-markup/text-highlights.md`).
- **Size:** headlines ≥ 80 px at 1080 wide; nothing under 40 px. Tight leading (1.0–1.1) on display type, generous tracking only on small caps.
- **Fit text** to its box with `@remotion/layout-utils` (`measureText`/`fitText`) rather than guessing font sizes — see `remotion-dev/remotion-markup/measuring-text.md`.
- **Load fonts before frame 0** (`@remotion/google-fonts` or local fonts with the upstream pattern) — a fallback font in one frame is a visible flash.

## 6. Layout, grids, safe zones

- Use a 12-column grid with margins of ~6% of width. Derive every position from `useVideoConfig()` width/height.
- **Platform UI safe zones** (approximate, 1080×1920 — platforms change their chrome, so leave margin):
  - Top ~220 px (status bar, account name)
  - Bottom ~420 px (caption, audio, CTA button)
  - Right ~140 px (like/comment/share rail)
  - Left ~60 px
- 16:9: keep text within the centre 90% (title-safe). 1:1 and 4:5: same 6% margins.
- Multi-format: keep copy in the intersection of all safe areas.

## 7. Colour and contrast

- One brand colour does the work; one accent for emphasis; neutrals for everything else.
- Contrast ≥ 4.5:1 for any copy — video compression and phone glare eat contrast.
- Use colour temperature for story: tension scenes cooler/desaturated, release scenes warmer/saturated, or the brand's equivalent.
- Gradients: subtle, two-stop, with a little noise to avoid banding after compression.

## 8. Depth and texture

Choose per scene — never all at once.

- **Parallax:** 2–4 layers moving at 0.3× / 0.6× / 1× of the camera move.
- **Camera push:** scale 1.00 → 1.04–1.08 across a held scene keeps it alive.
- **Soft shadows** and a slight blur on background layers.
- **Grain / noise overlay** at 3–6% opacity, re-seeded per frame with `random()` or `@remotion/noise` — kills banding and adds film texture.
- **Light leaks** on transitions (`remotion-dev/remotion-markup/light-leaks.md`).
- **Motion blur** on fast moves (`remotion-dev/remotion-markup/motion-blur.md`) — expensive at render; only where speed is the point.

## 9. Rhythm and sound

- Find the music BPM; one beat = `60 / BPM` seconds = `fps * 60 / BPM` frames. Put cuts on beats and hits on downbeats.
- Every meaningful visual hit has a sound: whoosh on fast moves, click/tick on UI, impact on stamps, riser into the turn. Keep SFX 6–12 dB under VO.
- Duck music under voiceover; fade music out over the last ~1.5 s or end on a hard button.
- Silence before the reveal makes the reveal louder.
- Audio patterns and visualisation: `remotion-dev/remotion-markup/audio.md`, `sfx.md`, `audio-visualization.md`, `voiceover.md`.

## 10. The signature moment

Every piece plans one shot people remember and spends disproportionate craft on it:

- A **match cut** (a phone screen becomes the next scene's background)
- A **3D product reveal** with a lighting sweep
- A **type morph** (the problem word transforms into the solution word)
- A **data wave** (cells, bars, or dots changing in a choreographed sweep on the beat)
- A **scale-through** transition into the UI

Name it in the scene map. If you cannot name it, the piece doesn't have one yet.

## 11. 3D in video

- Use `@remotion/three` with `<ThreeCanvas width height>`; drive every transform, material uniform, and camera move from `useCurrentFrame()`. **No `useFrame()`, no self-animating shaders or GLTF animations playing on their own** — scrub them by frame (`references/remotion-dev/remotion-markup/3d.md`).
- Light like a product shoot: key + rim + soft fill, or an HDR environment. Contact shadow under the object.
- Slow orbital camera moves (10–25° over the scene), expo in-out.
- Pair with **neuroforge-threejs** for scene-building craft.

## 12. Anti-patterns

- Everything fades in. Everything at once. Everything bounces.
- Linear position moves; identical durations on every element.
- Text on screen shorter than its reading time.
- Five transition styles in one piece.
- A long logo intro before the hook.
- Ending on a fade to black instead of a held CTA.
- Default system fonts, pure #000/#FFF with no tonal range, gradient banding.
- Decoration competing with the copy during the line that matters.
