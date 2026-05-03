Pull the latest shared files from the project template repo (sschmitt-cg/project-template) into the current project. Syncs universal rules and the next-step command without touching project-specific content.

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

### 3. .claude/settings.json — merge, do not overwrite

Fetch the template's settings.json:
```
gh api repos/sschmitt-cg/project-template/contents/.claude/settings.json --jq '.content' | base64 -d
```

Read the current project's `.claude/settings.json`. Merge the `permissions.allow` arrays:
- Add any entries from the template that are not already present
- Preserve all existing project-specific entries (path-based `mkdir`, `ls`, `find`, etc.)
- Write the merged result back to `.claude/settings.json`

If `.claude/settings.json` does not exist in the current project, create it with only the template's contents.

## What NOT to sync

Do not touch:
- `docs/project-vision.md`
- `docs/architecture.md`
- `BACKLOG.md`
- `.claude/settings.json` keys other than `permissions.allow`
- `.gitignore`

## After syncing

Show a diff of what changed (`git diff` on updated files). If there are no changes, say so. Do not commit — leave that to the user.
