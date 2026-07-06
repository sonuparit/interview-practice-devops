# 1. Linux Architecture

## 1. What is it?

Linux Architecture is the overall design of the operating system.

Think of Linux as several layers working together.

```
+---------------------------+
| Applications              |
| Nginx, Docker, Python     |
+---------------------------+
| Shell / Libraries         |
| Bash, glibc               |
+---------------------------+
| System Calls              |
+---------------------------+
| Linux Kernel              |
+---------------------------+
| Hardware                  |
| CPU RAM Disk Network      |
+---------------------------+
```

Everything eventually talks to the kernel.

---

## 2. Why is it required?

Without architecture there is no communication between software and hardware.

Example:

```
Python wants to read a file.

↓

Python

↓

glibc

↓

Kernel

↓

SSD

↓

Kernel

↓

Python receives data
```

Applications never touch hardware directly.

The kernel protects the hardware.

---

## 3. How does it help in DevOps?

Almost everything you deploy runs somewhere in this architecture.

Examples:

* Docker containers
* Kubernetes Pods
* Nginx
* PostgreSQL
* Redis

When something becomes slow, you know which layer is responsible.

Example

```
Website slow

Application?

Network?

Kernel?

Disk?

Hardware?
```

Instead of randomly guessing, you isolate the correct layer.

---

## 4. Why recruiters care?

Because production debugging is layer-by-layer.

Imagine interviewer asks

> Website is slow.

Beginner:

> Restart server.

Mid-level:

```
Application healthy?

↓

CPU?

↓

Memory?

↓

Disk I/O?

↓

Kernel logs?

↓

Network?

```

That's structured troubleshooting.

---

## 5. Troubleshooting

Example

```
Website slow
```

Possible layers

Application bug

↓

High CPU

↓

Disk bottleneck

↓

Kernel waiting on I/O

↓

Hardware issue

Architecture tells you where to look first.

---

## 6. What services depend on this?

Everything.

* Docker
* Kubernetes
* Databases
* Web servers
* SSH
* Systemd
* Networking

---

## 7. Analogy

Imagine a city.

```
People

↓

Roads

↓

Traffic Police

↓

Road Infrastructure
```

People are applications.

Roads are system calls.

Traffic police is the kernel.

Road infrastructure is hardware.

---

# 2. Kernel vs User Space

---

## 1. What is it?

Linux separates programs into two worlds.

```
User Space

-----------------

Kernel Space
```

User Space

* Chrome
* Python
* Docker CLI
* Bash
* Nginx

Kernel Space

* CPU scheduler
* Memory manager
* Drivers
* Filesystems
* Network stack

---

## 2. Why required?

Security.

If applications could directly access RAM or disks, one bug could destroy the OS.

Instead

```
Application

↓

Kernel checks permission

↓

Hardware
```

---

## 3. DevOps importance

Containers share one kernel.

This explains why Docker is lightweight.

```
Container A

Container B

Container C

↓

One Linux Kernel
```

Unlike virtual machines.

---

## 4. Recruiters care because

Questions like

Why are containers lighter than VMs?

Answer:

Because containers share the host kernel instead of running separate guest kernels.

---

## 5. Troubleshooting

Example

Application crashes.

Ask

Application problem?

or

Kernel problem?

If kernel logs show

```
OOM Killer
```

Application isn't the root cause.

Kernel killed it.

---

## 6. Affects

* Docker
* Kubernetes
* Memory
* CPU Scheduling
* Drivers
* Networking
* Security

---

## 7. Analogy

Imagine a bank.

Customers

↓

Security Guard

↓

Vault

Customers are User Space.

Vault is Hardware.

Guard is Kernel.

Nobody enters the vault directly.

---

# 3. Shell

---

## 1. What is it?

Shell is the program that lets humans communicate with Linux.

Examples

* Bash
* Zsh
* Fish

```
You

↓

Shell

↓

Kernel

↓

Hardware
```

---

## 2. Why required?

Without shell

No commands.

No automation.

No scripting.

---

## 3. DevOps importance

Every deployment uses shell.

Examples

```
kubectl

terraform

docker

git

aws

systemctl
```

All are executed from the shell.

---

## 4. Recruiters care

Because automation is mostly shell based.

Every CI/CD pipeline executes shell commands.

---

## 5. Troubleshooting

Suppose deployment failed.

