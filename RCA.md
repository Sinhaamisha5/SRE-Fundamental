# SRE Module: Root Cause Analysis (RCA)

## Overview

Root Cause Analysis is about looking past surface-level symptoms of an incident to find the deeper reasons it happened. The goal is not to assign blame, but to understand the system well enough to prevent the same failure in the future.

---

## The Layer-Peeling Example

### Initial Symptom
Argo CD is getting slower and slower

### Surface Question
Is it a bug? Where's the bottleneck?

### Deeper Investigation
Old jobs are piling up, clogging the system and need removal

### The Process
Root cause analysis helps us peel back these layers step by step until we uncover the true driver of failure.

---

## Four Common RCA Misconceptions

### ❌ Misconception 1: Single Cause
**Wrong**: "Complex systems fail from a single cause"
**Reality**: There are often multiple contributing factors

### ❌ Misconception 2: Human Error is Root Cause
**Wrong**: "The person who made the mistake is the problem"
**Reality**: Human error is a symptom that points to system design flaws

### ❌ Misconception 3: Only for Major Outages
**Wrong**: "RCA is just for big incidents"
**Reality**: Even small incidents can teach us a lot

### ❌ Misconception 4: Purpose is Blame
**Wrong**: "RCA finds who to blame"
**Reality**: Goal is to improve the system and prevent recurrence

---

## Root Cause vs. Contributing Factors

### Root Cause
**Definition**: The underlying reason that the incident occurred

**Example**: Faulty configuration deployed to production

**Characteristic**: If fixed, would have prevented the incident

---

### Contributing Factors
**Definition**: Conditions that made it worse or harder to recover from

**Example**: Monitoring didn't alert the right team

**Characteristic**: Made the incident more severe, but didn't cause it

---

## Four Layers of Causation

When analyzing incidents, separate these levels:

### Layer 1: Immediate Cause
**The direct technical issue**

Example: Database server crashed

### Layer 2: Contributing Factors
**Conditions that made the issue worse**

Example: Sudden spike in queries

### Layer 3: Root Causes
**Systemic gaps**

Example: Poor capacity planning

### Layer 4: System Weaknesses
**Cultural or organizational choices**

Example: Prioritizing speed over reliability

**Key Point**: Effective RCA digs through each of these layers so we can not only fix the symptom, but also strengthen the system as a whole.

---

## Case Study: 2017 Amazon S3 Outage

### The Incident
One of the most famous examples in modern cloud computing

### What Happened

**Initial Action**: Engineer debugging a billing issue

**Mistake**: Accidentally removed too many servers

**Impact**: Servers supported S3's index and placement systems (both critical)

**Consequence**: Restarting those subsystems at scale caused 4-hour outage in US-East-1 region

### The Cascade

**Primary Impact**: S3 became unavailable

**Secondary Impact**: 
- EC2 affected
- EBS affected
- Lambda affected
- Even AWS's own service health dashboard went down (it relied on S3!)

**Tertiary Impact**: Thousands of apps and websites experienced outages

**Lesson**: Cascading dependencies without safeguards create massive blast radius

---

## Six-Step RCA Process

Root cause analysis works best as a repeatable process:

### Step 1: Define the Problem Clearly
State exactly what went wrong from user perspective

### Step 2: Collect Relevant Data
Gather logs, metrics, timelines, chat transcripts

### Step 3: Identify Causal Factors
What conditions enabled this issue?

### Step 4: Determine Actual Root Causes
Dig deeper - why did those conditions exist?

### Step 5: Recommend and Test Solutions
Propose fixes and validate they work

### Step 6: Implement and Monitor Results
Deploy fixes and verify effectiveness

---

## Applying Six Steps to S3 Outage

### Step 1: Define the Problem
```
Problem Statement:
In US-East-1 region, AWS S3 was unable to handle 
core operations (GET, PUT, LIST) for 4 hours, 
disrupting many AWS services and customer applications.
```

---

### Step 2: Collect the Data

**Sources Examined**:
- Execution logs
- Recovery timelines
- Command history
- System metrics

**Key Findings**:
- Traced to routine capacity removal command
- Observability gaps discovered
- S3 health dashboard itself failed during outage
- Recovery took longer than expected

---

### Step 3: Identify Causal Factors

**Primary Factor**: Too many servers taken offline at once

