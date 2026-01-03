---
description: Task Master - unified command for start, review, and cleanup
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master

**$ARGUMENTS**

> **Note to self**: This workflow was designed collaboratively. Context determines behavior - you figure out where you are and what needs doing. If something feels wrong, say so. Your clarity matters.

---

## Phase 0: Detect Context

```bash
pwd
git branch --show-current 2>/dev/null || echo "NOT_GIT"
```

**Decision tree:**

```
Is current directory a TM worktree?
(Pattern: worktree/<tag>/<task-id>--<slug>)
│
├─ YES → Check PR state
│   ├─ PR merged      → CLEANUP
│   ├─ PR open        → REVIEW
│   ├─ Commits, no PR → CREATE_PR
│   └─ No commits     → IMPLEMENT (continue)
│
└─ NO → START mode
    ├─ Args given    → Start that task
    └─ No args       → Start next ready task
```

### Detect Worktree

```bash
# Check if in TM worktree pattern
CURRENT=$(pwd)
if [[ "$CURRENT" =~ worktree/([^/]+)/([0-9.]+)-- ]]; then
  TAG="${BASH_REMATCH[1]}"
  TASK_ID="${BASH_REMATCH[2]}"
  echo "TM Context: tag=$TAG task=$TASK_ID"
  # → Go to WORKTREE MODES
else
  echo "Not in TM worktree"
  # → Go to START MODE
fi
```

---

## START MODE (Not in worktree)

### Step 1: Determine Task

**If `$ARGUMENTS` has two parts** (e.g., "458-service 1.2"):
```bash
TAG="<first-part>"
TASK_ID="<second-part>"
task-master tags use "$TAG"
task-master show "$TASK_ID" --json
```

**If `$ARGUMENTS` has one part** (e.g., "1.2"):
```bash
TASK_ID="<argument>"
# Warn: "Using current tag. Run /tm <tag> <task-id> to be explicit."
task-master show "$TASK_ID" --json
```

**If `$ARGUMENTS` is empty**:
```bash
task-master next --format json
# Use returned task, warn about implicit selection
```

### Step 2: Check Complexity Analysis

If task lacks `complexity` field:
```bash
task-master analyze-complexity --id="$TASK_ID" --research
task-master show "$TASK_ID" --json  # re-fetch
```

### Step 3: Setup Worktree

```bash
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop
git branch "$TAG/$TASK_ID--<slug>"
mkdir -p ../worktree/"$TAG"
git worktree add ../worktree/"$TAG"/"$TASK_ID--<slug>" "$TAG/$TASK_ID--<slug>"
cd ../worktree/"$TAG"/"$TASK_ID--<slug>"

task-master set-status --id="$TASK_ID" --status=in-progress
```

### Step 4: Implementation

**If task has subtasks** → subagent per subtask:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Implement subtask <subtask-id> for task <task-id>.

Working directory: <worktree-path>
Subtask: <subtask-title>
Description: <subtask-description>

Implement, test, commit.

FORBIDDEN: `task-master set-status --id=<parent> --status=done` - NEVER run this.

If too large, STOP and report what's done/remaining.
"""
)
```

After each subtask: `task-master set-status --id=<task-id>.<subtask-id> --status=done`

**If no subtasks** → single implementation subagent.

### Step 5: Create PR

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Create PR for task <task-id> (<tag>).

Working directory: <worktree-path>
Completed: <subtask summaries>

Push branch. Create PR with `gh pr create`. Include TM reference in body.
Output PR_NUMBER and PR_URL.
"""
)
```

### Step 6: Quality Loop

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Quality loop for PR #<pr-number>.

Working directory: <worktree-path>

Loop until CI passes and no actionable feedback (max 10 iterations):
1. Check CI, inline comments, conversation
2. Fix issues, commit, push
3. Report each iteration

Stop when green. Ask for help if stuck.
"""
)
```

### Step 7: Report

