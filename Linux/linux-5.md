This is one of the **most important Linux topics for a Mid-level DevOps Engineer**.

Almost every production issue eventually comes down to **processes**.

Examples:

* Kubernetes Pods run processes.
* Docker containers run processes.
* Jenkins agents are processes.
* Nginx workers are processes.
* PostgreSQL is a process.
* Prometheus, Grafana, ArgoCD, Terraform, Helm... everything ultimately becomes one or more Linux processes.

So if you master Process Management, you understand what Linux is actually doing behind the scenes.

---

# First understand: What is a Process?

A **process** is simply:

> A running instance of a program.

Example

```
Program on disk
/usr/bin/nginx

↓

When executed

PID 2345
nginx process
```

Example

```
python app.py
```

Before execution

```
app.py
```

After execution

```
PID 4132
python process
```

Every running program has

* Process ID (PID)
* Parent PID (PPID)
* Owner
* CPU usage
* Memory usage
* State
* Priority
* Open files
* Network sockets

---

# Process Lifecycle

This is one of the favorite interview questions.

```
New
 ↓
Ready
 ↓
Running
 ↓
Waiting (sleep)
 ↓
Running
 ↓
Terminated
```

Linux scheduler continuously moves processes between these states.

Example

Imagine opening Chrome.

```
Click Chrome

↓

Created

↓

Gets CPU

↓

Loads website

↓

Waiting for network

↓

Receives data

↓

Runs again

↓

Exit
```

---

# Parent & Child Process

Every process is created by another process.

Example

```
systemd
   │
   ├── sshd
   │      │
   │      └── bash
   │              │
   │              └── python script.py
```

Here

```
systemd
```

is parent.

```
sshd
```

is child.

```
bash
```

is child of sshd.

```
python
```

is child of bash.

Check using

```bash
ps -ef --forest
```

or

```bash
pstree
```

---

## Why important?

When parent dies,

Linux decides what happens to child.

This becomes important in production.

---

## Interview

**Q**
How is every Linux process created?

**Answer**

By another process using fork() and exec().

---

# Zombie Process

A zombie process is

> A dead process whose parent hasn't collected its exit status.

Imagine

```
Parent
   │
Child
```

Child finishes.

But parent forgot to clean it.

Linux keeps tiny information.

State becomes

```
Z
```

Zombie.

Check

```bash
ps aux | grep Z
```

---

## Why dangerous?

Thousands of zombies

↓

PID table fills

↓

Cannot create new processes

↓

Production outage

---

# Orphan Process

Parent dies first.

Child still running.

Linux automatically assigns

```
systemd
```

(or historically `init`) as the new parent.

Nothing wrong.

This is normal.

---

# Daemon

Daemon = background service.

Examples

* nginx
* sshd
* systemd
* cron
* docker
* kubelet

Runs continuously.

Starts automatically.

No user interaction.

---

Example

```
systemctl start nginx
```

Now nginx becomes daemon.

---

# Signals

Signals are messages sent to processes.

Think of them like notifications.

Example

```
Stop

Pause

Continue

Reload

Kill
```

---

Common signals

| Signal  | Meaning              |
| ------- | -------------------- |
| SIGTERM | Please exit          |
| SIGKILL | Die immediately      |
| SIGINT  | Ctrl+C               |
| SIGHUP  | Reload configuration |
| SIGSTOP | Pause                |
| SIGCONT | Continue             |

---

Analogy

Manager talking to employee.

```
SIGTERM

"Please finish work and leave."

SIGKILL

"Security! Remove him immediately."

SIGSTOP

"Pause."

SIGCONT

"Continue."

SIGINT

"You pressed Ctrl+C."
```

---

# SIGTERM

Signal number

```
15
```

Graceful shutdown.

Application gets chance to

* save files
* close DB connections
* flush logs
* release locks

Example

```bash
kill PID
```

Actually sends

```
SIGTERM
```

---

Production example

Stopping PostgreSQL.

Need graceful shutdown.

