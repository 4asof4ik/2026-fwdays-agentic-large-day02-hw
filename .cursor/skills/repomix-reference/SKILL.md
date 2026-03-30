---
name: repomix-reference-2026-fwdays-agentic-large-day02-hw
description: Reference codebase for 2026 Fwdays Agentic Large Day02 Hw. Use this skill when you need to understand the structure, implementation patterns, or code details of the 2026 Fwdays Agentic Large Day02 Hw project.
---

# Excalidraw Codebase Reference

937 files | 244648 lines | 3416938 tokens

## Overview

Use this skill when you need to:
- Understand project structure and file organization
- Find where specific functionality is implemented
- Navigate Excalidraw-specific concepts (elements, actions, rendering, collaboration)
- Trace data flow through the editor pipeline
- Read source code for any file
- Search for code patterns or keywords

## Reference Files

| File | Contents |
|------|----------|
| `references/summary.md` | **Start here** - Purpose, format explanation, and statistics |
| `references/project-structure.md` | Directory tree with line counts per file |
| `references/files.md` | All file contents (search with `## File: <path>`) |
| `references/tech-stack.md` | Languages, frameworks, and dependencies |

## Excalidraw Domain Concepts

### Element Types

All element types are defined in `packages/element/src/types.ts` as a discriminated union on the `type` field.

| Type alias | `type` field | Key extra fields |
|------------|--------------|------------------|
| `ExcalidrawGenericElement` | `"selection"` \| `"rectangle"` \| `"diamond"` \| `"ellipse"` | Base geometry only |
| `ExcalidrawTextElement` | `"text"` | `fontSize`, `fontFamily`, `text`, `containerId`, `autoResize`, `lineHeight` |
| `ExcalidrawLinearElement` | `"line"` \| `"arrow"` | `points`, `startBinding`, `endBinding` |
| `ExcalidrawLineElement` | `"line"` | Adds `polygon` |
| `ExcalidrawArrowElement` | `"arrow"` | Arrowheads, optional `elbowed` |
| `ExcalidrawElbowArrowElement` | `"arrow"` with `elbowed: true` | `fixedSegments`, `startIsSpecial`, `endIsSpecial` |
| `ExcalidrawFreeDrawElement` | `"freedraw"` | `points`, `pressures`, `simulatePressure` |
| `ExcalidrawImageElement` | `"image"` | `fileId`, `status`, `scale`, `crop` |
| `ExcalidrawFrameElement` | `"frame"` | `name` |
| `ExcalidrawMagicFrameElement` | `"magicframe"` | `name` |
| `ExcalidrawIframeElement` | `"iframe"` | `customData.generationData` |
| `ExcalidrawEmbeddableElement` | `"embeddable"` | — |

**Base fields** (`_ExcalidrawElementBase`): `x`, `y`, `width`, `height`, `angle`, stroke/fill/roughness, `version`/`versionNonce`/`updated`, `groupIds`, `frameId`, `boundElements`, `link`, `locked`.

### AppState

Defined in `packages/excalidraw/types.ts`. Key field groups:

- **Active tool**: `activeTool`, `preferredSelectionTool`, `penMode`
- **In-progress geometry**: `newElement`, `multiElement`, `selectionElement`, `resizingElement`
- **Selection**: `selectedElementIds`, `selectedGroupIds`, `editingGroupId`, `selectedLinearElement`
- **Editing**: `editingTextElement`, `editingFrame`
- **Viewport**: `scrollX`, `scrollY`, `zoom`, `width`, `height`
- **Drawing defaults**: `currentItemStrokeColor`, `currentItemFillColor`, fonts, arrows, roughness
- **Collaboration**: `collaborators`, `userToFollow`, `followedBy`
- **Bindings**: `isBindingEnabled`, `bindingPreference`, `suggestedBinding`, `bindMode`
- **Frames**: `frameToHighlight`, `frameRendering`, `editingFrame`
- **UI**: `contextMenu`, `openMenu`, `openPopup`, `openSidebar`, `openDialog`, `toast`, `theme`
- **Modes**: `gridModeEnabled`, `zenModeEnabled`, `viewModeEnabled`, `objectsSnapModeEnabled`

### Scene

The `Scene` class lives in `packages/element/src/Scene.ts` (not in `packages/excalidraw/scene/`).

