# Day 58 Assignment: Define MERN Reliability Objectives

## Goal

Create a reliability policy for a production MERN application. The policy must connect user-facing measurements to alerting, deployment decisions and incident response.

## Tasks

1. Define at least four SLOs:
   - API availability
   - API latency
   - HTTP error rate
   - One business operation such as checkout, login or order creation
2. For each SLO, document:
   - The SLI formula
   - The measurement source
   - The time window
   - The target
   - What is excluded, if anything
3. Calculate the monthly error budget for each target.
4. Define warning and critical burn-rate alerts.
5. Define the deployment response when a canary consumes error budget too quickly.
6. Complete the production decision scenario below.

## Production decision scenario

Your service has a 99.9% monthly availability SLO. Halfway through the month, 80% of the error budget has been consumed. A new canary receives 10% of traffic:

```text
Stable version 5xx rate: 0.2%
Canary version 5xx rate: 4.5%
```

Write a response that explains:

- Whether rollout should continue
- Which measurements justify your decision
- Whether to stop, roll back or investigate in place
- How to preserve evidence
- What verification is required before resuming rollout

## Submission checklist

- [ ] `RELIABILITY-OBJECTIVES.md` is complete.
- [ ] Every SLO has a measurable SLI.
- [ ] Error-budget arithmetic is shown.
- [ ] Alert thresholds include an action.
- [ ] Canary policy includes stop and rollback criteria.
- [ ] Business reliability is measured separately from infrastructure health.
