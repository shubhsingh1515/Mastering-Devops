# Day 65 — Practical Assignment and Assessment

## Exercise: Diagnose a broken MERN deployment

Use a development or staging namespace containing:

```text
mern-api-a1b2c   CrashLoopBackOff
mern-api-c3d4e   Running
mern-api-e5f6g   Pending
```

### Tasks

1. Describe the crashing pod and collect current and previous logs.
2. Record the termination reason and the evidence supporting it.
3. Describe the Pending pod and identify the scheduler failure.
4. Inspect API and MongoDB Services and their EndpointSlices.
5. Compare Service selectors with pod labels.
6. Verify recovery with pod readiness, logs, metrics, and a business request.

### Deliverable

Write an incident note containing:

- User-facing symptom and impact.
- Timeline of observations.
- Root cause and supporting evidence.
- Smallest safe remediation.
- Recovery verification.
- One preventive control.

## Practical assessment: production outage

An e-commerce MERN application reports:

```text
API desired replicas: 5
API ready replicas:   2
Pod A: CrashLoopBackOff
Pod B: OOMKilled
Pod C: Pending
MongoDB Service: Exists
MongoDB endpoints: Empty
Checkout errors: 12%
API p95 latency: 4 seconds
```

Explain your response in five minutes:

1. What is your first priority?
2. Which evidence do you collect for each symptom?
3. How do you diagnose the empty MongoDB endpoints?
4. How do you decide whether to roll back, change capacity, or fix the application?
5. What proves that recovery is complete?

### Expected reasoning

Prioritize the checkout user journey and coordinate incident response. Inspect
previous logs and termination state for the crashing and OOMKilled pods. Read
scheduler events for the Pending pod. Compare MongoDB Service selectors with pod
labels, readiness, ports, and EndpointSlices. Investigate database load before
adding API replicas. Apply a targeted mitigation, then verify ready replicas,
endpoints, error rate, latency, and checkout success.

## Scoring rubric

| Area | Points |
|---|---:|
| Incident prioritization | 20 |
| Container and memory diagnosis | 20 |
| Scheduling and capacity analysis | 20 |
| Service and DNS diagnosis | 20 |
| Recovery verification and prevention | 20 |
| **Total** | **100** |

**90–100:** Strong production troubleshooting judgment.  
**75–89:** Good; refine diagnostic sequencing.  
**60–74:** Review pod states and Service networking.  
**Below 60:** Repeat the exercise after reviewing the runbook.

## Project deliverable

Add [KUBERNETES-TROUBLESHOOTING.md](./KUBERNETES-TROUBLESHOOTING.md) to the
October MERN operations project. Demonstrate at least three controlled failure
scenarios without relying on random pod restarts.

