# DevOps Mentorship Program — Weekly Revision #1

> **Sunday Special:** Reinforcing Day 1 (Linux Fundamentals).

---

## 📊 Progress Tracker
- **Phase:** Linux
- **Lessons Completed:** Day 1 — Linux Fundamentals
- **Overall Course Progress:** 1 lesson completed
- **Next New Lesson:** Day 2 — Linux File System

---

## 1) 🎯 Revision Objectives
By the end of today's revision, you should be able to:

- Explain what Linux is.
- Describe the role of the Linux kernel.
- Understand why Linux dominates DevOps.
- Navigate directories confidently.
- Differentiate between absolute and relative paths.
- Use basic Linux terminal commands without referring to notes.

---

## 2) 🔄 Quick Review Questions
Try answering these before looking at the answers.

### Question 1 — What is Linux?
<details>
<summary>Answer</summary>

An operating system that manages hardware resources and provides services to applications.

</details>

### Question 2 — What is the Linux kernel responsible for?
<details>
<summary>Answer</summary>

Managing CPU, memory, storage, devices, networking, and communication between software and hardware.

</details>

### Question 3 — Absolute vs relative paths
<details>
<summary>Answer</summary>

- **Absolute path:** starts from `/` (root).
- **Relative path:** starts from the current working directory.

</details>

### Question 4 — Why is Linux preferred in DevOps?
<details>
<summary>Answer</summary>

Because it is stable, secure, efficient, highly automatable, and powers most servers, containers, and cloud infrastructure.

</details>

### Question 5 — What command shows your current directory?
<details>
<summary>Answer</summary>

`pwd`

</details>

---

## 3) 🧠 Concept Recap

### Linux Stack
```text
Application
   |
Shell (Bash)
   |
Linux Kernel
   |
Hardware
```
- The shell accepts your commands.
- The kernel executes them.

### Navigation Commands
- `pwd` — shows current directory.
- `ls` — lists files.
- `cd folder` — moves into a folder.
- `cd ..` — moves up one directory.
- `cd ~` — returns to the home directory.
- `mkdir project` — creates a directory.

---

## 4) 🌍 Real-World DevOps Scenario
Imagine you receive this task:

> **“Deploy the latest version of our MERN backend to an Ubuntu EC2 server.”**

Before Docker/Kubernetes, you must:
- connect to the Linux server,
- navigate to the project directory,
- verify your location,
- inspect files,
- run deployment commands.

Without Linux fundamentals, later DevOps tools can’t be used effectively.

---

## 5) 🧪 Practical Assessment
Open your Linux terminal and complete the following without referring to notes.

### Task 1 — Create this structure
```text
practice/
  week1/
    linux/
      commands/
      notes/
      scripts/
```

**Expected command**
```bash
mkdir -p practice/week1/linux/{commands,notes,scripts}
```

### Task 2 — Navigate into scripts
```bash
cd practice/week1/linux/scripts
```

### Task 3 — Return to home directory
```bash
cd ~
```

### Task 4 — Print the current directory
```bash
pwd
```

### Task 5 — List all files (including hidden)
```bash
ls -a
```

---

## 6) 💼 Interview Revision

### Beginner
**Q1 — What is the difference between Linux and Ubuntu?**
- **Linux** is the kernel.
- **Ubuntu** is a Linux distribution built around the Linux kernel.

**Q2 — What does `pwd` do?**
- Displays the current working directory.

**Q3 — How do you move to the parent directory?**
- `cd ..`

### Intermediate
**Q4 — Why do DevOps engineers prefer the command line over a GUI?**
- Faster
- Scriptable
- Easy to automate
- Works remotely over SSH
- Uses fewer resources

**Q5 — Explain the Linux philosophy.**
- Small tools that each do one job well and can be combined together.

### Advanced
**Q6 — Why is Linux important for Docker and Kubernetes?**
- Containers rely on Linux kernel features (e.g., namespaces and cgroups). Linux is the native environment for containerization and orchestration.

---

## 7) 📝 Weekly Quiz (10 Questions)
1. What is the kernel?
2. What does `pwd` print?
3. Which command lists files?
4. Which command creates a directory?
5. What is the root directory called?
6. Which command moves to the home directory?
7. What is an absolute path?
8. What is a Linux distribution?
9. Name two Linux distributions.
10. Why is Linux preferred for production servers?

### ✅ Answer Key
1. Core component managing system resources.
2. Current working directory.
3. `ls`
4. `mkdir`
5. `/`
6. `cd ~`
7. A path starting from `/`.
8. A complete OS built around the Linux kernel.
9. Ubuntu, Debian (also Rocky Linux, Amazon Linux, etc.).
10. Stability, security, performance, and automation.

---

## 8) 👨‍💻 Practical Challenge
Without copying from previous notes:

1. Create a directory named `mern-deployment`.
2. Inside it create:
   - `frontend`
   - `backend`
   - `database`
3. Navigate into `backend`.
4. Return to `mern-deployment`.
5. Display the directory tree using:

```bash
ls -R
```

---

## 9) 📋 Mini Assignment
Write a one-page note covering:

- What Linux is.
- What the kernel does.
- Difference between terminal and GUI.
- Difference between absolute and relative paths.
- Five Linux commands you've learned, with examples.

---

## 10) 🌟 Best Practices Reinforced
- Always verify your location with `pwd` before running commands.
- Use Tab for auto-completion.
- Read error messages carefully—they often tell you exactly what’s wrong.
- Prefer repeatable terminal workflows over manual GUI actions.
- Practice commands daily until they become muscle memory.

---

## 11) 📖 Recommended Reading
- Linux manual pages (e.g., `man ls`, `man pwd`, `man mkdir`)
- Linux Documentation Project: https://tldp.org/
- Ubuntu Server docs: https://ubuntu.com/server/docs

---

## 12) 🏁 Weekly Summary
This week you established the foundation for your DevOps journey:

- Understood what Linux is.
- Learned the role of the Linux kernel.
- Practiced essential navigation commands.
- Learned why Linux is the backbone of modern DevOps.
- Connected these concepts to real-world MERN deployments.

A strong Linux foundation will make Docker, Kubernetes, CI/CD, Terraform, and AWS much easier to understand.

