These are some of the most important Docker interview topics because **Docker is not a virtual machine.** Containers work because Linux provides features like **Namespaces**, **cgroups**, and **OverlayFS**.

If you understand these, you can explain **how Docker actually works internally**, which is exactly what mid-level interviewers expect.

---

# Docker Architecture

```
Application
      │
Docker CLI
      │
Docker Engine (dockerd)
      │
containerd
      │
runc
      │
Linux Kernel
      │
 ├── Namespaces
 ├── cgroups
 ├── OverlayFS
 └── Capabilities/Seccomp/AppArmor
```

Docker itself doesn't create isolation.

**Linux Kernel does.**

Docker simply tells the kernel:

* Create isolated process space
* Limit memory
* Limit CPU
* Mount filesystem
* Create virtual network

---

# 1. Namespaces

## What is it?

Namespaces isolate system resources.

Without namespaces, every process could see everything on the host.

Namespaces create the illusion that the container has its own machine.

Each container gets its own view.

Examples:

* own processes
* own network interfaces
* own filesystem
* own hostname

---

## Why required?

Imagine 200 applications running.

Without namespaces:

* all processes visible
* all network interfaces shared
* all host files visible

Chaos.

Namespaces isolate everything.

---

## Production Importance

Without namespace isolation:

```
Container A

kill -9 postgres
```

could kill PostgreSQL of another application.

Namespaces prevent this.

---

## Troubleshooting Example

Developer:

> My application cannot see process running on host.

Reason:

Container uses PID namespace.

Run:

```
lsns
```

or

```
docker inspect
```

Check namespace configuration.

---

## Analogy

Imagine apartment building.

Every apartment has:

* own rooms
* own bathroom
* own electricity

Same building.

Different isolated living spaces.

Namespaces are apartments.

---

## Test Yourself

1. Why are namespaces required?
2. Does Docker implement namespaces or Linux?
3. What happens if namespaces don't exist?

---

# 2. cgroups (Control Groups)

## What is it?

cgroups limit resources.

They control

* CPU
* RAM
* Disk I/O
* Network priority
* PIDs

---

## Why required?

Imagine one container:

```
while true:
    allocate memory
```

Consumes all RAM.

Whole server crashes.

cgroups stop that.

---

## Production Example

Limit memory:

```
docker run --memory=512m nginx
```

Container cannot exceed 512MB.

---

## Troubleshooting

Container exits.

```
Exit Code 137
```

Usually means:

Killed by OOM Killer.

Check

```
docker inspect
```

or

```
docker stats
```

or

```
dmesg
```

Look for

```
Out of memory
```

---

## Analogy

Hotel buffet.

Without limits:

One customer eats everything.

With cgroups:

Each guest gets one plate.

---

## Test Yourself

1. What problem do cgroups solve?
2. What is Exit Code 137?
3. Which resources can cgroups limit?

---

# 3. OverlayFS

## What is it?

Docker images contain layers.

OverlayFS merges them into one filesystem.

Example

Ubuntu layer

↓

Python layer

↓

Application layer

↓

Writable layer

Container sees

```
/
```

as one filesystem.

---

## Why required?

Without OverlayFS,

every image would copy entire Ubuntu.

Huge storage waste.

---

## Production Importance

100 containers using Ubuntu.

Without OverlayFS:

```
100 × Ubuntu
```

With OverlayFS:

Ubuntu stored once.

Huge disk savings.

---

## Troubleshooting

Problem:

```
No space left on device
```

Check

```
docker system df
```

Look for

* images
* layers
* dangling images

---

## Analogy

Transparent sheets.

Layer 1

```
Road
```

Layer 2

```
Buildings
```

Layer 3

```
Cars
```

Together appear as one picture.

OverlayFS stacks them.

---

## Test Yourself

1. Why are Docker images layered?
2. What problem does OverlayFS solve?
3. Where are changes written?

---

# 4. Union Filesystem

## What is it?

Union filesystem combines multiple directories into one logical filesystem.

OverlayFS is one implementation.

Others:

* AUFS
* Btrfs
* ZFS

---

## Why required?

Allows image reuse.

Every image reuses lower layers.

---

## Production

Without Union FS:

Every image duplicates

* Ubuntu
* Python
* Java

Huge storage.

---

## Troubleshooting

Slow image pulls?

Large duplicated layers?

Investigate image history.

```
docker history image
```

---

## Analogy

Several transparent folders stacked together.

Looks like one folder.

---

## Test Yourself

1. Difference between OverlayFS and Union FS?
2. Why layered images?
3. Why are image pulls fast?

---

# 5. PID Namespace

## What is it?

Provides separate process tree.

Container believes:

```
PID 1
```

is its first process.

Host sees another PID.

---

## Example

Inside container

```
ps
```

```
PID 1 nginx
```

Host

```
PID 25431 nginx
```

Same process.

Different namespace.

---

## Production

Prevents

```
kill
```

from affecting host processes.

---

## Troubleshooting

Need host processes?

```
docker run --pid=host
```

or

```
nsenter
```

---

## Analogy

Two office buildings.

Each building numbers employees

1

2

3

Same numbers.

Different buildings.

---

## Test Yourself

1. Why does every container have PID 1?
2. Can PID 1 differ on host?
3. Why isolate processes?

---

# 6. Network Namespace

## What is it?

Each container gets its own:

* IP
* routing table
* interfaces
* ports

---

## Why required?

Otherwise

every application binds

```
80
```

Conflict.

Namespaces solve that.

---

## Production

Container A

```
80
```

Container B

```
80
```

Works perfectly.

Each has own network namespace.

---

## Troubleshooting

Cannot reach application.

Check

```
docker network ls
```

```
docker inspect
```

