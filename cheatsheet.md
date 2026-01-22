# 🐧 Linux Cheatsheet — Users, Groups & Permissions

**Lab 1: Root → Controlled Access**

> **Goal:** Build real admin instincts
> Users • Groups • Primary vs Secondary Groups • Ownership • Verification

---

## 🔐 Privilege & Identity

### Check current user

```bash
whoami
```

➡️ Shows who you’re currently logged in as.

---

### Become root (interactive login shell)

```bash
sudo -i
```

🧠 What it does:

* `sudo` → run as another user (default: root)
* `-i` → start a **login shell**
* Result: behaves like you logged in as `root` directly

✅ Use this for admin labs and system-wide changes.

---

## 👥 Groups

### Create a group

```bash
groupadd devops
groupadd aws
```

---

### Verify groups exist

```bash
getent group devops aws
```

**Output format**

```
groupname:x:GID:members
```

* `x` → password placeholder
* `GID` → Group ID
* `members` → empty if no users assigned

---

## 👤 Users

### Create a user (with home directory)

```bash
useradd -m user1
useradd -m user2
useradd -m user3
```

* `-m` → creates `/home/username`

---

### Set user password

```bash
passwd user1
```

🧠 Password rules (PAM enforced):

* ❌ Empty passwords rejected
* ❌ Simple patterns rejected (`12345678`)
* ✅ Needs **complexity** (symbols help)

✔️ Example that works in labs:

```
DevOps123!
```

---

### Verify users exist

```bash
getent passwd user1 user2 user3
```

---

## 📄 Understanding `/etc/passwd`

Example entry:

```
user1:x:1001:1003::/home/user1:/bin/bash
```

| Field       | Meaning                          |
| ----------- | -------------------------------- |
| user1       | Username                         |
| x           | Password stored in `/etc/shadow` |
| 1001        | UID (User ID)                    |
| 1003        | GID (Primary Group ID)           |
| ::          | Comment field                    |
| /home/user1 | Home directory                   |
| /bin/bash   | Login shell                      |

---

## 🧠 UID vs GID (IMPORTANT)

### Check user identity

```bash
id user1
```

Example output:

```
uid=1001(user1) gid=1003(user1) groups=1003(user1)
```

🧠 Key rule:

* **UID** → who you are
* **GID** → your **primary group**
* **groups=** → all groups (primary + secondary)

📌 Linux uses **User Private Groups (UPG)** by default
→ Each user gets their own group with the same name.

---

## 🔁 Modify Users & Groups

### Change **primary group**

```bash
usermod -g devops user2
usermod -g devops user3
```

* `-g` → set **primary group**
* New files created by user inherit this group

Verify:

```bash
id user2
```

---

### Add **secondary (supplementary) group**

```bash
usermod -aG aws user1
```

⚠️ **CRITICAL**

* `-a` → append (don’t overwrite)
* `-G` → supplementary groups

Without `-a`, existing groups are wiped.

Verify:

```bash
id user1
```

---

## 📁 Directories & Files

### Create directory structure (parents included)

```bash
mkdir -p /dir1 \
/dir2/dir1/dir2/dir10 \
/dir4 /dir5 /dir6 \
/dir7/dir10 \
/dir8 \
/opt/dir14/dir10
```

* `-p` → create parents + no error if exists

---

### Create files

```bash
touch /dir1/f1 /f2
```

---

## 👥 Group Ownership

### Change group owner

```bash
chgrp devops /dir1 /dir7/dir10 /f2
```

🧠 Notes:

* Changes **group only**
* Does NOT affect user owner or permissions
* **Idempotent** → safe to re-run

---

## 👤 User Ownership

### Change file owner

```bash
chown user1 /dir1 /dir7/dir10 /f2
```

* Changes **user owner**
* Group stays the same

---

### Change user **and** group together

```bash
chown user1:devops /dir1
```

---

## 🔍 Verification Commands

### Long listing (directory itself, not contents)

```bash
ls -ld /dir1 /dir7/dir10 /f2
```

🧠 Breakdown:

* `-l` → long format (permissions, owner, group, time)
* `-d` → show directory metadata, not contents

Example:

```
drwxr-xr-x 2 user1 devops 16 Jan 22 08:58 /dir1
```

---

## 🧠 Ownership vs Permissions (CORE RULE)

❗ Ownership ≠ Permissions

* **Owner** → who owns it
* **Group** → which group owns it
* **Permissions** → what they can do

⚠️ Only:

* `root`
* or sometimes the **current owner**

can change ownership.

---

## 🧩 One-Line Mental Model

> **UID = who you are**
> **GID = your default team**
> **Groups = all teams you belong to**
> **Ownership = who owns it**
> **Permissions = what they can do**
