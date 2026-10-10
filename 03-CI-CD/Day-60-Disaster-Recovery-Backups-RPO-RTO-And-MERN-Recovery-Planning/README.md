# 🚀 Day 60 — Disaster Recovery, Backups, RPO/RTO & MERN Recovery Planning
**Phase 3: Production Operations**
**Focus:** Disaster recovery, MongoDB backups, RPO/RTO, restore testing, incident preparedness, and DevOps interview preparation

Yesterday, **Day 59**, you learned how to manage incidents, perform RCA, and create blameless postmortems.

Today we answer a more serious question:

> **What happens when recovery from the normal failure path isn't enough?**
For example:

```
MongoDB becomes corrupted
        ↓
Primary infrastructure is lost
        ↓
Region becomes unavailable
        ↓
Critical data is deleted
        ↓
Can we recover?
```

A production DevOps engineer must plan for this **before** the disaster happens.

---

# 1. 🎯 Today's Objectives
By the end of today's lesson, you should be able to:

- Explain backup vs disaster recovery.
- Explain RPO and RTO.
- Design a backup strategy for a MERN application.
- Understand why backups must be tested through restoration.
- Distinguish high availability from disaster recovery.
- Design a basic MongoDB recovery plan.
- Explain disaster recovery decisions in interviews.
- Create a practical DR plan for your cumulative MERN project.

---

# 2. 🧠 High Availability ≠ Disaster Recovery
You already built:

```
                 Load Balancer
                  /    |    \
                 /     |     \
              API-1  API-2  API-3
                         |
                      MongoDB
```

High availability helps when:

```
API-2 fails
```

or:

```
One server disappears
```

But imagine:

```
Entire production region
        ↓
Unavailable
```

Your load-balanced API instances won't help if everything lives in the same failed environment.

That's where **disaster recovery** comes in.

---

# 3. 🛡️ What Is Disaster Recovery?
Disaster recovery is the ability to restore critical services and data after a major failure.

Potential disasters:

```
Infrastructure failure
Region outage
Database corruption
Accidental deletion
Ransomware/security incident
Bad deployment
Human error
Critical dependency failure
```

The goal isn't:

> "Nothing will ever fail."
The goal is:

> **"If something catastrophic happens, we can recover within an acceptable timeframe and with an acceptable amount of data loss."**

---

# 4. 💾 Backup vs Recovery
A backup is:

> **A recoverable copy of data.**
Recovery is:

> **The process of restoring usable service/data from available resources.**
You can have:

```
Daily backups
```

and still have poor disaster recovery if:

```
Nobody knows how to restore them.
```

That's why:

> **A backup that has never been restored is an assumption, not proof of recoverability.**

---

# 5. ⏱️ RPO — Recovery Point Objective
RPO answers:

> **How much data can we afford to lose?**
Example:

```
RPO = 1 hour
```

means the organization is prepared for approximately up to an hour's worth of data loss, depending on the recovery design.

For an e-commerce application:

```
Order data
```

may require a much tighter RPO than:

```
Application logs
```

---

# 6. ⏱️ RTO — Recovery Time Objective
RTO answers:

> **How quickly must the service be restored?**
Example:

```
RTO = 30 minutes
```

means the recovery plan should aim to restore the service within that target.

So:

```
RPO → How much data can we lose?

RTO → How long can we be unavailable?
```

This distinction is extremely common in DevOps interviews.

---

# 7. 🧮 RPO Example
Suppose your MongoDB backup runs every:

```
6 hours
```

A disaster occurs:

```
14:00
```

The latest usable backup is:

```
08:00
```

Potential data-loss window:

```
14:00 - 08:00
= 6 hours
```

So your backup frequency may not satisfy an RPO of:

```
1 hour
```

This is a key insight:

> **RPO determines how frequently and how reliably data must be captured.**

---

# 8. 🧮 RTO Example
Suppose:

```
RTO = 30 minutes
```

Your recovery process takes:

```
Database restore       20 min
Infrastructure setup   25 min
Application deployment 15 min
DNS/config changes     10 min
```

Total:

```
70 minutes
```

You do **not** have a 30-minute RTO.

Your documented recovery process is incompatible with your objective.

This is why DR must be **tested**, not merely documented.

