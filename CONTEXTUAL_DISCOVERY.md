# Contextual Discovery Pattern for Six Thinking Hats

## The Breakthrough Pattern

Just as Red Hat discovers actors dynamically, ALL hats should discover relevant considerations dynamically.

## The Universal Meta-Question

Each hat asks first: **"What dimensions matter in THIS specific context?"**

## Implementation Per Hat

### White Hat - Contextual Facts Discovery
Before investigating facts, ask:
- "What measurements matter in this domain?"
- "What compliance/regulatory facts apply here?"
- "What domain-specific data should I gather?"

Example: For a payment system, discovers PCI compliance matters. For healthcare, discovers HIPAA matters. For games, discovers player retention metrics matter.

### Red Hat - Already Implemented
Current actor discovery already does this - discovers WHO matters before channeling their emotions.

### Black Hat - Contextual Risk Discovery  
Before identifying risks, ask:
- "What failure modes are specific to this domain?"
- "What risks would an expert in this field worry about?"
- "What goes wrong in this type of system?"

Example: In finance, discovers reversibility risks. In healthcare, discovers patient safety risks. In social media, discovers content moderation risks.

### Yellow Hat - Contextual Value Discovery
Before identifying benefits, ask:
- "What does success look like in this context?"
- "What do stakeholders in this domain value?"
- "What opportunities are unique to this situation?"

Example: In education, discovers learning outcome improvements. In climate tech, discovers carbon reduction. In gaming, discovers player engagement.

### Green Hat - Contextual Possibility Discovery
Before generating alternatives, ask:
- "What constraints are unique to this domain?"
- "What solutions work in similar contexts?"
- "What would experts in this field consider?"

Example: In regulated industries, discovers compliant alternatives. In startups, discovers lean alternatives. In enterprise, discovers scalable alternatives.

### Blue Hat - Contextual Priority Discovery
Before orchestrating, ask:
- "What matters most in this specific situation?"
- "What thinking is most critical for this domain?"
- "What order makes sense for this context?"

Example: In crisis, prioritizes Black (risks) first. In opportunity, prioritizes Yellow first. In innovation, prioritizes Green first.

## The Power of This Approach

1. **Handles Unknown Unknowns**: Discovers considerations we didn't know to list
2. **Domain Adaptive**: Works in healthcare, finance, gaming, climate, education, etc.
3. **Future Proof**: Adapts to new domains without framework changes
4. **Simple Addition**: One question per hat, not complex protocols

## Real Example

**Problem**: "Implement user authentication"

**Without Contextual Discovery**:
- White: "Need username/password storage"
- Black: "Could be brute forced"

**With Contextual Discovery**:
- White: "What matters in THIS context? This is a children's education app, so COPPA compliance matters. Also need parental consent flows."
- Black: "What risks in THIS domain? Children's data breaches have severe legal/reputation consequences. Predator access risks. Age verification challenges."

The framework automatically elevates its thinking to match the domain.

## The Meta-Learning Loop

Each use of the framework in a new domain teaches the LLM:
1. What questions to ask for that domain
2. What patterns emerge
3. What matters that wasn't obvious

Over time, the framework becomes wiser about discovering what it doesn't know.

## Minimal Implementation

Add to each hat's definition:

```markdown
Before applying your [white/black/yellow/green/blue] perspective, briefly consider:
"What aspects of this specific context might I be overlooking?"
```

That's it. One line that unlocks infinite adaptability.

## Why This Is The Missing Piece

- NFRs → Too specific to enterprise software
- Enterprise considerations → Too specific to business
- Contextual discovery → Works for EVERYTHING

This makes Six Hats truly universal while maintaining all our hard-won LLM bias corrections.