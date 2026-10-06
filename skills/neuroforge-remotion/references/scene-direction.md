# Scene Direction

Turning an approved script into scenes a developer can build frame-accurately and a viewer understands on mute.

## Contents

1. Scene principles
2. The scene map
3. Scene file format
4. Transitions as storytelling
5. Multi-format planning

---

## 1. Scene principles

- **One job per scene.** If a scene does two things, it is two scenes.
- **Something changes every scene.** Ask: *what changed from the last moment?* If nothing, cut it. Change at least one of: background, layout, scale/camera, motion direction, colour temperature.
- **One idea per frame.** One statement, one supporting visual. Pause on any frame — it should still make sense.
- **Motion directs attention.** Movement introduces, scale ranks, timing sequences. If everything moves, nothing reads.
- **Cause → effect.** "This happens… which causes this… so now this." Scenes connect; they don't just follow.
- **Tension → release rhythm.** Problem scenes are tighter, darker, more constrained; release scenes open up — more space, more light, wider framing.
- **End with momentum.** The ending launches (CTA hits on an accent, logo holds), it doesn't fade to black.

## 2. The scene map

`scenes/01-scene-breakdown.md` starts with the whole piece on one table:

```markdown
| # | Name | Frames @30 | Seconds | Job | T/R | Transition out |
|---|---|---|---|---|---|---|
| 1 | Empty calendar | 0–75 | 0.0–2.5 | Hook: the pain, instantly | T | hard cut on hit |
| 2 | Weeks greying | 75–180 | 2.5–6.0 | Deepen the pain | T | push left |
| 3 | The turn | 180–240 | 6.0–8.0 | "Then this changed." | → | light leak overlay |
| 4 | Calendar fills | 240–600 | 8.0–20.0 | Proof: show the product | R | match cut on card |
| 5 | CTA | 600–900 | 20.0–30.0 | One directive + logo | R | — (hold) |

**Signature moment:** Scene 4 — the calendar cells flip green in a wave timed to the beat.
**Palette per beat:** T scenes desaturated cool; R scenes brand blue, full light.
**Music:** 120 BPM → one beat = 15 frames @30 fps. Cuts on beats; accents on downbeats.
```

Frames are for the human reader; in code every timing is `n * fps`.

## 3. Scene file format

```markdown
## Scene 4 — Calendar fills

**Frames:** 240–600 (8.0–20.0 s @ 30 fps)
**Job:** Prove the outcome by showing the real product fill up.
**Tension/Release:** Release.

### Copy
"Booked. Booked. Booked." → "In 7 days."

### Visual direction
- Background: brand blue gradient, soft vignette
- Layout: calendar UI centred at 80% width, 3D tilt 12°, copy top third
- Key elements: calendar screenshot as layered cells, cursor, toast notifications

### Motion direction
- Calendar enters: scale 0.92 → 1, rotateX 18° → 12°, ease-out (0.16, 1, 0.3, 1), 0.6 s
- Cells flip green in a diagonal wave, 2-frame stagger, on the beat
- "Booked." stamps: scale 1.3 → 1 with overshoot, one per downbeat
- Slow camera push 1.00 → 1.06 across the whole scene
- Attention anchor: the wave front → then the "In 7 days." line

### Audio
- Music lifts (drop) at frame 240; soft "tick" per cell group, "stamp" per Booked

### Remotion notes
- `<Series.Sequence>` with `premountFor={fps}`
- Cells: one `CalendarCell` component, delay = (row + col) * 2 frames
- Screenshot layers in `public/scenes/calendar/`
```

## 4. Transitions as storytelling

A transition says how two scenes relate. Choose by meaning, not by novelty — and use two or three types per piece, not ten.

| Relationship | Transition |
|---|---|
| Continuation, same idea | Hard cut on the beat |
| Next step / progression | Directional push or slide (keep one direction for "forward") |
| Transformation, before → after | Match cut, morph, wipe |
| Time passing / mood shift | Crossfade, light leak overlay |
| Zooming into detail | Scale-through (zoom into an element that becomes the next scene) |
| Impact / energy | Flash frame, whip with motion blur |

Implement with `<TransitionSeries>` (`references/remotion-dev/remotion-markup/transitions.md`). Remember a transition shortens the timeline by its duration; an overlay does not.

## 5. Multi-format planning

Plan formats in the scene map, not after the build.

- Build from the **tallest-constraint format first** (usually 9:16), with layout driven by `useVideoConfig()` width/height — never fixed pixel positions.
- Keep copy and key visuals inside the **shared safe area** of all target formats (see `references/motion-design.md`).
- One composition per format sharing the same scene components and props; or one composition with `calculateMetadata` setting dimensions from a `format` prop.
