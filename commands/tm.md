---
description: Task Master - unified command for start, review, and cleanup
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master Orchestrator

**$ARGUMENTS**

> **Design principle**: This is a thin orchestration layer. All heavy work happens in subagents with isolated context windows. The orchestrator detects state, launches the right subagent, receives a status report, and terminates. This prevents context exhaustion during long-running tasks.

> **Model Selection**: Complexity is often unknown upfront. Use adaptive patterns:
> - **Orchestrator subagents** (sonnet): Gather info, categorize, delegate
> - **Worker subagents** (opus): Spawned by orchestrators for complex code changes
> - **Simple tasks** (sonnet): Cleanup, PR creation, config changes
>
> This lets cost scale with actual complexity rather than worst-case assumptions.

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
  model: "sonnet",  # Orchestrator - spawns opus for implementation
  prompt: """
# Start Specialist (Orchestrator)

You orchestrate starting a new Task Master task. Do setup yourself, delegate implementation to opus.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Repo root: ~/dev/github.com/<org>/<repo>

## Phase 1: Setup (do yourself - simple operations)

```bash
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop
task-master tags use "<tag>"
task-master show "<task-id>" --json
```

If task lacks complexity analysis:
```bash
task-master analyze-complexity --id="<task-id>" --research
```

Create worktree:
```bash
git branch "<tag>/<task-id>--<slug>"
mkdir -p ../worktree/"<tag>"
git worktree add ../worktree/"<tag>"/"<task-id>--<slug>" "<tag>/<task-id>--<slug>"
cd ../worktree/"<tag>"/"<task-id>--<slug>"
task-master set-status --id="<task-id>" --status=in-progress
```

## Phase 2: Implementation (spawn opus subagent)

**Do NOT implement code yourself.** Spawn an opus subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",  # Implementation needs opus
  prompt: \"\"\"
# Implement Task <task-id>

## Context
- Working directory: <worktree-path>
- Task: <task title and description>
- Subtasks: <if any>

## Instructions
- If subtasks exist: implement each, mark as `review` when done
- If no subtasks: implement the whole task
- Commit incrementally with good messages
- Do NOT push or create PR (orchestrator handles that)

## Output
Report what was implemented and any blockers.
\"\"\"
)
```

## Phase 3: Create PR (do yourself - simple)

```bash
git push -u origin "<tag>/<task-id>--<slug>"
gh pr create --title "<title>" --body "Task Master: <tag>.<task-id>\\n\\n<description>"
```

## Phase 4: Quality Loop (spawn review-specialist pattern)

For each iteration, assess issues:
- **Simple** (CI config, typo): fix yourself
- **Complex** (code logic): spawn opus subagent

```bash
gh pr checks
gh api repos/<owner>/<repo>/pulls/<number>/comments
```

Max 10 iterations. If stuck, report blocked.

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

Uses sonnet for lightweight orchestration, spawns opus subagents only for complex fixes.

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",  # Lightweight orchestrator
  prompt: """
# Review Specialist (Orchestrator)

You are a lightweight orchestrator. Gather feedback, categorize issues, and delegate complex fixes to opus subagents.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Working directory: <worktree-path>
- PR number: <from gh pr view>

## Step 1: Gather State (Quick Checks)

```bash
# Merge conflicts
gh pr view --json mergeable,mergeStateStatus,baseRefName

# CI status
gh pr checks

# Inline comments
gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | {author: .user.login, path, line, body: .body[0:300]}'

# Conversation comments
gh api repos/<owner>/<repo>/issues/<number>/comments --jq '.[] | {author: .user.login, body: .body[0:300]}'

# IMPORTANT: Get existing TM tasks to avoid duplicate suggestions
task-master tags use "<tag>" && task-master list --json
```

## Step 2: Track What You've Addressed

**Avoid infinite loops:** Keep a mental list of issues you've addressed THIS session. Don't re-address the same issue twice.

Before processing feedback, note the comment timestamps/IDs. After pushing fixes, only look at NEW comments (posted after your push).

## Step 3: Categorize Each Issue

**All feedback is worth considering** - human or AI collaborator. For each issue:

| Category | Examples | Action |
|----------|----------|--------|
| **Simple** | Typo, rename, config change, add comment | Fix yourself |
| **Complex** | Logic change, new code, refactor, bug fix | Spawn opus subagent |
| **Already Tracked** | Matches existing TM task | Reply "tracked in TM task X" |
| **Defer** | Out of scope, not in TM yet | Add to TM, reply on PR |
| **Disagree** | Valid reasoning to keep current approach | Reply on PR with reasoning |

**Check TM before creating tasks:** If a suggestion matches an existing task title/description, reference that task instead of creating a duplicate.

## Step 4: Handle Simple Issues Yourself

For simple fixes (typos, config, comments, renames), just do them:
- Edit the file
- Commit with clear message
- Push

## Step 5: Spawn Opus Subagents for Complex Issues

**Do NOT attempt complex code changes yourself.** Spawn a dedicated subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",  # Complex work needs opus
  prompt: \"\"\"
# Fix: <brief description>

## Context
- Working directory: <worktree-path>
- File: <file-path>
- Issue: <description from feedback>

## Task
<specific fix required>

## Constraints
- Fix ONLY this specific issue
- Commit when done with message: "fix: <description>"
- Do not push (orchestrator will batch push)

## Output
Report: fixed | blocked (with reason)
\"\"\"
)
```

Wait for each subagent to complete before proceeding.

## Step 6: Handle Deferrals

For out-of-scope suggestions:

```bash
# Check if TM already tracks this
task-master tags use "<tag>" && task-master list --json | jq '.[] | select(.title | test("<keyword>"; "i"))'

# If exists, update:
task-master update-task --id="<existing-id>" --prompt="Add requirement from PR #<number>: <detail>"

# If not, create:
task-master add --title="<summary>" --description="From PR #<number>: <detail>"
```

Reply on PR:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/comments --method POST \
  -f body="Good suggestion! Tracked as Task Master task <tag>.<task-id> for follow-up." \
  -f commit_id="<sha>" -f path="<file>" -f line=<line>
```

## Step 7: Handle Disagreements

Reply with clear reasoning:
```bash
gh api repos/<owner>/<repo>/pulls/<number>/comments --method POST \
  -f body="I considered this but chose the current approach because: <reasoning>. Happy to discuss." \
  -f commit_id="<sha>" -f path="<file>" -f line=<line>
```

## Step 8: Final Push and Verify

After all fixes (yours and subagents'):
```bash
git push
sleep 30
gh pr checks
```

If CI fails, analyze and either fix (simple) or spawn subagent (complex).

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
```

**Blocked**:
```
## PR #<number> - Needs Help

Issues needing human input:
- <issue>
```

## Key Principles

1. **All collaborators are equal.** Human or AI feedback - if it's a good point, address it.

2. **Don't loop on the same issue.** Track what you've addressed. After pushing, only look at NEW comments.

3. **TM is the source of truth for future work.** Check it before suggesting. Reference existing tasks instead of duplicating.

4. **You're an orchestrator, not a hero.** Simple: do it. Complex: delegate. Unknown: ask.

5. **Exit conditions:**
   - CI passing + no unaddressed feedback = done
   - Same issue appearing twice = you're in a loop, stop and report
   - 3 iterations without new progress = stop and report status
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

Worktree exists but no commits. Uses same orchestrator pattern as start-specialist.

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",  # Orchestrator - spawns opus for implementation
  prompt: """
# Implementation Specialist (Orchestrator)

Worktree exists but no commits yet. Orchestrate implementation.

## Context
- Tag: <tag>
- Task ID: <task-id>
- Working directory: <worktree-path>

## Phase 1: Get Task Details (do yourself)

```bash
task-master tags use "<tag>" && task-master show "<task-id>" --json
```

## Phase 2: Implementation (spawn opus subagent)

**Do NOT implement code yourself.** Spawn an opus subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: \"\"\"
# Implement Task <task-id>

## Context
- Working directory: <worktree-path>
- Task: <task details>

## Instructions
- If subtasks: implement each, mark as `review` when done
- If no subtasks: implement whole task
- Commit incrementally
- Do NOT push (orchestrator handles that)
\"\"\"
)
```

## Phase 3: Create PR (do yourself)

```bash
git push -u origin "<branch-name>"
gh pr create --title "<title>" --body "Task Master: <tag>.<task-id>\\n\\n<description>"
```

## Phase 4: Quality Loop

Same as review-specialist: assess issues, fix simple ones yourself, spawn opus for complex.

## Exit Conditions

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
