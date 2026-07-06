One thing to keep in mind:

> Recruiters don't want a Linux administrator. They want someone who can **operate production Linux servers**.

That means interviewers care less about memorizing commands and more about understanding **why the OS behaves the way it does**.

---

# Linux Roadmap for Mid-Level DevOps

## 1. Linux Fundamentals

* Linux architecture
* Kernel vs User Space
* Shell
* Distributions
* Filesystem hierarchy (FHS)
* Boot process overview
* Runlevels / Targets
* init vs systemd
* Login process
* Virtual terminals

---

## 2. Files and Directories

* ls
* cp
* mv
* rm
* mkdir
* touch
* tree
* stat
* file
* basename
* dirname
* realpath
* pwd

Understand

* inode
* metadata
* hard links
* soft links
* hidden files

---

## 3. File Permissions

Very important.

Topics:

* chmod
* chown
* chgrp
* umask
* rwx permissions
* Numeric permissions
* Symbolic permissions

Advanced

* SUID
* SGID
* Sticky Bit

Understand

* permission inheritance
* default permissions

---

## 4. Users and Groups

Topics

* useradd
* usermod
* userdel
* passwd
* groupadd
* groups
* id
* who
* w
* last

Understand

* /etc/passwd
* /etc/shadow
* /etc/group
* sudo
* sudoers
* PAM basics

---

## 5. Process Management

One of the biggest interview areas.

Commands

* ps
* top
* htop
* pgrep
* pidof
* kill
* killall
* pkill
* nice
* renice
* jobs
* bg
* fg
* nohup
* disown

Concepts

* Process lifecycle
* Parent & child process
* Zombie process
* Orphan process
* Daemon
* Signals
* SIGTERM
* SIGKILL
* SIGINT
* Process priority
* CPU scheduling

---

## 6. Service Management (systemd)

Must know.

Commands

* systemctl
* journalctl

Concepts

* service units
* target
* socket activation
* timers
* dependencies

Know

* enable
* disable
* mask
* restart
* reload

---

## 7. Boot Process

Topics

BIOS

↓

UEFI

↓

GRUB

↓

Kernel

↓

initramfs

↓

systemd

↓

services

↓

login

Know

* bootloader
* kernel parameters
* rescue mode
* emergency mode

---

## 8. Filesystem

Topics

* ext4
* xfs
* tmpfs
* procfs
* sysfs
* overlayfs

Commands

* mount
* umount
* lsblk
* blkid
* df
* du
* findmnt

Understand

* UUID
* LABEL
* fstab
* inode exhaustion
* mount options

---

## 9. Disk Management

Commands

* fdisk
* parted
* mkfs
* fsck
* tune2fs
* resize2fs
* xfs_growfs
* mount

Concepts

* partitions
* filesystem
* block device

---

## 10. LVM

Very common.

Topics

* PV
* VG
* LV

Commands

* pvcreate
* vgcreate
* lvcreate
* lvextend
* lvreduce

---

## 11. Swap Memory

Topics

* swap file
* swap partition

Commands

* swapon
* swapoff
* free

Understand

* swappiness
* OOM killer

---

## 12. Memory Management

Understand

* RAM
* Cache
* Buffers
* Page Cache
* Virtual Memory

Commands

* free
* vmstat
* sar

---

## 13. CPU Management

Commands

* lscpu
* mpstat
* uptime
* vmstat

Understand

* Load Average
* CPU utilization
* Context switches

---

## 14. Storage Troubleshooting

Commands

* iostat
* smartctl
* dmesg

Understand

* I/O wait
* disk latency
* bad sectors
* filesystem corruption

---

## 15. Networking

Huge topic.

Commands

* ip
* ss
* netstat
* ping
* traceroute
* dig
* nslookup
* host
* curl
* wget
* nc
* telnet
* tcpdump

Understand

* TCP/IP
* UDP
* DNS
* Routing
* Gateway
* MTU
* ARP
* ICMP
* CIDR
* NAT

---

## 16. SSH

Commands

* ssh
* scp
* sftp
* ssh-copy-id

Understand

* key authentication
* ssh-agent
* authorized_keys
* ssh config

---

## 17. Logs

Very important.

Know

* journalctl
* syslog
* rsyslog

Files

* /var/log

Commands

* tail
* less
* head
* grep

