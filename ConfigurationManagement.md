# SRE Module: Configuration Management

## Overview

Code often gets the spotlight, but many outages come from configuration mistakes. Managing configuration carefully is just as important as managing code.

> **"At the end of the day, if done well, configuration is code."**

---

## Why Configuration Management Matters

### The Silent Killer of Reliability

**Shocking Statistic**: Around **60% of outages are caused by configuration issues**

### Why Configs Are Overlooked

- ❌ Harder to debug than code bugs
- ❌ Often overlooked in favor of testing features
- ❌ Not treated with same rigor as application code
- ❌ Skip review and testing processes
- ❌ Pushed straight to production

**Reality for SREs**: You'll spend more time debugging configuration mistakes than actual code bugs.

---

## Three Major Configuration Disasters

### Facebook Global Outage (2021)

**What Happened**: BGP configuration change

**Impact**: 
- 6 hours of global downtime
- All Facebook services unreachable
- Engineers locked out of their own data centers (couldn't fix it!)

**Lesson**: Configuration changes can lock you out of your own systems

---

### Google Metadata Config (2019)

**What Happened**: Bad metadata configuration

**Impact**:
- Broke authentication for every VM worldwide
- Global Google Cloud service disruption

**Lesson**: One configuration affects everything in distributed systems

---

### AWS S3 Outage (2017)

**What Happened**: Single AWS command line typo

**Impact**:
- Took down half of the internet
- Affected thousands of websites and services
- Hours of downtime

**Lesson**: Typos in configuration commands can have massive blast radius

**These aren't edge cases** - they're reminders of how dangerous configuration mistakes can be.

---

## Why Configuration is Deceptively Dangerous

### Unlike Code, Configs Often:

❌ **Skip review processes**
- No pull request
- No code review
- Straight to production

❌ **Skip testing**
- Not included in test suites
- No validation before apply
- Hope for the best

❌ **Have instant, catastrophic impact**
- One bad setting can take down service instantly
- No gradual degradation
- Immediate total failure

❌ **Can lock you out**
- Access rule configs can prevent fixes
- BGP configs can isolate data centers
- Authentication configs can block admins

**SRE Reality**: Config must be treated with the same rigor as code.

---

## Four Common Configuration Problems

### 1. Manual Config Drift

**What It Is**: Configuration diverges across servers over time

**How It Happens**:
```
3:00 AM - Production incident
Engineer: "Quick fix on server-1"
[Makes manual change]
Engineer: "That worked!"
[Goes back to bed]
[Forgets to update server-2, server-3, server-4...]

Result: 
- server-1: timeout = 30s
- server-2: timeout = 10s  
- server-3: timeout = 5s
- server-4: timeout = 60s
```

**Impact**:
- Inconsistent behavior
- Impossible to reproduce issues
- "Works on some servers, not others"
- No one knows correct configuration

---

### 2. Sensitive Data Exposure

**What It Is**: Secrets leaked in configuration files

**Examples**:
```yaml
# ❌ BAD: In public Git repo
database:
  host: db.prod.company.com
  username: admin
  password: SuperSecret123!  # Exposed!
  
api_keys:
  stripe: sk_live_51234567890  # Exposed!
  sendgrid: SG.abcdefg12345  # Exposed!
```

**Impact**:
- Security breaches
- Unauthorized access
- Data theft
- Compliance violations

---

### 3. Environment Inconsistency

**What It Is**: Dev, staging, prod running different settings

**Example**:
```
Development:  timeout = 60s, retry = 5
Staging:      timeout = 30s, retry = 3
Production:   timeout = 10s, retry = 1
```

**Impact**:
- "Works in dev/staging, fails in prod"
- Can't reproduce issues
- Testing doesn't reflect reality
- Surprises in production

---

### 4. Uncontrolled Changes

**What It Is**: Different engineers tweaking values constantly

**Timeline**:
```
Monday:   Engineer A changes timeout to 30s
Tuesday:  Engineer B changes retry to 5
Wednesday: Engineer C changes pool size to 100
Thursday:  Engineer D changes timeout to 60s
Friday:   Nobody knows what's actually live
```

**Impact**:
- Configuration chaos
- No audit trail
- Can't rollback (don't know what to rollback to)
- Tribal knowledge only

