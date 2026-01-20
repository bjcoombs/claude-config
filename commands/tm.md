---
description: Task Master - unified command for start, review, and close
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master Orchestrator

**$ARGUMENTS**

> Thin orchestrator. Uses Ralph Loop for iteration when available, falls back to subagents.

---

## Phase 0: Detect Ralph Plugin

```bash
ls ~/.claude/plugins/cache/claude-plugins-official/ralph-loop/*/commands/ralph-loop.md 2>/dev/null && echo "RALPH_AVAILABLE" || echo "RALPH_NOT_AVAILABLE"
```

Set `$RALPH_AVAILABLE` based on result. If Ralph not available, warn once:
> ⚠️ Ralph Loop plugin not installed. Using subagents (may burn context on long test runs). Install: `/plugin install ralph-loop`

---

## Phase 1: Detect Context

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
│   ├─ PR merged → Launch cleanup-specialist (subagent, always)
│   ├─ PR open   → Ralph: review loop / Fallback: review-specialist
│   └─ No PR     → Ralph: full cycle / Fallback: implement-specialist
│
└─ NO → START mode
    ├─ Args given → Setup worktree, then Ralph: full cycle / Fallback: start-specialist
    └─ No args    → Report ready tasks (no subagent needed)
```

**CRITICAL: Never auto-start new tasks.** Conversation context can continue existing work (review, cleanup), but starting a NEW task requires explicit `$ARGUMENTS`.

### Check PR State (Quick - No Heavy Processing)

```bash
# Quick state check only - subagent does the real work
gh pr view --json number,state,mergedAt 2>/dev/null
```

| Condition | Action (Ralph) | Action (Fallback) |
|-----------|----------------|-------------------|
| `mergedAt` is set | cleanup-specialist | cleanup-specialist |
| PR exists, open | Ralph review loop | review-specialist |
| No PR | Ralph full cycle | implement-specialist |

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

### With Arguments → Setup then Implement

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

**Step 2: Get task details**

```bash
task-master show <task-id> --json
```

**Step 3: Implement (Ralph or Fallback)**

See [Full Cycle: Ralph](#full-cycle-ralph) or [Full Cycle: Fallback](#full-cycle-fallback) below.

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

After subagent completes, cd to repo root (worktree was deleted, cwd is invalid):
```bash
cd ~/dev/github.com/<org>/<repo>
```

---

### Mode: REVIEW (PR open)

**If `$RALPH_AVAILABLE`:** Invoke Ralph for review loop.

**IMPORTANT**: Keep prompt SIMPLE - no multi-line args with special characters.

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: "Review PR #<number> in <worktree-path>. Check CI, inline comments, conversation. Fix issues, push, wait 60s, repeat. Output <promise>PR_READY</promise> when CI passes and no unaddressed feedback. --max-iterations 30 --completion-promise PR_READY"
)
```

**If Ralph NOT available:** Launch review-specialist subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Review PR #<number> for <tag>.<task-id>

Working directory: <worktree-path>

Check CI and feedback. Fix issues, push, wait for CI, repeat until ready.
Spawn opus for complex code changes.

Report: ready, waiting, or blocked.
"""
)
```

---

### Mode: IMPLEMENT or CREATE_PR (no PR exists)

**If `$RALPH_AVAILABLE`:** Invoke Ralph for full cycle.

**IMPORTANT**: Keep the Ralph prompt SIMPLE to avoid bash parsing issues. Do NOT embed task descriptions inline - they contain backticks and special characters that break parsing.

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: "Complete <tag>.<task-id> in <worktree-path>. Run task-master show <task-id> for requirements. TDD: test, fix, commit. Then push, create PR, monitor CI, address feedback. Output <promise>PR_READY</promise> when CI passes and no unaddressed feedback. --max-iterations 50 --completion-promise PR_READY"
)
```

**If Ralph NOT available:** Launch implement-specialist subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Implement <tag>.<task-id>: <task-title>

Working directory: <worktree-path>

You're an orchestrator. Spawn opus for complex code changes.

## Requirements
<task-description-and-subtasks>

## Flow
1. Implement using TDD (run tests, fix failures, commit)
2. Push and create PR
3. Monitor CI, fix issues
4. Report: PR ready, waiting, or blocked

⚠️ Long test runs may burn context. Consider installing Ralph plugin: `/plugin install ralph-loop`
"""
)
```

---

## Orchestrator Flow

```
/tm → check Ralph → detect context → route:
  │
  ├─ No PR (implement/create)
  │   ├─ Ralph available → invoke Ralph (full cycle)
  │   └─ Ralph missing   → launch implement-specialist (subagent)
  │
  ├─ PR open (review)
  │   ├─ Ralph available → invoke Ralph (review loop)
  │   └─ Ralph missing   → launch review-specialist (subagent)
  │
  ├─ PR merged (cleanup) → launch cleanup-specialist (always subagent)
  │
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