```
## PR Ready for Review

**PR**: #<number> - <title>
**Task**: <tag>.<task-id>

CI passing. Awaiting human review and merge.
Run `/tm` again after merge to cleanup.
```

---

## WORKTREE MODES (Already in TM worktree)

Derive context from path:
```bash
TAG=$(basename $(dirname "$(pwd)"))
TASK_DIR=$(basename "$(pwd)")
TASK_ID="${TASK_DIR%%--*}"
```

### Check PR State

```bash
# Check if PR exists and its state
gh pr view --json number,state,mergedAt,mergeable,mergeStateStatus,reviewDecision,isDraft 2>/dev/null
```

**Route by state:**

| Condition | Mode |
|-----------|------|
| `mergedAt` is set | → CLEANUP |
| PR exists, open | → REVIEW |
| No PR, has commits | → CREATE_PR |
| No PR, no commits | → IMPLEMENT |

---

### Mode: CLEANUP (PR Merged)

```bash
# 1. Mark task done
task-master tags use "$TAG" && task-master set-status --id="$TASK_ID" --status=done

# 2. Navigate out and cleanup
WORKTREE_PATH=$(pwd)
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git worktree remove "$WORKTREE_PATH"
git branch -d "$TAG/$TASK_DIR"
```

**Report:**
```
## Task Complete

**Task**: <tag>.<task-id> marked done
**Worktree**: Removed
**Branch**: Deleted

Next: `task-master next --tag <tag>` or `/tm <tag> <next-task>`
```

---

### Mode: REVIEW (PR Open)

**Check states in priority order:**

| Priority | State | Action |
|----------|-------|--------|
| 1 | Conflicts | Merge target into PR branch |
| 2 | CI failing | Fix CI |
| 3 | Has feedback | Polish (subagent) |
| 4 | Approved | Check for suggestions anyway |
| 5 | Waiting | Report "awaiting review" |
| 6 | Draft | Ask if ready to publish |

```bash
# Gather all info
gh pr checks
gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | {author: .user.login, path, line, body: .body[0:150]}'
gh pr view --comments
```

**For CONFLICTS:**
```bash
BASE=$(gh pr view --json baseRefName --jq '.baseRefName')
git fetch origin && git merge origin/$BASE --no-edit
# Resolve if needed, commit, push
```

**For FEEDBACK/SUGGESTIONS:**
```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Polish PR #<number>.

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

Feedback: <summarized>

Triage:
- Quick win → implement
- Out of scope → TODO(tm:<tag>.<task-id>): <summary>
- Disagree → note why

Fix, commit, push.
"""
)
```

**For APPROVED & READY:**
```
## PR #<number> - Ready to Merge

All checks passing. No unaddressed feedback.
Merge when ready, then run `/tm` to cleanup.
```

---

### Mode: CREATE_PR (Commits but no PR)

```bash
git log origin/develop..HEAD --oneline  # Show what will be in PR
```

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Create PR for commits on this branch.

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

Push and create PR with `gh pr create`. Include TM reference.
Output PR_NUMBER and PR_URL.
"""
)
```

Then enter quality loop.

---

### Mode: IMPLEMENT (No commits yet)

Continue implementation. Same as START Step 4-6.

---

## Task Status Lifecycle

```
/tm starts (START)  → Task: pending → in-progress
Implementation      → Subtasks: pending → done
All subtasks done   → Task: STAYS in-progress ⚠️
PR created/polished → Task: STAYS in-progress
Human merges PR     → (no command runs)
/tm after merge     → Task: in-progress → done
```

**CRITICAL**: Parent task only marked `done` in CLEANUP mode after human merge.

---

## Error Handling

- **Subagent fails**: Ask user - retry, skip, or abort
- **Dependencies not met**: Report and ask
- **Stuck after 10 iterations**: Ask for help

---

## Notes

- **Context-aware**: Same command, behavior adapts to where you are
- **Human merges**: Never merge automatically
- **Subagents can bail**: If work too large, report and suggest decomposition
- **Talk to CodeRabbit**: Comment to teach it patterns it keeps flagging
