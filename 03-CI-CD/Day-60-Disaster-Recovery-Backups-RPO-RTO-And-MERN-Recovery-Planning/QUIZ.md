# Day 60 Quiz — Disaster Recovery

Answer the questions before checking the key.

## Questions

1. What does RPO measure?
2. What does RTO measure?
3. Why is an untested backup not proof of recoverability?
4. How are replication and backup different?
5. Which recovery method is useful after an accidental destructive database
   change?
6. Why does Infrastructure as Code help disaster recovery?
7. Name three controls that protect backups.
8. If the RTO is 30 minutes and a tested recovery takes 90 minutes, what does
   that tell you?
9. What should be checked after restoring MongoDB?
10. Why should traffic be restored gradually?

## Answer key

1. The maximum acceptable data loss, measured as elapsed time.
2. The maximum acceptable time to restore service.
3. Restore access, compatibility, permissions, data integrity, and application
   behavior may all fail even when a backup job reports success.
4. Replication primarily improves availability; backups provide independent
   recovery points and protection from logical mistakes.
5. Point-in-time recovery or a backup restored to a timestamp before the
   destructive operation.
6. It makes infrastructure reproducible, reviewable, versioned, and faster to
   provision.
7. Encryption, least privilege, separate credentials, immutability or deletion
   protection, retention controls, and audit logging.
8. The recovery process misses its objective and needs improvement or an
   explicitly approved objective change.
9. Connectivity, indexes, record integrity, timestamps, relationships,
   permissions, and representative application workflows.
10. It limits additional impact and provides evidence that the recovered
    environment is healthy before full exposure.

## Score

```text
9–10: Ready to explain and test a basic DR strategy
7–8:  Good foundation; revisit restore validation and backup protection
0–6:  Re-read README.md and complete the assignment
```
