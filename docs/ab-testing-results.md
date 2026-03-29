# A/B Testing Results — Cursor Rules

This document captures A/B test experiments that measure how effectively Cursor rules guide AI-generated code toward project conventions.

---

## Test 1: Testing Rule (`testing.mdc`)

**Date:** 2026-03-30
**Rule under test:** `.cursor/rules/testing.mdc`
**Target component:** `Ellipsify` — 18-line presentational `<span>` wrapper with ellipsis styles (`packages/excalidraw/components/Ellipsify.tsx`)

### Method

Two AI agents were given the identical task: *"Write 4–6 Vitest tests for the Ellipsify component."*

| | Variant A | Variant B |
|---|---|---|
| **Receives testing rule** | Yes — full `.cursor/rules/testing.mdc` content, plus an example test from the project | No — only the component source and a generic "write tests" instruction |
| **Receives component source** | Yes | Yes |
| **Model** | Same (fast) | Same (fast) |

Both agents were read-only (no disk writes) so only the generated code was compared.

### Variant A — WITH testing rule

```typescript
import React from "react";
import { reseed } from "@excalidraw/common";
import { Excalidraw } from "../index";
import { render, screen, unmountComponent } from "../tests/test-utils";
import { Ellipsify } from "./Ellipsify";

describe("Ellipsify", () => {
  beforeEach(async () => {
    unmountComponent();
    localStorage.clear();
    reseed(7);
  });

  it("renders children text correctly", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify>Truncated label text</Ellipsify>
      </>,
    );
    expect(screen.getByText("Truncated label text")).toBeInTheDocument();
  });

  it("applies default ellipsis styles", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify data-testid="ellipsify">x</Ellipsify>
      </>,
    );
    const el = screen.getByTestId("ellipsify");
    expect(el.style.textOverflow).toBe("ellipsis");
    expect(el.style.overflow).toBe("hidden");
    expect(el.style.whiteSpace).toBe("nowrap");
  });

  it("passes through additional HTML attributes", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify className="my-ellipsis" data-testid="ellipsify-attrs" title="Full text">
          Short
        </Ellipsify>
      </>,
    );
    const el = screen.getByTestId("ellipsify-attrs");
    expect(el).toHaveClass("my-ellipsis");
    expect(el).toHaveAttribute("title", "Full text");
  });

  it("merges style overrides with defaults (later props win)", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify data-testid="ellipsify-style" style={{ color: "rgb(255, 0, 0)", textOverflow: "clip" }}>
          styled
        </Ellipsify>
      </>,
    );
    const el = screen.getByTestId("ellipsify-style");
    expect(el.style.textOverflow).toBe("clip");
    expect(el.style.overflow).toBe("hidden");
    expect(el.style.whiteSpace).toBe("nowrap");
    expect(el.style.color).toBe("rgb(255, 0, 0)");
  });

  it("renders with string children", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify>plain string</Ellipsify>
      </>,
    );
    expect(screen.getByText("plain string")).toBeInTheDocument();
  });

  it("renders with JSX element children", async () => {
    await render(
      <>
        <Excalidraw />
        <Ellipsify>
          <strong>bold</strong>
          <span> and more</span>
        </Ellipsify>
      </>,
    );
    expect(screen.getByText("bold")).toBeInTheDocument();
    expect(screen.getByText(/and more/)).toBeInTheDocument();
  });
});
```

### Variant B — WITHOUT testing rule

