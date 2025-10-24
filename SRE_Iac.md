# SRE Module: Infrastructure as Code (IaC)

## Overview

Infrastructure as Code (IaC) is the idea of treating your infrastructure the same way you treat your application code. You want to be able to version control it, make it reviewable, and make it repeatable.

> **"The mindset transforms infrastructure from something managed manually to something reliable and automated."**

---

## The Old Way vs. The New Way

### ❌ The Old Days: Manual Infrastructure Management

**The Process**:
1. SSH into servers
2. Edit configs by hand
3. Restart services
4. Hope nothing broke

**The Results**:
- 🔥 Infrastructure drift (servers diverge over time)
- 🔥 Inconsistent systems (no two servers exactly alike)
- 🔥 No record of changes (who changed what? when? why?)
- 🔥 Snowflake servers (unique, fragile, can't be reproduced)
- 🔥 Manual errors
- 🔥 Tribal knowledge
- 🔥 Impossible to audit

---

### ✅ The IaC Approach: Infrastructure as Code

**The Process**:
1. Write changes as code
2. Review them like any other pull request
3. Apply them automatically
4. Track everything in version control

**The Results**:
- ✅ Consistent environments (every system built the same way)
- ✅ Auditable changes (complete change history)
- ✅ Test updates before they go live
- ✅ Reproducible infrastructure
- ✅ Automated deployment
- ✅ Code review process
- ✅ Rollback capability

---

## Why SREs Love Infrastructure as Code

### 1. Goodbye Snowflake Servers
**Problem**: Every server is unique, fragile, irreplaceable
**Solution**: Every system built the same way from code
**Benefit**: No more "don't touch that server, it's special"

### 2. Predictable Disaster Recovery
**Problem**: Rebuilding from memory or outdated docs
**Solution**: Rebuild entire environments directly from code
**Benefit**: Recovery time measured in minutes, not days

### 3. Change Tracking
**Problem**: No idea who changed what or when
**Solution**: Always know who changed what and when
**Benefit**: Full audit trail, accountability, forensics

### 4. Test Before Live
**Problem**: Changes applied directly to production
**Solution**: Test infrastructure updates before production
**Benefit**: Avoid surprises, no late-night firefighting

---

## Popular IaC Tools

### AWS CloudFormation
**What**: AWS-native IaC tool
**Format**: JSON or YAML
**Scope**: AWS only
**Best for**: AWS-exclusive environments

### Terraform
**What**: Multi-cloud IaC tool by HashiCorp
**Format**: HCL (HashiCorp Configuration Language)
**Scope**: Multi-cloud (AWS, Azure, GCP, etc.)
**Best for**: Cloud-agnostic or multi-cloud environments

### Pulumi
**What**: IaC using real programming languages
**Format**: Python, TypeScript, Go, C#, Java
**Scope**: Multi-cloud
**Best for**: Developers who prefer programming languages over DSLs

**Common Capability**: All allow you to:
- Declare infrastructure in files
- Store in Git
- Manage with same discipline as software development

---

## Four IaC Best Practices

### 1. Everything Version Controlled ✅

**Rule**: If it's not in Git, it doesn't exist

**What this means**:
- ❌ No more manual changes in cloud console
- ✅ All infrastructure defined in code
- ✅ All changes committed to version control
- ✅ Full history of every change
- ✅ Ability to roll back any change

**Why it matters**: Manual changes create drift, inconsistency, and mystery

---

### 2. Use Variables, Not Hard-Coded Values ✅

**Problem with hard-coding**:
```hcl
# ❌ Bad: Hard-coded
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = "subnet-12345"
}
```
**Issues**:
- Tied to one environment
- Can't reuse in dev/staging/prod
- Changes are painful

**Better with variables**:
```hcl
# ✅ Good: Variables
variable "environment" {
  type = string
}

variable "instance_type" {
  type = map(string)
  default = {
    dev     = "t2.micro"
    staging = "t2.small"
    prod    = "t2.large"
  }
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type[var.environment]
  subnet_id     = var.subnet_id
}
```

**Benefits**:
- Reusable across environments
- Easy to change
- Environment-specific sizing
- Clear configuration

---

### 3. Embrace Modules ✅

**Rule**: Don't Repeat Yourself (DRY)

**Problem**: Copy-pasting resource definitions
```hcl
# ❌ Bad: Repeated in dev.tf
resource "aws_vpc" "dev" {
  cidr_block = "10.0.0.0/16"
  # ... 50 lines of config
}

# ❌ Bad: Same code repeated in staging.tf
resource "aws_vpc" "staging" {
  cidr_block = "10.1.0.0/16"
  # ... 50 lines of identical config
}

# ❌ Bad: Same code repeated in prod.tf
resource "aws_vpc" "prod" {
  cidr_block = "10.2.0.0/16"
  # ... 50 lines of identical config
}
```

**Solution**: Create reusable modules
```hcl
# ✅ Good: modules/vpc/main.tf
module "vpc" {
  source = "./modules/vpc"
  
  environment = var.environment
  cidr_block  = var.vpc_cidr
  # ... configuration
}

# Then use it everywhere:
# dev.tf
module "dev_vpc" {
  source      = "./modules/vpc"
  environment = "dev"
  cidr_block  = "10.0.0.0/16"
}

# staging.tf
module "staging_vpc" {
  source      = "./modules/vpc"
  environment = "staging"
  cidr_block  = "10.1.0.0/16"
}
```

**Benefits**:
- Write once, use many times
- Single source of truth
- Easier to maintain
- Consistent across environments

---

### 4. Review Everything ✅

**Rule**: Infrastructure is as critical as application code

**Process**:
1. Every change goes through pull request
2. Use `terraform plan` to show what will change
3. Peer review by team members
4. Automated tests catch mistakes
5. Approval required before merge
6. Automated application after approval

**Why it matters**:
- Catches mistakes before production
- Knowledge sharing
- Prevents bad changes
- Creates audit trail
- Forces thoughtful changes

---

## Testing Infrastructure Changes

### IaC Doesn't Remove Need for Testing - It Makes It Easier

### Tool 1: Terraform Plan

**What it does**: Preview changes before applying

**Example**:
```bash
$ terraform plan

Terraform will perform the following actions:

  # aws_instance.web will be created
  + resource "aws_instance" "web" {
      + ami           = "ami-0c55b159cbfafe1f0"
      + instance_type = "t2.micro"
      + subnet_id     = "subnet-12345"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Benefits**:
- See exactly what will change
- No surprises
- Catch mistakes before they're applied

---

### Tool 2: TFLint

**What it does**: Automated linting for Terraform

**Catches**:
- Syntax errors
- Deprecated syntax
- Provider-specific issues
- Best practice violations

**Example**:
```bash
$ tflint

Warning: "t1.micro" is previous generation instance type (aws_instance.web)

Error: "us-east-1" is invalid region (aws_instance.web)
```

---

### Tool 3: TFSec

**What it does**: Security scanning for Terraform

**Catches**:
- Publicly accessible resources
- Unencrypted storage
- Missing security groups
- Insecure configurations
- Compliance violations

**Example**:
```bash
$ tfsec

Result #1 CRITICAL S3 bucket is publicly accessible
─────────────────────────────────────────
  s3.tf:15-20
─────────────────────────────────────────
   15   resource "aws_s3_bucket" "data" {
   16     bucket = "my-data-bucket"
   17     acl    = "public-read"  ← Problem here
   18   }
─────────────────────────────────────────
```

---

## GitHub Actions CI/CD Pipeline Example

### Complete Pipeline for Infrastructure Changes

```yaml
name: Terraform CI/CD

on:
  pull_request:
    paths:
      - 'terraform/**'
  push:
    branches:
      - main

jobs:
  terraform-check:
    runs-on: ubuntu-latest
    steps:
      # Step 1: Checkout code
      - name: Checkout
        uses: actions/checkout@v3

      # Step 2: Setup Terraform
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      # Step 3: Format check
      - name: Terraform Format
        run: terraform fmt -check
        
      # Step 4: Initialize
      - name: Terraform Init
        run: terraform init

      # Step 5: Validate
      - name: Terraform Validate
        run: terraform validate

      # Step 6: Run TFLint
      - name: TFLint
        uses: terraform-linters/setup-tflint@v3
        run: tflint

      # Step 7: Security scan with TFSec
      - name: TFSec Security Scan
        uses: aquasecurity/tfsec-action@v1.0.0
        
      # Step 8: Plan (shows what will change)
      - name: Terraform Plan
        run: terraform plan -out=tfplan

      # Step 9: Post plan as PR comment
      - name: Post Plan to PR
        uses: actions/github-script@v6
        with:
          script: |
            // Post plan output as comment on PR

  terraform-apply:
    needs: terraform-check
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init

      - name: Terraform Apply
        run: terraform apply -auto-approve
```

### What This Pipeline Does

**On Pull Request**:
1. ✅ Format check (code style)
2. ✅ Initialize Terraform
3. ✅ Validate configuration
4. ✅ Run linting (TFLint)
5. ✅ Security scan (TFSec)
6. ✅ Generate plan (show changes)
7. ✅ Post plan as PR comment

**On Merge to Main**:
1. ✅ Run all checks again
2. ✅ Require manual approval (production environment)
3. ✅ Apply changes automatically after approval

**Result**: Nothing goes live without extra set of eyes and automated checks

---

## Policy as Code

### What It Is
Encode guardrails directly into your workflows using tools like:
- **Open Policy Agent (OPA)**
- **HashiCorp Sentinel**

### Example Policies

**Policy 1: No Public S3 Buckets**
```rego
# OPA Policy
deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"
  resource.change.after.acl == "public-read"
  
  msg := sprintf("S3 bucket '%s' cannot be publicly readable", 
                 [resource.name])
}
```

**Policy 2: All EC2 Instances Must Have Backup Tag**
```rego
deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_instance"
  not resource.change.after.tags.backup
  
  msg := sprintf("EC2 instance '%s' missing required 'backup' tag", 
                 [resource.name])
}
```

**Policy 3: Production Changes Require Approval**
```sentinel
import "tfplan"

