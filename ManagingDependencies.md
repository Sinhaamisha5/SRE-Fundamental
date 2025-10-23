# SRE Module: Managing Dependencies

## Overview

Dependencies are anything your system relies on - third-party libraries, APIs, internal services, or infrastructure. Without careful management, they can transform even simple systems into reliability nightmares.

### Core Principle
> **"You're not just running your own code, you're running everyone else's code."**

Every dependency added increases your risk surface, making dependency management crucial for system reliability.

---

## Why Dependencies Are Risky

### Four Key Coupling Types

1. **Availability Coupling**
   - If a dependency is down, your service may be down too
   - Dependencies can drag your system down with them

2. **Latency Coupling**
   - The slowest dependency determines your overall speed
   - Performance bottlenecks propagate through the system

3. **Cascading Failures**
   - One failure can trigger a domino effect across the entire system
   - Failures spread exponentially without proper safeguards

4. **Capacity Coupling**
   - A dependency under pressure can overwhelm your system
   - Dependencies can overwhelm each other bidirectionally

---

## Mitigation Strategies Overview

Well-managed dependencies minimize risks through:

- **Circuit Breakers** - Stop cascading failures
- **Fallbacks** - Keep core functions working during outages
- **Bulkheads** - Limit failure impact to one part of the system
- **Graceful Degradation** - Preserve core business value when parts are down

---

## Types of Dependencies

### 1. Direct Dependencies
Your component calls another one directly

### 2. Indirect Dependencies
Your component relies on something through a chain of calls

### 3. Runtime Dependencies
Services like APIs or databases needed while your service is running

### 4. Build-Time Dependencies
Libraries, frameworks, and tools you depend on when building or deploying

**Each type presents unique challenges for reliability.**

---

## Understanding Blast Radius

**Blast Radius** = The measure of how widely a failure can spread

### Key Question
*"If a single dependency fails, how much of your system is actually affected?"*

### Assessing Blast Radius

Evaluate these four factors:

1. **Dependent Services**
   - How many systems rely on this component?

2. **Criticality of Dependent Paths**
   - Are these core user journeys or edge cases?

3. **Traffic Volume**
   - How much user activity passes through this dependency?

4. **Recovery Time**
   - How quickly can the system recover if this dependency fails?

Understanding blast radius helps prioritize reliability improvements.

---

## Case Study: Code Cloud Record Store

### Key Dependencies Mapped

**User Interface**
↓
**Web API** (Product, Order, User, Inventory, Notification endpoints)
↓
**Infrastructure Layer:**
- RabbitMQ (message queue)
- Celery workers (background processing)
- Postgres database (data storage)
- Observability tools (Prometheus, Grafana, Loki)

By mapping dependencies, you can better understand potential blast radius and identify which components are most critical to keep healthy.

---

## Dependency Classification Framework

### Critical Dependencies
- **Definition**: Service cannot function without them
- **Examples**: Main database, core API
- **Impact**: Complete service failure if unavailable

### Important Dependencies
- **Definition**: Major features depend on them, but alternatives exist
- **Examples**: Message queues, background workers
- **Impact**: Significant feature degradation

### Non-Critical Dependencies
- **Definition**: Optional features that enhance experience but aren't essential
- **Examples**: Monitoring stack, email notifications
- **Impact**: Reduced functionality or visibility

### External Dependencies
- **Definition**: Third-party dependencies outside your direct control
- **Risk**: No direct control over availability or performance

---

## Code Cloud Record Store: Classification & Mitigation

### Critical Dependencies

**Postgres Database**
- **Classification**: Critical
- **Mitigation**: 
  - Connection pooling
  - Timeouts
  - Read replicas

**API Service**
- **Classification**: Critical
- **Mitigation**:
  - Auto-scaling
  - Load balancing
  - Comprehensive health checks

### Important Dependencies

**RabbitMQ**
- **Classification**: Important
- **Mitigation**:
  - Local queuing fallback
  - Synchronous options

**Celery Workers**
- **Classification**: Important
- **Mitigation**:
  - Task timeouts
  - Dead letter queues

### Non-Critical Dependencies

