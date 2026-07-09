# RAID (Redundant Array of Independent Disks)

RAID is one of the most important storage topics for Linux System Administrators and DevOps Engineers.

A lot of candidates misunderstand RAID. They think RAID is a backup mechanism. **It is not.**

Think of RAID as **a way to combine multiple physical disks into one logical storage system to improve reliability, performance, or both.**

---

# 1. What is RAID?

RAID stands for

> **Redundant Array of Independent Disks**

It combines two or more physical disks into one logical storage volume.

Example

```
Without RAID

Disk1 (1TB)

Application stores data here
```

With RAID

```
Disk1 (1TB)
Disk2 (1TB)

↓

RAID Controller

↓

One Logical Disk (1TB or 2TB depending on RAID level)
```

The operating system usually sees

```
/dev/md0

or

/dev/sda
```

depending on how RAID is configured.

---

# 2. Why do we use RAID?

Hard disks fail.

SSDs fail.

NVMe drives fail.

Imagine this production database

```
Orders
Payments
Customers
Invoices
```

Everything is stored on one disk.

If that disk dies

```
Entire production goes down.
```

RAID helps reduce this risk.

---

# 3. What problem does RAID solve?

RAID mainly solves three problems.

---

## Problem 1 — Disk Failure

Without RAID

```
One Disk

↓

Disk fails

↓

Everything stops
```

With RAID1

```
Disk A
Disk B

Same data

Disk A dies

↓

Disk B continues serving requests

No downtime
```

---

## Problem 2 — Performance

Suppose one disk can read

```
200 MB/s
```

Four disks in RAID0

```
≈800 MB/s
```

because data is read from all disks simultaneously.

---

## Problem 3 — Larger Storage

Instead of

```
4 × 2TB
```

Applications can see

```
One logical volume
```

making storage management easier.

---

# 4. Do we use RAID in production?

**Yes.**

Almost every enterprise storage system uses RAID or RAID-like redundancy.

Examples:

* Database servers
* VMware clusters
* NAS appliances
* SAN storage
* Enterprise storage arrays
* Hardware storage controllers
* Cloud provider storage (often with redundancy underneath)

However, in cloud environments such as AWS, Azure, and GCP, you often don't configure RAID yourself because the cloud provider manages redundancy at the storage layer.

---

# 5. Why is RAID important for DevOps?

A DevOps engineer manages production infrastructure.

Imagine this situation.

```
Production PostgreSQL

↓

Disk fails

↓

Company loses customer orders
```

If RAID was configured correctly

```
Disk fails

↓

Database keeps running

↓

Replace failed disk later
```

This improves:

* High Availability
* Reliability
* Fault Tolerance
* Reduced Downtime

These are core DevOps concerns.

---

# 6. RAID Levels (Possible Combinations)

## RAID 0 (Striping)

Minimum disks

```
2
```

Data

```
A B C D

↓

Disk1

A
C

Disk2

B
D
```

Advantages

* Very fast
* Full storage capacity

Disadvantages

* No redundancy
* One disk failure destroys all data

Use case

* Temporary data
* Scratch space
* Video editing

---

## RAID 1 (Mirroring)

```
Disk1

A B C

Disk2

A B C
```

Advantages

* Disk redundancy
* Easy recovery

Disadvantages

* 50% storage efficiency

2TB + 2TB

↓

Usable

2TB

---

## RAID 5 (Striping + Parity)

Minimum

```
3 disks
```

Can survive

```
1 disk failure
```

Parity allows rebuilding the failed disk.

Example

```
Disk1

A

Disk2

B

Disk3

Parity
```

---

## RAID 6

Like RAID5

But

```
Two parity blocks
```

Can survive

```
2 disk failures
```

Requires at least 4 disks.

---

## RAID 10 (1+0)

Mirror

↓

Stripe

```
4 disks

Mirror

↓

Stripe
```

Advantages

* Fast
* Fault tolerant
* Excellent for databases

Disadvantage

Needs many disks.

---

# Summary Table

| RAID   | Minimum Disks | Fault Tolerance                | Performance | Storage Efficiency |
| ------ | ------------- | ------------------------------ | ----------- | ------------------ |
| RAID0  | 2             | None                           | Excellent   | 100%               |
| RAID1  | 2             | 1 disk per mirror pair         | Good        | 50%                |
| RAID5  | 3             | 1 disk                         | Good reads  | (N−1)/N            |
| RAID6  | 4             | 2 disks                        | Good reads  | (N−2)/N            |
| RAID10 | 4             | Multiple (one per mirror pair) | Excellent   | 50%                |

---

# which combination is mostly use in productions and why?

The short answer is:

| Workload                                          | Most Common RAID                                      |
| ------------------------------------------------- | ----------------------------------------------------- |
| Database servers                                  | RAID 10                                               |
| General application servers                       | RAID 1 or RAID 10                                     |
| NAS/File servers                                  | RAID 5 or RAID 6                                      |
| Virtualization hosts                              | RAID 10                                               |
| Cloud VMs (AWS EC2, Azure VM, GCP Compute Engine) | Usually **no RAID configured by the DevOps engineer** |

