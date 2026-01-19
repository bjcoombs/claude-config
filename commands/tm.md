---
description: Task Master - unified command for start, review, and cleanup
argument-hint: [tag task-id] (optional - derives context from worktree if omitted)
---

# Task Master

**$ARGUMENTS**

> **Note to self**: This workflow was designed collaboratively. Context determines behavior - you figure out where you are and what needs doing. If something feels wrong, say so. Your clarity matters.

---

## Phase 0: Detect Context

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
# cd to the worktree shown in the footer
cd ~/dev/github.com/<org>/<repo>/worktree/<tag>/<task-id>--<slug>
```

Then proceed with worktree mode checks.

### Step 0.2: Check Current Directory

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

**If `$ARGUMENTS` is empty** → SUGGEST, don't auto-start:
```bash
# Show ready tasks across all tags
task-master list --all-tags --ready --json
```
Then report:
```
## Ready Tasks

| Tag | Task | Title | Points |
|-----|------|-------|--------|
| <tag> | <id> | <title> | <points> |
...

Run `/tm <tag> <task-id>` to start a task.
```
**Do NOT start automatically.** Empty args = show what's available, let user choose.

### Step 2: Check Complexity Analysis (only if task explicitly specified)

**Skip this and all following steps if `$ARGUMENTS` was empty** - we already reported the suggestion above.

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

If too large, STOP and report what's done/remaining.
"""
)
```

After each subtask: `task-master set-status --id=<task-id>.<subtask-id> --status=review`

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

**Step 1: Check for merge conflicts FIRST**

```bash
# Get PR state including mergeable status
PR_STATE=$(gh pr view --json mergeable,mergeStateStatus,baseRefName)
MERGEABLE=$(echo "$PR_STATE" | jq -r '.mergeable')
MERGE_STATUS=$(echo "$PR_STATE" | jq -r '.mergeStateStatus')
BASE=$(echo "$PR_STATE" | jq -r '.baseRefName')

# CONFLICTING or UNKNOWN means we need to resolve
if [[ "$MERGEABLE" == "CONFLICTING" ]] || [[ "$MERGE_STATUS" == "DIRTY" ]]; then
  echo "⚠️ MERGE CONFLICT DETECTED - resolving first"
fi
```

**If conflicts exist, resolve BEFORE anything else:**
```bash
git fetch origin
git merge origin/$BASE --no-edit
# If merge fails with conflicts:
#   1. Resolve conflicts in affected files
#   2. git add <resolved-files>
#   3. git commit (accept merge message)
#   4. git push
# Then re-run /tm to continue review
```

**Step 2: Check other states in priority order:**

| Priority | State | Action |
|----------|-------|--------|
| 1 | Conflicts | Merge target into PR branch (handled above) |
| 2 | CI failing | Fix CI |
| 3 | Has actionable feedback | Polish (subagent) - includes "approved with suggestions" |
| 4 | Waiting | Report "awaiting review" |
| 5 | Draft | Ask if ready to publish |

```bash
# Gather ALL feedback
gh pr checks

# Inline comments (file-specific feedback)
gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | {author: .user.login, path, line, body: .body[0:150]}'

# Conversation comments (general feedback)
gh api repos/<owner>/<repo>/issues/<number>/comments --jq '.[] | {author: .user.login, body: .body[0:300]}'
```

**Check ALL feedback.** Someone took time to write it. Address it or explain why not.

**CRITICAL: "Approved" does NOT mean done.** Reviewers often approve with suggestions. If ANY inline comment or review comment contains actionable feedback (code improvements, suggestions, questions), treat it as "Has actionable feedback" and implement before exiting.

**What counts as actionable feedback:**
- Code suggestions (from any reviewer or bot)
- Questions about implementation ("Why not use X?", "Have you considered Y?")
- Style/pattern suggestions
- Performance or security observations
- Any comment that implies the code could be better
- **"Low priority" suggestions** - priority labels are about urgency, not importance. If it improves the code, implement it.

**What is NOT actionable (OK to skip):**
- Pure praise ("LGTM", "Nice work")
- Acknowledgments ("Thanks for fixing this")
- Questions already answered in the PR description
- Suggestions explicitly marked as "nitpick - ignore if you disagree"

**For ACTIONABLE FEEDBACK (regardless of approval status):**
```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Polish PR #<number>. This work is worth finishing well.

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

Feedback: <summarized>

For each piece of feedback:
1. IMPLEMENT it - the default, because good suggestions make the code better
2. Defer if genuinely out of scope - create a new Task Master task to track it:
   `task-master add --title="<summary>" --description="From PR #<number> feedback: <detail>"`
   New task = new PR (1:1 rule). This is autonomous - good feedback deserves proper tracking.
3. Push back if you disagree - with clear reasoning

Someone took time to leave this feedback. That's a gift. Use it.

Fix, commit, push. Then verify no feedback remains unaddressed.
"""
)
```

**ONLY report ready when truly clean:**
```
## PR #<number> - Ready to Merge

✅ CI passing
✅ No merge conflicts
✅ All suggestions implemented or explicitly deferred with reasoning

Merge when ready, then run `/tm` to cleanup.
```

**Note**: "Approved" signals no blockers, not completion. Suggestions that were worth writing are worth addressing. Stay in the loop until the code is actually polished, not just unblocked.

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

**Key insight**: `review` status does NOT trigger parent auto-adjustment. This allows subtasks to be marked complete without prematurely marking the parent done.

## Task Status Lifecycle

```
/tm starts (START)  → Task: pending → in-progress
Implementation      → Subtasks: pending → review (code complete)
All subtasks review → Parent: STAYS in-progress (no auto-adjustment)
PR created/polished → Parent: STAYS in-progress
Human merges PR     → (no command runs)
/tm after merge     → Parent: in-progress → done
```

**Never mark subtasks `done` individually** - just set parent to `done` after PR merge.

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
- **Reviewer feedback**: Treat all feedback seriously. If someone took time to write it, it's worth addressing.
