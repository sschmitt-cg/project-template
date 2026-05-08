# /launch-session

Guide the user through a structured launch session for a new project. By the end of
this command, the project will have a vision doc, architecture doc, and backlog; a
GitHub repo will be created and cloned locally; the folder will be scaffolded with all
standard files; and the project will be registered in `PROJECTS.md`.

**Default project location:** `/Users/scottschmitt/Documents/Code Projects`
**Template source:** `/Users/scottschmitt/Documents/Code Projects/project-template`
**Project registry:** `/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/PROJECTS.md`

Work conversationally. Do not present all questions as a list — ask a few at a time,
listen, and build on what the user shares. The goal is a real discovery conversation,
not a form to fill out.

---

## Phase 1: Discovery

Ask questions in natural conversation to establish the following. You do not need to
ask each as a separate turn — use judgment to group related ones.

**Project fundamentals**
- What is the name of this project?
- Describe it in 2–3 sentences. What does it do, and why does it exist?
- Who is the primary user or audience? Be as specific as possible.
- What is the core problem it solves for them?

**Direction and constraints**
- What platform(s) will this run on? (web, iOS, Android, desktop, API, etc.)
- Any tech stack preferences or constraints? (existing codebases, team familiarity, etc.)
- Are there design sensibilities or references worth knowing about?

**Scope**
- What defines the MVP? What are the 3–5 things it must do to be worth shipping?
- What is explicitly out of scope for now?
- Are there any known risks or open questions that should be logged?

After gathering enough to proceed, summarize what you've heard in a short paragraph and
ask the user to confirm or correct before moving on.

---

## Phase 2: Document generation

Generate the following three documents based on the discovery conversation.
Present each one fully before asking for confirmation — do not write any files yet.

### `docs/project-vision.md`

```
# [Project Name] — Product Vision

## What it is
[2–3 sentence description]

## Problem
[The specific problem being solved, and for whom]

## Target audience
[Specific description of the primary user]

## Goals
- [Goal 1]
- [Goal 2]
- [Goal 3]

## Design principles
- [Principle 1]
- [Principle 2]
- [Principle 3]

## Out of scope (MVP)
- [Item 1]
- [Item 2]

## Open questions
- [Any known unknowns worth tracking]
```

### `docs/architecture.md`

```
# [Project Name] — Architecture

## Tech stack
| Layer | Choice | Rationale |
|---|---|---|
| [e.g. Frontend] | [e.g. React Native] | [brief reason] |

## Key constraints
- [Constraint 1]
- [Constraint 2]

## Folder structure
[Brief sketch — can be updated as the project develops]

## Notes
[Anything else relevant at this stage]
```

### `BACKLOG.md`

```
# Backlog

**Status:** Pre-development

## Phase 1 — MVP

| # | Feature | Notes |
|---|---|---|
| 1 | [Feature name] | [Brief description] |
| 2 | [Feature name] | [Brief description] |
...

## Phase 2 — Post-launch

| # | Feature | Notes |
|---|---|---|
...

## Parking lot
[Ideas worth keeping but not yet scheduled]
```

After presenting all three documents, ask: "Does this capture the project accurately?
Any changes before I write these files and set up the repo?"

Apply all requested changes before proceeding to Phase 3.

---

## Phase 3: Repository setup

After the user confirms the documents, execute the following steps. Confirm the
specifics with the user before running any commands.

**Step 1 — Confirm repo settings**
Ask:
- Should the GitHub repo be public or private? (default: private)
- Confirm the local parent directory. Default is `/Users/scottschmitt/Documents/Code Projects`.
  Only ask if the user has indicated they want a different location.

**Step 2 — Create the GitHub repo**
```
gh repo create <project-name> --private --description "<one-line description>"
```
Use `--public` if the user requested it. Use the project name lowercased and
hyphenated as the repo name (e.g., "My Cool App" → `my-cool-app`).

**Step 3 — Clone the repo locally**
```
git clone git@github.com:<username>/<project-name>.git /Users/scottschmitt/Documents/Code Projects/<project-name>
```

**Step 4 — Scaffold the folder structure**
Inside the cloned repo, create:
```
docs/
.claude/
  commands/
INBOX.md
SCRATCH.md
```

`INBOX.md` should contain only this header:
```
# Inbox

Raw ideas routed here from the Apple Notes inbox. Process with Claude before
promoting anything to the backlog or vision doc.
```

`SCRATCH.md` should contain only this header:
```
# Scratch

Working space for in-session discussion and exploration. Nothing here is authoritative.
Clear freely.
```

**Step 5 — Copy the CLAUDE.md template**
Copy `/Users/scottschmitt/Documents/Code Projects/project-template/CLAUDE.md` into
the root of the new repo as `CLAUDE.md`. If that file is not found, fall back to
`/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/templates/CLAUDE.md`
and warn the user that the project-template folder may not be set up.

**Step 6 — Write the generated documents**
Write the confirmed `docs/project-vision.md`, `docs/architecture.md`, and `BACKLOG.md`
to the new repo.

**Step 7 — Initial commit**
```
git add -A
git commit -m "init: project scaffold and vision docs"
git push
```

**Step 8 — Register the project**
Add a new row to `PROJECTS.md` in the Project Planning and Process folder with the
name, description, GitHub URL, status (`Active`), and start date. Confirm this edit
with the user before writing, per the firm document protection rule.

---

## Phase 4: Handoff

Confirm everything is in place. Tell the user:
- The local path where the repo was cloned
- That they can open this folder in VS Code to begin development
- That `/next-step` is available in Claude Code for driving development sessions
- That ideas can be routed to `INBOX.md` at any time via the Apple Notes inbox workflow

---

## Error handling

- If `gh` is not installed or not authenticated, stop and tell the user to run
  `gh auth login` before proceeding.
- If the repo name is already taken on GitHub, suggest alternatives and wait for a choice.
- If the clone fails due to SSH key issues, suggest `gh repo clone <name>` as an
  alternative and note that HTTPS can be used if SSH is not configured.
- If any step fails, report it clearly and ask how the user wants to proceed.
  Do not attempt to continue past a failed step automatically.