**Secondary Factor**: Safeguards that should have limited impact weren't in place

**Tertiary Factor**: Dashboard dependency on affected service prevented visibility

---

## The Five Whys Technique

### How It Works
Start with a symptom and ask "why" five times (or more/less as needed)

**Purpose**: Each answer leads you deeper into causal factors

**Note**: The number "5" isn't magical - sometimes you need more or fewer iterations

---

### Example: Website Down for 2 Hours

**Problem**: Website is down

**Why 1**: Why is the website down?
→ Database server ran out of connections

**Why 2**: Why did it run out of connections?
→ Too many connections were created and not closed

**Why 3**: Why were connections not closed?
→ Connection pooling mechanism failed

**Why 4**: Why did the pooling mechanism fail?
→ Timeouts weren't properly handled in the codebase

**Why 5**: Why weren't timeouts properly handled?
→ No clear timeout handling standards were in place, and reviews didn't check for them

**Root Cause Identified**: Lack of coding standards and review processes for connection management

---

### Actionable Improvements from Five Whys

Based on the example above:

1. ✅ **Update error handling standards** to cover timeouts
   - Owner: Development Lead
   - Due: 2 weeks

2. ✅ **Add timeout checks** to code review checklist
   - Owner: Engineering Manager
   - Due: 1 week

3. ✅ **Implement connection pool monitoring** with alerts at 80%
   - Owner: SRE Team
   - Due: 3 weeks

**Result**: Each step directly reduces the chance of the same outage happening again.

---

## The Ishikawa (Fishbone) Diagram

### What It Is
A visual tool for organizing possible causes into categories

### Common Categories
- **People**: Skills, training, communication
- **Processes**: Procedures, workflows, standards
- **Tools**: Technology, systems, software
- **Environment**: Infrastructure, dependencies, external factors
- **Technology**: Architecture, design, implementation

### How to Use It

**Step 1**: Draw main branches (categories above)

**Step 2**: Brainstorm possible causes for each branch

**Step 3**: Keep asking "why" to uncover sub-causes

**Step 4**: Narrow down the most significant ones

### Why It Works
Helps teams brainstorm broadly and avoid tunnel vision when investigating incidents

**Benefit**: Ensures you're looking at the full picture, not just obvious symptoms

---

## S3 Outage: Root Cause Determination (Step 4)

### The Mix
Issue came down to combination of human error and inadequate tooling

### Root Causes Identified

**1. Inadequate Safeguards**
- Single command could remove critical capacity
- No validation or confirmation for destructive operations
- No limits on removal scope

**2. Lack of Blast Radius Containment**
- System couldn't isolate failures
- Cascading dependencies not compartmentalized
- Single region failure affected global services

**3. Observability Gaps**
- Health dashboard depended on affected service
- Couldn't monitor the problem during the problem

**Key Insight**: Recognizing these root causes allowed Amazon to focus on building stronger safeguards and better operational tools.

---

## S3 Outage: Recommend and Test Solutions (Step 5)

### Solutions Proposed

**1. Add Safeguards**
- Implement confirmation prompts for large-scale operations
- Add rate limiting to capacity removal
- Require multi-step approval for critical operations

**2. Build Better Dashboards**
- Distribute health dashboard across regions
- Ensure monitoring independence from monitored services
- Create redundant observability systems

**3. Adopt Cell-Based Architecture**
- Limit scope of failures to smaller "cells"
- Prevent cross-cell cascading
- Reduce blast radius of any single failure

### Testing Stage
**Critical**: Cannot be overlooked - ensures fixes address root problem, not just symptoms

**Testing Activities**:
- Simulate failure scenarios
- Validate safeguards work as intended
- Verify dashboard independence
- Test cell isolation

---

## Effective Action Items: Vague vs. Strong

### ❌ Vague Action Items (Don't Help)

**Example 1**: "Be more careful next time"
- Not specific
- Not measurable
- No owner
- No deadline

**Example 2**: "Improve monitoring"
- Too broad
- Unclear success criteria
- No accountability

**Example 3**: "Write better code"
- Subjective
- No clear definition
- Can't be verified

---

### ✅ Strong Action Items (Drive Results)

**Characteristics**:
1. **Specific**: Exactly what will be done
2. **Measurable**: Clear success criteria
3. **Assigned**: Clear owner
4. **Realistic**: Resources available
5. **Time-bound**: Deadline specified
6. **Verified**: Checked after implementation

