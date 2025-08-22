# Six Thinking Hats Architecture Notes

## Critical Discovery (December 2024)

**Agents cannot call other agents in Claude Code.** This is a fundamental architectural constraint that affects how the Six Thinking Hats framework operates.

## What This Means

### The Misconception
We initially believed Blue Hat could orchestrate other hats by calling them with `Task(subagent_type="hat-name")`. This never actually worked.

### The Reality
- **Commands** (like `/6hats`) CAN call agents ✅
- **Main Claude instance** CAN call agents ✅  
- **Agents** CANNOT call other agents ❌

### The Actual Architecture

```
WORKING PATTERN:
/6hats command → Red Hat (calibration)
              → White Hat (facts)
              → Black Hat (risks)
              → Yellow Hat (benefits)
              → Green Hat (alternatives)
              → Blue Hat (synthesis only)

NOT WORKING:
/6hats → Blue Hat → Other Hats (impossible)
```

## Implications

1. **Blue Hat is a synthesizer, not an orchestrator**
   - It receives input from other hats
   - It cannot call other hats itself
   - Its role is purely synthesis and integration

2. **The `/6hats` command is the real orchestrator**
   - It determines sequences
   - It calls each hat
   - It manages the flow

3. **All agent definitions mentioning Task() for calling other agents are aspirational**
   - These instructions describe what SHOULD happen
   - But cannot actually be executed by agents

## Why This Happened

The confusion arose because:
1. Documentation suggested agents could spawn sub-agents
2. Blue Hat's instructions included Task() calls
3. When it "worked," it was actually the `/6hats` command doing the orchestration
4. Blue Hat appeared to orchestrate but was really just synthesizing

## Workarounds

### For Full Six Hats Analysis
Use the `/6hats` command which properly orchestrates all agents.

### For Direct Agent Use
Call agents individually from the main Claude instance:
```python
Task(subagent_type="red-hat", prompt="...")
Task(subagent_type="white-hat", prompt="...")
# etc.
```

### For Shared Context
Agents can write to files that other agents read, enabling indirect communication.

## References

- Reddit discussion: Users confirmed agents cannot call agents
- GitHub issues: Show some users successfully nesting agents (may be version-specific)
- Claude Code docs: Ambiguous about this capability

## Testing

The `agent-call-tester` agent was created specifically to test this and confirms:
- Task() function is not available to agents
- Agents report they cannot call other agents
- This is consistent across all agent types

## Future Considerations

If Claude Code adds agent→agent calling:
1. Blue Hat could become a true orchestrator
2. The architecture would match the original design
3. The `/6hats` command could simply call Blue Hat

Until then, the `/6hats` command must handle all orchestration.