---

# 9. 🛒 MERN Disaster Recovery Architecture
A basic production design:

```
                USERS
                  |
                  ▼
             DNS / CDN
                  |
                  ▼
            Load Balancer
                  |
           ┌──────┴──────┐
           ▼             ▼
         API-1         API-2
           |             |
           └──────┬──────┘
                  |
               MongoDB
                  |
            Backups / DR
                  |
        ┌─────────┴─────────┐
        ▼                   ▼
   Recent backups       Recovery copy
```

Your application images and infrastructure configuration should also be reproducible.

You don't want:

```
Backup database
+
Manually rebuild everything
```

if your RTO is short.

---

# 10. 🧱 Back Up More Than MongoDB
A common beginner mistake is:

> "We have database backups, so we're covered."
Your production system contains:

```
Application code
Database
Configuration
Secrets references
Infrastructure definitions
CI/CD configuration
Monitoring configuration
DNS configuration
Object storage
User uploads
```

You need to understand what must be recoverable.

A useful principle:

> **Anything required to recreate production should either be backed up or reproducible from source/configuration.**

---

# 11. 🏗️ Infrastructure as Code Helps DR
Suppose your infrastructure is manually configured.

Recovery becomes:

```
Find server
Configure server
Install dependencies
Configure networking
Configure monitoring
Configure application
Configure secrets
...
```

Slow and error-prone.

With Infrastructure as Code:

```
Git
 ↓
Infrastructure configuration
 ↓
Provision
 ↓
Deploy
 ↓
Recover
```

This can dramatically reduce recovery effort.

That's another reason DevOps isn't just about CI/CD.

---

# 12. 🔐 Backups Must Be Protected
Backups themselves are valuable targets.

Imagine:

```
Production database
       ↓
Backup
       ↓
Attacker deletes both
```

Now your "backup strategy" isn't very useful.

Consider:

```
Access controls
Encryption
Retention policies
Separate credentials
Separate environment/account
Immutable or protected copies
Audit logging
```

The exact controls depend on your platform and threat model.

---

# 13. 📦 Backup Types
Conceptually, you may encounter:

### Full backup
Complete dataset.

```
Database
 ↓
Full copy
```

### Incremental backup
Captures changes since an earlier backup point.

```
Full
 ↓
Changes
 ↓
Changes
 ↓
Changes
```

### Continuous / point-in-time recovery
Allows recovery toward a specific point using a base backup plus recorded changes.

For production databases, point-in-time recovery can provide much tighter recovery objectives than occasional snapshots alone.

---

# 14. 🎯 Why Point-in-Time Recovery Matters
Suppose someone accidentally executes:

```
DELETE FROM ...
```

or performs a destructive MongoDB operation.

The latest nightly backup might be:

```
23 hours old
```

Restoring it could lose almost a day's worth of legitimate transactions.

With point-in-time recovery, you may be able to restore the database to a point immediately before the destructive operation, depending on your database setup and retention.

This is much more powerful than:

> "We have yesterday's backup."

---

# 15. 🧪 Restore Testing
This is one of today's most important ideas.

Imagine your team says:

> "We have backups."
Ask:

> **"When was the last successful restore test?"**
If the answer is:

> "Never."
you don't know whether recovery actually works.

A restore test should validate:

```
Backup exists
 ↓
Backup can be accessed
 ↓
Restore succeeds
 ↓
Data is usable
 ↓
Application connects
 ↓
Application works
 ↓
Recovery time meets target
```

---

# 16. 🔄 Disaster Recovery Drill
A practical DR exercise:

```
Simulate production database loss
          ↓
Provision recovery environment
          ↓
Restore database
          ↓
Deploy MERN application
          ↓
Configure connectivity
          ↓
Run smoke tests
          ↓
Verify critical workflows
          ↓
Measure recovery time
```

Record:

```
Actual RTO
Actual RPO
Problems encountered
Missing dependencies
Manual steps
```

Then improve the runbook.

---

# 17. 🧠 Backup ≠ Replication
This distinction matters.

Replication:

```
Primary DB
    ↓
Replica
```

can improve availability.

But if an operator accidentally deletes data:

```
Primary
   ↓
DELETE
   ↓
Replica
   ↓
DELETE replicated
```

