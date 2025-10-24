# SRE Module: Preparing for Incidents

## Overview

Even with the best design, things will go wrong. Systems are complex, failure is inevitable, and incident management is about what happens when reliability breaks down - and how we learn from failures rather than repeat them.

> **"Incidents aren't just disruptions, they're opportunities to improve the system and build resilience over time."**

---

## The 2:47 AM Scenario

**Situation**: It's 2:47 AM on Black Friday. Your payment system goes down. Every minute costs $50,000.

### Two Types of Engineers Emerge:

**Engineer A (Prepared)**
- ✅ Calm, they've practiced before
- ✅ Follow playbooks
- ✅ Coordinate responses
- ✅ Know how to escalate

**Engineer B (Unprepared)**
- ❌ Frozen, unsure what to do next
- ❌ Makes things worse
- ❌ Doesn't know who to call
- ❌ Doesn't know what logs to check
- ❌ Doesn't even know how to declare an incident

**Reality**: Preparation doesn't prevent outages, but it shapes how effectively you respond when they happen.

---

## The Preparedness Paradox

### The Problem
When things are working fine, preparation feels like wasted effort. "Why run drills for problems that might never happen?"

### The Reality
- Teams that prepare see **fewer incidents**
- Teams that don't turn **minor issues into major disasters**
- The same exact issue can vary hugely in cost depending on team preparedness

---

## The Quote That Matters Most

> **"Hope is not a strategy." - Ben Treynor**

The key difference between a minor hiccup and a major outage often lies in how well prepared you and your team are.

---

## Five Pillars of Preparation

### 1. Clear Processes
Establish processes for detection, response, and recovery

### 2. Defined Roles
Create role-based responsibilities and escalation paths

### 3. Documented Playbooks
Develop playbooks for recurring issues

### 4. Training & Simulations
Train team members through exercises

### 5. Continuous Improvement Culture
Build a culture where preparedness keeps improving

**Key Insight**: Preparedness is an ongoing and highly improvable skill that gets better through experience and practice.

---

## The Cost of Being Unprepared

### 1. Extended Outage Durations
Disorganized responses drag out incidents

### 2. Higher Business Impact
Service disruptions cost more

### 3. Team Burnout
Chaotic, stressful incidents traumatize engineers
- Hard to recover from
- SREs experience genuine trauma

### 4. Reputation Damage
Repeated similar incidents erode trust

### 5. Knowledge Silos
Only specific engineers can resolve issues

**Bottom Line**: The cost of not preparing is almost always higher than the cost of the prep work itself.

---

## Anti-Patterns vs. Recommended Practices

### ❌ Anti-Pattern: Trial by Fire
Throwing juniors into on-call with no training

### ✅ Recommended: Structured Learning
Design concrete, sequential learning and ramp-up paths

---

### ❌ Anti-Pattern: Playbook Robots
Only following procedures to the letter

### ✅ Recommended: Critical Thinking
Encourage reverse engineering and statistical thinking to understand WHY incidents happen

---

### ❌ Anti-Pattern: Hiding Failures
Concealing outages or involvement to avoid blame

### ✅ Recommended: Blame-Free Culture
Celebrate analyzing failures and running postmortems to build resilience

---

### ❌ Anti-Pattern: Production Baptism
First attempt at solving problem is the real on-call incident

### ✅ Recommended: Chaos Engineering
Create contained, realistic breakages to practice before real incidents

---

### ❌ Anti-Pattern: Expert Hoarding
Compartmentalization of expertise among few individuals

### ✅ Recommended: Knowledge Sharing
Role-play theoretical disasters as a team; spread knowledge

---

### ❌ Anti-Pattern: Sink or Swim
Pushing juniors into primary on-call too soon

### ✅ Recommended: Shadow Rotations
Juniors shadow on-call heavily before being primary

---

### ❌ Anti-Pattern: Static Training
On-call training plans treated as sacred and unchanging

### ✅ Recommended: Dynamic Pairing
Juniors paired with expert senior SREs; routinely revise training

---

### ❌ Anti-Pattern: Senior-Only Impact
Important work only given to seniors

### ✅ Recommended: Junior Ownership
Include juniors in meaningful projects to build confidence

---

## Sustainable On-Call System Principles

### 1. Manageable Shifts
- **Duration**: One week maximum
- **Handoff**: 30-minute overlap between shifts
- Clear start and end times

### 2. Primary + Secondary Model
- Primary on-call engineer
- Secondary backup engineer
- Both always at ready

### 3. Follow-the-Sun for Global Teams
Distribute on-call across time zones

### 4. Dedicated Recovery Time
Scheduled recovery after on-call shifts

### 5. Appropriate Paging
Engineers only paged for services they can actually troubleshoot and remediate

---

