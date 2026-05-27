# Flowser

Flowser monorepo -- Next.js web app + Docker containers for AI agent-powered browser automation.

**Tech Stack**: Next.js 15 (App Router), tRPC, Prisma + PostgreSQL, Docker, OpenClaw, CloakBrowser, Tailwind CSS v4, shadcn/ui

---

## Development System

This monorepo uses **RIPER-5** -- a spec-driven development protocol shared across Claude and Codex. Every request is routed through a skill-discovery step, then to the right agent or workflow.

---

### How It Works

```
Your request
  -> Step 0: Skill Discovery (match keywords -> surface relevant skills)
  -> Intent Detection (feature / bug / question / refactor / UI)
  -> Route to agent or meta-workflow
  -> Phase-locked execution with explicit transitions
```

The orchestrator session **never does the work itself** -- it routes, monitors, and manages transitions.

For large multi-phase programs, use the repo's **phase program** protocol in
`process/development-protocols/phase-programs.md`:

- create one umbrella plan
- split the work into explicit phase plans
- advance one phase at a time
- run the required loop inside each phase:
  research -> approval -> execute -> validate -> durable report/context update

There is also a reusable **big-project kickoff prompt** in
`process/development-protocols/phase-programs.md` so future agents can start the workflow
consistently instead of improvising the structure each time. That kickoff should first recommend
the plan shape, phase sequence, and next actions before creating plan files.

---

### RIPER-5 Phases

| Mode               | What happens                                                                  | How to enter                              |
| ------------------ | ----------------------------------------------------------------------------- | ----------------------------------------- |
| **RESEARCH**       | Read-only fact gathering, codebase + web                                      | Automatic on feature/question requests    |
| **INNOVATE**       | Explore 2-3 approaches, trade-offs                                            | Say `go` after research                   |
| **PLAN**           | Write spec to `process/general-plans/active/` or `process/features/*/active/` | Say `go` after innovate                   |
| **EXECUTE**        | Implement per approved plan                                                   | Say `ENTER EXECUTE MODE`                  |
| **UPDATE PROCESS** | Capture learnings, archive plan                                               | Optional, say `ENTER UPDATE PROCESS MODE` |

**Phase rules:**

- Every transition requires explicit confirmation -- nothing auto-advances
- INNOVATE must produce a Decision Summary before PLAN begins
- EXECUTE requires a selected plan file in `process/general-plans/active/` or `process/features/*/active/` -- refuses without one
- Trivial fixes (single file, <15 lines, no schema/auth changes) skip straight to EXECUTE

---

### Quick Workflows (skip RIPER-5)

| Workflow      | When to use                                               | How                        |
| ------------- | --------------------------------------------------------- | -------------------------- |
| **Fast Mode** | You want RESEARCH+INNOVATE+PLAN in one shot, then confirm | `ENTER FAST MODE - [task]` |
| **Debugger**  | Root cause analysis before any substantial fix            | route to `vc-debugger`     |
| **Trivial fix** | Very small scoped change                                | route to `vc-execute-agent`|

---

### Agents

#### Core (RIPER-5 phases)

| Agent                     | Role                                                                          |
| ------------------------- | ----------------------------------------------------------------------------- |
| `vc-research-agent`       | Codebase + web research, read-only                                            |
| `vc-innovate-agent`       | Brainstorm approaches, no code or decisions                                   |
| `vc-plan-agent`           | Write spec to `process/general-plans/active/` or `process/features/*/active/` |
| `vc-execute-agent`        | Implement approved plan                                                       |
| `vc-fast-mode-agent`      | Compressed RESEARCH->INNOVATE->PLAN, then pause                               |
| `vc-update-process-agent` | Capture learnings, update rules, archive plans                                |

#### Specialists (invoked from EXECUTE or standalone)

| Agent                | Role                                            |
| -------------------- | ----------------------------------------------- |
| `vc-code-reviewer`   | Pre-PR adversarial review, edge case scouting   |
| `vc-code-simplifier` | Refactor for clarity without behavior change    |
| `vc-debugger`        | Evidence-first root cause analysis              |
| `vc-tester`          | Diff-aware test verification (per-package only) |
| `vc-git-manager`     | Conventional commits + push                     |
| `vc-ui-ux-designer`  | Design-aware frontend implementation            |

---

### Skills Catalog

Skills are auto-discovered in Step 0. Use the current assistant/runtime entry surface rather than relying on legacy slash-command assumptions.

#### Flowser Contract Skills

| Skill                  | Use for                                                               |
| ---------------------- | --------------------------------------------------------------------- |
| `vc-generate-plan`     | Durable implementation plans with touchpoints, blast radius, proof, and execute handoff in `process/general-plans/` or features |
| `vc-generate-context`  | Refresh `process/context/all-context.md`                              |
| `vc-audit-context`     | Context routing, grouping, discoverability, Claude/Codex wiring       |
| `vc-audit-plans`       | Active-plan reconciliation and cleanup                                |
| `vc-audit-vc`          | Agent harness, skill registry, README.md, and protocol wiring health  |
| `vc-autoresearch`      | Autonomous metric optimization (coverage, bundle size) -- post-execute |
| `vc-setup`             | Scaffold agent harness into a new project                             |
| `vc-update`            | Pull latest harness from remote kit repo                              |
| `vc-publish`           | Push harness improvements to remote kit repo                          |

