# NFR Integration for Six Hats (LLM Adaptation)

## Core Principle
Add NFR awareness as a **lens** each hat applies, not new protocols. Preserve all existing bias corrections.

## Minimal Additions per Hat

### White Hat
Add to investigation protocol:
```
When investigating, also capture:
- Current scale metrics (users, RPS, data volume)
- Current performance (latency P50/P99, throughput)
- Current reliability (uptime, error rates)
```

### Red Hat  
Add to actor discovery:
```
Additional actors to consider:
- The on-call engineer at 3am
- The CFO watching cloud costs
- The security team during an audit
```

### Black Hat
Add to critical questions:
```
NFR-specific risks:
- "What happens at 10x load?"
- "What's the security attack surface?"
- "What breaks when the primary region fails?"
- "What's the monthly AWS bill at scale?"
```

### Yellow Hat
Add to benefit categories:
```
NFR benefits to celebrate:
- Scales horizontally without code changes
- Self-healing capabilities
- Observable by default
- Costs scale sub-linearly
```

### Green Hat
Add to lazy developer questions:
```
- "What solution needs the least ops burden?"
- "What scales without our intervention?"
- "What's already monitored?"
```

### Blue Hat
Add to orchestration:
```
When setting context, specify:
- Current scale vs projected scale
- Critical NFRs for this problem
- Acceptable trade-offs
```

## Why This Works

1. **Preserves existing corrections** - All our LLM bias fixes remain
2. **Natural integration** - NFRs flow through existing thinking modes
3. **No new complexity** - Just awareness, not protocols
4. **Bridge maintained** - Still recognizably Six Hats

## The Key Insight

LLMs already understand NFRs conceptually. We just need to remind them to consider these aspects within their existing thinking patterns. The framework doesn't need restructuring - just gentle nudges toward production concerns.

## Example in Practice

When Blue Hat orchestrates for "adding a new API endpoint":

**Without NFR awareness:**
- White: "What data do we need?"
- Black: "What could fail?"

**With NFR awareness:**
- White: "What data do we need? Current API handles 1000 RPS, P99 latency 200ms"
- Black: "What could fail? At 10x load this would bottleneck the database"

The thinking mode stays the same, but considers production reality.