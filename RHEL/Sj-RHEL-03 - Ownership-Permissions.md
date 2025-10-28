# SJ-RHEL-03 — Ownership & Permissions

## 1. Introduction — When Linux Decides Who Can Touch What

Imagine you and your friends share a computer at work. Each of you has your own folder to store project files. You create a report named **sales.txt** — by default, you become its **owner**, and your group (e.g., *marketing*) becomes its **group owner**.

Now Linux must decide who can read, edit, or execute this file. That’s where **permissions** come in:

| Permission | Symbol | Meaning |
|-------------|---------|----------|
| Read | r | Can open and view contents |
| Write | w | Can modify or delete the file |
| Execute | x | Can run the file as a script or program |

Every file has two key identities:

| Identity | Meaning |
|-----------|----------|
| **Owner (User)** | The individual who created or owns the file |
| **Group** | A collection of users who share access |

These control how the three permission roles behave:

| Role | Applies To |
|------|-------------|
| **User (u)** | File’s owner |
| **Group (g)** | Members of the file’s group |
| **Others (o)** | Everyone else on the system |

When you create a file like `sales.txt`, Linux automatically assigns ownership and permission rules to protect it.

---

## 2. The `ls -l` Mystery — Reading File Identity Cards

Run this command:
```bash
ls -l
```
Example output:
```
-rw-r-xr--.  1 owner  group  1024 Sep 22 14:20 file.txt
```

That long string is a **code** describing who owns the file, who can use it, and how.

---

### 2.1 The First Character — File Type

| Symbol | Type | Example |
|---------|------|----------|
| - | Regular file | text, code, image |
| d | Directory | folders |
| l | Symbolic link | shortcuts |
| c | Character device | keyboard, terminal |
| b | Block device | hard drive, USB |

---

### 2.2 The Next Nine Characters — Permissions (rwx)

After the file type, there are nine symbols divided into three groups:

| Role | Example | Meaning |
|------|----------|----------|
| User (Owner) | rw- | What the file creator can do |
| Group | r-x | What team members can do |
| Others | r-- | What everyone else can do |

Each group contains three bits:

| Bit | Meaning |
|------|----------|
| r | Read |
| w | Write |
| x | Execute |

---

### 2.3 The Third Field — Hidden Secrets

After the permissions, you may see symbols like `.` or `+`.

| Symbol | Meaning |
|---------|----------|
| . | File has extended attributes (xattr) |
| + | File has Access Control Lists (ACLs) |
| (blank) | No extras — plain file |

These indicate extra metadata or advanced access rules.

---

### 2.4 Link Count — The File’s Popularity

Every file starts with **1 link** — its own directory entry. For directories, the count includes:
- `.` → itself
- `..` → its parent directory

**Formula:**
```
Link count = 2 + number of subdirectories
```

A file’s link count shows how many directory entries point to the same inode. Hard links increase this number.

---

### 2.5 Owner and Group — Who Owns the File

| Field | Meaning |
|--------|----------|
| Owner | The user who created the file (username or UID) |
| Group | A team of users (identified by GID) who share access |

Together, they determine access rights: **r, w, x**.

---

### 2.6 Size, Time, and Name — The File’s Final Identity

| Field | Meaning |
|--------|----------|
| File Size | Size in bytes (e.g., 1024 = 1 KB) |
| Modification Time | When the file content was last changed |
| File Name | The visible label (not the inode itself) |

🧠 **Insight:** Renaming changes only the name — not the inode or metadata.

---

## 3. How Permissions Are Evaluated

Linux checks access in strict order:

1. **User (u)** → Are you the owner? If yes, stop here.
2. **Group (g)** → If not owner, are you in the file’s group?
3. **Others (o)** → If neither, fall into others.

➡️ Only **one category applies** per access attempt.

This top-down evaluation ensures predictable and secure access behavior.

---

## 4. File Permission Modes

Permissions can be set using two methods:

| Mode | Type | Example |
|-------|------|----------|
| Symbolic | Letters and symbols | `chmod u+x file.txt` |
| Numeric | Octal numbers | `chmod 755 file.txt` |

---

### 4.1 Symbolic Mode (Letter Method)

| Symbol | Refers To |
|---------|------------|
| u | user (owner) |
| g | group |
| o | others |
| a | all (u + g + o) |

| Symbol | Meaning |
|---------|----------|
| + | Add permission |
| - | Remove permission |
| = | Set exact permission (overwrite all) |

