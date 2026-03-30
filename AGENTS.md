# AGENTS.md

## Project

Excalidraw — open-source collaborative whiteboard
TypeScript monorepo (Yarn workspaces)

## Tech Stack

- React 19, TypeScript 5.9 (strict mode)
- Vite 5 (dev server + bundler), Vitest 3 (test runner)
- Yarn 1.22 workspaces (monorepo package management)
- Node >= 18
- Browser targets: modern evergreen (no IE 11, no Safari < 12, no Chrome < 70, no Edge < 79); see `excalidraw-app/package.json` browserslist

## Project Structure

```
excalidraw-app/          — host application (entry: index.tsx)
packages/
  excalidraw/            — core library (entry: index.tsx)
  math/                  — geometry & linear-algebra helpers
  utils/                 — general-purpose utilities
  common/                — shared code across packages
  element/               — element-specific logic
examples/                — integration examples (with-nextjs, with-script-in-browser)
scripts/                 — build & release scripts
public/                  — static assets
docs/                    — documentation & memory bank
```

Tests live inside each package (`packages/*/tests/`) and in `excalidraw-app/tests/`.

Protected files (full paths — see also "Protected files" section):
- `packages/excalidraw/scene/Renderer.ts` — render pipeline
- `packages/excalidraw/data/restore.ts` — file format compatibility
- `packages/excalidraw/actions/manager.tsx` — action system
- `packages/excalidraw/types.ts` — core type definitions

## Setup

- `yarn` — install dependencies
- `yarn start` — dev server (port 3000)
- `yarn test` — run Vitest
- `yarn build` — production build

## Architecture

- State: custom actionManager (NOT Redux/Zustand)
- Rendering: canvas 2D (NOT React DOM for drawing)
- Packages: excalidraw (main), math, utils

## Constraints

- No new npm deps without approval
- No direct state mutation — use actionManager
- No class components — functional + hooks only
- TypeScript strict — no `any` or `@ts-ignore`

## Protected files

- renderer.ts, restore.ts, manager.ts, types.ts

## Memory Bank

- **Location:** `docs/memory/` — curated Markdown for project context. Overview and file roles: [docs/README.md](docs/README.md).
- **After each substantive change** (feature, fix, refactor, config, or doc that affects how the repo works): update the memory bank so the next session stays accurate.
  - **Always touch:** [docs/memory/activeContext.md](docs/memory/activeContext.md) — current focus, what changed, next steps, open questions.
  - **Also update when relevant:** [docs/memory/progress.md](docs/memory/progress.md) (status/WIP); [docs/memory/systemPatterns.md](docs/memory/systemPatterns.md) (new or changed code patterns); [docs/memory/techContext.md](docs/memory/techContext.md) (tooling, versions, scripts, env); [docs/memory/decisionLog.md](docs/memory/decisionLog.md) (meaningful architectural or process decisions); [docs/memory/projectbrief.md](docs/memory/projectbrief.md) / [docs/memory/productContext.md](docs/memory/productContext.md) only if charter or product-facing behavior shifted.
  - **Deep docs:** If the change belongs in long-form spec or architecture prose, update [docs/technical/](docs/technical/) or [docs/product/](docs/product/) and link from memory files instead of duplicating.
