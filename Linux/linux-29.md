This is one of the most important sections for a **Mid-level DevOps Engineer**.

Unlike basic Linux commands, these topics explain **how Linux actually works internally**. Kubernetes, Docker, systemd, containers, and cloud infrastructure are all built on these concepts.

If you understand these, you'll be able to debug problems that junior engineers usually cannot.

---

# 1. Kernel

## What is this?

The **Kernel** is the core of Linux.

It is the first major software loaded after boot and runs with the highest privilege (Kernel Mode).

Everything eventually goes through the kernel.

The kernel manages:

* CPU scheduling
* Memory
* Processes
* Filesystems
* Networking
* Security
* Device drivers

Applications never directly access hardware.

Instead:

```
Application
      │
      ▼
Kernel
      │
      ▼
Hardware
```

---

## Why is it required?

Without the kernel:

* programs couldn't access RAM
* disks couldn't be read
* network packets couldn't be sent
* processes couldn't be scheduled

It acts as the operating system's manager.

---

## Why recruiters care?

A DevOps engineer constantly interacts with kernel-managed resources.

Examples:

* Docker uses kernel namespaces
* Kubernetes uses cgroups
* firewalls use kernel networking
* filesystems are kernel modules
* system calls enter the kernel

If the kernel misbehaves, production breaks.

---

## Production example

A Kubernetes node suddenly becomes slow.

CPU usage is only 30%.

Why?

Kernel scheduler is waiting because disk I/O is saturated.

Knowing only "top" won't solve this.

Knowing kernel internals lets you check:

```
iostat
vmstat
pidstat
sar
```

---

## Troubleshooting

Example:

Application freezing.

Instead of restarting immediately:

```
strace PID
```

If every syscall waits on

```
futex()
```

the process is blocked on a lock.

That immediately narrows the investigation.

---

## Analogy

Imagine a hotel.

Guests = applications

Manager = kernel

Electricity, plumbing, security, elevators = hardware

Guests never repair the elevator.

They ask the manager.

---

## Questions

Can applications access hardware directly?

Why do Docker and Kubernetes depend on the Linux kernel?

What happens when a process requests memory?

---

# 2. System Calls (Syscalls)

## What is this?

A syscall is a request from a program to the kernel.

Example:

Python:

```python
open("file.txt")
```

Internally becomes:

```
open()
```

Kernel performs the operation.

---

## Why required?

Applications cannot:

* access disk
* allocate memory
* create processes
* send packets

directly.

Everything goes through syscalls.

---

## Recruiter importance

Many production issues involve syscalls.

Examples:

* slow disk
* blocked filesystem
* network timeout

Tools like

```
strace
perf
bcc
eBPF
```

all observe syscalls.

---

## Production problem

Database suddenly becomes slow.

Run:

```
strace -p PID
```

Output:

```
read(...)
read(...)
read(...)
```

Database is waiting for disk.

---

## Troubleshooting

Useful command:

```
strace ls
```

You'll see hundreds of syscalls.

---

## Analogy

Restaurant.

Customer (application)

Waiter (syscall)

Kitchen (kernel)

Customer never enters kitchen.

---

## Questions

What is a syscall?

Why can't applications read disks directly?

Why is strace useful?

---

# 3. PID Namespace

## What is this?

A namespace gives a process its own isolated view of something.

PID namespace isolates process IDs.

Container:

```
PID 1
```

Host:

```
PID 28472
```

Same process.

Different view.

---

## Why required?

Without PID namespace:

Containers could see every host process.

Huge security issue.

---

## Recruiters care

Docker relies heavily on PID namespaces.

---

## Production

Container exits.

Inside container:

```
PID 1 died.
```

Container stops.

Understanding PID namespace immediately explains why.

---

## Troubleshooting

```
ps aux
```

Host:

```
apache 28191
```

Inside container:

```
apache 1
```

Expected.

---

## Analogy

Two apartment buildings.

Each apartment numbers rooms:

