# Motion Recipes

Copy-paste building blocks for studio-grade scenes. They follow the upstream rules in `remotion-dev/remotion-markup/REFERENCE.md` — if the installed Remotion version disagrees with anything here, upstream wins.

All recipes are frame-driven (hard stop 1) and time in seconds via `fps`.

## Contents

1. Theme — easings and tokens
2. Clamped interpolation helper
3. Line mask reveal
4. Word stagger
5. Stamp with overshoot
6. Counter
7. Camera push + parallax
8. Grain overlay
9. Beat grid
10. Scene timeline with `Series`
11. Transitions with `TransitionSeries`
12. 3D hero with `ThreeCanvas`
13. Scene component template

---

## 1. Theme — easings and tokens

One file, imported everywhere. The brand lives here, not in scenes.

```ts
// src/theme.ts
import { Easing } from "remotion";

export const EASE = {
  out: Easing.bezier(0.16, 1, 0.3, 1),      // primary entrance
  inOut: Easing.bezier(0.87, 0, 0.13, 1),   // camera + big moves
  in: Easing.bezier(0.64, 0, 0.78, 0),      // exits
  pop: Easing.bezier(0.34, 1.56, 0.64, 1),  // accents only
} as const;

export const COLOR = {
  ink: "#0A0A0A",
  paper: "#F5F3EE",
  brand: "#0B84F3",
  accent: "#FFB800",
} as const;
```

## 2. Clamped interpolation helper

`interpolate` does not clamp by default. A tiny helper keeps scenes readable.

```ts
// src/lib/progress.ts
import { interpolate } from "remotion";

// 0→1 over [start, start + duration] frames, clamped both ends
export const progress = (
  frame: number,
  start: number,
  duration: number,
  easing?: (t: number) => number,
) =>
  interpolate(frame, [start, start + duration], [0, 1], {
    easing,
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
```

For values that should be editable in Studio, keep `interpolate(...)` inline in the `style` prop instead (see `remotion-dev/remotion-markup/timing.md`).

## 3. Line mask reveal

The premium alternative to a fade: the line rises from behind its own clip edge.

```tsx
import { useCurrentFrame, useVideoConfig } from "remotion";
import { EASE } from "../theme";
import { progress } from "../lib/progress";

export const LineReveal: React.FC<{ children: React.ReactNode; delay?: number }> = ({
  children,
  delay = 0,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const p = progress(frame, delay, 0.7 * fps, EASE.out);

  return (
    <div style={{ overflow: "hidden", paddingBottom: "0.08em" }}>
      <div style={{ translate: `0 ${(1 - p) * 110}%` }}>{children}</div>
    </div>
  );
};
```

## 4. Word stagger

```tsx
export const WordStagger: React.FC<{ text: string; stagger?: number; style?: React.CSSProperties }> = ({
  text,
  stagger = 3,
  style,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <div style={{ display: "flex", flexWrap: "wrap", gap: "0.25em", ...style }}>
      {text.split(" ").map((word, i) => {
        const p = progress(frame, i * stagger, 0.5 * fps, EASE.out);
        return (
          <span key={i} style={{ display: "inline-block", overflow: "hidden" }}>
            <span style={{ display: "inline-block", translate: `0 ${(1 - p) * 100}%`, opacity: p }}>
              {word}
            </span>
          </span>
        );
      })}
    </div>
  );
};
```

Cap the total: with many words, lower `stagger` so the last word lands within ~0.6 s of the first.

## 5. Stamp with overshoot

```tsx
const s = spring({ frame: frame - delay, fps, config: { damping: 12, stiffness: 180 } });
// scale from 1.4 down to 1 with a settle
<div style={{ scale: 1.4 - 0.4 * s, opacity: Math.min(1, s * 2) }}>Booked.</div>
```

Put a sound effect on the same frame (`remotion-dev/remotion-markup/sfx.md`).

## 6. Counter

```tsx
const value = interpolate(frame, [0, 1.5 * fps], [0, 12840], {
  easing: EASE.out,
  extrapolateLeft: "clamp",
  extrapolateRight: "clamp",
});

<span style={{ fontVariantNumeric: "tabular-nums" }}>
  {Math.round(value).toLocaleString("en-US")}
</span>
```

`tabular-nums` stops the digits jittering sideways as they change. Pass an explicit locale so the render machine's locale can't change the output.

## 7. Camera push + parallax

```tsx
import { AbsoluteFill, interpolate, useCurrentFrame, useVideoConfig } from "remotion";

export const ParallaxLayer: React.FC<{ depth: number; children: React.ReactNode }> = ({
  depth,
  children,
}) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();
  // whole-scene push; deeper layers move less
  const push = interpolate(frame, [0, durationInFrames], [1, 1.08]);
  const drift = interpolate(frame, [0, durationInFrames], [0, -40]);

  return (
    <AbsoluteFill style={{ scale: 1 + (push - 1) * depth, translate: `${drift * depth}px 0` }}>
      {children}
    </AbsoluteFill>
  );
};

// <ParallaxLayer depth={0.3}><Background /></ParallaxLayer>
// <ParallaxLayer depth={0.6}><Device /></ParallaxLayer>
// <ParallaxLayer depth={1}><Headline /></ParallaxLayer>
```

Inside a `<Sequence>`, `durationInFrames` from `useVideoConfig()` is the sequence's duration — the push spans the scene.

## 8. Grain overlay

Deterministic per frame, so render and preview match. Use one per composition (fixed filter id).

