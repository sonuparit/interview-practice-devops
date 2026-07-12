This is probably **the highest ROI Linux topic** for a Mid-Level DevOps interview.

Companies don't hire DevOps engineers because they know commands. They hire them because **production is down at 2 AM, customers are angry, and they can find the root cause quickly.**

Interviewers usually don't ask:

> What does `journalctl` do?

Instead they ask:

> Nginx is down. CPU is normal. Memory is normal. Port 80 isn't working. What will you check?

They want to know your **thinking process**.

---

# Golden Rule of Production Troubleshooting

Never randomly execute commands.

Always follow a systematic approach.

```
1. Understand the symptom
        ↓
2. Gather evidence
        ↓
3. Narrow possibilities
        ↓
4. Find root cause
        ↓
5. Fix
        ↓
6. Verify
        ↓
7. Prevent recurrence
```

This is exactly how experienced SREs work.

---

# 1. Analyze a production outage methodically

## What is it?

When production goes down, don't panic.

Follow a structured investigation.

---

Imagine:

```
Users
   ↓
Load Balancer
   ↓
Nginx
   ↓
Application
   ↓
Database
```

Website is down.

Where is the problem?

Could be

* DNS
* Load balancer
* Firewall
* Nginx
* Application
* Database
* Disk full
* CPU
* Memory
* SSL certificate
* Network

Instead of guessing...

Investigate.

---

## Production checklist

### Step 1

Is server alive?

```
ping server
```

or

```
ssh server
```

---

### Step 2

Is service running?

```
systemctl status nginx
```

---

### Step 3

Check logs

```
journalctl -u nginx
```

---

### Step 4

Check port

```
ss -tulnp
```

---

### Step 5

Check resources

```
top
free -h
df -h
```

---

### Step 6

Check dependencies

Can app reach

* database?
* redis?
* DNS?
* internet?

---

### Step 7

Fix

---

### Step 8

Verify

```
curl localhost
```

---

### Step 9

Monitor

---

## Interview answer

> I first identify the symptom, verify whether the host is reachable, check service status, review logs, inspect CPU, memory, disk and network usage, validate dependencies, identify the root cause, apply the fix, verify service recovery, and finally determine how to prevent the issue from recurring.

---

# 2. Read logs using journalctl and /var/log

## Why logs matter

Logs are usually the fastest way to identify failures.

---

## journalctl

Shows logs collected by systemd.

```
journalctl
```

Latest logs

```
journalctl -n 100
```

Live logs

```
journalctl -f
```

Specific service

```
journalctl -u nginx
```

Today's logs

```
journalctl --since today
```

Last hour

```
journalctl --since "1 hour ago"
```

Errors only

```
journalctl -p err
```

Boot logs

```
journalctl -b
```

Previous boot

```
journalctl -b -1
```

---

## /var/log

Traditional log files.

Common ones

```
/var/log/messages
```

General logs

```
/var/log/syslog
```

Ubuntu

```
/var/log/auth.log
```

SSH

```
/var/log/kern.log
```

Kernel

```
/var/log/dmesg
```

Boot/kernel messages

Application logs

```
/var/log/nginx/

 /var/log/apache2/

/var/log/mysql/

/var/log/postgresql/
```

---

Useful commands

```
tail -f logfile
```

```
grep ERROR logfile
```

```
less logfile
```

---

# 3. Identify CPU, Memory, Disk or Network Bottlenecks

This is extremely common.

---

## CPU

Tools

```
top
```

```
htop
```

```
ps aux --sort=-%cpu
```

Questions

* CPU 100%?
* Which process?

---

## Memory

```
free -h
```

```
top
```

```
ps aux --sort=-%mem
```

Look for

* OOM
* swap usage
* memory leak

Kernel messages

```
dmesg | grep -i oom
```

---

## Disk

```
df -h
```

Filesystem usage

```
du -sh *
```

Largest folders

```
find / -type f -size +500M
```

---

## Disk I/O

```
iostat
```

```
iotop
```

---

## Network

Connections

```
ss -tulnp
```

Bandwidth

```
iftop
```

or

```
nload
```

---

# 4. Trace why a service won't start

Most common interview question.

Example

```
systemctl start nginx
```

fails.

---

Check status

```
systemctl status nginx
```

Shows

* exit code
* last error

---

Then

```
journalctl -u nginx
```

Possible causes

```
Configuration error

Permission denied

Port already in use

Certificate missing

Disk full

User missing

Environment variable missing
```

---

Verify config

Example

```
nginx -t
```

---

# 5. Diagnose "No space left on device"

Most people think

> Disk is full.

Not always.

There are two possibilities.

---

## Case 1

Filesystem full

```
df -h
```

Shows

```
100%
```

---

## Case 2

Inode exhaustion

```
df -i
```

Shows

```
100%
```

Even if disk has free GB.

Millions of tiny files.

Example

```
cache/

logs/

mail queue
```

---

Find inode consumers

```
find /var -xdev -printf '%h\n' | sort | uniq -c | sort -n
```

---

Delete unnecessary files.

---

# 6. High Load Average vs High CPU

Very important interview topic.

---

## CPU Usage

Means

CPU is actually busy.

```
CPU
█████████
```

---

## Load Average

Means

Processes are waiting.

Not only CPU.

