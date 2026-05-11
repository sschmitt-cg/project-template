# project-template

The authoritative source for shared Claude Code infrastructure across all projects.

## What's in here

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Universal behavior rules — synced into every project |
| `.claude/commands/next-step.md` | The `/next-step` orchestration command |
| `.claude/commands/auto-build.md` | The `/auto-build` autonomous multi-phase build command |
| `.claude/commands/test-companion.md` | The `/test-companion` interactive testing and Q&A command |
| `.claude/settings-template/` | Template files used to rebuild `settings.json` in each project |
| `global-commands/` | Commands deployed to `~/.claude/commands/` for global use |
| `docs/project-vision.md` | Stub template for per-project product vision |
| `docs/architecture.md` | Stub template for per-project architecture notes |
| `docs/user-guide.md` | Stub template for per-project user-facing docs |
| `docs/admin-guide.md` | Stub template for per-project admin/deployment docs |
| `INBOX.md` | Stub template for the per-project idea inbox |
| `SCRATCH.md` | Stub template for the per-project working surface |
| `BACKLOG.md` | Stub template for the per-project feature backlog |

## Global commands

`global-commands/` contains commands deployed to `~/.claude/commands/` via `/deploy-globals`:

- **sync-template** — pulls latest shared files from this repo into the current project (CLAUDE.md, next-step, auto-build, test-companion, settings.json)
- **launch-session** — session startup routine
- **deploy-globals** — deploys `global-commands/` to `~/.claude/commands/`
- **marketstorm** — multi-agent competitive market analysis: discovers competitors, mines weaknesses, evaluates demand, and surfaces adjacent opportunities
- **namestorm** — multi-agent naming pipeline: generates candidates, screens availability, runs pro/con analysis, ranks and summarizes

## How to use

### Starting a new project

1. Copy the project-specific stubs into your project and fill them in: `BACKLOG.md`, `docs/project-vision.md`, `docs/architecture.md`
2. Run `/sync-template` — it creates `CLAUDE.md`, `INBOX.md`, `SCRATCH.md`, `next-step.md`, `auto-build.md`, `test-companion.md`, and `settings.json` automatically

### Keeping projects in sync

Run `/sync-template` in any project to pull the latest `CLAUDE.md`, command files (`next-step`, `auto-build`, `test-companion`), and rebuild `settings.json`.

### Build tracking files

`/auto-build` and `/next-step` maintain a `.build/` directory (gitignored) in each project with two persistent tracking files:

- **`OPEN_QUESTIONS.md`** — questions deferred during builds, with the default used and a revisit condition. Survives across builds; resolved via `/test-companion`.
- **`TEST_TRACKER.md`** — manual test plan generated after each build, grouped by app area and optimized for efficient execution. Survives across builds; worked through via `/test-companion`.

Run `/test-companion` after a build to walk through pending tests and answer open questions interactively.

### Updating shared infrastructure

Make changes on a feature branch and merge to `main`. Run `/sync-template` in downstream projects to pick them up. Run `/deploy-globals` to update the global commands in `~/.claude/commands/`.

### Keeping this README current

When adding or removing commands, changing the file structure, or updating how the template is used, update this file as part of the same PR. Both `/next-step` and `/auto-build` instruct their implementation agents to keep `README.md` in sync automatically.
