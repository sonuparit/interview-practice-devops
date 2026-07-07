This is one of the **most important Linux topics** for a DevOps Engineer.

Why?

Because almost every production server eventually runs into one of these problems:

* Disk becomes full
* New storage needs to be attached
* Filesystem becomes corrupted
* Database volume needs expansion
* Kubernetes Persistent Volume needs resizing
* EC2 EBS volume expanded but Linux doesn't see the new space
* Mounts fail after reboot
* Wrong filesystem causes performance issues

A Mid-level DevOps Engineer is expected to solve these problems confidently.

---

# First understand the storage hierarchy

Think of Linux storage like building a house.

```
Physical Disk (SSD/HDD/NVMe)
/dev/sda
       │
       ▼
Partitions
/dev/sda1
/dev/sda2
/dev/sda3
       │
       ▼
Filesystem
ext4
xfs
       │
       ▼
Mounted Directory
/
/var
/home
/data
       │
       ▼
Files
```

Everything starts from a **block device**.

---

# 1. Block Device

## What is this?

A block device is a device that stores data in fixed-size blocks.

Examples

```
/dev/sda
/dev/nvme0n1
/dev/vda
```

These represent actual storage devices.

Think of it as an empty notebook.

No filesystem.

No folders.

Nothing.

Just raw storage.

---

## Why required?

Linux reads and writes everything through block devices.

Without block devices:

* no filesystem
* no partitions
* no mount

Nothing works.

---

## DevOps Importance

Whenever AWS attaches a new EBS volume,

```
/dev/nvme1n1
```

You first see it as a block device.

Only then can you:

Partition it

↓

Format it

↓

Mount it

↓

Use it.

---

## Recruiters care because

Production servers constantly get new disks.

They expect you to know:

```
lsblk
blkid

```

instead of randomly formatting disks.

---

## Production Example

Database storage becomes full.

Cloud team attaches new 500GB EBS.

You identify

```
/dev/nvme1n1
```

Format

Mount

Update fstab

Done.

No downtime.

---

## Troubleshooting

```
lsblk
```

shows

```
NAME
sda
sdb
```

But application can't use it.

Why?

Because filesystem isn't created.

---

## Dependencies

Everything depends on block devices.

Filesystem

Partition

Mount

LVM

RAID

Docker volumes

Kubernetes PV

Everything.

---

## Analogy

Block device = Empty notebook.

---

## Questions

1.

Difference between file and block device?

2.

Can Linux store files directly on block device?

3.

How do you identify newly attached block device?

---

# 2. Partition

## What is this?

Partition divides one physical disk into multiple logical disks.

Example

One SSD

```
500GB
```

can become

```
100GB /
200GB /home
200GB /data
```

---

## Why required?

Separates workloads.

If

```
/var
```

becomes full,

root filesystem still survives.

---

## DevOps Importance

Separate

```
/var
```

```
/opt
```

```
/home
```

```
/data
```

improves stability.

---

## Recruiters care

Partition planning prevents outages.

---

## Production Problem

Logs fill

```
/var
```

Application still runs because

```
/
```

has space.

Without partitioning

Entire server crashes.

---

## Troubleshooting

```
lsblk

df -h

```

Shows which partition is full.

---

## Dependencies

Filesystem lives inside partition.

---

## Analogy

One cupboard.

Partitions are shelves.

---

## Questions

1.

Difference between disk and partition?

2.

Why separate /var?

3.

Can multiple filesystems exist on one disk?

---

# 3. Filesystem

You already studied ext4, XFS etc.

Filesystem organizes raw storage into files and folders.

Without filesystem,

Linux cannot store files.

---

Analogy

Notebook pages with page numbers and index.

---

# Commands

---

# fdisk

## What

Partition management tool (MBR primarily, GPT support is limited compared to parted).

```
fdisk /dev/sdb
```

Interactive menu

```
n

d

p

w
```

---

## Why

Create/delete partitions.

---

## DevOps

New EBS

↓

Create partition

↓

mkfs

↓

mount

---

## Production

Attach new disk

```
fdisk /dev/nvme1n1
```

---

## Troubleshooting

```
fdisk -l
```

Lists partitions.

---

## Dependencies

