# Six Thinking Hats Analysis

Perform a comprehensive analysis using Edward de Bono's Six Thinking Hats methodology.

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

## Instructions for Claude:

**CRITICAL ARCHITECTURAL NOTE**: Agents cannot call other agents in Claude Code. This command acts as the orchestrator, calling each hat directly. Blue Hat provides synthesis, not orchestration.

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
   - Create a TodoWrite list defining which hats to use
   - Call each thinking hat agent using Task(subagent_type="hat-name")
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
   - This command creates todo list reflecting the chosen sequence
   - This command spawns thinking hats (parallel where possible)
   - Each hat provides its perspective
   - Blue Hat synthesizes findings (cannot call other agents)
   - Scribe documents any needed issues/tickets

## Usage:
`/6hats [topic or problem to analyze]`

## Example:
`/6hats database query performance issue causing timeouts`

## No Arguments Behavior:
If called without arguments, respond with:
"What topic or problem would you like me to analyze using the Six Thinking Hats methodology?"