# IAM Access Matrix

This matrix applies least privilege to the production MERN platform. Permissions
are expressed as capabilities rather than provider-specific role names.

| Identity | May access | Required actions | Must not be able to do |
|---|---|---|---|
| Developers | Source repository, local development services, non-production logs | Read/write application code, run tests, deploy to development | Read production secrets, delete production data, deploy directly to production |
| CI build job | Source checkout and build cache | Install dependencies, run tests, produce a build artifact | Read production secrets, modify production data, deploy |
| Image publishing job | Approved container registry | Push immutable, scanned image tags | Delete unrelated images, change runtime secrets, access databases |
| Deployment identity | Target environment and deployment metadata | Deploy an approved image, update configuration references, restart workload | Read arbitrary secrets, alter IAM, delete backups, bypass approval |
| MERN API | Its database and required service endpoints | Read/write application collections, emit telemetry | Create users, change database roles, delete backups, access unrelated services |
| Backup process | Source database and protected backup destination | Read required data, write encrypted backups, verify restore metadata | Read application secrets broadly, alter live application data, delete retained backups |
| Monitoring agent | Metrics, health endpoints, and approved logs | Read telemetry and publish monitoring signals | Read secret values, modify workloads, access customer data unnecessarily |

## Guardrails

1. Use separate identities for build, publish, deploy, runtime, backup, and
   monitoring operations.
2. Scope permissions to the smallest resource and environment possible.
3. Require MFA and protected branches for human production access.
4. Prefer short-lived federated credentials for automation.
5. Require approval for production deployment and IAM changes.
6. Log authentication, authorization, secret access, deployment, and backup
   operations.
7. Review access quarterly and after role or team changes.
8. Use deny-by-default network and identity policies where supported.

## Review questions

- Can the API delete or rewrite backups?
- Can a CI build job read production credentials?
- Can a deployment identity change its own permissions?
- Can monitoring access raw customer payloads?
- Are staging and production identities separate?

Any “yes” answer should result in a least-privilege review before release.
