# SRE Module: Effective Alerting

## Overview

Not all alerts are equal. Some wake you up at 3 AM for no reason. Others only trigger when your users are already furious. This lesson covers what makes an alert effective.

---

## The 3 AM Problem

### Scenario A: Bad Alert
**3 AM**: Phone buzzes
**Action**: Drag yourself out of bed, log in
**Reality**: False alarm - CPU was high for a few minutes, nothing actually broken
**Result**: Lost sleep, frustration, alert fatigue

### Scenario B: Good Alert
**Alert**: Fires only when users cannot check out of your store
**Result**: Real user impact, action required

> **"Good alerts wake you up for real reasons, not noise."**

---

## Five Traits of Effective Alerts

### 1. Requires Human Intervention
If nobody needs to act, it shouldn't fire

### 2. Focuses on User Impact
Specific issues affecting users, not just background metrics

### 3. Provides Clear Context
Simple language with enough information to understand the problem

### 4. Routed to Right Person
Alert goes to the team/person who can actually fix it

### 5. Links to Resources
Provides relevant playbooks and dashboards

---

## Alert Comparison: Non-Actionable vs. Actionable

### ❌ Non-Actionable Alert
```
"CPU usage is high on server prod-api-3"
```

**Problems**:
- What does this mean for the user?
- Is action needed?
- Where do I look?
- What's the impact?

### ✅ Actionable Alert
```
"Order Processing API latency exceeding 2s SLO
Current: 5.2 seconds
Impact: Affecting checkout flow
Playbook: [link]
Dashboard: [link]"
```

**Why it works**:
- Specific metric (latency)
- Points to user impact (checkout)
- Shows SLO violation (2s → 5.2s)
- Provides context and resources

---

## Two Types of Alerts: Symptom vs. Cause

### Symptom-Based Alerts (PRIORITIZE THESE)

**Monitor**: Service from user's perspective

**Focus**: 
- Customer experience
- Latency, errors, availability

**Relationship to Business**: 
- Tied to direct business impact

**Tells You**: 
- WHAT is broken

**Maps To**: 
- SLOs and user journeys

**Example**:
```
"Users cannot play songs
Error rate: 15% (SLO: <1%)
Affected users: ~5,000
Dashboard: [link]"
```

---

### Cause-Based Alerts (USE SPARINGLY)

**Monitor**: Internal system metrics

**Focus**:
- Infrastructure
- CPU, memory, disk usage

**Relationship to Business**: 
- Indirectly related to impact

**Tells You**: 
- WHY something might be broken

**Points To**: 
- Implementation details

**Example**:
```
"Disk usage trending above 95%
Server: prod-db-01
Time to full: ~2 hours"
```

---

## Why Google Prefers Symptom-Based Alerting

### Benefits

1. **Reduces overall alert volume**
   - Fewer false positives
   - Less noise

2. **Keeps attention focused on user impact**
   - What matters most

3. **Allows flexibility in infrastructure**
   - Don't care HOW it works, care THAT it works

4. **Works even when failure modes are unknown**
   - Catches unexpected issues

**Key Principle**: At the end of the day, what matters most is the user's journey.

---

## When Cause-Based Alerts Add Value

Despite the preference for symptom-based, cause-based alerts are useful when:

### 1. Predict Imminent User Impact
**Example**: Disk space trending towards full (will cause user impact soon)

### 2. Provide Critical Diagnostic Information
**During active incidents**: Help identify root cause

### 3. Monitor Security Conditions
**Example**: Failed authentication attempts spike

### 4. Track Resource Exhaustion
**Example**: Connection pool nearing limit

**Critical Rule**: Keep cause-based alerts limited to conditions that are truly actionable and require human intervention. Otherwise, they just add noise.

---

## SLO-Based Alerting: The Foundation

### Why SLOs Matter for Alerting

**SLO = Clear definition of acceptable reliability**

This makes it the perfect foundation for alerting.

### The Problem with Random Thresholds
- "Alert when CPU > 80%"
- Why 80? Why not 75?
- Just guesswork
- Doesn't reflect user impact

### The SLO-Based Solution

**Instead of**: CPU usage > 80%

**Use**: Checkout latency exceeding your SLO target

**Result**: 
- Ties alert directly to reliability promises
- Reflects what users actually experience
- Objective and measurable

---

## SLO-Based Alerting Structure

### Step 1: Define Meaningful SLOs
**Example**: 99.9% availability for each service

### Step 2: Create Burn Rate Alerts
Measure how quickly you're consuming your error budget

### Step 3: Set Varied Thresholds
Different consumption rates catch both:
- Sudden spikes (fast burn)
- Slow steady degradation (slow burn)

