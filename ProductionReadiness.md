Five Whys Example: Website Down for 2 Hours
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