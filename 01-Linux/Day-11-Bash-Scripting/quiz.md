# Day 11 — Quiz: Bash Scripting Fundamentals

## Questions

1. **What does `#!/bin/bash` specify?**

   <details>
   <summary>Answer</summary>

   It specifies the **Bash interpreter** that should execute the script. This line is called the **shebang**.
   </details>

2. **Which command makes a script executable?**

   <details>
   <summary>Answer</summary>

   `chmod +x <script.sh>` — Adds execute permission to the script.
   </details>

3. **Which variable contains the first command-line argument?**

   <details>
   <summary>Answer</summary>

   `$1` — The first argument passed to the script when it's run.
   </details>

4. **What exit code indicates success?**

   <details>
   <summary>Answer</summary>

   **`0`** — Exit code 0 means the command completed successfully.
   </details>

5. **Which keyword starts a conditional?**

   <details>
   <summary>Answer</summary>

   `if` — For example: `if [ -f ".env" ]; then ... fi`.
   </details>

6. **Which keyword repeats over a list?**

   <details>
   <summary>Answer</summary>

   `for` — For example: `for service in nginx docker; do echo $service; done`.
   </details>

7. **How do you define a function?**

   <details>
   <summary>Answer</summary>

   `name() { ... }` — For example: `check_disk() { df -h; }`. Then call it with `check_disk`.
   </details>

8. **Which command reads user input?**

   <details>
   <summary>Answer</summary>

   `read` — For example: `read -p "Enter name: " NAME`.
   </details>

9. **What does `$?` contain?**

   <details>
   <summary>Answer</summary>

   The **exit code of the previous command** — `0` for success, non-zero for error.
   </details>

10. **Why should important commands be checked for failure?**

    <details>
    <summary>Answer</summary>

    To **stop or handle failures before continuing**. If a critical step fails (like `git pull`), the script should stop to prevent a broken or partial deployment.
    </details>

## Bonus Questions

11. **What is the difference between `$*` and `$@`?**

    <details>
    <summary>Answer</summary>

    Both represent all command-line arguments. `$@` treats each argument as a separate word (safer with spaces), while `$*` treats them all as a single string.
    </details>

12. **What does the `-f` test check?**

    <details>
    <summary>Answer</summary>

    Whether a **file exists** — `[ -f "/path/to/file" ]` returns true if the file exists.
    </details>

13. **What does `-d` test check?**

    <details>
    <summary>Answer</summary>

    Whether a **directory exists** — `[ -d "backend" ]` returns true if the directory exists.
    </details>

14. **What is the purpose of `set -e`?**

    <details>
    <summary>Answer</summary>

    It makes the script **exit immediately if any command fails** (returns a non-zero exit code). This is a safety net for critical scripts.
    </details>

15. **What does `command -v git` do?**

    <details>
    <summary>Answer</summary>

    It checks whether the command `git` **exists and is available** in the PATH. Returns 0 if found, non-zero if not.
    </details>
