This is an excellent set of topics because these are exactly the concepts that separate a **Junior DevOps Engineer** from a **Mid-level DevOps Engineer**.

Many production outages are **not caused by Kubernetes or Terraform**.
They're caused by **memory pressure, cache behavior, disk I/O, and storage failures.**

Let's go through each one.

---

# 1. RAM (Random Access Memory)

## 1. What is this?

RAM is the computer's **physical working memory**.

It stores:

* running processes
* application data
* kernel data
* caches
* buffers

Unlike disk, RAM is extremely fast.

Approximate speed:

* SSD → 100–500 MB/s
* NVMe → 3–7 GB/s
* RAM → 20–60 GB/s+

---

## 2. Why is it required?

CPU can only process data efficiently if it is in RAM.

Without RAM:

* every read
* every calculation
* every program

would need disk access.

The system would become extremely slow.

---

## 3. How does this help in DevOps?

You'll understand:

* why pods get OOMKilled
* why EC2 becomes slow
* why Jenkins hangs
* why database performance drops
* why swap is being used

---

## 4. Why recruiters care?

Because production failures often start with memory exhaustion.

Example:

```
Java application
↓

Consumes 15 GB RAM

↓

Server has 16 GB

↓

Kernel starts swapping

↓

Application becomes slow

↓

Health checks fail

↓

Kubernetes restarts Pod
```

A DevOps engineer should immediately recognize this.

---

## 5. Production problem solved

Example:

```
Website suddenly becomes slow.
```

You check:

```
free -h
```

Output

```
RAM Used : 31 GB
RAM Free : 500 MB
Swap Used : 6 GB
```

Problem:

Server is swapping.

Root cause:

Memory pressure.

---

## 6. Troubleshooting

Useful commands

```
free -h

vmstat 1

top

htop

sar -r

cat /proc/meminfo
```

---

Example

```
OOM Killer killed Java process.
```

Check

```
dmesg | grep -i oom
```

Now you know RAM exhaustion caused it.

---

## 7. Dependencies affected

Everything.

Especially:

* Databases
* JVM
* Kubernetes
* Docker
* Redis
* Nginx workers

---

## 8. Analogy

RAM is your office desk.

Disk is the storage room.

The larger your desk,

the fewer trips you make to storage.

---

## 9. Test yourself

Can you answer:

1. Why is RAM faster than SSD?
2. What happens when RAM becomes full?
3. Why doesn't Linux keep RAM empty?

---

# 2. Cache

## 1. What is Cache?

Cache stores frequently used data so it can be accessed faster.

Linux uses unused RAM as cache.

Example:

```
Open same file twice.

First time:
Disk read.

Second time:
RAM cache.
```

---

## 2. Why required?

Disk is slow.

Cache avoids repeated disk reads.

---

## 3. DevOps benefit

You'll understand why:

```
Application became much faster after first request.
```

Nothing magical happened.

Linux cached the file.

---

## 4. Recruiters care because

Many candidates mistakenly think:

```
Free RAM = healthy server
```

Wrong.

Linux intentionally uses RAM for cache.

Unused RAM is wasted RAM.

---

## 5. Production problem

Example

Application startup

First request:

```
3 seconds
```

Second request

```
300 ms
```

Reason

Filesystem cache.

---

## 6. Troubleshooting

```
free -h
```

Example

```
used

buff/cache

available
```

Look at **available**, not free.

---

## 7. Dependencies

* filesystem
* databases
* nginx
* apache
* docker image layers

---

## 8. Analogy

Instead of walking to library every time,

you keep frequently used books on your table.

That's cache.

---

## 9. Test

1. Why does Linux use RAM as cache?
2. Is cached RAM wasted?
3. Can cache be reclaimed?

---

# 3. Buffers

## 1. What are Buffers?

Buffers temporarily store data during input/output operations.

Example

Writing file

```
Program

↓

Buffer

↓

Disk
```

---

## 2. Why required?

Disk writes are slower than CPU.

Buffers smooth out speed differences.

---

## 3. DevOps benefit

Understand disk write behavior.

---

## 4. Recruiters care

Because logs, databases and networking all use buffers.

---

## 5. Production problem

Slow disk writes.

Buffer fills.

Applications wait.

---

## 6. Troubleshooting

```
vmstat

iostat

sar -b
```

---

## 7. Dependencies

* disks
* filesystems
* networking
* logging

---

## 8. Analogy

Like a loading dock between factory and truck.

---

## 9. Test

1. Why buffer before writing?
2. Why not write directly?
3. Difference between cache and buffer?

---

# 4. Page Cache

## 1. What is Page Cache?

A special Linux cache for file contents.

