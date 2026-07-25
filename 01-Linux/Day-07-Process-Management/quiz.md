# 📝 Quiz (10 Questions) — Linux Process Management

## Questions
1. What is a PID?
2. Which command shows all running processes?
3. Which command displays live CPU and memory usage?
4. What does `kill` do?
5. What does `kill -9` do?
6. Which command finds a Node.js process?
7. What symbol starts a background job?
8. Which command lists background jobs?
9. What command shows load averages?
10. Which command brings a background job to the foreground?

## Answers
1. Process ID — a unique numeric identifier for a running process
2. `ps -ef` (or `ps aux`)
3. `top`
4. Sends SIGTERM to terminate a process gracefully
5. Sends SIGKILL to force termination
6. `ps -ef | grep node`
7. `&`
8. `jobs`
9. `uptime`
10. `fg`
