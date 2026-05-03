> **Archived.** This command is superseded by the `settings.json` in the project template and the `/project-setup` global command. Kept here for reference.

---

Create or overwrite `.claude/settings.json` in the current project root with the standard allowlist below. If the file already exists, merge the `allow` arrays (deduplicate) and preserve any existing keys. Do not touch `additionalDirectories` — that stays project-specific.

Write this exact `permissions.allow` list (plus any already present entries):

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Bash(git pull *)",
      "Bash(git fetch *)",
      "Bash(git checkout *)",
      "Bash(git log *)",
      "Bash(git diff *)",
      "Bash(git remote *)",
      "Bash(gh auth *)",
      "Bash(gh repo *)",
      "Bash(gh pr *)",
      "Bash(gh issue *)",
      "Bash(gh run *)",
      "Bash(npm --version)",
      "Bash(npm install *)",
      "Bash(npm run *)",
      "Bash(npm test *)",
      "Bash(npx tsc *)",
      "Bash(npx skills *)",
      "Bash(pnpm --version)",
      "Bash(pnpm install *)",
      "Bash(pnpm create *)",
      "Bash(pnpm approve-builds *)",
      "Bash(pnpm typecheck *)",
      "Bash(pnpm lint *)",
      "Bash(claude mcp *)",
      "Bash(git log * | head *)",
      "Bash(git branch * | grep *)",
      "Bash(npm run * 2>&1 | head *)",
      "Read(//usr/local/bin/**)",
      "mcp__supabase__get_project_url",
      "mcp__supabase__get_publishable_keys"
    ]
  }
}
```

After writing the file, confirm what was written and then explain the two ways to extend the allowlist for this specific project:

**1. Extra `allow` entries** — add patterns to the `permissions.allow` array for any tool calls that should be pre-approved in this project. Examples:
- A specific `mkdir` path: `"Bash(mkdir -p \"/path/to/dir\")"`
- A `cd`-prefixed command: `"Bash(cd \"/path/to/project*\" && npm run *)"`
- A destructive one-off: `"Bash(rm \"/path/to/file\")"`

**2. `additionalDirectories`** — list any directories outside the project root that Claude needs Read/Write access to. Example:
```json
"additionalDirectories": [
  "/path/to/some/other/directory"
]
```

Both keys live under `"permissions"` in `.claude/settings.json`. This file is project-local and typically git-ignored or committed depending on team preference.