When Linux reads files,

it stores pages in RAM.

---

## 2. Why required?

Reading disk repeatedly is expensive.

---

## 3. DevOps benefit

Database

Docker

Kubernetes

Nginx

all benefit.

---

## 4. Recruiters care

Because page cache dramatically affects performance.

---

## 5. Production problem

Large backup

↓

Consumes page cache

↓

Database cache evicted

↓

Database slows.

---

## 6. Troubleshooting

```
free -h

cat /proc/meminfo

Cached:
```

---

## 7. Dependencies

* databases
* web servers
* containers
* backups

---

## 8. Analogy

Recently opened pages of a book remain on your desk.

---

## 9. Test

1. What is page cache?
2. Can Linux reclaim it?
3. Why does database performance improve?

---

# 5. Virtual Memory

## 1. What is Virtual Memory?

Virtual memory gives each process the illusion that it has its own large, continuous address space, regardless of the actual physical RAM. The operating system maps virtual addresses to physical RAM and, if necessary, to swap space on disk.

Key points:

* Every process has its own virtual address space.
* The kernel uses page tables to translate virtual addresses to physical memory.
* Pages that are not actively used may be moved to swap if RAM is under pressure.

---

## 2. Why is it required?

Without virtual memory:

* Programs would need to know the exact physical memory layout.
* Memory protection between processes would be difficult.
* Running applications larger than available RAM would be impossible.

Virtual memory provides:

* Process isolation
* Memory protection
* Efficient memory allocation
* Controlled use of swap when RAM is full

---

## 3. DevOps benefit

Understanding virtual memory helps you:

* Diagnose high swap usage.
* Investigate Out Of Memory (OOM) events.
* Tune memory-intensive workloads like databases and JVM applications.
* Size cloud instances appropriately.

---

## 4. Why recruiters care?

Because many production incidents involve memory pressure. A DevOps engineer should understand why an application is slow even when "free" RAM appears low, and when swap usage is acceptable versus problematic.

---

## 5. Production problem

A server has:

* 16 GB RAM
* 8 GB Swap

A memory leak gradually consumes RAM. The kernel starts moving inactive pages to swap. Disk I/O increases, response times grow, and eventually the OOM killer terminates the leaking process.

---

## 6. Troubleshooting

Useful commands:

```bash
free -h
vmstat 1
sar -r
cat /proc/meminfo
swapon --show
```

Look for:

* High swap usage
* High page-in/page-out activity
* Low available memory

---

## 7. Dependencies

* JVM applications
* Databases
* Containers
* Kubernetes nodes
* Hypervisors

---

## 8. Analogy

Think of RAM as your desk and swap as a cabinet. Frequently used documents stay on the desk; rarely used ones are moved to the cabinet. Retrieving from the cabinet works but is much slower.

---

## 9. Test

1. What is virtual memory?
2. Does virtual memory always mean swap is being used?
3. Why does high swap usage usually reduce performance?

---

# 6. sar

## 1. What is it?

`sar` (System Activity Reporter) is part of the **sysstat** package. It records and displays historical system performance metrics such as CPU, memory, disk, network, and swap usage.

---

## 2. Why is it required?

Many production problems are intermittent. By the time you log in, CPU or memory usage may already be normal. `sar` provides historical data so you can investigate what happened earlier.

---

## 3. DevOps benefit

You can answer questions like:

* What was CPU usage at 2 AM?
* Was swap increasing before the outage?
* Was disk I/O saturated yesterday?

---

## 4. Why recruiters care?

They expect a mid-level engineer to investigate incidents using historical metrics, not just live monitoring.

---

## 5. Production problem

A nightly backup slows the application.

Using:

```bash
sar -b
```

shows a spike in disk activity exactly when the backup runs.

Root cause identified.

---

## 6. Troubleshooting

Common commands:

```bash
sar -u      # CPU
sar -r      # Memory
sar -B      # Paging
sar -b      # Block I/O
sar -n DEV  # Network
sar -q      # Load average
```

---

## 7. Dependencies

Useful for troubleshooting:

* CPU
* Memory
* Storage
* Network
* Swap
* Databases
* Containers

---

## 8. Analogy

`top` is like a live CCTV camera.

`sar` is like reviewing yesterday's CCTV recordings.

---

## 9. Test

1. What problem does `sar` solve that `top` cannot?
2. Which command shows historical memory usage?
3. Which command shows historical disk activity?

---

# 7. RAID

## 1. What is RAID?

RAID (Redundant Array of Independent Disks) combines multiple physical disks into a single logical storage unit to improve performance, fault tolerance, or both.

Common levels:

