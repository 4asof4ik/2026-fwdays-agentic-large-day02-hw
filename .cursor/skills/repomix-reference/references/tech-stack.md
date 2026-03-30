# Tech Stack

## Runtime & Languages

- Node.js >= 18
- TypeScript 5.9 (strict mode)
- Browser targets: modern evergreen (no IE 11, no Safari < 12)

## Core Frameworks

- React 19 (class-based `App` root + functional components/hooks elsewhere)
- Vite 5 (dev server + bundler)
- Vitest 3 (test runner)
- Canvas 2D API (rendering — not React DOM for drawing)
- Rough.js (hand-drawn appearance for shapes)

## State Management

- Custom `ActionManager` (command/action pattern — NOT Redux/Zustand)
- Jotai + jotai-scope (scoped UI atoms per editor instance)
- React Context (API, tunnels, UI state slices)
- tunnel-rat (named slots for host-injected UI)

## Monorepo

- Yarn 1.22 workspaces
- Packages: `excalidraw` (editor), `element` (domain), `math` (geometry), `common` (shared), `utils` (general)

## Collaboration & Persistence

- Socket.IO (real-time collab via WebSocket)
- Firebase / Firestore (durable room storage + image blobs)
- AES encryption (end-to-end for collab rooms)
- IndexedDB via `idb-keyval` (local persistence)
- pako (deflate/inflate for binary data)

## Package Manager

- yarn

## Dev Dependencies

- @babel/preset-env (7.26.9)
- @excalidraw/eslint-config (1.0.3)
- @excalidraw/prettier-config (1.0.2)
- @types/chai (4.3.0)
- @types/jest (27.4.0)
- @types/lodash.throttle (4.1.7)
- @types/react (19.0.10)
- @types/react-dom (19.0.4)
- @types/socket.io-client (3.0.0)
- @vitejs/plugin-react (3.1.0)
- @vitest/coverage-v8 (3.0.7)
- @vitest/ui (2.0.5)
- chai (4.3.6)
- dotenv (16.0.1)
- eslint-config-prettier (8.5.0)
- eslint-config-react-app (7.0.1)
- eslint-plugin-import (2.31.0)
- eslint-plugin-prettier (3.3.1)
- http-server (14.1.1)
- husky (7.0.4)
- jsdom (22.1.0)
- lint-staged (12.3.7)
- pepjs (0.5.3)
- prettier (2.6.2)
- rewire (6.0.0)
- rimraf (^5.0.0)
- typescript (5.9.3)
- vite (5.0.12)
- vite-plugin-checker (0.7.2)
- vite-plugin-ejs (1.7.0)
- vite-plugin-pwa (0.21.1)
- vite-plugin-svgr (4.2.0)
- vitest (3.0.6)
- vitest-canvas-mock (0.3.3)

## Configuration Files

- .editorconfig
- .eslintrc.json
- .gitattributes
- .gitignore
- docker-compose.yml
- Dockerfile
- package.json
- tsconfig.json
