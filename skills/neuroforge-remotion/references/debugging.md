# Debugging Remotion

Load before your second fix attempt. Diagnose mode rules apply: at most five files, two likely causes, one cheapest check, then ask.

## The cheapest checks

- **Scrub to the frame.** Ask the developer for the frame number where it breaks, or open `http://localhost:<port>/<composition-id>` and step frame by frame.
- **Render one still:** `npx remotion still <composition-id> --frame=<n> out.png` — a single frame is seconds, not a full render. Suggest it; it writes a file.
- **Compare Studio vs render** at the same frame. If they differ, something isn't frame-driven or something isn't awaited.

## Symptom → likely causes

| Symptom | Most likely | Then check |
|---|---|---|
| Flicker / jitter in render, smooth in Studio | Animation not driven by `useCurrentFrame()` — CSS transition, Tailwind `animate-*`, `useFrame`, a timer, GLTF animation playing itself | `Math.random()` instead of `random(seed)` |
| Element pops in late / blank first frames of a scene | Media not premounted | Missing `premountFor={fps}`; image not loaded before frame |
| Wrong font in some frames or in render | Font not loaded before render | Font loaded inside a scene instead of module scope; local font path outside `public/` |
| Value overshoots (opacity > 1, negative scale) | `interpolate` without clamping | Easing with overshoot used on opacity |
| Timing wrong after changing fps | Hardcoded frame numbers | Timings not written as `n * fps` |
| Scene ends early / total too short | `TransitionSeries` transitions shorten the timeline | Composition `durationInFrames` not recomputed |
| Audio drifts from visuals | Audio offset relative to the wrong sequence | Trim props on the audio vs. sequence `from` |
| Render hangs then times out | A `delayRender` never continued | A fetch or asset load that fails silently; network asset unreachable from the render machine |
| `window`/`document` undefined | Browser API at module scope or in `calculateMetadata` | Code that runs in Node during bundling/metadata |
| 3D renders black or flickers | No lights / camera inside the object / self-animating shader | `useFrame()` used; textures not loaded before frame |
| Studio edits don't stick in code | Values computed outside JSX, or loops generating instances | See `remotion-dev/remotion-interactivity/REFERENCE.md` |
| Render slow | Motion blur, heavy shadows, huge images, unnecessary `<Video>` decoding | Downscale assets; render concurrency; Lambda for long jobs |

## Rules while debugging

- Instrument before you guess: one `console.log(frame, value)` in the suspect component, scrub, read.
- Two fixes that didn't move the symptom → stop. Say what you know, what you don't, and the one thing that would tell you.
- Never mask a symptom you haven't explained (e.g. adding a fade to hide a pop-in).
- Version-specific API doubt → `remotion-dev/remotion-docs/REFERENCE.md` for the live docs.
