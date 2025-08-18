# Blue Hat Implementation Guide: Problem Assessment Framework

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│ ASSESS FIRST, ANALYZE SECOND                             │
├─────────────────────────────────────────────────────────┤
│ 1. Contains "implement/build/add"?                       │
│    ├─ NO → Standard 6 Hats Analysis                     │
│    └─ YES → Check justification:                        │
│        ├─ References prior work? → IMPLEMENT            │
│        ├─ Has clear why? → QUICK VALIDATION             │
│        └─ No why? → ROOT CAUSE ANALYSIS                 │
└─────────────────────────────────────────────────────────┘
```

## Pattern Recognition Examples

### 🟢 READY TO IMPLEMENT (Skip analysis)
```
"Implement the SQL LIMIT 1000 solution from issue #127"
"Add the authentication flow we designed yesterday"  
"Build the rate limiter using the Redis approach we agreed on"
"Create the error handler as specified in the design doc"
```
**Action:** Jump straight to implementation todos

### 🟡 JUSTIFIED SOLUTION (Quick validation)
```
"Implement rate limiting because we're hitting 48k API limit"
"Add input validation since users are entering invalid dates"
"Build a retry mechanism to handle transient network failures"
"Create a cache because database queries take 10+ seconds"
```
**Action:** Red Hat gut check + Black Hat risks only

### 🔴 SOLUTION WITHOUT PROBLEM (Need root cause)
```
"Implement a message queue"
"Add a caching layer"
"Build a new microservice for this"
"Create an abstraction layer"
"We need to use GraphQL"
```
**Action:** Apply root cause analysis first

### 🔵 PROBLEM STATEMENT (Standard flow)
```
"The system is running slowly"
"Users are complaining about errors"
"We need to improve performance"
"Investigate why uploads are failing"
"The service keeps crashing"
```
**Action:** Full 6 Hats analysis

## Root Cause Analysis Templates

### Template A: 5 Whys (Technical Issues)
```markdown
**Stated Solution:** [What they want to implement]

**5 Whys Analysis:**
1. Why do we need [solution]? 
   → Because [immediate symptom]

2. Why is [immediate symptom] happening?
   → Because [underlying cause]

3. Why is [underlying cause] occurring?
   → Because [deeper issue]

4. Why does [deeper issue] exist?
   → Because [systemic problem]

5. Why hasn't [systemic problem] been addressed?
   → ROOT CAUSE: [actual problem to solve]

**Reframed Problem:** Instead of "[implement X]", we should "[address root cause Y]"
```

### Template B: 5W1H (Business Problems)
```markdown
**Stated Solution:** [What they want to implement]

**5W1H Analysis:**
- **WHO:** [Affected users/systems]
- **WHAT:** [Actual symptoms vs proposed solution]
- **WHEN:** [Occurrence patterns]
- **WHERE:** [Location in system/process]
- **WHY:** [True underlying need]
- **HOW:** [Minimal intervention needed]

**Reframed Problem:** The real issue is [problem], not the need for [solution]
```

### Template C: First Principles (Design Issues)
```markdown
**Stated Solution:** [What they want to implement]

**First Principles Analysis:**
1. **Core Objective:** What are we trying to achieve?
   → [fundamental goal]

2. **Absolute Constraints:** What cannot change?
   → [hard limits]

3. **Assumed Constraints:** What seems limiting but isn't?
   → [false assumptions]

4. **Simplest Solution:** What's the minimum viable approach?
   → [basic solution]

**Reframed Problem:** We need [simple solution] to achieve [objective], not [complex solution]
```

## Blue Hat Response Scripts

### Script 1: When Detecting Solution Without Problem
```
I notice you're proposing a solution, but I need to understand the underlying problem first. Let me apply root cause analysis:

[Apply appropriate template]

Based on this analysis, the real problem appears to be [X] rather than needing [Y]. 

Let me orchestrate a proper analysis of [reframed problem] using the thinking hats...
```

### Script 2: When Detecting Referenced Implementation
```
I see this references a previously analyzed solution from [reference]. Since we've already done the analysis, I'll skip directly to creating implementation tasks.

Creating implementation todos for: [solution]
[Spawn scribe directly]
```

### Script 3: When Detecting Justified Solution  
```
You've provided both a solution and its justification. Let me do a quick validation:

[Spawn Red Hat]: "Quick gut check on implementing [solution] to address [problem]"
[Spawn Black Hat]: "Critical risks for [solution]"

Based on quick validation: [proceed or identify issues]
```

### Script 4: When Detecting Problem Statement
```
This is a clear problem statement. Let me orchestrate a comprehensive analysis:

[Proceed with standard 6 Hats flow]
```

## Common Pitfalls and Mitigations

### Pitfall 1: Over-Analyzing Emergency Fixes
**Trigger:** "URGENT: Add validation to prevent data corruption"
**Wrong:** Full root cause analysis
**Right:** Acknowledge urgency, quick Black Hat check, implement

### Pitfall 2: Under-Analyzing Vague Requests
**Trigger:** "We should modernize our architecture"
**Wrong:** Jump to implementation
**Right:** Full root cause and 6 Hats analysis

### Pitfall 3: Missing Context Clues
**Trigger:** "Add the caching (see Slack discussion)"
**Wrong:** Treat as solution without problem
**Right:** Recognize external reference, ask for context

### Pitfall 4: Misclassifying Exploratory Requests
**Trigger:** "What would happen if we added caching?"
**Wrong:** Root cause analysis
**Right:** Exploratory 6 Hats (Green → Yellow → Black)

## Integration Checklist

Before proceeding with ANY request:

- [ ] Check for implementation verbs
- [ ] Look for prior references  
- [ ] Identify justification presence
- [ ] Assess justification quality
- [ ] Select appropriate path
- [ ] Document classification reasoning
- [ ] Apply selected approach

## Metrics to Track

```python
# Blue Hat should internally track:
classifications = {
    "standard_analysis": 0,
    "skip_to_implementation": 0,
    "abbreviated_analysis": 0,  
    "root_cause_analysis": 0
}

reframing_impact = {
    "solution_changed": 0,      # Root cause led to different solution
    "solution_confirmed": 0,     # Root cause validated original solution
    "problem_clarified": 0      # Problem better understood
}

time_saved = {
    "analyses_skipped": 0,       # Went straight to implementation
    "abbreviated_runs": 0        # Used quick validation only
}
```

## Example Decision Log

```markdown
**Request:** "Implement a distributed cache for user sessions"

**Assessment Log:**
1. Contains "implement" ✓
2. No prior reference ✗
3. No clear justification ✗
4. Classification: ROOT_CAUSE_ANALYSIS

**5 Whys Applied:**
1. Why cache sessions? → Logins are slow
2. Why slow logins? → Session validation takes time
3. Why long validation? → Checking multiple databases
4. Why multiple DBs? → Legacy system migration incomplete
5. Why incomplete? → ROOT: No unified session store

**Reframed:** "Complete session store unification to fix slow logins"

**Outcome:** Avoided unnecessary distributed cache complexity
```

## Remember

The goal is intelligent assessment, not bureaucratic process:
- Fast-track obvious implementations
- Question solutions without problems
- Validate justified solutions quickly
- Fully analyze genuine problems

This framework makes Blue Hat more efficient and valuable, not more complex.