---

### Examples Transformed

**Before**: "Improve monitoring"
**After**: 
```
Implement connection pool monitoring with alerts 
at 80% utilization
Owner: Jane (SRE Team)
Due: March 15, 2025
Success criteria: Alert fires within 30 seconds 
when threshold breached
```

---

**Before**: "Write better code"
**After**:
```
Create and implement timeout handling standards 
for all API calls
Owner: Development Team
Due: Q2 2025
Success criteria: 
- Standards document published
- Training completed for all engineers
- Code review checklist updated
- 100% of new code passes timeout checks
```

---

## S3 Outage: Implementation and Monitoring (Step 6)

### What AWS Did

**1. Rolled Out Safeguards**
- Added operation size limits
- Implemented confirmation prompts
- Created safety checks for critical commands

**2. Distributed Service Health Dashboard**
- Spread across multiple regions
- Removed dependency on single service
- Ensured visibility during failures

**3. Adopted Cell-Based Architecture**
- Compartmentalized services
- Limited failure scope
- Prevented cross-cell cascading

**4. Improved Communication**
- Better incident updates
- Clearer customer communication
- More transparent status reporting

### They Didn't Stop at Deployment

**Continuous Monitoring**:
- Tracked effectiveness of fixes
- Measured incident reduction
- Validated safeguards prevented similar issues
- Refined based on operational experience

---

## Common Technical Root Causes

**Across industries, certain patterns show up repeatedly**

### 1. Insufficient Monitoring
- Blind spots in observability
- Missing critical metrics
- Alerts not covering failure modes

### 2. Poor Capacity Planning
- Resource exhaustion
- Unexpected load handling
- Growth not anticipated

### 3. Missing Error Handling
- Unhandled exceptions
- No graceful degradation
- Crashes instead of recoveries

### 4. Silent Failures
- Errors that go unnoticed
- Background job failures
- Data corruption without alerts

### 5. Cascading Dependencies Without Safeguards
- No circuit breakers
- No bulkheads
- Failures spread uncontrolled

**Key Point**: These aren't rare surprises - they're well-known failure modes. Recognizing them early lets us learn from others' experience and strengthen our systems before history repeats itself.

---

## Common Process Root Causes

Beyond technical issues, many incidents trace back to process problems:

### 1. Inadequate Testing
- Especially around failure modes
- Missing edge cases
- Insufficient load testing
- No chaos engineering

### 2. Unclear Ownership
- Nobody responsible for service
- Accountability gaps
- Unclear escalation paths

### 3. Inconsistent Deployment Procedures
- Manual steps
- Undocumented processes
- Tribal knowledge
- No standard pipeline

### 4. Missing or Outdated Documentation
- Runbooks don't exist
- Documentation not maintained
- Procedures changed but not documented

### 5. Poor Knowledge Transfer
- Information siloed
- No cross-training
- Dependencies on individuals
- Onboarding gaps

---

## Common Organizational Root Causes

Finally, many failures come down to organizational patterns:

### 1. Features Prioritized Over Reliability
- Teams under pressure
- "Ship it now, fix it later"
- Technical debt accumulates
- Shortcuts become permanent

### 2. Rushed or Incomplete Training
- Fast onboarding without depth
- Learning on the job without support
- No time for proper ramp-up

### 3. Siloed Teams
- Cross-functional communication harder
- Knowledge not shared
- Dependencies not understood
- Blame between teams

### 4. Time Pressure and Technical Debt
- Deadlines drive shortcuts
- "We'll fix it later" never happens
- Debt compounds over time
- System becomes fragile

### 5. Normalized Warning Signs
- Small issues ignored
- "It's always been that way"
- Gradual degradation accepted
- Red flags become background noise

**Critical Insight**: These cultural and organizational issues are often the hardest to fix, but addressing them creates the biggest long-term gains in resilience.

---

## RCA Best Practices

### Do's

✅ **Look for multiple causes** - Complex systems fail in complex ways

✅ **Dig deeper than "human error"** - Always ask what system allowed the error

✅ **Use structured techniques** - Five Whys, Fishbone diagrams

✅ **Focus on systems, not people** - Blameless throughout

✅ **Create specific action items** - With owners and deadlines

✅ **Test your solutions** - Validate fixes work before declaring victory

