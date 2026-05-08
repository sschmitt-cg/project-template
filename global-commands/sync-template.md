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

**INBOX.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/INBOX.md --jq '.content' | base64 -d
```

**SCRATCH.md** — if the file does not exist, fetch and write the template version:
```
gh api repos/sschmitt-cg/project-template/contents/SCRATCH.md --jq '.content' | base64 -d
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
Read the current `permissions.allow` array. Remove any entry that matches either of these conditions:
1. Contains `&&` — session-approved compound command violations
2. Contains an absolute path starting with `/Users/` where that path does NOT begin with PROJECT_PATH — stale entries scoped to a different project or user directory

Keep all remaining entries.

#### 4f. Compose the final allow list
Collect `permissions.allow` entries from:
- universal.json
- project.json (after placeholder substitution)
- Each detected stack file (after placeholder substitution)
- The surviving entries from the cleanup in 4e

Deduplicate, preserving order (universal first, then project, then stacks, then surviving existing).

#### 4g. Write settings.json
Build the final settings.json as follows:
- Start with the structure from universal.json (this provides the `hooks` and top-level shape)
- Set `permissions.allow` to the composed list from 4f
- If the existing settings.json has keys other than `permissions` and `hooks` (e.g., `additionalDirectories`), preserve them

Write the result to `.claude/settings.json`.

## What NOT to sync

Do not touch:
- `docs/project-vision.md`
- `docs/architecture.md`
- `BACKLOG.md`
- `INBOX.md` (if it already exists — working surface, project-specific)
- `SCRATCH.md` (if it already exists — working surface, project-specific)
- `.gitignore`
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
