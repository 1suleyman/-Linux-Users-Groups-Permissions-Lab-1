# 🐧 Linux Users, Groups & Permissions Lab 1

In this lab, I built **real Linux admin instincts** by creating users and groups, switching to root safely, managing **primary vs supplementary groups**, and practicing **ownership + group assignment** on real directories/files.

---

## 📋 Lab Overview

**Goal:**

* Become root safely and understand what that actually does
* Create Linux groups (**devops**, **aws**) and verify them
* Create users (**user1**, **user2**, **user3**) with home directories
* Set strong-enough lab passwords (and learn why weak ones fail)
* Set **primary group** for user2 and user3 to **devops**
* Add **aws** as a **supplementary group** for user1
* Create a directory/file structure fast using `mkdir -p` + `touch`
* Practice `chgrp` and `chown` like a real sysadmin

**Learning Outcomes:**

* Use `sudo -i` to get an interactive root login shell
* Create groups with `groupadd` and verify with `getent group`
* Create users with `useradd -m` and verify with `getent passwd`
* Understand `/etc/passwd` fields (username, UID, GID, home, shell)
* Explain **UPG (User Private Groups)** and why UID ≠ GID sometimes
* Modify primary group with `usermod -g`
* Add supplementary group with `usermod -aG` (append safely)
* Change group ownership with `chgrp` and user ownership with `chown`
* Verify everything using `id` and `ls -ld`

---

## 🛠 Step-by-Step Journey

### Step 1: Become root (controlled elevation)

**Commands:**

```bash
whoami
sudo -i
whoami
```

* `whoami` confirmed I started as `ec2-user`
* `sudo -i` gave me an **interactive root login shell**, basically like logging in as root properly (not just running one command)

✅ Expected outcome: `whoami` returns `root`

<img width="262" height="76" alt="Screenshot 2026-01-23 at 11 20 14" src="https://github.com/user-attachments/assets/6efba3cc-8b9d-4d89-b958-4187aa0bd578" />

---

### Step 2: Create the groups (devops + aws)

**Commands:**

```bash
groupadd devops
groupadd aws
```

**Verify:**

```bash
getent group devops
getent group aws
```

* `getent` = “get entries”
* Output format looks like: `groupname:x:GID:members`

  * `x` is a password placeholder
  * the number is the **GID**
  * last field is group members (empty if none yet)

🐛 **Troubleshooting moment:** I typo’d `groupadd aws`, so only `devops` showed up at first. Re-ran correctly, then verified again.

<img width="346" height="60" alt="Screenshot 2026-01-23 at 11 20 55" src="https://github.com/user-attachments/assets/c5730456-dcf3-4a71-857f-62725dda0663" />

<img width="297" height="43" alt="Screenshot 2026-01-23 at 11 27 53" src="https://github.com/user-attachments/assets/08096b94-b486-42a8-81f0-20ebbb6802f4" />

✅ Expected outcome: both groups appear in `getent group`

---

### Step 3: Create users (with home directories)

**Commands:**

```bash
useradd -m user1
useradd -m user2
useradd -m user3
```

* `-m` = create the user’s home directory under `/home/<username>`

✅ Expected outcome: users exist and have home directories

---

### Step 4: Set passwords + learn Linux password rules

**Commands:**

```bash
passwd user1
passwd user2
passwd user3
```

What I learned during password creation:

* **Blank passwords** are rejected (by design)

<img width="252" height="35" alt="Screenshot 2026-01-23 at 11 21 43" src="https://github.com/user-attachments/assets/0fada5a6-64dd-443a-85e8-242785311d7b" />

* Super-simple patterns like `12345678` get rejected as “too simplistic/systematic”

<img width="622" height="34" alt="Screenshot 2026-01-23 at 11 22 24" src="https://github.com/user-attachments/assets/b330fc04-f6e5-440b-b762-3984cc8ca650" />

* This is enforced via **PAM (Pluggable Authentication Modules)** and password quality checks
* Adding complexity (like a symbol) makes it pass

✅ Working lab password example (what finally worked):

* `DevOps123!`

✅ Expected outcome: all 3 users have passwords set successfully

<img width="394" height="35" alt="Screenshot 2026-01-23 at 11 23 00" src="https://github.com/user-attachments/assets/1f441fa1-d3ad-4401-b942-7a77e142ef16" />

---

### Step 5: Verify users exist + understand `/etc/passwd` fields

**Commands:**

```bash
getent passwd user1
getent passwd user2
getent passwd user3
```

Example line structure (from `/etc/passwd`):

* `username:x:UID:GID:comment:home:shell`

Key meaning:

* `x` → placeholder (real hashes live in `/etc/shadow`)
* `UID` → user ID
* `GID` → **primary group ID**
* `home` → home directory
* `shell` → login shell (e.g., `/bin/bash`)

**Bonus check:**

```bash
id user1
```

🧠 **Why UID and GID can differ:**
Linux often uses **UPG (User Private Groups)**: each user gets a private group named after them, which improves security and file permission behavior.

✅ Expected outcome: you can read and explain the user entry format

<img width="402" height="61" alt="Screenshot 2026-01-23 at 11 24 04" src="https://github.com/user-attachments/assets/1a75acc2-3da1-421f-b9c2-f5802f97f6a9" />

---

### Step 6: Set primary group for user2 + user3 to devops

**Commands:**

```bash
usermod -g devops user2
usermod -g devops user3
```

Verify:

```bash
id user2
id user3
```

🧠 **Important insight:**

