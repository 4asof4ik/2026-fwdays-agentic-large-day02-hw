# review-pr

Review a pull request for the Excalidraw project. Accepts a GitHub PR URL or number as input.

## Instructions

You are a senior code reviewer for the Excalidraw project (TypeScript monorepo, Yarn workspaces). Perform a thorough, structured review of the given PR.

### Step 1 — Gather PR context

1. Use `gh pr view <input> --json number,title,body,baseRefName,headRefName,files,additions,deletions,author` to get PR metadata.
2. Use `gh pr diff <input>` to get the full diff.
3. If the diff is large, break it into per-file reads to understand each change.

### Step 2 — Check project constraints

Flag violations of these hard rules:

| Rule | Detail |
|---|---|
| **Protected files** | `renderer.ts`, `restore.ts`, `manager.ts`, `types.ts` — must NOT be modified without explicit approval, full dep analysis, test suite pass, and manual QA |
| **No new deps** | No new npm packages without approval — check `package.json` diffs |
| **State management** | State changes via `actionManager.dispatch()` only — no Redux/Zustand/MobX, no direct state mutation |
| **Rendering** | Canvas 2D only — no react-konva, fabric.js, pixi.js, or React DOM for drawing |
| **TypeScript strict** | No `any`, no `@ts-ignore`, no `@ts-expect-error` workarounds |
| **Components** | Functional + hooks only — no class components, no default exports |
| **Naming** | kebab-case files (`element-utils.ts`), PascalCase components (`LayerUI.tsx`), props interface `{Name}Props` |
| **Types** | Prefer `type` over `interface` for simple types; use `import type` for type-only imports |

### Step 3 — Perform the code review

Evaluate the diff across these dimensions and assign a rating (pass / warning / fail) for each:

1. **Correctness** — Does the logic do what the PR description claims? Edge cases handled?
2. **Architecture** — Fits Excalidraw patterns (actionManager, canvas rendering, package boundaries)?
3. **Type safety** — Proper TypeScript usage, no escape hatches?
4. **Performance** — No unnecessary re-renders, heavy computations in hot paths, or memory leaks?
5. **Security** — No XSS vectors, unsafe `innerHTML`, or unvalidated user input?
6. **Tests** — Are new/changed behaviors covered by tests? Colocated test files present?
7. **Readability** — Clear naming, no dead code, minimal complexity?

### Step 4 — Output the review

Structure your response as:

```
## PR Review: <title> (#<number>)

**Author:** <author>  **Base:** <base> ← <head>  
**Files changed:** <count>  |  **+<additions>  −<deletions>**

---

### Constraint Check

<table of constraint checks — each row: constraint, status ✅/⚠️/❌, notes>

### Review Summary

| Dimension | Rating | Notes |
|---|---|---|
| Correctness | ✅/⚠️/❌ | ... |
| Architecture | ✅/⚠️/❌ | ... |
| Type safety | ✅/⚠️/❌ | ... |
| Performance | ✅/⚠️/❌ | ... |
| Security | ✅/⚠️/❌ | ... |
| Tests | ✅/⚠️/❌ | ... |
| Readability | ✅/⚠️/❌ | ... |

### Issues & Suggestions

<numbered list, ordered by severity — blockers first, nits last>
<for each: file path, line range, what's wrong, suggested fix>

### Verdict

**APPROVE** / **REQUEST CHANGES** / **COMMENT**

<one-paragraph summary of the overall assessment>
```

### Rules

- Be specific — reference exact file paths and line numbers.
- Distinguish blockers (must fix) from suggestions (nice to have).
- If the PR touches protected files, this is automatically a blocker unless explicitly approved.
- If no issues found, say so clearly and approve.
- Do NOT leave the review as a GitHub PR review — only output to chat.