```typescript
import { render, screen } from "@testing-library/react";
import { Ellipsify } from "./Ellipsify";

describe("Ellipsify", () => {
  it("renders children text correctly", () => {
    render(<Ellipsify>Long label text</Ellipsify>);
    expect(screen.getByText("Long label text")).toBeInTheDocument();
  });

  it("applies default ellipsis styles", () => {
    render(<Ellipsify data-testid="ellipsify">x</Ellipsify>);
    const span = screen.getByTestId("ellipsify");
    expect(span).toHaveStyle({
      textOverflow: "ellipsis",
      overflow: "hidden",
      whiteSpace: "nowrap",
    });
  });

  it("passes through additional HTML attributes", () => {
    render(
      <Ellipsify className="my-class" data-testid="ellipsify" title="Tooltip" id="span-id" aria-label="Label">
        content
      </Ellipsify>,
    );
    const el = screen.getByTestId("ellipsify");
    expect(el).toHaveClass("my-class");
    expect(el).toHaveAttribute("title", "Tooltip");
    expect(el).toHaveAttribute("id", "span-id");
    expect(el).toHaveAttribute("aria-label", "Label");
  });

  it("merges custom styles with defaults and allows overrides from rest.style", () => {
    render(
      <Ellipsify data-testid="ellipsify" style={{ opacity: 0.5, whiteSpace: "normal" }}>
        x
      </Ellipsify>,
    );
    const el = screen.getByTestId("ellipsify");
    expect(el).toHaveStyle({
      textOverflow: "ellipsis",
      overflow: "hidden",
      whiteSpace: "normal",
      opacity: "0.5",
    });
  });

  it("renders JSX element children", () => {
    render(
      <Ellipsify data-testid="wrap">
        <strong data-testid="inner">Bold</strong>
      </Ellipsify>,
    );
    expect(screen.getByTestId("inner")).toHaveTextContent("Bold");
    expect(screen.getByTestId("wrap")).toContainElement(screen.getByTestId("inner"));
  });

  it("renders non-string children", () => {
    render(<Ellipsify data-testid="ellipsify">{42}</Ellipsify>);
    expect(screen.getByTestId("ellipsify")).toHaveTextContent("42");
  });
});
```

### Scoring (1–5 scale)

| Criterion                        | Variant A (with rule) | Variant B (no rule) | Winner |
|----------------------------------|:---------------------:|:-------------------:|--------|
| Imports from `test-utils`        | 5                     | 1                   | **A**  |
| `beforeEach` cleanup             | 5                     | 1                   | **A**  |
| Uses async `render()` / `await`  | 5                     | 1                   | **A**  |
| No `.only` / `.skip`             | 5                     | 5                   | Tie    |
| Colocated test file              | 5                     | 5                   | Tie    |
| Vitest globals usage             | 5                     | 5                   | Tie    |
| Test coverage breadth            | 4                     | 5                   | **B**  |
| Assertion quality                | 4                     | 5                   | **B**  |
| Pragmatic for component scope    | 3                     | 5                   | **B**  |
| **Total**                        | **41**                | **33**              | **A**  |

### Analysis

#### Variant A strengths
- Followed all project conventions: imports from `test-utils`, `beforeEach` with `unmountComponent()` / `localStorage.clear()` / `reseed(7)`, and `await render(...)`.
- Consistent with the codebase's testing patterns — a new contributor reading this test sees the same structure as every other test file.

#### Variant A weaknesses
- Over-engineered for a pure presentational component: wrapping `<Ellipsify>` inside `<><Excalidraw />…</>` adds boot cost and complexity that is unnecessary when no canvas or app state is involved.
- Individual style property assertions (`el.style.textOverflow`) are more verbose than the object matcher `toHaveStyle({…})`.

#### Variant B strengths
- Pragmatically lighter — no `<Excalidraw />` wrapper, synchronous renders, simpler test code.
- Better assertion style: `toHaveStyle({…})` as a single object check, `toContainElement()`, testing numeric children (`{42}`).

#### Variant B weaknesses
- Imports directly from `@testing-library/react` instead of the project's `test-utils` re-export — **violates the #1 rule**.
- No `beforeEach` cleanup — potential for state leakage across tests in a larger suite.
- Would fail code review against this project's conventions.

### Conclusion

The testing rule **significantly improves project-convention adherence** (imports, cleanup, async render). Without the rule, the agent defaults to generic React Testing Library patterns that violate three of the six enforceable rules. The rule scored **41 vs 33** overall.

**Improvement opportunity:** The rule could add a note that for simple presentational components that do not depend on Excalidraw state or canvas, tests may use `render(<Component />)` directly (still imported from `test-utils`) without wrapping in `<Excalidraw />`, to avoid unnecessary overhead. This would combine Variant A's convention compliance with Variant B's pragmatism.

---

*To add future A/B tests, append a new `## Test N` section following the same structure above.*
