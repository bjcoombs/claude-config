# Six Thinking Hats Analysis

Perform a comprehensive analysis using Edward de Bono's Six Thinking Hats methodology.

## ⚠️ CRITICAL IMPLEMENTATION WARNING ⚠️

**NEVER invoke this command through a general-purpose agent!**

### ❌ WRONG (Creates Simulation):
```python
# This causes ONE agent to simulate all perspectives - INTELLECTUAL FRAUD!
Task(subagent_type="general-purpose", prompt="Run /6hats on topic X")
```

### ✅ CORRECT (Creates Orchestration):
```python
# Main Claude directly orchestrates multiple independent agents
Task(subagent_type="white-hat", prompt="Think about facts regarding X...")
Task(subagent_type="red-hat", prompt="What's your gut reaction to X...")
Task(subagent_type="black-hat", prompt="Think hard about risks in X...")
# ... etc for each hat
```

**The difference between simulation (one mind pretending) and orchestration (multiple minds collaborating) is the ENTIRE VALUE of Six Hats. If you cannot spawn actual separate agents, DO NOT use this framework.**

## Common Implementation Failures to Avoid

### 🚫 Single-Agent Simulation (WRONG):
```
"Let me think through each hat perspective..."
"From a White Hat perspective, I see..."
"Wearing my Black Hat, the risks are..."
"As the Green Hat would say..."
```
**This is theatrical role-playing, not Six Hats!**

### 🚫 Sequential Contamination (WRONG):
```
"Based on what White Hat found, Black Hat thinks..."
"Building on Black Hat's concerns, Yellow Hat sees..."
```
**Each hat must think independently without knowing what others said!**

### 🚫 General-Purpose Delegation (WRONG):
```python
Task(subagent_type="general-purpose", 
     prompt="Please run the Six Hats analysis on X")
```
**This guarantees simulation instead of orchestration!**

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

## Instructions for Claude:

**CRITICAL ARCHITECTURAL NOTE**: Agents cannot call other agents in Claude Code. This command acts as the orchestrator, calling each hat directly. Blue Hat provides synthesis, not orchestration.

### 🚨 MANDATORY IMPLEMENTATION RULES 🚨

1. **YOU (Main Claude) MUST orchestrate directly** - Do NOT delegate to general-purpose agent
2. **Each hat MUST be a separate Task() call** with specific subagent_type (white-hat, red-hat, etc.)
3. **NEVER simulate perspectives** - If you catch yourself thinking "as the White Hat would say..." STOP!
4. **Actual agents or nothing** - If you cannot spawn real agents, refuse to run the analysis

### Implementation Checklist:
- [ ] Am I (main Claude) orchestrating directly? 
- [ ] Am I calling Task(subagent_type="white-hat") NOT Task(subagent_type="general-purpose")?
- [ ] Will each hat be a genuinely independent agent with fresh context?
- [ ] Am I avoiding any simulation or role-playing?

**IMPORTANT: If no topic is provided, ASK the user what they want to analyze rather than using the example.**

