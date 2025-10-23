# SRE Module: Capacity Planning

## Overview

Capacity planning is the practice of keeping systems reliable as demand grows. It covers how to forecast, provision, and monitor the resources your services need **before users feel the pain**.

### The Black Friday Scenario

**Picture this**: It's Black Friday, your biggest traffic day of the year, and your service goes down.

**Why?** Because no one expected the spike. No one planned for it.

**Problem**: This is **reactive capacity management** - it's too late once the alert fires.

---

## The Critical Question

> **"Do you have enough resources to serve users reliably now and in the future?"**

If not, you either:
- **Overspend** on unused capacity
- **Fail under load** with insufficient capacity

Good planning prevents outages, latency, and wasted money.

---

## Five Components of Capacity Planning

### 1. Resource Management
**What you're using now**
- Track CPU, memory, disk, network, and other resources
- Understand current consumption patterns

### 2. Workload Categorization
**How demand behaves**
- Understand the nature and patterns of demand
- Identify usage characteristics

### 3. Growth Forecasting
**Where demand is headed**
- How it will change over time
- Predict future needs

### 4. Threshold Management
**Setting appropriate capacity limits**
- Set thresholds for different resource types
- Define triggers that activate at certain stages
- When threshold is met → specific action takes place

### 5. Capacity Adjustment
**Automatically scale up or down**
- Add or reduce resources based on needs
- Respond to demand changes

**Together, these build a proactive reliability foundation.**

---

## Benefits of Good Capacity Planning

✅ **Fewer outages** due to resource exhaustion

✅ **Lower cloud bills** by right-sizing infrastructure beforehand

✅ **Support business growth** with adequate capacity as demand varies

✅ **Happier customers** by maintaining expected performance

✅ **Predictable budgeting** for future planning

✅ **Avoid both** over-engineering and expensive failures

---

## Resource Types & Key Metrics

### 1. Compute
**What it is**: Processing capacity for calculations and logic

**Key Metrics**:
- Utilization percentage
- Queue length
- Context switches

---

### 2. Memory
**What it is**: RAM for active data the application is using

**Key Metrics**:
- Memory used (percentage)
- Page faults
- Swap usage

---

### 3. Storage
**What it is**: Disk space for persistent data (data you want there every time you return)

**Key Metrics**:
- Used space (percentage)
- IOPS (Input/Output Operations Per Second)
- Latency
- Throughput

---

### 4. Network
**What it is**: Bandwidth for data transmission between components

**Key Metrics**:
- Bandwidth usage
- Packet loss
- Connection count

---

### 5. Database
**What it is**: Query capacity and connection capacity between components

**Key Metrics**:
- Query throughput
- Execution time
- Connection count

---

### 6. API Rate Limits
**What it is**: External service quotas and connection limits

**Key Metrics**:
- Requests per second
- Error rates
- Latency

---

## Real-World Metric Examples

### Example 1: Postgres Database Disk Space
- **Current**: 72% used
- **Growth**: 5% monthly
- **Insight**: Know when to provision more storage
- **Action**: Plan expansion before hitting critical threshold

### Example 2: FastAPI Memory During Sales Event
- **Peak**: 90% memory usage
- **Insight**: Critical moment indicator
- **Action**: Add memory capacity before next event

### Example 3: Celery Task Queue During Peak Processing
- **Load**: 10x normal load
- **Insight**: Critical capacity need
- **Action**: Scale up workers to handle peak

### Example 4: Loki Log Storage
- **Current**: 70% used
- **Growth**: 8% monthly
- **Insight**: Predictable exhaustion timeline
- **Action**: Schedule storage expansion

> **Key Principle**: These signals help identify when to scale **before failure** is introduced and **before users** are impacted.

---

## Code Cloud Record Store: Metrics Architecture

### Component Observation
The different components that make up the application are "living" - they are independent components with:
- Certain levels of resource needs
- Usage that changes over time

### Monitoring Infrastructure

**Metrics Flow**:
```
Application Components
    ↓ (emit metrics)
Monitoring Tools (Prometheus, Grafana)
    ↓ (feed into)
Dashboards + Alerting Pipelines
    ↓ (trigger)
Automated Capacity Expansion Responses
```

### Critical Insight
> **"Without visibility, there's no capacity that we can plan for."**

Thresholds trigger automated responses to ensure systems always adapt to user needs.

