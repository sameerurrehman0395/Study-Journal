# SJ-RHEL-01 — Linux Filesystem Structure

## 1. Introduction

Everything in Linux starts at the **root directory (/)**, the single starting point for all files and folders. The Linux filesystem follows the **Filesystem Hierarchy Standard (FHS)**, maintained by the Linux Foundation, to ensure consistency across all distributions.

The latest version (FHS 3.0.3, June 2015) defines how directories are organized and what each should contain. Understanding this hierarchy helps in navigation, troubleshooting, and configuration.

Think of it as a **tree**: `/` is the root, top-level folders are branches, and deeper subdirectories are leaves. Every process and file ultimately traces back to this root.

To visualize it:

```bash
tree -D -L 1 /
```

This command lists the first-level directories under `/`, giving a snapshot of your system layout.

---

## 2. Types of Files in Linux

In Linux, everything is treated as a **file**. Files generally fall into three categories:

- **Regular files** – Text, executables, images, and scripts.
- **Directories** – Special files that contain other files or subdirectories.
- **Special files** – Devices, links, and sockets representing hardware or communication interfaces.

---

## 3. Who This Tutorial Is For

- RHEL, CentOS, and Fedora learners building fundamentals.
- System administrators managing production systems.
- DevOps engineers automating configurations.

---

## 4. What You'll Learn

You’ll understand:

- Purpose of first-level directories under `/`.
- Role of subdirectories like `/usr`, `/etc`, `/var`.
- Key configuration and log files.

---

## 5. Level 1 — Top-Level Directories

| Directory | Purpose | Key Examples |
|------------|----------|---------------|
| /bin | Essential user commands | ls, cp, mv, bash |
| /sbin | Core admin tools | fsck, mount, reboot |
| /usr | User programs & shared data | Applications, libraries, docs |
| /etc | System configuration | Config files, startup scripts |
| /var | Variable or changing data | Logs, caches, databases |
| /lib, /lib64 | Shared libraries | .so files |
| /home | User directories | Personal data |
| /root | Root’s home | Admin files |
| /boot | Bootloader & kernel | vmlinuz, grub.cfg |
| /dev | Device files | sda, null, zero |
| /proc | Virtual process info | Kernel runtime data |
| /mnt | Temporary mount point | /mnt/usb |
| /sys | Virtual hardware info | Device tree |
| /tmp | Temporary files | Auto-cleared on reboot |
| /run | Volatile runtime data | PID, lock, socket files |

**Note:** `/etc` → *Edit To Configure*, `/var` → *Variable data*.

Check partitions:

```bash
lsblk -f
```

---

## 6. Level 2 — Important Subdirectories

### 6.1 Inside /usr

| Subdirectory | Description |
|---------------|--------------|
| /usr/bin | Non-essential user commands (vim, git, python3) |
| /usr/sbin | Admin tools & daemons (sshd, httpd) |
| /usr/share | Architecture-independent data (docs, locales) |
| /usr/lib, /usr/lib64 | Libraries for /usr programs |

### 6.2 Inside /etc

| Subdirectory | Description |
|---------------|-------------|
| /etc/ssh/ | SSH client/server configs |
| /etc/systemd/system/ | Custom service units |
| /etc/yum.repos.d/ | Repository definitions |
| /etc/network/ | Network settings (distro dependent) |

Everything here can be edited with `cat`, `nano`, or `vim`.

### 6.3 Inside /var

| Subdirectory | Description |
|---------------|-------------|
| /var/log/ | System and service logs |
| /var/lib/ | Databases and app data |
| /var/spool/ | Queued jobs (print, cron) |
| /var/cache/ | Cached data |

### 6.4 Other Notables

| Parent | Subdirectory | Description |
|---------|---------------|-------------|
| /boot | /boot/grub2/ | GRUB configuration |
| /lib | /lib/modules/ | Kernel modules |
| /proc | /proc/sys/ | Kernel parameters |
| /sys | /sys/class/ | Devices by type (net, block) |
| /home | /home/<user>/ | User dotfiles (.bashrc, .ssh/) |

---

## 7. Level 3 — Key Configuration & System Files

### 7.1 /etc — Configuration Nucleus

| File | Function |
|------|-----------|
| /etc/passwd | User account info |
| /etc/shadow | Encrypted passwords |
| /etc/group | Group definitions |
| /etc/fstab | Filesystem mount points |
| /etc/hostname | System hostname |
| /etc/hosts | Local host–IP mapping |
| /etc/resolv.conf | DNS resolver |
| /etc/sudoers | Sudo privileges |
| /etc/sysctl.conf | Kernel tuning |
| /etc/httpd/conf/httpd.conf | Apache config |
| /etc/selinux/config | SELinux mode |

