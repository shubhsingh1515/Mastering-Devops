# Day-05: Linux Users & Groups

## 🎯 Learning Objectives

By the end of today's lesson, you'll understand:

- Why Linux has multiple users
- Root vs normal users
- Primary and secondary groups
- Creating and deleting users
- Managing groups
- Changing passwords
- Understanding sudo
- Service accounts
- Security best practices for production environments

---

## 🤔 Why This Topic Is Important

Imagine your production server hosts:

- React Frontend
- Node Backend
- MongoDB
- Nginx
- Jenkins

**Should they all run as `root`?**

❌ Absolutely not.

If one application is compromised, an attacker gains complete control over the server.

Instead, every production service runs under its own restricted user.

For example:

```
root
├── nginx
├── mongodb
├── jenkins
├── nodeapp
└── ubuntu
```

This limits damage if one service is compromised.

---

## 📖 Theory Explained in Simple Language

### What is a User?

A user is an identity Linux uses to determine:

- Who can log in
- Which files can be accessed
- Which commands can be executed
- What permissions are available

Every process runs as a user.

### Types of Users

#### 1. Root User

`root`

**Characteristics:**
- Full control
- Can modify any file
- Can delete any file
- Can install software
- Can create users
- Can stop any process

**UID:** `0`

> ⚠️ Never use the root account for everyday work.

#### 2. Normal User

**Example:** `ubuntu`, `john`, `alice`

Normal users have limited permissions. They require `sudo` for administrative tasks.

#### 3. Service Users

Created specifically for applications.

**Examples:** `www-data`, `nginx`, `mongodb`, `jenkins`, `mysql`

These accounts typically:
- Cannot log in interactively
- Own only their application files
- Have minimal permissions

### What is a Group?

A group is a collection of users.

Instead of granting permissions to users one by one, you grant them to a group.

**Example:**

```
developers
├── Alice
├── Bob
└── Charlie
```

Grant access once to the `developers` group, and all members inherit it.

### Primary vs Secondary Groups

**Example:**

```
User: john
Primary group: john
Secondary groups: docker, developers, sudo
```

A user always has **one primary group** and can belong to **multiple secondary groups**.

### View Current User

```bash
whoami
```

**Example:** `ubuntu`

### View Current User ID

```bash
id
```

**Example:**
```
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo)
```

### View Logged-in Users

```bash
who
```

### Create a User

```bash
sudo useradd devops
```

Create a home directory:

```bash
sudo useradd -m devops
```

### Set Password

```bash
sudo passwd devops
```

### Delete User

```bash
sudo userdel devops
```

Delete the home directory too:

```bash
sudo userdel -r devops
```

### Create a Group

```bash
sudo groupadd developers
```

### Add User to Group

```bash
sudo usermod -aG developers devops
```

`-aG` means **Append to Group**.

> ⚠️ Never omit `-a` when adding to additional groups, or you may overwrite existing secondary group memberships.

### View Groups

For current user:

```bash
groups
```

Specific user:

```bash
groups devops
```

### The `sudo` Command

```bash
sudo apt update
```

`sudo` means **Run as administrator**.

Instead of logging in as root, trusted users perform privileged actions with `sudo`.

### The Principle of Least Privilege

Give only the permissions necessary to perform the job.

**Bad:**
```
Everyone → Root
```

**Good:**
```
Developers → Limited Access → Only Required Commands
```

This reduces the impact of mistakes and attacks.

---

## 🌍 Real-World Example: MERN Deployment Server

Suppose your server hosts:

- Node Backend
- React Frontend
- MongoDB
- Nginx

A secure setup might look like:

```
root
│
├── ubuntu        (SSH login)
├── deploy        (CI/CD deployments)
├── nodeapp       (Node.js service)
├── nginx         (Web server)
└── mongodb       (Database)
```

The Node.js process runs as `nodeapp`, not root, limiting the permissions available to the application.

---

## 🏗️ Architecture Diagram (ASCII)

```
                 Root
                  │
     ┌────────────┼────────────┐
     │            │            │
   ubuntu      deploy      nodeapp
                               │
                          developers
                      ┌────────┴────────┐
                    Alice             Bob
```

---

## 🛠️ Step-by-Step Practical Implementation

> **Note:** These commands require `sudo`. If you're using a shared machine or cloud instance, understand their impact before running them.

**Step 1** — View current user:
```bash
whoami
```

**Step 2** — View user information:
```bash
id
```

**Step 3** — View logged-in users:
```bash
who
```

**Step 4** — Create a practice user:
```bash
sudo useradd -m devopsdemo
```

**Step 5** — Set password:
```bash
sudo passwd devopsdemo
```
You'll be prompted to enter and confirm a password.

**Step 6** — Create a group:
```bash
sudo groupadd developers
```
> If the group already exists, Linux will report an error — that's expected.

**Step 7** — Add the user to the group:
```bash
sudo usermod -aG developers devopsdemo
```

**Step 8** — Verify group membership:
```bash
groups devopsdemo
```
Expected output: `devopsdemo : devopsdemo developers`

**Step 9** — View account details:
```bash
id devopsdemo
```

**Step 10** — Cleanup (optional):
```bash
sudo userdel -r devopsdemo
```

---

## ✅ Best Practices

- Never log in as **root** for daily work.
- Use `sudo` only when necessary.
- Run applications under **dedicated service accounts**.
- Grant users only the permissions they need.
- Use **groups** to manage shared access instead of changing permissions for individual users repeatedly.
- Regularly review user accounts and remove unused ones.

---

## ❌ Common Mistakes

- Running applications as **root**.
- Forgetting the `-a` flag with `usermod -aG`, which can remove existing group memberships.
- Sharing a single account among multiple administrators.
- Leaving unused user accounts enabled.
- Granting broad `sudo` access without a business need.