---

## Capacity Forecasting: The Heart of Proactive Planning

### Core Principle
**You don't just watch usage, you predict it.**

### Forecasting Inputs
- Historical data
- Growth patterns
- Workload characteristics

### Outcome
Predict future demand across CPU, memory, storage, network, and more.

---

## Growth Pattern Types

Different workloads follow different patterns - understanding yours is critical for accurate forecasting.

### 1. Linear Growth
**Pattern**: Steady, consistent growth over time
```
Usage
  |    /
  |   /
  |  /
  | /
  |/_________ Time
```
**Example**: Mature product with stable user base growing steadily

---

### 2. Exponential Growth
**Pattern**: Viral growth on an exponential curve
```
Usage
  |         ___/
  |      __/
  |    _/
  |  _/
  |_/________ Time
```
**Example**: Introduction of popular new features, viral adoption

---

### 3. Seasonal Patterns
**Pattern**: Cyclical variations based on time
```
Usage
  |  /\    /\    /\
  | /  \  /  \  /  \
  |/    \/    \/    \_ Time
```
**Example**: Weekend peaks, holiday traffic, tax season, sporting events

---

### 4. Step Functions
**Pattern**: Sudden jumps in demand
```
Usage
  |      _____
  |      |
  |  ____|
  | |
  |_|_________ Time
```
**Example**: Product launches, marketing campaigns, major feature releases

---

### 5. Combination Patterns
**Pattern**: Multiple patterns overlaid
**Reality**: Many large systems exhibit combined patterns

**Example**: Linear baseline + seasonal peaks + occasional step functions

---

## Real Example: Database Size Growth Over 12 Months

Understanding your specific growth pattern enables building accurate scaling strategies.

**Key Insight**: Choose forecasting models based on **your specific workload behavior** - not generic assumptions.

---

## Proactive vs. Reactive Capacity Management

**You need both** to stay in good position for the future while handling today's demands.

### Proactive Capacity Management

**Mindset**: Strategic, looking ahead

**Approach**: Provision before capacity is needed

**Examples**:
- ✅ Accurately forecasting demand
- ✅ Load testing
- ✅ Designing for future growth
- ✅ Scheduling upgrades
- ✅ Planning infrastructure expansions

**When to Use**: When not occupied with immediate reactive actions

---

### Reactive Capacity Management

**Mindset**: Tactical, responding to now

**Approach**: React to real-time conditions

**Examples**:
- ✅ Autoscaling based on usage
- ✅ Graceful degradation under pressure
- ✅ Throttling requests
- ✅ Adding tasks to queues during spikes
- ✅ Responding to unexpected changes

**When to Use**: For real-time unexpected changes and traffic spikes

---

## Reactive Strategy Example: Horizontal Pod Autoscaler (HPA)

### What It Is
A Kubernetes tool that automatically scales pods based on resource utilization

**Classification**: Classic reactive tool

### How It Works
Set thresholds (e.g., 70% CPU) → Automatically add pods as needed

### Configuration Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: fastapi-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: fastapi
  minReplicas: 3        # Start with minimum 3 pods
  maxReplicas: 10       # Never exceed 10 pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Trigger when CPU > 70%
