# SRE Module: Change Management for Reliability

## Overview

In SRE, change is both necessary and risky. This lesson explores how to accept the reality of change without breaking things - balancing velocity with stability.

### The Reality of Change

> **"Change is the primary source of production incidents at Google. About 70% of incidents were traced back to system modifications."**

Most production issues come from changes - deployments, config changes, and infrastructure modifications. If you care about uptime, you must care about how changes are made.

---

## The Cautionary Tale

**Scenario**: Engineer pushes a hotfix directly to production

"Wait, you didn't follow change review?"

"It was just a quick config tweak..."

**Result**: Small unreviewed changes often lead to big, preventable problems.

### Career Reality Check

> **"There's a non-trivial chance that most engineers will cause, be involved in, or experience a production outage at some stage in their careers."**

**The difference between a career-limiting event and a learning experience is often how well your change management process contains the damage.**

---

## Common Changes That Lead to Failures

1. **Bad Deployments**
   - Ship faulty code
   - Poor rollout process

2. **Misconfigured Settings**
   - Configuration errors
   - Parameter mistakes

3. **Buggy Dependency Upgrades**
   - Broken external libraries
   - Incompatible versions

4. **Environment Changes**
   - Infrastructure modifications
   - Platform updates

5. **Database Migrations**
   - Schema changes
   - Data transformation issues

**All of these happen frequently, and all can be managed better.**

---

## The Core SRE Challenge: Balance Two Competing Goals

### 1. Velocity
Deploying changes that deliver business value quickly

### 2. Stability
Ensuring the service stays reliable as changes are introduced

**Mantra**: Move fast by all means, but do it safely.

---

## The Gym Analogy: Rethinking Change Risk

### Question
**Who's more likely to get hurt at the gym?**
- A) Person who goes once a week
- B) Person who goes five times a week for five years

### Answer
**The once-a-week person** - Infrequent, untested activity is riskier.

### Application to Change Management

**Common Misconception**: More changes = More instability

**Reality**: This is only true when changes are rare and untested.

---

## The Validation-Agility-Stability Triangle

### The Relationship

```
IF testing and validation improves
  THEN agility can improve
  AND stability doesn't have to suffer
```

### Key Insight
**Change isn't inherently bad. Infrequent, untested change is the bad stuff.**

### Benefits of Frequent, Small, Validated Changes

✅ Better learning overall

✅ Greater system resilience

✅ Faster recovery when things go wrong

✅ Faster feedback loops

**Like the gym**: You get better with every rep.

---

## The Change Confidence Loop (Flywheel)

This isn't a trade-off, it's a **flywheel** that builds momentum:

```
1. Small, Frequent Changes
        ↓
2. Clear, Improved Testing
        ↓
3. Better Validation
        ↓
4. Greater Confidence
        ↓
5. Overall Greater Stability
        ↓
   (Loop back to 1)
```

**Result**: As you go around the wheel, your confidence to deliver more reliable changes increases.

---

## Deployment Strategy 1: Blue-Green Deployments

### How It Works
Deploy a new version (Green) alongside the existing version (Blue), then switch traffic when validated.

### Code Cloud Record Store Example
Perfect for fast API service updates

### Benefits

✅ **Instant rollback** if issues are detected

✅ **Isolation** - What happens in one environment doesn't affect the other

✅ **Zero-downtime** deployments

✅ **Full testing** before traffic switch

### When to Use
- API service updates
- Major version changes
- When you need guaranteed rollback capability

---

## Deployment Strategy 2: Canary Deployments

### How It Works
Deploy a new version to a small subset of users and servers first, monitor closely, then gradually expand.

### Code Cloud Record Store Example
Test new order processing logic on 5% of traffic

### Benefits

✅ **Limited blast radius** - Only small percentage affected if issues arise

✅ **Early issue detection** - Catch problems before full rollout

✅ **Gradual validation** - Build confidence incrementally

✅ **Real user feedback** - Test with actual production traffic

### When to Use
- Testing significant logic changes
- Validating performance improvements
- Rolling out potentially risky features

---

## Deployment Strategy 3: Feature Flags

### How It Works
Ship code with feature flags (toggles) built directly into the service that can be turned on/off at will without redeployment.

### Code Cloud Record Store Example
Ship a new product recommendation engine and toggle it for certain users or instances

### Benefits

✅ **Separate deployment from release** - Deploy code safely, enable features later