Replication doesn't necessarily protect you from logical mistakes.

Backups and point-in-time recovery address a different problem.

Interview takeaway:

> **Replication helps availability; backups help recoverability.**
You often need both.

---

# 18. 🎤 Interview Question

> **"What is the difference between RPO and RTO?"**
Strong answer:

> "RPO is the maximum acceptable amount of data loss measured in time, while RTO is the maximum acceptable time to restore the service. RPO influences backup and replication strategy; RTO influences how quickly and automatically the infrastructure and application can be recovered."

---

# 19. 🎤 Interview Question

> **"Are database replicas a replacement for backups?"**
Strong answer:

> "No. Replicas primarily improve availability and can reduce recovery time, but they may replicate accidental deletes, corruption, or malicious changes. A proper backup and recovery strategy provides additional protection against those scenarios."

---

# 20. 🎤 Interview Scenario

> **"Your MongoDB production database is accidentally deleted. What do you do?"**
Strong answer:

> "I'd first stop further destructive activity and assess the scope. I'd preserve evidence where relevant, identify the latest valid recovery point, and initiate the documented restore procedure. I'd restore into a controlled environment first where possible, validate data integrity and critical application workflows, then redirect production traffic when recovery is verified. Afterward I'd investigate the cause and improve access controls, backup protection, and recovery procedures."
Notice:

```
Don't panic.
Don't immediately overwrite the recovery environment.
Don't assume the backup works.
```

---

# 21. 🔁 Review Questions From Earlier Topics

### Q1 — Incident Management
What should happen before a detailed RCA during a major outage?

**Answer:** Reduce user impact and restore service through safe mitigation.

### Q2 — SLOs
What is an error budget?

**Answer:** The amount of unreliability permitted by an SLO.

### Q3 — Burn Rate
Why is a 20× burn rate concerning?

**Answer:** The service is consuming its allowed reliability budget much faster than intended.

### Q4 — Performance
If MongoDB is saturated while API CPU is low, should you automatically add API replicas?

**Answer:** No. Investigate and address the database bottleneck first.

### Q5 — High Availability
What does a load balancer provide when one API instance fails?

**Answer:** It can route traffic to healthy instances, improving availability.

---

# 22. 📝 Today's Quiz

### Q1
What does RPO measure?

A. Maximum acceptable data loss
B. Maximum CPU usage
C. Maximum number of servers
D. Deployment duration

### Q2
What does RTO measure?

A. Maximum acceptable recovery time
B. Database size
C. Request rate
D. Error percentage

### Q3
Are backups sufficient if you've never tested restoring them?

A. Yes
B. No
C. Only for development
D. Only if they are compressed

### Q4
What is a major difference between replication and backup?

A. Replication improves availability; backups provide recovery points
B. They are identical
C. Backups cannot contain data
D. Replication prevents all data loss

### Q5
Which situation is point-in-time recovery especially useful for?

A. Recovering from an accidental destructive database change
B. Increasing CPU
C. Building React assets
D. Scaling API replicas

### Q6
Why use Infrastructure as Code in DR?

A. To make infrastructure reproducible
B. To replace databases
C. To remove monitoring
D. To prevent all outages

### Q7
Which is a backup security control?

A. Restricted access
B. Public credentials
C. Shared admin passwords
D. No encryption

### Q8
Your RTO is 30 minutes but your tested recovery takes 90 minutes. What does that indicate?

A. Your recovery process doesn't meet the objective
B. The RTO doesn't matter
C. The database is healthy
D. Monitoring is unnecessary

### Q9
What should you verify after restoring a database?

A. Data integrity and application functionality
B. Only that files exist
C. Only CPU usage
D. Only DNS

### Q10
Which statement is correct?

A. Backups and replication solve different reliability problems
B. Replication makes backups unnecessary
C. Backups eliminate the need for HA
D. RTO and RPO mean the same thing

---

## ✅ Answer Key

```
1 → A
2 → A
3 → B
4 → A
5 → A
6 → A
7 → A
8 → A
9 → A
10 → A
```

### Score
ScoreResult9–10🟢 Excellent7–8🟡 Good5–6🟠 Review DR concepts<5🔴 Revisit today's lesson

---

