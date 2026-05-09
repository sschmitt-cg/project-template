# /launch-session

Guide the user through a structured launch session for a new project, or kick off work
in a project repo that was already scaffolded from the template. By the end of this
command, the project will have a vision doc, architecture doc, and backlog; optionally,
a GitHub repo will be created and cloned locally; the folder will be scaffolded with
all standard files; and the project will be registered in `PROJECTS.md` if applicable.

**Default project location:** `/Users/scottschmitt/Documents/Code Projects`
**Template source:** `/Users/scottschmitt/Documents/Code Projects/project-template`
**Project registry:** `/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/PROJECTS.md`

Work conversationally. Do not present all questions as a list — ask a few at a time,
listen, and build on what the user shares. The goal is a real discovery conversation,
not a form to fill out.

---

## Phase 0: Detect context

Before starting, silently check the following. Do not narrate this to the user.

**Is this an existing scaffolded repo?**
Check whether the current working directory already has the project-template structure:
look for `CLAUDE.md`, a `docs/` folder, `.claude/`, `BACKLOG.md`, and `INBOX.md`.
If at least three of these are present, treat this as **existing repo mode**.

**Did the user provide input?**
Check `$ARGUMENTS`. If non-empty, determine whether it looks like a file path or an
idea dump:
- A **file path**: starts with `/` or `./`, ends with a known extension (`.md`, `.txt`),
  or is a short token (no spaces) that matches an existing file in the current directory.
  Read the file and treat its contents as the idea dump.
- An **idea dump** (freeform text): use it directly.
If a file path is detected but the file cannot be read, tell the user and ask them to
paste the content directly or provide a corrected path — then wait before proceeding.

**What docs already exist?**
If in existing repo mode, check which of `docs/project-vision.md`,
`docs/architecture.md`, and `BACKLOG.md` already exist and are populated (not stubs).
A stub contains the marker `> **Template:**`. A file that does not exist is not populated.

**Is PROJECTS.md accessible?**
Try to read `/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/PROJECTS.md`.
Set a flag `projects_md_accessible` based on whether the file exists and is readable.

With these flags set, proceed to Phase 1.

---

## Phase 1: Discovery

Choose the intake mode based on what Phase 0 found.

**Special case — existing repo, all docs populated, no input:**
If in existing repo mode, all three docs are populated, and `$ARGUMENTS` is empty,
do not launch a discovery interview. Instead, ask: "What would you like to update?"
Wait for the user's answer, then proceed with only the relevant questions and doc
regeneration for what they want to change.

### Mode A — No input provided

Run the full discovery conversation. Ask questions in natural conversation to establish
the following. You do not need to ask each as a separate turn — use judgment to group
related ones.

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

### Mode B — Idea dump or file provided

