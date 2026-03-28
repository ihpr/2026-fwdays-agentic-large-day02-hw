---
name: memory-bank-update
description: Updates the Memory Bank with recent changes, ensuring all technical details are accurate and up to date. Use when the user asks to update the memory bank, sync documentation, or refresh project context.
---

# Skill: Memory Bank Update

## When to use

After new features, refactors, architecture changes, dependency updates, or completed milestones.
Triggers: "update memory bank", "sync docs", "refresh project docs".

## Steps

1. `git diff --stat HEAD~5` — identify recent changes
2. Read relevant files in `docs/memory/`
3. Update matching file(s) per table; input: git diff, conversation, or user description
4. Verify all claims against actual source code
5. Keep each file under 200 lines

### Change → File Mapping

| Changed area | Update these files |
|---|---|
| New feature / element type | `progress.md` + `activeContext.md` |
| Architecture change | `systemPatterns.md` + `decisionLog.md` |
| Dependency / toolchain change | `techContext.md` |
| Scope or product goal change | `projectbrief.md` + `productContext.md` |

## Excalidraw-specific examples

### `packages/excalidraw/` — core library

- **New action** (`actions/actionFoo.ts`): `systemPatterns.md` → Actions System; `decisionLog.md`.
- **New `AppState` field** (`appState.ts`): `systemPatterns.md` → State Management table (name, type, default).
- **New React context provider** in `App.render()`: `systemPatterns.md` → React Context Tree (provider + hook).
- **New canvas layer**: `systemPatterns.md` → Rendering Pipeline table.
- **Renderer change** (`renderer/staticScene.ts` / `interactiveScene.ts`): `systemPatterns.md` → Data Flow; if new implicit contract → `docs/technical/undocumented-behaviors.md`.
- **New locale key** (`packages/excalidraw/locales/`): note in `techContext.md` that Crowdin manages this — do not document individual keys.
- **Protected file changed** (`types.ts`, `actions/manager.ts`, `scene/renderer.ts`, `data/restore.ts`): `decisionLog.md` entry with rationale, risk, test coverage.

### `excalidraw-app/` — web application

- **New Firebase persistence path** (`excalidraw-app/src/data/`): `techContext.md` → Backend / Persistence.
- **Collaboration feature change** (`excalidraw-app/src/`): `productContext.md` → Real-time Collaboration; `systemPatterns.md` if new cross-cutting pattern.
- **New env variable** (`.env` / Vite config): `techContext.md` → Environment (name + purpose).

### `packages/` — shared packages

- **New package or dependency direction change**: `systemPatterns.md` → Monorepo Package Layering (preserve one-way graph: `common → math → element → excalidraw → excalidraw-app`).
- **New export from `@excalidraw/element`** (`Scene.ts` / `store.ts`): `activeContext.md` → Key Source Locations table.
- **Build script changed** (`scripts/buildPackage.js`, `vitest.config.mts`, `vite.config.ts`): `techContext.md` → Build System.

### Monorepo-wide changes

- **`yarn.lock` modified**: `techContext.md` → Dependencies (new version).
- **New Vitest / ESLint / Prettier rule**: `techContext.md` → Toolchain.
- **New pre-commit hook**: `techContext.md` → Dev Workflow.
- **New undocumented behavior** (HACK/FIXME/TODO scan): `docs/technical/undocumented-behaviors.md` + cross-reference in `systemPatterns.md` → Dangerous Patterns.

## Outputs

- List of updated Memory Bank files
- Summary of what changed and why

## Safety

- Do NOT remove manually curated content without asking
- Do NOT add speculative information — verified facts only
- Do NOT exceed 200 lines per file — summarize if needed
- Do NOT document Crowdin-managed locale files individually
- Do NOT modify protected files without a `decisionLog.md` entry
