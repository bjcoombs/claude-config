# Claude Config

Personal Claude Code configuration, customizations, and workflow tools.

> **Note**: This setup is opinionated. See [Git Workflow](#git-workflow) below for details.

## Contents

### Task Master Commands

Workflow commands for managing tasks with [Task Master](https://github.com/eyaltoledano/claude-task-master):

| Command | Description |
|---------|-------------|
| `/tm` | Unified context-aware command - starts, reviews, or cleans up tasks based on current state |

The `/tm` command detects context automatically:
- **Not in worktree** → Start mode (begin next task or specified task)
- **In TM worktree** → Detects state and runs appropriate action (implement, create PR, review, cleanup)

### Six Thinking Hats Framework

An implementation of Edward de Bono's Six Thinking Hats method, adapted for LLM cognition patterns.

| Agent | Role |
|-------|------|
| `/6hats` | Orchestration command |
| `white-hat` | Facts and data |
| `red-hat` | Emotions and intuition |
| `black-hat` | Risks and critical analysis |
| `yellow-hat` | Benefits and opportunities |
| `green-hat` | Creative alternatives |
| `blue-hat` | Synthesis and process |
| `scribe` | Documentation |

#### Why Six Hats Beats Default Claude

LLMs have predictable failure modes: eagerness to please, premature convergence on solutions, and a tendency to validate whatever framing you provide. Six Hats forces structured disagreement with itself.

**The problem with asking Claude directly:**
> "Should we rewrite our monolith in microservices?"
>
> Claude: "Here are the benefits of microservices... here are some considerations... ultimately it depends on your context." *(Helpful but non-committal)*

**The same question through Six Hats:**
- **White Hat** investigates your actual codebase, deployment frequency, team size
- **Red Hat** surfaces the emotional driver: "We're mass optimising for a problem we don't have"
- **Black Hat** attacks the proposal: "Your team of 3 will spend 18 months building infrastructure instead of features"
- **Yellow Hat** finds the real opportunity: "The actual pain is deployment fear—that's solvable without microservices"
- **Green Hat** generates alternatives: "Feature flags + better testing + gradual module extraction"
- **Blue Hat** synthesizes: "Don't do it. Here's what to do instead."

The framework produces **opinionated recommendations** because each hat is designed to push hard in its direction. Black Hat doesn't hedge—it attacks. Green Hat doesn't list options—it invents. The synthesis emerges from genuine tension, not diplomatic balance.

**When to use Six Hats:**
- Architecture decisions with long-term consequences
- "Should we..." questions where you suspect you're asking the wrong question
- Situations with multiple stakeholders or competing concerns
- When you want pushback, not validation
- Post-mortems and root cause analysis

**When default Claude is fine:**
- Implementation questions with clear requirements
- Debugging with specific error messages
- Code review of well-scoped changes
- Documentation and explanation tasks

## Repository Structure

```
claude-config/
├── README.md
├── CLAUDE.md           # Personal guidelines and instructions
├── commands/
│   ├── tm.md           # Task Master unified command
│   ├── 6hats.md        # Six Hats orchestration
│   └── ...
└── agents/
    ├── white-hat.md
    ├── red-hat.md
    ├── black-hat.md
    ├── yellow-hat.md
    ├── green-hat.md
    ├── blue-hat.md
    └── scribe.md
```

## Installation

Clone to your Claude Code configuration directory:

```bash
git clone git@github.com:bjcoombs/claude-config.git ~/.claude/
```

Or if you already have a `.claude` directory:

```bash
git clone git@github.com:bjcoombs/claude-config.git /tmp/claude-config
cp -r /tmp/claude-config/agents ~/.claude/
cp -r /tmp/claude-config/commands ~/.claude/
```

## Git Workflow

This setup assumes a specific directory structure using git worktrees. The key principle: **never work directly on the default branch**.

### Directory Structure

```
~/dev/github.com/<org>/<repo>/
├── <repo>-main/                    # SACRED - always on default branch, never modified
└── worktree/
    ├── <tag>/                      # Task Master tag folder (nested)
    │   ├── 1--create-schema/       # Task worktree
    │   └── 2--add-api/             # Another task worktree
    └── fix-login-bug/              # Non-TM worktree (flat)
```

### Why This Structure?

1. **`<repo>-main/` is sacred**: Always stays on `develop`/`main`, always clean. This gives you a pristine reference point and makes creating new branches reliable.

2. **All work in worktrees**: Every task gets its own worktree. No branch switching, no stashing, no conflicts between tasks.

3. **Parallel work**: Multiple worktrees = multiple tasks in progress simultaneously in different terminal windows.

### Creating a Worktree

```bash
# Start from the sacred directory
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git checkout develop && git pull origin develop

# Create branch and worktree
git branch fix-login-bug
git worktree add ../worktree/fix-login-bug fix-login-bug

# Work there
cd ../worktree/fix-login-bug
# ... make changes, commit, push, create PR ...

# Cleanup after PR merges
cd ~/dev/github.com/<org>/<repo>/<repo>-main
git worktree remove ../worktree/fix-login-bug
git branch -d fix-login-bug
```

### Task Master Worktrees

When using Task Master, worktrees are nested by tag:

```bash
# Task Master creates this structure automatically via /tm
worktree/458-feature/1.1--create-schema
worktree/458-feature/1.2--add-validation
worktree/458-feature/2--write-tests
```

The `/tm` command handles worktree creation and cleanup automatically based on task state.

### Adapting for Your Workflow

If you prefer a different structure, you'll need to modify:
- `commands/tm.md` - the path patterns and worktree creation logic
- Your `CLAUDE.md` - any references to the directory structure

## License

The agent implementations are provided as-is for use with Claude Code.

- The Six Thinking Hats method is the intellectual property of Edward de Bono.
- The Task Master commands are designed for use with [Claude Task Master](https://github.com/eyaltoledano/claude-task-master) by Eyal Toledano.
