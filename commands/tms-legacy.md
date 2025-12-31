---
description: "[LEGACY] Monolithic Task Master start - use /tms instead"
argument-hint: [task-id] (optional - finds next available if omitted)
---

# Task Master Start (Legacy - Monolithic)

**Note: This is the legacy version. Use `/tms` for the improved orchestrator pattern.**

Start working on Task Master task: **$ARGUMENTS**

This command will:
1. Set up a worktree for the task
2. Implement the task fully
3. Commit changes and raise a PR
4. Enter autonomous PR fixing mode until all checks pass

## Phase 1: Setup

### Step 0: Determine Task ID (if not provided)

If `$ARGUMENTS` is empty or not provided:
1. Run `task-master next --format json` to find the next available task
2. Check the returned task's status:
   - If `.task.status == "pending"` → use this task
   - If `.task.status != "pending"` (e.g., `in-progress`) → fall back to manual selection
3. Extract the task ID from the JSON response: `.task.id`
4. If no task is found (`.found == false`), report "No pending tasks available" and stop

**Fallback if `next` returns a non-pending task**:
The `task-master next` command may return in-progress tasks. If the returned status is not "pending":
1. Run `task-master list pending --format json`
2. Identify all tasks with `status == "pending"`
3. Filter for tasks where dependencies are met:
   - `dependencies` is empty, OR
   - All dependency task IDs have `status == "done"` (check via `task-master show <dep-id>` if needed)
4. Sort remaining candidates by:
   - Priority: high > medium > low
   - Then by ID (numerically lowest first for determinism)
5. Use the first matching task
6. If no pending tasks with met dependencies exist, report "No available pending tasks - all are blocked or in progress"

### Step 1: Get task details from Task Master

Using the task ID (from `$ARGUMENTS` or determined in Step 0):
- Current tag (from `task-master tags list` - find the active one)
- Task title (from `task-master show <task-id>`)

### Step 2: Derive worktree naming

- Format: `worktree/<tm-tag>/<task-id>--<slugified-title>`
- Example: `worktree/458-transaction-service/1.2--create-schema-migration`
- Slugify rules: lowercase, spaces to hyphens, remove special chars, max 40 chars

### Step 3: Prepare the worktree

```bash
# From repo root (parent of <repo>-main)
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop

# Create branch with same name as worktree path (flattened)
# Branch name: <tag>/<task-id>--<slug>
git branch <tag>/<task-id>--<slug>

# Create nested worktree directory structure
mkdir -p ../worktree/<tag>
git worktree add ../worktree/<tag>/<task-id>--<slug> <tag>/<task-id>--<slug>
```

### Step 4: Set Task Master status

```bash
task-master set-status --id=<task-id> --status=in-progress
```

## Phase 2: Implementation

### Step 5: Implement the task

1. Read the task details thoroughly (title, description, details, testStrategy)
2. Explore the codebase to understand the context and patterns
3. Create a todo list using TodoWrite to track implementation steps
4. Implement the task following existing patterns and conventions
5. Run tests and fix any failures
6. Update implementation notes: `task-master update-subtask --id=<task-id> --prompt="implementation notes..."`

**Implementation principles:**
- Follow existing code patterns in the codebase
- Write tests as specified in testStrategy
- Keep changes focused on the task scope
- Don't over-engineer or add unrequested features

## Phase 3: PR Creation

### Step 6: Commit and push

```bash
# Stage all changes
git add .

# Commit with conventional format referencing task
git commit -m "feat: <task-title-lowercase> (task <task-id>)"

# Push branch
git push -u origin <branch-name>
```

### Step 7: Create PR

```bash
gh pr create --title "<task-title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points from task description>

## Task Master Reference
- **Tag**: <tm-tag>
- **Task ID**: <task-id>

## Changes Made
<List key changes>

## Test Plan
<From task's testStrategy field>
EOF
)"
```

## Phase 4: Autonomous PR Fixing

### Step 8: Enter autonomous fixing loop

After creating the PR, autonomously fix issues until all checks pass:

1. **Auto-trigger after push**: Immediately check PR status
2. **Autonomous loop** (NO user input needed):
   - Wait 30 seconds for CI to start
   - Check CI status: `gh pr checks <number> --repo <owner>/<repo>`
   - Check inline comments: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
   - Check conversation comments: `gh pr view <number> --comments`
   - **If ANY failures or bot comments exist:**
     - Analyze root cause
     - Fix issues
     - Commit and push
     - Report: "🔄 Iteration N: Fixed X failures, Y comments"
     - CONTINUE loop (no permission needed)
   - **If ALL passing and NO unresolved comments:**
     - Report: "✅ PR #X: All checks passing, ready for human review"
     - STOP and wait for user

3. **Only ask for help if:**
   - You hit a blocker you genuinely cannot resolve
   - After 5+ iterations with no progress
   - You need clarification on a design decision

## Important Notes

- The worktree path encodes: tag, task ID, and task description
- This allows `/tmr` to derive context without arguments
- Branch name mirrors worktree structure for traceability
- NEVER work in `<repo>-main` - always use the worktree
- Complete the ENTIRE task before raising PR - no partial implementations
- Follow the Autonomous PR Fixing Protocol from CLAUDE.md
