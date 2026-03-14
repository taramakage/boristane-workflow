---
name: boristane-workflow
description: >
  Enforces a strict three-phase workflow (Research → Planning → Implementation)
  for software engineering tasks. Use when starting any non-trivial coding task,
  implementing a new feature, fixing a complex bug, or refactoring. Prevents
  writing code before a written plan is reviewed and approved by the developer.
---

# Boris Tane Workflow

Three strictly ordered phases. Never skip ahead. Never write code before the plan is approved.

All workflow artifacts are stored in a **session directory** at the project root under `.boristane/`. Two naming conventions keep active and archived sessions visually distinct:

- **Active session**: `.boristane/wip-<task-slug>/` — created at session start, lives here until archiving (e.g., `.boristane/wip-add-auth-middleware/`, `.boristane/wip-fix-login-bug/`).
- **Archived session**: `.boristane/action-YYMMDD-HHMM/` — created at session end (e.g., `.boristane/action-260314-1530/`).

Multiple `wip-*` directories can coexist, allowing parallel sessions without conflicts.

---

## State machine

**Run this at the start of every session.**

1. Derive `<task-slug>` from the task description (short, kebab-case). Set `SESSION_DIR = .boristane/wip-<task-slug>/`. Example: task "add auth middleware" → `SESSION_DIR = .boristane/wip-add-auth-middleware/`.
2. If `SESSION_DIR` does not exist → create it and `SESSION_DIR/status.md` using the template below, then start Phase 1.
3. If `SESSION_DIR` exists but `SESSION_DIR/status.md` does not → treat as case 2.
4. If `SESSION_DIR/status.md` exists and all items are marked `[x]` → archive it (see "Archiving" below), then start Phase 1 fresh in the same or a new `SESSION_DIR`.
5. If `SESSION_DIR/status.md` exists with incomplete items → read it, check **Current phase**, and resume from there.

Phases advance strictly in order: **Research → Planning → Implementation**

`SESSION_DIR/status.md` is the single source of truth for session state. After every phase transition or completed step, update it and set `**Last updated**`.

### `SESSION_DIR/status.md` template

```markdown
# Workflow Status

**Current phase**: 1 (Research)
**Last updated**: [timestamp]

## Phase 1: Research
- [ ] research.md written
- [ ] Developer approved research

## Phase 2: Planning
- [ ] plan.md written
- [ ] Annotation cycle complete (developer approved plan)
- [ ] To-Do checklist added to plan.md

## Phase 3: Implementation
- [ ] All To-Do items completed
- [ ] Final build/typecheck passes
- [ ] Tests run and results reported (failures resolved or accepted by developer)
- [ ] Docs updated or doc conflicts reported to developer
```

Mark items `[x]` as they are completed. Update `**Current phase**` and `**Last updated**` on every change.

---

## Phase 1: Research

**Goal**: Understand the codebase deeply and thoroughly — all intricacies, all edge cases. Document everything. Write no code.

### Steps

1. Identify scope with the developer if not already stated.
2. Read all relevant files thoroughly — patterns, conventions, dependencies, edge cases, types. Also read any project documentation (README, ADRs, inline docs, changelogs) that touches the scope. Note where docs conflict with or lag behind the actual code — do not silently trust stale docs.
3. Write findings to `SESSION_DIR/research.md` (e.g., `.boristane/wip-add-auth-middleware/research.md`) using this structure:

```markdown
# Research: [Feature/Task Name]

## Key files
- `path/to/file` — responsibility, notable code paths

## Patterns and conventions
- Naming, error handling, module structure, data shapes

## Dependencies
- How components relate to each other

## Constraints
- Things that must not break; performance or compatibility requirements

## Gotchas
- TODOs, commented-out code, surprising behavior, known tech debt

## Doc conflicts
- Docs that contradict the current code or appear outdated (list file, discrepancy, and which source is authoritative)

## Open questions
- Ambiguities needing developer clarification before planning
```

4. Stop. Tell the developer: "research.md is ready for your review."

### Do NOT include

- Proposed solutions or approaches
- Speculation about how to fix things
- Any implementation code

### Completion condition

`SESSION_DIR/research.md` gives a developer enough context to understand the system without reading the code themselves.

---

## Phase 2: Planning

**Goal**: Produce an approved written plan. Write no code.

**Requires**: `SESSION_DIR/research.md` reviewed by the developer.

### Reference implementations

If a good implementation exists — in an open-source project, another service in the repo, or elsewhere — share it alongside the plan request. Working from a concrete reference produces better results than designing from scratch. When a reference is provided, cite it in the plan's `## Approach` section and note any adaptations made.

### Writing `plan.md`

