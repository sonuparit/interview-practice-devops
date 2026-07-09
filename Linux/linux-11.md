This is an excellent topic for a Mid-level DevOps interview because it combines **Linux internals + performance tuning + production troubleshooting**.

Many candidates know Docker, Kubernetes, and AWS, but struggle when the interviewer asks:

> "Why did the server suddenly become slow even though CPU usage is low?"

Often the answer is related to **memory management**, **swap**, or the **OOM Killer**.

---

# Memory Hierarchy

Before learning swap, understand Linux memory.

```
CPU Registers
      │
      ▼
CPU Cache (L1/L2/L3)
      │
      ▼
RAM (Main Memory)
      │
      ▼
Swap (Disk)
      │
      ▼
Storage (SSD/HDD)
```

RAM is extremely fast.

Swap is very slow because it lives on disk.

Linux uses swap only when necessary.

---

# 1. Swap File

## 1. What is this?

A swap file is a **regular file on the filesystem** that Linux treats as extra virtual memory.

Example

```
/swapfile
```

Linux can move inactive memory pages from RAM into this file.

It behaves exactly like swap partition.

---

## 2. Why is it required?

RAM is limited.

Suppose

```
RAM = 8 GB

Applications need

Chrome      3 GB
VSCode      2 GB
Docker       4 GB

Total = 9 GB
```

Linux cannot fit everything into RAM.

Instead of immediately killing applications,

Linux moves inactive pages into swap.

---

## 3. How does it help in Mid-Level DevOps?

You'll:

* create swap on cloud servers
* resize swap
* troubleshoot memory pressure
* optimize database servers
* prevent OOM kills

---

## 4. Why recruiters care?

Because memory problems are among the most common production issues.

Example

```
Java server crashed.

Was RAM exhausted?

Was swap disabled?

Was OOM Killer invoked?

```

A DevOps engineer should know how to answer these.

---

## 5. Production problem solved

Example

```
Server

RAM = 2GB

Application occasionally spikes to 2.3GB
```

Without swap

```
OOM Killer
↓

Application dies
```

With 2GB swap

```
Temporary spike handled

Application survives
```

---

## 6. Troubleshooting example

```
free -h
```

Output

```
Swap: 2G
Used: 2G
```

Problem

Swap completely full.

Next

```
top
```

Find large memory consumers.

---

## 7. Dependencies

Affected services:

* Databases
* Java
* Docker
* Kubernetes nodes
* Redis
* Elasticsearch

Why?

They consume lots of memory.

---

## 8. Analogy

RAM = Office desk

Swap = Cabinet

```
Desk full?

Move old files

↓

Cabinet

Need them again?

Bring them back.

```

Cabinet works.

But it's much slower than desk.

---

## 9. Three questions

Can you answer?

1.

Why is swap slower than RAM?

2.

Can swap replace RAM?

3.

When should swap be used?

---

# 2. Swap Partition

---

## 1. What is it?

A dedicated disk partition reserved for swap.

Example

```
/dev/sda3
```

instead of

```
/swapfile
```

---

## Difference

Swap File

```
Filesystem

↓

Regular file
```

Swap Partition

```
Dedicated partition

↓

No filesystem
```

---

## Why required?

Historically

Partitions were faster.

Today

Performance difference is tiny (especially SSDs).

---

## Production

Cloud providers mostly use swap files because they are easy to resize.

---

## Troubleshooting

```
cat /proc/swaps
```

Shows

```
Filename
Type
Priority
```

---

## Dependencies

Same as swap file.

---

## Analogy

Swap partition

=

Separate warehouse.

Swap file

=

Room inside warehouse.

---

## Three questions

1.

Which is easier to resize?

2.

Which is commonly used in cloud?

3.

Can Linux use multiple swap devices?

(Answer: Yes.)

---

# swapon

## What?

Enables swap.

```
sudo swapon /swapfile
```

or

```
sudo swapon -a
```

Enable every swap in

```
/etc/fstab
```