**Monitoring Stack** (Prometheus, Grafana)
- **Classification**: Non-critical (doesn't affect app functionality)
- **Mitigation**:
  - Buffer metrics locally
  - Reduce polling frequency
  - Adjust sampling rates
  - Fall back to logs

**Email Notifications**
- **Classification**: Non-critical
- **Mitigation**:
  - Queue for later delivery
  - Background processing
  - Retry mechanisms

---

## Strategy 1: Circuit Breakers

### Purpose
Prevent cascading failures by breaking the circuit when a dependency starts failing - like an electric circuit breaker stopping flow to prevent wider damage.

### Three States

1. **CLOSED** (Normal Operation)
   - Everything is normal
   - Calls go through freely

2. **OPEN** (Failure Detected)
   - Too many failures detected
   - Calls are blocked
   - Prevents cascading damage

3. **HALF-OPEN** (Testing Recovery)
   - System tests if dependency has recovered
   - Limited calls allowed through
   - Determines if circuit can close

### Implementation Pattern

```
IF failures cross threshold:
    OPEN the circuit
    USE fallback response
    
WHEN enough time has passed:
    TRY a few requests
    
    IF they succeed:
        CLOSE the circuit
        RESUME normal operation
```

### Popular Libraries
- Resilience4J (Java)
- PyBreaker (Python)

**Benefit**: Fail fast and recover gracefully

---

## Strategy 2: Fallbacks & Graceful Degradation

### Core Concepts

**Graceful Degradation**
- Ability to continue functioning when parts of the system fail

**Partial Availability**
- Provide core functionality even when some components are unavailable

**Functional Core vs. Enhancement Shell**
- Distinguish between must-have and nice-to-have features

**Progressive Enhancement**
- Build systems that add features when resources are available

### Common Fallback Patterns

1. **Return cached data** if a service is unavailable
2. **Offer simplified functionality** or default values
3. **Queue work** for later processing
4. **Trigger manual intervention** as a last resort

**Key Principle**: Degrade gracefully rather than failing completely

---

## Strategy 3: Bulkheads

### Inspiration
Named after watertight compartments in ship design that prevent the entire vessel from sinking if one section is breached.

### Purpose
Isolate components so failures in one area can't sink the entire system.

### Implementation Methods

1. **Separate thread pools** for different dependencies
2. **Deploy critical functions** as independent services
3. **Keep databases or resources** dedicated to specific domains
4. **Partition requests** based on user type or criticality

### Benefits

✅ Prevent resource exhaustion from spreading

✅ Allow partial system functionality during outages

✅ Create clear isolation boundaries

✅ Make systems easier to test and deploy

✅ Improve overall resilience to failures

### Practical Examples

- Separate task queues for different workloads
- Independent connection pools
- Resource limits for each dependency
- Isolated failure domains for different services

**Result**: Failures remain contained, reducing overall blast radius

---

## Real-World Case Study: Amazon S3 Outage (2017)

### What Happened (February 28, 2017)

**Initial Incident**
- Engineer debugging S3 billing system issue in US-East-1
- Executed command to remove small number of servers
- **Typo/incorrect input** removed much larger set than intended

**Critical Systems Affected**
1. **Index Subsystem** - Managing metadata for all S3 objects
2. **Placement System** - Responsible for managing storage for new objects

### Impact

**Downtime**: 3.5 hours in US-East-1 region

**Blast Radius**: Massive
- Amazon EC2 affected
- EBS (Elastic Block Storage) affected
- Lambda services affected
- All dependent on S3 for storage and metadata

**Root Cause**: Removing significant capacity caused systems to require full restart; during restart, S3 couldn't serve requests

### Key Lessons

⚠️ **Even highly engineered systems can fail due to simple mistakes**

### Amazon's Response

**Immediate Improvements:**
- Improved tooling to remove capacity more slowly
- Added safeguards to block large-scale removals

**Long-Term Changes:**
- ✅ Improved operational safety protocols
- ✅ Faster and safer recovery procedures
- ✅ More resilient service health dashboards
- ✅ Stronger dependency and failover management
- ✅ More robust approach to handling critical dependencies

### Takeaway
This incident demonstrates the critical importance of understanding dependency chains and implementing proper safeguards, even in mature systems.

---

## Best Practices Summary

### 1. Map Your Dependencies
- Create visual architecture maps
- Identify all direct and indirect dependencies
- Document runtime vs. build-time dependencies

### 2. Classify by Criticality
- Determine which dependencies are critical vs. nice-to-have
- Assess blast radius for each dependency
- Prioritize mitigation efforts accordingly

### 3. Implement Defense in Depth
- **Circuit Breakers** - Stop the bleeding
- **Fallbacks** - Provide alternative paths
- **Bulkheads** - Contain the damage
- **Graceful Degradation** - Maintain core value

### 4. Design for Failure
- Assume dependencies will fail
- Plan for partial availability
- Test failure scenarios regularly
- Document recovery procedures

### 5. Monitor and Iterate
- Track dependency health metrics
- Measure blast radius over time
- Adjust mitigation strategies based on incidents
- Continuously improve isolation

---

## Key Takeaways

1. **Every dependency increases risk** - Be intentional about what you add

2. **Understand your blast radius** - Know how failures spread through your system

3. **Classify and prioritize** - Not all dependencies deserve the same level of protection

4. **Use multiple strategies** - Circuit breakers, fallbacks, and bulkheads work together

5. **Design for failure** - Assume dependencies will fail and plan accordingly

6. **Simple mistakes cause major outages** - Even giants like Amazon aren't immune

7. **Isolation is protection** - Bulkheads prevent total system failure

8. **Graceful degradation preserves value** - Better to provide limited service than no service

---

## Reflection Questions

1. Have you mapped all your system's dependencies?
2. Do you know the blast radius of your critical dependencies?
3. What happens when your slowest dependency fails?
4. Are you using circuit breakers to prevent cascading failures?
5. Can your system degrade gracefully, or does it fail completely?
6. What's your fallback strategy for external dependencies?
7. Are your critical functions isolated with bulkheads?

---

## Next Steps

The module continues with:
- **Change Management** - How to introduce new changes safely
- How to keep systems stable as they evolve
- Managing the release process for reliability

---

## Connection to Simplicity

Remember from Module 1: **Managing dependencies ties directly to simplicity**

- Every dependency adds complexity
- Complexity increases the blast radius
- The simplest system has the fewest dependencies
- Question whether each dependency is truly necessary

**Before adding a dependency, ask:**
- Can we eliminate this need entirely?
- Is there a simpler alternative?
- What's the blast radius if this fails?
- Do we have the right mitigation strategies?