```tsx
import { AbsoluteFill, useCurrentFrame } from "remotion";

export const Grain: React.FC<{ opacity?: number }> = ({ opacity = 0.05 }) => {
  const frame = useCurrentFrame();
  return (
    <AbsoluteFill style={{ opacity, mixBlendMode: "overlay", pointerEvents: "none" }}>
      <svg width="100%" height="100%">
        <filter id="nf-grain">
          <feTurbulence type="fractalNoise" baseFrequency="0.85" numOctaves={2} seed={frame % 12} stitchTiles="stitch" />
        </filter>
        <rect width="100%" height="100%" filter="url(#nf-grain)" />
      </svg>
    </AbsoluteFill>
  );
};
```

## 9. Beat grid

```ts
// src/lib/beat.ts
export const beatFrames = (bpm: number, fps: number) => (fps * 60) / bpm;
export const onBeat = (beat: number, bpm: number, fps: number) => Math.round(beat * beatFrames(bpm, fps));

// cut scene 2 on beat 8 of a 120 BPM track: onBeat(8, 120, fps)
```

## 10. Scene timeline with `Series`

```tsx
import { AbsoluteFill, Series, useVideoConfig } from "remotion";

export const Ad: React.FC<AdProps> = (props) => {
  const { fps } = useVideoConfig();
  return (
    <AbsoluteFill style={{ backgroundColor: COLOR.ink }}>
      <Series>
        <Series.Sequence durationInFrames={2.5 * fps} premountFor={fps}>
          <HookScene headline={props.hook} />
        </Series.Sequence>
        <Series.Sequence durationInFrames={3.5 * fps} premountFor={fps}>
          <ProblemScene />
        </Series.Sequence>
        <Series.Sequence durationInFrames={12 * fps} premountFor={fps}>
          <ProofScene />
        </Series.Sequence>
        <Series.Sequence durationInFrames={5 * fps} premountFor={fps}>
          <CtaScene cta={props.cta} />
        </Series.Sequence>
      </Series>
      <Grain />
    </AbsoluteFill>
  );
};
```

## 11. Transitions with `TransitionSeries`

```tsx
import { TransitionSeries, springTiming, linearTiming } from "@remotion/transitions";
import { slide } from "@remotion/transitions/slide";
import { fade } from "@remotion/transitions/fade";

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={3 * fps} premountFor={fps}>
    <ProblemScene />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={slide({ direction: "from-right" })}
    timing={springTiming({ config: { damping: 200 }, durationInFrames: 0.6 * fps })}
  />
  <TransitionSeries.Sequence durationInFrames={4 * fps} premountFor={fps}>
    <SolutionScene />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition presentation={fade()} timing={linearTiming({ durationInFrames: 0.4 * fps })} />
  <TransitionSeries.Sequence durationInFrames={3 * fps} premountFor={fps}>
    <CtaScene />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

Each transition shortens the total by its duration — account for it in `durationInFrames` on the composition (or compute it in `calculateMetadata`). Full API: `remotion-dev/remotion-markup/transitions.md`.

## 12. 3D hero with `ThreeCanvas`

```tsx
import { ThreeCanvas } from "@remotion/three";
import { Easing, interpolate, useCurrentFrame, useVideoConfig } from "remotion";

export const ProductHero: React.FC = () => {
  const frame = useCurrentFrame();
  const { width, height, fps } = useVideoConfig();

  // slow orbital move, frame-driven — never useFrame()
  const orbit = interpolate(frame, [0, 4 * fps], [-0.35, 0.35], {
    easing: Easing.bezier(0.87, 0, 0.13, 1),
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });

  return (
    <ThreeCanvas width={width} height={height} camera={{ fov: 30, position: [0, 0.4, 6] }}>
      <ambientLight intensity={0.25} />
      <directionalLight position={[4, 5, 3]} intensity={2.2} />   {/* key */}
      <directionalLight position={[-5, 2, -4]} intensity={1.4} /> {/* rim */}
      <group rotation={[0, orbit, 0]}>
        <mesh>
          <boxGeometry args={[1.6, 3.2, 0.2]} />
          <meshStandardMaterial color="#111" metalness={0.6} roughness={0.25} />
        </mesh>
      </group>
    </ThreeCanvas>
  );
};
```

GLTF models, textures, and Suspense inside `<ThreeCanvas>`: follow `remotion-dev/remotion-markup/3d.md`.

## 13. Scene component template

```tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from "remotion";
import { COLOR, EASE } from "../theme";
import { progress } from "../lib/progress";
import { LineReveal } from "../components/LineReveal";

type HookSceneProps = { headline: string; sub?: string };

export const HookScene: React.FC<HookSceneProps> = ({ headline, sub }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames, width } = useVideoConfig();
  const exit = progress(frame, durationInFrames - 0.35 * fps, 0.35 * fps, EASE.in);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: COLOR.ink,
        justifyContent: "center",
        padding: width * 0.08,
        opacity: 1 - exit,
        translate: `0 ${-exit * 40}px`,
      }}
    >
      <LineReveal>
        <h1 style={{ color: COLOR.paper, fontSize: width * 0.1, lineHeight: 1.02, margin: 0 }}>{headline}</h1>
      </LineReveal>
      {sub ? (
        <LineReveal delay={6}>
          <p style={{ color: COLOR.brand, fontSize: width * 0.045, margin: 0 }}>{sub}</p>
        </LineReveal>
      ) : null}
    </AbsoluteFill>
  );
};
```

Sizes derive from `width`, so the same scene works at 1080×1920 and 1920×1080.
