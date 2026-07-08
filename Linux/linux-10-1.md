This is one of the most important topics for a **Mid-level DevOps Engineer** because almost every production interview has at least one question about **backup and disaster recovery (DR)**.

Recruiters aren't interested in whether you know the `tar` command. They want to know:

* Can you recover production after a disaster?
* Can you restore data without corruption?
* Do you know the difference between a backup and a snapshot?
* Can you minimize downtime?
* Do you verify backups?

Let's build this from the ground up.

---

# Imagine this Production Server

```
Production Server

CPU
RAM

Disk

├── /
├── /etc
├── /var
│    ├── www
│    ├── logs
│    └── mysql
├── /home
└── application
```

Suppose this server hosts

* Nginx
* MySQL
* Web application
* User uploaded files

If the disk dies...

Everything disappears.

You need a recovery strategy.

---

# There are TWO major concepts

```
Disaster Recovery

          |
   ------------------
   |                |
Snapshot        Backup
```

People confuse these constantly.

They are NOT the same.

---

# What is a Snapshot?

A snapshot is

> a point-in-time copy of a disk or volume.

Imagine freezing time.

```
10:00 AM

Disk

A
B
C
D
E
```

Create Snapshot

Now snapshot remembers

```
A
B
C
D
E
```

After one hour

```
Disk becomes

A
B
C
D
E
F
G
```

Snapshot still contains

```
A
B
C
D
E
```

It is frozen at 10:00 AM.

---

# Important

A snapshot is NOT another complete disk.

Most snapshots are

## Copy-on-write

Example

Original block

```
Block 25

Hello
```

Snapshot created.

Nothing copied.

Storage used

```
0 MB
```

Now you edit

```
Hello

↓

Hello World
```

Before changing,

storage system saves old block

```
Snapshot

Hello

Current disk

Hello World
```

Only changed blocks consume space.

Very efficient.

---

# Types of Snapshots

Several technologies exist.

## 1. LVM Snapshot

Linux

```
lvcreate --snapshot
```

---

## 2. Filesystem Snapshot

Examples

* Btrfs
* ZFS

---

## 3. Cloud Snapshot

Examples

AWS

* EBS Snapshot

Azure

* Managed Disk Snapshot

Google Cloud

* Persistent Disk Snapshot

These are most common in DevOps.

---

# Practical LVM Snapshot

Imagine

```
Volume Group

vg_prod
```

Logical Volume

```
lv_data
```

Mounted

```
/data
```

See LVs

```
lvs
```

Output

```
LV       VG
lv_data  vg_prod
```

---

Create snapshot

```
sudo lvcreate \
-L 5G \
-s \
-n lv_data_snap \
/dev/vg_prod/lv_data
```

Meaning

```
-L 5G

Snapshot storage
```

```
-s

snapshot
```

```
-n

snapshot name
```

---

Verify

```
lvs
```

Output

```
lv_data
lv_data_snap
```

Snapshot created.

---

# Mount Snapshot

```
mkdir /mnt/snapshot

mount \
/dev/vg_prod/lv_data_snap \
/mnt/snapshot
```

Now browse

```
cd /mnt/snapshot
```

Everything is exactly as it was.

---

# Restore LVM Snapshot

Suppose

```
rm -rf /data/*
```

Oops.

Production deleted.

Restore

Unmount filesystem

```
umount /data
```

Merge snapshot

```
lvconvert \
--merge \
/dev/vg_prod/lv_data_snap
```

Reboot (or activate appropriately depending on whether the LV is in use)

```
reboot
```

System comes back exactly at snapshot time.

---

# Problems with LVM Snapshots

Huge interview topic.

## Problem 1

Snapshot fills up

Suppose snapshot size

```
5 GB
```

Production changes

```
20 GB
```

Snapshot becomes

```
100% full
```

Result

Snapshot becomes invalid.

---

Problem 2

Performance

Every write

Needs copy-on-write.

Disk becomes slower.

---

Problem 3

Not backup

If disk dies

```
Disk gone

Snapshot gone
```

Both disappear.

This surprises many beginners.

---

# Cloud Snapshot

Most companies use cloud.

Example AWS.

Volume

```
EBS Volume

500GB
```

Create snapshot.

AWS copies blocks to object storage.

Even if volume dies

Snapshot survives.

Huge advantage.

---

Create Snapshot

Using CLI

```bash
aws ec2 create-snapshot \
--volume-id vol-12345 \
--description "Before deployment"
```

AWS begins snapshot.

---

Restore

Create new volume

```bash
aws ec2 create-volume \
--snapshot-id snap-12345
```

Attach

```
EC2

↓

Attach Volume
```

Mount

```
mount /dev/nvme1n1 /data
```

Recovered.

---

Problems

Snapshot takes time.

Large volumes

```
2TB
```

can take hours before fully initialized, although many cloud providers allow immediate use while blocks are fetched on demand.

---

Problem

Application consistency.

Imagine

MySQL writing

```
Transaction

Half complete
```

Snapshot occurs.

Database becomes inconsistent.

Need

```
FLUSH TABLES WITH READ LOCK;
```

or use database-aware backup mechanisms before taking snapshots.

---

# What is Backup?

