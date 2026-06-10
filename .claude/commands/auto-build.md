# /auto-build

Autonomous multi-phase build mode. Carries the project forward from its current
state as far as the user specifies, running the full implementation pipeline on
each backlog item and proceeding automatically when all quality gates pass. Stops
only when something is truly broken or a decision cannot be safely deferred.

---

## Pre-flight

### Pending items check

Before checking for a prior build plan, check whether persistent tracking files
exist with unresolved content:

- Run `ls .build/OPEN_QUESTIONS.md 2>/dev/null || echo absent` — if present, read
  the Unresolved section and count items
- Run `ls .build/TEST_TRACKER.md 2>/dev/null || echo absent` — if present, first
  run the "Pre-step — Sweep externally-checked items" procedure defined at the
  top of `.claude/commands/test-companion.md` to move any `- [x]` entries from
  Pending to Completed, then read the Pending section and count items

If either has content, report the count to the user:
> "X open questions and Y pending tests remain from a prior build."

Ask: "Would you like to review them now before starting, or carry them forward to
be addressed at the end of this build?"

- **Review now**: follow the `/test-companion` process to work through these
  items. When the session is complete, return here and continue with the prior
  build check below.
- **Carry forward**: proceed — the items will persist and be consolidated with new
  items at the end of this build.

### Prior build check

If `.build/BUILD_PLAN.md` exists, offer to resume the previous build or start
fresh. If resuming, skip to Phase 3 and continue from the first uncompleted item
in BUILD_PLAN.md.

---

## Phase 1 — Clarification

### 1a. Read project state

Read in full: `BACKLOG.md`, `docs/project-vision.md`, `docs/architecture.md`.
Run `gh pr list --state open` and `gh issue list --state open` to check for
in-flight work.

### 1b. Select build scope

List each backlog phase with a one-line summary of its contents. Ask the user how
far they want to build — a phase boundary (e.g., "through Phase 1"), specific items
by number, or the full backlog.

This scope choice is the one required input before planning. Get it, then build the
plan without further interruption.

### 1c. Build the plan (internal — do not stop between these)

For the selected scope, work out the full plan before presenting anything:

- **Sequence:** order items dependency-clean (infrastructure before features, data
  models before UI, auth before protected routes).
- **Exclusions:** flag any item to drop because it needs infrastructure or a
  dependency not established earlier, would likely force revisiting other items, or
  is a stretch/post-launch goal — one-line reason each.
- **Decisions & defaults:** for each decision an item needs, record the reasonable
  default you will use if the user does not override it.
- **Prerequisites:** list env vars, third-party services, or local setup that must
  happen before the first item runs.
- **Blocking questions only:** list only questions that genuinely block
  implementation and have no safe default. Everything else uses its default — do
  not ask.

Self-check the result: is the sequence dependency-clean, does every item have
enough spec to proceed on a default, and are all prerequisites identified? Resolve
any gaps before presenting.

### 1d. Confirm the plan

Present in one message:
- The numbered development sequence
- Recommended exclusions and why
- Key decisions and the defaults to be used (one line each)
- Prerequisites to run before the build starts
- Any blocking questions from 1c

**Stop here once.** The user confirms, adjusts the sequence/exclusions/decisions,
and answers any blocking questions in a single pass. Re-confirm only if their
answers materially change the plan.

### 1e. Write `.build/BUILD_PLAN.md`

Create the `.build/` directory if it does not exist.

```markdown
# Build Plan

## Scope
[Selected phase(s) or item list]

## Development Sequence
1. [Item name] — [one-line description; note key files or approach if not obvious]
2. ...

## Key Decisions
- [Decision]: [choice made and why]

## Deferred Decisions
- [Decision]: [default chosen, revisit condition]

## Exclusions
- [Item]: [reason]

## Prerequisites
- [Any env setup, global installs, or manual steps to complete before starting]
```

---

## Phase 2 — Setup

### 2a. Front-load prerequisites

For each item in the Prerequisites list from BUILD_PLAN.md:
- Present all prerequisite commands as a grouped list
- Get one confirmation from the user
- Run them sequentially, stopping if any fails

These are the only commands that may operate outside the project directory
(e.g., `npm install -g`, `brew install`, database setup). All subsequent
commands are scoped to the project.

### 2b. Initialize tracking files