* RAID 0: Striping (performance, no redundancy)
* RAID 1: Mirroring (redundancy)
* RAID 5: Striping with distributed parity
* RAID 6: Double distributed parity
* RAID 10: Mirroring + striping

---

## 2. Why is it required?

RAID provides:

* Higher availability
* Better read/write performance (depending on the level)
* Protection against disk failures

---

## 3. DevOps benefit

You'll understand:

* Why a server survives a disk failure.
* How storage redundancy affects production uptime.
* How to plan maintenance without downtime.

---

## 4. Why recruiters care?

Storage reliability is critical in production. Mid-level engineers should know the trade-offs between performance, capacity, and fault tolerance.

---

## 5. Production problem

A disk fails in a database server.

If using RAID 1:

* The server continues running from the mirrored disk.
* Replace the failed disk and rebuild the array.

Without RAID:

* The server may become unavailable until storage is restored.

---

## 6. Troubleshooting

Useful commands (software RAID):

```bash
cat /proc/mdstat
mdadm --detail /dev/md0
lsblk
smartctl -a /dev/sda
```

Look for:

* Degraded arrays
* Failed disks
* Rebuild progress

---

## 7. Dependencies

RAID affects:

* Databases
* Virtual machines
* Kubernetes worker nodes
* Filesystems
* Backup systems

---

## 8. Analogy

Imagine writing important notes.

* RAID 0: Split one notebook across two friends for faster writing, but losing either notebook loses everything.
* RAID 1: Keep an identical copy in a second notebook.
* RAID 5: Share notes across several friends with enough extra information to reconstruct one lost notebook.
* RAID 10: Combine the safety of copies with the speed of working in parallel.

---

## 9. Test

1. What is the difference between RAID 0 and RAID 1?
2. Why can RAID not replace backups?
3. What happens when a RAID 5 array loses one disk?

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. Why does Linux use almost all available RAM?

**Answer:** Linux uses unused RAM for caches and buffers to improve performance. This memory is reclaimed automatically when applications need it.

---

### 2. Why is **Available** memory more important than **Free** memory?

**Answer:** Available memory estimates how much RAM can be given to applications without swapping. Free memory ignores reclaimable caches and is therefore less meaningful.

---

### 3. What is the difference between cache and buffer?

**Answer:** Cache stores frequently accessed data to speed up future reads, while buffers temporarily hold data during I/O operations to smooth differences in processing speed.

---

### 4. What is page cache?

**Answer:** Page cache stores file data in RAM after it is read from disk so subsequent reads can be served from memory instead of the storage device.

---

### 5. What is virtual memory?

**Answer:** Virtual memory gives each process its own logical address space, allowing isolation, protection, and efficient memory management while mapping addresses to physical RAM (and swap if needed).

---

### 6. Why is swap slower than RAM?

**Answer:** Swap resides on disk, which has much higher latency and lower throughput than physical memory.

---

### 7. What causes the OOM killer to terminate a process?

**Answer:** When the kernel cannot satisfy memory allocation requests, even after reclaiming memory and using swap, it invokes the OOM killer to free memory by terminating one or more processes.

---

### 8. How do you investigate a server that became slow due to memory pressure?

**Answer:** Check `free -h`, `vmstat`, `sar -r`, `top`, `dmesg | grep -i oom`, and `swapon --show` to identify RAM exhaustion, swapping, or OOM events.

---

### 9. What is `sar`, and why is it useful?

**Answer:** `sar` provides historical performance data for CPU, memory, paging, disk, and network usage, making it invaluable for investigating past incidents.

---

### 10. When would you use `sar` instead of `top`?

**Answer:** Use `top` for live monitoring and `sar` for historical analysis of performance trends.

---

### 11. What is RAID?

**Answer:** RAID combines multiple disks to improve performance, fault tolerance, or both, depending on the RAID level.

---

### 12. Why isn't RAID a backup?

**Answer:** RAID protects against hardware disk failure but not accidental deletion, corruption, malware, or site-wide disasters. Those require backups.

---

### 13. Which RAID levels are commonly used in production?

**Answer:** RAID 1 for mirroring, RAID 5/6 for capacity with redundancy, and RAID 10 for high-performance, high-availability workloads such as databases.

---

### 14. A server has low free memory but no swap usage. Should you be concerned?

**Answer:** Not necessarily. Linux intentionally uses RAM for caches. If available memory is healthy and the system isn't swapping, this is normal.

---

### 15. A server is swapping heavily. What would you check first?

**Answer:** Check for memory leaks, unusually large processes (`top`, `ps`), recent workload increases, OOM events (`dmesg`), `vmstat` paging activity, and whether the server has enough RAM for its workload.
