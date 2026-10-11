# Day 61 Commands

These commands are intentionally provider-neutral where possible. Replace
placeholder values locally and never paste real credentials into shell history.

## Repository checks

```bash
git status --short
git check-ignore .env
git grep -n -I -E 'MONGODB_URI=|JWT_SECRET=|AWS_SECRET_ACCESS_KEY=|API_KEY=' -- .
```

The final command is a basic check, not a replacement for a dedicated secret
scanner. Review every match because examples and test fixtures can create
false positives.

## Safe environment template

```bash
copy .env.example .env
```

On PowerShell:

```powershell
Copy-Item .env.example .env
Get-Content .gitignore
git check-ignore .env
```

## Docker inspection

```bash
docker history --no-trunc your-image:tag
docker inspect your-image:tag
```

Look for credentials in image history, environment metadata, labels, and copied
configuration files. Do not pass secrets as Docker build arguments.

## Secret-scanning examples

```bash
gitleaks detect --source . --redact
trufflehog filesystem .
```

Run one scanner in pull requests and another periodically if the risk profile
requires defense in depth. A finding must be triaged, and a confirmed exposed
credential must be revoked or rotated.

## Incident evidence collection

```bash
git log --all --oneline -- .env
git log --all -S'JWT_SECRET' --oneline
```

These commands identify possible exposure locations; they do not make a secret
safe. Revoke first, then investigate history and access logs.