**Result**: Systems become incredibly fragile and almost always fail during incidents.

---

## The Solution: Treat Configuration Like Code

### Code Cloud Example

**What They Do**:
- Store monitoring dashboards in Git
- Store environment-specific configs in Git
- Track all changes with version control

**Result**:
- Changes are reviewed
- Changes are tested
- Changes are tracked
- Just like application code

---

## Five Techniques for Safe Configuration Management

### 1. Infrastructure as Code (Previous Lesson)

**What It Is**: Environment settings stored and versioned in Git

**Tools**: Terraform, CloudFormation, Pulumi

**Benefits**:
- ✅ Consistency across environments
- ✅ Version controlled
- ✅ Code review process
- ✅ Rollback capability

---

### 2. Feature Flags

**What It Is**: Deploy code without activating it until ready

**How It Works**:
```python
# Code is deployed but feature is off
if feature_flag('new_recommendation_engine'):
    return new_recommendation(user)
else:
    return old_recommendation(user)
```

**Benefits**:
- ✅ Deploy != Release (separate concerns)
- ✅ Instant rollback (toggle off)
- ✅ A/B testing capability
- ✅ Gradual rollout control
- ✅ No code deployment needed to enable/disable

**Use Cases**:
- Testing new features safely
- Gradual rollouts
- Emergency kill switches
- A/B experiments

---

### 3. Gradual Rollouts

**What It Is**: Expose new configs to small percentage of users

**Process**:
```
1. Deploy to 1% of users
2. Monitor metrics closely
3. If good: expand to 5%
4. If good: expand to 25%
5. If good: expand to 50%
6. If good: expand to 100%

At ANY sign of problems: rollback immediately
```

**Benefits**:
- ✅ Limited blast radius
- ✅ Early problem detection
- ✅ Real user testing
- ✅ Confidence building

**Monitoring During Rollout**:
- Error rates
- Latency
- User complaints
- Business metrics

---

### 4. Version Control for Config

**What It Is**: Every config change tracked in Git

**Requirements**:
- All configs in version control
- Meaningful commit messages
- Code review for changes
- Approval before merge

**Git History Example**:
```
commit abc123
Author: Alice <alice@company.com>
Date: 2025-10-23
Message: Increase database connection pool from 50 to 100

Why: Seeing connection exhaustion during peak traffic
Impact: Allows more concurrent users
Rollback: git revert abc123
```

**Benefits**:
- ✅ Always know who changed what
- ✅ Can rollback any change
- ✅ Audit trail for compliance
- ✅ Blame-free accountability

---

### 5. Environment Promotion

**What It Is**: Step-by-step progression from dev → staging → prod

**The Flow**:
```
Development
    ↓ (passes validation)
Staging
    ↓ (passes integration tests)
Production
```

**Never**: Dev → Prod directly

**Guardrails at Each Stage**:

**Development**:
- Config validation
- Build and testing
- Unit tests
- Linting

**Staging**:
- Container scans
- Integration tests
- Performance tests
- Security scans
- Sometimes manual approval

**Production**:
- Final approval required
- Smoke tests after deployment
- Gradual rollout
- Heavy monitoring

**Result**: By the time changes hit production, they've been tested in realistic environments and validated at every checkpoint.

---

## Environment Promotion Pipeline Example

### GitHub Actions / GitLab CI Pipeline