## Code Cloud Record Store: Example Rotation

### Primary Rotation
- **Shift Length**: Maximum 1 week
- **Handoff**: 30-minute overlap
- **Responsibility**: Systems they can actually troubleshoot
- **Escalation**: After 15 minutes if no action on alert

### Secondary Rotation
- **Timing**: Staggered 3 days behind primary
- **Purpose**: 
  - Primary and secondary aren't always same pair
  - More exposure to different scenarios
- **Role**:
  - Backup if primary unavailable
  - Next in escalation ladder
  - Fresh perspective on complex issues

### Recovery Rotation
- **After Primary**: At least 2 weeks off from on-call duties
- **After Major Incident**: Not placed on-call right away
- **Weekend Pages**: Compensation time given

### Escalation Ladder

**0-15 min**: Primary (acknowledge and begin working)
↓
**15-30 min**: Secondary (if unacknowledged or persists)
↓
**30-60 min**: Team Lead
↓
**60+ min**: Engineering Manager (P0/P1 only)

---

## Incident Playbooks: Your 2 AM Lifesaver

### What Is a Playbook?
A step-by-step guide for diagnosing and resolving a common issue.

**When you're woken up at 2 AM, you don't want to rely on memory alone.**

### Benefits
- ✅ Reduces stress
- ✅ Increases consistency
- ✅ Speeds up resolution
- ✅ Enables junior engineers

---

## Key Elements of Effective Playbooks

### 1. Clear Trigger Conditions
Alerts or symptoms that indicate the issue

### 2. Step-by-Step Troubleshooting
Guide responder through initial checks
- Eliminate obvious causes
- Narrow down the problem

### 3. Dashboard Links
Direct links to relevant monitoring tools

### 4. Escalation Criteria
Exactly who to reach if you need help
- Contact information
- When to escalate

### 5. Recovery & Verification Steps
- Restore system to healthy state
- Confirm the fix actually worked

---

## Example Playbook: Database Connection Pool Exhaustion

### Trigger Conditions
- ✓ Database connection limit at 90%
- ✓ Error logs: "connection refused" or "too many connections"

### Impact
- User transactions failing with 500 errors
- Admin dashboard shows degraded performance

### Initial Triage Steps
1. **Check connection pool metrics dashboard** (direct link provided)
2. **Verify database service health** (command included)
3. **Check recent code deployments** (potential connection leaks)

### Mitigation Steps

**If recent deployment:**
- Follow rollback procedure (steps provided)

**If no recent changes:**
- Restart application servers in sequence (exact steps)

**If issue persists:**
- Temporarily increase connection pool size (command provided)

### Escalation Guidance
**Timeframe**: If not resolved within 15 minutes
**Action**: Escalate to database team
**Contact**: Email / PagerDuty handle / on-call schedule

### Prevention
- Review application code for connection leaks
- Implement connection timeout monitoring
- Add alerts before exhaustion occurs

---

## Incident Priority Levels

Used as shorthand to indicate severity - crucial for communication and deciding who needs involvement.

### P0: Critical Incident
- **Impact**: Complete service outage
- **Examples**: Payment system down, entire site unreachable
- **Response**: Immediate all-hands
- **Business Impact**: Maximum

### P1: Severe Degradation
- **Impact**: Major functionality disrupted or broken
- **Examples**: Login broken, checkout failing
- **Response**: Immediate primary + secondary
- **Business Impact**: High

### P2: Partial Impact
- **Impact**: Degraded service, partially usable
- **Examples**: Slow load times, some features unavailable
- **Response**: Primary on-call
- **Business Impact**: Moderate

### P3: Non-Critical Issues
- **Impact**: Service works, but risk identified
- **Examples**: Warnings, potential future issues
- **Response**: Can wait for business hours
- **Business Impact**: Low

---

## When to Escalate

### 1. Time Thresholds Passed
Most common reason - no resolution within expected timeframe

### 2. Requires Specialized Expertise
Issue beyond your scope or knowledge area

### 3. Spans Multiple Teams/Services
Cross-functional coordination needed

### 4. Impact Increasing
Problem getting worse while you're attempting to resolve it

**Key Rule**: Better to escalate early than to struggle alone and extend the incident.

---

## Practice Makes Resilient: Four Exercise Types

### 1. Tabletop Scenarios
**Format**: Verbal walkthroughs of hypothetical situations
**Stakes**: Low
**Focus**: Decision-making practice
**Example**: "What would you do if the database becomes read-only?"

### 2. Game Days
**Format**: Scheduled tests for specific failure modes
**Stakes**: Medium (controlled environment)
**Focus**: Rehearse response to known scenarios
**Example**: Simulate load balancer failure during business hours