Partition table.

---

## Analogy

Drawing room boundaries.

---

## Questions

1.

Does fdisk create filesystem?

(No)

2.

Does fdisk erase data?

(Not unless partition changes overwrite existing layout.)

3.

When use fdisk?

---

# parted

## What

Modern partition tool supporting both GPT and MBR, especially useful for large disks (>2 TB).

---

## Why

GPT

Large disks

Automation

---

DevOps

Cloud servers usually use GPT.

---

Production

Create 8TB partition.

Need parted.

---

Analogy

Modern version of fdisk.

---

Questions

1.

Why parted over fdisk?

2.

GPT support?

3.

Large disk support?

---

# mkfs

## What

Creates filesystem.

```
mkfs.ext4 /dev/sdb1
```

or

```
mkfs.xfs /dev/sdb1
```

---

Why

Raw partition becomes usable.

---

Production

Without mkfs

Mount fails.

---

Troubleshooting

```
blkid
```

shows filesystem.

---

Analogy

Writing index inside notebook.

---

Questions

1.

Can mount without mkfs?

(No)

2.

What does mkfs destroy?

(Existing filesystem/data on that target.)

3.

Difference ext4 vs xfs?

---

# fsck

## What

Filesystem consistency checker.

---

Why

Repairs corrupted filesystem.

---

Production

Unexpected power failure.

Filesystem damaged.

```
fsck
```

repairs.

---

Troubleshooting

```
fsck /dev/sdb1
```

Run only on **unmounted** filesystems (or from rescue mode for the root filesystem).

---

Analogy

Repairing damaged book pages.

---

Questions

1.

When run fsck?

2.

Can fsck repair?

3.

Why unmount first?

---

# tune2fs

## What

Changes ext2/ext3/ext4 filesystem parameters without recreating the filesystem.

---

Why

Reserved blocks

Mount count

Filesystem label

UUID-related settings (view/change)

Automatic fsck interval

---

DevOps

Production tuning.

---

Example

Reserve 1%

instead of 5%.

---

Analogy

Changing notebook settings.

---

Questions

1.

Works on XFS?

(No)

2.

Can change label?

3.

Reserved block purpose?

---

# resize2fs

## What

Resize ext filesystem.

---

Production

EBS expanded

Need filesystem resize.

```
resize2fs
```

---

Analogy

Adding notebook pages.

---

Questions

1.

Works for XFS?

(No)

2.

Need after EBS resize?

Yes (for ext filesystems).

3.

Offline or online?

Growing is generally online if mounted and supported by the kernel/filesystem; shrinking requires the filesystem to be unmounted.

---

# xfs_growfs

## What

Grow XFS filesystem.

---

Production

Increase disk.

```
xfs_growfs /data
```

---

Analogy

Extend warehouse.

---

Questions

1.

Shrink?

No, XFS cannot be shrunk.

2.

Grow mounted?

Yes.

3.

Works on ext4?

No.

---

# mount

Already studied.

Mount connects filesystem to directory.

---

# Complete Production Flow

New EBS attached.

Linux detects

```
/dev/nvme1n1
```

↓

Check

```
lsblk
```

↓

Partition

```
fdisk /dev/nvme1n1
```

↓

Filesystem

```
mkfs.ext4 /dev/nvme1n1p1
```

↓

Create mount point

```
mkdir /data
```

↓

Mount

```
mount /dev/nvme1n1p1 /data
```

↓

Persistent

Edit

```
/etc/fstab
```

↓

Verify

```
df -h
```

This is a very common real-world workflow for AWS, Azure, GCP, VMware, and bare-metal Linux servers.

---

# Production Troubleshooting Scenario

**Problem:** An application suddenly reports **"No space left on device"** after the infrastructure team increases an AWS EBS volume from **100 GB to 200 GB**.

**Investigation:**

1. Verify the operating system detects the larger disk.

   ```bash
   lsblk
   ```

   You see the disk is now 200 GB, but the partition is still 100 GB.

2. Expand the partition (using an appropriate partitioning tool such as `parted` or `growpart`, depending on the environment).

3. Check the filesystem type.

   ```bash
   lsblk -f
   ```