Example:

```bash
cat /etc/fstab
```

Shows filesystems that auto-mount at boot.

### 7.2 /usr/bin — User Executables

| File | Purpose |
|------|----------|
| vim | Text editor |
| python3 | Python interpreter |
| git | Version control |
| curl | HTTP requests |
| grep | Text search |
| systemctl | Manage services |

Example:

```bash
which systemctl
systemctl status sshd
```

### 7.3 /usr/sbin — Administrative Daemons

| File | Function |
|------|-----------|
| sshd | SSH daemon |
| httpd | Apache web server |
| firewalld | Firewall daemon |
| useradd | Create users |
| cron | Job scheduler |

Example:

```bash
systemctl status sshd
```

### 7.4 /var — Dynamic Runtime Data

| File | Purpose |
|------|----------|
| /var/log/messages | General system log |
| /var/log/secure | Authentication logs |
| /var/lib/mysql/ | Database files |
| /var/spool/cron/root | Root’s cron jobs |

Example:

```bash
journalctl -xe
```

### 7.5 /boot — Boot Essentials

| File | Purpose |
|------|----------|
| /boot/vmlinuz-* | Linux kernel image |
| /boot/initramfs-* | Initial RAM disk |
| /boot/grub2/grub.cfg | Bootloader configuration |

Regenerate GRUB:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 7.6 /dev, /proc, /sys — Virtual Interfaces

| Directory | Description |
|------------|-------------|
| /dev/ | Hardware devices (sda, tty, null) |
| /proc/ | Kernel and process info (cpuinfo, meminfo) |
| /sys/ | Hardware tree and topology |

Example:

```bash
cat /proc/cpuinfo
```

---

## 8. Confusion Over /bin and /sbin

| Directory | Type of Commands | Access | Purpose |
|------------|------------------|---------|----------|
| /bin | Essential user cmds | All users | Boot and repair tools |
| /usr/bin | User apps | All users | Editors, utilities |
| /sbin | Essential admin cmds | Root | Low-level maintenance |
| /usr/sbin | Non-essential daemons | Root | Server/admin tools |

Modern RHEL merges `/bin` → `/usr/bin` and `/sbin` → `/usr/sbin` using symlinks.

---

## 9. Category Reference Table

| Category | Example Paths | Editable | Volatile | Purpose |
|-----------|----------------|-----------|-----------|----------|
| Configuration | /etc/ | ✔ | ✖ | System settings |
| Commands | /bin, /usr/bin | ✖ | ✖ | Executables |
| Admin Tools | /sbin, /usr/sbin | ✖ | ✖ | Maintenance |
| Libraries | /lib, /usr/lib | ✖ | ✖ | Shared code |
| Variable Data | /var/ | ✔ | ✔ | Logs, caches |
| Temporary | /tmp, /run | ✔ | ✔ | Short-term files |
| Virtual | /proc, /sys | ✖ | ⚡ | Kernel/runtime info |

---

## 10. Hierarchy Overview

```
/
├── bin/ → core user commands
├── sbin/ → admin commands
├── etc/ → configs
├── usr/
│   ├── bin/, sbin/, share/, lib/
├── var/
│   ├── log/, lib/, spool/, cache/
├── boot/ → kernel & GRUB
├── dev/, proc/, sys/ → system info
├── home/, root/ → user data
├── tmp/, run/ → temporary data
└── …
```

---

## 11. Memory Hooks

- `/etc` → Edit To Configure  
- `/var` → Variable Data  
- `/usr` → User programs  
- `/lib` → Libraries  
- `/proc` → Processes  
- `/sys` → System hardware  
- `/boot` → Boot files  
- `/home` → Personal data

---

## 12. Access-Based Directory Summary

| Directory | Purpose | Access |
|------------|----------|---------|
| /bin | Essential user commands | All users |
| /sbin | Admin commands | Root only |
| /usr/bin | User utilities | All users |
| /usr/sbin | System daemons | Root only |
| /etc | System configs | Root only |
| /var | Runtime data | System writes, users read |
| /lib | Shared libraries | System only |
| /boot | Kernel & GRUB | Root only |

**Key Insight:**
- **All users:** Safe to use (`/bin`, `/usr/bin`)
- **Root only:** Admin-level changes (`/sbin`, `/etc`)
- **System-level:** OS-managed directories

---

## 13. Summary

You’ve now explored the Linux filesystem three levels deep — directories, subdirectories, and key files. This hierarchy is foundational for mastering RHEL administration.

Filesystem knowledge isn’t just structure — it’s **power**. It enables recovery, optimization, and confident automation.

Next in **SJ-RHEL-02**, we’ll dive deeper into **Paths, Inodes, and Links**.
