Instructions for deploying global commands from this template repo to `~/.claude/commands/`. Run this after committing changes to any file in `global-commands/`.

## Commands to run

Run each separately from the repo root:

```
cp global-commands/sync-template.md ~/.claude/commands/sync-template.md
cp global-commands/launch-session.md ~/.claude/commands/launch-session.md
cp global-commands/marketstorm.md ~/.claude/commands/marketstorm.md
cp global-commands/namestorm.md ~/.claude/commands/namestorm.md
```

## Notes

- `setup-allowlist.md` is archived — do not deploy it
- `project-setup.md` is archived — do not deploy it (entries moved to `settings-template/project.json`)
- `deploy-globals.md` itself is not a global command — do not copy it
- After deploying, the updated commands are immediately available in any project