**Session artifacts** — create fresh each build:

Create `.build/BUILD_SUMMARY.md`:

```markdown
# Build Summary

## Status: In Progress

## Items Completed
[Populated as each item merges.]

## Items Remaining
[Full sequence from BUILD_PLAN.md — items removed as they complete.]

## Key Decisions
[Populated as the build proceeds — one entry per decision made, in sequence.]
```

Open questions are written straight to `.build/OPEN_QUESTIONS.md` in real time (see
below) — there is no separate questions file.

**Persistent tracking files** — create only if absent:

Run `ls .build/OPEN_QUESTIONS.md 2>/dev/null || echo absent`:
- If absent, create `.build/OPEN_QUESTIONS.md`:

```markdown
# Open Questions

<!--
Entry format (Unresolved section):
- **Q**: [question text]
  **Default used**: [what was chosen to unblock the build]
  **Revisit condition**: [what would trigger revisiting this decision]
  **Added**: Build [YYYY-MM-DD]
-->

## Unresolved

## Resolved
```

Run `ls .build/TEST_TRACKER.md 2>/dev/null || echo absent`:
- If absent, create `.build/TEST_TRACKER.md`:

```markdown
# Test Tracker

<!-- Checkbox conventions: [ ] pending, [x] passed, [!] failed (known failure) -->

## Pending

## Completed
```

If either file has items in its Unresolved or Pending section when Phase 2 begins
(whether because the user chose "carry forward" in pre-flight, or chose "review now"
but didn't clear everything), append a note to the Status line in BUILD_SUMMARY.md:
`## Status: In Progress — N open questions and M pending tests carried forward`

---

## Phase 3 — Build loop

For each item in the development sequence from BUILD_PLAN.md:

### Per-item pipeline

There is no per-item session-plan file and no per-item confirmation gate — the
whole plan, including each item's decisions and done-condition, was confirmed once
in Phase 1d. The item's entry in BUILD_PLAN.md is its spec.

**3a. Run the per-item pipeline inline**

Run the same 10-step inline pipeline defined in `/next-step` Step 5b, directly in
this session as your own orchestration checklist, with these per-item adjustments:

- **Spec source:** implement from this item's entry in BUILD_PLAN.md (description,
  key files, and any relevant rows from the Key Decisions / Deferred Decisions
  sections) instead of from a session-plan file.
- **Branch:** work on `feature/<item-slug>` for this item.
- **Done-condition:** use the standard condition — "PR is open, all CI checks pass,
  and the item behaves as described in BUILD_PLAN.md" — unless the item was flagged
  in Phase 1 as needing a custom one.
- **Open questions:** when one arises that cannot be resolved autonomously, append
  it to `.build/OPEN_QUESTIONS.md` (Unresolved section) with question, default used,
  revisit condition, and today's date. (No separate questions file.)
- **BACKLOG:** mark the item complete as part of step 5 of the pipeline.

Stop and return control to the user any time a step requires a decision you cannot
resolve autonomously.

> Optional: for a long, unattended build you may instead paste the pipeline
> sequence into the built-in `/goal` command yourself, to get a Stop-hook
> completion guard that re-checks the done condition after each turn. This is a
> manual fallback only — the orchestrator does not invoke `/goal`.

When the item's pipeline completes (done condition met), proceed to 3b.

**3b. Docs and stub check** (per Step 6 of `/next-step`).

**3c. Merge and continue.** Once CI passes:
- Run `gh pr merge <number> --merge --delete-branch`
- Run `git checkout dev`
- Run `git pull`
- Append to `.build/BUILD_SUMMARY.md`:
  ```
  ✓ [Item name] — [3-sentence summary] — PR #[n]
  ```
- Proceed to the next item

### Stop conditions

Stop the loop and report to the user when:
- CI fails and the fix requires a design decision (cannot be resolved with a
  reasonable default that is refactorable later without touching other items)
- Two items have a dependency conflict that makes safe ordering impossible
- A prerequisite from Phase 2 was skipped and is now blocking progress
- The build runner context approaches its limit (see Notes)

**Log and continue** (do not stop) when:
- An open question arises that has a reasonable default answer — record it in
  `.build/OPEN_QUESTIONS.md` per the per-item adjustments above and continue; the
  orchestrator does not need to act on it
