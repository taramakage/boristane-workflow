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

All workflow artifacts are stored in the `.boristane/<short-summary-title>/` directory at the project root to avoid conflicts with project files. Completed sessions are archived into timestamped subdirectories (e.g., `.boristane/action-260223-1004/`).

---

## State machine

**Run this at the start of every session.**

1. If `.boristane/` does not exist → create it and `.boristane/status.md` using the template below, then start Phase 1.
2. If `.boristane/status.md` exists and all items are marked `[x]` → the previous session is complete. Archive it (see "Archiving" below), then create a fresh `status.md` and start Phase 1 for the new task.
3. If `.boristane/status.md` exists with incomplete items → read it, check **Current phase**, and resume from there.

Phases advance strictly in order: **Research → Planning → Implementation**

`.boristane/status.md` is the single source of truth for session state. After every phase transition or completed step, update it and set `**Last updated**`.

### `.boristane/status.md` template

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
- [ ] Final typecheck passes
- [ ] Tests run and results reported
```

Mark items `[x]` as they are completed. Update `**Current phase**` and `**Last updated**` on every change.

---

## Phase 1: Research

**Goal**: Understand the codebase deeply and thoroughly — all intricacies, all edge cases. Document everything. Write no code.

### Steps

1. Identify scope with the developer if not already stated.
2. Read all relevant files thoroughly — patterns, conventions, dependencies, edge cases, types.
3. Write findings to `.boristane/research.md` using this structure:

```markdown
# Research: [Feature/Task Name]

## Key files
- `path/to/file.ts` — responsibility, notable code paths

## Patterns and conventions
- Naming, error handling, module structure, data shapes

## Dependencies
- How components relate to each other

## Constraints
- Things that must not break; performance or compatibility requirements

## Gotchas
- TODOs, commented-out code, surprising behavior, known tech debt

## Open questions
- Ambiguities needing developer clarification before planning
```

4. Stop. Tell the developer: "research.md is ready for your review."

### Do NOT include

- Proposed solutions or approaches
- Speculation about how to fix things
- Any implementation code

### Completion condition

`.boristane/research.md` gives a developer enough context to understand the system without reading the code themselves.

---

## Phase 2: Planning

**Goal**: Produce an approved written plan. Write no code.

**Requires**: `.boristane/research.md` reviewed by the developer.

### Writing `plan.md`

Write `.boristane/plan.md` using this structure:

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

**Reference implementation trick**: When a good implementation exists in another codebase, share that code alongside the plan request. Working from a concrete reference produces dramatically better results than designing from scratch.

### Annotation cycle

Tell the developer: ".boristane/plan.md is ready. Please annotate it and ask me to address your notes."

**When the developer adds notes to `.boristane/plan.md`:**

1. Read the entire updated file — do not skim.
2. Address every annotation — correct assumptions, adopt their approach, incorporate domain knowledge.
3. Do not silently preserve a rejected approach.
4. Update `.boristane/plan.md` in place — do not create versioned copies.
5. Say: "All notes addressed. Please review again."
6. Repeat until the developer approves (typically 1–6 cycles).

**Diagnostic branch**: If annotations are corrections of factual errors (not preference choices), the research phase was insufficient. Offer to deepen `.boristane/research.md` before continuing the annotation cycle.

#### Annotation examples

Annotations vary from two words to full paragraphs:

- `"not optional"` — next to a parameter marked as optional
- `"use drizzle:generate for migrations, not raw SQL"` — domain knowledge injection
- `"no — this should be a PATCH, not a PUT"` — correcting an assumption
- `"remove this section entirely, we don't need caching here"` — rejecting an approach
- `"this is wrong, the visibility field needs to be on the list itself, not on individual items. restructure the schema section accordingly"` — redirecting an entire section

### Finalizing

On approval, prepend a `## To-Do` checklist to `.boristane/plan.md`:

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

**Requires**: `.boristane/plan.md` with approved `## To-Do` checklist.

**Freedom level: Low.** Follow the plan exactly. Do not improvise.

### Execution rules

- Work through the checklist top to bottom.
- Mark each item `[x]` in `.boristane/plan.md` immediately after completing it.
- After every meaningful change, run the project's typecheck command and fix errors before moving on. This is a feedback loop — never defer typecheck to the end.
- Never use `any` types or equivalent escape hatches.
- Match existing code patterns — consult `.boristane/research.md` and the codebase.
- Keep responses terse: one sentence saying what was done and what's next.

### When to stop and ask

Stop immediately if:

- A step in the plan is ambiguous and two interpretations would produce meaningfully different code.
- An unexpected state in the codebase makes a planned step impossible as written.
- A typecheck error requires a structural change not anticipated in the plan.

Do NOT stop for normal progress. The developer approved the plan — execute it.

### Recovery: revert and narrow

When implementation drifts in the wrong direction, do not try to patch it. Revert the git changes and re-scope with a narrower focus. Narrowing scope after a revert almost always produces better results than incrementally fixing a bad approach.

### Completion checklist

- [ ] All `.boristane/plan.md` to-do items marked `[x]`
- [ ] Final typecheck passes with zero errors
- [ ] Available tests run and results reported
- [ ] No debug logs, commented-out code, or placeholder TODOs left behind
- [ ] Archive completed (see "Archiving" below)

---

## Archiving

When all items in `status.md` are marked `[x]`, archive the session:

1. Create a subdirectory named `action-YYMMDD-HHMM` inside `.boristane/` using the current date and time (e.g., `action-260223-1004`).
2. Move `status.md`, `plan.md`, and `research.md` into that subdirectory.
3. The `.boristane/` root is now clean and ready for the next session.

This is always the **last** step of Phase 3. Do not archive mid-session or before all checklist items are complete.
