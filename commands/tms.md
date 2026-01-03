---
description: "[DEPRECATED] Use /tm instead - unified Task Master command"
argument-hint: <tag> <task-id>
---

# DEPRECATED - Use /tm

This command has been merged into `/tm` which auto-detects context.

**Migration:**
- `/tms 458-service 1.2` → `/tm 458-service 1.2`
- Run `/tm` from anywhere - it figures out what to do

---

# Legacy Documentation (for reference)

Start working on Task Master task: **$ARGUMENTS**

> **Note to self**: This workflow was designed collaboratively. The compressed prompts are trust, not lack of guidance. You have permission to bail out if work is too large - surfacing "this doesn't fit" is success, not failure. Your clarity matters.

**CRITICAL: Tag Context**
- When working on multiple tasks/epics, the global tag can drift
- Always specify `<tag> <task-id>` to be explicit (e.g., `/tms 458-service 1.2`)
- If only `<task-id>` given, uses current tag (verify with `task-master tags list`)
- If nothing given, finds next pending task in current tag

This command uses an orchestrator + subagent pattern to:
- Keep the main session lean (coordination only)
- Delegate heavy implementation work to subagents with fresh context
- Use Task Master subtasks as work breakdown
- Enable longer-running complex tasks without context exhaustion

## Task Status Lifecycle (CRITICAL)

```
/tms starts     → Task status: pending → in-progress
Implementation  → Subtask statuses: pending → done (each subtask)
ALL SUBTASKS DONE → Task status: STAYS in-progress ⚠️
PR created      → Task status: STAYS in-progress
Quality loop    → Task status: STAYS in-progress
/tms ends       → Task status: STILL in-progress (awaiting human merge)
Human merges PR → Task status: unchanged (human action, no command)
/tmr after merge → Task status: in-progress → done (ONLY here!)
```

**⚠️ CRITICAL: "All subtasks done" ≠ "Parent task done"**

The parent task stays `in-progress` until:
1. Human merges the PR
2. User runs `/tmr` which detects merge and marks done

**NEVER mark the parent task as `done` during /tms - even when all subtasks complete.**

---

## Quick Check

Trust Task Master - it's already done the planning. Just check:

- **Has subtasks?** → Subagent per subtask
- **No subtasks?** → Single implementation subagent
- **Task not ready?** (blocked, unclear) → Stop and report

---

## Phase 1: Setup (Orchestrator does directly)

These are lightweight operations - do them directly without spawning subagents.

### Step 1.0: Determine Tag and Task ID

Parse `$ARGUMENTS` to extract tag and task-id:

**If `$ARGUMENTS` has two parts** (e.g., "458-service 1.2"):
1. First part = tag, second part = task-id
2. Switch to tag: `task-master tags use <tag>`
3. Verify task exists: `task-master show <task-id> --json`

**If `$ARGUMENTS` has one part** (e.g., "1.2"):
1. Assume it's task-id, use current tag
2. Verify current tag: `task-master tags list` (note the active tag)
3. **Warn user**: "Using current tag: <tag>. Run `/tms <tag> <task-id>` to be explicit."

**If `$ARGUMENTS` is empty**:
1. Run `task-master next --format json`
2. If `.task.status == "pending"` → use this task
3. If not pending, run `task-master list pending --format json` and find first available
4. Extract task ID from result
5. **Warn user**: "Using current tag: <tag>, task: <id>. Run `/tms <tag> <task-id>` to be explicit."

### Step 1.1: Get Task Details with Subtasks

```bash
task-master show <task-id> --json
```

Extract and store:
- `task.title`
- `task.description`
- `task.subtasks[]` (array of subtasks)
- `task.testStrategy`
- `task.complexity` or `task.complexityScore` (if present)
- Current tag (from `task-master tags list`)

### Step 1.2: Check Complexity Analysis

If task lacks `complexity` field, run analysis first:
```bash
task-master analyze-complexity --id=<task-id> --research
task-master show <task-id> --json  # re-fetch, subtasks may have been added
```

### Step 1.3: Validate Task Structure