# 23. 💻 Practical Exercise — Define Your MERN DR Policy
Create:

```
DISASTER-RECOVERY.md
```

Define:

```
Service:
MERN Production Platform

Critical data:

RPO:

RTO:

Backup frequency:

Backup retention:

Backup location:

Backup security:

Restore procedure:

Restore validation:

Failover strategy:

Communication process:

Recovery owner:
```

Then answer:

> **What would happen if the production region disappeared completely?**
Your answer should not depend on manually rebuilding infrastructure from memory.

---

# 24. 🧪 Practical Assessment — Disaster Scenario
Your production MERN system has:

```
RPO = 1 hour
RTO = 30 minutes
```

At:

```
14:00
```

your production database is lost.

Latest recoverable database state:

```
13:45
```

Recovery test shows:

```
Database restore:       8 min
Infrastructure:          7 min
Application deployment:  5 min
Validation:              6 min
Traffic switch:          3 min
```

### Question 1 — RPO
Potential data loss:

```
14:00 - 13:45
= 15 minutes
```

This is within the:

```
1-hour RPO
```

assuming the 13:45 recovery point is valid.

### Question 2 — RTO
Total recovery time:

```
8 + 7 + 5 + 6 + 3
= 29 minutes
```

This meets the:

```
30-minute RTO
```

with only one minute of margin.

### Question 3
What should concern you?

**The recovery process has very little margin.**

A small unexpected delay could cause an RTO breach.

You might improve:

```
Automation
Infrastructure provisioning
Restore process
Validation
Traffic switching
```

---

# 25. 🏆 Monthly Cumulative Project — Disaster Recovery Milestone
Your cumulative MERN DevOps project now includes:

```
Reliability
├── SLIs
├── SLOs
├── Error budgets
├── Burn-rate monitoring
└── Reliability-driven deployment

Incident Management
├── Detection
├── Mitigation
├── Recovery
├── RCA
├── Postmortems
└── Corrective actions

Disaster Recovery
├── Backup strategy
├── Restore strategy
├── RPO
├── RTO
├── Recovery environment
├── Restore testing
└── DR runbook
```

### New deliverable
Create:

```
DR-RUNBOOK.md
```

It should contain:

```
1. Disaster declaration criteria
2. Incident roles
3. RPO/RTO
4. Backup locations
5. Restore procedure
6. Infrastructure recovery
7. Application recovery
8. Database validation
9. Smoke tests
10. Traffic restoration
11. Communication
12. Post-recovery verification
13. Lessons learned
```

---

# 26. 🎯 Final Interview Challenge
Answer aloud:

> **"Design a disaster recovery strategy for a production MERN application."**
A strong answer:

> "I'd first define business-driven RPO and RTO targets. I'd protect MongoDB with an appropriate backup and point-in-time recovery strategy, while also using replication where needed for availability. Application and infrastructure configuration would be reproducible through source control and Infrastructure as Code. Backups would be protected with appropriate access controls and retention. I'd maintain a documented recovery environment and regularly test restores rather than assuming backups work. During a disaster I'd declare the incident, restore infrastructure and data according to the runbook, validate data integrity and critical workflows, switch traffic when safe, and measure actual RPO and RTO against the objectives."

---

# 🧠 Day 60 Summary
The DR model to remember:

```
             DISASTER
                 │
                 ▼
              DECLARE
                 │
                 ▼
              ASSESS
                 │
        ┌────────┴────────┐
        ▼                 ▼
   Recover Infra      Recover Data
        │                 │
        └────────┬────────┘
                 ▼
             Validate
                 │
                 ▼
          Restore Traffic
                 │
                 ▼
              Monitor
                 │
                 ▼
              Improve
```

And the two numbers:

```
RPO
↓
How much data can we afford to lose?

RTO
↓
How long can we afford to be unavailable?
```

### ⭐ Interview takeaway

> **A disaster recovery strategy is only real when you can repeatedly demonstrate that the system can recover within its RPO and RTO.**
For your MERN project, think beyond:

```
"Do we have backups?"
```

Ask:

```
Can we restore them?
How quickly?
To what point?
Is the data valid?
Can the application run?
Can users safely return?
Have we tested it?
```

That is the difference between **having backups** and **having disaster recovery**.