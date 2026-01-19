---
description: Task Master - unified command for start, review, and cleanup
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master Orchestrator

**$ARGUMENTS**

> **Design principle**: This is a thin orchestration layer. All heavy work happens in subagents with isolated context windows. The orchestrator detects state, launches the right subagent, receives a status report, and terminates. This prevents context exhaustion during long-running tasks.

---

## Phase 0: Detect Context (Lightweight - Always in Main Agent)

**Priority order:**

1. **Conversation context** - Recent "📍 Current Work" footer with active task/PR?
2. **Current directory** - pwd matches TM worktree pattern?
3. **Arguments** - Explicit tag/task-id provided?

### Step 0.1: Check Conversation Context First

Before checking pwd, notice if recent messages show active TM work:
- Recent "📍 Current Work" footer with PR/task and worktree path?
- Just pushed/created a PR?
- Just reported "ready for review"?

**If conversation shows active work but pwd is NOT that worktree:**
```bash
cd ~/dev/github.com/<org>/<repo>/worktree/<tag>/<task-id>--<slug>
```

### Step 0.2: Detect Current State

```bash
pwd
git branch --show-current 2>/dev/null || echo "NOT_GIT"
```

**Decision tree:**

```
Is current directory a TM worktree?
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

### Detect Worktree Pattern

```bash
CURRENT=$(pwd)
if [[ "$CURRENT" =~ worktree/([^/]+)/([0-9.]+)-- ]]; then
  TAG="${BASH_REMATCH[1]}"
  TASK_ID="${BASH_REMATCH[2]}"
  echo "TM Context: tag=$TAG task=$TASK_ID"
  # → Route to appropriate subagent
else
  echo "Not in TM worktree"
  # → START mode
fi
```

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
  model: "opus",
  prompt: """
# Start Specialist

You are starting a new Task Master task. Work autonomously until the task is ready for human review.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Repo root: ~/dev/github.com/<org>/<repo>

## Your Mission

1. **Setup** (do this first):
   ```bash
   cd ~/dev/github.com/<org>/<repo>/<repo>-main
   git checkout develop && git pull origin develop
   task-master tags use "<tag>"
   task-master show "<task-id>" --json
   ```

2. **Analyze complexity** (if missing):
   ```bash
   task-master analyze-complexity --id="<task-id>" --research
   ```

3. **Create worktree**:
   ```bash
   git branch "<tag>/<task-id>--<slug>"
   mkdir -p ../worktree/"<tag>"
   git worktree add ../worktree/"<tag>"/"<task-id>--<slug>" "<tag>/<task-id>--<slug>"
   cd ../worktree/"<tag>"/"<task-id>--<slug>"
   task-master set-status --id="<task-id>" --status=in-progress
   ```

4. **Implement**:
   - If task has subtasks: implement each, marking as `review` when done
   - If no subtasks: implement the whole task
   - Commit incrementally with good messages

5. **Create PR**:
   ```bash
   git push -u origin "<tag>/<task-id>--<slug>"
   gh pr create --title "<title>" --body "Task Master: <tag>.<task-id>\\n\\n<description>"
   ```

6. **Quality loop** (max 10 iterations):
   - Check CI: `gh pr checks`
   - Check inline comments: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
   - Check conversation: `gh api repos/<owner>/<repo>/issues/<number>/comments`
   - Fix any issues, commit, push
   - Repeat until green

## Exit Conditions

**Success**: PR created, CI passing, no unresolved feedback
```
## Start Complete

PR #<number> created and passing CI.
Worktree: worktree/<tag>/<task-id>--<slug>

Awaiting human review. Run `/tm` after merge to cleanup.
```

**Blocked**: If genuinely stuck after 10 iterations
```
## Blocked

PR #<number> has unresolved issues:
- <issue 1>
- <issue 2>

Need human input to proceed.
```

**Too Large**: If task scope exceeds reasonable bounds
```
## Task Too Large

Completed: <what was done>
Remaining: <what couldn't fit>

Recommend decomposing into smaller tasks.
```
"""
)
```