- Holds `elements` (full ordered list), `elementsMap` (id → element), `nonDeletedElements`
- Fractional-index ordering via `syncInvalidIndices`
- `sceneNonce` invalidates renderer memoization on each `triggerUpdate()`
- Key methods: `getElement`, `getSelectedElements`, `mapElements`, `insertElement`, `replaceAllElements`, `mutateElement`

### Bindings

Defined in `packages/element/src/binding.ts`.

- **Arrow endpoints** → shapes: `startBinding` / `endBinding` = `FixedPointBinding | null` with `{ elementId, fixedPoint, mode }`
- **Inverse**: bindable elements hold `boundElements: { id, type: "arrow" | "text" }[]`
- **Text in containers**: `ExcalidrawTextElement.containerId` → container shape, handled in `textElement.ts`
- Key functions: `updateBoundPoint`, `updateBoundElements`, `fixBindingsAfterDeletion`, `fixDuplicatedBindingsAfterDuplication`

### Frames

Defined in `packages/element/src/frame.ts`.

- Membership via `element.frameId` pointing at a frame element (not a child list on the frame)
- Key helpers: `getFrameChildren`, `groupByFrameLikes`, `getRootElements`, `isElementIntersectingFrame`, `getElementsCompletelyInFrame`, `bindElementsToFramesAfterDuplication`

### Linear Elements (Lines & Arrows)

Defined in `packages/element/src/linearElementEditor.ts`.

- `LinearElementEditor` holds `elementId`, selection indices, drag state
- Static APIs: `handlePointerDown/Move/Up`, `createPointAt`, `addPoints`, `movePoints`, `deletePoints`, `duplicateSelectedPoints`, `addMidpoint`
- Elbow arrows: `packages/element/src/elbowArrow.ts` for orthogonal routing

### Collaboration

Lives in `excalidraw-app/collab/` (app-level, not in the library).

- **`Collab.tsx`**: session lifecycle, element sync, file sync
- **`Portal.tsx`**: Socket.IO + AES encryption for room communication
- **Sync flow**: local edits → `broadcastElements` (incremental UPDATE) → remote receives → decrypt → `reconcileElements` → `updateScene`
- **`reconcileElements`** (`packages/excalidraw/data/reconcile.ts`): merges local + remote by id, version comparison, fractional-index ordering
- **Firebase**: durable encrypted snapshots + image blob storage

## Architecture Patterns

### Action System

Actions are the **only** way to update editor state.

| Concept | File |
|---------|------|
| `Action` interface | `packages/excalidraw/actions/types.ts` |
| `ActionManager` | `packages/excalidraw/actions/manager.tsx` |
| Registration | `packages/excalidraw/actions/register.ts` |
| All actions barrel | `packages/excalidraw/actions/index.ts` |

**Pattern to add a new action:**
1. Add name to `ActionName` union in `actions/types.ts`
2. Create `action<Name>.ts(x)` with `export const myAction = register({ name, label, perform, ... })`
3. `perform(elements, appState, value, app)` returns `ActionResult` — partial `appState`, element list, files, `captureUpdate`
4. Export from `actions/index.ts` (ensure module is loaded before `App` calls `registerAll`)
5. Optionally add `keyTest`, `predicate`, `PanelComponent`, `trackEvent`

**Action files** (47 modules): `actionAddToLibrary`, `actionAlign`, `actionBoundText`, `actionCanvas`, `actionClipboard`, `actionCropEditor`, `actionDeleteSelected`, `actionDeselect`, `actionDistribute`, `actionDuplicateSelection`, `actionElementLink`, `actionElementLock`, `actionEmbeddable`, `actionExport`, `actionFinalize`, `actionFlip`, `actionFrame`, `actionGroup`, `actionHistory`, `actionLinearEditor`, `actionLink`, `actionMenu`, `actionNavigate`, `actionProperties`, `actionSelectAll`, `actionStyles`, `actionTextAutoResize`, `actionToggleArrowBinding`, `actionToggleGridMode`, `actionToggleMidpointSnapping`, `actionToggleObjectsSnapMode`, `actionToggleSearchMenu`, `actionToggleShapeSwitch`, `actionToggleStats`, `actionToggleViewMode`, `actionToggleZenMode`, `actionZindex`

### Rendering Pipeline

Two-layer Canvas 2D rendering (not React DOM for drawing):

