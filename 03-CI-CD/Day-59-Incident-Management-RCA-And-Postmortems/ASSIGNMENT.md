# Day 59 Assignment — Write a Blameless RCA

## Goal

Practice taking a MERN production incident from detection through prevention.
Your submission must separate immediate mitigation from root-cause analysis and
must connect corrective actions to measurable reliability improvements.

## Scenario

Release `v15` introduced a checkout calculation change. Ten minutes after the
deployment:

```text
Checkout success: 99.1% -> 78.0%
API 5xx rate:     0.2%  -> 8.4%
API p95 latency:  420ms -> 3.8s
MongoDB CPU:      58%   -> 96%
```

The new code performs an unbounded product lookup for every cart item. The
database has no compound index matching the new filter. A rollback restores
checkout success within eight minutes.

## Required work

1. Declare the incident severity and explain the user impact.
2. Write the first five incident-channel updates.
3. Create a minute-by-minute timeline using UTC timestamps.
4. Identify the mitigation and the verification signals.
5. Use Five Whys to reach a system-level root cause.
6. List at least three contributing factors without assigning blame.
7. Complete [POSTMORTEM-TEMPLATE.md](./POSTMORTEM-TEMPLATE.md) for this scenario.
8. Create corrective actions with an owner, due date, priority and success metric.
9. Explain how the incident consumed the checkout and availability error budgets.

## Acceptance criteria

- [ ] Symptoms, root cause and contributing factors are distinct.
- [ ] The rollback decision is supported by measurements.
- [ ] Evidence preservation is included before cleanup.
- [ ] Actions improve tests, observability or deployment safety.
- [ ] Every action has an owner and measurable completion condition.
- [ ] The postmortem is blameless and avoids personal criticism.

