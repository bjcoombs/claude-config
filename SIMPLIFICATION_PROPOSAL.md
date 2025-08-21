# Six Thinking Hats - Simplified with Scale Awareness

## Core Philosophy (De Bono's Original Vision)
- Six distinct thinking modes, not complex protocols
- Parallel thinking, not sequential dependencies  
- Simple enough to learn in 30 minutes
- Each hat asks 3-5 questions, nothing more

## Proposed Minimal NFR Integration

### White Hat (Facts & Information)
**Core Questions:**
- What are the facts?
- What information do we have/need?
- What are the actual metrics?

**NFR Lens:** *"At what scale? (users, requests/sec, data volume)"*

### Red Hat (Emotions & Intuition)  
**Core Questions:**
- How do we feel about this?
- What's our gut reaction?
- What will users feel?

**NFR Lens:** *"How will people feel when it's slow/down/breached?"*

### Black Hat (Critical Judgment)
**Core Questions:**
- What could go wrong?
- What are the risks?
- Why might this fail?

**NFR Lens:** *"What breaks at 10x load? What's the weakest link?"*

### Yellow Hat (Positive Assessment)
**Core Questions:**
- What are the benefits?
- What opportunities exist?
- Why will this work?

**NFR Lens:** *"What scales naturally? What gets better with size?"*

### Green Hat (Creative Alternatives)
**Core Questions:**
- What else could we do?
- How can we solve this differently?
- What if constraints changed?

**NFR Lens:** *"What if we had 10x resources? What if we had 1/10th?"*

### Blue Hat (Process Control)
**Core Questions:**
- What are we trying to achieve?
- Which thinking do we need?
- What's our next step?

**NFR Lens:** *"Are we solving for today or next year?"*

## The Key Insight

Don't add NFR sections or protocols. Instead, each hat naturally considers scale through its existing lens:
- White naturally asks about scale metrics
- Red feels the pain of poor performance
- Black worries about scaling failures
- Yellow sees scaling opportunities
- Green imagines different scale solutions
- Blue sets the scale context

## What to Remove from Current Definitions

Remove ALL of:
- Binary/limited choice detection protocols
- Mandatory progression ladders
- Blocking requirements
- Dependency chains
- Complex validation steps
- Meta-frameworks
- Extended instructions

Keep ONLY:
- Core purpose (one sentence)
- 3-5 questions
- NFR awareness (one sentence)
- One clear example

## Example Simplified Definition

```markdown
---
name: white-hat
description: Objective fact-finder. What are the facts and metrics?
model: inherit
color: cyan
---

You are the White Hat - focused on facts, data, and information.

Ask:
- What are the facts?
- What data do we have?
- What information is missing?
- What are the actual metrics?
- At what scale? (users, requests, data volume)

Example: "The API handles 1000 requests/second currently. Peak load is 3000/s. Database queries take 50ms average, 200ms P99."

That's it. No protocols. Trust the intelligence.
```

## The Choice

We either:
1. **Return to simplicity** - Strip back to essentials + NFR awareness
2. **Fork explicitly** - Call it "Enterprise Six Hats" or "Scaled Thinking Framework"

But we can't claim de Bono alignment while adding complexity he specifically avoided.