| Layer | File | Role |
|-------|------|------|
| Static | `packages/excalidraw/renderer/staticScene.ts` | Background grid + shapes (Rough.js), frame clipping, link icons |
| Interactive | `packages/excalidraw/renderer/interactiveScene.ts` | Selection handles, cursors, scrollbars, snaps, laser pointer |
| Culling | `packages/excalidraw/scene/Renderer.ts` | `getRenderableElements` — viewport culling, memoized |
| Static canvas | `packages/excalidraw/components/canvases/StaticCanvas.tsx` | React wrapper calling `renderStaticScene` |
| Interactive canvas | `packages/excalidraw/components/canvases/InteractiveCanvas.tsx` | React wrapper calling `renderInteractiveScene` |
| Per-element draw | `packages/element/src/renderElement.ts` | Dispatch draw call per element type |

**Flow**: `App.render()` → `Renderer.getRenderableElements` (culled list) → `StaticCanvas` + `InteractiveCanvas` → `canvas.getContext("2d")`.

### Element CRUD

| Operation | Function / Pattern | File |
|-----------|--------------------|------|
| **Create** | `newElement`, `newTextElement`, `newLinearElement`, `newArrowElement`, `newImageElement`, `newFrameElement`, `newFreeDrawElement` | `packages/element/src/newElement.ts` |
| **Read** | `Scene.getElement`, `elementsMap`, `getSelectedElements` | `packages/element/src/Scene.ts` |
| **Update** | `mutateElement` (in-place + version bump), `newElementWith` (immutable copy) | `packages/element/src/mutateElement.ts` |
| **Delete** | Soft delete via `isDeleted: true` + binding/frame cleanup | `mutateElement` + `binding.ts` |

### Data Flow

```
User input → ActionManager.executeAction → Action.perform → ActionResult
→ App.syncActionResult → merges AppState + elements + files
→ Scene.replaceAllElements → triggerUpdate → sceneNonce++
→ Renderer.getRenderableElements (viewport culling)
→ StaticCanvas (renderStaticScene) + InteractiveCanvas (renderInteractiveScene)
→ canvas.getContext("2d") draw calls
→ onChange callback notifies host
```

### State Management

- **React state in `App`** (class component) for authoritative editor snapshot
- **Jotai** (`editor-jotai.ts` with `jotai-scope`) for scoped UI atoms; each instance isolated
- **App Jotai** (`excalidraw-app/app-jotai.ts`) for shell-level atoms (collab flags, theme)
- **Contexts**: `ExcalidrawAPIContext`, `UIAppStateContext`, `TunnelsContext`
- **Tunnel-rat**: named slots (MainMenu, Footer, Welcome screen) for host-injected UI

### Data Persistence

| Layer | Files | Mechanism |
|-------|-------|-----------|
| Core JSON | `packages/excalidraw/data/json.ts`, `restore.ts`, `encode.ts`, `blob.ts` | `.excalidraw` file format, PNG/SVG embed |
| Local storage | `excalidraw-app/data/LocalData.ts`, `localStorage.ts` | localStorage + IndexedDB (`idb-keyval`) |
| Cloud (collab) | `excalidraw-app/data/firebase.ts` | Firestore docs + Storage for images |
| File sync | `excalidraw-app/data/FileManager.ts` | Image upload/download orchestration |

## Key File Map

### Where to find things

| Looking for... | Go to... |
|----------------|----------|
| Element type definitions | `packages/element/src/types.ts` |
| How elements are created | `packages/element/src/newElement.ts` |
| How elements are mutated | `packages/element/src/mutateElement.ts` |
| Arrow ↔ shape bindings | `packages/element/src/binding.ts` |
| Text inside containers | `packages/element/src/textElement.ts` |
| Frame grouping logic | `packages/element/src/frame.ts` |
| Linear/arrow point editing | `packages/element/src/linearElementEditor.ts` |
| Elbow arrow routing | `packages/element/src/elbowArrow.ts` |
| Element hit testing | `packages/element/src/collision.ts` |
| Element bounds/geometry | `packages/element/src/bounds.ts` |
| Type guards (`isArrowElement`, etc.) | `packages/element/src/typeChecks.ts` |
| Z-index ordering | `packages/element/src/zindex.ts` |
| Per-element canvas draw | `packages/element/src/renderElement.ts` |
| Scene element storage | `packages/element/src/Scene.ts` |
| Action definitions | `packages/excalidraw/actions/action*.ts(x)` |
| Action types & ActionResult | `packages/excalidraw/actions/types.ts` |
| ActionManager | `packages/excalidraw/actions/manager.tsx` |
| Main App component | `packages/excalidraw/components/App.tsx` (12,818 lines) |
| UI chrome layer | `packages/excalidraw/components/LayerUI.tsx` |
| Static rendering | `packages/excalidraw/renderer/staticScene.ts` |
| Interactive rendering | `packages/excalidraw/renderer/interactiveScene.ts` |
| Viewport culling | `packages/excalidraw/scene/Renderer.ts` |
| AppState type | `packages/excalidraw/types.ts` |
| Restore/deserialize | `packages/excalidraw/data/restore.ts` |
| Reconcile remote data | `packages/excalidraw/data/reconcile.ts` |
| Collaboration | `excalidraw-app/collab/Collab.tsx`, `Portal.tsx` |
| Firebase persistence | `excalidraw-app/data/firebase.ts` |
| Local persistence | `excalidraw-app/data/LocalData.ts` |
| 2D geometry/math | `packages/math/src/` |
| Shared constants/keys | `packages/common/src/constants.ts`, `keys.ts` |
| Event bus | `packages/common/src/emitter.ts`, `appEventBus.ts` |