Shell history

```
history

echo

env

which

type
```

helps identify environment issues.

---

## 6. Affects

Everything CLI-based.

* Docker
* Terraform
* Git
* Kubernetes
* Jenkins
* GitHub Actions

---

## 7. Analogy

Shell is a receptionist.

You tell the receptionist

"Call the manager."

Receptionist forwards your request.

---

# 4. Distributions

---

## 1. What is it?

A Linux distribution packages the Linux kernel with user-space tools, package managers, default configurations, and software into a complete operating system.

Common examples include Ubuntu, Debian, Rocky Linux, AlmaLinux, and Arch Linux.

```
Linux Kernel

+

GNU Tools

+

Package Manager

+

Utilities

=

Distribution
```

---

## 2. Why required?

The kernel alone isn't enough.

You need

* package manager
* shell
* libraries
* networking tools

---

## 3. DevOps importance

Cloud providers use different distributions.

Examples

* Ubuntu
* Amazon Linux
* Debian
* RHEL-compatible systems

Knowing distributions helps you adapt package management, service names, and defaults quickly.

---

## 4. Recruiters care

Because production isn't always Ubuntu.

You may SSH into multiple Linux distributions on the same day.

---

## 5. Troubleshooting

Package installation failures.

```
apt

yum

dnf

apk
```

Different commands for different distributions.

---

## 6. Affects

* Package management
* System updates
* Repositories
* Services
* Security patches

---

## 7. Analogy

Linux kernel is an engine.

Different distributions are different car manufacturers using that engine.

---

# 5. Filesystem Hierarchy (FHS)

---

## 1. What is it?

FHS defines where files should live.

```
/

├── bin
├── etc
├── var
├── home
├── usr
├── tmp
├── boot
├── dev
├── proc
└── sys
```

---

## 2. Why required?

Imagine every application storing files wherever it wanted.

Administration would become chaos.

---

## 3. DevOps importance

When troubleshooting, you immediately know where to look.

Examples

```
Logs

↓

/var/log

Configs

↓

/etc

User data

↓

/home

Bootloader

↓

/boot
```

---

## 4. Recruiters care

Because Linux administration depends heavily on knowing standard locations.

---

## 5. Troubleshooting

Disk full?

Check

```
/var/log

/tmp

/home
```

Configuration issue?

```
/etc
```

Boot issue?

```
/boot
```

---

## 6. Affects

Every service stores configuration, logs, binaries, or data somewhere within the filesystem hierarchy.

---

## 7. Analogy

Think of a hospital.

Emergency, pharmacy, ICU, radiology all have fixed locations.

FHS gives Linux the same organization.

---

# 6. Boot Process Overview

---

## 1. What is it?

The boot process is the sequence Linux follows from pressing the power button until the login prompt appears.

Typical flow:

```
Power On
↓

BIOS/UEFI
↓

Bootloader
↓

Kernel
↓

init/systemd
↓

Services
↓

Login
```

---

## 2. Why required?

Each stage prepares the next one.

Without this sequence, the operating system cannot start.

---

## 3. DevOps importance

If a production server won't boot after a kernel update or configuration change, understanding this sequence helps you identify where it failed.

---

## 4. Recruiters care

Boot failures are common operational incidents. Engineers who understand the boot chain can recover systems more quickly.

---

## 5. Troubleshooting

Examples:

* Stuck before bootloader → firmware or disk issue.
* Bootloader menu missing → bootloader problem.
* Kernel panic → kernel or root filesystem issue.
* Login never appears → userspace/service initialization issue.

---

## 6. Affects

* Bootloader
* Kernel
* init/systemd
* Mounted filesystems
* Startup services

---

## 7. Analogy

It's like starting an airplane:

```
Battery
↓

Engine start
↓

Flight computers

↓

Hydraulics

↓

Takeoff
```

If any stage fails, the flight doesn't continue.

---

# 7. Runlevels / Targets

---

## 1. What is it?

Older Linux systems used **runlevels**. Modern systems using `systemd` use **targets**. Both define the operating state of the machine.

Examples:

* Rescue mode
* Multi-user mode
* Graphical mode

---

## 2. Why required?

Different situations require different sets of services.

For example, a server usually doesn't need a graphical desktop.

---

## 3. DevOps importance

You'll switch to rescue or emergency targets when recovering broken systems or fixing configuration issues that prevent normal boot.

