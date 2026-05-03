One-time initialization for a project created from the sschmitt-cg/project-template. Run once after creating a new repo from the template.

Do not run this in the sschmitt-cg/project-template repo itself.

## Steps

### 1. Get the project path

Run `pwd` to get the absolute path of the current project root. This is the value you will use as `<project-path>` below.

### 2. Add project-path-specific allow entries

Read `.claude/settings.json`. Add the following entries to `permissions.allow` if not already present, substituting the actual project path for `<project-path>`:

```json
"Bash(mkdir -p \"<project-path>/**\")",
"Bash(find \"<project-path>\" *)",
"Bash(ls \"<project-path>/**\")"
```

Preserve all existing entries. Write the updated file.

### 3. Prompt for additional project-specific entries

Ask the user: "Are there any other project-specific commands you want pre-approved? For example, a package manager not in the standard list, or paths outside the project root that Claude will need access to."

If they provide entries, add them to `permissions.allow` or `additionalDirectories` as appropriate.

### 4. Confirm

Show the final contents of `.claude/settings.json` and confirm what was added.