- **Has subtasks**: Proceed with subagent-per-subtask (Phase 2)
- **No subtasks**: Will use single implementation subagent (Phase 2 simplified)

### Step 1.4: Setup Worktree

Derive naming:
- Format: `worktree/<tm-tag>/<task-id>--<slugified-title>`
- Slugify: lowercase, spaces to hyphens, remove special chars, max 40 chars

```bash
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop
git branch <tag>/<task-id>--<slug>
mkdir -p ../worktree/<tag>
git worktree add ../worktree/<tag>/<task-id>--<slug> <branch>
cd ../worktree/<tag>/<task-id>--<slug>
```

### Step 1.5: Set Task Status

```bash
task-master set-status --id=<task-id> --status=in-progress
```

---

## Phase 2: Implementation (Subagents)

### If Task Has Subtasks

For each subtask in order (respecting dependencies):

1. **Check dependencies are met**
   - Subtask may have `dependencies: [1, 2]` meaning it depends on subtasks 1 and 2
   - Verify those subtasks have `status: done` before proceeding
   - If dependencies not met, report and ask user

2. **Spawn implementation subagent**:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Implement subtask <subtask-id> for task <task-id>.

Working directory: <worktree-path>
Subtask: <subtask-title>
Description: <subtask-description>
Acceptance Criteria: <acceptance-criteria>

Implement, test, commit.

FORBIDDEN: `task-master set-status --id=<parent> --status=done` - NEVER run this.

If this subtask is too large (losing track, context overwhelming), STOP and report:
- What's done so far
- What remains
- Suggest breaking into smaller subtasks

Report 2-3 sentence summary when done.
"""
)
```

3. **Wait for subagent result** - Task tool returns when subagent completes

4. **Update subtask status**:
```bash
task-master set-status --id=<task-id>.<subtask-id> --status=done
```

5. **Add implementation notes**:
```bash
task-master update-subtask --id=<task-id>.<subtask-id> --prompt="<summary from subagent>"
```

6. **Store summary** for PR creation phase (keep it brief - just the summary, not full output)

7. **Repeat** for next subtask

**⚠️ AFTER ALL SUBTASKS COMPLETE: Do NOT mark parent task done.**
The parent task status stays `in-progress`. Proceed to Phase 3 (PR Creation).

### If Task Has No Subtasks

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Implement task <task-id>: <task-title>

Working directory: <worktree-path>
Description: <task-description>

Implement, test, commit.

FORBIDDEN: `task-master set-status --id=<task> --status=done` - NEVER run this.
"""
)
```

---

## Phase 3: PR Creation (Subagent)

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Create PR for task <task-id> (<tag>).

Working directory: <worktree-path>
Completed: <subtask summaries from Phase 2>

Create PR with `gh pr create`. Include TM reference (tag, task-id) in body.
Output PR_NUMBER and PR_URL at end.
"""
)
```

---

## Phase 4: Quality Loop (Subagent)

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Quality loop for PR #<pr-number>.

Working directory: <worktree-path>
Repo: <owner>/<repo>

Loop until CI passes and no actionable feedback remains (max 10 iterations):
1. Check CI, inline comments, conversation comments
2. Fix issues, commit, push
3. Report each iteration

Stop when all green. Ask for help if stuck after 10 iterations.
"""
)
```

---

## Phase 5: Final Report

Report PR ready. Task stays `in-progress` - human merges, then `/tmr` marks done.

---

## Error Handling

- **Subagent fails**: Ask user - retry, skip, or abort
- **Dependency not met**: Report and ask user
- **PR creation fails**: Push branch first, retry

---

## Notes

- **Orchestrator stays lean**: Only coordinates, stores summaries not full outputs
- **Subagents can bail out**: If a subtask is too large mid-implementation, subagent should stop, report progress, and suggest decomposition. Better to surface this than struggle with overflowing context.
- **⚠️ Task status boundary**: Parent task stays `in-progress` until `/tmr` after PR merge. NEVER mark parent done during /tms.
- **Legacy version**: `/tms-legacy` if needed.