---

## 4. Recruiters care

Recovery scenarios often require changing boot targets to repair a server without starting unnecessary services.

---

## 5. Troubleshooting

If the graphical environment fails, boot into a text-only target to diagnose logs, reinstall drivers, or repair configuration.

---

## 6. Affects

* Display manager
* Networking
* Login services
* Multi-user services
* Maintenance mode

---

## 7. Analogy

Think of a building operating in different modes:

* Normal business hours
* Maintenance mode
* Emergency mode

Each mode enables only the services it needs.

---

# 8. init vs systemd

---

## 1. What is it?

Both are the first userspace process started by the kernel (PID 1), responsible for starting and managing the rest of the system.

`init` is the older, simpler approach.

`systemd` is the modern service manager used by most Linux distributions.

---

## 2. Why required?

Once the kernel is running, something must start all the required services in the correct order.

---

## 3. DevOps importance

You'll use `systemd` daily to:

* Start and stop services
* Enable services at boot
* Check service status
* View service logs
* Diagnose startup failures

---

## 4. Recruiters care

Many production incidents involve services that won't start or repeatedly crash. Engineers are expected to be comfortable with `systemd`.

---

## 5. Troubleshooting

Typical workflow:

```
systemctl status nginx
↓

journalctl -u nginx
↓

Fix configuration
↓

systemctl restart nginx
```

---

## 6. Affects

Almost every long-running service:

* Web servers
* Databases
* Monitoring agents
* SSH
* Container runtimes

---

## 7. Analogy

Imagine a hotel manager.

The manager ensures housekeeping, security, maintenance, and reception all begin work in the proper order and restarts them if needed.

---

# 9. Login Process

---

## 1. What is it?

The login process authenticates a user, creates a session, loads the user's environment, and starts the shell.

---

## 2. Why required?

It ensures only authorized users gain access and that they receive the correct permissions and environment.

---

## 3. DevOps importance

Authentication problems, SSH access issues, incorrect environments, and permission errors often involve the login process.

---

## 4. Recruiters care

Access management is fundamental to Linux administration and production operations.

---

## 5. Troubleshooting

Examples:

* SSH login denied
* Incorrect home directory
* Wrong shell
* Expired password
* Authentication failures in logs

Understanding the login sequence helps narrow down the cause.

---

## 6. Affects

* SSH
* PAM
* User accounts
* Home directories
* Environment variables
* Shell initialization files

---

## 7. Analogy

It's like entering a secure office building:

1. Show your ID.
2. Security verifies it.
3. You receive access permissions.
4. You enter your assigned workspace.

---

# 10. Virtual Terminals

---

## 1. What is it?

Virtual terminals (also called virtual consoles) are multiple independent text-based login sessions available on the same Linux machine.

You can typically switch between them using `Ctrl + Alt + F1` through `Ctrl + Alt + F6` (the exact keys may vary by distribution).

---

## 2. Why required?

They provide alternative access when the graphical desktop isn't available or when multiple console sessions are needed.

---

## 3. DevOps importance

If a graphical environment freezes but the kernel is still healthy, a virtual terminal often lets you log in and recover the system without rebooting.

---

## 4. Recruiters care

Engineers should know how to recover systems when the GUI is unavailable.

---

## 5. Troubleshooting

Common uses:

* Kill a runaway process.
* Restart a failed display manager.
* Check logs.
* Restart services.
* Investigate resource exhaustion.

---

## 6. Affects

* Local console access
* Login sessions
* Display manager recovery
* Emergency maintenance

---

## 7. Analogy

Imagine a building with multiple emergency entrances.

If the main entrance (the graphical desktop) is blocked, you can still enter through another door (a virtual terminal) and fix the problem from inside.

---

# What a Mid-Level DevOps Engineer Should Master

For interviews and production work, these topics are not just theory. They form a mental model of how Linux works. Once you understand them together, you'll be able to answer questions like:

* **Why won't this server boot?**
* **Why did this service fail to start?**
* **Is the problem in the application, the kernel, or the hardware?**
* **Why was the process killed?**
* **Why can SSH log in but the web application is unavailable?**
* **Where should I look first for logs or configuration?**

These are exactly the kinds of questions that distinguish someone who memorizes commands from someone who can systematically troubleshoot production systems—a key expectation for a mid-level DevOps engineer.
