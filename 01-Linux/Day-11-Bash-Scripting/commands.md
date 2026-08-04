# Day 11 — Commands: Bash Scripting Fundamentals

## Core Bash Concepts

| Item | Purpose | Example |
|------|---------|---------|
| `#!/bin/bash` | Shebang - specifies Bash interpreter | First line of every script |
| `chmod +x` | Make script executable | `chmod +x deploy.sh` |
| `./script.sh` | Execute a script | `./deploy.sh` |
| `bash script.sh` | Run script without exec permission | `bash deploy.sh` |

## Variables & Input

| Command | Purpose | Example |
|---------|---------|---------|
| `VAR="value"` | Define a variable | `APP_NAME="mern-api"` |
| `$VAR` | Access variable value | `echo $APP_NAME` |
| `read -p "prompt: " VAR` | Read user input | `read -p "Name: " NAME` |
| `$1, $2, ...` | Command-line arguments | `echo $1` |
| `$#` | Number of arguments | `echo $#` |
| `$@` | All arguments | `echo $@` |
| `$0` | Script name | `echo $0` |

## Conditional Statements

| Test | Meaning | Example |
|------|---------|---------|
| `-f` | File exists | `[ -f ".env" ]` |
| `-d` | Directory exists | `[ -d "backend" ]` |
| `-z` | Empty string | `[ -z "$VAR" ]` |
| `-n` | Non-empty string | `[ -n "$VAR" ]` |
| `-eq` | Equal (numbers) | `[ $NUM -eq 5 ]` |
| `-ne` | Not equal | `[ $NUM -ne 5 ]` |
| `-gt` | Greater than | `[ $NUM -gt 5 ]` |
| `-lt` | Less than | `[ $NUM -lt 5 ]` |

## Loops

| Loop | Purpose | Example |
|------|---------|---------|
| `for` | Iterate over a list | `for s in nginx docker; do echo $s; done` |
| `while` | Repeat while condition is true | `while [ $C -le 3 ]; do echo $C; C=$((C+1)); done` |

## Functions

| Syntax | Purpose |
|--------|---------|
| `name() { ... }` | Define a function |
| `name` | Call a function |

## Exit Codes & Error Handling

| Command | Purpose | Example |
|---------|---------|---------|
| `$?` | Check last exit code | `echo $?` |
| `cmd \|\| exit 1` | Exit if command fails | `git pull \|\| exit 1` |
| `cmd && echo ok` | Run if command succeeds | `mkdir dir && echo "created"` |
| `set -e` | Exit on any error | `set -e` at top of script |

## Quick Reference

```bash
#!/bin/bash
# A basic deploy script

APP_DIR="/opt/mern/backend"

echo "Deploying..."
cd "$APP_DIR" || exit 1
git pull || exit 1
npm install || exit 1
sudo systemctl restart myapp || exit 1
curl -f http://localhost:3000/health || exit 1
echo "Deployment successful"