✅ **Targeted rollouts** - Enable for specific user segments

✅ **Instant kill switch** - Turn off problematic features immediately

✅ **A/B testing** - Compare feature variants

✅ **Progressive rollout** - Gradually increase feature exposure

### When to Use
- New feature development
- A/B testing scenarios
- Gradual feature adoption
- When you want decoupled deployment and release

---

## Critical: Change Without Monitoring = Flying Blind

> **"Monitoring is crucial during and after deployments. Without monitoring, it's your users who tell you if something's wrong - and that should be avoided at all costs."**

### Key Metrics to Monitor During Changes

Track the "LETS" framework:

#### 1. **Latency**
- Is the system responding more slowly?
- Are response times degrading?

#### 2. **Errors**
- Are error rates spiking?
- Are errors increasing after the change?

#### 3. **Traffic**
- Are users able to access the service as before?
- Is traffic distribution normal?

#### 4. **Saturation**
- Are system resources under pressure?
- CPU, memory, disk usage abnormal?

#### 5. **Deployment Progress**
- Is the deployment proceeding as expected?
- Any stuck or stalled processes?

---

## Automatic Rollbacks: Your Best Defense

> **"When in doubt, roll back. Modern deployment systems should detect issues and revert changes before a human even responds."**

### Common Automatic Rollback Triggers

1. **Error Rate Threshold**
   - Errors exceed defined percentage (e.g., >5%)

2. **Response Time Degradation**
   - Latency significantly increases

3. **Health Check Failures**
   - New version fails health checks

4. **Deployment Stalls**
   - Process takes too long or gets stuck

**Important**: These thresholds should match your SLOs and error budgets.

---

## Example: Argo Rollouts Configuration

### Rollout Definition
```yaml
# Rollout calls error rate check template
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  analysis:
    templates:
    - templateName: error-rate-check
    args:
    - name: error-threshold
      value: "5"  # Trigger if >5% errors
```

### Analysis Template
```yaml
# Prometheus query tracks HTTP error rates
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate-check
spec:
  metrics:
  - name: error-rate
    provider:
      prometheus:
        query: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) 
          / 
          sum(rate(http_requests_total[5m])) 
          * 100
    successCondition: result < {{args.error-threshold}}
```

**This operationalizes safety with tooling** - the system automatically stops unsafe rollouts.

---

## Post-Deployment Validation

> **"You don't stop validating after deployment."**

Systematic verification helps catch issues before they impact many users.

### Validation Methods

1. **Smoke Tests**
   - Basic functionality checks
   - Core workflows working?

2. **Integration Tests**
   - Service interaction validation
   - Dependencies communicating properly?

3. **Performance Tests**
   - Real-world behavior testing
   - Load and stress testing

4. **Canary User Testing**
   - Real users on new version
   - Monitor their experience closely

5. **Gradual Traffic Shifting**
   - Slowly increase traffic to new version
   - Watch monitoring data continuously

---

## Change Management Policy Framework

### Different Procedures for Different Change Types

| Change Type | Example Schedule | Approval Required | Postmortem Required |
|-------------|-----------------|-------------------|---------------------|
| **Regular Deployments** | Tuesdays & Thursdays | Maybe not | No (unless incident) |
| **Database Changes** | Sundays | Maybe not | No (unless incident) |
| **Emergency Changes** | Any time | YES | YES |

### Why These Schedules?

**Regular Deployments (Tuesdays/Thursdays)**
- Team is available during the week
- Quick support if issues arise
- Avoid Mondays (start of week) and Fridays (end of week)

**Database Changes (Sundays)**
- Lowest traffic possible
- More time for migrations
- Less user impact if issues occur

**Emergency Changes (Any time)**
- By definition, can't wait
- Requires leadership approval
- Always followed by postmortem

---

## Risk Classification System

### Low-Risk Changes
- Simple configuration updates
- Minor bug fixes
- Documentation updates
- **Approval Process**: Streamlined, possibly automated

### Medium-Risk Changes
- Feature updates
- Dependency upgrades
- Performance optimizations
- **Approval Process**: Designated team approvers

### High-Risk Changes
- Architecture changes
- Major feature launches
- Database schema changes
- **Approval Process**: Senior leadership (SRE organization leader)

---

## Freeze Periods

### Definition
Moments when certain types of changes are frozen due to high criticality of the period.

### Common Freeze Periods

