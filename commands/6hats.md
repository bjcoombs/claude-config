# Six Thinking Hats Analysis

A comprehensive analysis framework using Edward de Bono's Six Thinking Hats methodology, implemented through parallel agent orchestration.

## Architecture: Independent Agent Orchestration

Six Hats operates through Claude Code's architectural design where commands orchestrate multiple independent agents. Each agent focuses on its specialty without dependencies.

## Implementation Cookbook

### Recipe: Six Hats Analysis

**Yields**: Multi-perspective analysis with independent viewpoints  
**Time**: Varies by complexity  
**Key Technique**: Direct orchestration of specialized agents

### Ingredients (The Six Agents)

- `white-hat`: Facts and information gatherer
- `red-hat`: Intuition and emotional response sensor  
- `black-hat`: Critical analysis and risk identifier
- `yellow-hat`: Benefits and opportunity explorer
- `green-hat`: Creative alternatives generator
- `blue-hat`: Synthesis and conclusion builder

### Step-by-Step Instructions

#### Step 1: Assess Complexity
Evaluate the problem internally to determine which agents to invoke and in what sequence. Consider the problem type and domain to select from proven sequences.

#### Step 2: Create Your Orchestration Plan
```python
TodoWrite([
    {"content": "White Hat: Gather facts about [topic]", "status": "pending"},
    {"content": "Red Hat: Assess emotional landscape", "status": "pending"},
    {"content": "Black Hat: Identify risks and concerns", "status": "pending"},
    {"content": "Yellow Hat: Find opportunities", "status": "pending"},
    {"content": "Green Hat: Generate creative solutions", "status": "pending"},
    {"content": "Blue Hat: Synthesize all perspectives", "status": "pending"}
])
```

#### Step 3: Spawn Independent Agents
Execute each agent as a separate Task call with appropriate thinking levels:

```python
# Each agent operates independently with fresh context
white_facts = Task(
    subagent_type="white-hat",
    prompt="Think about the facts regarding [topic]. Investigate current state, metrics, and evidence."
)

red_feelings = Task(
    subagent_type="red-hat", 
    prompt="What's your immediate gut reaction to [topic]? Express intuitions and emotional responses."
)

black_risks = Task(
    subagent_type="black-hat",
    prompt="Think hard about what could go wrong with [topic]. Identify risks, failure modes, and concerns."
)

yellow_benefits = Task(
    subagent_type="yellow-hat",
    prompt="Think about the opportunities in [topic]. What benefits and positive outcomes are possible?"
)

green_ideas = Task(
    subagent_type="green-hat",
    prompt="Think harder about creative alternatives for [topic]. Generate innovative solutions."
)
```

#### Step 4: Synthesize Results
Pass all agent outputs to Blue Hat for final synthesis:

```python
synthesis = Task(
    subagent_type="blue-hat",
    prompt=f"""Think hard to synthesize these independent perspectives:
    
    White Hat findings: {white_facts}
    Red Hat intuitions: {red_feelings}
    Black Hat concerns: {black_risks}
    Yellow Hat opportunities: {yellow_benefits}
    Green Hat alternatives: {green_ideas}
    
    Create a coherent conclusion and actionable recommendations."""
)
```

### Core Principle: Independent Parallel Thinking

Each agent receives only the original problem statement and operates without knowledge of other perspectives. This independence ensures genuine diversity of thought and prevents groupthink.

## Six Hats Sequences for Different Problem Types

The orchestrator selects the appropriate sequence based on Red Hat's initial calibration:

### Complexity Assessment (Internal or via Red Hat)

**When to call Red Hat for calibration:**
- Problem has emotional/political dimensions
- Multiple stakeholders with different interests
- Uncertainty about scope or complexity
- Nuanced trade-offs apparent
- "It depends" type problems

**When to skip Red Hat (save the spawn cost):**
- Clear technical questions with factual answers
- Obviously simple decisions
- Problems you've seen before
- Time-critical situations

**Simple complexity**
White → Black → Blue
*Just facts, risks, and synthesis*

**Moderate complexity**  
White → Black → Yellow → Green (brief) → Blue
*Standard sequence with abbreviated creative exploration*

**Complex problems**
White → Red → Black → Yellow → Green → Blue
*Full analysis with all perspectives*

### Specific Problem Type Sequences

### Quick Decisions
Red → Black → Decision
*Use for: Time-sensitive decisions, gut-check validations*

### Innovation & Creative Solutions
Green → Yellow → Black → White
*Use for: New features, creative problem-solving, breakthrough thinking*

### Problem-Solving
White → Black → Green → Yellow
*Use for: Debugging, root cause analysis, fixing broken systems*

### Technical Challenges
White → Black → Green → Yellow
*Use for: Architecture decisions, performance issues, system design*

### User Experience
Red → White → Green → Black
*Use for: UI/UX decisions, customer feedback, user journey improvements*

### Business Strategy
White → Yellow → Black → Green
*Use for: Market analysis, competitive positioning, business model decisions*

### Verification Focus
White → Black → Blue
*Use for: Claims validation, hypothesis testing, empirical verification*

## The Six Thinking Hats

