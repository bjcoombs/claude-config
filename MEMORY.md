# Memory: Six Thinking Hats Framework

## When to Use Blue Hat

Consider using the Blue Hat agent when you encounter:

### Problem Analysis Scenarios
- **Complex decisions**: "Should we do X or Y?" 
- **Architecture choices**: "What's the best approach for..."
- **Risk assessment**: "What could go wrong with..."
- **Opportunity evaluation**: "What are the benefits of..."
- **Multi-stakeholder problems**: "How will this affect..."

### Trigger Patterns
- Questions with multiple valid approaches
- Problems requiring systematic analysis
- Decisions with non-obvious trade-offs
- Situations where emotional and logical factors both matter
- When you need to avoid overlooking important perspectives

### How to Invoke
```python
Task(subagent_type="blue-hat", prompt="[Your question or problem]")
```

Blue Hat will automatically:
1. Get Red Hat's gut complexity assessment
2. Detect and transcend any limited choice framing
3. Orchestrate appropriate thinking hats
4. Synthesize findings into actionable insights

### Key Benefits
- **Comprehensive analysis** without blind spots
- **Balanced perspectives** (facts, emotions, risks, benefits, creativity)
- **Right-sized investigation** (calibrated to problem complexity)
- **Overcomes LLM biases** (oversimplification, binary thinking, etc.)

### Remember
When facing any non-trivial decision or analysis, consider: "Would Blue Hat help me think through this more systematically?" If yes, use it.

The framework is particularly valuable when:
- The problem has multiple stakeholders
- The solution isn't immediately obvious
- You want to avoid missing important considerations
- You need to document your thinking process
- You're making decisions that will be hard to reverse

Blue Hat orchestration ensures you've thought through problems from all angles before committing to a solution.