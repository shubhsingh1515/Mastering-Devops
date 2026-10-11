# Day 61 Quiz

## Questions

1. Where should production database credentials normally be stored?
2. What does least privilege mean?
3. What is the first priority when a production secret is committed publicly?
4. What problem does OIDC help reduce in CI/CD?
5. Why should secrets not be included in Docker image layers?
6. Which database identity should a routine MERN API use?
7. What does secret scanning detect?
8. Why must backup access be restricted?
9. Why are `.gitignore` rules not enough after a secret leak?
10. What evidence should be collected after credential rotation?

## Answer key

1. In an approved managed secrets service with controlled access and auditing.
2. Granting only the minimum required permissions and scope.
3. Revoke or rotate the credential and contain possible misuse.
4. It reduces the need for long-lived cloud credentials in CI.
5. Layers and registries can retain and expose them.
6. A dedicated application identity with only required collection actions.
7. Patterns and known signatures that may represent exposed credentials.
8. To prevent a compromised application from destroying recovery data.
9. Tracked history, clones, forks, logs, and artifacts may still contain it.
10. Rotation time, affected identity, access-log results, validation checks, and
   remaining corrective actions.