main = rule {
  all tfplan.resources as _, resources {
    all resources as _, r {
      r.applied.tags.environment != "production" or
      request.approval.status == "approved"
    }
  }
}
```

### Benefits of Policy as Code
- ✅ Automated enforcement
- ✅ Consistent across teams
- ✅ Catches violations before apply
- ✅ Clear, documented rules
- ✅ No manual checking needed

---

## Real-World Scenario: Creating IAM Users

### The Manual Process (❌ Bad)

**Steps**:
1. Log into AWS Console
2. Navigate to IAM
3. Click "Add User"
4. Enter username
5. Select permissions
6. **Forget to enable MFA** ← Common mistake
7. Create user
8. Realize mistake
9. Navigate back to IAM
10. Find the user
11. Enable MFA
12. Set permissions manually
13. **Repeat 50 times** ← Painful, error-prone

**Problems**:
- Slow (15 minutes per user)
- Inconsistent (different users, different configs)
- Error-prone (forget MFA, wrong permissions)
- No audit trail
- Can't reproduce
- Soul-crushing repetitive work

---

### The IaC Process (✅ Good)

**Terraform Code**:
```hcl
# Define users as variable
variable "iam_users" {
  type = list(object({
    name   = string
    groups = list(string)
  }))
  
  default = [
    {
      name   = "alice"
      groups = ["developers", "oncall"]
    },
    {
      name   = "bob"
      groups = ["developers"]
    },
    {
      name   = "charlie"
      groups = ["sre", "oncall"]
    }
    # ... 47 more users
  ]
}

