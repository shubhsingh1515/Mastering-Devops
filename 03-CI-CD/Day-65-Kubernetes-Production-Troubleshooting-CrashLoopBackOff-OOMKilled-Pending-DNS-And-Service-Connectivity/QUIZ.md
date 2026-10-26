# Day 65 — Kubernetes Troubleshooting Quiz

## Questions

1. What does `CrashLoopBackOff` indicate?  
   A. A container repeatedly failed and Kubernetes is delaying restart attempts.  
   B. A cluster backup completed.  
   C. The pod is guaranteed healthy.  
   D. The Service has no selector.

2. What does `OOMKilled` describe?  
   A. A DNS failure.  
   B. An out-of-memory termination.  
   C. An invalid Service port.  
   D. A failed image signature.

3. A pod is Pending because of insufficient memory. What should you inspect?  
   A. Application CSS.  
   B. Scheduler events, requests, and node capacity.  
   C. Git branch names.  
   D. Database indexes only.

4. What commonly causes `ImagePullBackOff`?  
   A. An invalid image reference or registry access failure.  
   B. A successful liveness probe.  
   C. A healthy database.  
   D. A valid Service selector.

5. The application cannot resolve the MongoDB Service name. What should you investigate?  
   A. DNS and Service discovery.  
   B. CPU limits only.  
   C. Image signing only.  
   D. Rollout history only.

6. What can cause a Service to have no backend endpoints?  
   A. A selector that matches no intended pods.  
   B. A valid image digest.  
   C. A healthy DNS lookup.  
   D. A successful build.

7. Why inspect previous container logs?  
   A. They can reveal the error that terminated the preceding instance.  
   B. They automatically repair the pod.  
   C. They increase memory limits.  
   D. They replace monitoring.

8. Why should you not blindly scale the API when MongoDB is saturated?  
   A. More replicas may increase database pressure without fixing the bottleneck.  
   B. Kubernetes cannot run multiple API pods.  
   C. HPA always disables networking.  
   D. MongoDB cannot accept connections.

9. Which command helps inspect scheduling and container events?  
   A. `kubectl describe pod <pod-name>`.  
   B. `npm publish`.  
   C. `git commit`.  
   D. `docker login`.

10. What is the best troubleshooting approach?  
    A. Restart everything immediately.  
    B. Collect evidence, isolate the failure layer, apply a targeted fix, and verify recovery.  
    C. Ignore events.  
    D. Increase every resource limit.

## Answer key

| Question | Answer |
|---:|:---:|
| 1 | A |
| 2 | B |
| 3 | B |
| 4 | A |
| 5 | A |
| 6 | A |
| 7 | A |
| 8 | A |
| 9 | A |
| 10 | B |

## Score guide

- **9–10:** Excellent troubleshooting fundamentals.
- **7–8:** Good; review missed concepts.
- **5–6:** Revisit pod states and networking.
- **Below 5:** Repeat the lesson before advancing.