Also

* disk
* I/O
* locks
* uninterruptible sleep

Example

```
Load Average

18

CPU

15%
```

Possible cause

Disk waiting.

Database waiting.

NFS hanging.

---

Check

```
top
```

Look at

```
load average
```

---

Also

```
vmstat
```

```
iostat
```

---

# 7. Why application cannot bind to a port

Example

App wants

```
8080
```

Fails.

---

Find who owns port

```
ss -tulnp
```

or

```
lsof -i :8080
```

or

```
fuser 8080/tcp
```

---

Possible reasons

Port already occupied

Firewall

Permission

SELinux/AppArmor

Wrong IP

---

Kill process

```
kill PID
```

Restart.

---

# 8. Debug DNS resolution problems

Application

```
Cannot resolve database.internal
```

---

Check

```
ping google.com
```

Works?

---

Then

```
dig database.internal
```

or

```
nslookup
```

or

```
host
```

---

Check

```
/etc/resolv.conf
```

Nameservers.

---

Possible problems

Wrong DNS

Firewall

VPN

Split DNS

Missing search domain

---

# 9. Troubleshoot SSH login failures

Cannot SSH.

Check

```
systemctl status ssh
```

---

Logs

```
journalctl -u ssh
```

or

```
tail /var/log/auth.log
```

Possible causes

Wrong password

Wrong key

Permission issue

```
~/.ssh

authorized_keys
```

Firewall

Disk full

User locked

SELinux/AppArmor

Fail2Ban

---

# 10. Filesystem corruption or Disk I/O errors

Symptoms

```
Input/output error
```

```
Read-only filesystem
```

```
Filesystem corruption
```

---

Kernel logs

```
dmesg
```

```
journalctl -k
```

Look for

```
EXT4-fs error

I/O error

Buffer error

nvme timeout

SATA error
```

---

Check filesystem

Unmount

```
fsck
```

Never run on mounted filesystem unless explicitly supported.

---

Check SMART

```
smartctl
```

---

If hardware failing

Replace disk.

---

# 11. Common Tools Used Together

## ps

Process information

```
ps aux
```

---

## top

Live monitoring

```
top
```

---

## ss

Sockets

```
ss -tulnp
```

---

## lsof

Who opened file?

Who owns port?

```
lsof -i :80
```

Deleted files still open

```
lsof | grep deleted
```

---

## fuser

Who is using file?

```
fuser -v file
```

or

```
fuser 8080/tcp
```

---

## strace (Basic)

One of the most powerful Linux debugging tools.

It shows **every system call** a process makes to the kernel.

Example:

```
strace ls
```

You'll see calls like:

```
open()
read()
write()
close()
stat()
execve()
```

This helps you understand exactly where a process is failing.

Common use cases:

Trace a running process:

```bash
strace -p <PID>
```

Save output to a file:

```bash
strace -o trace.log ./myapp
```

Trace file-related system calls only:

```bash
strace -e trace=file ./myapp
```

Example scenario:

```
Application says:

Cannot open config file
```

Run:

```bash
strace -e trace=file ./app
```

Output:

```
open("/etc/app/config.yml", O_RDONLY)
= -1 ENOENT (No such file or directory)
```

Immediately you know the application is looking for a configuration file that doesn't exist.

> **Interview tip:** You don't need to memorize every `strace` option. Interviewers mainly expect you to know that it traces system calls and is useful when logs don't explain why a process is failing.

---

# How Interviewers Test Production Troubleshooting

Rather than asking isolated command questions, they often present a scenario.

### Example 1: Nginx won't start

A strong troubleshooting flow would be:

```text
systemctl status nginx
        ↓
journalctl -u nginx
        ↓
nginx -t
        ↓
ss -tulnp | grep :80
        ↓
df -h
```

Possible root causes:

* Configuration syntax error
* Port 80 already in use
* SSL certificate missing
* Disk full

---

### Example 2: Website is slow

A methodical investigation might be:

```text
top
        ↓
free -h
        ↓
df -h
        ↓
iostat
        ↓
ss -tulnp
        ↓
journalctl
```

Possible findings:

* CPU saturation
* Memory pressure or OOM events
* Disk I/O bottleneck
* Excessive network connections
* Application errors in logs

---

### Example 3: "No space left on device"

Check both storage capacity and inodes:

```text
df -h
        ↓
df -i
        ↓
du -sh /*
        ↓
find /var -type f -size +100M
```

Possible root causes:

* Filesystem full
* Inode exhaustion due to many small files
* Large log files
* Orphaned files still held open by a process (`lsof | grep deleted`)

---

# What Recruiters Expect from a Mid-Level DevOps Engineer

For this topic, they are looking for evidence that you can:

* Follow a structured troubleshooting process instead of guessing.
* Correlate logs, metrics, and system state to identify the root cause.
* Distinguish between CPU, memory, disk, network, and application-layer issues.
* Use core Linux tools (`top`, `ps`, `ss`, `lsof`, `journalctl`, `strace`, `fuser`) together rather than in isolation.
* Verify that a fix has resolved the problem and think about preventing the same incident in the future.

The specific commands matter, but your **diagnostic methodology** is what interviewers value most. A calm, systematic approach to narrowing down the cause of an outage is one of the clearest indicators of production readiness.