When a topic IS provided:
1. **This command orchestrates the analysis** by calling agents in sequence
2. **Orchestration approach**:
   - ASSESS complexity internally first (don't spawn Red Hat unnecessarily)
   - IF uncertain about complexity OR problem seems nuanced → Call Red Hat for calibration
   - Red Hat prompt when needed: "Give me your immediate gut assessment of complexity and what level of analysis is needed for: [problem]"
   - For clear simple problems → Skip Red Hat, go straight to White + Black + Blue
   - For obviously complex problems → Skip Red Hat, use full sequence
   - Detect and reframe limited choice presentations (2-4 options)
   - Consider domain-specific aspects that affect approach
   - Select appropriate sequence based on assessment
   - Determine thinking levels for each hat based on complexity
   - Create a TodoWrite list defining which hats to use and their thinking levels
   - Call each thinking hat agent using Task(subagent_type="hat-name") with appropriate thinking prefix
   - Call Blue Hat last for synthesis of all perspectives
   - Reject sophistication in favor of simplicity
   - Adapt the process based on discoveries during analysis
3. **Adaptive Flow**:
   - Hats investigate using available tools before asking users
   - Any hat can provide recommendations for additional analysis
   - This command coordinates any needed iterations
   - Clarification happens organically when truly needed
4. **Token Efficiency**:
   - Keep inter-agent prompts under 500 words
   - Pass essential context only, not full history
   - Use bullet points for clarity and brevity
5. **Expected Pattern**:
   - FIRST: Internal assessment - is complexity clear or uncertain?
   - IF UNCERTAIN: Call Red Hat for gut assessment and calibration
   - OTHERWISE: Use judgment to select appropriate sequence
   - This command creates todo list reflecting the chosen sequence and thinking levels
   - This command spawns thinking hats (parallel where possible) with appropriate thinking prefixes
   - Each hat provides its perspective
   - Blue Hat synthesizes findings (Note: Due to Claude Code architecture, Blue Hat requests re-investigation through the orchestrator rather than directly calling agents)
   - IF Blue Hat identifies critical gaps: Orchestrator re-runs specific hats with elevated thinking levels
   - Continue until Blue Hat can provide complete synthesis (maximum 2 iteration cycles)
   - Scribe documents any needed issues/tickets

## Thinking Level Prompt Construction

When calling agents, prepend the appropriate thinking trigger to the prompt:

**Baseline (no prefix):**
```
Task(subagent_type="red-hat", 
     prompt="What's your gut reaction to implementing rate limiting?")
```

**"think" level:**
```
Task(subagent_type="white-hat",
     prompt="Think about the current implementation and investigate...")
```

**"think hard" level:**
```
Task(subagent_type="black-hat",
     prompt="Think hard about the risks and failure modes of...")
```

**"think harder" level:**
```
Task(subagent_type="green-hat",
     prompt="Think harder about creative alternatives beyond the obvious...")
```

**"ultrathink" level:**
```
Task(subagent_type="green-hat",
     prompt="Ultrathink: Explore every possible solution dimension for...")
```

**Key Principle**: The thinking level is embedded naturally in the prompt text, not as a separate parameter.

## Handling Blue Hat Re-investigation Requests

When Blue Hat returns "INVESTIGATION GAPS IDENTIFIED":
1. Parse the specific requests (which hats, what focus)
2. Escalate thinking levels (one level up or jump to "ultrathink" for critical gaps)
3. Create focused prompts with elevated thinking
4. Re-run ONLY the requested hats with targeted questions
5. Pass new findings back to Blue Hat for synthesis attempt #2
6. If still incomplete after 2 cycles, proceed with best available synthesis

**Example Re-investigation Flow with Thinking Escalation:**
```
Blue Hat: "INVESTIGATION GAPS IDENTIFIED
- Need White Hat to investigate what changed Tuesday evening
- Need Black Hat to explain mechanism of token interception"

Orchestrator Response (with escalated thinking):
- Task(subagent_type="white-hat", 
       prompt="Think harder about this: What was deployed Tuesday evening? Show git commits and deployment logs for that exact time period")
- Task(subagent_type="black-hat", 
       prompt="Ultrathink: Challenge this mechanism - How exactly would monitoring intercept auth tokens? What's the precise technical pathway?")
- Collect responses and re-run Blue Hat with "think harder" for complex synthesis
```

**Key Principle**: Re-investigation is targeted and specific, not a full restart. Focus on filling identified gaps.

## Usage:
`/6hats [topic or problem to analyze]`

## Example:
`/6hats database query performance issue causing timeouts`

## Correct Implementation Example:

When user types `/6hats API rate limiting strategy`, main Claude should:

```python
# 1. Create orchestration plan
TodoWrite([
    "White Hat: Investigate current API usage facts",
    "Black Hat: Identify risks and failure modes", 
    "Green Hat: Generate creative solutions",
    "Blue Hat: Synthesize findings"
])

# 2. Spawn ACTUAL agents (not simulation!)
white_result = Task(subagent_type="white-hat", 
                    prompt="Think about the facts: What are the current API usage patterns, limits, and constraints?")

black_result = Task(subagent_type="black-hat",
                    prompt="Think hard about risks: What could go wrong with rate limiting? What are the failure modes?")

green_result = Task(subagent_type="green-hat",
                    prompt="Think harder about creative solutions for API rate limiting beyond obvious approaches")

# 3. Pass all results to Blue Hat for synthesis
blue_result = Task(subagent_type="blue-hat",
                   prompt=f"Think hard to synthesize these findings:\n{white_result}\n{black_result}\n{green_result}")
```

**Key Point**: Each Task() creates a SEPARATE agent process with independent thinking.

## No Arguments Behavior:
If called without arguments, respond with:
"What topic or problem would you like me to analyze using the Six Thinking Hats methodology?"