#### Planning & Analysis

| Skill                    | Use for                                                  |
| ------------------------ | -------------------------------------------------------- |
| `vc-predict`             | 5-persona risk debate before risky changes               |
| `vc-scenario`            | Edge case generation across 12 dimensions                |
| `vc-sequential-thinking` | Step-by-step reasoning with revision                     |
| `vc-problem-solving`     | Cognitive unblocking when stuck                          |

#### Debug & Security

| Skill         | Use for                                 |
| ------------- | --------------------------------------- |
| `vc-debug`    | Systematic root cause analysis          |
| `vc-security` | STRIDE + OWASP audit, optional auto-fix |

#### Research & Docs

| Skill            | Use for                                     |
| ---------------- | ------------------------------------------- |
| `vc-docs-seeker` | Library/API docs via context7               |
| `vc-scout`       | Fast parallel codebase search               |
| `vc-docs`        | Generate/update project documentation       |
| `vc-repomix`     | Pack repos into references-only artifacts   |
| `vc-xia`         | Repo comparison and adaptation-prep research |

#### Frontend & Browser

| Skill                | Use for                                            |
| -------------------- | -------------------------------------------------- |
| `vc-frontend-design` | Production UI from screenshots/designs (anti-slop) |
| `vc-chrome-devtools` | Puppeteer automation, screenshots, debugging       |
| `vc-agent-browser`   | Long browser sessions, Browserbase cloud           |
| `vc-web-testing`     | Playwright E2E, Vitest, k6 load testing            |

#### Utilities

| Skill                    | Use for                                                    |
| ------------------------ | ---------------------------------------------------------- |
| `vc-context-engineering` | Token usage monitoring, context optimization               |
| `vc-mcp-management`      | MCP server tool discovery and execution                    |
| `vc-preview`             | Visualize diagrams, slides, file diffs                     |
| `vc-team`                | Parallel multi-agent sessions (requires experimental flag) |
| `vc-tech-graph`          | Publish-grade SVG/PNG technical diagrams                   |
| `vc-watzup`              | Read-only branch/worktree/active-plan summary              |
| `vc-merge-worktree`      | Merge and clean up a git worktree                          |

---

### Invocation Notes

- Use explicit RIPER mode commands for phase transitions, for example `ENTER PLAN MODE` or `ENTER EXECUTE MODE`.
- Use real reusable skills directly by name, for example `vc-generate-plan`, `vc-generate-context`, or `vc-audit-plans`.
- Do not rely on retired command aliases or deleted legacy workflow-owner skills.

---

### Key Conventions

- **Plans:** `process/general-plans/active/[feature]_PLAN_[dd-mm-yy].md` or the matching `process/features/{feature}/active/` folder
- **Context:** `process/context/all-context.md` -- authoritative project reference
- **Tests:** `process/context/tests/all-tests.md` -- testing quick-start and deeper test routing
- **Default verification:** `pnpm test:local` is the currently verified safe default; `pnpm typecheck` and `pnpm lint` are desired quality gates but currently tracked as Phase 09 blockers until repaired or intentionally split; live/provider/costful gates are manual opt-in only
- **Package manager:** `pnpm` only (monorepo -- never `npm run` from root)
- **Commits:** conventional format, no AI references, run the current verified gate before commit;
  run lint once the Phase 09 root lint blocker is repaired or intentionally scoped
- **Files:** keep TS/JS source under 200 lines; split if larger

---

### Typical Session

```
# New feature
You: "add webhook support to the API"
-> Orchestrator surfaces: vc-scenario, vc-security (keywords: API, webhook)
-> Routes to vc-research-agent
-> You say "go" -> innovate-agent
-> You say "go" -> plan-agent writes process/general-plans/active/webhook-support_PLAN_07-04-26.md
-> You say "ENTER EXECUTE MODE" -> vc-execute-agent implements
-> Specialist chain: vc-tester -> vc-code-reviewer -> vc-code-simplifier (when useful) -> vc-git-manager

# Large program
You: "build a full autonomous testing platform for the repo"
-> Orchestrator recognizes a phase program
-> Research to define the whole program
-> Plan-agent creates umbrella plan + phase plans under a feature folder
-> For each phase:
   research subagent
   -> user approves phase scope
   -> execute subagent
   -> validate subagent
   -> report/context update
-> Finished foundation work may be split from later expansion work into follow-up feature folders

# Quick bug fix
You: "fix the login redirect not working"
-> Orchestrator routes to vc-debugger for root cause analysis
-> Then vc-execute-agent implements the approved fix
-> Specialist chain: vc-tester -> vc-code-reviewer -> vc-git-manager

# Quick question
You: "how does the tRPC context work here?"
-> Orchestrator answers directly from codebase (trivial conceptual)
```