4. If it's **ext4**:

   ```bash
   resize2fs /dev/nvme0n1p1
   ```

   If it's **XFS**:

   ```bash
   xfs_growfs /
   ```

5. Verify the new capacity.

   ```bash
   df -h
   ```

**Result:** The application immediately has access to the additional space without recreating the filesystem.

---

# 15 Mid-Level DevOps Interview Questions

### 1. Explain the difference between a disk, a partition, and a filesystem.

**Answer:** A disk is the physical or virtual storage device (block device). A partition divides that disk into logical sections. A filesystem (such as ext4 or XFS) organizes data within a partition so Linux can store files and directories.

---

### 2. What is a block device?

**Answer:** A block device stores data in fixed-size blocks and is represented under `/dev` (for example, `/dev/sda` or `/dev/nvme0n1`). Filesystems are created on block devices or their partitions.

---

### 3. What does `fdisk` do?

**Answer:** `fdisk` creates, deletes, and modifies partitions on a disk. It manages the partition table but does **not** create a filesystem.

---

### 4. When would you use `parted` instead of `fdisk`?

**Answer:** `parted` is preferred for GPT partition tables, disks larger than 2 TB, and scripting/automation. `fdisk` is commonly used for simpler partitioning tasks.

---

### 5. What does `mkfs` do?

**Answer:** `mkfs` creates a filesystem (such as ext4 or XFS) on a partition or block device, making it usable for storing files.

---

### 6. Why can't you mount a newly created partition immediately?

**Answer:** Because a partition alone contains no filesystem. You must first create one using `mkfs`, then mount it.

---

### 7. What is `fsck`, and when should you use it?

**Answer:** `fsck` checks and repairs filesystem inconsistencies, typically after an improper shutdown or disk corruption. It should generally be run on an unmounted filesystem.

---

### 8. What is `tune2fs` used for?

**Answer:** It modifies parameters of ext2/ext3/ext4 filesystems, such as reserved block percentage, filesystem label, mount count, and automatic check intervals.

---

### 9. How do you extend an ext4 filesystem after increasing the disk size?

**Answer:** First ensure the partition has been expanded if needed, then run:

```bash
resize2fs /dev/<partition>
```

---

### 10. How do you grow an XFS filesystem?

**Answer:** Expand the underlying storage first, then run:

```bash
xfs_growfs <mount_point>
```

XFS supports growing online but cannot be shrunk.

---

### 11. Can an XFS filesystem be shrunk?

**Answer:** No. XFS supports online expansion but does not support shrinking.

---

### 12. What information does `lsblk` provide?

**Answer:** It displays block devices, partitions, mount points, sizes, and (optionally) filesystem information, making it one of the first commands used when investigating storage issues.

---

### 13. What is the difference between `df -h` and `du -sh`?

**Answer:** `df -h` reports filesystem usage (total, used, available space), while `du -sh` reports the space consumed by specific directories or files.

---

### 14. What happens if `/etc/fstab` contains an incorrect entry?

**Answer:** The system may fail to mount the filesystem during boot, and in some cases may enter emergency mode. Testing with `mount -a` after editing `fstab` helps catch errors before rebooting.

---

### 15. A new EBS volume is attached to a Linux server. What are the typical steps to make it usable?

**Answer:**

1. Verify the new disk with `lsblk`.
2. Create a partition (`fdisk` or `parted` if required).
3. Create a filesystem using `mkfs`.
4. Create a mount point (for example, `/data`).
5. Mount the filesystem using `mount`.
6. Add a persistent entry to `/etc/fstab`.
7. Verify with `df -h` and `findmnt`.

---

## Recruiter Tip

For a **Mid-level DevOps Engineer**, interviewers are rarely interested in whether you can memorize command syntax. They want to know whether you understand the **storage lifecycle** and can troubleshoot it under pressure.

A strong candidate can confidently explain this sequence:

```
Disk (Block Device)
        ↓
Partition
        ↓
Filesystem
        ↓
Mount Point
        ↓
Application Data
```

If you can describe how data flows through these layers and diagnose where a failure occurs (disk detection, partitioning, filesystem creation, mounting, or persistence via `/etc/fstab`), you'll demonstrate the level of practical Linux knowledge expected in production DevOps environments.