```yaml
name: Config Promotion Pipeline

stages:
  - validate
  - dev
  - build
  - staging
  - production

# Stage 1: Validate Configuration
validate_config:
  stage: validate
  script:
    - echo "Validating configuration syntax..."
    - yamllint config/*.yaml
    - jsonlint config/*.json
    - python validate_config.py
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

# Stage 2: Deploy to Development
deploy_dev:
  stage: dev
  script:
    - echo "Deploying to development..."
    - kubectl apply -f config/dev/
    - ./run_smoke_tests.sh dev
  environment:
    name: development
  only:
    - main

# Stage 3: Build Container
build_container:
  stage: build
  needs: [deploy_dev]
  script:
    - echo "Building container..."
    - docker build -t app:$CI_COMMIT_SHA .
    - docker scan app:$CI_COMMIT_SHA  # Security scan
    - docker push app:$CI_COMMIT_SHA
  only:
    - main

# Stage 4: Deploy to Staging
deploy_staging:
  stage: staging
  needs: [build_container]
  script:
    - echo "Deploying to staging..."
    - kubectl apply -f config/staging/
    - kubectl set image deployment/app app=app:$CI_COMMIT_SHA
    - ./run_integration_tests.sh staging
    - ./run_performance_tests.sh staging
  environment:
    name: staging
  only:
    - main

# Stage 5: Deploy to Production (Manual Approval)
deploy_production:
  stage: production
  needs: [deploy_staging]
  script:
    - echo "Deploying to production..."
    - kubectl apply -f config/production/
    - kubectl set image deployment/app app=app:$CI_COMMIT_SHA
    - ./gradual_rollout.sh  # Canary deployment
    - ./run_smoke_tests.sh production
  environment:
    name: production
  when: manual  # Requires manual approval
  only:
    - main
```

### Key Features

**Each stage depends on previous**:
- Validation must pass before dev
- Dev must pass before build
- Build must pass before staging
- Staging must pass before production

**Automatic gates**:
- Syntax validation
- Security scans
- Integration tests
- Performance tests

**Manual gate at production**:
- Human approval required
- Final review checkpoint

---

## Environment-Specific Configurations

### The Problem
Different environments need different settings, but code should be the same.

### Docker Compose Example

**Development** (docker-compose.dev.yml):
```yaml
version: '3'
services:
  api:
    image: myapp:latest
    environment:
      - DATABASE_URL=postgresql://localhost:5432/dev_db
      - REDIS_URL=redis://localhost:6379
      - LOG_LEVEL=DEBUG
      - TIMEOUT=60
      - RETRY_ATTEMPTS=5
    env_file:
      - .env.dev
```

**Staging** (docker-compose.staging.yml):
```yaml
version: '3'
services:
  api:
    image: myapp:latest
    environment:
      - DATABASE_URL=postgresql://staging-db.company.com:5432/staging_db
      - REDIS_URL=redis://staging-redis.company.com:6379
      - LOG_LEVEL=INFO
      - TIMEOUT=30
      - RETRY_ATTEMPTS=3
    env_file:
      - .env.staging
```

**Production** (docker-compose.prod.yml):
```yaml
version: '3'
services:
  api:
    image: myapp:latest
    environment:
      - DATABASE_URL=postgresql://prod-db.company.com:5432/prod_db
      - REDIS_URL=redis://prod-redis.company.com:6379
      - LOG_LEVEL=WARNING
      - TIMEOUT=10
      - RETRY_ATTEMPTS=1
    env_file:
      - .env.production
```

**Run specific environment**:
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Staging
docker-compose -f docker-compose.yml -f docker-compose.staging.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### Benefits

✅ **Realistic settings** at each stage
- Dev: Generous timeouts for debugging
- Staging: Production-like settings
- Prod: Optimized for performance

✅ **Avoid mixing configs** across environments
- Can't accidentally use dev settings in prod
- Each environment self-contained

✅ **Test with appropriate settings**
- Dev tests with dev configs
- Staging tests with prod-like configs
- Prod runs with prod configs

✅ **Easy to manage**
- One file per environment
- Version controlled
- Clear what's different

---

## Configuration Management Best Practices

### Version Control Everything
- [ ] All configs in Git
- [ ] No manual changes
- [ ] Meaningful commit messages
- [ ] Code review required

### Environment Parity with Differences
- [ ] Same structure across environments
- [ ] Only values differ (URLs, timeouts, etc.)
- [ ] Document what differs and why

### Secrets Management
- [ ] Never commit secrets to Git
- [ ] Use secret managers (AWS Secrets Manager, HashiCorp Vault)
- [ ] Rotate secrets regularly
- [ ] Audit secret access