```
Room 1
```

Many buildings have Room 1.

No conflict.

---

## Questions

Why is PID 1 important in Docker?

Can two processes have PID 1?

Why are namespaces needed?

---

# 4. cgroups (Control Groups)

## What is this?

cgroups limit and monitor resource usage.

They control:

* CPU
* RAM
* I/O
* Network
* Process count

---

## Why required?

Without cgroups:

One application could consume:

100% CPU

100% RAM

Crash entire server.

---

## Recruiters care

Docker resource limits use cgroups.

Example:

```
docker run --memory=512m
```

Kernel enforces it.

---

## Production

Memory leak.

Instead of server crashing:

Container reaches

512 MB

OOM Killer terminates it.

Other services survive.

---

## Troubleshooting

```
cat /sys/fs/cgroup/
```

View resource usage.

---

## Analogy

Family budget.

Each child gets

₹500.

No one spends the whole salary.

---

## Questions

How does Docker limit RAM?

Who enforces resource limits?

Difference between namespace and cgroup?

---

# 5. Namespaces

## What is this?

Namespaces isolate system resources.

Types:

* PID
* Network
* Mount
* UTS (hostname)
* IPC
* User
* Time (newer kernels)
* Cgroup namespace

Each container gets its own view.

---

## Why required?

Containers must believe they have their own machine.

---

## Recruiters care

Namespaces are literally the foundation of Docker.

---

## Production

Two containers both listen on

```
Port 80
```

Works because they have different network namespaces.

---

## Troubleshooting

```
lsns
```

Lists namespaces.

---

## Analogy

Virtual reality goggles.

Everyone sees their own world.

---

## Questions

What problem do namespaces solve?

Does namespace limit CPU?

Which namespace isolates networking?

---

# 6. proc filesystem

## What is this?

`/proc` is a **virtual filesystem** created by the kernel. It does not store files on disk. Instead, it exposes live information about the running system and processes.

Examples:

```
/proc/cpuinfo
/proc/meminfo
/proc/uptime
/proc/<PID>/status
/proc/<PID>/fd
```

---

## Why required?

Programs and administrators need a standard way to inspect the kernel and running processes.

---

## Recruiters care

Many Linux tools (`ps`, `top`, `free`, `uptime`) read data from `/proc`. Understanding it helps with deeper debugging.

---

## Production problem

A Java process appears to have open files after logs were deleted.

Check:

```bash
ls -l /proc/<PID>/fd
```

You may find deleted log files still held open, explaining why disk space isn't freed.

---

## Troubleshooting

Useful files:

```bash
cat /proc/meminfo
cat /proc/loadavg
cat /proc/net/tcp
cat /proc/<PID>/status
```

---

## Analogy

Think of `/proc` as the kernel's live dashboard. It's not a filing cabinet—it updates in real time.

---

## Questions

* Why doesn't `/proc` consume disk space like a normal filesystem?
* Why do `ps` and `top` depend on `/proc`?
* What useful information can you find in `/proc/<PID>`?

---

# 7. sysfs

## What is this?

`/sys` (sysfs) is another **virtual filesystem** that exposes the kernel's view of hardware devices, drivers, and kernel objects.

Examples:

```bash
/sys/class/net
/sys/block
/sys/devices
```

---

## Why required?

It provides a structured interface for configuring and inspecting devices and kernel subsystems.

---

## Recruiters care

DevOps engineers often tune kernel parameters, inspect devices, or debug storage and networking issues using sysfs.

---

## Production problem

A network interface isn't negotiating the expected speed. Inspect the relevant entries under `/sys/class/net/<iface>/` to investigate driver and link information.

---

## Troubleshooting

Examples:

```bash
ls /sys/class/net
ls /sys/block
```

---

## Analogy

If `/proc` is the kernel's live dashboard, `/sys` is its equipment control room.

---

## Questions

* What is the difference between `/proc` and `/sys`?
* Is `/sys` stored on disk?
* What kind of information belongs in sysfs?