Read the input (or the file contents) and synthesize what was provided. In a single
turn: present a brief summary (three to five sentences covering what the project is,
who it's for, the core problem, any platform/stack signals, and the rough scope), then
immediately follow with any targeted follow-up questions for information that is missing
or genuinely ambiguous. Ask all questions in that same message — do not wait for
confirmation before asking them.

Questions to ask only if missing or genuinely ambiguous (skip any that can be reasonably
inferred from the input):

- Project name (if not stated)
- Primary audience (if vague or unstated)
- Platform(s) (if not mentioned)
- Tech stack preferences or constraints (if not indicated)
- MVP cut: if the dump lists many features, which are must-have vs. nice-to-have?
- What is explicitly out of scope?
- Any known risks or open questions?

Do not re-ask for information already present in the dump. The goal is to fill gaps,
not re-cover ground. Wait for the user's response, then proceed to Phase 2.

---

## Phase 2: Document generation

**Before generating anything:** if in existing repo mode, check which docs are
populated (from Phase 0). For each populated doc, tell the user up front which files
would be replaced and ask for confirmation before generating those specific docs.
Only generate docs the user has confirmed, plus any that don't yet exist.

Generate the following three documents (or the confirmed subset) based on the discovery
conversation. Present each one fully before asking for final confirmation — do not write
any files yet.

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

For new projects, use `**Status:** Pre-development`. For existing repos, infer the
current state from any available signals (existing BACKLOG.md content, what the user
has shared about project progress, phase completion) and propose it to the user for
confirmation before using it (e.g., "I'll set the status to 'In development — Phase 1'
— does that sound right?").

```
# Backlog

**Status:** [see above]

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

After presenting the generated documents, ask: "Does this capture the project
accurately? Any changes before I write these files?"

Apply all requested changes, then proceed to Phase 3. Carry forward which specific
docs were confirmed for writing — only write those files.

---

## Phase 3: Repository setup

This phase has two paths depending on whether the repo already exists.

### If in existing repo mode

Skip repo creation, cloning, and scaffold creation — those are already done.

**CLAUDE.md check**
Check whether `CLAUDE.md` exists in the repo root:
- If it **exists**, ask: "There's already a CLAUDE.md here — do you want to replace it
  with the current template version, or keep the existing one?" Copy from the template
  only if they confirm.
- If it **does not exist**, ask: "There's no CLAUDE.md in this repo — do you want to
  copy in the standard one from the project template?" Copy it if they confirm. Source:
  `/Users/scottschmitt/Documents/Code Projects/project-template/CLAUDE.md`. If that
  file is not found, fall back to:
  `/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/templates/CLAUDE.md`
  and warn the user that the project-template folder may not be set up.

**Write the confirmed documents**
Write only the docs that were confirmed in Phase 2 (`docs/project-vision.md`,
`docs/architecture.md`, and/or `BACKLOG.md`). Do not write any doc the user chose
to keep.

**Commit the changes**
After writing, offer to commit the updated files:
```
git add <only the files that were written>
git commit -m "docs: update project docs"
```
Use a more specific commit message if the scope is clear (e.g., `docs: regenerate
vision doc and backlog`). Do not commit files that were kept unchanged.

**PROJECTS.md registration**
If `projects_md_accessible` is true, ask: "Should I register this project in
PROJECTS.md?" If yes, add a new row with the name, description, GitHub URL (if known),
status (`Active`), and start date. Confirm this edit with the user before writing,
per the firm document protection rule.

If `projects_md_accessible` is false, skip this step silently.

### If this is a new project

Execute the following steps. Confirm the specifics with the user before running any
commands.

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

**Step 5 — CLAUDE.md**
Copy the standard `CLAUDE.md` from the project template into the root of the new repo
unless the user explicitly says they don't want it. Source:
`/Users/scottschmitt/Documents/Code Projects/project-template/CLAUDE.md`

If that file is not found, fall back to:
`/Users/scottschmitt/Documents/Claude/Projects/Project Planning and Process/templates/CLAUDE.md`
and warn the user that the project-template folder may not be set up.

**Step 6 — Write the generated documents**
Write the confirmed `docs/project-vision.md`, `docs/architecture.md`, and `BACKLOG.md`
to the new repo.

**Step 7 — Initial commit**
```
git add BACKLOG.md INBOX.md SCRATCH.md docs/project-vision.md docs/architecture.md .claude/
git commit -m "init: project scaffold and vision docs"
git push
```
If CLAUDE.md was included (Step 5), add it to the `git add` line. If it was skipped,
omit it.

**Step 8 — Register the project**
If `projects_md_accessible` is true, add a new row to `PROJECTS.md` in the Project
Planning and Process folder with the name, description, GitHub URL, status (`Active`),
and start date. Confirm this edit with the user before writing, per the firm document
protection rule.

If `projects_md_accessible` is false, skip this step silently.

---

## Phase 4: Handoff

**If this was a new project:**
Tell the user:
- The local path where the repo was cloned
- That they can open this folder in VS Code to begin development
- That `/next-step` is available in Claude Code for driving development sessions
- That ideas can be routed to `INBOX.md` at any time via the Apple Notes inbox workflow

**If this was an existing repo:**
Confirm which docs were updated (or note if none were changed). Remind the user that
`/next-step` is available whenever they're ready to start a development session.

---

## Error handling

- If `gh` is not installed or not authenticated, stop and tell the user to run
  `gh auth login` before proceeding.
- If the repo name is already taken on GitHub, suggest alternatives and wait for a choice.
- If the clone fails due to SSH key issues, suggest `gh repo clone <name>` as an
  alternative and note that HTTPS can be used if SSH is not configured.
- If a provided file path does not exist or cannot be read, tell the user and ask
  them to paste the content directly or provide a corrected path.
- If any git operation fails (commit, push, permissions), stop, report the issue, and
  suggest the minimal manual resolution. Do not attempt to continue past a failed step.