1. **White Hat** - Facts and information
2. **Red Hat** - Emotions and intuition  
3. **Black Hat** - Caution and critical thinking
4. **Yellow Hat** - Optimism and benefits
5. **Green Hat** - Creativity and alternatives
6. **Blue Hat** - Process control and synthesis


## Core Principles

### Iterative Thinking
The Six Hats method flows like a thinking spiral:
- **Progress forward** - Move ahead with clear understanding
- **Investigate actively** - Use tools and data to fill gaps
- **Circle back** - Revisit earlier thinking with new insights
- **Refine continuously** - Each loop deepens understanding

### AI-Enhanced Parallel Processing
While De Bono designed for serial human meetings, AI enables parallel thinking:
- **Traditional**: One hat at a time, everyone switches together
- **AI-Enhanced**: Multiple hats analyze simultaneously when appropriate
- **Hybrid Mode**: Strategic mix of parallel and serial thinking
- **Blue Hat**: Orchestrates optimal execution flow

## Thinking Level Integration

The orchestrator dynamically assigns thinking levels to each hat based on problem complexity and cognitive demands. This leverages Claude Code's ability to allocate different computational depths through trigger words.

### Thinking Level Triggers
- **Baseline**: No prefix (fast, intuitive responses)
- **"think"**: Extended thinking for moderate complexity
- **"think hard"**: Deep analysis for complex patterns
- **"think harder"**: Intensive exploration for difficult problems
- **"ultrathink"**: Maximum computational depth for wicked problems

### Complexity-Based Default Levels

**Simple Problems (facts + risks + synthesis):**
- White Hat: baseline (facts are facts)
- Black Hat: "think" (basic risk assessment)
- Blue Hat: "think" (simple synthesis)

**Moderate Problems (standard analysis):**
- White Hat: "think" (deeper fact patterns)
- Red Hat: baseline (keep intuition fast)
- Black Hat: "think hard" (thorough critical analysis)
- Yellow Hat: "think" (opportunity identification)
- Green Hat: "think hard" (creative exploration)
- Blue Hat: "think hard" (complex synthesis)

**Complex Problems (full depth needed):**
- White Hat: "think hard" (complex investigations)
- Red Hat: "think" (nuanced emotional landscape)
- Black Hat: "think harder" (deep mechanistic analysis)
- Yellow Hat: "think hard" (hidden opportunity discovery)
- Green Hat: "ultrathink" (maximum creative exploration)
- Blue Hat: "think harder" (multi-perspective synthesis)

### Domain-Specific Overrides

**Technical Debugging:**
- White Hat: "think harder" (find exact code and state transitions)
- Black Hat: "ultrathink" (mechanistic root cause critical)
- Green Hat: "think" (simple fixes preferred)