✅ **Monitor after implementation** - Verify effectiveness over time

✅ **Learn from others** - Study public post-mortems

✅ **Address all layers** - Technical, process, and organizational

✅ **Document thoroughly** - Knowledge for future incidents

---

### Don'ts

❌ **Don't stop at first answer** - Keep digging deeper

❌ **Don't accept "human error"** - It's never the full story

❌ **Don't skip testing** - Unvalidated fixes may not work

❌ **Don't create vague action items** - "Be more careful" helps nobody

❌ **Don't ignore organizational causes** - They're often the most important

❌ **Don't forget to follow up** - Implementation is where value is created

❌ **Don't work in isolation** - Involve multiple perspectives

❌ **Don't rush** - Thorough RCA takes time

---

## RCA Template

### Incident Summary
- Date/Time
- Duration
- Severity
- Services affected
- User impact

### Timeline
- Key events in chronological order
- Decision points
- Actions taken

### Immediate Cause
What directly caused the failure?

### Contributing Factors
What made it worse or harder to recover?

### Root Causes
What systemic issues enabled this?

### System Weaknesses
What organizational/cultural factors contributed?

### Five Whys Analysis
Walk through the causal chain

### Fishbone Diagram
(If applicable) Categorized causes

### Action Items
| Priority | Action | Owner | Due Date | Status |
|----------|--------|-------|----------|--------|
| P0 | ... | ... | ... | ... |

### Lessons Learned
Key takeaways for the organization

### Follow-Up Plan
How we'll verify fixes worked

---

## Connection to Previous Modules

### Simplicity (Module 1)
- Simple systems have clearer root causes
- Complexity hides causal relationships
- RCA reveals unnecessary complexity

### Dependency Management (Module 2)
- Dependencies are common root causes
- Cascading failures need RCA
- Circuit breakers prevent deeper analysis

### Change Management (Module 3)
- Many incidents trace to changes
- RCA improves change processes
- Better validation prevents issues

### Capacity Planning (Module 4)
- Resource exhaustion is common root cause
- RCA reveals planning gaps
- Better forecasting prevents incidents

### Toil Management (Module 5)
- Manual processes are common causes
- RCA identifies automation opportunities
- Eliminating toil prevents incidents

### Incident Preparation (Module 6)
- RCA improves preparation
- Learnings update playbooks
- Better responses next time

### Alerting (Module 7)
- RCA reveals monitoring gaps
- Missed alerts are often factors
- Improves future alerting

### Blameless Post-Mortems (Module 8)
- RCA is core post-mortem activity
- Blameless culture enables honest RCA
- Together they drive improvement

### Root Cause Analysis (This Module)
- Completes the incident learning loop
- Turns failures into improvements
- Builds long-term resilience

---

## Key Takeaways

1. **RCA looks past symptoms** to find deeper systemic issues

2. **Multiple factors, not single causes** - Complex systems fail in complex ways

3. **Human error is never the root cause** - It points to system design flaws

4. **Use structured techniques** - Five Whys, Fishbone diagrams prevent tunnel vision

5. **Four layers of causation** - Immediate, contributing, root, systemic weakness

6. **Strong action items are specific** - Measurable, assigned, time-bound, verified

7. **Test your solutions** - Validate before declaring victory

8. **Monitor after implementation** - Verify fixes actually work

9. **Learn from common patterns** - Technical, process, organizational causes repeat

10. **Address organizational issues** - Hardest to fix but biggest long-term gains

---

## Reflection Questions

1. Do you dig deeper than "human error" in your RCAs?
2. Are your action items specific with owners and deadlines?
3. Do you use structured techniques like Five Whys?
4. Do you test solutions before considering RCA complete?
5. Do you address organizational root causes, not just technical?
6. Do you monitor effectiveness of implemented fixes?
7. Do you learn from others' public post-mortems?
8. Do you analyze small incidents, or only major outages?

---

## Incident Management Module Complete

This completes the **Incident Management** module covering:

1. ✅ **Preparing for Incidents** - Hope is not a strategy
2. ✅ **Effective Alerting** - Wake up for real reasons
3. ✅ **Blameless Post-Mortems** - Learn without blame
4. ✅ **Root Cause Analysis** - Understand deeply, prevent recurrence

