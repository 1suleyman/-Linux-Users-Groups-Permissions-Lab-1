# 📓 Linux Users, Groups & Permissions (Lab 1)

> **Purpose:**
> This document is my raw engineering diary for this lab.
> It captures what I actually did, what surprised me, what confused me, and the mental models I’m forming as a Linux admin.
>
> This file is written for **future me**, not for presentation.

---

## 🧠 Lab Context

**Lab name:** Linux Users, Groups & Permissions — Lab 1 <br>
**Environment:** Amazon Linux EC2 <br>
**Goal:** Build real Linux admin instincts by creating users and groups, safely becoming root, understanding UID/GID behaviour, and practising ownership and group assignment on real directories and files.

This lab is about **foundations**, not speed.

---

## 🧩 Initial Mental Model (Before Starting)

What I thought would happen:

* Creating users and groups would be mostly mechanical.
* UID and GID would usually match.
* Changing ownership and group ownership would be straightforward.
* Password rules would be relaxed in a lab environment.

Assumptions I was making:

* “Basic” Linux admin tasks don’t hide much depth.
* Password enforcement wouldn’t be strict on EC2.
* Group behaviour would be intuitive without deep inspection.

I expected this lab to be easy. That expectation was only partly correct.

---

## 🛠 What I Actually Did (Chronological)

### Step 1 — Becoming root (controlled elevation)

Commands run:

```bash
whoami
sudo -i
whoami
```

Observations:

* I started as `ec2-user`.
* After `sudo -i`, `whoami` returned `root`.

Key realisation:

* `sudo -i` gives an **interactive root login shell**, not just permission to run a single command.
* This changes the environment (PATH, context) and feels closer to logging in as root.

Mental note:

* This distinction matters later when environment and permissions come into play.

---

### Step 2 — Creating groups (devops + aws)

Commands:

```bash
groupadd devops
groupadd aws
```

Verification:

```bash
getent group devops
getent group aws
```

Understanding `getent group` output:

```
groupname:x:GID:members
```

Notes:

* `x` is a password placeholder.
* GID is auto-assigned.
* Members field is empty until users are added.

🐛 **Small troubleshooting moment:**
I mistyped `groupadd aws` the first time, so only `devops` appeared.
Re-running correctly and verifying reinforced a habit: **never assume, always verify**.

---

### Step 3 — Creating users (with home directories)

Commands:

```bash
useradd -m user1
useradd -m user2
useradd -m user3
```

Key flag:

* `-m` creates `/home/<username>` automatically.

Observed behaviour:

* Users were created successfully.
* Home directories existed as expected.

This step was smooth, but I kept verification in mind for later steps.

---

### Step 4 — Setting passwords & learning Linux password rules

Commands:

```bash
passwd user1
passwd user2
passwd user3
```

What I learned *during* this step:

* Blank passwords are rejected.
* Very simple passwords (e.g. `12345678`) are rejected as “too simplistic”.
* Errors are explicit and intentional.

This enforcement comes from:

* PAM (Pluggable Authentication Modules)
* password quality rules

What finally worked:

```text
DevOps123!
```

Insight:

* Linux would rather block access than allow insecure accounts, even in a lab.
* Security defaults are protective, not inconvenient.

---

### Step 5 — Verifying users & understanding `/etc/passwd`

Commands:

```bash
getent passwd user1
getent passwd user2
getent passwd user3
```

Entry structure:

```
username:x:UID:GID:comment:home:shell
```

Key meanings:

* `x` → password hash stored in `/etc/shadow`
* UID → user ID
* GID → **primary group**
* home → home directory
* shell → login shell

Additional check:

```bash
id user1
```

🧠 **Important insight:**
UID and GID don’t always match because of **UPG (User Private Groups)**.

Linux often creates:

* a private group per user
* named after the user
* improves default permission behaviour

This explained behaviour I previously found confusing.

---

### Step 6 — Setting primary group for user2 and user3

Commands:

```bash
usermod -g devops user2
usermod -g devops user3
```

Verification:

```bash
id user2
id user3
```

Key observation:

* `gid=` shows the **primary group**
* `groups=` lists **all groups**, including the primary group

This clarified the relationship between group types.

---

### Step 7 — Adding aws as a supplementary group for user1

Command:

```bash
usermod -aG aws user1
```

Critical detail:

* `-a` (append) is **non-optional**

Without `-a`:

* existing supplementary groups would be wiped.

Verification:

```bash
id user1
```

Observed:

* primary group remained `user1`
* `aws` appeared as a supplementary group

This is an easy mistake with real consequences.

---

### Step 8 — Creating directory & file structure efficiently

Directories:

```bash
mkdir -p /dir1 /dir2/dir1/dir2/dir10 /dir4 /dir5 /dir6 /dir7/dir10 /dir8 /opt/dir14/dir10
```

Files:

```bash
touch /dir1/f1 /f2
```

Notes:

* `-p` avoids errors and creates parent directories automatically.
* This is the fastest and safest way to build deep structures.

---

### Step 9 — Changing group ownership (chgrp)

Command:

```bash
chgrp devops /dir1 /dir7/dir10 /f2
```

Verification:

```bash
ls -ld /dir1 /dir7/dir10 /f2
```

Flags reminder:

* `-l` = long listing
* `-d` = show directory itself

🧠 **Idempotency lesson:**
Re-running `chgrp` doesn’t break anything.
If it’s already correct, it stays correct.

---

### Step 10 — Changing ownership (chown)

Command:

```bash
chown user1 /dir1 /dir7/dir10 /f2
```

Verification:

```bash
ls -ld /dir1 /dir7/dir10 /f2
```

Extra insight:

* `chown user:group` can change both at once.
* Ownership ≠ permissions.
* Only root can freely change ownership.

This reinforced the privilege boundary.

---

## 🔑 Actual Lessons (Beyond Commands)

What stood out most:

* Linux strongly separates **identity**, **group**, and **permissions**
* Defaults prioritise safety over convenience
* Verification commands matter more than the change itself

This lab made those ideas concrete.

---

## 🧠 Mental Models / Rules I’m Taking Forward

* Always verify identity changes using `id` and `getent`
* Primary and supplementary groups behave differently and must be set intentionally
* `usermod -aG` must always include `-a`
* Ownership and permissions are related but distinct
* Linux security defaults are deliberate

**One-liner to remember:**

> *“Linux identity and ownership are simple in concept, but strict by design.”*

---

## 🔄 If I Did This Again

**What I’d do the same:**

* Verify after every change
* Use real users, groups, and filesystem paths
* Treat root access as intentional, not casual

**What I’d do differently:**

* Inspect `/etc/passwd` earlier
* Confirm UPG behaviour sooner

**What I’d check earlier next time:**

* `id <user>` after every group modification

---

## 🧠 Final Reflection

This lab wasn’t about memorising commands.

It was about learning **responsibility**:

* who owns what
* who belongs where
* who is allowed to change things

These fundamentals sit underneath every secure Linux system.
If I get these wrong, nothing above them matters.
