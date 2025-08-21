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

## Installation

Place these agent files in your Claude Code agents folder:

```bash
# Clone or copy the agents to your Claude Code directory
cp *.md ~/.claude/agents/
```

Once installed, the agents are available through Claude Code's Task system.

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

### Agent Invocation
```python
# For comprehensive analysis, use Blue Hat to orchestrate all perspectives
Task(subagent_type="blue-hat", prompt="Should we migrate our database to PostgreSQL?")

# Or call specific hats directly for targeted analysis
Task(subagent_type="white-hat", prompt="Analyze current database performance metrics")
Task(subagent_type="black-hat", prompt="What could go wrong with this migration?")
```

### Typical Workflow
1. Blue Hat receives the problem
2. Determines optimal thinking sequence based on problem type
3. Spawns appropriate hat agents (often in parallel)
4. Each hat provides its specialized perspective
5. Blue Hat synthesizes findings into actionable insights
6. Scribe creates documentation if needed

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