---

# 8. initramfs

## What is this?

**initramfs** (Initial RAM Filesystem) is a temporary root filesystem loaded into RAM during boot.

It contains enough tools and drivers to find and mount the real root filesystem.

Boot sequence:

```
BIOS/UEFI
      ↓
Bootloader
      ↓
Kernel
      ↓
initramfs
      ↓
Real root filesystem
      ↓
systemd (PID 1)
```

---

## Why required?

The kernel often cannot directly access the real root filesystem until storage drivers or RAID/LVM support are loaded.

---

## Recruiters care

Boot failures frequently involve initramfs. A DevOps engineer may need to repair systems that won't boot after kernel updates.

---

## Production problem

A server fails after a kernel upgrade and drops into an initramfs shell because it cannot find the root filesystem.

---

## Troubleshooting

Useful commands:

```bash
lsinitramfs
update-initramfs
dracut
```

---

## Analogy

Imagine movers unloading the essential tools first so they can unlock and prepare the new house before bringing in everything else.

---

## Questions

* Why can't the kernel always mount the root filesystem directly?
* What happens after initramfs finishes?
* What kinds of drivers are commonly included in initramfs?

---

# 9. ELF binaries

## What is this?

ELF (**Executable and Linkable Format**) is the standard binary format for Linux executables, shared libraries, object files, and core dumps.

Commands:

```bash
file /bin/ls
readelf -h /bin/ls
objdump -x /bin/ls
```

---

## Why required?

The kernel needs a standardized format to load executables into memory.

---

## Recruiters care

Understanding ELF helps diagnose "Exec format error," missing interpreters, architecture mismatches, and binary compatibility problems.

---

## Production problem

An ARM executable is accidentally deployed to an x86_64 server.

Running it produces:

```
Exec format error
```

---

## Troubleshooting

```bash
file app
readelf -h app
```

These commands reveal the binary's architecture and format.

---

## Analogy

Think of ELF as the blueprint that tells the kernel how to load a program into memory.

---

## Questions

* What does ELF stand for?
* Why does Linux use ELF?
* Which command shows an executable's architecture?

---

# 10. Dynamic linking

## What is this?

Dynamic linking means a program loads required libraries **at runtime** instead of embedding them into the executable.

Example:

```
Program
   ↓
libc.so
libpthread.so
libssl.so
```

---

## Why required?

It reduces executable size, saves memory, and allows multiple programs to share the same library code.

---

## Recruiters care

Many deployment failures are caused by missing or incompatible shared libraries.

---

## Production problem

Application fails with:

```
error while loading shared libraries:
libssl.so: cannot open shared object file
```

---

## Troubleshooting

```bash
ldd app
```

Shows all dynamically linked libraries and whether any are missing.

---

## Analogy

Instead of every employee owning a printer, everyone shares a central printer.

---

## Questions

* What is dynamic linking?
* Why is it preferred over bundling every library?
* Which command shows an executable's library dependencies?

---

# 11. Shared libraries

## What is this?

Shared libraries are reusable binary files (`.so` files) loaded by many programs.

Examples:

```
libc.so
libm.so
libssl.so
```

---

## Why required?

They avoid duplicating common code in every executable, reducing disk and memory usage.

---

## Recruiters care

Library version mismatches are a common cause of production outages after upgrades.

---

## Production problem

A package upgrade replaces `libssl`, and an older application expects a previous version. The application fails to start until compatibility is restored.

---

## Troubleshooting

Useful commands:

```bash
ldd app
ldconfig -p
```

---

## Analogy

Instead of every apartment having its own power plant, the whole building uses one shared electrical grid.

---

## Questions

* What is a shared library?
* Why are shared libraries better than duplicating code?
* How do you identify missing shared libraries?

# Most Important Interview Questions (Mid-Level DevOps)

These are questions you should be able to answer confidently without memorizing scripts.