### Validation
- [ ] Syntax validation in CI/CD
- [ ] Schema validation
- [ ] Policy validation (no public access, etc.)
- [ ] Test configs before deployment

### Gradual Rollouts
- [ ] Start with small percentage
- [ ] Monitor closely
- [ ] Expand gradually
- [ ] Rollback immediately if issues

### Documentation
- [ ] Document what each config does
- [ ] Explain why values are set
- [ ] Provide examples
- [ ] Keep updated

---

## Configuration Anti-Patterns

### ❌ Manual Configuration Changes
**Problem**: No audit trail, can't reproduce
**Fix**: All changes through code/version control

### ❌ Secrets in Git
**Problem**: Security risk, exposed forever
**Fix**: Use secret managers

### ❌ Hardcoded Values in Code
**Problem**: Need code deployment to change config
**Fix**: Externalize configuration

### ❌ No Environment Promotion
**Problem**: Untested configs in production
**Fix**: Dev → Staging → Prod pipeline

### ❌ No Rollback Plan
**Problem**: Stuck with bad config
**Fix**: Version control enables instant rollback

### ❌ Configuration Sprawl
**Problem**: Configs scattered everywhere
**Fix**: Centralized configuration management

### ❌ No Validation
**Problem**: Broken configs reach production
**Fix**: Automated validation in pipeline

---

## The 60% Statistic

### Configuration Issues Cause 60% of Outages

**Why?**
- Overlooked in testing
- Skip review processes
- Pushed directly to production
- Harder to debug than code
- Affect multiple services simultaneously

**Solution**: Treat configuration with same rigor as code

---

## Configuration Management Checklist

Before deploying any configuration change:

### Pre-Deployment
- [ ] Configuration in version control?
- [ ] Code review completed?
- [ ] Syntax validation passed?
- [ ] Security scan passed?
- [ ] No secrets in code?
- [ ] Tested in dev environment?
- [ ] Tested in staging environment?

### Deployment
- [ ] Gradual rollout plan ready?
- [ ] Monitoring in place?
- [ ] Rollback plan tested?
- [ ] Team notified?
- [ ] Change documented?

### Post-Deployment
- [ ] Monitoring metrics normal?
- [ ] No error rate increase?
- [ ] User reports normal?
- [ ] Documentation updated?
- [ ] Lessons learned captured?

---

## Connection to Previous Modules

### Infrastructure as Code
- Configuration is part of IaC
- Same tools (Terraform, etc.)
- Same principles (version control, review)

### Change Management
- Configuration changes are changes
- Need same safeguards
- Gradual rollouts apply

### Incident Management
- 60% of incidents are config-related
- Config mistakes harder to debug
- Good config management prevents incidents

### Production Readiness
- Config validation is part of readiness
- Environment parity requirement
- Rollback plans essential

---

## Key Takeaways

1. **60% of outages are configuration-related** - Not code bugs

2. **Major disasters from config mistakes** - Facebook, Google, AWS all hit

3. **Configuration is deceptively dangerous** - Skips review, instant impact

4. **Four common problems** - Drift, exposed secrets, inconsistency, uncontrolled changes

5. **Treat config like code** - Version control, review, test

6. **Five safe techniques** - IaC, feature flags, gradual rollouts, version control, environment promotion

7. **Environment promotion is critical** - Dev → Staging → Prod with gates

8. **Environment-specific configs** - Same structure, different values

9. **Never skip validation** - Catch errors before production

10. **SREs spend more time on config bugs** - Than code bugs

---

## Reflection Questions

1. Are all your configurations in version control?
2. Do configuration changes go through code review?
3. Do you have environment-specific config files?
4. Can you roll back a configuration change instantly?
5. Do you validate configs before deployment?
6. Are your secrets managed securely?
7. Do you test configs in staging before production?
8. How long would it take to find which config changed?

---

## Final Thought

> **"At the end of the day, if done well, configuration is code."**

The same discipline applies:
- Version control
- Code review
- Testing
- Gradual rollouts
- Monitoring
- Rollback capability

60% of outages are configuration-related because configuration is often overlooked.

Stop overlooking it. Treat configuration with the respect (and rigor) it deserves.

Your production systems will thank you.