- **Black Friday / Cyber Monday** (retail)
- **Tax season** (financial services)
- **Holiday periods** (customer-facing services)
- **Major sporting events** (streaming/media)
- **End of quarter** (business-critical periods)

### Rationale
During high-traffic, high-value periods, stability takes absolute priority over new features.

### Exception
Emergency security patches may still be allowed with enhanced approval process.

---

## The Complete Change Management Checklist

### Before Change
- [ ] Change classified by risk level
- [ ] Appropriate approvals obtained
- [ ] Deployment strategy selected (blue-green, canary, feature flag)
- [ ] Rollback plan documented
- [ ] Monitoring dashboards prepared
- [ ] Alert thresholds configured
- [ ] Team notified and available

### During Change
- [ ] Monitor error rates continuously
- [ ] Watch latency metrics
- [ ] Track deployment progress
- [ ] Verify traffic distribution
- [ ] Check resource saturation
- [ ] Communicate status to stakeholders

### After Change
- [ ] Run smoke tests
- [ ] Execute integration tests
- [ ] Perform performance validation
- [ ] Monitor for delayed issues (24-48 hours)
- [ ] Document any issues encountered
- [ ] Update runbooks if needed

### If Issues Arise
- [ ] Trigger automatic or manual rollback
- [ ] Communicate incident to stakeholders
- [ ] Begin incident response process
- [ ] Schedule postmortem
- [ ] Update change management process

---

## Key Principles Summary

### 1. Change Is Inevitable
70% of incidents trace back to changes - accept this and manage it

### 2. Frequency Reduces Risk
Frequent, small, validated changes are safer than rare, large changes

### 3. Testing Enables Speed
Better validation → More agility → Maintained stability

### 4. Monitoring Is Non-Negotiable
Never deploy without comprehensive monitoring in place

### 5. Rollbacks Are Normal
Automatic rollbacks should be standard, not exceptional

### 6. Process Contains Damage
Good change management is what separates career-limiting events from learning experiences

### 7. Build The Flywheel
Each successful change builds confidence for the next

---

## Best Practices

✅ **Start small** - Begin with low-risk changes to build confidence

✅ **Automate validation** - Let tools catch issues before humans

✅ **Make rollbacks easy** - Should be one-click or automatic

✅ **Monitor everything** - You can't manage what you don't measure

✅ **Document exceptions** - Track when and why rules were broken

✅ **Learn from incidents** - Every failure improves the process

✅ **Respect freeze periods** - Stability sometimes trumps velocity

✅ **Communicate changes** - Keep stakeholders informed

---

## Connection to Previous Modules

### Simplicity (Module 1)
- Simpler systems are easier to change safely
- Fewer dependencies mean fewer change-related risks
- Complex systems have unpredictable change impacts

### Dependency Management (Module 2)
- Changes to dependencies require careful management
- Blast radius assessment guides change risk classification
- Circuit breakers and fallbacks protect against bad changes

### Change Management (Module 3 - This Module)
- Connects velocity to reliability
- Provides practical frameworks for safe evolution
- Builds on simplicity and dependency principles

---

## Looking Ahead: Capacity Planning

> **"Every deployment, configuration tweak, or new feature impacts the resources our systems consume."**

Next module explores:
- How to plan for growth
- Resource consumption patterns
- Scaling strategies
- Capacity forecasting

Change management ensures changes are safe; capacity planning ensures the system can handle them.

---

## Reflection Questions

1. What's your current deployment frequency? Could you safely increase it?
2. Do you have automatic rollback capabilities?
3. Are your changes small and frequent, or large and rare?
4. Can you deploy without releasing (feature flags)?
5. What monitoring is in place during deployments?
6. Do you have documented freeze periods?
7. What's your risk classification system?
8. How long does a rollback take in your system?

---

## Key Takeaways

1. **70% of incidents come from changes** - This is the main battlefield for reliability

2. **More changes can mean more stability** - When done right with good testing

3. **The gym analogy applies** - Frequent practice reduces injury risk

4. **Multiple deployment strategies exist** - Blue-green, canary, feature flags each serve different needs

5. **Never change without monitoring** - Flying blind leads to user-reported incidents

6. **Automatic rollbacks save careers** - Detect and revert faster than humans can

7. **Process contains damage** - Good change management prevents career-limiting events

8. **Build the flywheel** - Success breeds confidence breeds more success

9. **Respect freeze periods** - Sometimes stability must take priority

10. **Change management is a skill** - It improves with practice and feedback