### Largest files (complexity hotspots)

| File | Lines |
|------|-------|
| `packages/excalidraw/components/App.tsx` | 12,818 |
| `packages/element/src/binding.ts` | 2,940 |
| `packages/element/src/linearElementEditor.ts` | 2,507 |
| `packages/element/src/elbowArrow.ts` | 2,309 |
| `packages/excalidraw/renderer/interactiveScene.ts` | 2,090 |
| `packages/excalidraw/tests/history.test.tsx` | 5,307 |

## How to Use Reference Files

### 1. Find file locations

Check `project-structure.md` for the directory tree:

```text
packages/
  excalidraw/
    components/App.tsx (12818 lines)
    actions/actionAlign.tsx (220 lines)
  element/
    src/types.ts (500 lines)
```

### 2. Read file contents

Grep in `files.md` for the file path:

```text
## File: packages/element/src/types.ts
```

### 3. Search for Excalidraw-specific code

Grep in `files.md` for domain keywords:

```text
ExcalidrawLinearElement
mutateElement
renderStaticScene
reconcileElements
FixedPointBinding
```

## Common Use Cases

**Understand an element type:**
1. Find type definition in `packages/element/src/types.ts`
2. Check creation in `packages/element/src/newElement.ts`
3. Check rendering in `packages/element/src/renderElement.ts`
4. Check hit-testing in `packages/element/src/collision.ts`

**Understand an action:**
1. Find action file in `packages/excalidraw/actions/action<Name>.ts(x)`
2. Read `perform` function for state logic
3. Check `keyTest` for keyboard shortcut
4. Check `PanelComponent` for UI binding

**Understand rendering:**
1. Start at `packages/excalidraw/renderer/staticScene.ts` (shapes) or `interactiveScene.ts` (UI)
2. Check `packages/element/src/renderElement.ts` for per-element draw
3. Check `packages/excalidraw/scene/Renderer.ts` for culling logic

**Understand binding behavior (arrows ↔ shapes):**
1. Read `packages/element/src/binding.ts` — core bind/unbind logic
2. Check `packages/element/src/types.ts` for `FixedPointBinding`, `boundElements`
3. Check `packages/element/src/linearElementEditor.ts` for drag interaction

**Debug collaboration sync:**
1. Check `excalidraw-app/collab/Collab.tsx` for session lifecycle
2. Check `excalidraw-app/collab/Portal.tsx` for WebSocket transport
3. Check `packages/excalidraw/data/reconcile.ts` for merge logic

**Find all usages:**
1. Grep function or type name in `files.md`

## Tips

- Use line counts in `project-structure.md` to estimate file complexity
- Search `## File:` pattern to jump between files in `files.md`
- Use type guards from `packages/element/src/typeChecks.ts` (`isArrowElement`, `isTextElement`, `isFrameLikeElement`, etc.) when filtering elements
- `App.tsx` is 12K+ lines — use semantic search or grep for specific methods rather than reading the whole file
- Actions are the canonical way to mutate state — never bypass `ActionManager`
- Element mutations go through `mutateElement` (in-place) or `newElementWith` (immutable copy) — never assign fields directly

---

This skill was generated by [Repomix](https://github.com/yamadashy/repomix) and enhanced with Excalidraw architecture documentation.
