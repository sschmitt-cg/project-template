# project-template

The authoritative source for shared Claude Code infrastructure across all projects.

## What's in here

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Universal behavior rules — synced into every project |
| `.claude/commands/next-step.md` | The `/next-step` orchestration command |
| `.claude/settings-template/` | Template files used to rebuild `settings.json` in each project |
| `global-commands/` | Commands deployed to `~/.claude/commands/` for global use |
| `docs/project-vision.md` | Stub template for per-project product vision |
| `docs/architecture.md` | Stub template for per-project architecture notes |
| `INBOX.md` | Stub template for the per-project idea inbox |
| `SCRATCH.md` | Stub template for the per-project working surface |
| `BACKLOG.md` | Stub template for the per-project feature backlog |

## Global commands

`global-commands/` contains commands deployed to `~/.claude/commands/` via `/deploy-globals`:

- **sync-template** — pulls latest shared files from this repo into the current project
- **launch-session** — session startup routine
- **deploy-globals** — deploys `global-commands/` to `~/.claude/commands/`

## How to use

### Starting a new project

1. Copy the project-specific stubs into your project and fill them in: `BACKLOG.md`, `docs/project-vision.md`, `docs/architecture.md`
2. Run `/sync-template` — it creates `CLAUDE.md`, `INBOX.md`, `SCRATCH.md`, `next-step.md`, and `settings.json` automatically

### Keeping projects in sync

Run `/sync-template` in any project to pull the latest `CLAUDE.md` and `next-step.md` and rebuild `settings.json`.

### Updating shared infrastructure

Make changes on a feature branch and merge to `main`. Run `/sync-template` in downstream projects to pick them up. Run `/deploy-globals` to update the global commands in `~/.claude/commands/`.
