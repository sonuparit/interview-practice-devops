This is one of the most important Linux topics for a Mid-level DevOps Engineer. Almost every production server depends on filesystems. If storage is not mounted correctly, containers won't start, databases may fail, logs won't be written, and even the OS may not boot.

Recruiters ask these questions because filesystem problems are among the most common production incidents.

---

# 1. ext4

## 1. What is this?

**ext4 (Fourth Extended Filesystem)** is the default Linux filesystem used by most Linux distributions.

It stores:

* Files
* Directories
* Permissions
* Ownership
* Metadata
* Journals

Think of ext4 as the "database" that stores your files on disk.

---

## 2. Why is it required?

Without a filesystem, Linux only sees raw blocks on the disk.

Filesystem converts:

```
Raw Disk
↓

Meaningful Files

/home
/etc
/var
/usr
```

---

## 3. DevOps Importance

Most servers you SSH into will use ext4.

You'll need to

* mount disks
* resize partitions
* recover files
* check inode usage
* troubleshoot storage issues

---

## 4. Why recruiters care?

Imagine production server:

```
Application can't write logs.
```

Why?

Maybe

```
Disk Full

OR

Inodes Full

OR

Filesystem became Read-only
```

DevOps engineer should immediately know how ext4 works.

---

## 5. Production Problem

Example

```
Database suddenly becomes read-only.
```

After checking

```
dmesg
```

You find

```
EXT4-fs error
```

Filesystem automatically remounted itself

```
Read Only
```

to protect data.

Without filesystem knowledge,

people restart database forever.

Real issue:

Filesystem corruption.

---

## 6. Troubleshooting

Check

```
mount
```

```
df -h
```

```
dmesg
```

```
journalctl
```

```
lsblk
```

---

## 7. Dependencies

Everything depends on ext4

* nginx
* Docker
* Kubernetes
* PostgreSQL
* Jenkins
* Prometheus

Everything writes files.

---

## 8. Analogy

Disk = Empty land

Filesystem = City map

Files = Houses

Directories = Roads

Without city planning,

land exists,

but nobody knows where anything is.

---

## 9. Test Yourself

✔ Why do we need filesystem?

✔ What happens if ext4 becomes read-only?

✔ Why can a disk have free space but still fail writing files?

---

# 2. XFS

## What is it?

High-performance filesystem developed by SGI.

Very common on

* RHEL
* CentOS
* Rocky
* Enterprise servers

---

## Why required?

Designed for

* huge disks
* parallel IO
* enterprise workloads

Excellent for databases and large files.

---

## DevOps Importance

AWS EC2 volumes

RHEL servers

Production databases

often use XFS.

---

## Recruiters care because

Large companies use

```
XFS
```

more often than ext4.

---

## Production Example

Large logging server writing

```
100 GB logs/day
```

XFS performs better.

---

## Troubleshooting

```
xfs_info
```

```
xfs_repair
```

---

## Dependencies

Database

Log servers

Large storage

---

## Analogy

ext4 = Good family car

XFS = Heavy-duty truck

---

## Questions

Why XFS?

Difference from ext4?

When would you choose XFS?

---

# 3. tmpfs

## What is?

Filesystem stored entirely in RAM.

Nothing is stored permanently.

---

## Why required?

Very fast temporary storage.

Examples

```
/tmp

/run

/dev/shm
```

---

## DevOps Importance

Applications needing temporary files.

Containers use it.

---

## Production

Application writes millions of temporary files.

Instead of SSD,

use RAM.

Much faster.

---

## Troubleshooting

```
df -h
```

You'll see

```
tmpfs
```

---

## Dependencies

Docker

Kubernetes

systemd

---

## Analogy

Writing on whiteboard.

Power goes off.

Everything disappears.

---

## Questions

Why tmpfs is fast?

Where is data stored?

When not to use tmpfs?

---

# 4. procfs

## What?

Virtual filesystem.

Located