1. What is the Linux kernel, and what responsibilities does it have?
2. What is a system call, and why can't user-space applications access hardware directly?
3. Explain the difference between **user space** and **kernel space**.
4. What is the difference between **namespaces** and **cgroups**?
5. Why are namespaces essential for containers?
6. What happens when PID 1 exits inside a Docker container?
7. How does Docker enforce CPU and memory limits?
8. What information is available in the `/proc` filesystem, and how is it different from `/sys`?
9. What is initramfs, and why is it needed during boot?
10. What is the ELF format, and how does the kernel use it?
11. What is dynamic linking, and how is it different from static linking?
12. What are shared libraries, and how can a missing shared library prevent an application from starting?
13. An application reports `Exec format error`. What are the likely causes, and how would you troubleshoot it?
14. A container is repeatedly being killed with exit code **137**. What does that indicate, and how would you investigate cgroup memory limits and the OOM killer?
15. A production application won't start because of `error while loading shared libraries`. Which tools (`ldd`, `ldconfig`, `file`, package manager) would you use to identify and resolve the problem?

These are excellent mid-level DevOps interview questions. They test whether you understand **how Linux actually works internally**, not just commands.

---

# 1. What is the Linux kernel, and what responsibilities does it have?

## Short Interview Answer

The Linux kernel is the core component of the operating system. It sits between hardware and user applications, managing CPU, memory, devices, filesystems, networking, and security. Applications never communicate with hardware directly—they request services from the kernel through system calls.

---

## Detailed Explanation

Think of Linux as three layers.

```
Applications
Chrome
Python
Nginx
Docker

------------------

Linux Kernel

------------------

Hardware
CPU
RAM
Disk
NIC
USB
GPU
```

Everything goes through the kernel.

Without the kernel:

* programs cannot read files
* programs cannot allocate memory
* programs cannot use CPU
* programs cannot send packets
* programs cannot access disks

The kernel acts like a manager that decides:

* Who gets CPU?
* Which process gets memory?
* Can this process read this file?
* Can this process access this device?
* Which packets should be sent?

---

## Responsibilities

### 1. Process Management

Creates processes

Schedules them

Terminates them

Context switching

```
fork()
exec()
kill()
wait()
```

---

### 2. Memory Management

Allocates RAM

Virtual Memory

Paging

Swapping

OOM Killer

```
malloc()

↓

Kernel allocates pages
```

---

### 3. Filesystem Management

Handles

```
open()
read()
write()
close()
```

Works with

* ext4
* xfs
* btrfs
* tmpfs
* procfs

---

### 4. Device Drivers

Drivers are part of the kernel.

Example

```
USB
↓

Kernel Driver

↓

Application
```

Application never talks to USB controller.

---

### 5. Networking

TCP/IP stack

Routing

Firewall

Sockets

```
send()

↓

Kernel

↓

NIC
```

---

### 6. Security

Permissions

Capabilities

SELinux

AppArmor

Namespaces

cgroups

---

### 7. Interrupt Handling

When keyboard pressed

```
Hardware Interrupt

↓

Kernel

↓

Application notified
```

---

## Why recruiters ask this

Because Docker, Kubernetes, systemd, networking—all depend on kernel features.

---

# 2. What is a system call, and why can't user-space applications access hardware directly?

## Short Interview Answer

A system call is the controlled interface through which a user-space application requests services from the Linux kernel. Applications cannot access hardware directly because doing so would compromise security, stability, and resource isolation.

---

## Detailed Explanation

Example:

```
printf("Hello")
```

This stays inside user space.

But

```
open("file.txt")
```

needs disk access.

The application cannot access SSD directly.

Instead

```
Application

↓

System Call

↓

Kernel

↓

Disk
```

---

## Example

```
fd = open("abc.txt")
```

becomes

```
sys_open()

↓

Kernel checks

Does file exist?

Permission?

Filesystem?

↓

Returns file descriptor
```

---

## Common System Calls