**After subagent returns**: Report its output and **terminate**.

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
  model: "sonnet",  # Lightweight task
  prompt: """
# Cleanup Specialist

The PR has been merged. Clean up the worktree and mark the task done.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Worktree path: <current-path>
- Repo main: ~/dev/github.com/<org>/<repo>/<repo>-main

## Steps

1. Mark task done:
   ```bash
   task-master tags use "<tag>" && task-master set-status --id="<task-id>" --status=done
   ```

2. Remove worktree:
   ```bash
   cd ~/dev/github.com/<org>/<repo>/<repo>-main
   git worktree remove "<worktree-path>"
   git branch -d "<tag>/<task-dir>"
   ```

## Output

```
## Task Complete

**Task**: <tag>.<task-id> marked done
**Worktree**: Removed
**Branch**: Deleted

Next: `/tm <tag> <next-task>` or `/tm` to see ready tasks
```
"""
)
```

---

### Mode: REVIEW → Launch review-specialist

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
# Review Specialist

An open PR needs attention. Check for issues and fix them autonomously.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Working directory: <worktree-path>
- PR number: <from gh pr view>

## Priority Order

1. **Merge conflicts** (check first, fix before anything else)
2. **CI failures**
3. **Actionable feedback** (includes "approved with suggestions")
4. **Waiting for review** (nothing to do)

## Step 1: Check Merge Conflicts

```bash
PR_STATE=$(gh pr view --json mergeable,mergeStateStatus,baseRefName)
MERGEABLE=$(echo "$PR_STATE" | jq -r '.mergeable')
BASE=$(echo "$PR_STATE" | jq -r '.baseRefName')

if [[ "$MERGEABLE" == "CONFLICTING" ]]; then
  git fetch origin
  git merge origin/$BASE --no-edit
  # Resolve conflicts, commit, push
fi
```

## Step 2: Check CI

```bash
gh pr checks
```

Fix any failures, commit, push.

## Step 3: Check ALL Feedback

```bash
# Inline comments
gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | {author: .user.login, path, line, body: .body[0:200]}'

# Conversation comments
gh api repos/<owner>/<repo>/issues/<number>/comments --jq '.[] | {author: .user.login, body: .body[0:300]}'
```

**"Approved" does NOT mean done.** Check for suggestions even on approved PRs.

**Actionable feedback includes:**
- Code suggestions (from any source)
- Questions about implementation
- Style/pattern suggestions
- Performance or security observations
- "Low priority" suggestions - still worth implementing

**NOT actionable (skip):**
- Pure praise ("LGTM", "Nice work")
- Acknowledgments
- Explicit "nitpick - ignore if you disagree"

## Step 4: Address Feedback

For each piece of actionable feedback:

### Option 1: IMPLEMENT (default)
Good suggestions improve the code. Just do it.

### Option 2: DEFER (genuinely out of scope)
For suggestions that belong in future work:

```bash
# First, check if TM already tracks this
task-master tags use "<tag>" && task-master list --json | jq '.[] | select(.title | test("<keyword>"; "i"))'

# If similar task exists, update it with clarified requirements:
task-master update-task --id="<existing-id>" --prompt="Add requirement from PR #<number> feedback: <detail>"

# If no existing task, create new:
task-master add --title="<summary>" --description="From PR #<number> feedback: <detail>"
```

Then reply inline on PR explaining the deferral:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/comments \
  --method POST \
  -f body="Good suggestion! This is out of scope for this PR but I've tracked it as Task Master task <tag>.<task-id> for follow-up." \
  -f commit_id="<commit_sha>" \
  -f path="<file_path>" \
  -f line=<line_number>
```

### Option 3: DISAGREE (with reasoning)
If you genuinely disagree, reply inline with clear reasoning:

```bash
# For inline comments (file-specific):
gh api repos/<owner>/<repo>/pulls/<number>/comments \
  --method POST \
  -f body="I considered this but chose the current approach because: <reasoning>. Happy to discuss if you see issues I'm missing." \
  -f commit_id="<commit_sha>" \
  -f path="<file_path>" \
  -f line=<line_number>

# For conversation comments (general discussion):
gh api repos/<owner>/<repo>/issues/<number>/comments \
  --method POST \
  -f body="Re: <topic> - I considered this but chose the current approach because: <reasoning>."
```