Otherwise

Recovery required.

---

# SIGKILL

Signal number

```
9
```

Cannot be ignored.

Kernel immediately destroys process.

No cleanup.

No save.

No logging.

No closing connections.

Example

```bash
kill -9 PID
```

---

Use only if

SIGTERM failed.

---

Interview question

Why shouldn't you always use kill -9?

Because application cannot clean up resources.

---

# CPU Scheduling

CPU is shared.

Linux scheduler decides

Who gets CPU.

When.

For how long.

Factors

* Priority
* Nice value
* Scheduling policy
* Fairness

---

Example

100 processes.

Only 8 CPU cores.

Scheduler keeps switching.

---

# Process Priority

Priority determines CPU preference.

Higher priority

↓

Gets CPU earlier.

---

Default

Nice

```
0
```

Range

```
-20 (highest priority)

19 (lowest priority)
```

---

# nice

Start process with different priority.

Example

```bash
nice -n 10 python backup.py
```

Lower priority.

Production users unaffected.

---

# renice

Change priority later.

```bash
renice 5 PID
```

---

Production

Backup consuming CPU.

Instead of killing

↓

Reduce priority.

---

# ps

Shows snapshot of running processes.

Example

```bash
ps
```

Common

```bash
ps aux
```

or

```bash
ps -ef
```

Useful columns

```
PID

USER

%CPU

%MEM

STAT

COMMAND
```

---

Production

```
Server slow

↓

ps aux --sort=-%cpu

↓

Found Java process using 700% CPU
```

---

# top

Real-time process monitor.

Shows

* CPU
* RAM
* Swap
* Load
* Running processes

Refreshes continuously.

Run

```bash
top
```

---

Production

```
CPU 100%

↓

top

↓

Found python script consuming CPU.
```

---

# htop

Improved version of top.

Features

* Colors
* Mouse support
* Tree view
* Easier sorting
* Easier killing

Preferred by admins.

Install

```bash
sudo apt install htop
```

---

# pgrep

Search process by name.

Example

```bash
pgrep nginx
```

Returns PID.

---

Useful

Instead of

```bash
ps aux | grep nginx
```

---

# pidof

Returns PID of running program.

```bash
pidof nginx
```

Simple.

Mostly for exact executable names.

---

# kill

Send signal to PID.

Example

```bash
kill 2134
```

Default

SIGTERM.

---

# killall

Kill all processes with same executable name.

```bash
killall nginx
```

Kills every nginx process.

---

# pkill

Kill by matching name or attributes.

Example

```bash
pkill python
```

or

```bash
pkill -u sonu
```

Kill all processes owned by a user.

More flexible than `killall` because it can match patterns, users, terminals, and other attributes.

---

# jobs

Shows background jobs of the **current shell**.

Example

```bash
sleep 100 &
```

Then

```bash
jobs
```

Output

```
[1]+ Running sleep 100
```

This is **not** a system-wide process list. It only tracks jobs started from your current interactive shell.

---

# bg

Resume stopped job in background.

```
Ctrl+Z

↓

bg
```

---

# fg

Bring job back.

```bash
fg
```

Useful when you accidentally backgrounded something.

---

# nohup

Keeps process alive after logout.

Example

```bash
nohup python backup.py &
```

Without nohup

```
Logout

↓

Process dies
```

With nohup

```
Logout

↓

Process continues
```

Output is written to `nohup.out` by default unless redirected.

---

Production

Running a long migration over SSH.

Without nohup

SSH disconnect

↓

Migration stops.

---

# disown

Removes a job from the shell's job table.

Example

```bash
python app.py &
disown
```

Now even if the shell exits, the shell no longer manages that job. Combined with `nohup` (or if the program already ignores SIGHUP), this is useful for letting a process continue independently.

---

# Production Troubleshooting Examples

## Scenario 1

Users complain

Website is slow.

Steps

```bash
top
```

↓

CPU 100%

↓

```bash
ps aux --sort=-%cpu
```

↓