- An item requires a minor refactor to a previously completed item that does
  not affect unrelated code — fold it into the current PR
- CI fails with a mechanical fix (type error, lint, broken import) — fix
  autonomously per the standard pipeline

### Context discipline

After each item's pipeline completes, write completion status to BUILD_SUMMARY.md
(branch name, PR number, 3-sentence summary) and treat the tracking files as the
source of truth. Re-read BUILD_PLAN.md and BUILD_SUMMARY.md from disk when needed
rather than relying on accumulated context. The orchestrator does not need to
carry forward implementation detail between items.

---

## Phase 4 — Wrap-up

### 4a. Finalize open questions

Open questions were written to `.build/OPEN_QUESTIONS.md` in real time during
Phase 3, so no migration is needed. As a defensive check, scan the build for any
deferred decision that was not recorded there and append it now with today's date.

### 4b. Spawn a test-plan sub-agent

Give it:
- The full item list from BUILD_SUMMARY.md
- The complete list of changed files across the build
- `docs/user-guide.md` if it exists and is populated (no `> **Template:**` stub
  marker) — for user-facing flow coverage

The sub-agent produces test entries in TEST_TRACKER.md format, grouped by app
location/screen. It should cover happy paths and key edge cases. Each entry must
follow this format:

```markdown
### [Area — e.g., "Login", "Dashboard", "Settings"]

- [ ] [Test title]
  **Steps:** 1. [step]. 2. [step].
  **Expected:** [what the user should see or experience]
  **Added:** Build [YYYY-MM-DD]
```

Append the output to the Pending section of `.build/TEST_TRACKER.md` — do not
replace existing entries.

### 4c. Consolidate tracking files

Spawn a consolidation sub-agent. Give it:
- Full contents of `.build/OPEN_QUESTIONS.md`
- Full contents of `.build/TEST_TRACKER.md`
- The list of items built in this build (from BUILD_SUMMARY.md Items Completed)

The sub-agent rewrites both files in place:

**For OPEN_QUESTIONS.md (Unresolved section only):**
- Merge duplicate or near-duplicate questions into a single entry, combining
  defaults and revisit conditions
- Remove any question that was definitively answered by the work in this build
- Do not touch the Resolved section

**For TEST_TRACKER.md (Pending section only):**
- Merge duplicate or overlapping tests into a single entry
- Group remaining tests by app location/screen/user flow (e.g., "Login",
  "Dashboard", "Settings"). Create a `### [Area]` heading for each group.
- Within each group, order tests to minimize back-and-forth navigation — complete
  everything testable in one location before moving to another
- Across groups, order groups by natural user flow sequence (onboarding before
  features, setup before use, happy path before edge cases)
- Honor explicit dependency notes — if a test says "requires X first," preserve
  that ordering even if it crosses group boundaries
- Do not touch the Completed section

### 4d. Complete BUILD_SUMMARY.md

```markdown
# Build Summary

## Status: Complete

## Items Built
[Full list with PR numbers]

## Items Excluded
[From BUILD_PLAN.md exclusions]

## Key Decisions
[From the Key Decisions section of BUILD_SUMMARY.md, accumulated during the build]

## Open Questions
See `.build/OPEN_QUESTIONS.md` — [N] unresolved items

## Test Plan
See `.build/TEST_TRACKER.md` — [N] pending tests
```

### 4e. Final report

Report to the user:
- Build complete (or partial, with reason for stopping)
- PR list with merge status
- Link to `.build/BUILD_SUMMARY.md`
- Count of open questions and pending tests, with a prompt to run `/test-companion`

---

## Notes

- **Context budget:** Each sub-agent starts fresh. The orchestrator accumulates
  roughly one brief summary per item — for most projects (5–20 items) this is
  well within limits. For very large builds (30+ items), if the orchestrator
  context grows large, stop at a natural phase boundary, report status, and
  offer to continue in a new session (BUILD_PLAN.md and BUILD_SUMMARY.md
  preserve full state for resuming).
- **`.build/` is gitignored.** These files are session artifacts, not project
  history. Add `.build/` to `.gitignore` if not already present.
- **Persistent files survive builds.** OPEN_QUESTIONS.md and TEST_TRACKER.md
  accumulate across builds. BUILD_PLAN.md and BUILD_SUMMARY.md are per-build
  artifacts and may be overwritten.