```
ip addr
```

```
ip route
```

---

## Analogy

Each house has own WiFi router.

All use

```
192.168.1.x
```

No conflict.

---

## Test Yourself

1. Why can multiple containers use port 80?
2. Does every container get separate routing table?
3. Which namespace isolates networking?

---

# 7. Mount Namespace

## What is it?

Provides isolated filesystem mount points.

Container mounts

```
/
```

Host has different

```
/
```

---

## Why required?

Without it,

container could see every mounted disk.

---

## Production

Bind mount

```
-v /data:/backup
```

Only specified mount becomes visible.

---

## Troubleshooting

Volume missing.

Check

```
mount
```

inside container.

Check

```
docker inspect
```

Mounts section.

---

## Analogy

Each office has its own filing cabinet.

Employees only see their cabinet.

---

## Test Yourself

1. What is mount namespace?
2. Why are bind mounts possible?
3. Does container see host filesystem?

---

# 8. IPC Namespace

## What is it?

Isolates Inter-Process Communication resources.

Examples:

* Shared memory
* Message queues
* Semaphores

---

## Why required?

Without IPC namespace,

containers could interfere with each other's shared memory.

---

## Production

Database using shared memory.

Another container cannot access it.

---

## Troubleshooting

Application expects shared memory.

Run

```
ipcs
```

inside container.

Check if `/dev/shm` is sized correctly.

Some applications (e.g., Chrome, PostgreSQL) may require increasing shared memory.

---

## Analogy

Each office has its own meeting room.

Employees cannot overhear meetings from another office.

---

## Test Yourself

1. What does IPC namespace isolate?
2. Why is shared memory important?
3. Which applications commonly use IPC?

---

# How These Topics Help in Production

| Feature           | Real Production Problem Solved                    |
| ----------------- | ------------------------------------------------- |
| Namespaces        | Containers cannot interfere with each other       |
| cgroups           | One container cannot consume all CPU/RAM          |
| OverlayFS         | Saves disk space and speeds up image distribution |
| Union FS          | Reuses common image layers efficiently            |
| PID namespace     | Prevents accidental process interference          |
| Network namespace | Allows overlapping ports/IP isolation             |
| Mount namespace   | Restricts filesystem visibility                   |
| IPC namespace     | Prevents shared memory conflicts                  |

---

# 15 Important Mid-Level Interview Questions (with Answers)

### 1. Why are Docker containers lightweight compared to VMs?

**Answer:** Containers share the host Linux kernel and isolate resources using namespaces and cgroups, while VMs run a complete guest operating system with its own kernel.

---

### 2. Does Docker create isolation?

**Answer:** No. The Linux kernel provides isolation through namespaces. Docker configures and uses these kernel features.

---

### 3. What problem do namespaces solve?

**Answer:** They isolate system resources such as processes, networking, mounts, IPC, hostnames, and users so containers cannot directly interfere with one another.

---

### 4. What problem do cgroups solve?

**Answer:** They limit and account for resource usage (CPU, memory, disk I/O, PIDs, etc.), preventing a container from exhausting host resources.

---

### 5. What usually causes Docker Exit Code 137?

**Answer:** The process received `SIGKILL`, commonly because the kernel's OOM killer terminated it after it exceeded its memory limit or the host ran out of memory.

---

### 6. What is OverlayFS?

**Answer:** OverlayFS is a Linux filesystem driver that combines multiple read-only image layers with a writable container layer into a single unified filesystem view.

---

### 7. Where are container file changes stored?

**Answer:** In the container's writable layer (the upper layer). The underlying image layers remain read-only.

---

### 8. What is the difference between OverlayFS and a Union Filesystem?

**Answer:** A Union Filesystem is the general concept of merging multiple directories into one view. OverlayFS is one Linux implementation of that concept.

---

### 9. Why does every container have PID 1?

**Answer:** Inside its PID namespace, the first process started becomes PID 1. On the host, that same process has a different PID.

---

### 10. Why can two containers both listen on port 80?

**Answer:** Each container has its own network namespace with its own network stack. Port conflicts only occur when publishing to the same host port.

---

### 11. What is a mount namespace?

**Answer:** It gives a container its own view of mounted filesystems, so it only sees the mounts that are explicitly made available.

---

### 12. What is an IPC namespace?

**Answer:** It isolates inter-process communication resources such as shared memory segments, message queues, and semaphores between containers.

---

### 13. How would you investigate a memory-related container crash?

**Answer:** Check `docker inspect` for memory limits, `docker stats` for usage, `dmesg` for OOM messages, and confirm whether Exit Code 137 indicates the process was killed.

---

### 14. Why are Docker images built in layers?

**Answer:** Layering enables caching, reuse of common components, smaller downloads, and faster builds because unchanged layers don't need to be rebuilt or transferred.

---

### 15. Explain Docker internals in one minute.

**Answer:** Docker packages applications into images made of layered, read-only filesystems. When a container starts, the Linux kernel creates isolated namespaces for processes, networking, mounts, and IPC. cgroups enforce CPU and memory limits, while OverlayFS merges image layers with a writable layer. Docker Engine orchestrates these kernel features, allowing many lightweight containers to share the same host kernel efficiently.

---

## Interview Tip

For a mid-level DevOps interview, you should be comfortable answering this question smoothly:

> **"How does Docker isolate containers from each other?"**

A strong answer is:

> "Docker relies on Linux kernel features rather than implementing isolation itself. Namespaces isolate resources such as processes, networking, mounts, and IPC so each container has its own view of the system. cgroups enforce resource limits like CPU and memory. OverlayFS provides a unified layered filesystem with read-only image layers and a writable container layer. Together, these features allow multiple lightweight containers to run securely and efficiently on the same host."
