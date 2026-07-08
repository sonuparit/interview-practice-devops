These topics are **extremely important** for a Mid-level DevOps Engineer because they are directly related to **production storage management, disaster recovery, business continuity, and uptime**.

If Kubernetes is about **running applications**, then LVM, backups, snapshots, RPO, and RTO are about **protecting the application's data**.

---

# 1. PV (Physical Volume)

## 1. What is this?

A **Physical Volume (PV)** is a storage device (or partition) that has been initialized for LVM.

It can be

* Entire disk (`/dev/sdb`)
* Partition (`/dev/sdb1`)
* RAID device
* SAN disk
* Cloud block storage (AWS EBS)

Example

```
Disk
 ┌──────────────┐
 │ /dev/sdb     │
 └──────────────┘

pvcreate /dev/sdb
```

Now Linux treats it as an LVM disk.

---

## 2. Why required?

Without PV, LVM cannot manage storage.

Think of PV as the raw material.

```
Raw Disk
      ↓
Physical Volume
      ↓
Volume Group
      ↓
Logical Volume
      ↓
Filesystem
```

---

## 3. How it helps in DevOps?

Imagine production server needs another 500GB.

Without LVM

```
New disk
↓

Application migration
↓

Downtime
```

With LVM

```
Attach disk
↓

pvcreate

↓

vgextend

↓

lvextend

↓

resize filesystem

Done
```

Almost no downtime.

---

## 4. Why recruiters care?

Because production storage grows.

Developers don't stop writing logs.

Databases keep increasing.

Someone must safely add storage.

---

## 5. Production problem solved

Application crashes because

```
No space left on device
```

Instead of migrating data

DevOps simply adds new disk

```
AWS EBS

↓

pvcreate

↓

vgextend

↓

lvextend

↓

resize2fs
```

Application continues.

---

## 6. Troubleshooting

Problem

```
Cannot extend LV
```

Check

```
pvs
vgs
lvs
```

Maybe new disk wasn't initialized.

Solution

```
pvcreate /dev/sdc
```

---

## 7. Dependencies

Depends on

* Block devices
* LVM metadata
* VG

Affects

* Logical Volumes
* Filesystems
* Databases

---

## 8. Analogy

Think of PV as **bricks**.

Bricks alone are useless.

Later they'll become rooms.

---

## 9. Test yourself

Can you answer

1. Can one PV be one partition?
2. Can multiple disks become multiple PVs?
3. Does application directly use PV?

---

# 2. VG (Volume Group)

## 1. What is this?

A Volume Group combines multiple PVs into one storage pool.

```
Disk1 100GB

Disk2 200GB

Disk3 300GB

↓

VG = 600GB
```

Applications don't know disks anymore.

They only know storage pool.

---

## 2. Why required?

Without VG

Every application must know

```
Which disk?
```

With VG

Everything comes from one pool.

---

## 3. DevOps benefit

Need 2TB?

No problem.

Just

```
Add disk

↓

vgextend
```

Done.

---

## 4. Recruiters

Storage pooling is used everywhere.

Cloud

VMware

Databases

Enterprise Linux

---

## 5. Production

Database growing quickly.

Instead of replacing disk

Just

```
vgextend
```

No migration.

---

## 6. Troubleshooting

```
vgs
```

Shows

```
Free space

Used space

Missing PV
```

---

## 7. Dependencies

Depends on

* PV

Affects

* LV

---

## 8. Analogy

Several water tanks connected together.

Application only sees one giant tank.

---

## 9. Test yourself

1. Why create VG?
2. Can VG contain multiple disks?
3. Can VG be extended?

---

# 3. LV (Logical Volume)

## 1. What is this?

Logical Volume behaves exactly like a partition.

Example

```
/dev/vgdata/lvdb
```

Filesystem lives here.

Applications store data here.

---

## 2. Why required?

Because partitions are rigid.

LVs are flexible.

Resize anytime.

---

## 3. DevOps

Production PostgreSQL

```
Database

↓

LV

↓

VG

↓

PV

↓

Disk
```

Need more storage?

```
lvextend
resize2fs
```

Done.

---

## 4. Recruiters

Every production database usually sits on LVM.

---

## 5. Production

```
95% disk usage
```

Increase LV.

No migration.

---

## 6. Troubleshooting

```
lvs
```

Check

```
LV active?

Snapshot exists?

Enough space?
```

---

## 7. Dependencies

Depends on

VG

Affects

Filesystem

Applications

Databases

---

## 8. Analogy

Apartment inside a building.

Building = VG

Apartment = LV

---

## 9. Test

1. Can LV shrink?
2. Can LV grow online?
3. Does application know about VG?

---

# 4. Snapshots

## 1. What is this?

