# Summary

This file is a merged representation of the entire codebase, combined into a single document by Repomix.

## Purpose

This is a reference codebase organized into multiple files for AI consumption.
It is designed to be easily searchable using grep and other text-based tools.

## Project: Excalidraw

Open-source collaborative whiteboard — a virtual canvas for sketching hand-drawn-like diagrams. TypeScript monorepo (Yarn workspaces) with a publishable editor library and a first-party host app.

### Key Concepts

- **Elements**: drawable objects on the canvas (rectangles, ellipses, arrows, text, images, frames, etc.) — defined in `packages/element/src/types.ts`
- **AppState**: editor state (active tool, selection, viewport, drawing defaults, UI flags) — defined in `packages/excalidraw/types.ts`
- **Scene**: authoritative element storage with maps, ordering, and change tracking — `packages/element/src/Scene.ts`
- **Actions**: command objects executed via `ActionManager` to update state — `packages/excalidraw/actions/`
- **Rendering**: two-layer Canvas 2D (static scene + interactive overlay) — `packages/excalidraw/renderer/`
- **Bindings**: arrows snap to shapes via `FixedPointBinding`; text lives inside containers — `packages/element/src/binding.ts`
- **Frames**: grouping containers where membership is via `element.frameId` — `packages/element/src/frame.ts`
- **Collaboration**: Socket.IO + AES encryption + Firebase storage — `excalidraw-app/collab/`

## File Structure

This skill contains the following reference files:

| File | Contents |
|------|----------|
| `project-structure.md` | Directory tree with line counts per file |
| `files.md` | All file contents (search with `## File: <path>`) |
| `tech-stack.md` | Languages, frameworks, dependencies, and Excalidraw-specific tools |
| `summary.md` | This file - purpose and format explanation |

## Usage Guidelines

- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.

## Notes

- Some files may have been excluded based on .gitignore rules and Repomix's configuration
- Binary files are not included in this packed representation. Please refer to the Repository Structure section for a complete list of file paths, including binary files
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)

## Statistics

937 files | 244,648 lines

| Language | Files | Lines |
|----------|------:|------:|
| TypeScript | 312 | 82,356 |
| TypeScript (TSX) | 289 | 84,621 |
| JSON | 91 | 41,932 |
| SCSS | 82 | 9,549 |
| Markdown | 36 | 5,318 |
| MDX | 33 | 3,628 |
| JavaScript | 25 | 3,115 |
| No Extension | 21 | 266 |
| YAML | 14 | 309 |
| SVG | 13 | 85 |
| Other | 21 | 13,469 |

**Largest files (complexity hotspots):**
- `packages/excalidraw/components/App.tsx` (12,818 lines) — core editor lifecycle, input, state
- `packages/excalidraw/fonts/ComicShanns/ComicShanns-Regular.sfd` (12,221 lines) — font data
- `packages/excalidraw/tests/history.test.tsx` (5,307 lines) — history tests
- `packages/element/src/binding.ts` (2,940 lines) — arrow/shape binding logic
- `packages/element/src/linearElementEditor.ts` (2,507 lines) — polyline/arrow editing
- `packages/element/src/elbowArrow.ts` (2,309 lines) — elbow arrow routing
- `packages/excalidraw/renderer/interactiveScene.ts` (2,090 lines) — interactive overlay rendering
