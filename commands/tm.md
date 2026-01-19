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
│   ├─ PR merged → Launch cleanup-specialist (subagent)
│   ├─ PR open   → Invoke Ralph for review loop
│   └─ No PR     → Invoke Ralph for full cycle (implement → PR → review)
│
└─ NO → START mode
    ├─ Args given → Setup worktree, then invoke Ralph for full cycle
    └─ No args    → Report ready tasks (no subagent needed)
```

**CRITICAL: Never auto-start new tasks.** Conversation context can continue existing work (review, cleanup), but starting a NEW task requires explicit `$ARGUMENTS`.

### Check PR State (Quick - No Heavy Processing)

```bash
# Quick state check only - subagent does the real work
gh pr view --json number,state,mergedAt 2>/dev/null
```

| Condition | Action |
|-----------|--------|
| `mergedAt` is set | Launch `cleanup-specialist` (subagent) |
| PR exists, open | Invoke Ralph for review loop |
| No PR | Invoke Ralph for full cycle |

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

**Step 3: Invoke Ralph for full cycle (implement → PR → review)**

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: """
Complete <tag>.<task-id>: <task-title>

Working directory: <worktree-path>

## Requirements
<task-description-and-subtasks>

## Phase 1: Implementation (TDD)
1. Run tests for affected code
2. If tests fail, fix the code
3. If coverage low, add tests
4. Commit incrementally

## Phase 2: Create PR (when local tests pass)
1. Push branch: `git push -u origin <branch>`
2. Create PR: `gh pr create --title '<title>' --body '<TM reference + summary>'`
3. Note the PR number

## Phase 3: Review Loop (after PR created)
1. Wait 60s for CI to start
2. Check CI: `gh pr checks <number>`
3. Check feedback: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
4. If CI fails or feedback exists:
   - Fix issues
   - Commit and push
   - Go to step 1
5. If CI passes and no unaddressed feedback, done

## Completion Criteria
- PR created and pushed
- CI passing
- No unaddressed review feedback

Output <promise>PR_READY</promise> when done.
--max-iterations 50 --completion-promise PR_READY
"""
)
```

After Ralph completes, report PR URL and status.

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

### Mode: REVIEW → Invoke Ralph for review loop

PR exists and is open. Use Ralph to iterate on CI and feedback:

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: """
Review PR #<number> for <tag>.<task-id>

Working directory: <worktree-path>

## Review Loop
1. Check CI: `gh pr checks <number>`
2. Check inline feedback: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
3. Check conversation: `gh pr view <number> --comments`
4. If CI fails or unaddressed feedback:
   - Fix issues (check TM before creating new tasks for suggestions)
   - Commit and push
   - Wait 60s for CI
   - Go to step 1
5. If CI passes and no unaddressed feedback, done

## Completion Criteria
- CI passing
- No unaddressed review feedback

Output <promise>PR_READY</promise> when done.
--max-iterations 30 --completion-promise PR_READY
"""
)
```

After Ralph completes, report PR status.

---

### Mode: CREATE_PR or IMPLEMENT → Invoke Ralph for full cycle

Whether commits exist (needs PR) or not (needs implementation), use Ralph for the full cycle.

```bash
# Get task details for the prompt
task-master tags use "<tag>" && task-master show <task-id> --json
```

Then invoke Ralph (same prompt as START mode Step 3):

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: """
Complete <tag>.<task-id>: <task-title>

Working directory: <worktree-path>

## Requirements
<task-description-and-subtasks>

## Phase 1: Implementation (TDD) - skip if commits already exist
1. Run tests for affected code
2. If tests fail, fix the code
3. If coverage low, add tests
4. Commit incrementally

## Phase 2: Create PR (when local tests pass) - skip if PR exists
1. Push branch: `git push -u origin <branch>`
2. Create PR: `gh pr create --title '<title>' --body '<TM reference + summary>'`
3. Note the PR number

## Phase 3: Review Loop (after PR created)
1. Wait 60s for CI to start
2. Check CI: `gh pr checks <number>`
3. Check feedback: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
4. If CI fails or feedback exists:
   - Fix issues
   - Commit and push
   - Go to step 1
5. If CI passes and no unaddressed feedback, done

## Completion Criteria
- PR created and pushed
- CI passing
- No unaddressed review feedback

Output <promise>PR_READY</promise> when done.
--max-iterations 50 --completion-promise PR_READY
"""
)
```

After Ralph completes, report PR URL and status.

---

## Orchestrator Flow

```
/tm → detect context → route:
  ├─ No PR (implement/create) → invoke Ralph (full cycle)
  ├─ PR open (review)         → invoke Ralph (review loop)
  ├─ PR merged (cleanup)      → launch cleanup-specialist (subagent)
  └─ No context               → list ready tasks
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