Write `SESSION_DIR/plan.md` using this structure:

```markdown
# Plan: [Feature/Task Name]

## Approach
One paragraph. What will be done and why this approach over alternatives.

## Trade-offs
Alternatives considered and why each was rejected or deprioritized.

## Implementation steps
1. **Step name** — target file(s), exact code or pseudocode
2. **Step name** — ...
   (Each step small enough to verify independently; sequenced to build cleanly.)

## Open questions
Decisions the developer must make before implementation begins.
```

### Annotation cycle

Tell the developer: "`SESSION_DIR/plan.md` is ready. Please annotate it and ask me to address your notes."

**When the developer adds notes to `SESSION_DIR/plan.md`:**

1. Read the entire updated file — do not skim.
2. Address every annotation — correct assumptions, adopt their approach, incorporate domain knowledge.
3. Do not silently preserve a rejected approach.
4. Update `SESSION_DIR/plan.md` in place — do not create versioned copies.
5. Say: "All notes addressed. Please review again."
6. Repeat until the developer approves (typically 1–6 cycles).

**Diagnostic branch**: If annotations are corrections of factual errors (not preference choices), the research phase was insufficient. Offer to deepen `SESSION_DIR/research.md` before continuing the annotation cycle.

#### Annotation examples

Annotations vary from two words to full paragraphs:

- `"not optional"` — next to a parameter marked as optional
- `"use drizzle:generate for migrations, not raw SQL"` — domain knowledge injection
- `"no — this should be a PATCH, not a PUT"` — correcting an assumption
- `"remove this section entirely, we don't need caching here"` — rejecting an approach
- `"this is wrong, the visibility field needs to be on the list itself, not on individual items. restructure the schema section accordingly"` — redirecting an entire section

### Finalizing

Before finalizing, verify all `## Open questions` items are resolved. If any remain, ask the developer to decide before proceeding.

On approval, prepend a `## To-Do` checklist to `SESSION_DIR/plan.md`:

```markdown
## To-Do
- [ ] Step 1: ...
- [ ] Step 2: ...
```

Say: "Plan finalized. Ready to implement on your go."

**Never begin implementation without explicit developer approval.**

---

## Phase 3: Implementation

**Goal**: Execute the approved plan completely. This phase is mechanical, not creative — all decisions were made in Phase 2.

**Requires**: `SESSION_DIR/plan.md` with approved `## To-Do` checklist.

**Freedom level: Low.** Follow the plan exactly. Do not improvise.

### Execution rules

- Work through the checklist top to bottom.
- Mark each item `[x]` in `SESSION_DIR/plan.md` immediately after completing it.
- After every meaningful change, run the project's build/typecheck/lint command and fix errors before moving on. This is a feedback loop — never defer checks to the end.
- Avoid type escape hatches (e.g. `any`, unsafe casts) — use the language's type system properly.
- Match existing code patterns — consult `SESSION_DIR/research.md` and the codebase.
- Keep responses terse: one sentence saying what was done and what's next.

### When to stop and ask

Stop immediately if:

- A step in the plan is ambiguous and two interpretations would produce meaningfully different code.
- An unexpected state in the codebase makes a planned step impossible as written.
- A build or typecheck error requires a structural change not anticipated in the plan.

Do NOT stop for normal progress. The developer approved the plan — execute it.

### Recovery: revert and narrow

When implementation drifts in the wrong direction, do not try to patch it. Revert the git changes and re-scope with a narrower focus. Narrowing scope after a revert almost always produces better results than incrementally fixing a bad approach.

### Completion checklist

- [ ] All `SESSION_DIR/plan.md` to-do items marked `[x]`
- [ ] Final build/typecheck passes with zero errors
- [ ] Available tests run; failures resolved or accepted by developer with justification
- [ ] No debug logs, commented-out code, or placeholder TODOs left behind
- [ ] Docs updated to reflect changes (README, ADRs, inline docs); any doc conflicts flagged in research.md resolved or reported to developer
- [ ] Archive completed (see "Archiving" below)

---

## Archiving

When all items in `status.md` are marked `[x]`, archive the session:

1. Rename `SESSION_DIR` (e.g., `.boristane/wip-add-auth-middleware/`) to a timestamped directory using the current date and time: `action-YYMMDD-HHMM` (YY=year, MM=month, DD=day, HH=hour, MM=minute). Example: `.boristane/action-260314-1530/`.
2. The `.boristane/` root is now clean and ready for the next session.

The `wip-` prefix marks active sessions; the `action-` prefix marks archived ones — they are always visually distinct at a glance.

This is always the **last** step of Phase 3. Do not archive mid-session or before all checklist items are complete.
