# vc-update Reference

Detailed reference for the vc-update skill.

## Error Handling Matrix

| Error | When | Action |
|-------|------|--------|
| Network failure during clone | Step 3 | Print error, clean up temp dir, stop |
| GitHub auth failure | Step 3 | Print "check SSH keys or HTTPS token", clean up, stop |
| Repo not found (404) | Step 3 | Print "remote repo not found, check URL in SKILL.md", clean up, stop |
| Malformed vc-manifest.json | Step 4 | Print JSON parse error with line number, clean up, stop |
| Missing vc-manifest.json | Step 4 | Print "vc-manifest.json not found in remote", clean up, stop |
| .vc-version missing | Step 2 | Not an error -- treat as `0.0.0` (first update) |
| Permission denied on copy | Step 13 | Print which file, suggest `chmod`, **continue** with remaining files |
| Permission denied on delete | Step 13 | Print which file, suggest `chmod`, **continue** |
| Symlink creation fails | Step 13 | Print error, suggest checking if target exists, **continue** |

## Edge Cases

### User modified a managed file locally

vc-update **overwrites** managed files without checking for local modifications. This is by design -- managed files are owned by the harness, not the user. The dry-run shows exactly what will change, giving the user a chance to back out.

If the user has intentional local changes to a managed file:
1. Copy their changes to a separate file before running vc-update
2. Re-apply after the update
3. Or better: move customizations to `process/context/` where they belong

### Remote has fewer managedDirs than local

If a skill directory exists locally but is NOT in the remote's `managedDirs` array, vc-update **does not touch it**. Only listed directories are synced.

### First run (no .vc-version)

Treated as upgrading from `0.0.0`. Every managed file shows as "modified" or "new". The full harness gets applied.

### Already up to date

If `.vc-version` matches the remote manifest version, report "Already up to date" and exit. No diff computed.

### Empty process/_seeds/ locally

If the project has no `process/_seeds/` (pre-v3 project), the entire seeds directory is created fresh from the remote.

## Dry-Run Output Format

```
vc-update dry run: v1.0.0 -> v1.2.0

MANAGED FILES:
  [modified]  CLAUDE.md  (+15 -8)
  [modified]  .claude/agents/execute-agent.md  (+3 -1)
  [new]       .claude/hooks/lib/new-util.cjs
  [unchanged] AGENTS.md
  ... (42 more unchanged)

MANAGED DIRECTORIES:
  .claude/skills/vc-scout/
    [modified]  SKILL.md  (+5 -2)
    [added]     references/scout-patterns.md
  ... (28 more unchanged)

SEEDS (process/_seeds/):
  [modified]  context/all-context.md.seed  (+8 -1)
  [added]     context/infra-seed/all-infra.md.seed
  [unchanged] (15 more)

DELETIONS:
  (none)

SYMLINKS:
  [ok]  .agents/skills -> ../.claude/skills
  [ok]  .codex/hooks -> ../.claude/hooks

Summary: 5 modified, 2 new, 0 deletions, 0 symlink fixes, 85 unchanged
```

## Applied Changes Output Format

```
vc-update complete: v1.0.0 -> v1.2.0

Applied:
  3 managed files modified
  1 managed file added
  2 skill directories updated
  Seeds directory synced (1 modified, 1 added)
  0 files deleted
  0 symlinks fixed

Version written to .vc-version: 1.2.0
Temp directory cleaned up.
```

## vc-manifest.json Schema Reference

```json
{
  "version": "1.2.0",
  "managed": [
    "CLAUDE.md",
    "AGENTS.md",
    ".claude/agents/execute-agent.md",
    "process/development-protocols/orchestration.md",
    "process/context/planning/example-simple-prd.md"
  ],
  "managedDirs": [
    ".claude/skills/vc-scout",
    ".claude/skills/vc-update"
  ],
  "seedsDir": "process/_seeds/",
  "symlinks": {
    ".agents/skills": "../.claude/skills",
    ".codex/hooks": "../.claude/hooks"
  },
  "deletions": []
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | Semver version of this release |
| `managed` | string[] | Individual files overwritten on update |
| `managedDirs` | string[] | Directories synced entirely (rsync-style replace) |
| `seedsDir` | string | Path to seeds directory (always `process/_seeds/`) |
| `symlinks` | object | Symlink path -> target mappings to create/fix |
| `deletions` | string[] | Paths to delete (accumulated across versions) |