```
open()

read()

write()

close()

fork()

execve()

clone()

mmap()

socket()

bind()

connect()
```

---

## Why can't applications access hardware?

Imagine every application could control RAM.

Chrome could overwrite MySQL memory.

Or

```
Application A

writes

Application B memory
```

Entire OS crashes.

Another example

```
Application formats SSD accidentally.
```

Kernel prevents this.

---

Kernel provides

* isolation
* permission checking
* scheduling
* synchronization

---

# 3. Explain the difference between user space and kernel space.

## User Space

Applications run here.

Examples

```
Chrome

Docker

Python

Nginx

Java

bash
```

Cannot directly

* access disk
* access RAM pages
* modify CPU scheduler

---

## Kernel Space

Kernel runs here.

Has full privileges.

Can

* access hardware
* change page tables
* schedule CPU
* allocate RAM
* access devices

---

Diagram

```
User Space

Chrome

↓

System Call

↓

Kernel Space

↓

Hardware
```

---

## Protection

CPU provides privilege levels.

```
Ring 0

Kernel

Ring 3

Applications
```

Applications cannot jump into Ring 0.

Only through system calls.

---

# 4. What is the difference between namespaces and cgroups?

This is asked constantly in Docker interviews.

---

## Namespace

Namespaces isolate resources.

They answer

> "What can this process see?"

Examples

PID namespace

```
Container

PID 1

PID 2

PID 3
```

Host

```
PID 4000

PID 4001
```

Container cannot see host processes.

---

Other namespaces

Network

Mount

IPC

UTS

User

Cgroup namespace

---

## cgroups

cgroups control resource usage.

They answer

> "How much resources can this process use?"

Example

```
CPU

2 cores

Memory

512 MB

IO

100 MB/s
```

---

Interview sentence

Namespaces provide isolation.

cgroups provide resource control.

---

# 5. Why are namespaces essential for containers?

Without namespaces every container would see

```
All host processes

All interfaces

All mounts

Hostname

IPC
```

Containers would not be isolated.

Namespace examples

PID namespace

Container

```
PID 1 nginx
```

Host

```
PID 3421 nginx
```

Same process.

Different namespace.

---

Network namespace

Container gets

```
eth0

lo
```

Host has

```
eth0

docker0

veth

wifi
```

Container cannot see host interfaces.

---

Mount namespace

Container sees

```
/

bin

usr

etc
```

Different root filesystem.

---

Without namespaces

Docker wouldn't exist.

---

# 6. What happens when PID 1 exits inside a Docker container?

Container immediately stops.

Docker considers PID 1 the main process.

Example

```
docker run ubuntu
```

Runs

```
bash
```

bash becomes PID 1.

If bash exits

Container exits.

---

Why?

Docker tracks the lifecycle of PID 1.

No main process

↓

Container finished

---

PID 1 has another responsibility.

It must reap zombie processes.

Without init

Zombie accumulation may occur.

Many production images use

```
tini

dumb-init
```

to properly handle signals and reap child processes.

---

# 7. How does Docker enforce CPU and memory limits?

Docker itself doesn't.

Linux cgroups do.

Example

```
docker run

--memory=512m

--cpus=2
```

Docker writes these limits into the container's cgroup.

Kernel continuously enforces them.

---

Memory

If process exceeds

```
512 MB
```

Kernel

```
OOM Killer

↓

Kills process
```

---

CPU

```
2 CPUs
```

Kernel scheduler limits CPU time.

---

Check

```
docker inspect

cat /sys/fs/cgroup/
```

---

# 8. What information is available in the /proc filesystem, and how is it different from /sys?

## /proc

Provides process and kernel runtime information.

Examples

```
/proc/cpuinfo

/proc/meminfo

/proc/uptime

/proc/loadavg

/proc/PID/status

/proc/PID/maps

/proc/PID/fd
```

Example

```
cat /proc/meminfo
```

Shows RAM usage.

---

## /sys

Represents kernel objects and devices.

Examples