---

## Why?

Linux won't use swap until enabled.

---

## Production

Suppose

```
Swap exists

but

disabled
```

Memory spikes.

OOM Killer starts.

Solution

```
swapon
```

---

## Troubleshooting

```
swapon --show
```

Output

```
NAME
TYPE
SIZE
USED
```

---

## Three questions

1.

Does creating swap automatically enable it?

(No.)

2.

How do you check active swap?

3.

What does swapon -a do?

---

# swapoff

---

## What?

Disable swap.

```
sudo swapoff /swapfile
```

---

## Why?

Maintenance

Resize swap

Delete swap

---

## Production

Need larger swap.

```
swapoff

delete

create new

swapon
```

---

## Troubleshooting

If

```
swapoff
```

hangs

Reason

Linux is moving swapped pages back into RAM.

Need enough free RAM.

---

## Three questions

1.

Why may swapoff take time?

2.

Can swapoff fail?

3.

Should you swapoff on production under memory pressure?

(No.)

---

# free

---

## What?

Displays memory usage.

```
free -h
```

Example

```
              total
RAM           16G
used           8G
free           2G
buff/cache     6G

Swap
2G
0G used
```

---

## Why?

Quick health check.

---

## Production

First command when memory issue occurs.

---

## Troubleshooting

Example

```
Swap Used = 0

RAM = 98%
```

Maybe swappiness is low.

---

## Dependencies

Everything using RAM.

---

## Three questions

1.

Why isn't cache bad?

2.

What does available memory mean?

3.

Difference between free and available?

---

# Swappiness

---

## What?

Kernel parameter deciding **how aggressively Linux moves memory pages from RAM to swap**.

```
0–100
```

Check

```
cat /proc/sys/vm/swappiness
```

Temporary

```
sudo sysctl vm.swappiness=10
```

Permanent

```
/etc/sysctl.conf
```

---

## Meaning

```
0

↓

Avoid swap

100

↓

Use swap aggressively
```

Ubuntu default

```
60
```

---

## Why?

Different workloads need different behavior.

---

## DevOps benefit

Databases

Need RAM.

Use

```
10
```

Desktop

Maybe

```
60
```

---

## Recruiters care

Shows Linux tuning knowledge.

---

## Production example

MySQL

```
High swap

↓

Slow queries
```

Reduce

```
swappiness
```

Performance improves.

---

## Troubleshooting

```
vmstat 1
```

Observe

```
si

so
```

Swap in/out.

Heavy swapping

↓

Tune swappiness.

---

## Dependencies

Databases

Redis

Java

Kubernetes

---

## Analogy

Imagine moving books.

High swappiness

↓

Move books to storage quickly.

Low swappiness

↓

Keep books on your desk.

---

## Three questions

1.

Default Ubuntu swappiness?

2.

Should database servers use high swappiness?

3.

Can swappiness disable swap?

(No.)

---

# OOM Killer

OOM = Out Of Memory

---

## What?

Kernel mechanism that kills processes when memory is exhausted.

Without OOM

Entire system freezes.

---

## Example

```
RAM

100%

Swap

100%

No memory left.
```

Kernel chooses

```
Kill process
```

---

## Why?

Protect kernel stability.

---

## DevOps benefit

You can identify

Why application suddenly exited.

---

## Recruiters care

Extremely common production interview topic.

---

## Production example

```
Java service

Killed

No logs
```

Check

```
dmesg
```

or

```
journalctl -k
```

Output

```
Out of memory:

Killed process 3842
java
```

Now you know.

OOM Killer.

---

## Troubleshooting workflow

```
Application died

↓

journalctl -k

↓

dmesg

↓

free

↓

top

↓

ps aux --sort=-%mem
```

Find memory hog.

---

## Dependencies

Everything using RAM.

Especially

* Kubernetes Pods
* Docker Containers
* Databases
* Java
* Node.js

---

## Analogy

Restaurant.

50 seats.

51 customers arrive.