Python process consuming CPU.

↓

```bash
renice 10 PID
```

↓

CPU stabilizes.

Later investigate the code.

---

## Scenario 2

Deployment stuck.

Check

```bash
ps -ef | grep deployment
```

Process waiting.

Send

```bash
kill PID
```

If ignored

```bash
kill -9 PID
```

---

## Scenario 3

Memory issue.

```bash
top
```

↓

Java using 40 GB.

↓

Restart gracefully

```bash
kill PID
```

If hung

```bash
kill -9 PID
```

Then investigate why memory usage grew unexpectedly.

---

## Scenario 4

SSH disconnected during backup.

Instead of

```bash
python backup.py
```

Use

```bash
nohup python backup.py &
```

Backup continues.

---

# Dependencies & Services Affected

Understanding processes helps you manage almost every server component:

| Service    | Why Process Knowledge Matters                              |
| ---------- | ---------------------------------------------------------- |
| Nginx      | Worker processes, graceful reloads, CPU usage              |
| Apache     | Worker lifecycle and signal handling                       |
| Docker     | Every container has a PID 1 process                        |
| Kubernetes | Pods are collections of Linux processes inside containers  |
| PostgreSQL | Graceful shutdown avoids data corruption and long recovery |
| MySQL      | Process health directly affects database availability      |
| Jenkins    | Build agents and executors are processes                   |
| Prometheus | High CPU or memory usage affects monitoring accuracy       |
| Grafana    | Process crashes make dashboards unavailable                |
| SSH        | Each login creates processes such as `sshd` and a shell    |
| systemd    | Starts, monitors, and restarts service processes           |

---

# Simple Analogy

Imagine a large company.

* **Kernel** = CEO deciding who gets resources.
* **Scheduler** = HR assigning employees to work.
* **Process** = An employee doing a task.
* **PID** = Employee ID card.
* **Parent Process** = Manager who hired the employee.
* **Child Process** = Employee working under that manager.
* **Signal** = An instruction from management ("pause", "stop", "continue").
* **Priority (nice)** = VIP level determining how urgently work is scheduled.
* **Daemon** = Security guards working 24×7 even after everyone leaves.
* **Zombie** = An employee who has resigned, but HR hasn't finished the exit paperwork.
* **Orphan** = An employee whose manager left; the CEO (`systemd`) assigns a new manager.

---

# Three Questions You Must Be Able to Answer

### 1. A server suddenly reaches 100% CPU. What commands will you use to identify and fix the problem?

A strong answer should mention tools like `top` or `htop`, `ps` sorted by CPU usage, identifying the PID, deciding whether to `renice`, gracefully stop with `kill` (SIGTERM), or use `kill -9` only if necessary.

---

### 2. What is the difference between SIGTERM and SIGKILL, and when would you use each?

You should explain that **SIGTERM (15)** requests a graceful shutdown so the application can clean up, while **SIGKILL (9)** immediately terminates the process without cleanup. Use SIGKILL only when the process is unresponsive.

---

### 3. What is the difference between a Zombie process, an Orphan process, and a Daemon?

Your answer should clearly distinguish:

* **Zombie:** Finished process waiting for its parent to collect its exit status.
* **Orphan:** Running process whose parent exited; adopted by `systemd`.
* **Daemon:** Long-running background service that provides system functionality (for example, `sshd`, `nginx`, or `docker`).

---

## Interview Tip for Mid-level DevOps Roles

At this level, interviewers usually care less about memorizing command syntax and more about your troubleshooting process. When discussing process management, explain **how you diagnose an issue**:

1. Observe system health (`top`/`htop`).
2. Identify the problematic process (`ps`, `pgrep`, `pidof`).
3. Understand its parent, state, and resource usage.
4. Choose the least disruptive action (`renice`, `SIGTERM`, then `SIGKILL` if required).
5. Verify the service has recovered and investigate the root cause.

That structured approach demonstrates production troubleshooting skills, which is what interviewers for mid-level DevOps positions are looking for.