Examples:
```bash
chmod u+x file.txt       # add execute to owner
chmod g-w file.txt       # remove write from group
chmod o=r file.txt       # others read only
chmod u=rwx,g=rx,o=      # set full for user, read/exec for group, none for others
```

---

### 4.2 Numeric (Octal) Mode

| Symbol | Value |
|---------|--------|
| r | 4 |
| w | 2 |
| x | 1 |

Sum the values to form octal digits:

| Permissions | Value | Meaning |
|--------------|--------|----------|
| --- | 0 | No access |
| r-- | 4 | Read only |
| rw- | 6 | Read + Write |
| rwx | 7 | Full access |

Example:
```bash
chmod 744 file.txt
```
= rwx (user), r-- (group), r-- (others)

---

### 4.3 Symbolic vs Numeric Comparison

| Aspect | Symbolic Mode | Numeric Mode |
|---------|----------------|---------------|
| Format | `chmod u+x file` | `chmod 755 file` |
| Style | Human-readable | Compact numeric |
| Use | Modify specific bits | Set full modes quickly |
| Typical Use | Manual admin work | Scripts & config files |

---

## 5. What Ownership Really Means

Each file has two owners:

| Type | Description |
|-------|--------------|
| **User owner (UID)** | The person who created or owns the file |
| **Group owner (GID)** | The team that shares access |

Together with **Others**, they form the permission triad: `u | g | o`.

---

### 5.1 Dual Ownership Structure

Each file stores two numeric IDs internally:
- **UID** (User ID)
- **GID** (Group ID)

These map to names in `/etc/passwd` and `/etc/group`.

---

### 5.2 Who Can Change Ownership

| Action | Who Can Do It | Notes |
|---------|---------------|-------|
| Change owner | Root only | Regular users can’t transfer ownership |
| Change group | Owner (if in target group) | Allowed only within group membership |
| Change permissions | Owner or root | Uses `chmod` |
| Delete file | Anyone with w+x on directory | Directory controls deletion |

⚠️ **To protect a file from deletion, protect its directory — not just its permissions.**

---

### 5.3 Ownership Logic

| ID | Type | Example |
|----|------|----------|
| UID | User ID | 1000 → ali |
| GID | Group ID | 1001 → devteam |

Copying files between systems can cause ownership mismatch if UID/GID values differ.

---

### 5.4 Commands for Ownership Management

#### `chown` — Change File Owner
```bash
chown sameer file.txt
chown sameer:devteam file.txt
chown -R sameer:devteam /project/
```

#### `chgrp` — Change Group Ownership
```bash
chgrp devteam file.txt
chgrp -R devteam /shared/
```

Changing ownership does **not** change permissions — only who they apply to.

Example:
```
-rw-r-----  sameer devteam file.txt
```
Means:
- sameer → read/write
- devteam → read
- others → none

---

### 5.5 Root Privilege Requirement

Only **root** (or via `sudo`) can assign ownership to another user.
```bash
sudo chown sameer file.txt
```

Normal users can only change groups (if they’re members).

---

## 6. Linux File vs Directory Permissions

| Type | Purpose |
|-------|----------|
| File | Protect content (rwx controls read/edit/execute) |
| Directory | Protect names (rwx controls listing, access, and modification) |

---

### 6.1 File Permissions — Protect Content

| Bit | Meaning | Example |
|------|----------|----------|
| 4 (r) | Read content | `cat file.txt` |
| 2 (w) | Modify file | `echo > file.txt` |
| 1 (x) | Execute | `./script.sh` |

Example:
```
-rw-r--r-- file.txt
```
Only owner can edit; others can view.

---

### 6.2 Directory Permissions — Protect Names

| Bit | Meaning | Example |
|------|----------|----------|
| 4 (r) | List contents | `ls dir` |
| 2 (w) | Modify entries | `touch file`, `rm file` |
| 1 (x) | Access directory | `cd dir` |

Deleting or renaming files requires **w+x on the directory**, not on the file.

---

### 6.3 Common Scenarios

| File | Directory | Result |
|-------|------------|--------|
| 000 | 777 | Anyone can delete, none can read |
| 644 | 755 | Everyone reads, only owner edits |
| 600 | 700 | Only owner full access |
| 777 | 000 | Inaccessible even if file is openable |

---

### 6.4 Quick Truth Table (file 000, dir 777)

