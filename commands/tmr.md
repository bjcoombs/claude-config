---
description: Review and polish a Task Master PR - assess state, address feedback
argument-hint: [pr-ref] (optional - #123 or URL to recover orphaned PR)
allowed-tools: Bash(task-master:*), Bash(git:*), Bash(gh:*), Bash(pwd:*), Bash(basename:*), Bash(dirname:*), Task, TodoWrite
---

# Task Master Review

Review a PR: assess its state and address any feedback. Also handles cleanup after merge.

> **Note to self**: You're trusted here. Short prompts = trust, not missing info. If suggestions are valuable, address them - "approved" doesn't mean "ignore feedback." If something feels wrong, say so. This is collaborative.

**Arguments**: `$ARGUMENTS`

**Use cases**:
- No argument: Review PR for current worktree (or cleanup if merged)
- `#123` or URL: Recover an orphaned PR (e.g., after crash)

---

## Phase 0: Determine Context

### If `$ARGUMENTS` is empty → Use Current Worktree

```bash
pwd
git branch --show-current
gh pr view --json number,url
```

Skip to Phase 1.

### If `$ARGUMENTS` contains PR reference → Take Ownership Mode

1. Get PR details: `gh pr view <number> --json number,title,headRefName,url,body`
2. Extract TM context from branch name (`<tag>/<task-id>--<slug>` pattern)
3. Find or create worktree for the branch
4. If TM context found: `task-master tags use <tag> && task-master set-status --id=<task-id> --status=in-progress`
5. Review commits and diff to understand state
6. Report takeover summary, then continue to Phase 1

---

## Phase 1: Assess State (Avoid Binary Thinking)

**Don't assume one-size-fits-all.** Check the actual state:

### PR State Spectrum

Check states in this order (earlier states block later ones):

| Priority | State | Indicators | Action |
|----------|-------|------------|--------|
| 1 | **Merged** | PR merged | Mark task done, cleanup worktree |
| 2 | **Conflicts** | mergeable=false, mergeStateStatus=DIRTY | Merge target into PR branch |
| 3 | **CI failing** | Checks failing | Fix CI first |
| 4 | **Has feedback** | Comments/suggestions exist | Enter polish mode |
| 5 | **Approved & ready** | Approvals, CI passing, no suggestions | Check for suggestions, then ready |
| 6 | **Waiting for review** | No comments, CI passing | Nothing to do yet |
| 7 | **Draft** | Draft state | Ask if ready to publish |
| 8 | **No PR** | Branch has no PR | Offer to create one |

### Gather Info

```bash
# Get current context
pwd
git branch --show-current

# Check if merged FIRST (determines cleanup vs review flow)
gh pr view --json mergedAt --jq '.mergedAt'
# If mergedAt is not null → State: MERGED, skip other checks

# Get PR state (only if not merged)
gh pr view --json number,title,state,url,isDraft,mergeable,reviewDecision

# Check CI
gh pr checks

# Check for merge conflicts (CRITICAL - check before other states)
gh pr view --json mergeable,mergeStateStatus --jq '{mergeable, mergeStateStatus}'
# If mergeable=false or mergeStateStatus="DIRTY" → State: CONFLICTS, handle first
```

---

## Phase 2: Respond to State

### State: MERGED → Cleanup

PR is merged. Mark task done and clean up worktree.

**CRITICAL: Derive tag from worktree path, not global tag state.**

```bash
# 1. Derive context from current directory
# Path format: .../worktree/<tag>/<task-id>--<slug>
WORKTREE_PATH=$(pwd)
TAG=$(basename $(dirname "$WORKTREE_PATH"))
TASK_DIR=$(basename "$WORKTREE_PATH")
TASK_ID="${TASK_DIR%%--*}"

# 2. Mark task done (chain tag switch for safety)
task-master tags use "$TAG" && task-master set-status --id="$TASK_ID" --status=done

# 3. Navigate to main and cleanup
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git worktree remove "$WORKTREE_PATH"
git branch -d "$TAG/$TASK_DIR"  # or use full branch name from earlier
```

**Report:**
```
## Task Complete

**Task**: <tag>/<task-id> marked done
**Worktree**: Removed
**Branch**: Deleted

Next task: `task-master next --tag <tag>`
```

---

### State: APPROVED & READY

**⚠️ "Approved" does NOT mean "ignore suggestions"**

Check for unaddressed suggestions even on approved PRs:

```bash
# Fetch ALL inline comments (not just blocking ones)
gh api repos/<owner>/<repo>/pulls/<pr-number>/comments \
  --jq '.[] | {author: .user.login, path, line, body: .body[0:200]}'

# Also check review comments
gh api repos/<owner>/<repo>/pulls/<pr-number>/reviews \
  --jq '.[] | select(.body != "") | {author: .user.login, state, body: .body[0:200]}'
```

**If suggestions exist** (even on approved PRs):
1. Spawn polish subagent with Suggestion Triage rules
2. Address quick wins, add TODOs for out-of-scope items
3. Push changes
4. THEN report ready to merge

**If genuinely no suggestions**:
```
## PR #<number> - Ready to Merge

All checks passing. Approvals received. No unaddressed suggestions.

**Action**: You can merge when ready, then run `/tmr` to cleanup.
```

**Key principle**: Approval means "no blocking issues", not "nothing to improve". Good suggestions improve the codebase - address them.

### State: CI FAILING → Fix First

1. Diagnose: `gh pr checks` and `gh run view <run-id> --log-failed`
2. Spawn fix subagent (focused on CI, not style)
3. After fix, reassess state

### State: WAITING FOR REVIEW

```
## PR #<number> - Awaiting Review

CI passing. No feedback yet.
Nothing to polish - waiting for reviewers.
```

### State: HAS FEEDBACK → Polish

1. **Fetch all feedback**:
   ```bash
   # Inline comments
   gh api repos/<owner>/<repo>/pulls/<pr-number>/comments \
     --jq '.[] | {author: .user.login, file: .path, line, body}'

   # Conversation comments
   gh pr view <pr-number> --comments

   # Reviews
   gh api repos/<owner>/<repo>/pulls/<pr-number>/reviews \
     --jq '.[] | {author: .user.login, state, body}'
   ```

2. **Summarize for user**:
   ```
   ## PR #<number> - Feedback to Address

   ### Summary
   - <N> inline comments
   - <N> conversation comments

   ### Suggestions
   1. <author>: <summary>
   2. <author>: <summary>
   ```

3. **Spawn polish subagent**:

```
Task(
  subagent_type: "general-purpose",
  model: "opus",
  prompt: """
Polish PR #<pr-number> based on feedback.

Working directory: <worktree-path>
Task Master: <tag>.<task-id>

Feedback: <summarized feedback>

Triage each suggestion:
- Quick win (< 5 min) → implement
- Out of scope → add TODO(tm:<tag>.<task-id>): <summary>
- Disagree → note why

Implement, test, commit, push. Report triage results.
"""
)
```

### State: CONFLICTS → Merge Target into PR Branch

```bash
BASE_BRANCH=$(gh pr view --json baseRefName --jq '.baseRefName')
git fetch origin
git merge origin/$BASE_BRANCH --no-edit
```

If conflicts occur, resolve them, commit, and push. Then reassess PR state.

### State: DRAFT → Ask

"PR is in draft. Mark ready for review?" If yes: `gh pr ready`

### State: NO PR → Offer to Create

"No PR found. Create one?" If yes, spawn PR creation subagent.

---

## Notes

- **/tmr is idempotent**: Run multiple times as feedback comes in
- **/tmr handles cleanup**: When PR is merged, marks task done and removes worktree
- **Tag is derived from path**: When cleaning up, tag comes from worktree path, not global state
- **Talk to CodeRabbit**: If you see recurring suggestions (same pattern across PRs), comment on the PR to teach it: "This pattern is intentional because X" or "We've addressed this project-wide in Y". CodeRabbit reads and learns from comments.
- **Human merges**: Never merge automatically
- **Avoid false urgency**: If just waiting for review, say so