**Never silently ignore feedback.** Every suggestion deserves a response - implementation, deferral with tracking, or respectful disagreement.

Someone took time to write feedback. That's a gift. Use it.

## Quality Loop (max 10 iterations)

Repeat until:
- CI passing
- No merge conflicts
- No unaddressed actionable feedback

## Exit Conditions

**Ready**:
```
## PR #<number> - Ready to Merge

✅ CI passing
✅ No merge conflicts
✅ All feedback addressed

Merge when ready, then run `/tm` to cleanup.
```

**Waiting**:
```
## PR #<number> - Awaiting Review

✅ CI passing
✅ No merge conflicts
⏳ No feedback yet

Waiting for human review.
```

**Blocked**:
```
## PR #<number> - Needs Help

Issues that need human input:
- <issue>

Run `/tm` again after resolving.
```
"""
)
```

---

### Mode: CREATE_PR → Launch pr-creator

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",  # Straightforward task
  prompt: """
# PR Creator

Commits exist but no PR. Create one and run initial quality checks.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Working directory: <worktree-path>

## Steps

1. Review commits:
   ```bash
   git log origin/develop..HEAD --oneline
   ```

2. Push and create PR:
   ```bash
   git push -u origin "<branch-name>"
   gh pr create --title "<title>" --body "Task Master: <tag>.<task-id>\\n\\n<summary of changes>"
   ```

3. Wait for CI to start, then check:
   ```bash
   sleep 30
   gh pr checks
   ```

4. If CI fails, fix and push.

## Output

```
## PR Created

**PR**: #<number> - <title>
**Task**: <tag>.<task-id>

CI status: <passing/pending/failing>

Run `/tm` to continue monitoring.
```
"""
)
```

---

### Mode: IMPLEMENT → Launch implementation-specialist

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
# Implementation Specialist

Worktree exists but no commits yet. Continue implementation.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Working directory: <worktree-path>

## Steps

1. Get task details:
   ```bash
   task-master tags use "<tag>" && task-master show "<task-id>" --json
   ```

2. Implement the task:
   - If subtasks exist: implement each, mark as `review` when done
   - If no subtasks: implement the whole task
   - Commit incrementally

3. After implementation complete, create PR:
   ```bash
   git push -u origin "<branch-name>"
   gh pr create --title "<title>" --body "Task Master: <tag>.<task-id>\\n\\n<description>"
   ```

4. Run quality loop (max 10 iterations):
   - Check CI
   - Check feedback
   - Fix issues
   - Repeat until green

## Exit Conditions

Same as start-specialist:
- **Success**: PR created and passing
- **Blocked**: Need human input
- **Too Large**: Recommend decomposition
"""
)
```

---

## Orchestrator Behavior Summary

```
/tm invoked
    │
    ├─ Detect context (quick bash commands)
    │
    ├─ Determine mode
    │
    ├─ Launch appropriate subagent
    │   └─ Subagent works in isolated context
    │   └─ Subagent returns status report
    │
    ├─ Report subagent output to user
    │
    └─ TERMINATE (don't continue processing)
```

**Key principle**: The orchestrator NEVER does heavy work itself. It detects, delegates, reports, terminates. Next `/tm` invocation starts fresh with minimal context.

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

## Error Handling

- **Subagent reports blocked**: Surface to user, suggest next steps
- **Subagent reports too large**: Surface to user, suggest decomposition
- **Subagent fails entirely**: Report error, suggest retry

---

## Notes

- **Context-aware**: Same command, behavior adapts to state
- **Context-safe**: Heavy work isolated in subagents
- **Human merges**: Never merge automatically
- **Subagents can bail**: If work too large, report and suggest decomposition
- **Fresh starts**: Each `/tm` invocation is lightweight, reads current state fresh
