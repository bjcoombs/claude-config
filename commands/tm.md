---
description: Task Master - unified command for start, review, and close
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master Orchestrator

**$ARGUMENTS**

> Thin orchestrator. Sonnet subagents delegate complex code to opus. Cost scales with actual complexity.

---

## Phase 0: Detect Context

**Check in order:**
1. Current directory - pwd matches TM worktree pattern?
2. Conversation context - Recent "📍 Current Work" footer with worktree path?
3. Arguments - Explicit tag/task-id provided?

### Detect Worktree Pattern

```bash
pwd
git branch --show-current 2>/dev/null || echo "NOT_GIT"
```

**If conversation shows active work but pwd is NOT that worktree:**
```bash
cd ~/dev/github.com/<org>/<repo>/worktree/<tag>/<task-id>--<slug>
```

**Decision tree:**

```
Is current directory a TM worktree (or cd'd to one via conversation context)?
(Pattern: worktree/<tag>/<task-id>--<slug>)
│
├─ YES → Check PR state (quick gh commands only)
│   ├─ PR merged      → Launch cleanup-specialist
│   ├─ PR open        → Launch review-specialist
│   ├─ Commits, no PR → Launch pr-creator
│   └─ No commits     → Launch implementation-specialist
│
└─ NO → START mode
    ├─ Args given    → Launch start-specialist
    └─ No args       → Report ready tasks (no subagent needed)
```

**CRITICAL: Never auto-start new tasks.** Conversation context can continue existing work (review, cleanup), but starting a NEW task requires explicit `$ARGUMENTS`.

### Check PR State (Quick - No Heavy Processing)

```bash
# Quick state check only - subagent does the real work
gh pr view --json number,state,mergedAt 2>/dev/null
```

| Condition | Subagent to Launch |
|-----------|-------------------|
| `mergedAt` is set | `cleanup-specialist` |
| PR exists, open | `review-specialist` |
| No PR, has commits | `pr-creator` |
| No PR, no commits | `implementation-specialist` |

---

## START MODE (Not in worktree)

### No Arguments → Report Only (No Subagent)

```bash
task-master list --all-tags --ready --json
```

Report and **terminate**:
```
## Ready Tasks

| Tag | Task | Title | Points |
|-----|------|-------|--------|
| <tag> | <id> | <title> | <points> |

Run `/tm <tag> <task-id>` to start a task.
```

**Do NOT launch subagent.** User chooses, then re-runs `/tm` with args.

### With Arguments → Launch start-specialist

Extract tag and task-id from arguments, then:

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Start Task <task-id>

Tag: <tag>
Repo: ~/dev/github.com/<org>/<repo>

You're an orchestrator. Do setup yourself, spawn opus for implementation.

## Setup (do yourself)
1. From <repo>-main: checkout develop, pull, create branch and worktree
2. `task-master set-status --id="<task-id>" --status=in-progress`

## Implementation (spawn opus)
```
Task(model: "opus", prompt: "Implement task <task-id>: <description>. Working dir: <path>. Commit incrementally, don't push.")
```

## After implementation
1. Push and create PR with `gh pr create`
2. Monitor CI, fix issues (simple yourself, complex spawn opus)
3. Report: PR ready, blocked, or task too large
"""
)
```

---

## WORKTREE MODES

Derive context from path:
```bash
TAG=$(basename $(dirname "$(pwd)"))
TASK_DIR=$(basename "$(pwd)")
TASK_ID="${TASK_DIR%%--*}"
```

---

### Mode: CLEANUP → Launch cleanup-specialist

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Cleanup <tag>.<task-id>

PR merged. Mark task done in TM, remove worktree from <repo>-main, delete branch. Report complete.
"""
)
```

---

### Mode: REVIEW → Launch review-specialist

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Review PR #<number>

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

You're an orchestrator. Fix simple things yourself, spawn opus for complex code changes.

## What's different from normal

1. **TodoWrite tracks issues** - Create one item per issue (include comment ID). This prevents loops. Exit when all done + no new feedback after push.

2. **Spawn opus for complex fixes** - Don't attempt complex code changes yourself:
   ```
   Task(model: "opus", prompt: "Fix <specific issue> in <file>. Commit only, don't push. Report FIXED or BLOCKED.")
   ```

3. **Check TM before creating tasks** - Future work suggestions may already be tracked:
   ```bash
   task-master tags use "<tag>" && task-master list --json
   ```

## Flow

Gather state → TodoWrite issues → Fix/delegate/defer each → Push → Check for NEW feedback only → Repeat or exit

Report: ready, waiting, or blocked.
"""
)
```

---

### Mode: CREATE_PR → Launch pr-creator

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Create PR for <tag>.<task-id>

Working directory: <worktree-path>

Commits exist, no PR yet. Push, create PR (include TM reference in body), check CI. Report PR number and status.
"""
)
```

---

### Mode: IMPLEMENT → Launch implementation-specialist

Worktree exists but no commits. Same pattern as start-specialist:

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Continue Task <task-id>

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

Worktree exists, no commits yet. Spawn opus to implement, then create PR and monitor CI.

Same flow as start-specialist: spawn opus for code, do PR/CI yourself, report ready/blocked/too-large.
"""
)
```

---

## Orchestrator Flow

```
/tm → detect context → launch subagent → report result → terminate
```

---

## Task Status Semantics

| Status      | Meaning                                   |
|-------------|-------------------------------------------|
| pending     | Not started                               |
| in-progress | Currently being worked on                 |
| review      | Implementation complete, PR pending merge |
| done        | PR merged, work verified and complete     |
| blocked     | Cannot proceed due to external dependency |
| deferred    | Postponed for later                       |
| cancelled   | Will not be done                          |

## Task Status Lifecycle

```
/tm starts (START)  → Task: pending → in-progress
Implementation      → Subtasks: pending → review (code complete)
All subtasks review → Parent: STAYS in-progress
PR created/polished → Parent: STAYS in-progress
Human merges PR     → (no command runs)
/tm after merge     → Parent: in-progress → done
```

---

## Notes

- Human merges PRs, never auto-merge
- Subagents can bail if work is too large
- Each `/tm` invocation starts fresh
