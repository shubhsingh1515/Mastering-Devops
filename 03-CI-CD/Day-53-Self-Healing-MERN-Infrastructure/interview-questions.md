# Day 53 Interview Questions: Self-Healing MERN Infrastructure

## 1. What is the difference between liveness and readiness?

Liveness determines whether an application instance is alive enough to continue running and potentially needs a restart. Readiness determines whether that instance should receive traffic. A readiness failure does not necessarily mean the process should be restarted.

## 2. Why should liveness be lightweight?

Liveness is used to decide whether restarting may help. If it depends on MongoDB or another external service, one dependency outage can make every API instance appear dead and trigger a restart storm. Liveness should usually test the process; readiness should test traffic eligibility and critical dependencies.

## 3. What should `/health` return?

It should return HTTP 200 with a small, non-sensitive response such as `{ "status": "ok" }` when the process can respond. It should not expose credentials, connection strings or internal failure details.

## 4. What should `/ready` return when MongoDB is unavailable?

If MongoDB is critical for the work this instance serves, `/ready` should return HTTP 503 and a safe response such as `not_ready`. The load balancer should stop sending new traffic to that instance. The process may remain running while MongoDB recovers.

## 5. Why can a restart policy be insufficient for production reliability?

A restart policy helps when the process exits. It does not detect every application failure, because a process can remain alive while returning errors or losing dependency access. It also cannot fix bad deployments, invalid configuration, database corruption or external outages. Health checks, load balancing, logs, metrics and alerts are still required.

## 6. What is a restart loop and how do you handle it?

A restart loop is repeated startup, crash and restart behavior. I would inspect exit codes and startup logs, stop treating repeated restarts as recovery, check the image and configuration, apply backoff or pause the rollout, and alert the responsible team. I would preserve evidence before replacing the container.

## 7. Should an API restart when MongoDB is temporarily unavailable?

Not automatically. I would determine whether the API can remain safely running, fail readiness for MongoDB-dependent traffic and let the load balancer isolate the instance. Restarting all replicas does not restore MongoDB and can amplify the outage.

## 8. How do load-balancer health checks support high availability?

The load balancer probes readiness and routes traffic only to instances that report they can serve it. If one replica returns 503 while two return 200, the unhealthy replica is removed from new traffic and the healthy replicas continue serving.

## 9. How do readiness checks protect a rolling deployment?

A new instance can be running before it is initialized or connected to critical dependencies. The load balancer waits for readiness to pass before routing production requests, which prevents users from reaching a partially started version.

## 10. What is graceful shutdown?

Graceful shutdown removes an instance from traffic, stops accepting new work, allows active requests to finish, closes server and dependency connections and exits within a termination deadline. It reduces dropped requests and inconsistent background work during replacement or deployment.

## 11. Should email failure make the whole API unready?

Usually not if email is asynchronous or non-critical to the request. The API can accept the business operation, queue email delivery and alert on the email failure. The classification depends on the workflow contract; checkout or authentication may have different critical dependencies.

## 12. What proves a recovered instance is safe to return to traffic?

Its liveness and readiness checks should pass continuously for a stability window. I would also verify error rate, latency, dependency connectivity, restart count, logs and a safe representative user workflow. Business signals such as checkout success and duplicate-order rate should remain normal.

## 13. How would you answer: one API instance crashes, another loses MongoDB connectivity and a new version is deploying?

I would run multiple replicas behind a load balancer with separate liveness and readiness checks. The crashed instance can be restarted or replaced while healthy replicas serve traffic. The instance that loses MongoDB connectivity should fail readiness and be removed from traffic rather than causing all replicas to restart. I would pause or control the rollout, wait for the new instance to pass readiness, then introduce traffic gradually while monitoring errors, latency, logs and business metrics.

## 14. Why is CPU alone not a health signal?

An API can use normal CPU and memory while returning errors, timing out on MongoDB or failing checkout. Resource metrics should be combined with application health, dependency status, error rate, latency and business outcomes.

## 15. What does mature self-healing look like?

It combines narrowly designed health checks, appropriate restart behavior, traffic isolation, graceful shutdown, observability, backoff and human escalation. Automation handles known failures while engineers investigate bad releases, data problems, security incidents and external outages.