**The Complete Loop**:
- Prepare → Detect (Alert) → Respond → Learn (Post-Mortem) → Analyze (RCA) → Improve → Prepare Better

---

## Looking Ahead: Release Engineering

> **"Next, we'll move into the release engineering module where we'll look at how software delivery practices can reduce risk before incidents even happen."**

**Prevention is better than cure**

Release engineering focuses on:
- Reducing incidents at the source
- Safe deployment practices
- Quality gates
- Automated testing
- Continuous delivery

The best incident is the one that never happens.

---

## Final Thought

> **"Root cause analysis is not about finding who to blame - it's about understanding our systems deeply enough to make them better."**

Every incident is an opportunity. RCA is how we seize that opportunity to build more resilient systems, better processes, and stronger organizations.

Don't waste your incidents. Learn from them deeply.


## Five Whys Example: Website Down for 2 Hours
The Problem
Website was down for 2 hours

Why #1: Why was the website down?
Answer: The database server ran out of connections
What this tells us: The symptom - we know it's a database connection issue, but not why it happened

Why #2: Why did the database run out of connections?
Answer: Too many connections were created and not properly closed
What this tells us: We're getting closer - there's a connection leak, but we don't know why the leak exists

Why #3: Why were connections not closed properly?
Answer: The connection pooling mechanism failed
What this tells us: The system designed to manage connections broke down, but we need to know why it failed

Why #4: Why did the connection pooling mechanism fail?
Answer: Timeouts weren't properly handled in the codebase
What this tells us: We've found a technical implementation gap - missing timeout handling broke the pooling logic

Why #5: Why weren't timeouts properly handled?
Answer: No clear timeout handling standards were in place, and code reviews didn't check for them
What this tells us: ROOT CAUSE IDENTIFIED - This is a systemic/organizational issue, not just a technical bug

Root Cause Summary
Surface symptom: Website down
Immediate technical cause: Database connection exhaustion
Deeper technical cause: Connection leak from missing timeout handling
ROOT CAUSE: Lack of coding standards and review processes for connection management

Why This is the Root Cause
If we had proper standards and review processes in place:

✅ Timeout handling would be a requirement
✅ Code reviews would catch the missing implementation
✅ Connection pooling would work correctly
✅ Connections would be properly closed
✅ Database wouldn't exhaust connections
✅ Website would stay up

Fixing just the immediate bug (adding timeout handling to this one piece of code) = Band-aid
Fixing the root cause (implementing standards and review processes) = Prevents this entire class of problems

Actionable Improvements
Based on reaching the root cause, we can create specific action items:
Action Item 1: Create Standards
Action: Update error handling standards to include 
timeout handling requirements for all database operations

Owner: Development Lead
Due Date: 2 weeks
Success Criteria: Standards document published and 
reviewed by team
```

### Action Item 2: Update Review Process
```
Action: Add timeout handling checks to code review 
checklist

Owner: Engineering Manager
Due Date: 1 week
Success Criteria: Checklist updated and enforced in 
all PRs
```

### Action Item 3: Implement Monitoring
```
Action: Implement connection pool monitoring with 
alerts at 80% utilization

Owner: SRE Team (Jane)
Due Date: 3 weeks
Success Criteria: Alert fires within 30 seconds when 
threshold breached
```

### Action Item 4: Fix Current Code
```
Action: Audit existing codebase for missing timeout 
handling and remediate

Owner: Development Team
Due Date: 1 month
Success Criteria: 100% of database operations have 
proper timeout handling
```

---

## The Key Insight

Notice how each "Why" takes us deeper:

1. **Why #1**: Symptom level (website down)
2. **Why #2**: Immediate technical cause (connection exhaustion)
3. **Why #3**: Implementation issue (connections not closed)
4. **Why #4**: Technical gap (pooling failed)
5. **Why #5**: **Systemic/organizational issue** (no standards or reviews)

**If we stopped at Why #3**, we'd just fix the connection leak in that one place.

**By going to Why #5**, we fix the underlying system that allowed this problem (and prevents similar problems in the future).

---

## Visual Representation
```
Website Down (2 hours)
    ↓ WHY?
Database out of connections
    ↓ WHY?
Connections created but not closed
    ↓ WHY?
Connection pooling failed
    ↓ WHY?
Timeouts not handled
    ↓ WHY?
No standards + No review checks
    ↓
ROOT CAUSE: Process Gap