# AGENTS.md

> Read `docs/memory/` before starting any task.

---

## Project Overview

**Excalidraw** — open-source browser-based whiteboard for hand-drawn diagrams. This repo is the official monorepo, used as a 2026 fwdays agentic workshop homework workspace.

Two products from one codebase:

- **`@excalidraw/excalidraw`** (npm library, v0.18.0) — embeddable React component with public API (`<Excalidraw />` + `ExcalidrawImperativeAPI`). Client-side only; no SSR; requires non-zero height container.
- **`excalidraw-app`** (excalidraw.com) — adds Socket.io collaboration, Firebase persistence, Sentry, PWA on top of the library.

Core: rectangle/ellipse/arrow/freehand/text/image/frame/embed tools, roughjs hand-drawn style, real-time multi-user collab, IndexedDB autosave, Firebase sharing, PNG/SVG/JSON export, element library, i18n, keyboard shortcuts.

GitHub: <https://github.com/excalidraw/excalidraw> — MIT

---

## Project Structure

```text
excalidraw-app/       # Full web app (private)
packages/
  excalidraw/         # @excalidraw/excalidraw — main React library
  element/            # @excalidraw/element — element model, Scene, Store
  math/               # @excalidraw/math — geometry primitives
  common/             # @excalidraw/common — constants, utilities, palette
  utils/              # @excalidraw/utils — export helpers, bbox (leaf, no upward deps)
examples/             # NextJS + browser script integration examples
docs/memory/          # Memory bank — read before any task
scripts/              # buildPackage.js (esbuild library builds)
firebase-project/     # Firestore + Storage rules
```

One-way dependency graph (no circular imports):

```text
@excalidraw/common → @excalidraw/math → @excalidraw/element → @excalidraw/excalidraw → excalidraw-app
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Language | TypeScript (strict) | 5.9.3 |
| UI | React + React DOM | 19.0.0 |
| App bundler | Vite | 5.0.12 |
| Library bundler | esbuild (`scripts/buildPackage.js`) | 0.19.10 |
| UI state (cross-cutting) | Jotai + jotai-scope | 2.11.0 / 0.7.2 |
| Drawing | roughjs, perfect-freehand | 4.6.4 / 1.2.0 |
| Collaboration | socket.io-client | 4.7.2 |
| Cloud persistence | Firebase | 11.3.1 |
| Local persistence | idb-keyval (IndexedDB) | 6.0.3 |
| Styling | Sass (.scss) + clsx | 1.51.0 / 1.1.1 |
| UI primitives | radix-ui | 1.4.3 |
| Testing | Vitest + @testing-library/react + jsdom | 3.0.6 / 16.2.0 / 22.1.0 |
| Linting/formatting | ESLint + Prettier + lint-staged + husky | — |
| Node.js | ≥ 18.0.0 | — |
| Package manager | Yarn 1 workspaces (1.22.22) | — |

> Core canvas/element state is **plain React class component state** (`AppState`, 80+ fields) — not Jotai. Jotai covers sidebar, library panel, search, i18n only.

---

## Conventions

- TypeScript strict; no `any` without justification.
- No new npm packages without approval; check `packages/utils/` first.
- No `redux`, `zustand`, `mobx`, `react-konva`, `fabric.js`, `pixi.js`.
- All element mutations via `mutateElement()` (`packages/element/src/mutateElement.ts`) — never direct assignment. Bumps `version` + `versionNonce` for collab sync.
- `AppState` updates via `actionManager.dispatch()` only.
- No `innerHTML` / unsanitized `dangerouslySetInnerHTML` for scene, library, or collaborator input.
- Editor features → `packages/*`; app-specific (collab, Firebase, Sentry) → `excalidraw-app/`.
- Firebase and Socket.io must stay in `excalidraw-app` — never in library packages.
- Imports must respect the one-way dependency graph; use aliases from `vitest.config.mts`.
- All uploaded/fetched JSON must pass `isValidExcalidrawData` / `isValidLibrary` before use.
- Remote library URLs: `importLibraryFromURL` → `toValidURL` → `validateLibraryUrl` — no blind `fetch(userString)`.
- Room/share keys stay in URL **hash** only — never in query params or logs.
- Full security rules: `.cursor/rules/security.mdc`.

---

## Development Workflow

1. Package features → `packages/*`; app features → `excalidraw-app/`.
2. Before committing: `yarn test:update`.
3. Type check: `yarn test:typecheck`.
4. Auto-fix: `yarn fix`.

```bash
yarn start                # dev server (Vite)
yarn build                # build full app
yarn build:packages       # build all library packages
yarn build:app:docker     # docker build (Sentry disabled)
yarn test                 # vitest watch
yarn test:all             # typecheck + lint + prettier + vitest
yarn test:typecheck       # tsc only
yarn test:code            # eslint only
yarn test:coverage        # vitest with coverage
yarn fix                  # prettier + eslint --fix
yarn release:latest       # publish @latest to npm
yarn release:next         # publish @next tag
```

---

## Architecture Notes

### Three-Canvas Rendering

| Canvas | Component | Responsibility |
|---|---|---|
| Static | `StaticCanvas` | Background, grid, committed elements |
| New element | `NewElementCanvas` | In-progress element while drawing |
| Interactive | `InteractiveCanvas` | Selection, handles, snap lines, cursors |

`Renderer.getRenderableElements()` memoized on `sceneNonce` + viewport params. Redraws via `scene.triggerUpdate()` (nonce bump) or `App.triggerRender(force)`.

### State — Three Layers

| Layer | Owner | Purpose |
|---|---|---|
| Document Model | `Scene` + `Store` + `History` | Element graph, undo/redo deltas |
| Editor UI | `App` class (`AppState`) | Tool, scroll, zoom, selections, dialogs |
| Cross-cutting UI | Jotai (per-instance isolated) | Library panel, sidebar, search, i18n |

### Actions System

`actions/action*.ts` → `register(action)` → module-level array. Each action: `perform()`, optional `keyTest` + `PanelComponent`, `captureUpdate` (undo policy). `ActionManager` wires to keyboard + toolbar.

### Data Flow

```text
Event → ActionManager.handleKeyDown → action.perform() → ActionResult
      → App.syncActionResult() [withBatchedUpdates]
          ├─ store.scheduleAction()     → undo delta
          ├─ scene.replaceAllElements() → document model
          └─ this.setState()            → React re-render
      → componentDidUpdate()
          ├─ store.commit()             → finalize undo
          └─ props.onChange()           → host callback
      → StaticCanvas / InteractiveCanvas → pixels
```

---

## Do-Not-Touch / Constraints

Protected files — no changes without explicit approval + full test suite + manual QA:

| File | Reason |
|---|---|
| `packages/excalidraw/scene/renderer.ts` | Render pipeline |
| `packages/excalidraw/data/restore.ts` | File format compatibility |
| `packages/excalidraw/actions/manager.ts` | Action system wiring |
| `packages/excalidraw/types.ts` | Core types |

**Verification for protected file changes:**

1. `git diff --name-only` — confirm no protected file in diff.
2. `yarn test:update` — all tests pass, no regressions.
3. `yarn test:typecheck` — no new TypeScript errors.
4. Manual QA: exercise the affected feature in the app.

**Hard constraints:**

- `Scene.getScene()` is tech debt — no new call-sites.
- `isElementInFrame()` is a perf bottleneck — no new call-sites.
- `flushSync` calls in `App.tsx` are load-bearing — do not remove.
- Do not regress: curved element bounding boxes, groupId ordering after undo/redo, invisible elements in undo history.