Snapshot is a **point-in-time copy**.

Imagine

```
10:00 AM

Take snapshot

↓

11:00 AM

Data changes

↓

Snapshot still represents 10:00 AM
```

---

## 2. Why required?

Protection before risky operation.

Example

```
Database upgrade

Kernel upgrade

Filesystem resize

Application deployment
```

---

## 3. DevOps

Before migration

Take snapshot.

If deployment fails

Rollback.

---

## 4. Recruiters

Snapshots reduce downtime.

Fast rollback.

---

## 5. Production

Upgrade corrupts database.

Restore snapshot.

5 minutes.

---

## 6. Troubleshooting

Snapshot becomes full.

```
lvs
```

Shows

```
Data%
```

Need larger snapshot.

---

## 7. Dependencies

Depends

LV

Filesystem

Storage backend

---

## 8. Analogy

Taking photo before renovating house.

---

## 9. Test

1. Is snapshot backup?
2. Can snapshot fail?
3. Why snapshots slow down writes?

---

# 5. Backups

## 1. What is this?

Independent copy stored elsewhere.

```
Production

↓

Backup Server

↓

Cloud

↓

Tape
```

---

## 2. Why?

If production dies

Data survives.

---

## 3. DevOps

Accidental deletion.

Restore.

---

## 4. Recruiters

Backups are business survival.

---

## 5. Production

Developer

```
rm -rf
```

Restore backup.

---

## 6. Troubleshooting

Backup failed.

Check

Logs

Storage

Permissions

Network

Retention

Integrity

---

## 7. Dependencies

Storage

Network

Backup software

Cloud

Database consistency

---

## 8. Analogy

Photocopy kept in another city.

---

## 9. Test

1. Is snapshot backup?
2. Where should backup be stored?
3. Why verify backup?

---

# 6. RPO (Recovery Point Objective)

## 1. What is this?

Maximum acceptable amount of data loss.

Example

```
RPO = 10 minutes
```

Worst case

Lose 10 minutes of data.

---

## 2. Why?

Business decides acceptable loss.

---

## 3. DevOps

Backup frequency depends on RPO.

```
RPO

↓

Backup every 5 min

Replication

Continuous backup
```

---

## 4. Recruiters

Infrastructure must meet business SLA.

---

## 5. Production

Database crashes.

Latest backup 5 min old.

Lost 5 min.

RPO achieved.

---

## 6. Troubleshooting

Ask

```
Why lost 2 hours?

Backup failed?

Replication stopped?

Monitoring missing?
```

---

## 7. Dependencies

Backups

Replication

Storage

Monitoring

---

## 8. Analogy

Notebook saved every 5 minutes.

Power failure loses only 5 minutes.

---

## 9. Test

1. Does RPO measure downtime?
2. Lower RPO means?
3. Which technologies improve RPO?

---

# 7. RTO (Recovery Time Objective)

## 1. What is this?

Maximum acceptable downtime.

Example

```
RTO = 30 minutes
```

Must recover within 30 minutes.

---

## 2. Why?

Business cannot wait forever.

---

## 3. DevOps

Design automation.

Fast restore.

Infrastructure as Code.

---

## 4. Recruiters

Shows disaster recovery skills.

---

## 5. Production

Server dies.

Terraform recreates infrastructure.

Ansible configures it.

Restore backup.

Application online within 20 minutes.

RTO achieved.

---

## 6. Troubleshooting

Recovery took 4 hours.

Find bottleneck.

```
Backup slow?

Network?

Manual steps?

Storage?

DNS?

Database restore?
```

---

## 7. Dependencies

Automation

IaC

Backups

DNS

Cloud

Monitoring

---

## 8. Analogy

House catches fire.

Question is

"How fast can family move back?"

That's RTO.

---

## 9. Test

1. Does RTO measure data loss?
2. What reduces RTO?
3. Difference between RTO and uptime?

---

# Relationship Between All Topics

```
          Physical Disks
                │
         pvcreate (PV)
                │
                ▼
      +--------------------+
      |   Volume Group     |
      | (Storage Pool)     |
      +--------------------+
                │
      lvcreate / lvextend
                │
                ▼
      +--------------------+
      | Logical Volume     |
      +--------------------+
                │
           mkfs.ext4/xfs
                │
                ▼
         Mount Filesystem
                │
                ▼
        Applications / Databases
                │
      ┌─────────┴─────────┐
      ▼                   ▼
  Snapshot           Backup
(Point-in-time)   (Independent copy)
      │                   │
      └─────────┬─────────┘
                ▼
       Disaster Recovery
        ├─ RPO: How much data can we lose?
        └─ RTO: How quickly must we recover?
```

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What is the difference between a Physical Volume, Volume Group, and Logical Volume?

