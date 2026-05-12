Pull the latest shared files from the project template repo (sschmitt-cg/project-template) into the current project. Syncs universal rules, command files (next-step, auto-build, test-companion), and rebuilds settings.json from the template files.

## Before syncing — set up the branch

Before making any file edits, ensure you are on a clean sync branch:

1. Confirm the current branch is `main` (run `git branch --show-current`). If not, stop and tell the user.
2. Check whether `chore/sync-template` already exists:
   - Run `git branch --list chore/sync-template` (local)
   - Run `git branch -r --list origin/chore/sync-template` (remote)
3. If it exists locally or remotely:
   - If it exists locally, run `git checkout chore/sync-template` then `git reset --hard main`
   - If it only exists remotely (not locally), run `git checkout -b chore/sync-template main`
4. If it does not exist anywhere, run `git checkout -b chore/sync-template`

All file edits below are made on this branch, not on `main`.

---

## Files to sync

### 1. CLAUDE.md
Fetch and overwrite the local copy:
```
gh api repos/sschmitt-cg/project-template/contents/CLAUDE.md --jq '.content' | base64 -d
```

### 2. .claude/commands/next-step.md
Fetch and overwrite the local copy:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/commands/next-step.md --jq '.content' | base64 -d
```

### 3. .claude/commands/auto-build.md
Fetch and overwrite the local copy:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/commands/auto-build.md --jq '.content' | base64 -d
```

### 3b. .claude/commands/test-companion.md
Fetch and overwrite the local copy:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/commands/test-companion.md --jq '.content' | base64 -d
```

### 4. One-time stub files

These files should exist in every project but contain project-specific content — create them only if absent; never overwrite.

Check existence with `ls "PROJECT_PATH/FILENAME" 2>/dev/null || echo "absent"` — any output other than `"absent"` means the file is present. Do not use `&&` in the existence check; use only the `||` form.

**INBOX.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/INBOX.md --jq '.content' | base64 -d
```

**SCRATCH.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/SCRATCH.md --jq '.content' | base64 -d
```

**docs/user-guide.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/docs/user-guide.md --jq '.content' | base64 -d
```

**docs/admin-guide.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/docs/admin-guide.md --jq '.content' | base64 -d
```

### 5. .claude/settings.json — rebuild from template files

`settings.json` is gitignored and never committed. This step is the authoritative source for what goes in it.

#### 5a. Determine the project path
Run `git rev-parse --show-toplevel` and store the result as PROJECT_PATH.

#### 5b. Fetch the template files
Fetch each of the following from the template repo:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/universal.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/project.json --jq '.content' | base64 -d
```

#### 5c. Substitute placeholders
In project.json, replace every occurrence of `{{PROJECT_PATH}}` with the actual PROJECT_PATH value before reading any allow entries from it.

Also compute `PROJECT_PATH_ESCAPED` by replacing every space in PROJECT_PATH with `\ ` (backslash-space). For example, if PROJECT_PATH is `/Users/scott/Documents/Code Projects/myapp`, then PROJECT_PATH_ESCAPED is `/Users/scott/Documents/Code\ Projects/myapp`.

For each `Bash(...)` allow entry in the substituted project.json and stack files that contains a **quoted string beginning with PROJECT_PATH** (e.g., `Bash(cmd "PROJECT_PATH/...")`, `Bash("PROJECT_PATH/bin/tool" *)`), generate a parallel escaped-path entry:
- Remove the double-quotes that surround the quoted segment containing PROJECT_PATH
- Replace the PROJECT_PATH portion within that segment with PROJECT_PATH_ESCAPED

These parallel entries are collected alongside the originals in step 5e. Skip this for `Read(...)` entries — the Read tool receives the raw file path directly, not a shell string, so no escaping is needed.

Examples (PROJECT_PATH = `/Users/scott/Code Projects/myapp`):