```

### Key Parameters
- **Min Replicas**: 3 pods (baseline capacity)
- **Max Replicas**: 10 pods (capacity cap)
- **Trigger**: CPU > 70%

**Assessment**: Simple but incredibly effective for handling bursty traffic

**Important**: HPA extends automatic response capabilities but **won't replace capacity planning** - you still need proactive forecasting.

---

## Threshold Types: Turning Metrics into Warnings

Proper alerting ensures you have time to address capacity issues - detect early enough to act, but not so early that you overreact.

### 1. Utilization Threshold
**Based on**: Percentage of capacity used for any resource
**Example**: CPU at 85%

### 2. Growth Rate Threshold
**Based on**: Rate of change
**Example**: 10% weekly increase

### 3. Time to Exhaustion
**Based on**: When resources will predictably run out
**Example**: Disk full in 30 days at current growth

### 4. Performance Degradation Threshold
**Based on**: Response time increases
**Example**: Latency doubles under load

### 5. Error Rate Threshold
**Based on**: System errors related to capacity
**Example**: Connection pool exhaustion errors spike

---

## Practical Threshold Examples

### CPU
| Threshold | Level | Action |
|-----------|-------|---------|
| 70% | Warning | Monitor closely |
| 85% | Critical | **Scale out CPU capacity** |

---

### Database Connections
| Threshold | Level | Action |
|-----------|-------|---------|
| 70% | Warning | Monitor pool usage |
| 85% | Critical | **Increase connection pool / Add DB replicas** |

---

### Memory
| Threshold | Level | Action |
|-----------|-------|---------|
| 80% | Warning | Watch for leaks |
| 90% | Critical | **Add more memory** |

---

### Request Queues
| Threshold | Level | Action |
|-----------|-------|---------|
| 60% | Warning | Monitor queue depth |
| 80% | Critical | **Add more worker capacity** |

---

### Disk Space
| Threshold | Level | Action |
|-----------|-------|---------|
| 70% | Warning | Plan expansion |
| 85% | Critical | **Add more storage** |

---

## The Complete Capacity Planning Process

### Step 1: Measure & Monitor
- Track all resource types continuously
- Collect metrics from all components
- Feed into centralized monitoring

### Step 2: Understand Patterns
- Identify growth pattern type (linear, exponential, seasonal, step, combo)
- Analyze historical trends
- Recognize cyclical behaviors

### Step 3: Forecast Demand
- Use appropriate forecasting model for your pattern
- Predict future resource needs
- Account for known events (launches, holidays, etc.)

### Step 4: Set Thresholds
- Define warning levels (typically 70-80%)
- Define critical levels (typically 85-90%)
- Link thresholds to specific actions

### Step 5: Automate Responses
- Configure autoscaling (reactive)
- Schedule capacity additions (proactive)
- Test automated responses

### Step 6: Review & Refine
- Analyze prediction accuracy
- Adjust thresholds based on incidents
- Update forecasting models
- Document learnings

---

## Best Practices

### Do's

✅ **Monitor everything** - You can't plan what you don't measure

✅ **Understand your patterns** - One size doesn't fit all

✅ **Set graduated thresholds** - Warning before critical

✅ **Link thresholds to actions** - Automate the response

✅ **Use both proactive and reactive** - Strategic planning + tactical response

✅ **Load test** - Validate capacity before peak events

✅ **Document decisions** - Why these thresholds? Why this capacity?

✅ **Review regularly** - Patterns change as systems evolve

### Don'ts

❌ **Don't wait for alerts** - Plan before problems arise

❌ **Don't over-provision everywhere** - Costs spiral unnecessarily

❌ **Don't under-provision critical paths** - Users feel the pain

❌ **Don't ignore growth patterns** - Yesterday's data predicts tomorrow

❌ **Don't set and forget** - Continuous review is essential

❌ **Don't rely only on autoscaling** - Reactive tools have limits

❌ **Don't skip load testing** - Assumptions fail under real load

---

## Connection to Previous Modules

### Simplicity (Module 1)
- Simpler systems have more predictable capacity needs
- Fewer dependencies = easier capacity planning
- Complex systems have unpredictable resource consumption

### Dependency Management (Module 2)
- Dependencies consume capacity
- Cascade failures can cause capacity spikes
- Bulkheads help isolate capacity problems

### Change Management (Module 3)
- Every change impacts resource consumption
- Deployments may alter capacity requirements
- Monitor capacity during and after changes

### Capacity Planning (Module 4 - This Module)
- Ensures systems can handle growth
- Prevents resource-based outages
- Optimizes cost and performance

---

## Key Metrics to Remember

### The "Big Four" Resources
1. **Compute** (CPU) - Processing power
2. **Memory** (RAM) - Active data
3. **Storage** (Disk) - Persistent data
4. **Network** (Bandwidth) - Data transmission

### The "Extended Two"
5. **Database** (Connections) - Query capacity
6. **API Limits** (Rate) - External service quotas

**Each resource type has specific metrics that guide capacity decisions.**

---

## Common Pitfalls

### 1. The "We'll Scale When Needed" Trap
**Problem**: Reactive-only approach
**Reality**: Scaling takes time; users suffer first
**Solution**: Proactive forecasting + reactive tools

### 2. The "Linear Assumption" Trap
**Problem**: Assuming all growth is linear
**Reality**: Most systems show complex patterns
**Solution**: Analyze actual pattern, choose appropriate model

### 3. The "Black Friday Every Day" Trap
**Problem**: Over-provisioning for peak at all times
**Reality**: Massive waste during normal periods
**Solution**: Autoscaling + scheduled capacity adjustments

### 4. The "Perfect Forecast" Trap
**Problem**: Spending too much time perfecting predictions
**Reality**: Good enough + quick response beats perfect + slow
**Solution**: 80% accurate forecast + strong reactive tools

### 5. The "Monitoring Blindspot" Trap
**Problem**: Not tracking all resource types
**Reality**: Bottleneck appears in unmonitored resource
**Solution**: Comprehensive monitoring across all resources

---

## Real-World Scenarios

### Scenario 1: E-commerce Platform
**Pattern**: Seasonal (holiday peaks)
**Strategy**: 
- Baseline capacity for normal days
- Pre-provisioned capacity before Black Friday
- Autoscaling for unexpected spikes
- Load testing weeks before peak season

### Scenario 2: Social Media App
**Pattern**: Exponential (viral growth)
**Strategy**:
- Aggressive forecasting
- Quick scaling capabilities
- Database sharding prepared
- Cost monitoring (exponential growth = exponential cost)

### Scenario 3: B2B SaaS
**Pattern**: Linear (steady business growth)
**Strategy**:
- Quarterly capacity reviews
- Gradual infrastructure expansion
- Cost optimization focus
- Customer-based forecasting

### Scenario 4: News Website
**Pattern**: Step functions (breaking news)
**Strategy**:
- High burst capacity
- CDN for static content
- Auto-scaling with high max
- Cost acceptance during events

---

## Capacity Planning Maturity Model

### Level 1: Reactive Only
- Wait for outages
- Manual scaling
- No forecasting
- **Result**: Frequent incidents, high stress

### Level 2: Basic Monitoring
- Track resource usage
- Manual capacity additions
- Simple autoscaling
- **Result**: Fewer incidents, still reactive

### Level 3: Forecasting
- Historical analysis
- Growth predictions
- Proactive provisioning
- **Result**: Predictable capacity, cost control

### Level 4: Automated
- Comprehensive monitoring
- Auto-forecasting tools
- Automated scaling and provisioning
- **Result**: Reliable, efficient, low-touch

### Level 5: Optimized
- ML-based forecasting
- Cost optimization algorithms
- Capacity efficiency metrics
- **Result**: Maximum reliability at minimum cost

**Most organizations should aim for Level 3-4.**

---

## Key Takeaways

1. **Plan before problems arise** - Reactive capacity management is too late

2. **Track all resource types** - CPU, memory, storage, network, database, APIs

3. **Understand your growth pattern** - Linear, exponential, seasonal, step, or combined

4. **Use both proactive and reactive** - Strategic forecasting + tactical autoscaling

5. **Set meaningful thresholds** - Warning (70-80%) and Critical (85-90%)

6. **Link thresholds to actions** - Automate the response to breaches

7. **Monitor continuously** - Without visibility, there's no capacity to plan

8. **Load test regularly** - Validate assumptions before peak events

9. **Review and adjust** - Growth patterns change, thresholds need tuning

10. **Balance cost and reliability** - Neither over-provision nor under-provision

---

## Reflection Questions

1. Do you know your current resource utilization across all types?
2. What growth pattern does your system follow?
3. Have you set thresholds for warnings and critical alerts?
4. What automated actions trigger when thresholds are breached?
5. Are you using both proactive (forecasting) and reactive (autoscaling) approaches?
6. When was your last load test?
7. Can you predict when your current capacity will be exhausted?
8. What's your plan for the next major traffic event?

---

## Looking Ahead: Toil Management

> **"Now we're going to move on to the last lesson of the module where we're going to explore a concept that has surely haunted more than one SRE, and that's the concept of toil."**

**Next module explores:**
- What is toil?
- Why it's the enemy of reliability
- How to identify and eliminate it
- Automating away manual work

Capacity planning ensures you have resources; toil management ensures you spend them wisely.

---

## Final Thoughts

Capacity planning isn't about having infinite resources - it's about having **the right resources at the right time**.

Good capacity planning:
- Prevents Black Friday disasters
- Optimizes cloud costs
- Enables business growth
- Keeps customers happy
- Makes SRE lives bearable

**Start simple**: Track your resources, understand your pattern, set basic thresholds, and iterate from there.