**Answer:** A Physical Volume (PV) is a disk or partition initialized for LVM. One or more PVs are combined into a Volume Group (VG), which acts as a storage pool. Logical Volumes (LVs) are created from the VG and behave like flexible partitions that can host filesystems and applications.

---

### 2. Why do organizations prefer LVM over traditional partitions?

**Answer:** LVM provides flexibility. You can extend or shrink logical volumes, combine multiple disks into one storage pool, take snapshots, and add storage with minimal or no downtime. Traditional partitions are fixed and much harder to resize safely.

---

### 3. Your production database has reached 100% disk usage. How would you increase storage with minimal downtime?

**Answer:** Attach a new disk (or increase the cloud block volume), initialize it as a PV (`pvcreate`), extend the VG (`vgextend`), extend the LV (`lvextend`), and then grow the filesystem (for example, `resize2fs` for ext4 or `xfs_growfs` for XFS). Verify the result with `df -h`.

---

### 4. What is an LVM snapshot, and how is it different from a backup?

**Answer:** An LVM snapshot is a point-in-time copy of a logical volume stored in the same storage system, mainly used for quick rollback. A backup is an independent copy stored on separate media or locations and is intended for disaster recovery. If the underlying storage fails, snapshots are typically lost, while backups remain available.

---

### 5. When would you take a snapshot in production?

**Answer:** Before risky operations such as database upgrades, operating system upgrades, filesystem resizing, major deployments, or configuration changes. If the change fails, the snapshot can be used for a rapid rollback.

---

### 6. Why is a snapshot not considered a complete backup?

**Answer:** Because snapshots usually reside on the same storage system as the original data. If that storage is corrupted or destroyed, both the original volume and its snapshots may be lost. Backups are stored independently to protect against storage failures and site-wide disasters.

---

### 7. What are RPO and RTO?

**Answer:** Recovery Point Objective (RPO) defines the maximum acceptable amount of data loss, measured in time. Recovery Time Objective (RTO) defines the maximum acceptable downtime before the service must be restored.

---

### 8. Give an example of RPO and RTO.

**Answer:** If an online payment system has an RPO of 5 minutes and an RTO of 20 minutes, then after a disaster the business can tolerate losing up to 5 minutes of transactions, and the service must be operational again within 20 minutes.

---

### 9. How would you achieve a very low RPO?

**Answer:** Use more frequent backups, continuous replication, database replication, transaction log shipping, storage replication, and monitor backup jobs so they don't silently fail.

---

### 10. How would you reduce RTO?

**Answer:** Automate recovery with Infrastructure as Code (Terraform), configuration management (Ansible), automated deployment pipelines, tested recovery procedures, and fast restoration processes.

---

### 11. What commands would you use to troubleshoot LVM?

**Answer:** `pvs` to inspect Physical Volumes, `vgs` to inspect Volume Groups, `lvs` to inspect Logical Volumes, `lsblk` to view block devices, `df -h` to check filesystem usage, `mount` to verify mounted filesystems, and `journalctl` or system logs for LVM-related errors.

---

### 12. What happens if the Volume Group has no free space?

**Answer:** Logical volumes in that VG cannot be extended until additional storage is added (for example, by adding another PV to the VG) or unused space is freed.

---

### 13. Why do backups sometimes fail in production?

**Answer:** Common reasons include insufficient storage, network interruptions, permission issues, failed backup agents, inconsistent application data, storage quota exhaustion, or backup schedules that never ran successfully. Monitoring and regular restore testing are essential.

---

### 14. How would you verify that a backup is actually usable?

**Answer:** Perform periodic test restores in a non-production environment, verify data integrity, confirm applications can start using the restored data, and ensure recovery meets the required RPO and RTO. A backup that has never been restored is not proven to be reliable.

---

### 15. A manager asks you to improve disaster recovery. What areas would you evaluate?

**Answer:** I would review the current backup strategy, snapshot usage, backup frequency, retention policies, replication, restore procedures, recovery automation, monitoring, documentation, and regularly test recovery drills. I would compare the current capabilities against the business's required RPO and RTO and address any gaps.

---

## What a Mid-level DevOps Engineer should be able to explain confidently

By the end of your interview preparation, you should be able to explain:

* The complete LVM architecture (PV → VG → LV → Filesystem → Mount → Application).
* How to safely extend storage online with minimal downtime.
* The difference between snapshots and backups, including when to use each.
* How RPO influences backup frequency and replication strategy.
* How RTO influences automation, restore procedures, and infrastructure design.
* Practical troubleshooting of LVM, storage exhaustion, failed backups, and disaster recovery scenarios.

These are the kinds of production-focused discussions that distinguish a mid-level DevOps engineer from someone who only knows the commands.
