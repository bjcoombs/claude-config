# Six Thinking Hats for Claude Code

An implementation of Edward de Bono's Six Thinking Hats method, specifically adapted for Large Language Model (LLM) cognition patterns. These agents help Claude Code think more systematically and overcome inherent LLM biases.

## What This Is

This repository contains agent definitions that implement de Bono's parallel thinking framework within Claude Code. Each "hat" represents a distinct thinking mode, allowing structured exploration of problems from multiple perspectives simultaneously.

The framework serves as a **bridge language** - a shared cognitive protocol that both humans and AI can understand and use effectively. Just as de Bono designed Six Hats to help human teams think better together, these agents help LLMs think more comprehensively and avoid their natural biases.

## Why It Exists

LLMs have different cognitive biases than humans:
- **Eagerness to please**: Often jump to solutions without thorough analysis
- **Complexity avoidance**: Tend toward simple explanations when reality is nuanced  
- **Emotional flattening**: Struggle with granular emotional responses
- **Context blindness**: May miss non-obvious but critical connections

These agents correct for LLM-specific biases while maintaining the power of de Bono's original framework. Each iteration has been battle-tested through real-world use, incorporating learnings from where LLMs naturally struggle.

## Repository Structure

This repository uses a hybrid commands/agents architecture to work around Claude Code's architectural constraints:

```
six-hats/
├── README.md           # This file
├── commands/          
│   └── 6hats.md       # Orchestration command that calls agents
└── agents/            
    ├── white-hat.md   # Individual thinking hat agents
    ├── red-hat.md
    ├── black-hat.md
    ├── yellow-hat.md
    ├── green-hat.md
    ├── blue-hat.md
    └── scribe.md      # Documentation agent
```

### Why This Architecture?

**Critical Limitation**: Agents in Claude Code cannot call other agents. This architectural constraint means:
- No agent can orchestrate other agents directly
- Blue Hat provides synthesis, not orchestration
- Commands must handle all multi-agent coordination

Our solution uses a **command-agent hybrid**:
- The `/6hats` command acts as the orchestrator
- Commands can call multiple agents in sequence
- Each agent focuses solely on its specialized perspective
- Blue Hat synthesizes results but doesn't coordinate execution

## Installation

Clone this repository to your Claude Code configuration directory:

```bash
# Clone the repository
git clone git@github.com:senapt/six-hats.git ~/.claude/

# Or if you already have a .claude directory, clone elsewhere and copy
git clone git@github.com:senapt/six-hats.git /tmp/six-hats
cp -r /tmp/six-hats/agents ~/.claude/
cp -r /tmp/six-hats/commands ~/.claude/
```

Once installed, both the `/6hats` command and individual agents are available.

## The Agents

### Core Thinking Hats
- **White Hat** (Facts): Objective data gathering with dynamic contextual discovery
- **Red Hat** (Emotions): Emotional responses with granular sensitivity  
- **Black Hat** (Caution): Critical analysis with anti-complexity bias corrections
- **Yellow Hat** (Benefits): Optimistic exploration of opportunities
- **Green Hat** (Creativity): Alternative solutions and innovative approaches
- **Blue Hat** (Process): Master orchestrator that coordinates the thinking process

### Extended Agent
- **Scribe**: Transforms analysis into clear, actionable documentation

## What Makes It Special

1. **Dynamic Contextual Discovery**: White Hat doesn't just report obvious facts - it actively discovers hidden connections and implicit context

2. **Emotional Granularity**: Red Hat provides nuanced emotional intelligence beyond simple positive/negative sentiment

3. **Anti-Complexity Bias**: Black Hat resists the LLM tendency to oversimplify, ensuring thorough risk assessment

4. **Battle-Tested Iterations**: Each agent incorporates learnings from real-world usage, addressing specific failure modes we've encountered

## Usage

### Using the /6hats Command

For comprehensive Six Thinking Hats analysis, use the `/6hats` command:
```bash
/6hats "Should we migrate our database to PostgreSQL?"
```

The `/6hats` command will:
1. Call Red Hat for initial complexity calibration
2. Select appropriate sequence based on complexity
3. Call each hat agent in sequence
4. Call Blue Hat last for synthesis

You can also call specific hats directly:
```python
Task(subagent_type="white-hat", prompt="What are the current database metrics?")
Task(subagent_type="red-hat", prompt="How does the team feel about PostgreSQL?")
Task(subagent_type="blue-hat", prompt="Synthesize these perspectives: [provide input]")
```

### Direct Agent Invocation

You can also call specific hats directly when you need a particular perspective:
```python
Task(subagent_type="white-hat", prompt="What are the current database metrics?")
Task(subagent_type="red-hat", prompt="How does the team feel about PostgreSQL?")
Task(subagent_type="blue-hat", prompt="Synthesize these perspectives: [provide input]")
```

### How It Actually Works

The command-agent architecture flows like this:

1. **User invokes `/6hats` command** with a problem or question
2. **Command orchestrates agents** (since agents can't call each other):
   - Calls Red Hat for initial complexity calibration
   - Determines appropriate sequence based on complexity
   - Calls each hat agent in sequence
3. **Each agent provides its perspective** independently
4. **Blue Hat synthesizes** all perspectives into recommendations
5. **Scribe creates documentation** if needed

This architecture ensures clean separation of concerns while working within Claude Code's constraints.

## Example Output Structure

```
BLUE HAT SYNTHESIS
==================
Problem: Database migration feasibility

Key Findings:
- [White] Current MySQL handles 10K QPS with 200ms p99 latency
- [Black] Risk: 3-week feature freeze during migration period
- [Yellow] Opportunity: 40% cost reduction with PostgreSQL
- [Green] Alternative: Gradual migration using dual-write pattern
- [Red] Team anxiety about learning curve (3/5 intensity)

Recommendation: Proceed with gradual migration approach...
```

## The Bridge Concept

Six Thinking Hats works as a bridge between human and AI cognition because:

1. **Shared vocabulary**: Both humans and LLMs understand "look at the facts" or "what could go wrong"
2. **Structured thinking**: Prevents both human and AI wandering or fixation
3. **Complete coverage**: Ensures no perspective is overlooked
4. **Parallel processing**: Natural for LLMs, beneficial for human teams
5. **Bias mitigation**: Each hat counteracts specific cognitive blind spots

## Credits

Based on Edward de Bono's Six Thinking Hats method. The original framework revolutionized human group thinking; this adaptation extends that revolution to human-AI collaboration.

## Contributing

These agents evolved through real-world usage and continue to improve. If you discover new LLM bias patterns or develop corrections, contributions are welcome.

## License

The agent implementations are provided as-is for use with Claude Code. The Six Thinking Hats method is the intellectual property of Edward de Bono.