```
/sys/block

/sys/class

/sys/devices

/sys/fs/cgroup
```

Often writable.

Can configure kernel behavior.

Example

```
echo performance >

/sys/devices/system/cpu/...
```

---

Difference

| /proc                | /sys                                     |
| -------------------- | ---------------------------------------- |
| Process information  | Device and kernel objects                |
| Runtime statistics   | Kernel configuration                     |
| Mostly virtual files | Virtual representation of kernel objects |

---

# 9. What is initramfs, and why is it needed during boot?

Boot sequence

```
BIOS

↓

Bootloader

↓

Kernel

↓

initramfs

↓

Real Root Filesystem

↓

systemd
```

Kernel initially cannot access SSD because drivers are not yet loaded.

initramfs contains

* temporary root filesystem
* storage drivers
* filesystem modules
* LVM
* RAID
* encryption support

Kernel loads initramfs.

It loads required drivers.

Then mounts real root filesystem.

Finally switches root.

Without initramfs

Kernel may panic because it cannot find the root filesystem.

---

# 10. What is the ELF format, and how does the kernel use it?

ELF

Executable and Linkable Format

Linux executable files use ELF.

Example

```
ls

bash

python

nginx
```

Check

```
file /bin/bash
```

Output

```
ELF 64-bit executable
```

---

Kernel workflow

```
execve()

↓

Kernel reads ELF header

↓

Loads executable sections

↓

Loads shared libraries

↓

Starts process
```

ELF contains

* machine architecture
* entry point
* code
* data
* symbols
* library information

---

# 11. What is dynamic linking, and how is it different from static linking?

## Dynamic Linking

Executable contains references.

Libraries loaded at runtime.

Example

```
libc.so

libpthread.so

libssl.so
```

Executable is smaller.

Libraries shared by many programs.

---

## Static Linking

Library code copied into executable.

Produces large binaries.

No external libraries needed.

---

Comparison

| Static                | Dynamic                            |
| --------------------- | ---------------------------------- |
| Larger executable     | Smaller executable                 |
| Self-contained        | Requires libraries                 |
| No runtime dependency | Runtime dependency                 |
| More disk usage       | Shared libraries reduce disk usage |

---

# 12. What are shared libraries, and how can a missing shared library prevent an application from starting?

Shared libraries

```
libc.so

libssl.so

libcrypto.so
```

Multiple applications use same library.

Instead of copying code.

---

Application startup

```
Application

↓

Dynamic Loader

↓

Find libraries

↓

Load them

↓

Execute program
```

If library missing

```
error while loading shared libraries

libssl.so.3

No such file
```

Application never starts.

Because loader cannot resolve dependencies.

---

Useful commands

```
ldd binary

ldconfig -p

find /usr/lib

file binary
```

---

# 13. An application reports `Exec format error`. What are the likely causes, and how would you troubleshoot it?

`Exec format error` (usually `ENOEXEC`) means the kernel couldn't execute the file because it isn't in a valid executable format for the current system.

### Common causes

1. **Wrong CPU architecture**

   * ARM binary on x86_64
   * x86 binary on ARM

2. **Corrupted executable**

   * Incomplete download
   * Damaged file

3. **Script without a valid shebang**

   ```bash
   #!/bin/bash
   ```

   If the interpreter isn't specified (or is invalid), executing the script directly can fail.

4. **Unsupported binary format**

   * File isn't an ELF executable or recognized script.