| Original (quoted) | Parallel (escaped) |
|---|---|
| `Bash(ls "/Users/scott/Code Projects/myapp/**")` | `Bash(ls /Users/scott/Code\ Projects/myapp/**)` |
| `Bash(git -C "/Users/scott/Code Projects/myapp" add *)` | `Bash(git -C /Users/scott/Code\ Projects/myapp add *)` |
| `Bash(cat "/Users/scott/Code Projects/myapp/**")` | `Bash(cat /Users/scott/Code\ Projects/myapp/**)` |
| `Bash("/Users/scott/Code Projects/myapp/venv/bin/python" *)` | `Bash(/Users/scott/Code\ Projects/myapp/venv/bin/python *)` |
| `Bash(cd "/Users/scott/Code Projects/myapp")` | `Bash(cd /Users/scott/Code\ Projects/myapp)` |

#### 5d. Clean up the existing settings.json (if it exists)
Read files in the target project using `cat "PROJECT_PATH/..."` via Bash — not the Read tool. The Read tool prompts for permission on files outside the current project directory; `cat` is auto-allowed.

Read the current `permissions.allow` array. Remove any entry that matches either of these conditions:
1. Contains `&&` — session-approved compound command violations
2. Contains an absolute path starting with `/Users/` (quoted or backslash-escaped) where that path does NOT begin with PROJECT_PATH or PROJECT_PATH_ESCAPED — stale entries scoped to a different project or user directory

Keep all remaining entries.

#### 5e. Compose the final allow list
Collect `permissions.allow` entries from:
- universal.json
- project.json (after placeholder substitution and escaped-path variants from step 5c)
- The surviving entries from the cleanup in 5d

For each source above, interleave the escaped-path variant immediately after its original entry (e.g., the escaped `git -C PROJECT_PATH_ESCAPED add *` entry follows the quoted `git -C "PROJECT_PATH" add *` entry).

Deduplicate, preserving order (universal first, then project, then surviving existing).

#### 5f. Write settings.json
Build the final settings.json as follows:
- Start with the structure from universal.json (this provides the `hooks` and top-level shape)
- Set `permissions.allow` to the composed list from 5e
- If the existing settings.json has keys other than `permissions` and `hooks` (e.g., `additionalDirectories`), preserve them

Write the result to `.claude/settings.json`.

## .gitignore — ensure template files are excluded

`settings.json`, `settings.local.json`, `.build/`, `INBOX.md`, and `SCRATCH.md` are never committed. Verify `.gitignore` has entries for all five.

If `.gitignore` does not exist, create it.
If `.gitignore` does not contain `.claude/settings.json`, append it.
If `.gitignore` does not contain `.claude/settings.local.json`, append it.
If `.gitignore` does not contain `.build/`, append it.
If `.gitignore` does not contain `INBOX.md`, append it.
If `.gitignore` does not contain `SCRATCH.md`, append it.

**Untrack if currently committed:** After updating `.gitignore`, check each:
- Run `git ls-files .claude/settings.json` — if non-empty, run `git rm --cached .claude/settings.json`
- Run `git ls-files .claude/settings.local.json` — if non-empty, run `git rm --cached .claude/settings.local.json`
- Run `git ls-files ".build/"` — if non-empty, run `git rm -r --cached ".build/"`
- Run `git ls-files INBOX.md` — if non-empty, run `git rm --cached INBOX.md`
- Run `git ls-files SCRATCH.md` — if non-empty, run `git rm --cached SCRATCH.md`

Include any staged removals in the same sync commit below — do NOT create a separate branch or PR for them.

### 6. Test infrastructure check

Check whether the project has a test runner configured. At least one of these must be true:

- `package.json` has a `"test"` script (check with `cat "PROJECT_PATH/package.json" | jq '.scripts.test // empty'`)
- `jest.config.js`, `jest.config.ts`, or `jest.config.json` exists
- `vitest.config.js` or `vitest.config.ts` exists
- `pytest.ini` exists
- `pyproject.toml` contains `[tool.pytest.ini_options]`
- `.mocharc.js`, `.mocharc.yml`, or `.mocharc.json` exists
- A `__tests__/` directory exists
- A `tests/` or `test/` directory containing at least one file exists

If none of these are found, inject the following item into `BACKLOG.md`. Find the `## Phase 0` section (or the first phase section if Phase 0 does not exist) and insert after the section header, before any existing items:

```
- [ ] Set up test infrastructure — configure test runner, write initial tests covering all functionality built prior to this setup
```

If `BACKLOG.md` does not exist, create it with this content:

```
# Backlog

## Phase 0 — Foundation

- [ ] Set up test infrastructure — configure test runner, write initial tests covering all functionality built prior to this setup
```

Do not inject if the item already exists anywhere in BACKLOG.md (check for "Set up test infrastructure" before inserting).

## .build/ migration

These files are gitignored and local-only — no commit is needed for them.

If `.build/BUILD_QUESTIONS.md` exists locally and has an "Open Questions" section
with items, and `.build/OPEN_QUESTIONS.md` does not exist:
- Read the Open Questions section from BUILD_QUESTIONS.md
- Create `.build/OPEN_QUESTIONS.md` with those items in the Unresolved section and
  an empty Resolved section
- Append `[Migrated to .build/OPEN_QUESTIONS.md]` to the Open Questions section of
  BUILD_QUESTIONS.md so the session artifact reflects where they went

If `.build/BUILD_SUMMARY.md` exists locally, `.build/BUILD_QUESTIONS.md` had no
open questions (or did not exist), and `.build/OPEN_QUESTIONS.md` does not exist:
- Extract the "Open Questions" section from BUILD_SUMMARY.md instead and create
  `.build/OPEN_QUESTIONS.md` the same way

If `.build/BUILD_SUMMARY.md` exists locally and has an "End-to-End Test Plan"
section with items, and `.build/TEST_TRACKER.md` does not exist:
- Extract that section from BUILD_SUMMARY.md
- Create `.build/TEST_TRACKER.md` with those items in the Pending section and an
  empty Completed section

## What NOT to sync

Do not touch:
- `docs/project-vision.md`
- `docs/architecture.md`
- `BACKLOG.md` (except the test infrastructure injection above — that single item may be added if absent)
- `INBOX.md` (if it already exists — working surface, project-specific)
- `SCRATCH.md` (if it already exists — working surface, project-specific)
- `docs/user-guide.md` (if it already exists — project-specific content)
- `docs/admin-guide.md` (if it already exists — project-specific content)
- Any key in settings.json other than `permissions.allow` and `hooks`

## After syncing

Check for changes with `git status`. If there are no changes, report that everything is already up to date and stop.

If changes are detected:
1. Stage the changed files
2. Commit with the message: `chore: sync from project-template`
3. Push the branch. If the branch was reset to main earlier (i.e., it already existed), use `--force-with-lease`. Otherwise use a plain push.
4. Open a PR targeting `main` with:
   - Title: `chore: sync from project-template`
   - Body listing which files changed and a note that this was generated by `/sync-template`
   Check for an existing open PR on this branch with `gh pr list --head chore/sync-template --state open` — if one exists, skip creating a new PR and report the existing URL instead.
5. Re-write `.claude/settings.json` to disk using the content built in step 5f — even if it still exists, write it again. If `git rm --cached` was run earlier, a `git pull` during this session will delete the file when the commit is applied; writing it here ensures it is present as a properly gitignored, untracked local file before handing back to the user.

Note: `.claude/settings.json` is gitignored and will not appear in `git status`. If it was the only file that changed, the status check above will show no tracked changes — report that settings.json was rebuilt locally and no commit is needed. Similarly, any files created by the `.build/` migration above are gitignored and will not appear in `git status`.