| Actor | Read | Write | Delete | Create |
|--------|-------|--------|---------|---------|
| root | Yes | Yes | Yes | Yes |
| owner | No | No | Yes | Yes |
| others | No | No | Yes | Yes |

---

## 7. Immutable & Append-Only Attributes (`chattr`)

Beyond `chmod`, Linux uses **filesystem-level attributes** for deeper control.

### 7.1 What Are Attributes?

| Command | Function |
|----------|-----------|
| `lsattr` | View file attributes |
| `chattr +i file` | Make file immutable |
| `chattr +a file` | Make file append-only |

Attributes live below normal permissions — even root must remove them before editing.

---

### 7.2 Immutable Attribute (+i)
```bash
chattr +i important.txt
```
Locks the file completely:
- Cannot edit, delete, or rename
- Can only read or copy

To unlock:
```bash
chattr -i important.txt
```

---

### 7.3 Append-Only Attribute (+a)
```bash
chattr +a logfile.log
```
Allows appending new data but disallows modification or deletion.

To remove:
```bash
chattr -a logfile.log
```

Ideal for security and audit logs.

---

### 7.4 Attribute Summary Table

| Attribute | Command | Description |
|------------|----------|--------------|
| Immutable (+i) | `chattr +i file.txt` | Prevents modification or deletion |
| Remove Immutable | `chattr -i file.txt` | Restores normal access |
| Append-only (+a) | `chattr +a file.txt` | Allows append-only writes |
| Remove Append | `chattr -a file.txt` | Restores full write |

---

## 8. Access Control Lists (ACLs)

ACLs allow fine-grained permissions beyond the basic rwx model.

### 8.1 What Are ACLs?

They let you grant specific rights to individual users or groups without changing file ownership.

---

### 8.2 Adding or Modifying ACLs
```bash
setfacl -m u:sameer:rw file.txt
setfacl -R -m u:sameer:rw myfolder/
```
- `-m` → modify/add entry
- `-R` → apply recursively

**Default ACLs:**
```bash
setfacl -m d:u:sameer:rw /shared/
```
Automatically inherited by new files.

View ACLs:
```bash
getfacl /shared
```

---

### 8.3 The Mask — Hidden Gatekeeper

The mask defines the **maximum effective permissions**.
```bash
setfacl -m u:sameer:rw file.txt
setfacl -m m:r file.txt
```
Effective rights = r (limited by mask).

---

### 8.4 Viewing ACLs
```bash
getfacl file.txt
```
Example output:
```
user::rw-
user:ali:rw-
group::r--
mask::rw-
other::r--
```

Interpretation:
- `user:ali:rw-` → extra ACL for Ali
- `mask::rw-` → limits effective permissions

---

### 8.5 How ACLs Work with Regular Permissions

Linux first checks rwx, then ACLs (if `+` appears after permissions).

Example:
```
-rw-r--r--+ 1 root root file.txt
```
Use `getfacl` to view all details.

ACLs provide flexible control for shared environments.

---

### 8.6 Managing ACLs — Common Flags

| Flag | Meaning | Example |
|-------|----------|----------|
| -m | Modify or add entry | `setfacl -m u:ali:rw file.txt` |
| -x | Remove entry | `setfacl -x u:ali file.txt` |
| -b | Remove all ACLs | `setfacl -b file.txt` |
| -k | Remove default ACLs | `setfacl -k /shared` |
| -d | Define default ACL | `setfacl -m d:g:designers:r /projects` |
| -R | Apply recursively | `setfacl -R -m g:qa:rwx /testdata` |

---

## 9. Revision Table — Commands & Meanings

| Command | Description |
|----------|--------------|
| `ls -l` | View permissions, owner, group |
| `chmod` | Change file permissions |
| `chown` | Change owner and group |
| `chgrp` | Change group only |
| `lsattr` | View file attributes |
| `chattr` | Modify file attributes (+i, +a) |
| `setfacl` | Add or modify ACLs |
| `getfacl` | Display ACLs |
| `stat` | View detailed file metadata |

---

## 10. Summary

You’ve now learned how Linux decides **who can access what** — from file ownership to permission bits, attributes, and ACLs. These layers ensure both flexibility and security.

In short:
- `chmod` = permission bits
- `chown` / `chgrp` = ownership
- `chattr` = kernel-level protection
- `setfacl` = fine-grained access

**Next Lesson → SJ-RHEL-04 — User Management & Advanced Permissions**
