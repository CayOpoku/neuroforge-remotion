# Review — Sweep Order and the Motion Quality Verdict

Load when reviewing a cut, a scene, or a Remotion codebase. The verdict closes every Tier 2 analysis and every review.

## Sweep order

Review as a viewer first, then as an engineer. A technically clean video nobody finishes has failed.

### 1. Viewer pass (in Studio, at real speed, on mute first)

- **3-second test** — is the hook landing before 3 s? Would you stop scrolling?
- **Mute test** — does the story read with sound off?
- **Pause test** — scrub to five random frames. Does each one make sense on its own?
- **One-message test** — after one viewing, can you say the message in one sentence?
- **Drag test** — note the first second you'd skip. That scene is too long.
- **CTA test** — is there exactly one, concrete, held ≥ 1.5 s?

### 2. Craft pass

- Easing consistent (one primary, one accent); no linear position moves
- Stagger and overlap present; nothing important starts on the same frame as its neighbour
- Text on screen ≥ its reading time; sizes ≥ 40 px at 1080 wide; inside safe zones
- Contrast ≥ 4.5:1 on all copy
- Transitions mean something; ≤ 3 types in the piece
- A signature moment exists and got the extra craft
- Audio: music matched to energy, SFX on hits, no accidental silence, no clipping

### 3. Engineering pass

- Every animated value derives from `useCurrentFrame()`; no CSS/Tailwind animations, `useFrame`, timers, or `Math.random`
- Timings expressed as `n * fps`; `premountFor={fps}` on timed items
- Props typed with a zod schema, `defaultProps` set; copy and colours are props or theme tokens, not literals in scenes
- Fonts loaded before frame 0; assets in `public/` via `staticFile()`
- Media from the package the installed version expects (`@remotion/media` on current versions)
- Layout derived from `useVideoConfig()` dimensions, so other formats don't break
- Strict TypeScript, zero `any`; scene files small and single-purpose
- Matches `references/remotion-dev/` for the installed version

## Finding format

```markdown
### [Severity] Title — `src/scenes/Proof.tsx:42` (frames 240–270)
**What:** The calendar slides in linearly over 10 frames.
**Why it matters:** Reads as mechanical and too fast to register on phone.
**Fix:** EASE.out over 0.6 s; stagger cells 2 frames.
```

Severity: **Critical** (render-breaking, flicker, wrong output) · **High** (message unclear, hook misses, illegible copy) · **Medium** (craft — easing, timing, rhythm) · **Low** (polish, naming).

Rank by consequence. Never pad the list to look thorough.

## The Motion Quality Verdict

Close with this block. Be honest with the number — a strong piece is told it is strong, with reasons; nothing is inflated to be kind.

```markdown
## Motion Quality Verdict — 7/10

| Axis | Score | Note |
|---|---|---|
| Message & script | 8 | Clear one message; CTA is concrete |
| Hook (0–3 s) | 6 | Starts with logo; pain lands at 2.8 s |
| Motion craft | 7 | Good easing; stagger missing in Scene 4 |
| Typography & layout | 8 | Strong type; CTA too close to bottom UI |
| Sound | 5 | No SFX on the stamps |
| Engineering | 8 | Frame-driven, typed; two hardcoded frame counts |

**Biggest win available:** Cut the logo intro; open on the empty calendar.
**Ship-ready?** After the three High findings.
```
