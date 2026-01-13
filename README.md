# Claude Config

Personal Claude Code configuration, customizations, and workflow tools.

## Contents

### Task Master Commands

Workflow commands for managing tasks with [Task Master](https://github.com/eyecuelab/task-master-ai):

| Command | Description |
|---------|-------------|
| `/tm` | Unified context-aware command - starts, reviews, or cleans up tasks based on current state |
| `/tms` | (Deprecated) Use `/tm` instead |
| `/tmr` | (Deprecated) Use `/tm` instead |
| `/tms-legacy` | Legacy monolithic start command |

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

## Repository Structure

```
claude-config/
├── README.md
├── CLAUDE.md           # Personal guidelines and instructions
├── commands/
│   ├── tm.md           # Task Master unified command
│   ├── tms.md          # (deprecated)
│   ├── tmr.md          # (deprecated)
│   ├── tms-legacy.md
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

## License

The agent implementations are provided as-is for use with Claude Code. The Six Thinking Hats method is the intellectual property of Edward de Bono.
