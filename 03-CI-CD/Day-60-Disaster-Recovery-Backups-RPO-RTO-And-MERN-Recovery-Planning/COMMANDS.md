# Day 60 Commands — Disaster Recovery, Backups and Restore Validation

Use these examples as templates only. Run production commands with approved
credentials, change control, and redacted output. Never paste secrets,
connection strings, customer data, or backup credentials into an incident
channel or source repository.

## 1. Record the recovery context

Before changing production, record the incident ID, UTC time, active release,
database version, affected region, declared RPO/RTO, and recovery owner.

```bash
date -u
git rev-parse HEAD
git log -5 --oneline
kubectl config current-context
kubectl get deploy -n production -o wide
kubectl get pods -n production -o wide
```

The purpose is reproducibility. A later reviewer should be able to determine
which application and infrastructure version was recovered.

## 2. MongoDB backup and restore examples

For self-managed MongoDB, `mongodump` creates a logical backup and `mongorestore`
loads it into a target database. Prefer a temporary recovery environment first;
do not overwrite the only production copy while testing a backup.

```bash
# Export a selected database. Keep credentials in an approved secret store.
mongodump \
  --uri="$MONGODB_URI" \
  --db=commerce \
  --archive=commerce-$(date -u +%Y%m%dT%H%M%SZ).archive.gz \
  --gzip

# Restore into an isolated recovery database.
mongorestore \
  --uri="$RECOVERY_MONGODB_URI" \
  --archive=commerce-backup.archive.gz \
  --gzip \
  --nsFrom='commerce.*' \
  --nsTo='commerce_recovery.*' \
  --drop
```

Managed MongoDB services may provide snapshots and point-in-time recovery
instead. Use the provider's documented restore workflow and record the exact
recovery timestamp. The important outcome is not the command; it is a verified
recovery point and an auditable restore result.

## 3. Validate the restored data

```bash
mongosh "$RECOVERY_MONGODB_URI/commerce_recovery" \
  --eval 'db.runCommand({ ping: 1 }); db.orders.countDocuments({}); db.products.countDocuments({})'

curl -fsS "$RECOVERY_API_URL/health"
curl -fsS "$RECOVERY_API_URL/ready"
```

Compare counts and representative records with expected business invariants.
Counts alone are insufficient: validate indexes, relationships, timestamps,
permissions, and a representative order or checkout flow with test data.

## 4. Recreate infrastructure from source

```bash
terraform init
terraform plan -out=recovery.tfplan
terraform apply recovery.tfplan

# Or, for a Kubernetes recovery environment:
kubectl apply --filename=k8s/ --namespace=production
kubectl rollout status deployment/api -n production --timeout=10m
```

Review the plan before applying it. Recovery automation must not silently
destroy unrelated resources or point a restored application at the wrong
database.

## 5. Verify application artifacts and configuration

```bash
docker pull registry.example.com/commerce-api:$RECOVERY_IMAGE_TAG
docker inspect registry.example.com/commerce-api:$RECOVERY_IMAGE_TAG
kubectl get configmap,secret -n production
kubectl get ingress -n production
```

Secrets should be restored from the approved secret manager, not from a backup
file checked into Git. Confirm that configuration points to the recovery
database and that telemetry is enabled before traffic is moved.

## 6. Smoke-test before traffic restoration

```bash
curl -fsS "$RECOVERY_URL/health"
curl -fsS "$RECOVERY_URL/ready"
curl -fsS -X POST "$RECOVERY_URL/api/orders/test" \
  -H 'Content-Type: application/json' \
  -d '{"testMode":true,"items":[{"sku":"recovery-check","quantity":1}]}'
```

The test endpoint must be explicitly safe and non-billing. Never create a real
customer order or charge a payment method during a recovery drill.

## 7. Measure RPO and RTO

```text
actual RPO = disaster time - newest valid recovered data timestamp
actual RTO = traffic restoration time - disaster declaration time
```

Store the calculated values, evidence links, failed steps, manual actions, and
remaining risks in the incident record. A recovery that succeeds but misses
the objective is a useful finding, not a successful DR result.

