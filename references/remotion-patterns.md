# Remotion Patterns Reference

Copy-paste ready patterns for production Remotion compositions.
Always check `https://skills.sh/remotion-dev/skills/remotion-best-practices` for the latest — this file is a local cache of the most stable patterns.

---

## Table of Contents

1. [Project Entry Point](#1-project-entry-point)
2. [useCurrentFrame + useVideoConfig](#2-usecurrentframe--usevideoconfig)
3. [interpolate — with clamp](#3-interpolate--with-clamp)
4. [spring — physical motion](#4-spring--physical-motion)
5. [Sequence — scene timing](#5-sequence--scene-timing)
6. [AbsoluteFill — base layer](#6-absolutefill--base-layer)
7. [Audio + staticFile](#7-audio--staticfile)
8. [Img + Video assets](#8-img--video-assets)
9. [OffthreadVideo](#9-offthreadvideo)
10. [Fonts](#10-fonts)
11. [Scene component template](#11-scene-component-template)
12. [Full composition template](#12-full-composition-template)
13. [Common animation recipes](#13-common-animation-recipes)

---

## 1. Project Entry Point

```tsx
// src/index.ts
import { registerRoot } from "remotion";
import { RemotionRoot } from "./Root";

registerRoot(RemotionRoot);
```

```tsx
// src/Root.tsx
import { Composition } from "remotion";
import { MyVideo } from "./MyVideo";

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="MyVideo"
        component={MyVideo}
        durationInFrames={30 * 45} // 45 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          headline: "Default text",
        }}
      />
    </>
  );
};
```

---

## 2. useCurrentFrame + useVideoConfig

```tsx
import { useCurrentFrame, useVideoConfig } from "remotion";

const MyScene: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames, width, height } = useVideoConfig();

  // Always derive time from fps, never hardcode frame counts
  const secondsElapsed = frame / fps;

  return <div>{secondsElapsed.toFixed(2)}s</div>;
};
```

---

## 3. interpolate — with clamp

```tsx
import { interpolate, useCurrentFrame } from "remotion";

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

// ✅ Always clamp both ends
const opacity = interpolate(
  frame,
  [0, fps * 0.5], // input range: 0 to 0.5s
  [0, 1], // output range: transparent to opaque
  {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  },
);

// ✅ Slide up entrance
const translateY = interpolate(frame, [0, fps * 0.4], [40, 0], {
  extrapolateRight: "clamp",
});

// ✅ Delayed entrance (starts at 0.5s)
const delayedOpacity = interpolate(frame, [fps * 0.5, fps * 1], [0, 1], {
  extrapolateRight: "clamp",
  extrapolateLeft: "clamp",
});
```

---

## 4. spring — physical motion

```tsx
import { spring, useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

// ✅ Default spring — smooth and natural
const scale = spring({
  frame,
  fps,
  config: {
    damping: 12, // higher = less bounce
    stiffness: 100, // higher = faster
    mass: 1,
  },
});

// ✅ Delayed spring
const delayedScale = spring({
  frame: frame - fps * 0.3, // delay by 0.3s
  fps,
  config: { damping: 14 },
  durationInFrames: fps * 0.5,
});

// ✅ Bouncy spring (for emphasis)
const bouncy = spring({
  frame,
  fps,
  config: { damping: 8, stiffness: 150 },
});
```

**spring config presets:**

- Smooth entrance: `{ damping: 12, stiffness: 100 }`
- Snappy: `{ damping: 20, stiffness: 200 }`
- Bouncy: `{ damping: 8, stiffness: 150 }`
- Very slow: `{ damping: 30, stiffness: 40 }`

---

## 5. Sequence — scene timing

```tsx
import { Sequence } from "remotion";

// ✅ Scenes laid out in time
const MyVideo: React.FC = () => {
  const { fps } = useVideoConfig();

  return (
    <>
      {/* Hook: 0–3s */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <HookScene />
      </Sequence>

      {/* Problem: 3–8s */}
      <Sequence from={fps * 3} durationInFrames={fps * 5}>
        <ProblemScene />
      </Sequence>

      {/* Agitation: 8–15s */}
      <Sequence from={fps * 8} durationInFrames={fps * 7}>
        <AgitationScene />
      </Sequence>

      {/* Solution: 15–25s */}
      <Sequence from={fps * 15} durationInFrames={fps * 10}>
        <SolutionScene />
      </Sequence>

      {/* Proof: 25–40s */}
      <Sequence from={fps * 25} durationInFrames={fps * 15}>
        <ProofScene />
      </Sequence>

      {/* CTA: 40–45s */}
      <Sequence from={fps * 40} durationInFrames={fps * 5}>
        <CTAScene />
      </Sequence>
    </>
  );
};
```

---

## 6. AbsoluteFill — base layer

```tsx
import { AbsoluteFill } from "remotion";

// ✅ Every scene wraps in AbsoluteFill
const HookScene: React.FC = () => (
  <AbsoluteFill
    style={{
      backgroundColor: "#0a0a0a",
      justifyContent: "center",
      alignItems: "center",
    }}
  >
    {/* scene content */}
  </AbsoluteFill>
);
```

---

## 7. Audio + staticFile

```tsx
import { Audio, staticFile } from 'remotion';

// ✅ Background music
<Audio
  src={staticFile('audio/background.mp3')}
  volume={0.4}
  startFrom={0}
/>

// ✅ SFX at a specific frame
<Sequence from={fps * 2}>
  <Audio src={staticFile('audio/swipe.mp3')} volume={0.8} />
</Sequence>

// ✅ Volume fade out at end
<Audio
  src={staticFile('audio/background.mp3')}
  volume={(f) =>
    interpolate(f, [durationInFrames - fps * 2, durationInFrames], [0.4, 0], {
      extrapolateLeft: 'clamp',
      extrapolateRight: 'clamp',
    })
  }
/>
```

Place all assets in `public/` — `staticFile('audio/bg.mp3')` maps to `public/audio/bg.mp3`.

---

## 8. Img + Video assets

```tsx
import { Img, staticFile } from 'remotion';

// ✅ Images
<Img src={staticFile('images/logo.png')} style={{ width: 200 }} />

// ✅ Remote image (Remotion handles CORS)
<Img src="https://example.com/image.jpg" />
```

---

## 9. OffthreadVideo

```tsx
import { OffthreadVideo, staticFile } from "remotion";

// ✅ For local video files — better than <Video> for rendering
<OffthreadVideo
  src={staticFile("videos/demo.mp4")}
  style={{ width: "100%", height: "100%", objectFit: "cover" }}
/>;
```

---

## 10. Fonts

```tsx
import { loadFont } from "@remotion/google-fonts/Inter";

const { fontFamily } = loadFont();

// Use in styles
<p style={{ fontFamily, fontSize: 64, fontWeight: 700 }}>Bold headline</p>;
```

Install first: `npm install @remotion/google-fonts`

---

## 11. Scene component template

```tsx
import {
  AbsoluteFill,
  interpolate,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

type SceneProps = {
  headline: string;
  subtext?: string;
};

export const SceneTemplate: React.FC<SceneProps> = ({ headline, subtext }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Entrance animation
  const opacity = interpolate(frame, [0, fps * 0.4], [0, 1], {
    extrapolateRight: "clamp",
  });

  const scale = spring({
    frame,
    fps,
    config: { damping: 12, stiffness: 100 },
    durationInFrames: fps * 0.6,
  });

  const translateY = interpolate(frame, [0, fps * 0.4], [30, 0], {
    extrapolateRight: "clamp",
  });

  // Exit animation (last 0.3s of scene)
  // Note: 'frame' here is local to the Sequence — frame 0 = start of this scene
  // To exit, you'd pass durationInFrames as a prop and use:
  // const exitOpacity = interpolate(frame, [durationInFrames - fps * 0.3, durationInFrames], [1, 0], { extrapolateLeft: 'clamp' })

  return (
    <AbsoluteFill
      style={{
        backgroundColor: "#0a0a0a",
        justifyContent: "center",
        alignItems: "center",
        flexDirection: "column",
        gap: 24,
        padding: 80,
      }}
    >
      <p
        style={{
          opacity,
          transform: `translateY(${translateY}px) scale(${scale})`,
          fontSize: 72,
          fontWeight: 800,
          color: "#ffffff",
          textAlign: "center",
          lineHeight: 1.1,
          margin: 0,
        }}
      >
        {headline}
      </p>

      {subtext && (
        <p
          style={{
            opacity: interpolate(frame, [fps * 0.4, fps * 0.8], [0, 1], {
              extrapolateRight: "clamp",
            }),
            fontSize: 36,
            color: "#999999",
            textAlign: "center",
            margin: 0,
          }}
        >
          {subtext}
        </p>
      )}
    </AbsoluteFill>
  );
};
```

---

## 12. Full composition template

```tsx
// src/MyVideo.tsx
import {
  AbsoluteFill,
  Audio,
  Sequence,
  staticFile,
  useVideoConfig,
} from "remotion";
import { HookScene } from "./scenes/HookScene";
import { ProblemScene } from "./scenes/ProblemScene";
import { SolutionScene } from "./scenes/SolutionScene";
import { CTAScene } from "./scenes/CTAScene";

type MyVideoProps = {
  headline: string;
};

export const MyVideo: React.FC<MyVideoProps> = ({ headline }) => {
  const { fps, durationInFrames } = useVideoConfig();

  return (
    <AbsoluteFill>
      {/* Background audio */}
      <Audio
        src={staticFile("audio/background.mp3")}
        volume={(f) =>
          interpolate(
            f,
            [durationInFrames - fps * 2, durationInFrames],
            [0.35, 0],
            { extrapolateLeft: "clamp", extrapolateRight: "clamp" },
          )
        }
      />

      {/* Scenes */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <HookScene headline={headline} />
      </Sequence>

      <Sequence from={fps * 3} durationInFrames={fps * 5}>
        <ProblemScene />
      </Sequence>

      <Sequence from={fps * 8} durationInFrames={fps * 17}>
        <SolutionScene />
      </Sequence>

      <Sequence from={fps * 25} durationInFrames={fps * 20}>
        <CTAScene />
      </Sequence>
    </AbsoluteFill>
  );
};
```

---

## 13. Common animation recipes

### Fade in

```tsx
const opacity = interpolate(frame, [0, fps * 0.5], [0, 1], {
  extrapolateRight: "clamp",
});
```

### Slide up

```tsx
const y = interpolate(frame, [0, fps * 0.4], [40, 0], {
  extrapolateRight: "clamp",
});
```

### Scale pop (spring)

```tsx
const scale = spring({ frame, fps, config: { damping: 10, stiffness: 120 } });
```

### Stagger children (delay each by index)

```tsx
const items = ["one", "two", "three"];
items.map((item, i) => {
  const delay = i * (fps * 0.15); // 0.15s stagger
  const opacity = interpolate(frame, [delay, delay + fps * 0.4], [0, 1], {
    extrapolateRight: "clamp",
    extrapolateLeft: "clamp",
  });
  return (
    <p key={item} style={{ opacity }}>
      {item}
    </p>
  );
});
```

### Counter / number animation

```tsx
const value = Math.round(
  interpolate(frame, [0, fps * 2], [0, 10000], { extrapolateRight: "clamp" }),
);
<p>{value.toLocaleString()}</p>;
```

### Pulse / heartbeat (looping)

```tsx
const pulse = Math.sin((frame / fps) * Math.PI * 2);
const scale = interpolate(pulse, [-1, 1], [0.95, 1.05]);
```

### Progress bar

```tsx
const progress = interpolate(frame, [0, fps * 3], [0, 100], {
  extrapolateRight: "clamp",
});
<div
  style={{ width: `${progress}%`, height: 8, backgroundColor: "#00ff88" }}
/>;
```

### Type-on text (character reveal)

```tsx
const charsToShow = Math.floor(
  interpolate(frame, [0, fps * 2], [0, text.length], {
    extrapolateRight: "clamp",
  }),
);
<p>{text.slice(0, charsToShow)}</p>;
```
