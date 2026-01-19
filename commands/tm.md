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
│   └─ No commits     → Invoke Ralph for TDD implementation
│
└─ NO → START mode
    ├─ Args given    → Setup worktree, then invoke Ralph
    └─ No args       → Report ready tasks (no subagent needed)
```

**CRITICAL: Never auto-start new tasks.** Conversation context can continue existing work (review, cleanup), but starting a NEW task requires explicit `$ARGUMENTS`.

### Check PR State (Quick - No Heavy Processing)

```bash
# Quick state check only - subagent does the real work
gh pr view --json number,state,mergedAt 2>/dev/null
```

| Condition | Action |
|-----------|--------|
| `mergedAt` is set | Launch `cleanup-specialist` |
| PR exists, open | Launch `review-specialist` |
| No PR, has commits | Launch `pr-creator` |
| No PR, no commits | Invoke Ralph for TDD |

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

### With Arguments → Setup then Ralph

Extract tag and task-id from arguments.

**Step 1: Setup (quick, no subagent needed)**

```bash
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop
git branch <tag>/<task-id>--<slug>
mkdir -p ../worktree/<tag>
git worktree add ../worktree/<tag>/<task-id>--<slug> <tag>/<task-id>--<slug>
cd ../worktree/<tag>/<task-id>--<slug>
task-master tags use "<tag>" && task-master set-status --id="<task-id>" --status=in-progress
```

**Step 2: Get task details for Ralph prompt**

```bash
task-master show <task-id> --json
```

**Step 3: Invoke Ralph for TDD implementation**

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: """
Implement <tag>.<task-id>: <task-title>

Working directory: <worktree-path>

## Requirements
<task-description-and-subtasks>

## TDD Loop
1. Run tests for affected code
2. If tests fail, fix the code
3. If tests pass but coverage low, add tests
4. Commit incrementally with clear messages

## Completion Criteria
- All tests passing
- All subtasks addressed
- Code committed

Output <promise>IMPLEMENTATION_COMPLETE</promise> when done.
--max-iterations 30 --completion-promise IMPLEMENTATION_COMPLETE
"""
)
```

After Ralph completes, report: "Implementation complete. Run `/tm` to create PR."

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

PR merged. Clean up in this order:
1. cd to <repo>-main FIRST (you're about to delete current directory)
2. Mark task done: `task-master tags use "<tag>" && task-master set-status --id=<task-id> --status=done`
3. Remove worktree: `git worktree remove ../worktree/<tag>/<task-id>--<slug>`
4. Delete branch: `git branch -d <branch-name>`

Report complete.
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

### Mode: IMPLEMENT → Invoke Ralph Loop for TDD

Worktree exists but no commits. Use Ralph for iterative TDD:

```bash
# Get task details for the prompt
task-master tags use "<tag>" && task-master show <task-id> --json
```

Then invoke Ralph directly (not via subagent):

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: """
Implement <tag>.<task-id>: <task-title>

Working directory: <worktree-path>

## Requirements
<task-description-and-subtasks>

## TDD Loop
1. Run tests: <test-command>
2. If tests fail, fix the code
3. If tests pass but coverage low, add tests
4. Commit incrementally with clear messages
5. Repeat until all requirements met

## Completion Criteria
- All tests passing
- All subtasks addressed
- Code committed

Output <promise>IMPLEMENTATION_COMPLETE</promise> when done.
--max-iterations 30 --completion-promise IMPLEMENTATION_COMPLETE
"""
)
```

After Ralph completes, report: "Implementation complete. Run `/tm` to create PR."

---

## Orchestrator Flow

```
/tm → detect context → route:
  ├─ Implementation needed → invoke Ralph (TDD loop)
  ├─ PR work needed → launch subagent (review/cleanup/pr-create)
  └─ No context → list ready tasks
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
