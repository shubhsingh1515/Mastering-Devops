# Day 59 Quiz — Incident Management, RCA and Postmortems

Answer each question before checking the key.

## Questions

1. What should be optimized first during an active incident: root-cause certainty or reduction of user impact?
2. Give one example of a mitigation that does not prove root cause.
3. Why should incident timestamps use UTC?
4. What is the purpose of an incident commander?
5. Name two business SLIs for a MERN commerce application.
6. What is the difference between a symptom and a root cause?
7. Why should a postmortem include contributing factors?
8. Calculate the burn rate when the observed error rate is 2% and the permitted error rate is 0.1%.
9. What makes a corrective action measurable?
10. Why should a rollback be followed by verification?

## Answer key

1. Reduction of user impact. Investigation continues after a safe mitigation.
2. Rolling back a release may restore service without explaining why it failed.
3. UTC gives distributed teams and systems one unambiguous timeline.
4. To coordinate decisions, roles, communication and next actions.
5. Checkout success, payment success, order creation, login success or cart
   completion.
6. A symptom is an observed effect; a root cause is the causal mechanism that
   produced it.
7. They expose conditions that increased likelihood, severity or detection time.
8. `2% / 0.1% = 20x`.
9. It has an owner, due date and observable success condition.
10. The rollback command succeeding does not prove users recovered; metrics must
    confirm the original impact has returned to an acceptable level.

## Score

```text
9–10: Ready to lead a basic incident review
7–8:  Good foundation; revisit timeline and action quality
0–6:  Re-read the incident workflow and complete the assignment
```