---

## 18. Text Processing

DevOps interviews love this.

Commands

* grep
* egrep
* awk
* sed
* cut
* tr
* sort
* uniq
* paste
* join
* split
* xargs
* wc

Understand

* Regular Expressions

---

## 19. Find Files

Commands

* find
* locate
* which
* whereis

Advanced

* find with exec
* xargs
* pruning
* time filters

---

## 20. Archives

Commands

* tar
* gzip
* gunzip
* bzip2
* xz
* zip
* unzip

---

## 21. Scheduling Jobs

Commands

* cron
* crontab
* at

systemd timers

---

## 22. Environment Variables

Topics

* PATH
* HOME
* SHELL

Commands

* env
* export
* printenv
* source

---

## 23. Package Management

Ubuntu

* apt

RHEL

* yum
* dnf

Know

* repositories
* GPG keys
* package dependencies

---

## 24. Bash Scripting

Must know.

Topics

* variables
* loops
* arrays
* functions
* arguments
* exit codes
* conditions
* case
* pipes
* redirection

Advanced

* trap
* set -e
* set -u
* set -o pipefail

---

## 25. Pipes and Redirection

Understand

* >
* > >
* <
* <<
* |
* tee
* stdin
* stdout
* stderr

---

## 26. Monitoring & Performance

Commands

* top
* htop
* vmstat
* iostat
* mpstat
* sar
* free
* df
* du

---

## 27. Troubleshooting

Interview favorite.

Know how to troubleshoot

* High CPU
* High Memory
* Full Disk
* Server Slow
* Service Down
* Network issue
* Permission issue
* SSH failure
* DNS failure
* Filesystem read-only
* Disk I/O errors
* OOM Killer
* Kernel panic (high level)
* Application crash
* Port already in use

---

## 28. Security

Topics

* SSH hardening
* File permissions
* sudo
* Fail2Ban (high level)
* firewalld
* ufw
* iptables (basics)
* SELinux (basics)
* AppArmor (basics)

---

## 29. Linux Internals (Mid-Level)

Understand

* Kernel
* Syscalls
* PID namespace
* cgroups
* Namespaces
* proc filesystem
* sysfs
* initramfs
* ELF binaries
* Dynamic linking
* Shared libraries

---

## 30. Production Linux (Most Important)

This is where mid-level DevOps interviews are often decided. You should be comfortable explaining how to:

* Analyze a production outage methodically.
* Read and interpret logs with `journalctl` and files under `/var/log`.
* Identify CPU, memory, disk, or network bottlenecks.
* Trace why a service won't start.
* Diagnose "No space left on device" (including inode exhaustion).
* Investigate high load average versus high CPU usage.
* Determine why an application cannot bind to a port.
* Debug DNS resolution problems.
* Troubleshoot SSH login failures.
* Investigate filesystem corruption or disk I/O errors.
* Use common tools (`ps`, `top`, `ss`, `lsof`, `fuser`, `strace` at a basic level) together to isolate root causes.

---

# Priority for Mid-Level DevOps Interviews

If I ranked these by importance for the types of roles you're targeting:

⭐⭐⭐⭐⭐ **Master these**

1. Process Management
2. systemd
3. Networking
4. Filesystem
5. Storage
6. Bash Scripting
7. Text Processing (`grep`, `awk`, `sed`)
8. Logs
9. Performance Analysis
10. Troubleshooting

⭐⭐⭐⭐ **Know well**
11. Users & Permissions
12. Boot Process
13. SSH
14. Package Management
15. LVM
16. Swap & Memory

⭐⭐⭐ **Understand conceptually**
17. Kernel Internals
18. SELinux/AppArmor
19. cgroups & Namespaces
20. Advanced Filesystems

---

Looking at your progress over the past few days, you've already covered topics like `sed`, `nohup`, `fuser`, `lsof`, hard vs. soft links, and you've debugged real Linux disk I/O issues on your own system. You're moving beyond command memorization into production troubleshooting, which is exactly the direction that mid-level DevOps interviews expect.

I think we can turn this into a structured **30-day Linux interview curriculum**, where each day focuses on one topic with theory, production scenarios, common interview questions, hands-on labs, troubleshooting exercises, and a mock interview. That would give you a comprehensive Linux preparation plan aligned with international DevOps interviews.