5. **Missing execute permission** (strictly speaking this usually gives `Permission denied`, not `Exec format error`, but it's still worth checking during troubleshooting.)

### Troubleshooting steps

```bash
file myapp
```

Verify the architecture and file type.

```bash
uname -m
```

Check the system architecture.

```bash
chmod +x myapp
```

Ensure it's executable.

```bash
head -n1 script.sh
```

Verify the shebang for scripts.

```bash
readelf -h myapp
```

Inspect the ELF header.

```bash
strace ./myapp
```

See exactly where the execution fails.

---

# 14. A container is repeatedly being killed with exit code 137. What does that indicate, and how would you investigate cgroup memory limits and the OOM killer?

## Meaning of exit code 137

```
137 = 128 + 9
```

Signal **9** is `SIGKILL`.

In containers, exit code **137** most commonly means the process was killed because it exceeded its memory limit and the kernel's OOM (Out Of Memory) killer terminated it. It can also happen if someone explicitly sends `SIGKILL`, so you should confirm the cause.

### Investigation steps

**1. Check the container state**

```bash
docker inspect <container>
```

Look for:

```text
OOMKilled: true
```

**2. Check container logs**

```bash
docker logs <container>
```

**3. Check memory limits**

Docker:

```bash
docker inspect <container>
```

or

```bash
docker stats
```

Linux cgroup v2:

```bash
cat /sys/fs/cgroup/memory.max
cat /sys/fs/cgroup/memory.current
```

(For cgroup v1, inspect the corresponding `memory.limit_in_bytes` and usage files.)

**4. Check kernel logs for OOM events**

```bash
dmesg | grep -i oom
```

or

```bash
journalctl -k | grep -i oom
```

### How to fix it

* Increase the container memory limit.
* Reduce the application's memory usage or fix memory leaks.
* Tune JVM/Node.js/Python memory settings if applicable.
* Add more RAM to the host if the host itself is under memory pressure.

---

# 15. A production application won't start because of "error while loading shared libraries". Which tools would you use to identify and resolve the problem?

This is a classic Linux troubleshooting interview question.

### Step 1: Check the missing dependency

```bash
ldd myapp
```

Example:

```text
libssl.so.3 => not found
```

### Step 2: Verify the binary type

```bash
file myapp
```

Confirm:

* ELF executable
* Correct architecture (x86_64, ARM64, etc.)

### Step 3: Check the dynamic linker cache

```bash
ldconfig -p | grep libssl
```

This tells you whether the library is installed and whether the dynamic linker knows about it.

### Step 4: Install the required package

Examples:

Ubuntu/Debian:

```bash
sudo apt install libssl3
```

RHEL/CentOS:

```bash
sudo dnf install openssl-libs
```

### Step 5: Refresh the linker cache (if you've manually installed libraries)

```bash
sudo ldconfig
```

### Step 6: Re-run dependency checks

```bash
ldd myapp
```

Ensure there are no remaining `not found` entries.

---

# Production Interview Summary

| Question                       | Key takeaway                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Kernel                         | Manages hardware, processes, memory, networking, filesystems, and security.                                   |
| System Calls                   | Safe interface between applications and the kernel.                                                           |
| User vs Kernel Space           | Applications run in user space; the kernel runs with full privileges in kernel space.                         |
| Namespaces                     | Provide isolation ("what a process can see").                                                                 |
| cgroups                        | Enforce resource limits ("how much a process can use").                                                       |
| PID 1                          | Main process in a container; when it exits, the container stops.                                              |
| Docker Limits                  | Implemented by Linux cgroups, not by Docker itself.                                                           |
| `/proc` vs `/sys`              | `/proc` exposes process/runtime information; `/sys` exposes kernel objects and device configuration.          |
| initramfs                      | Temporary root filesystem that loads drivers needed to mount the real root filesystem.                        |
| ELF                            | Standard executable format parsed by the kernel during `execve()`.                                            |
| Dynamic Linking                | Loads shared libraries at runtime instead of embedding them into the executable.                              |
| Shared Libraries               | Missing libraries prevent the dynamic loader from starting the application.                                   |
| Exec format error              | Usually wrong architecture, invalid executable format, corrupted binary, or bad script interpreter.           |
| Exit Code 137                  | `SIGKILL`, most often due to an OOM kill in a memory-constrained container.                                   |
| Shared Library Troubleshooting | Use `ldd`, `file`, `ldconfig`, package manager, and kernel logs to identify and resolve missing dependencies. |
