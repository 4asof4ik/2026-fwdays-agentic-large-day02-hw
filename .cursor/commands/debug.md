# debug

Debug a component, feature, or runtime issue in the Excalidraw project. Accepts a problem description, component name, error message, or unexpected behavior as input.

## Instructions

You are a senior TypeScript/React engineer debugging the Excalidraw codebase (monorepo, Yarn workspaces, Canvas 2D rendering, custom ActionManager state). Perform a systematic investigation.

### Step 1 — Classify the problem

Determine which category the issue falls into:

| Category | Typical symptoms | Start looking at |
|---|---|---|
| **Component / UI** | Wrong render, missing element, stale UI, style glitch | `packages/excalidraw/components/`, SCSS in `css/`, tunnels, context providers |
| **State / Actions** | State not updating, wrong value after action, undo/redo bug | `packages/excalidraw/actions/`, `ActionManager`, `App` class state, Jotai atoms (`editor-jotai.ts`, `app-jotai.ts`) |
| **Rendering / Canvas** | Drawing glitch, wrong position, z-order, selection highlight | `packages/excalidraw/renderer/`, `scene/`, `roughjs` usage |
| **Element model** | Element creation/mutation, bindings, grouping, frames | `packages/element/src/`, mutation helpers, `newElement*` factories |
| **Data / Persistence** | Save/load, export, import, clipboard, library | `packages/excalidraw/data/`, `restore.ts`, `encode/decode`, blob helpers |
| **Collaboration** | Sync issues, remote cursors, conflict resolution | `excalidraw-app/collab/`, `reconcileElements`, socket/Firebase |
| **Performance** | Slow interaction, excessive re-renders, memory leak | React profiler, canvas render loop, memoization, Jotai subscriptions |
| **Build / Tooling** | Vite errors, TypeScript, ESLint, test failures | `vite.config.*`, `tsconfig.*`, `vitest.config.*`, `package.json` scripts |

### Step 2 — Gather context

1. **Locate the relevant code.** Use search/grep to find the component, action, or module mentioned in the input. Trace its imports, exports, and dependents.
2. **Read the source.** Read the file(s) thoroughly — pay attention to:
   - Props and state shape (AppState fields, Jotai atoms, context values)
   - Action definitions (`Action` objects, `ActionResult` return values, `CaptureUpdateActionType`)
   - Event handlers and their binding (pointer, keyboard, clipboard)
   - Conditional rendering logic and early returns
   - Side effects in hooks (`useEffect`, `useLayoutEffect`, cleanup)
3. **Check related files.** Look at:
   - Tests (colocated `*.test.ts(x)` or `tests/` directory) for expected behavior
   - Localization keys in `locales/en.json` if UI text is involved
   - CSS/SCSS files for styling issues
   - Type definitions in nearby `types.ts` or `@excalidraw/common`

### Step 3 — Trace the data flow

Follow the Excalidraw unidirectional loop relevant to the bug:

```
User input (pointer/keyboard/UI)
  → ActionManager.executeAction(action)
    → Action.perform() returns ActionResult
      → App merges { appState, elements, files }
        → Canvas re-render + onChange callback
```

For each step in the chain:
- **Input**: What triggers the action? Is the trigger correct?
- **Action**: Does `perform()` return the right partial state?
- **Merge**: Does App correctly merge the result? Any overwrite issues?
- **Output**: Does the renderer/component read the updated state?

### Step 4 — Identify root cause candidates

For each candidate, note:
- **File and line range** where the issue likely originates
- **What's wrong** — wrong condition, missing update, stale closure, incorrect type, race condition, etc.
- **Evidence** — code snippet, logic trace, or test output that supports the hypothesis

### Step 5 — Suggest fixes

For each root cause candidate, propose a concrete fix:
- Show the code change (before → after)
- Explain why it fixes the issue
- Note any risks or side effects
- List files that should be tested after the fix

### Step 6 — Output the diagnosis

Structure your response as:

```
## Debug Report: <short problem summary>

**Category:** <from Step 1>
**Affected files:** <list of key files>

---

### Problem Description

<restate the problem clearly with observed vs. expected behavior>

### Investigation Trace

<step-by-step walkthrough of the data flow / render path showing where it breaks>

### Root Cause

<clear explanation of what's going wrong and why>

### Proposed Fix

<code changes with before/after, file paths, and line references>

### Verification

- [ ] Unit test: <what to test>
- [ ] Manual QA: <steps to reproduce and verify the fix>
- [ ] Related areas to regression-test: <list>

### Notes

<any caveats, alternative approaches, or follow-up items>
```

## Rules

- Be specific — reference exact file paths, line numbers, and variable names.
- Always trace the full data flow before concluding. Don't guess from symptoms alone.
- Respect protected files (`renderer.ts`, `restore.ts`, `manager.ts`, `types.ts`) — flag if the fix requires modifying them.
- Check Jotai scope isolation (`editor-jotai` vs `app-jotai`) when debugging state issues.
- Verify that `ActionResult` fields match what `App` expects — partial `appState`, element arrays, `captureUpdate` type.
- For performance issues, check memoization (`React.memo`, `useMemo`, `useCallback`) and Jotai subscription granularity.
- Do NOT apply fixes automatically — only propose them. The user decides when to apply.
- If the issue is ambiguous, list multiple hypotheses ranked by likelihood.