```
/proc
```

No real files.

Kernel generates everything dynamically.

---

Contains

```
CPU

Memory

Processes

Kernel info
```

---

## Why required?

Allows userspace to communicate with kernel.

---

## DevOps Importance

Almost every monitoring tool reads

```
/proc
```

Prometheus

Node Exporter

top

ps

free

uptime

---

## Production

High CPU.

Need to know process memory.

Read

```
/proc/PID
```

---

## Troubleshooting

```
cat /proc/cpuinfo

cat /proc/meminfo

cat /proc/loadavg
```

---

## Dependencies

Monitoring

Performance

Containers

---

## Analogy

Hospital monitor.

Shows body condition.

Doesn't store body.

---

## Questions

Why proc isn't real filesystem?

Who creates proc?

Can you save files there?

---

# 5. sysfs

## What?

Virtual filesystem

```
/sys
```

Represents Linux devices.

---

Allows configuring hardware and kernel.

---

## DevOps Importance

Container runtimes

Kubernetes

udev

Drivers

use sysfs.

---

## Production

Need to disable CPU scaling.

Done via

```
/sys
```

---

## Troubleshooting

```
ls /sys/class/net

ls /sys/block
```

---

## Analogy

Car dashboard controls.

---

## Questions

Difference between proc and sys?

Who creates sysfs?

Why is it virtual?

---

# 6. overlayfs

Probably the MOST IMPORTANT filesystem for Kubernetes and Docker interviews.

---

## What?

Filesystem that combines

```
Read-only layers

+

Writable layer
```

into one filesystem.

---

Docker images

```
Ubuntu

↓

Python

↓

Application

↓

Writable Layer
```

---

## Why required?

Avoid duplicate files.

Very storage efficient.

---

## DevOps Importance

Docker

containerd

Kubernetes

depend heavily on OverlayFS.

---

## Production

Need 100 containers.

Without OverlayFS

Need 100 Ubuntu copies.

Huge waste.

OverlayFS shares common layers.

---

## Troubleshooting

Docker disk full.

Need to inspect

```
overlay2
```

directory.

---

## Dependencies

Docker

Kubernetes

containerd

BuildKit

---

## Analogy

Projector transparencies.

Several transparent sheets stacked together look like one picture.

---

## Questions

Why OverlayFS saves space?

What is writable layer?

Why containers start quickly?

---

# Commands

---

# mount

Mounts filesystem.

```
mount /dev/sdb1 /data
```

Production

Attach new EBS volume.

Mount it.

---

# umount

Safely removes filesystem.

```
umount /data
```

---

# lsblk

Shows disks.

```
NAME
SIZE
TYPE
MOUNTPOINT
```

First command after adding new disk.

---

# blkid

Shows

UUID

Filesystem type

Labels

Very important while editing

```
fstab
```

---

# df

Disk usage.

```
df -h
```

One of the most used commands in DevOps.

---

# du

Directory usage.

Disk full?

Need largest folder.

```
du -sh *
```

---

# findmnt

Shows mount hierarchy.

Very useful for debugging mounts.

---

# Understand

---

# UUID

Unique disk identifier.

Never changes because device names like

```
/dev/sdb
```

can change after reboot.

Use UUID in

```
/etc/fstab
```

---

# LABEL

Human-friendly disk name.

Example

```
BACKUP

DATABASE

LOGS
```

---

# fstab

Automatic mount configuration.

System reads this during boot.

Wrong fstab

↓

System may fail to boot.

---

# Inode Exhaustion

Very common production issue.

```
Disk

Free Space

Available

BUT

Cannot create files.
```

Reason

No free inodes.

Check

```
df -i
```

---

# Mount Options

Examples

```
ro

rw

noexec

nosuid

nodev
```

Improve

Security

Performance

Control.

Example

```
noexec
```

Prevents execution from that filesystem.

Great for

```
/tmp
```

---

# 15 Mid-Level DevOps Interview Questions