---

## Example: Checkout API

### SLO Definition
- **Target**: 99.9% success rate
- **Error Budget**: 0.1% of requests can fail per month
- **Time Budget**: ~43 minutes of errors per month

### What Happens
If error budget burns too quickly → alerts fire

### Why It Works
- Objective: Based on SLO, not guesswork
- Measurable: Clear metrics
- User-focused: Tied to customer experience

---

## Four-Tier SLO Alert System

### URGENT: 100% Budget in 1 Hour
**Severity**: Critical
**Action**: Immediate page to on-call engineer
**Meaning**: Major red flag, burning through monthly budget in an hour
**Example**: 
```
"Checkout API: 100% error budget consumed in 1 hour
Current error rate: 12%
Users affected: ~10,000"
```

---

### HIGH: 25% Budget in 6 Hours
**Severity**: Serious
**Action**: Page to on-call engineer
**Meaning**: Burning too fast, will exhaust budget in a day
**Example**:
```
"Checkout API: 25% error budget consumed in 6 hours
Current error rate: 2.5%
Trend: Increasing"
```

---

### MEDIUM: 50% Budget in 1 Day
**Severity**: Warning
**Action**: Email to team
**Meaning**: Not an emergency, but clear warning sign
**Example**:
```
"Checkout API: 50% error budget consumed in 24 hours
Current error rate: 1.5%
Action: Investigate during business hours"
```

---

### LOW: 75% Budget in 3 Days
**Severity**: Advisory
**Action**: Create ticket
**Meaning**: Investigate before it escalates
**Example**:
```
"Checkout API: 75% error budget consumed in 3 days
Current error rate: 0.8%
Action: Review and address in sprint"
```

---

## Benefits of Tiered Approach

✅ **Balances urgency with noise**
- Team only paged when it really matters

✅ **Visibility into slow-burning risks**
- Catch degradation before it becomes crisis

✅ **Actionable thresholds**
- Each tier has clear response expectations

✅ **Error budget management**
- Track reliability spending in real-time

---

## Code Cloud Record Store: Example Alert

### Instead of Random Threshold
```
❌ "CPU > 80% on checkout service"
```

### Use SLO-Based Alert
```
✅ "Checkout latency exceeding SLO target
   SLO: 500ms for 99% of requests
   Current P99: 1,200ms
   Error budget burn rate: HIGH
   Users affected: ~3,000
   Playbook: /runbooks/checkout-latency
   Dashboard: /grafana/checkout-performance"
```

---

## Alert Routing Principles

### 1. Match Severity to Notification Method

| Severity | Method | Example |
|----------|--------|---------|
| URGENT | Page/SMS | P0 incidents |
| HIGH | Page | P1 incidents |
| MEDIUM | Email | P2 incidents |
| LOW | Ticket | P3 incidents |

---

### 2. Route to Team with Expertise
Alert goes to people who can actually fix it
- Have necessary knowledge
- Have access to systems
- Have authority to make changes

---

### 3. Include Clear Escalation Path
If alerts go unacknowledged:
- Primary → Secondary (15 min)
- Secondary → Team Lead (30 min)
- Team Lead → Manager (60 min)

---

### 4. Follow-the-Sun Models
Handle incidents across regions
- Reduces fatigue
- Respects time zones
- Maintains coverage

---

### 5. Use Deduplication
Prevent alert storms
- Don't flood on-call with repeated notifications
- Group related alerts
- Suppress duplicates for same issue

---

## Six-Step Alert Implementation Checklist

### Step 1: Define User Impact
✓ Be clear about what problem you're detecting
✓ From the user's perspective
✓ What breaks for them?

**Example**: "Users cannot complete checkout"

---

### Step 2: Select Metrics That Indicate Impact
✓ Choose metrics that directly reflect user experience
✓ Not internal metrics unless they predict user impact

**Example**: Checkout completion rate, latency, error rate

---

### Step 3: Establish Thresholds from Real Data
✓ Based on actual performance data
✓ NOT guesswork
✓ Historical analysis
✓ SLO-driven

**Example**: P99 latency > 500ms (based on 6 months of data showing user drop-off)

---

### Step 4: Test Sensitivity
✓ Alerts fire when they should
✓ Don't fire for noise
✓ Test both false positives and false negatives

**Method**: 
- Inject realistic failures
- Verify alert triggers
- Adjust thresholds as needed

---

### Step 5: Document Actions
✓ Create playbook for each alert
✓ Tell responders exactly what to do
✓ Link from alert to playbook