Backup means

Copy data

to another location

for recovery.

Example

```
Production

↓

Backup Server
```

or

```
AWS S3
```

or

```
Tape
```

or

```
NAS
```

Backup exists independently.

If production burns

Backup survives.

---

Difference

Snapshot

```
Same storage system

Fast

Mostly block level

Temporary
```

Backup

```
Independent copy

Long-term

Different location

Disaster recovery
```

---

Think like this

Snapshot

Undo button.

Backup

Insurance policy.

---

# Backup Types

```
1 Full

2 Incremental

3 Differential
```

---

## Full Backup

Copies everything.

```
Server

500GB

↓

Backup

500GB
```

Simple.

Restore easy.

Slow.

---

Incremental

Monday

```
500GB
```

Tuesday

Only changed

```
5GB
```

Wednesday

```
2GB
```

Storage efficient.

Restore

Need

```
Full

+

Every Increment
```

---

Differential

Monday

```
500GB
```

Tuesday

```
5GB
```

Wednesday

```
7GB
```

Thursday

```
10GB
```

Every backup stores changes since the last full backup. Restore needs the full backup plus the latest differential.

---

# Practical Linux Backup

Simplest

Using

```
tar
```

Backup

```
tar \
-czvf \
backup.tar.gz \
/etc \
/home \
/var/www
```

Meaning

```
c

create
```

```
z

gzip
```

```
v

verbose
```

```
f

file
```

---

Restore

```
tar \
-xzvf \
backup.tar.gz \
-C /
```

Files restored.

---

# rsync Backup

Very common.

```
rsync \
-avh \
/var/www \
/backup/
```

Only changed files copied.

Very fast.

---

Restore

```
rsync \
-avh \
/backup/ \
/
```

---

# Database Backup

Never trust filesystem backup alone.

MySQL

```
mysqldump \
-u root \
-p \
database \
> backup.sql
```

Restore

```
mysql \
-u root \
-p \
database \
< backup.sql
```

For very large databases, physical backup tools (such as Percona XtraBackup for MySQL/MariaDB) are often preferred because they are faster and support hot backups.

---

# Enterprise Backup Tools

Common production tools include:

* BorgBackup
* Restic
* Duplicity
* Bacula
* Amanda
* Veeam (especially in virtualized environments)
* Enterprise cloud backup services

These support:

* encryption
* compression
* deduplication
* scheduling
* retention policies
* verification

---

# Production Strategy

Typical company

```
Application

↓

Daily Database Backup

↓

Nightly Filesystem Backup

↓

Weekly Full Backup

↓

Monthly Archive

↓

S3
```

Another layer

```
EBS Snapshot

Every 6 hours
```

Now you have

```
Quick recovery

+

Long-term recovery
```

---

# Production Deployment Example

Before deployment

```
Take Snapshot

↓

Deploy

↓

Success

Delete snapshot

Failure

Restore snapshot
```

This is very common.

---

# Best Practices

For production systems, follow these principles:

1. Follow the **3-2-1 rule**:

   * 3 copies of your data
   * 2 different storage media
   * 1 copy stored off-site or in another region

2. Test restores regularly. A backup that cannot be restored is effectively useless.

3. Encrypt backups, especially before sending them off-site.

4. Automate backups with schedulers such as `cron` or systemd timers.

5. Monitor backup jobs and alert on failures.

6. Verify backup integrity with checksum validation or restore tests.

7. For databases, use database-aware backup methods instead of only copying database files while the database is running.

8. Define retention policies (for example, keep daily backups for 30 days, weekly for 3 months, monthly for 1 year).

9. Document the recovery procedure so it can be executed during an incident.

10. Periodically perform full disaster recovery drills.

---

# Interview Question (Very Common)

> **Interviewer:** Your production EC2 instance becomes corrupted after a deployment. How would you recover it?

A strong answer would be:

1. Determine whether the problem is application-level, filesystem-level, or infrastructure-level.
2. If the deployment caused the issue and a recent pre-deployment EBS snapshot exists, create a new EBS volume from that snapshot.
3. Attach the restored volume to a recovery instance or replace the affected volume, depending on the recovery plan.
4. Restore any newer database transactions from database backups or transaction logs if necessary.
5. Validate application health, data integrity, and monitoring before putting the service back into production.
6. Perform a root cause analysis and document the incident to prevent recurrence.

---

## What you should know for a Mid-level DevOps interview

By the end of this topic, you should be comfortable explaining and demonstrating:

* The difference between snapshots and backups, and when to use each.
* LVM snapshots: creation, mounting, restoration, and limitations.
* Cloud snapshots (such as EBS snapshots): creation, restoration, and consistency considerations.
* File-level backups using `tar` and `rsync`.
* Database backups using logical and physical backup tools.
* Full, incremental, and differential backup strategies.
* Recovery Point Objective (RPO) and Recovery Time Objective (RTO), and how they influence backup design.
* The 3-2-1 backup rule and retention policies.
* Backup automation, monitoring, verification, and periodic restore testing.

Mastering these concepts will prepare you for many Linux, cloud, and DevOps interview scenarios because they combine systems administration, storage, cloud infrastructure, and disaster recovery into one practical skill set.
