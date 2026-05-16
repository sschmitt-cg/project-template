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

- **sync-template** — aligns the current project with this template: pulls latest `CLAUDE.md` and command files (`next-step`, `auto-build`, `test-companion`), rebuilds `.claude/settings.json` from the template, creates one-time stub files if absent (`INBOX.md`, `SCRATCH.md`, `docs/user-guide.md`, `docs/admin-guide.md`), updates `.gitignore`, injects a test-infrastructure backlog item if no test runner is configured, and prompts to register the project in the Scottsidian vault if no matching entry exists. Opens a PR targeting `dev`.
- **launch-session** — interactive new-project bootstrap: runs a discovery conversation, generates `docs/project-vision.md`, `docs/architecture.md`, and `BACKLOG.md`, creates the GitHub repo, clones it locally with both `main` and `dev` branches, scaffolds standard files, copies in `CLAUDE.md`, and registers the project in the Scottsidian vault. Hand-off is to `/sync-template` for command and settings installation.
- **deploy-globals** — copies the contents of this repo's `global-commands/` to `~/.claude/commands/`.
- **marketstorm** — multi-agent competitive market analysis: discovers competitors, mines weaknesses, evaluates demand, and surfaces adjacent opportunities.
- **namestorm** — multi-agent naming pipeline: generates candidates, screens availability, runs pro/con analysis, ranks and summarizes.

## How to use

### Starting a new project

**Recommended:** Run `/launch-session`. It runs a discovery conversation, generates `docs/project-vision.md`, `docs/architecture.md`, and `BACKLOG.md`, creates the GitHub repo, clones it locally with both `main` and `dev` branches, scaffolds standard files, and registers the project in the Scottsidian vault.

**Manual path:** If you'd rather scaffold by hand, copy `BACKLOG.md`, `docs/project-vision.md`, and `docs/architecture.md` into your project and fill them in, then run `/sync-template` — it creates `CLAUDE.md`, `INBOX.md`, `SCRATCH.md`, the command files, `settings.json`, and prompts to register the project in the Scottsidian vault. You'll need to create the `dev` branch yourself.

### Keeping projects in sync

Run `/sync-template` in any project to pull the latest `CLAUDE.md`, command files (`next-step`, `auto-build`, `test-companion`), and rebuild `settings.json`.

### Build tracking files

`/auto-build` and `/next-step` maintain a `.build/` directory (gitignored) in each project with two persistent tracking files:

- **`OPEN_QUESTIONS.md`** — questions deferred during builds, with the default used and a revisit condition. Survives across builds; resolved via `/test-companion`.
- **`TEST_TRACKER.md`** — manual test plan generated after each build, grouped by app area and optimized for efficient execution. Survives across builds; worked through via `/test-companion`.

Run `/test-companion` after a build to walk through pending tests and answer open questions interactively.

### Updating shared infrastructure

1. Make changes on a feature branch and open a PR targeting `dev`.
2. Once merged into `dev`, manually merge `dev → main` (this step is human-driven; never automated).
3. `/sync-template` pulls from `main` (GitHub's default ref), so downstream projects only see changes after the `dev → main` merge — not after `dev`.
4. Run `/sync-template` in each downstream project to pick up the new template files.
5. Run `/deploy-globals` from this repo to update `~/.claude/commands/` with any changes under `global-commands/`.

### Keeping this README current

When adding or removing commands, changing the file structure, or updating how the template is used, update this file as part of the same PR. The `/goal` condition that `/next-step` and `/auto-build` invoke includes an explicit step to keep `README.md` in sync.
