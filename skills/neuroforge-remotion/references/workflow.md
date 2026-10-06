# NeuroForge Remotion — Tier 2 Workflow

The full protocol for a new video, a re-cut, a templated video system, or a project audit. Tier 0 and Tier 1 never touch this file.

## Contents

1. Activation sequence
2. The `neuroforge/` folder
3. `00-answers.md`
4. Domain 1 — `project/`
5. Domain 2 — `scripts/`
6. Domain 3 — `scenes/`
7. The approval gates
8. Building
9. Closing the loop — prune and handoff

---

## 1. Activation sequence

Run in order. Do not reorder, do not skip.

1. Open your reply with `Activating NeuroForge Remotion analysis...` as its own first line. It is the developer's signal the protocol engaged — exempt from the no-filler rule.
2. **Locate the project.** Look for `remotion.config.ts`, `src/Root.tsx`, or `@remotion/*` in `package.json`. None found and the task is a new video → plan a new project using `references/remotion-dev/remotion-create/REFERENCE.md`. None found and the task implies one exists → ask for the path, once.
   - **Scaffolding into a non-empty folder.** `create-video` refuses a folder that already has files — and step 3 puts `neuroforge/` and `.gitignore` there. Scaffolding happens after approval anyway, so plan it now: scaffold into a temporary subfolder and move its contents up (merging the `.gitignore`), or scaffold into a named subfolder (`video/`) if the folder holds other work. State which in the plan.
3. **Create `neuroforge/`** with `project/`, `scripts/`, `scenes/` and add `neuroforge/` to `.gitignore` (create it if missing). Mandatory, no permission needed.
4. **Inventory `neuroforge/` before reading code.** Read every file already there, check its claims against the current code, and report a status table:

   | File | Status | Note |
   |---|---|---|
   | `scripts/02-script.md` | current | approved 2026-09-12 |
   | `project/04-animation-patterns.md` | stale | Scene 3 rewritten since |
   | `scenes/03-proof.md` | superseded | by `03-v2-proof.md` |

   What was already decided is not re-derived. What was already fixed is not reported again.
5. **Scan** — structure before detail: `Root.tsx` compositions, scene folders, shared components, `public/` assets, fonts, installed `@remotion/*` versions.
6. Write `project/` → `scripts/` → present → **wait** → `scenes/` → present → **wait for "Proceed"**.
7. Track the work in the IDE's native task list (Claude Code todos, Cursor to-dos, Antigravity task panel). **Never** a `task.md`, `todo.md`, `plan.md`, or `checklist.md` inside `neuroforge/` — no native list → keep the checklist inline in your reply.

## 2. The `neuroforge/` folder

```
neuroforge/
  00-answers.md      ← settled answers (append-only log)
  project/           ← technical analysis of the codebase
  scripts/           ← one message, script, psychology
  scenes/            ← scene-by-scene direction
```

- **Analysis only.** No executable code in these files — frame ranges, copy, and direction, not TSX.
- **Supersede, never destroy.** Replace a file with a versioned name (`02-v2-script.md`) and say so. Never overwrite, never delete without approval, never archive (`old/`, `archive/`, `-deprecated`). A file is current or a proposed delete.
- **Small files, one concern each.** Never one giant analysis file.

## 3. `00-answers.md`

A settled question should cost once. Read this before asking anything.

```markdown
## Brand
- Primary #0B84F3, ink #0A0A0A, display font "Clash Display", body "Inter". (2026-10-06)

## Formats
- Master 1080×1920 @ 30fps; cut-downs 1080×1080 and 1920×1080. (2026-10-06)

## Decisions
- No voiceover — captions carry the copy. Settled. (2026-10-06)

## Preferences
- Studio always running on :3000 — never start it. (2026-10-06)
```

One dated line per entry, append-only, never pruned. Correct a stale line by replacing it, dated. Ask in chat, log the answer — never use this file as a queue of open questions.

## 4. Domain 1 — `project/`

Only what the task needs. Typical set:

| File | Holds |
|---|---|
| `01-project-overview.md` | Entry points, folder layout, compositions, Remotion version. Amend in place across tasks — never overwrite |
| `02-composition-audit.md` | Every composition: id, size, fps, duration, schema, `calculateMetadata` |
| `03-component-map.md` | Scenes and shared components, reuse opportunities, dead files |
| `04-animation-patterns.md` | How timing and easing are done today; frame-driven violations |
| `05-assets-and-brand.md` | Fonts (and how they load), audio, images, video, Lottie; the brand system |
| `06-best-practice-gaps.md` | Delta between the code and `references/remotion-dev/` — with file:line |

Flag, with evidence: non-frame-driven animation (hard stop 1), hardcoded frame numbers, missing `premountFor`, `Math.random`, fonts not awaited, `<Video>`/`<Audio>` from the wrong package for the installed version, `any`, missing schemas, assets outside `public/`.

## 5. Domain 2 — `scripts/`

Written **before** any scene work. See `references/scripting.md` for the content.

| File | Holds |
|---|---|
| `01-one-message.md` | The one message, audience, platform, length, format, the single CTA |
| `02-script.md` | Beat-by-beat script with timings in seconds |
| `03-psychology.md` | Which persuasion models are applied, where, and why |

Existing video? Analyse what the current script achieves and where it loses people before proposing changes. Never throw away what works.

## 6. Domain 3 — `scenes/`

Only after the script direction is approved. `01-scene-breakdown.md` holds the full map; add one file per scene when a scene is complex (3D, multi-layer, data-driven). Format in `references/scene-direction.md`.

## 7. The approval gates

Two gates, never more:

1. **After `project/` + `scripts/`** — "Is this the message and the script?"
2. **After `scenes/`** — "Proceed?" No code before this.

Once the developer says "Proceed", execute the approved plan end to end. Don't re-ask per scene, and don't silently expand it.

**Arriving already approved.** If the developer hands you an approved plan in the message ("scene plan is approved, proceed") and no `neuroforge/` files exist, don't re-run the analysis — build. Record the plan you were given in `scenes/01-scene-breakdown.md` and any settled facts (brand, format, BPM) in `00-answers.md` so the next session starts from it.

## 8. Building

- Open Studio (or confirm it is open) before the first scene, so the developer watches it come together.
- Build **scene by scene** in plan order. After each, check it in Studio at its first, middle, and last frame, then tick it in the task list.
- Scene components live in `src/scenes/`, shared motion primitives in `src/components/`, brand tokens in one `src/theme.ts`.
- Sound effects and music live in `public/` for anything that ships. Remote SFX URLs are fine for a draft, but say that rendering then needs network access.
- A scene that drifts from its file (new timing, new copy) → update the scene file in the same step.
- Tell the developer the composition id and Studio URL (`http://localhost:<port>/<composition-id>`) when done.

## 9. Closing the loop — prune and handoff

After the build:

1. Verify every file you wrote exists with the expected content.
2. Promote anything durable (brand, formats, conventions) into `project/01-project-overview.md` or `00-answers.md`.
3. Close with the **Motion Quality Verdict** (`references/review.md`).
4. **Propose a prune** of `neuroforge/` — a list, one reason per file. Delete only what the developer approves. `00-answers.md` is never pruned.

**Context getting heavy?** Say so and offer a handoff note written to the OS temp directory, never into the repo: the skills to load, the `neuroforge/` files to read, the scenes done and remaining, the Studio port. No secrets.