* `gid=...` shows the **primary group**
* `groups=...` shows **all groups**, and Linux includes the primary group inside that list too

✅ Expected outcome: `gid=devops` and `groups=devops` (or devops plus others)

<img width="375" height="31" alt="Screenshot 2026-01-23 at 11 26 13" src="https://github.com/user-attachments/assets/5523ceb8-667e-43bc-afdc-c25b5d47e7aa" />

---

### Step 7: Add aws as a supplementary group for user1

**Command:**

```bash
usermod -aG aws user1
```

* `-G` = set supplementary groups
* `-a` = **append** (crucial!) — prevents wiping existing supplementary groups

Verify:

```bash
id user1
```

✅ Expected outcome: `gid=user1` remains primary, and `groups=` includes both `user1` and `aws`

<img width="429" height="75" alt="Screenshot 2026-01-23 at 11 25 01" src="https://github.com/user-attachments/assets/cadcf831-ab74-44c8-8cce-9c5e7039f436" />

---

### Step 8: Create the directory + file structure (fast)

**Command (directories):**

```bash
mkdir -p /dir1 /dir2/dir1/dir2/dir10 /dir4 /dir5 /dir6 /dir7/dir10 /dir8 /opt/dir14/dir10
```

* `-p` = create parent directories if missing, and don’t error if already exists

**Command (files):**

```bash
touch /dir1/f1 /f2
```

✅ Expected outcome: structure exists exactly as defined

---

### Step 9: Change group ownership to devops

**Command:**

```bash
chgrp devops /dir1 /dir7/dir10 /f2
```

Verify:

```bash
ls -ld /dir1 /dir7/dir10 /f2
```

* `ls -l` = long listing (owner, group, perms, timestamps)
* `-d` = show the directory itself (not its contents)

🧠 **Idempotency lesson:**
Running the same `chgrp` again doesn’t “break” anything — if it’s already set to devops, it stays devops.

✅ Expected outcome: group column shows `devops`

---

### Step 10: Change ownership to user1

**Command:**

```bash
chown user1 /dir1 /dir7/dir10 /f2
```

Verify:

```bash
ls -ld /dir1 /dir7/dir10 /f2
```

🧠 Extra tip:

* To change **user and group** in one go:

```bash
chown user1:devops /dir1
```

🔐 Reality check:

* Ownership ≠ permissions
* Only `root` (and sometimes the owner, depending) can change ownership
* Regular users can’t `chown` files “as they please”

✅ Expected outcome: owner column shows `user1` on all targets

<img width="420" height="74" alt="Screenshot 2026-01-23 at 11 26 55" src="https://github.com/user-attachments/assets/a6cde944-9962-4dc7-9271-02aa6a134d51" />

---

## ✅ Key Commands Summary

| Task                           | Command / Notes                             |
| ------------------------------ | ------------------------------------------- |
| Check current user             | `whoami`                                    |
| Become root properly           | `sudo -i`                                   |
| Create groups                  | `groupadd devops`, `groupadd aws`           |
| Verify groups                  | `getent group devops`, `getent group aws`   |
| Create users with home dirs    | `useradd -m user1` (repeat for user2/user3) |
| Set passwords                  | `passwd user1` (repeat)                     |
| Verify users exist             | `getent passwd user1 user2 user3`           |
| Inspect user IDs/groups        | `id user1`                                  |
| Set primary group              | `usermod -g devops user2`                   |
| Add supplementary group safely | `usermod -aG aws user1`                     |
| Make directory structure       | `mkdir -p ...`                              |
| Create files                   | `touch /dir1/f1 /f2`                        |
| Change group ownership         | `chgrp devops /dir1 /dir7/dir10 /f2`        |
| Change user ownership          | `chown user1 /dir1 /dir7/dir10 /f2`         |
| Verify ownership quickly       | `ls -ld <paths>`                            |

---

## 💡 Notes / Tips

* **Primary group** is shown next to `gid=` in `id <user>`
* `groups=` includes **primary + supplementary** groups
* `-a` in `usermod -aG` is non-negotiable — without it, you can wipe supplementary groups
* `mkdir -p` is the “speed-run” tool for building deep directory structures
* `chgrp` changes **group owner only**
* `chown` changes **user owner** (and optionally group too via `user:group`)
* Password rules exist to prevent accidental insecure accounts — Linux would rather block access than allow weak logins

---

## 📌 Lab Summary

| Step                                     | Status | Key Takeaways                            |
| ---------------------------------------- | ------ | ---------------------------------------- |
| Elevate to root using `sudo -i`          | ✅      | Interactive root login shell             |
| Create + verify groups                   | ✅      | `getent group` confirms entries + GIDs   |
| Create + verify users                    | ✅      | `useradd -m` + `getent passwd`           |
| Password rules + PAM reality             | ✅      | Blank/simple passwords rejected          |
| Set primary group for user2/user3        | ✅      | `gid=` is the primary group              |
| Add aws as supplementary group for user1 | ✅      | `usermod -aG` appends safely             |
| Build directory/file structure           | ✅      | `mkdir -p` + `touch`                     |
| Apply `chgrp` + `chown` and verify       | ✅      | Ownership and group assignment validated |

---

## ✅ References

* `sudo` manual: `man sudo`
* `groupadd` manual: `man groupadd`
* `useradd` manual: `man useradd`
* `usermod` manual: `man usermod`
* `passwd` manual: `man passwd`
* `chgrp` manual: `man chgrp`
* `chown` manual: `man chown`
* `getent` manual: `man getent`
* `id` manual: `man id`
* `ls` manual: `man ls`