### 1.

Why is UUID preferred over `/dev/sdb1` in `/etc/fstab`?

**Answer:** Device names can change after reboot, but UUID remains constant, ensuring the correct filesystem is mounted.

---

### 2.

What happens if `/etc/fstab` contains an invalid entry?

**Answer:** The system may pause during boot or enter emergency mode because it cannot mount the required filesystem.

---

### 3.

A server shows 30 GB free, but applications cannot create new files. What do you check?

**Answer:** Check inode usage with `df -i`. The filesystem may have exhausted its available inodes.

---

### 4.

What is the difference between `df` and `du`?

**Answer:** `df` reports filesystem-level disk usage (free/used space), while `du` reports how much space specific files or directories consume.

---

### 5.

Why do Docker containers use OverlayFS?

**Answer:** OverlayFS shares read-only image layers between containers and adds a small writable layer for each container, saving storage and speeding up container startup.

---

### 6.

What is `tmpfs`, and when would you use it?

**Answer:** `tmpfs` is a RAM-backed filesystem used for temporary data that needs fast access and does not need to persist after a reboot.

---

### 7.

What is the difference between `procfs` and `sysfs`?

**Answer:** `procfs` exposes process and kernel runtime information, while `sysfs` exposes kernel objects, devices, and hardware configuration.

---

### 8.

A newly attached EBS volume is not visible to your application. How would you troubleshoot?

**Answer:** Check `lsblk` to confirm the disk exists, `blkid` to identify its filesystem, `mount` or `findmnt` to verify it's mounted, and `df -h` to ensure it's available at the expected mount point.

---

### 9.

What is the purpose of `findmnt`?

**Answer:** It displays the current mount tree and shows which filesystems are mounted where, making mount troubleshooting easier.

---

### 10.

Why might you use the `noexec` mount option?

**Answer:** To prevent execution of binaries from a filesystem such as `/tmp`, reducing the impact of malicious or accidental executable files.

---

### 11.

When would you choose XFS instead of ext4?

**Answer:** For enterprise workloads with very large filesystems, high parallel I/O, or heavy logging and database workloads where XFS typically performs better.

---

### 12.

What does `blkid` provide that `lsblk` may not?

**Answer:** `blkid` identifies filesystem metadata such as UUIDs, labels, and filesystem types, which are useful when configuring persistent mounts.

---

### 13.

How would you identify what is consuming disk space on a nearly full server?

**Answer:** Use `df -h` to identify the full filesystem, then `du -sh` (or `du -xh --max-depth=1`) within that filesystem to locate the largest directories.

---

### 14.

Why are virtual filesystems like `/proc` and `/sys` important for monitoring tools?

**Answer:** They provide live kernel, process, memory, CPU, and device information without storing it on disk, allowing tools like `top`, `ps`, and Prometheus exporters to collect system metrics.

---

### 15.

What would you do if a filesystem suddenly becomes read-only in production?

**Answer:** Confirm the mount state with `mount` or `findmnt`, inspect kernel messages using `dmesg` and `journalctl`, check for disk or filesystem errors, stop unnecessary writes to protect data, repair the filesystem during a maintenance window if needed, and investigate the underlying storage issue before remounting it read-write.

---

## Mid-level DevOps Interview Tip

For filesystem questions, interviewers often present a production incident rather than asking for definitions. A structured troubleshooting approach leaves a strong impression:

1. Verify storage devices (`lsblk`, `blkid`).
2. Verify mounts (`mount`, `findmnt`).
3. Check capacity (`df -h`) and inode usage (`df -i`).
4. Identify large directories (`du`).
5. Review kernel and system logs (`dmesg`, `journalctl`).
6. Inspect `/etc/fstab` for persistent mount issues.
7. Repair or remount the filesystem only after identifying the root cause.

Being able to explain *why* each command is used, not just *what* it does, is a key distinction between a junior and a mid-level DevOps engineer.