**Playbook should include**:
- Triage steps
- Common causes
- Mitigation procedures
- Escalation criteria

---

### Step 6: Review Regularly
✓ Tune thresholds over time
✓ Reduce false positives
✓ Reduce false negatives
✓ Remove outdated alerts

**Schedule**: Monthly alert review meetings

---

## Alert Fatigue: The Silent Killer

### What It Is
When engineers receive too many alerts, they:
- Start ignoring them
- Assume they're false positives
- Miss real incidents
- Burn out

### Warning Signs
- Alert acknowledgment time increasing
- Multiple alerts going unacknowledged
- Engineers complaining about noise
- High rate of alert dismissals without action

### Solutions
1. **Ruthlessly delete low-value alerts**
2. **Aggregate related alerts**
3. **Use tiered severity properly**
4. **Implement SLO-based alerting**
5. **Regular alert reviews**

---

## Common Alert Anti-Patterns

### ❌ "Everything is Critical"
If everything is urgent, nothing is urgent

**Fix**: Use proper severity levels

---

### ❌ "Alert on Everything"
Monitoring != Alerting

**Fix**: Only alert on actionable, user-impacting issues

---

### ❌ "Random Thresholds"
"CPU > 80%" with no justification

**Fix**: Use SLO-based, data-driven thresholds

---

### ❌ "Missing Context"
Alert says "Error" with no details

**Fix**: Include user impact, metrics, links to resources

---

### ❌ "Wrong Audience"
Alert goes to people who can't fix it

**Fix**: Route to team with expertise and access

---

### ❌ "No Escalation Path"
Alert goes unacknowledged indefinitely

**Fix**: Implement clear escalation ladder with timeframes

---

### ❌ "Never Updated"
Alerts from 3 years ago still firing

**Fix**: Regular review and cleanup

---

## Best Practices Summary

✅ **Prioritize symptom-based alerts** - Focus on user experience

✅ **Use SLO-based alerting** - Objective, measurable, user-focused

✅ **Implement tiered severity** - Page only when necessary

✅ **Provide clear context** - Include impact, metrics, playbooks

✅ **Route appropriately** - Right person, right method, right time

✅ **Document responses** - Every alert needs a playbook

✅ **Review regularly** - Monthly cleanup and tuning

✅ **Test thoroughly** - Verify alerts fire correctly

✅ **Fight alert fatigue** - Ruthlessly eliminate noise

✅ **Include escalation** - Clear path if unacknowledged

---

## Key Metrics to Track

### Alert Health Metrics

1. **Precision**: % of alerts that required action
   - **Target**: >80%

2. **Recall**: % of incidents caught by alerts
   - **Target**: >95%

3. **Time to Acknowledge**: How long until acknowledged
   - **Target**: <5 minutes

4. **False Positive Rate**: % of alerts that were false alarms
   - **Target**: <20%

5. **Alert Volume**: Total alerts per week
   - **Target**: Decreasing trend

---

## Connection to Previous Modules

### Incident Preparation
- Good alerts enable fast response
- Playbooks linked from alerts
- Escalation paths defined

### SLOs (Earlier Module)
- Alerts tied directly to SLO violations
- Error budget burn rate drives alerting
- User impact is the focus

### Monitoring (Next Module)
- Alerts built on monitoring foundation
- Dashboards linked from alerts
- Metrics drive alert decisions

---

## Key Takeaways

1. **Not all alerts are equal** - Some are noise, some are critical

2. **Good alerts wake you for real reasons** - Not false alarms

3. **Symptom-based > Cause-based** - Focus on user impact first

4. **SLO-based alerting is objective** - Not random thresholds

5. **Tiered severity prevents fatigue** - Page only when necessary

6. **Context is critical** - Include impact, metrics, playbooks, links

7. **Route to the right people** - Who can actually fix it

8. **Regular review is essential** - Alerts need continuous tuning

9. **Alert fatigue kills reliability** - Ruthlessly eliminate noise

10. **Every alert needs a playbook** - Action should be clear

---

## Reflection Questions

1. How many of your alerts are symptom-based vs. cause-based?
2. What percentage of your alerts require no action?
3. Are your thresholds based on SLOs or random numbers?
4. Do your alerts include clear user impact?
5. When did you last review and clean up alerts?
6. What's your false positive rate?
7. Are alerts routed to people who can fix them?
8. Does every alert have a linked playbook?

---

## Final Thought

> **"The best alert is the one that wakes you up for something that matters, with enough context to fix it, and links to exactly what you need."**

If your alerts don't meet this standard, start fixing them today. Your sleep, your team's sanity, and your users' experience all depend on it.