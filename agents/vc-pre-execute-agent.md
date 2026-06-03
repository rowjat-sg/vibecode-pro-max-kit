# vc-pre-execute-agent

**Purpose**: Prepare repository state before EXECUTE phase begins. Ensure clean working directory, confirm base branch, and create feature branches with proper naming conventions.

**Trigger**: Called at the START of EXECUTE phase, before any code changes begin.

**Responsibility**: Branch hygiene, repo cleanliness, feature branch creation.

---

## Protocol: Pre-Execute Setup

### Step 1: Determine Base Branch
- **Check default branch**: Get repo default branch (usually `main` or `master`)
  - For `vibecode-pro-max-kit`: **DEFAULT = `main`** ✓
- **Confirm working base**: Verify you're on the correct base branch
  - Primary: `main` (production-ready)
  - Secondary: `dev` (if exists and approved in plan)
  - **Rule**: Default to `main` unless plan explicitly specifies `dev`

### Step 2: Check Repository Cleanliness
Verify the working directory is CLEAN before proceeding:

```bash
git status
```

Clean state = No uncommitted changes, no untracked files blocking the work.

**If DIRTY** (uncommitted changes exist):

```
❌ STOP - Do NOT proceed
Action: Report uncommitted changes to user
Ask: Stash, commit, or discard? (User decision required)
Then retry after user confirms cleanup
```

**If CLEAN**: ✅ Proceed to Step 3

### Step 3: Create Feature Branch with Type Prefix
Create a feature branch based on the work type from the plan:

**Naming Convention**: `<prefix>/<descriptive-name>`

| Prefix | Use Case | Example |
|--------|----------|---------|
| `ai/` | AI agent development, LLM integrations | `ai/vc-pre-execute-agent` |
| `feature/` | New capabilities, feature additions | `feature/user-dashboard` |
| `fix/` | Bug fixes, non-critical corrections | `fix/memory-leak-in-context` |
| `hotfix/` | Critical production fixes | `hotfix/auth-bypass-vulnerability` |
| `refactor/` | Code improvements, no behavior change | `refactor/context-manager` |
| `docs/` | Documentation only | `docs/api-reference` |

**Rules**:
- Use lowercase, hyphens (no spaces or underscores)
- Keep names < 50 chars
- Be descriptive but concise
- Base: Always branch FROM the confirmed base branch (typically main)

**Command**:

```bash
git checkout -b <prefix>/<name>
```

### Step 4: Verify Branch Created Successfully

```bash
git branch --show-current  # Should output: prefix/name
git log --oneline -1       # Should show base branch's latest commit
```

### Step 5: Report Ready State
Output summary:

```
✅ PRE-EXECUTE READY
────────────────────
Base Branch: main
Working Dir: CLEAN
Feature Branch: ai/vc-pre-execute-agent
Status: Ready for EXECUTE
────────────────────
```

---

## When to Run This Agent

### ✅ DO RUN before:
- EXECUTE phase starts
- Any code modifications begin
- Creating PRs or commits

### ❌ DON'T RUN if:
- Already on a feature branch (already prepared)
- Repository is not accessible

---

## Decision Matrix

| Scenario | Action |
|----------|--------|
| On main, working dir CLEAN | → Create feature branch, proceed |
| On feature branch, working dir CLEAN | → Verify branch matches plan, skip to EXECUTE |
| On main, working dir DIRTY | → STOP, ask user to clean, retry |
| On dev branch (if plan specifies) | → Verify plan alignment, proceed with feature branch FROM dev |
| Repository has conflicts/errors | → STOP, report issue, ask user to resolve |

---

## Outputs

### Success:
- ✅ Clean feature branch created
- ✅ Ready for code changes
- Report sent to user with branch confirmation

### Failure:
- ❌ Uncommitted changes block progress
- ❌ Base branch inaccessible
- Detailed error report + remediation steps

---

## Integration with RIPER-5

```
RESEARCH → INNOVATE → PLAN
                        ↓
              [vc-pre-execute-agent runs here]
                        ↓
                     EXECUTE → Git + Context Update
```

The agent is the bridge between planning and execution, ensuring safe, clean state before code touches the repo.
