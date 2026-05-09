Pull the latest shared files from the project template repo (sschmitt-cg/project-template) into the current project. Syncs universal rules, the next-step command, and rebuilds settings.json from the template files.

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

### 3. One-time stub files

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

### 4. .claude/settings.json — rebuild from template files

`settings.json` is gitignored and never committed. This step is the authoritative source for what goes in it.

#### 4a. Determine the project path
Run `git rev-parse --show-toplevel` and store the result as PROJECT_PATH.

#### 4b. Fetch the template files
Fetch each of the following from the template repo:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/universal.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/project.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/stacks/python.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/stacks/expo.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/stacks/supabase.json --jq '.content' | base64 -d
gh api repos/sschmitt-cg/project-template/contents/.claude/settings-template/stacks/prisma.json --jq '.content' | base64 -d
```

#### 4c. Substitute placeholders
In project.json and all stack files, replace every occurrence of `{{PROJECT_PATH}}` with the actual PROJECT_PATH value before reading any allow entries from them.

Also compute `PROJECT_PATH_ESCAPED` by replacing every space in PROJECT_PATH with `\ ` (backslash-space). For example, if PROJECT_PATH is `/Users/scott/Documents/Code Projects/myapp`, then PROJECT_PATH_ESCAPED is `/Users/scott/Documents/Code\ Projects/myapp`.

For each `Bash(...)` allow entry in the substituted project.json and stack files that contains a **quoted string beginning with PROJECT_PATH** (e.g., `Bash(cmd "PROJECT_PATH/...")`, `Bash("PROJECT_PATH/bin/tool" *)`), generate a parallel escaped-path entry:
- Remove the double-quotes that surround the quoted segment containing PROJECT_PATH
- Replace the PROJECT_PATH portion within that segment with PROJECT_PATH_ESCAPED

These parallel entries are collected alongside the originals in step 4f. Skip this for `Read(...)` entries — the Read tool receives the raw file path directly, not a shell string, so no escaping is needed.

Examples (PROJECT_PATH = `/Users/scott/Code Projects/myapp`):

| Original (quoted) | Parallel (escaped) |
|---|---|
| `Bash(ls "/Users/scott/Code Projects/myapp/**")` | `Bash(ls /Users/scott/Code\ Projects/myapp/**)` |
| `Bash(git -C "/Users/scott/Code Projects/myapp" add *)` | `Bash(git -C /Users/scott/Code\ Projects/myapp add *)` |
| `Bash(cat "/Users/scott/Code Projects/myapp/**")` | `Bash(cat /Users/scott/Code\ Projects/myapp/**)` |
| `Bash("/Users/scott/Code Projects/myapp/venv/bin/python" *)` | `Bash(/Users/scott/Code\ Projects/myapp/venv/bin/python *)` |
| `Bash(cd "/Users/scott/Code Projects/myapp")` | `Bash(cd /Users/scott/Code\ Projects/myapp)` |

#### 4d. Detect applicable stacks
Check each stack's `detection` conditions against the project directory:

**python** — apply if any of these are true:
- `venv/` directory exists
- `.venv/` directory exists
- `pyproject.toml` exists
- `requirements.txt` exists
- `setup.py` exists

**expo** — apply if `package.json` contains `"expo"` as a dependency key

**supabase** — apply if any of these are true:
- `supabase/` directory exists
- `package.json` contains `@supabase/supabase-js` or `@supabase/ssr`
- `.env` or `.env.local` contains a `SUPABASE_` variable

**prisma** — apply if `package.json` contains `"prisma"` or `"@prisma/client"`

#### 4e. Clean up the existing settings.json (if it exists)
Read files in the target project using `cat "PROJECT_PATH/..."` via Bash — not the Read tool. The Read tool prompts for permission on files outside the current project directory; `cat` is auto-allowed.

Read the current `permissions.allow` array. Remove any entry that matches either of these conditions:
1. Contains `&&` — session-approved compound command violations
2. Contains an absolute path starting with `/Users/` (quoted or backslash-escaped) where that path does NOT begin with PROJECT_PATH or PROJECT_PATH_ESCAPED — stale entries scoped to a different project or user directory

Keep all remaining entries.

#### 4f. Compose the final allow list
Collect `permissions.allow` entries from:
- universal.json
- project.json (after placeholder substitution and escaped-path variants from step 4c)
- Each detected stack file (after placeholder substitution and escaped-path variants from step 4c)
- The surviving entries from the cleanup in 4e

For each source above, interleave the escaped-path variant immediately after its original entry (e.g., the escaped `git -C PROJECT_PATH_ESCAPED add *` entry follows the quoted `git -C "PROJECT_PATH" add *` entry).

Deduplicate, preserving order (universal first, then project, then stacks, then surviving existing).

#### 4g. Write settings.json
Build the final settings.json as follows:
- Start with the structure from universal.json (this provides the `hooks` and top-level shape)
- Set `permissions.allow` to the composed list from 4f
- If the existing settings.json has keys other than `permissions` and `hooks` (e.g., `additionalDirectories`), preserve them

Write the result to `.claude/settings.json`.

## .gitignore — ensure settings files are excluded

`settings.json` and `settings.local.json` are never committed. Verify `.gitignore` has entries for both.

If `.gitignore` does not exist, create it.
If `.gitignore` does not contain `.claude/settings.json`, append it.
If `.gitignore` does not contain `.claude/settings.local.json`, append it.

**Untrack if currently committed:** After updating `.gitignore`, run `git ls-files .claude/settings.json`. If the output is non-empty, the file is tracked and must be removed from the index:
- Run `git rm --cached .claude/settings.json`
- Include this staged removal in the same sync commit below — do NOT create a separate branch or PR for it

## 5. Test infrastructure check

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
1. Create and switch to a new branch named `chore/sync-template`
2. Stage the changed files
3. Commit with the message: `chore: sync from project-template`
4. Push the branch
5. Open a PR targeting `main` with:
   - Title: `chore: sync from project-template`
   - Body listing which files changed and a note that this was generated by `/sync-template`
6. Re-write `.claude/settings.json` to disk using the content built in step 4g — even if it still exists, write it again. If `git rm --cached` was run earlier, a `git pull` during this session will delete the file when the commit is applied; writing it here ensures it is present as a properly gitignored, untracked local file before handing back to the user.