### 3. Chaos Engineering
**Format**: Controlled introduction of failure into production
**Stakes**: Medium-High (real system, controlled chaos)
**Focus**: Strengthen both team AND system resilience
**Example**: Netflix's Chaos Monkey killing random instances

### 4. Post-Mortem Reenactments
**Format**: Walk through past incidents with new team members
**Stakes**: Low (educational)
**Focus**: Keep lessons alive, build shared experience
**Example**: Re-simulate last quarter's major outage

**Key Point**: Regular practice makes both people and systems more resilient when real incidents happen.

---

## Google's "Wheel of Misfortune"

### What It Is
A specific incident preparation exercise developed at Google

### How It Works

**Step 1**: Select an incident scenario (real past incident or hypothetical)

**Step 2**: Assign roles
- Incident Commander
- Other team member roles (engineer, communications, etc.)

**Step 3**: Work through scenario step-by-step
- What do you check first?
- What commands do you run?
- When do you escalate?
- How do you communicate?

### Why It's Effective
- Safe environment to make mistakes
- Builds muscle memory
- Reveals gaps in playbooks
- Trains decision-making under pressure

---

## Building Incident Muscle Memory

### The Goal
When things go wrong, you act instead of scrambling

### How to Build It
1. **Practice regularly** - Monthly game days minimum
2. **Rotate roles** - Everyone should IC at some point
3. **Document learnings** - Update playbooks after each exercise
4. **Start small** - Simple scenarios before complex ones
5. **Debrief thoroughly** - What worked? What didn't?

### Signs of Good Muscle Memory
- Engineers know their roles instinctively
- Common issues resolved in < 15 minutes
- Escalations happen smoothly
- Communication flows naturally
- Team stays calm under pressure

---

## Connection to Previous Modules

### Simplicity (Module 1)
- Simple systems have fewer incident types
- Easier to troubleshoot
- Clearer playbooks

### Dependency Management (Module 2)
- Understanding dependencies helps incident response
- Blast radius assessment guides escalation
- Circuit breakers limit incident scope

### Change Management (Module 3)
- Many incidents come from changes
- Good change management reduces incidents
- Rollback is incident mitigation

### Capacity Planning (Module 4)
- Capacity incidents are predictable
- Good planning prevents resource exhaustion incidents
- Monitoring aids incident detection

### Toil Management (Module 5)
- Automating incident response reduces toil
- Playbooks eliminate repetitive work
- Good preparation reduces manual intervention

### Incident Preparation (This Module)
- Ties everything together
- When all else fails, preparation saves you
- Learning from incidents improves all modules

---

## Best Practices Summary

✅ **Document everything** - Playbooks, runbooks, escalation paths

✅ **Practice regularly** - Monthly minimum, more for critical systems

✅ **Manageable rotations** - 1 week max, primary + secondary

✅ **Clear priorities** - P0-P3 everyone understands

✅ **Escalate early** - Better too soon than too late

✅ **Build culture** - Blame-free, learning-focused

✅ **Shadow before primary** - Juniors learn by watching

✅ **Recovery time** - Protect engineers after intense incidents

✅ **Update playbooks** - Living documents, not static

✅ **Measure effectiveness** - Track MTTR, incident frequency

---

## Key Takeaways

1. **Hope is not a strategy** - Preparation is mandatory, not optional

2. **The cost of unpreparedness** is always higher than the cost of preparation

3. **Two types of engineers emerge** at 2:47 AM - be the prepared one

4. **Playbooks are lifesavers** - don't rely on memory at 2 AM

5. **Practice builds muscle memory** - regular exercises are essential

6. **Escalation is not weakness** - it's good judgment

7. **Priority levels matter** - P0-P3 guides response intensity

8. **Junior development is intentional** - shadow, then primary

9. **On-call should be sustainable** - burnout helps no one

10. **Incidents are learning opportunities** - preparation improves from each one

---

## Reflection Questions

1. Do you have documented playbooks for your top 5 incidents?
2. When did your team last practice incident response?
3. Are your on-call rotations sustainable?
4. Can juniors shadow before going primary?
5. Do you have clear P0-P3 definitions?
6. Is your escalation ladder documented and known?
7. What's your 2 AM scenario? Are you ready for it?
8. Does your culture blame or learn from incidents?

---

## Looking Ahead

**Next lesson**: Alerting - the right signals to know when something is breaking.

You can prepare perfectly, but without good alerts, you won't know when to respond. The next lesson covers how to design alerts that wake you up for the right reasons.

---

## Final Thought

> **"The best teams don't have fewer incidents - they just handle them better."**

Preparation turns chaos into coordination. When everyone knows their role, has practiced their responses, and trusts their playbooks, incidents become manageable challenges instead of career-defining disasters.

Start preparing today. Your future 2 AM self will thank you.