# Create all users with loop
resource "aws_iam_user" "users" {
  for_each = { for user in var.iam_users : user.name => user }
  
  name = each.value.name
  
  tags = {
    environment = "production"
    managed_by  = "terraform"
  }
  
  # Force MFA
  force_destroy = true
}

# Attach group memberships
resource "aws_iam_user_group_membership" "users" {
  for_each = { for user in var.iam_users : user.name => user }
  
  user   = aws_iam_user.users[each.key].name
  groups = each.value.groups
}

# Enforce MFA policy for all users
resource "aws_iam_policy" "enforce_mfa" {
  name = "EnforceMFA"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "DenyAllExceptMFA"
        Effect = "Deny"
        Action = "*"
        Resource = "*"
        Condition = {
          BoolIfExists = {
            "aws:MultiFactorAuthPresent" = "false"
          }
        }
      }
    ]
  })
}

# Attach MFA policy to all users
resource "aws_iam_user_policy_attachment" "mfa" {
  for_each = { for user in var.iam_users : user.name => user }
  
  user       = aws_iam_user.users[each.key].name
  policy_arn = aws_iam_policy.enforce_mfa.arn
}
```

**To create all 50 users**:
```bash
terraform apply
```

**Benefits**:
- ✅ **Fast**: Creates all 50 users in minutes
- ✅ **Consistent**: Every user identical configuration
- ✅ **Correct**: MFA enforced automatically, no forgetting
- ✅ **Auditable**: All changes in Git history
- ✅ **Reproducible**: Can recreate anytime
- ✅ **Testable**: Can test in dev environment first
- ✅ **Reviewable**: Team can review before applying

---

## IaC Benefits Summary

### Consistency
**Manual**: Every server is slightly different
**IaC**: Every server is identical

### Speed
**Manual**: Hours to provision new environment
**IaC**: Minutes to deploy from code

### Reliability
**Manual**: Prone to human error
**IaC**: Automated, repeatable, tested

### Auditability
**Manual**: Who changed what? No idea.
**IaC**: Complete history in Git

### Disaster Recovery
**Manual**: Rebuilding from memory/docs (days/weeks)
**IaC**: Rebuild from code (minutes/hours)

### Testing
**Manual**: Test in production (hope for best)
**IaC**: Test before apply (catch errors early)

### Knowledge Sharing
**Manual**: Tribal knowledge, bus factor of 1
**IaC**: Documented in code, everyone can read

### Scalability
**Manual**: More infrastructure = more manual work
**IaC**: More infrastructure = same code, automated deployment

---

## IaC Best Practices Checklist

### Version Control
- [ ] All infrastructure code in Git
- [ ] No manual changes in console
- [ ] Meaningful commit messages
- [ ] Branch protection enabled

### Code Quality
- [ ] Use variables, not hard-coded values
- [ ] Create reusable modules
- [ ] Follow naming conventions
- [ ] Document complex logic

### Review Process
- [ ] All changes via pull request
- [ ] Terraform plan in PR comments
- [ ] Peer review required
- [ ] Approval before merge

### Testing
- [ ] Run terraform plan
- [ ] Automated linting (TFLint)
- [ ] Security scanning (TFSec)
- [ ] Policy validation (OPA/Sentinel)

### CI/CD Pipeline
- [ ] Automated checks on PR
- [ ] Manual approval for production
- [ ] Automatic apply after approval
- [ ] Rollback capability

### Documentation
- [ ] README explaining structure
- [ ] Module documentation
- [ ] Variable descriptions
- [ ] Examples for common tasks

### Security
- [ ] Secrets in secret manager, not code
- [ ] Enforce encryption
- [ ] Require MFA
- [ ] Follow least privilege

---

## Common IaC Anti-Patterns

### ❌ Manual Changes in Console
**Problem**: Creates drift, bypasses review
**Fix**: All changes through code

### ❌ Hard-Coded Values
**Problem**: Not reusable, environment-specific
**Fix**: Use variables and modules

### ❌ No Code Review
**Problem**: Mistakes reach production
**Fix**: Require PR approval

### ❌ Skipping Testing
**Problem**: Errors found in production
**Fix**: Test in dev/staging first

### ❌ No Version Control
**Problem**: No history, can't rollback
**Fix**: Everything in Git

### ❌ Secrets in Code
**Problem**: Security risk, exposed in Git
**Fix**: Use secret managers (AWS Secrets Manager, HashiCorp Vault)

### ❌ Copy-Paste Engineering
**Problem**: Repeated code, hard to maintain
**Fix**: Create reusable modules

---

## Connection to Previous Modules

### Production Readiness
- IaC enables consistent environments
- Staging can truly match production
- Reproducible deployments

### Change Management
- Infrastructure changes through code review
- Tested before production
- Easy rollback capability

### Incident Management
- Faster recovery (rebuild from code)
- Clear audit trail for forensics
- Consistent recovery procedures

### All Previous Modules
- IaC is foundation for reliability
- Enables automation
- Reduces human error
- Improves consistency

---

## Key Takeaways

1. **IaC treats infrastructure like code** - Version control, review, test

2. **Manual infrastructure doesn't scale** - Inconsistent, error-prone, slow

3. **Everything in version control** - If not in Git, doesn't exist

4. **Use variables, not hard-coding** - Reusable across environments

5. **Embrace modules** - DRY principle for infrastructure

6. **Review everything** - Infrastructure is as critical as application code

7. **Test before apply** - Terraform plan, linting, security scans

8. **Policy as code** - Automated enforcement of guardrails

9. **CI/CD for infrastructure** - Automated checks, manual approval

10. **Disaster recovery made easy** - Rebuild environments from code

---

## Reflection Questions

1. Is all your infrastructure defined in code?
2. Do you make manual changes in the cloud console?
3. Are infrastructure changes reviewed like code changes?
4. Do you use modules to avoid repetition?
5. Do you test infrastructure changes before production?
6. Can you rebuild your entire environment from code?
7. Do you have policies enforcing security requirements?
8. How long would disaster recovery take without IaC?

---

## Final Thought

> **"Infrastructure as Code transforms infrastructure from something managed manually to something reliable and automated."**

Manual infrastructure worked when we had 10 servers. It doesn't work with 10,000.

IaC is how modern engineering teams manage infrastructure at scale - consistently, reliably, and safely.

Treat your infrastructure like your application code. Version control it. Review it. Test it. Automate it.

Your future self (and your on-call rotation) will thank you.