Manager asks one customer to leave.

Restaurant continues operating.

OOM Killer does exactly that.

---

## Three questions

1.

When does OOM Killer activate?

2.

How do you check if OOM Killer killed your process?

3.

Can OOM Killer kill any process?

(Practically yes, but the kernel scores processes using `oom_score` and `oom_score_adj` to decide which is the best candidate. Critical processes can be made less likely to be killed.)

---

# Production Troubleshooting Flow

```
Application Slow
        │
        ▼
free -h
        │
        ▼
RAM Full?
        │
        ▼
YES
        │
        ▼
top / htop
        │
        ▼
High Memory Process?
        │
        ▼
YES
        │
        ▼
Check Swap
        │
        ▼
Swap Full?
        │
        ▼
YES
        │
        ▼
Check Swappiness
        │
        ▼
Check OOM Logs
        │
        ▼
Increase RAM?
Tune Application?
Add Swap?
```

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What is the purpose of swap in Linux?

Swap extends virtual memory by allowing inactive memory pages to be moved from RAM to disk. It helps the system survive temporary memory pressure but is much slower than RAM.

---

### 2. What is the difference between a swap file and a swap partition?

A swap file is a regular file on an existing filesystem, while a swap partition is a dedicated disk partition with no filesystem. Functionally they are almost identical, but swap files are easier to create, resize, and manage, which is why they are common in cloud environments.

---

### 3. Does swap increase system performance?

No. Swap generally decreases performance because disk I/O is far slower than RAM. Its primary purpose is to prevent crashes during temporary memory shortages.

---

### 4. How do you check memory and swap usage?

Use:

```bash
free -h
swapon --show
cat /proc/swaps
```

---

### 5. How do you enable and disable swap?

Enable:

```bash
sudo swapon /swapfile
```

Disable:

```bash
sudo swapoff /swapfile
```

---

### 6. What is swappiness?

Swappiness is a kernel parameter (`0–100`) that controls how aggressively Linux moves memory pages from RAM to swap.

---

### 7. Why might you reduce swappiness on a database server?

Databases perform best when their working data stays in RAM. Lowering swappiness reduces the chance that active database pages will be swapped out, improving query performance.

---

### 8. What is the OOM Killer?

The Out-Of-Memory (OOM) Killer is a kernel mechanism that terminates one or more processes when the system has exhausted both RAM and usable swap, preventing the entire system from hanging.

---

### 9. How do you determine whether the OOM Killer terminated your application?

Check kernel logs:

```bash
journalctl -k
```

or

```bash
dmesg
```

Look for messages containing `Out of memory` or `Killed process`.

---

### 10. Why is a server slow even when CPU usage is low?

One common reason is heavy swapping. If the system spends most of its time reading and writing memory pages to disk (swap thrashing), CPU utilization may remain low while overall performance becomes very poor.

---

### 11. What is swap thrashing?

Swap thrashing occurs when the system constantly moves pages between RAM and swap because there isn't enough memory for the workload. This results in severe performance degradation.

---

### 12. Is it safe to disable swap on a production server?

Only if you have enough free RAM. Disabling swap forces all swapped pages back into RAM, which can trigger the OOM Killer if memory is insufficient.

---

### 13. What would you check if swap usage is constantly increasing?

I would inspect overall memory usage (`free -h`), identify the largest memory consumers (`top`, `ps aux --sort=-%mem`), review application memory leaks, check swappiness, and determine whether the server simply needs more RAM.

---

### 14. How does swap relate to Kubernetes?

On Kubernetes worker nodes, excessive swap can lead to unpredictable application latency. For this reason, Kubernetes has historically required swap to be disabled, though newer versions offer limited configurable support. In interviews, it's important to know that many production Kubernetes clusters still operate with swap disabled.

---

### 15. When would you add more RAM instead of increasing swap?

If the workload regularly exceeds available memory, adding RAM is the correct long-term solution. Swap is intended for temporary memory pressure and cannot replace the performance of physical memory.
