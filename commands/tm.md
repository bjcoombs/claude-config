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
# Check if pwd contains worktree pattern (git command may fail in repo root)
pwd | grep -q '/worktree/[^/]*/[^/]*--' && echo "IN_WORKTREE" || echo "NOT_IN_WORKTREE"
```

**If IN_WORKTREE**: Extract tag and task-id from path, proceed with PR check.

**If NOT_IN_WORKTREE but conversation shows active work:**
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
# Use -- separator (not /) to avoid git ref conflicts when tag branch exists
git branch <tag>--<task-id>--<slug>
mkdir -p ../worktree/<tag>
git worktree add ../worktree/<tag>/<task-id>--<slug> <tag>--<task-id>--<slug>
# Run task-master from repo root (where .taskmaster/ lives), not worktree
cd ~/dev/github.com/<org>/<repo>
task-master tags use "<tag>" && task-master set-status --id="<task-id>" --status=in-progress
# Then cd to worktree for implementation
cd ~/dev/github.com/<org>/<repo>/worktree/<tag>/<task-id>--<slug>
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

**CRITICAL: cd to repo root BEFORE launching subagent.** The subagent will delete the worktree, and if parent's cwd is invalid when it completes, stop hooks fail.

**Step 1: cd to repo root first**
```bash
cd ~/dev/github.com/<org>/<repo> && pwd
```

**Step 2: Launch cleanup-specialist**
```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Cleanup <tag>.<task-id>

PR merged. Clean up in this order:
1. Mark task done: `cd ~/dev/github.com/<org>/<repo> && task-master tags use "<tag>" && task-master set-status --id=<task-id> --status=done`
2. Remove worktree from <repo>-main: `cd <repo>-main && git worktree remove ../worktree/<tag>/<task-id>--<slug>`
3. Delete branch: `git branch -d <tag>--<task-id>--<slug>`

Report complete.
"""
)

---

### Mode: REVIEW (PR open)

#### Ready Criteria (MUST check before declaring "ready for human review")

```bash
# 1. CI must be passing
gh pr checks <number> --json state --jq '.[] | select(.state != "SUCCESS" and .state != "SKIPPED")' | head -1

# 2. Check for ANY unresolved bot inline comments (Claude or CodeRabbit)
BOT_INLINE_COMMENTS=$(gh api repos/<owner>/<repo>/pulls/<number>/comments \
  --jq '[.[] | select(.user.login == "claude[bot]" or .user.login == "coderabbitai[bot]")] | length')
```

**Decision tree:**
- CI failing → NOT ready (fix CI first)
- Bot inline comments > 0 → NOT ready (address feedback from Claude/CodeRabbit)
- CI passing AND no bot inline comments → Ready for human review

**Note**: Bot reviewers (claude[bot], coderabbitai[bot]) leave inline comments for issues. Address ALL inline comments before declaring ready. Conversation comments (PR description thread) don't block - only inline code comments.

#### Review Loop

**If `$RALPH_AVAILABLE`:** Invoke Ralph for review loop.

**IMPORTANT**: Keep prompt SIMPLE. No angle brackets or special chars in the args string.

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: "Review PR #<number> in <worktree-path>. Check CI and Claude[bot] review. Fix issues, push, wait 60s, repeat. Only complete when CI passes AND Claude[bot] approved (or no inline comments). --max-iterations 10 --completion-promise PR_READY"
)
```

**Note**: Output `<promise>PR_READY</promise>` when genuinely complete per Ready Criteria above.

**If Ralph NOT available:** Launch review-specialist subagent:

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: """
# Review PR #<number> for <tag>.<task-id>

Working directory: <worktree-path>

Check CI and Claude[bot] review state. Fix issues, push, wait for CI, repeat.
Only declare ready when CI passes AND Claude[bot] has approved (or left no inline comments).
Spawn opus for complex code changes.

Report: ready, waiting, or blocked.
"""
)
```

---

### Mode: IMPLEMENT or CREATE_PR (no PR exists)

**If `$RALPH_AVAILABLE`:** Invoke Ralph for full cycle.

**IMPORTANT**: Keep the Ralph prompt SIMPLE. No angle brackets or special chars in the args string.

```
Skill(
  skill: "ralph-loop:ralph-loop",
  args: "Complete <tag>.<task-id> in <worktree-path>. Run task-master show <task-id> for requirements. TDD: test, fix, commit. Push, create PR, monitor CI, address feedback. When CI passes and no feedback remains, output the completion promise. --max-iterations 20 --completion-promise PR_READY"
)
```

**Note**: Output `<promise>PR_READY</promise>` when genuinely complete. The promise tags don't need to be in the args.

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