Let's understand why.

---

# 1. RAID 10 (Most common for production databases)

```
Disk1  <---- Mirror ----> Disk2
Disk3  <---- Mirror ----> Disk4

Stripe across both mirrors
```

Advantages

✅ Very fast reads

✅ Very fast writes

✅ Excellent fault tolerance

✅ Fast rebuild

Databases perform thousands of random reads and writes every second.

Examples

* PostgreSQL
* MySQL
* MariaDB
* Oracle
* Microsoft SQL Server

A slow storage subsystem directly impacts application latency, so RAID 10 is often preferred.

### Why not RAID 5?

RAID 5 must calculate parity on every write.

Example

```
Write request

↓

Read old blocks

↓

Calculate parity

↓

Write data

↓

Write parity
```

This additional work is called the **RAID 5 write penalty**, making it less suitable for write-heavy databases.

---

# 2. RAID 1 (Very common for operating systems)

Many production servers mirror only the OS disk.

```
Disk1

Ubuntu

Disk2

Ubuntu
```

If one disk dies

```
Server keeps booting.
```

Typical use:

* Linux OS
* `/boot`
* Small application servers
* Bastion hosts

---

# 3. RAID 5 (Common for file storage)

Suppose a company stores

* Videos
* Images
* PDFs
* Logs
* Documents

Mostly reads, fewer writes.

RAID 5 offers a good balance between:

* Capacity
* Cost
* Redundancy

Example:

```
8 disks

Usable storage ≈ 7 disks
```

Only one disk's worth of space is used for parity.

---

# 4. RAID 6 (Large storage systems)

Large storage arrays often use RAID 6 because rebuilding a failed large-capacity drive can take many hours or even days.

Imagine:

```
12 × 20TB disks
```

One disk fails.

During rebuild, another disk could fail.

With RAID 5:

```
Second disk failure

↓

Array lost
```

With RAID 6:

```
Second disk failure

↓

Still operational
```

That's why RAID 6 is common in:

* Backup storage
* Enterprise NAS
* Archival systems

---

# 5. RAID 0 (Rare in production)

RAID 0 provides excellent performance but **no redundancy**.

If one disk fails:

```
Entire array is lost.
```

It may be used for:

* Temporary rendering data
* Scratch space
* Build caches
* High-speed data that can be recreated

It is generally **not** used for critical production data.

---

# What do companies like Amazon, Google, and Microsoft use?

This depends on the layer you're talking about.

## Physical storage layer

Large cloud providers use sophisticated distributed storage systems with redundancy across many disks, servers, and often multiple locations. They do not rely solely on traditional RAID.

## Your Linux server

If you're managing your own physical servers:

* RAID 1 is common for OS disks.
* RAID 10 is common for databases and virtualization hosts.
* RAID 5 or RAID 6 is common for shared file storage.

## AWS EC2

If you're using Amazon Web Services with Amazon Elastic Block Store volumes:

```
EC2

↓

EBS Volume

↓

AWS-managed redundant storage
```

Since EBS already replicates data within an Availability Zone, many teams don't configure RAID for redundancy. They may still use RAID 0 to stripe multiple EBS volumes for higher throughput or larger capacity when needed.

---

# Which RAID should you choose?

| Scenario                                        | Recommended RAID | Reason                                                                   |
| ----------------------------------------------- | ---------------- | ------------------------------------------------------------------------ |
| Linux OS disk                                   | RAID 1           | Simple redundancy for boot and system files                              |
| Database server                                 | RAID 10          | High performance and fault tolerance                                     |
| Kubernetes worker with local persistent storage | RAID 10          | Better I/O for stateful workloads (if using local disks)                 |
| File server                                     | RAID 5           | Good balance of capacity and redundancy                                  |
| Backup server                                   | RAID 6           | Better protection during long rebuilds                                   |
| Video editing workstation                       | RAID 0           | Maximum performance when data can be recreated or is backed up elsewhere |

---

# What interviewers expect from a Mid-Level DevOps Engineer

A strong interview answer would be:

> "RAID isn't one-size-fits-all. For production databases, I'd typically choose RAID 10 because it provides excellent read/write performance, fast rebuilds, and good fault tolerance. For operating system disks, RAID 1 is a common choice because it's simple and resilient. For file servers where capacity matters more than write performance, RAID 5 or RAID 6 are often appropriate. In cloud environments like AWS using EBS, I usually rely on the storage redundancy provided by the cloud platform instead of configuring RAID for fault tolerance, unless I need RAID 0 to increase throughput or combine multiple volumes."

That answer demonstrates that you understand **both traditional on-premises infrastructure and modern cloud-based deployments**, which is what interviewers typically look for in a DevOps candidate.

# 7. Can RAID be used with LVM?

**Yes.**

This is extremely common.

Typical production stack:

```
Application

↓

Filesystem (ext4/xfs)

↓

LVM Logical Volume (LV)

↓

Volume Group (VG)

↓

Physical Volume (PV)

↓

RAID Device (/dev/md0)

↓

Physical Disks
```

RAID provides redundancy.

