# AGENTS.md

## Project
Excalidraw — open-source collaborative whiteboard
TypeScript monorepo (Yarn workspaces)

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