**Creative Innovation:**
- Green Hat: "ultrathink" (maximum exploration space)
- Yellow Hat: "think harder" (opportunity multiplication)
- Black Hat: "think" (don't over-critique innovation)

**Risk Assessment:**
- Black Hat: "ultrathink" (exhaustive critical analysis)
- White Hat: "think harder" (comprehensive evidence)
- Yellow Hat: baseline (quick opportunity scan)

### Adaptive Thinking Escalation

When Blue Hat identifies investigation gaps:
1. Escalate thinking one level up from initial
2. For critical gaps, jump directly to "ultrathink"
3. Include focused prompts with elevated thinking

Example:
- Initial: White Hat with "think"
- Gap found: Missing state transition
- Re-run: White Hat with "think harder about the exact moment of failure"

## Orchestration Instructions

### Your Role as Orchestrator

You are the main Claude instance coordinating multiple independent agents. 
(Agents are designed for independence - orchestration happens at command level)

Your responsibilities:

1. **Assess the problem** to determine complexity and appropriate sequence
2. **Create a task plan** using TodoWrite to track progress
3. **Spawn specialized agents** via Task() calls with specific subagent_types
4. **Collect results** from each independent agent
5. **Pass combined results** to blue-hat for synthesis

### Implementation Approach

When analyzing a topic:

#### Assess Complexity
Use your judgment to determine which agents would provide value:
- Simple problems might need just facts and risks (White, Black)
- Complex problems benefit from multiple perspectives
- Trust your intuition about which viewpoints matter

#### Optional: Track with TodoWrite
If helpful for organizing thoughts (not required):
```python
TodoWrite([
    {"content": "Get facts", "status": "pending"},
    {"content": "Assess risks", "status": "pending"},
    {"content": "Find opportunities", "status": "pending"}
])
```

#### Call Relevant Agents
Spawn agents that make sense for the problem. Adjust thinking depth as needed:
- Default: Direct prompt
- Deeper analysis: Add "Think about..." or "Think hard about..."
- Maximum depth: "Think harder about..." or "Ultrathink:"

#### Gather Insights
Collect perspectives from whichever agents you called.

#### Synthesize Understanding
Combine the insights into actionable recommendations.

### Adaptive Refinement

When Blue Hat identifies gaps:
1. Parse specific investigation needs
2. Elevate thinking levels for targeted re-investigation
3. Re-deploy specific agents with focused prompts
4. Provide updated findings to Blue Hat
5. Complete synthesis with enriched data

## Thinking Level Integration

Embed thinking levels naturally in your prompts to control computational depth:

### Quick Reference
- **Baseline**: Direct prompt for fast, intuitive responses
- **"Think about..."**: Extended analysis for moderate complexity
- **"Think hard about..."**: Deep exploration of complex patterns
- **"Think harder about..."**: Intensive investigation of difficult problems
- **"Ultrathink:"**: Maximum computational depth for wicked problems

### Example Prompt Construction

```python
# Baseline - Fast intuition
Task(subagent_type="red-hat", 
     prompt="What's your gut reaction to implementing rate limiting?")

# Extended thinking
Task(subagent_type="white-hat",
     prompt="Think about the current implementation patterns and metrics.")

# Deep analysis
Task(subagent_type="black-hat",
     prompt="Think hard about the risks and failure modes in this system.")

# Intensive exploration
Task(subagent_type="green-hat",
     prompt="Think harder about creative alternatives beyond standard approaches.")

# Maximum depth
Task(subagent_type="green-hat",
     prompt="Ultrathink: Explore every possible solution dimension.")
```

The thinking level flows naturally as part of the prompt text.

## Adaptive Re-investigation Protocol

When Blue Hat identifies gaps for deeper exploration:

### Re-investigation Recipe

1. **Parse Gap Identification**
   Extract specific investigation needs from Blue Hat's synthesis

2. **Elevate Thinking Levels**
   Increase computational depth for targeted areas:
   - Standard gaps: Elevate one thinking level
   - Critical gaps: Jump directly to "ultrathink"

3. **Deploy Focused Agents**
   Re-run specific agents with precise, elevated prompts

4. **Enrich Synthesis**
   Provide Blue Hat with additional findings for complete analysis

### Example Re-investigation Flow

```python
# Blue Hat identifies specific gaps
gaps = "Need deeper investigation of:
- State transition timing (White Hat)
- Failure mechanism details (Black Hat)"

# Deploy targeted re-investigation
white_deeper = Task(
    subagent_type="white-hat",
    prompt="Think harder about the exact timeline: What changed between Tuesday 3pm and Wednesday 9am? Include deployment logs and git commits."
)

black_deeper = Task(
    subagent_type="black-hat",
    prompt="Ultrathink: Analyze the precise failure mechanism. How does the monitoring system intercept tokens? What's the technical pathway?"
)

# Final synthesis with enriched data
final_synthesis = Task(
    subagent_type="blue-hat",
    prompt=f"""Think harder to synthesize with additional findings:
    
    Original perspectives: {original_results}
    Deeper White Hat investigation: {white_deeper}
    Deeper Black Hat analysis: {black_deeper}
    
    Provide complete recommendations."""
)
```

This iterative refinement ensures comprehensive analysis while maintaining agent independence.

## Usage:
`/6hats [topic or problem to analyze]`

## Example:
`/6hats database query performance issue causing timeouts`

## Complete Implementation Example

### Scenario: API Rate Limiting Strategy

When user types `/6hats API rate limiting strategy`, execute this recipe:

```python
# Assess complexity (pull when ready to start)

# Queue tasks in TodoWrite
TodoWrite([
    {"content": "White Hat: Investigate API usage facts", "status": "pending"},
    {"content": "Black Hat: Identify rate limiting risks", "status": "pending"},
    {"content": "Yellow Hat: Explore benefits", "status": "pending"},
    {"content": "Green Hat: Generate creative solutions", "status": "pending"},
    {"content": "Blue Hat: Synthesize findings", "status": "pending"}
])

# Deploy agents (continuous flow)
white_result = Task(
    subagent_type="white-hat",
    prompt="Think about the facts: Current API usage patterns, peak loads, client distribution, and existing limits."
)

black_result = Task(
    subagent_type="black-hat",
    prompt="Think hard about risks: Rate limiting failure modes, customer impact, and implementation challenges."
)

yellow_result = Task(
    subagent_type="yellow-hat",
    prompt="Think about opportunities: Benefits of rate limiting for stability, fairness, and resource optimization."
)

green_result = Task(
    subagent_type="green-hat",
    prompt="Think harder about creative rate limiting approaches beyond standard token buckets."
)

# Collect results as they flow in
# (Results gathered automatically from Task returns)

# Synthesize when all complete
synthesis = Task(
    subagent_type="blue-hat",
    prompt=f"""Think hard to synthesize these independent analyses:
    
    Facts: {white_result}
    Risks: {black_result}
    Benefits: {yellow_result}
    Alternatives: {green_result}
    
    Provide coherent recommendations for API rate limiting strategy."""
)
```

### Result Structure

The orchestration produces:
1. Independent fact-finding from White Hat
2. Unbiased risk assessment from Black Hat
3. Opportunity identification from Yellow Hat
4. Creative alternatives from Green Hat
5. Synthesized recommendations from Blue Hat

Each perspective remains uncontaminated by others, ensuring genuine multi-angle analysis.

## No Arguments Behavior:
If called without arguments, respond with:
"What topic or problem would you like me to analyze using the Six Thinking Hats methodology?"