LVM provides flexibility.

They solve different problems.

---

# 8. Can RAID create snapshots?

**No.**

RAID itself has **no snapshot feature**.

Snapshots are provided by:

* LVM
* Filesystems like Btrfs
* ZFS
* Storage arrays
* Cloud storage (for example, block volume snapshots)

---

# 9. Can RAID create backups?

No.

RAID is **not** a backup.

Many beginners confuse these concepts.

```
RAID

Protects against

↓

Disk failure
```

Backup protects against

* Accidental deletion
* Ransomware
* File corruption
* Software bugs
* Human mistakes
* Fire
* Flood
* Data center loss

---

# 10. How do we restore snapshots on RAID?

If using LVM snapshots over RAID:

```
Disks

↓

RAID

↓

LVM

↓

Snapshot
```

Workflow:

1. Create snapshot.
2. Make changes.
3. Need rollback.
4. Merge or restore the snapshot.
5. Filesystem returns to the snapshot state.

The RAID layer is transparent during this process; snapshot operations occur at the LVM layer.

---

# 11. How do we restore backups on RAID?

Example

```
RAID1

↓

Filesystem corrupted

↓

Restore from backup
```

Steps:

1. Replace failed disk (if needed).
2. Rebuild the RAID array.
3. Format or repair the filesystem if required.
4. Restore files from backup using your backup tool (for example, `rsync`, `tar`, enterprise backup software, or cloud backups).
5. Verify application integrity.

RAID does not change the backup restoration process.

---

# 12. What interviewers expect you to know

For a Mid-level DevOps role, you should know:

### Basic

* What RAID is
* Why RAID is needed
* Difference between RAID and backup
* Difference between RAID0 and RAID1
* RAID5 vs RAID6
* RAID10 advantages
* Hardware RAID vs Software RAID
* Rebuilding a failed RAID array
* Monitoring RAID health
* When to use each RAID level

### Practical Linux knowledge

* Linux software RAID using `mdadm`
* Checking RAID status:

  ```bash
  cat /proc/mdstat
  ```
* Viewing RAID details:

  ```bash
  mdadm --detail /dev/md0
  ```
* Recognizing failed disks and rebuild states

For senior roles, interviewers may also expect familiarity with storage performance trade-offs, parity overhead, and recovery procedures.

---

# 13. Is RAID better than LVM?

No.

This is like asking:

> "Is a car engine better than a steering wheel?"

They solve different problems.

| Feature                       | RAID    | LVM                                                                    |
| ----------------------------- | ------- | ---------------------------------------------------------------------- |
| Combines disks                | ✅       | ✅                                                                      |
| Protects against disk failure | ✅       | ❌                                                                      |
| Resize volumes                | Limited | ✅                                                                      |
| Snapshots                     | ❌       | ✅                                                                      |
| Flexible storage management   | ❌       | ✅                                                                      |
| Mirroring                     | ✅       | Limited (depending on configuration)                                   |
| Striping                      | ✅       | Supported, but RAID is the typical solution for redundancy/performance |
| Backup                        | ❌       | ❌                                                                      |

In production, it's common to use both together.

---

# 14. 10 RAID Interview Questions (with Answers)

### Q1. What is RAID?

**Answer:** RAID combines multiple physical disks into a logical storage device to improve availability, performance, or both.

---

### Q2. Is RAID a backup?

**Answer:** No. RAID protects against hardware disk failures, but it does not protect against accidental deletion, corruption, malware, or site disasters. You still need backups.

---

### Q3. Which RAID provides the best performance?

**Answer:** RAID 0 provides the highest read/write performance because it stripes data across disks, but it offers no fault tolerance.

---

### Q4. Which RAID is best for a production database?

**Answer:** RAID 10 is a common choice because it provides high performance and strong fault tolerance. The best choice still depends on workload, budget, and capacity requirements.

---

### Q5. What is parity?

**Answer:** Parity is redundant information calculated from data blocks that allows reconstruction of lost data when a disk fails in parity-based RAID levels such as RAID 5 and RAID 6.

---

### Q6. What happens when a disk fails in RAID 5?

**Answer:** The array continues operating in a degraded state. After replacing the failed disk, the RAID rebuilds the missing data using parity information.

---

### Q7. Difference between RAID 5 and RAID 6?

**Answer:** RAID 5 tolerates one disk failure, while RAID 6 tolerates two simultaneous disk failures by storing two sets of parity.

---

### Q8. Can LVM be used on top of RAID?

**Answer:** Yes. A common production design is to create a RAID device first and then use it as an LVM Physical Volume, combining RAID's redundancy with LVM's flexibility.

---

### Q9. How do you check software RAID status in Linux?

**Answer:**

```bash
cat /proc/mdstat
```

or

```bash
mdadm --detail /dev/md0
```

---

### Q10. What is the difference between hardware RAID and software RAID?

**Answer:** Hardware RAID uses a dedicated RAID controller that manages the array independently of the operating system, often with better performance and battery-backed cache. Software RAID is managed by the operating system (commonly with `mdadm` on Linux), is less expensive, and is widely used in Linux servers.
