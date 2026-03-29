# Documentation

This folder holds **product**, **technical**, and **memory-bank** documentation for the Excalidraw monorepo. The [memory bank](#memory-bank-pattern) is the lightweight, agent-friendly layer; deeper references live alongside it.

---

## Memory bank pattern

A **memory bank** is a small set of **curated Markdown files** that act as **durable project memory** for humans and coding agents. Instead of rediscovering the same facts every session, you maintain **fixed roles** for each file and **update** them as the project evolves.

### Why use it

- **Continuity:** New sessions start from stable briefs (purpose, stack, patterns) instead of guessing from code alone.
- **Layered depth:** Hot “what we’re doing now” stays separate from slow-changing architecture and product specs.
- **Single source of truth:** One place to align terminology, decisions, and “where to look next” in the repo.

### How it works

1. **Define layers** — Separate **rolling state** (changes often) from **reference** (changes rarely).
2. **Name files by role** — Each file answers one kind of question (e.g. “what is this project?”, “how is code organized?”, “what are we focused on today?”).
3. **Link outward** — Memory files stay short; they **point to** long PRDs, architecture diagrams, and setup guides instead of duplicating them.
4. **Refresh deliberately** — After meaningful work, update **active context** and **progress**; promote durable learnings into **patterns**, **tech context**, or **decision logs**.

### Keeping the memory bank updated

**Agents and humans:** treat the memory bank as part of the deliverable. After **each substantive change** to the codebase or repo behavior, refresh memory files so future work does not rely on stale notes.

| When | Update |
|------|--------|
| Every meaningful session or PR-sized change | [memory/activeContext.md](memory/activeContext.md) — focus, what changed, next steps |
| Scope or completion shifts | [memory/progress.md](memory/progress.md) |
| New conventions, refactors, or API patterns | [memory/systemPatterns.md](memory/systemPatterns.md) |
| Dependencies, scripts, env, or CI | [memory/techContext.md](memory/techContext.md) |
| Non-obvious “why we chose X” | [memory/decisionLog.md](memory/decisionLog.md) |
| Charter or high-level product behavior | [memory/projectbrief.md](memory/projectbrief.md), [memory/productContext.md](memory/productContext.md) |

If the truth lives in long documents, edit [technical/](technical/) or [product/](product/) and add a short pointer from the relevant memory file.

Repo-root agent instructions: [AGENTS.md](../AGENTS.md) (Memory Bank section).

### Memory bank in this repository

| File | Role |
|------|------|
| [memory/activeContext.md](memory/activeContext.md) | **Rolling session layer:** current focus, recent changes, immediate next steps, doc map. |
| [memory/projectbrief.md](memory/projectbrief.md) | **Charter:** purpose, stack summary, success criteria, code entry points. |
| [memory/productContext.md](memory/productContext.md) | **Product lens:** personas, UX principles, journeys, shell workflows. |
| [memory/techContext.md](memory/techContext.md) | **Tooling:** versions, layout, scripts, env and deployment pointers. |
| [memory/systemPatterns.md](memory/systemPatterns.md) | **Code patterns:** ActionManager, Jotai, tunnels, data flow, naming conventions. |
| [memory/decisionLog.md](memory/decisionLog.md) | **Decisions:** compact table of major choices and pending items. |
| [memory/progress.md](memory/progress.md) | **Status:** completion notes, WIP, roadmap hints, known hotspots. |

**Deeper docs** (loaded when detail is needed):

- [technical/](technical/) — Full architecture, dev setup, extended decision records.
- [product/](product/) — PRD, glossary, feature traceability and appendices.

### Rule of thumb for agents

Default to **memory** files for breadth and navigation; open **technical** or **product** docs when you need diagrams, full env tables, PRD-level requirements, or long-form rationale. The doc map in [memory/activeContext.md](memory/activeContext.md) lists when to load each path.

Project-wide agent constraints and commands are summarized in the repo root [AGENTS.md](../AGENTS.md), including **keeping the memory bank updated after changes**.
