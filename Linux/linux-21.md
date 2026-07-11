This is an excellent topic for a **Mid-level DevOps Engineer** because almost every production environment needs scheduled automation.

Think of it this way:

> **Bash/Python scripts tell Linux *what* to do.**
>
> **cron, at, and systemd timers tell Linux *when* to do it.**

Without scheduling, every maintenance task would require a human.

---

# 1. cron

## What is this?

`cron` is a background daemon that executes commands at scheduled times.

It continuously checks its schedule every minute and runs jobs whose time matches.

Examples:

* Backup database every night
* Rotate logs
* Delete temporary files
* Send monitoring reports
* Restart services

---

## Why is it required?

Servers need repetitive maintenance.

Instead of hiring someone to login every night,

Linux performs these jobs automatically.

Example

Without cron:

```
11 PM

Engineer:
Login
Run backup.sh
Logout
```

With cron

```
11 PM

cron
 ↓
backup.sh
```

---

## Mid-Level DevOps Importance

You'll automate things like:

* backups
* log cleanup
* certificate renewal
* cache cleanup
* health checks
* database optimization
* disk monitoring
* sending alerts

A DevOps engineer who doesn't know cron ends up doing manual work.

---

## Why recruiters care

They expect you to automate operations.

Interviewers want someone who thinks

> "Can this be automated?"

instead of

> "I'll do it manually."

Automation reduces:

* human error
* operational cost
* downtime

---

## Production problem it solves

Example:

Logs reach 100 GB every week.

Without automation:

```
Disk becomes full
↓

Application crashes
↓

Customers affected
```

With cron:

```
Daily

find /var/log/app -mtime +7 -delete
```

Disk never becomes full.

---

## Troubleshooting

Suppose backup never runs.

Check:

```
systemctl status cron
```

or

```
systemctl status crond
```

Check logs

```
journalctl -u cron
```

or

```
journalctl -u crond
```

Check crontab

```
crontab -l
```

Check permissions

Check script executable

```
chmod +x backup.sh
```

Use absolute paths

Wrong:

```
python backup.py
```

Correct

```
/usr/bin/python3 /opt/scripts/backup.py
```

Cron has a minimal environment.

---

## Services/Dependencies

Depends on

* cron daemon
* system clock
* filesystem
* executable permissions
* PATH
* shell

If cron daemon stops

Nothing executes.

---

## Analogy

Imagine a security guard.

Every hour

He checks every room.

If one room says

"Clean me at 10 PM"

He calls housekeeping.

Cron is that guard.

---

## Test Yourself

### Q1

Why should cron jobs use absolute paths?

Because cron has a minimal PATH.

---

### Q2

How often does cron check schedules?

Every minute.

---

### Q3

Where are user cron jobs stored?

```
crontab -e
```

System cron:

```
/etc/crontab

/etc/cron.d/

/etc/cron.daily/

/etc/cron.hourly/
```

---

# Cron Syntax

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of Week
│ │ │ └──── Month
│ │ └────── Day
│ └──────── Hour
└────────── Minute
```

Example

```
0 2 * * * backup.sh
```

2 AM every day.

---

# crontab

## What is this?

`crontab` is the configuration file that cron reads.

Cron = worker

Crontab = schedule

---

## Why required?

Need a place to store schedules.

```
0 1 * * * cleanup.sh
```

belongs inside crontab.

---

## Mid-Level DevOps

You'll manage schedules for

* applications
* monitoring
* maintenance
* automation

---

## Recruiter expectation

They expect you to know

```
crontab -e
crontab -l
crontab -r
```

---

## Production use

Different users need different schedules.

Example

postgres user

```
0 1 * * * pg_dump
```

Application user

```
*/5 * * * * healthcheck.sh
```

---

## Troubleshooting

Check

```
crontab -l
```

Check ownership.

Running under wrong user causes failures.

---

## Dependencies

Depends on

* cron daemon
* user permissions

---

## Analogy

Cron = Manager

Crontab = Employee timetable

---

## Test Yourself

1.

Difference between cron and crontab?

Cron executes.

Crontab stores schedules.

2.

Command to edit?

```
crontab -e
```

3.

Command to list?

```
crontab -l
```

---

# at

## What is this?

Runs a job **once** at a specified future time.

Unlike cron,

it is not repetitive.

Example

```
at 11:30 PM
```

Run cleanup once.

---

## Why required?

Temporary future tasks.

Examples

Restart server after migration.

Delete temporary files tomorrow.

Shutdown VM after testing.

---

## Mid-Level DevOps

Useful during

* migrations
* maintenance
* rollback
* temporary fixes

---

## Recruiter expectation

Shows you know

Recurring task?

→ cron

One-time task?

→ at

---

## Production problem

Example

Database migration.

Need restart after 2 hours.

```
echo "systemctl restart mysql" | at now + 2 hours
```

No engineer needs to stay awake.

---

## Troubleshooting

Check queue

```
atq
```

Delete

```
atrm jobid
```

Check

```
systemctl status atd
```

---

## Dependencies

Needs

```
atd
```

daemon.

Without it,

jobs never execute.

---

## Analogy

Cron

Morning alarm every day.

At

Doctor appointment tomorrow.

---

## Test Yourself

1.

Recurring?

No.

2.

Daemon?

atd

3.

Queue command?

```
atq
```

---

# systemd timers

This is increasingly preferred in modern Linux distributions.

---

## What is this?

systemd timers schedule execution of **systemd services**.

Instead of running commands directly,

timer activates a service unit.

Example

```
backup.timer

↓

backup.service

↓

backup.sh
```

---

## Why required?

Better integration with systemd.

Supports

* logging
* dependencies
* missed execution
* accuracy
* boot ordering

---

## Mid-Level DevOps

Modern production Linux often prefers timers over cron.

Especially

RedHat

Ubuntu

Amazon Linux

---

## Recruiters care

Because enterprise Linux uses systemd heavily.

Timers are considered more modern than cron.

---

## Production problem solved

Server rebooted at 2 AM.

Cron job scheduled at 2 AM.

Missed forever.

Systemd timer:

```
Persistent=true
```

After reboot

Runs immediately.

Huge advantage.

---

## Troubleshooting

List timers

```
systemctl list-timers
```

Status

```
systemctl status backup.timer
```

Logs

```
journalctl -u backup.service
```

---

## Dependencies

Needs

* systemd
* service unit
* timer unit

---

## Analogy

Cron

Alarm clock.

Systemd timer

Smartphone reminder.

If phone was off,

it reminds you after turning it on.

---

## Test Yourself

1.

Difference from cron?

Runs services instead of commands and integrates with systemd.

2.

Missed jobs?

Persistent timers can execute missed jobs after reboot.

3.

Logs?

Available through `journalctl` because timers invoke systemd services.

---

# cron vs at vs systemd timers

| Feature                               | cron                  | at           | systemd timer                              |
| ------------------------------------- | --------------------- | ------------ | ------------------------------------------ |
| Recurring                             | ✅                     | ❌            | ✅                                          |
| One-time                              | ❌                     | ✅            | ✅ (with appropriate timer configuration)   |
| Integrated with systemd               | ❌                     | ❌            | ✅                                          |
| Logs in journalctl                    | Limited (daemon logs) | Limited      | ✅ Excellent                                |
| Handles missed execution after reboot | ❌                     | ❌            | ✅ (`Persistent=true`)                      |
| Service dependencies                  | ❌                     | ❌            | ✅ (`After=`, `Requires=` in service units) |
| Best for                              | Simple recurring jobs | One-off jobs | Production services                        |

---

# Production Example

Suppose every night:

```
Application

↓

Database backup

↓

Compress

↓

Upload to S3

↓

Delete local backup older than 30 days

↓

Send Slack notification
```

Scheduling options:

* **cron:** Simple and widely used for this workflow.
* **systemd timer:** Better if you want structured logging, dependency management, restart policies, and recovery after missed executions.
* **at:** Not suitable because the task repeats every night.

---

# 15 Mid-Level DevOps Interview Questions

### 1. What is the difference between cron and crontab?

**Answer:** `cron` is the daemon that executes scheduled jobs. `crontab` is the schedule file that defines what jobs to run and when.

---

### 2. Why do cron jobs often fail even though the script works manually?

**Answer:** Cron runs with a minimal environment. Common causes are missing environment variables, incorrect `PATH`, relative paths, missing execute permissions, or the wrong working directory.

---

### 3. Why should you use absolute paths in cron jobs?

**Answer:** Because cron does not inherit your interactive shell's `PATH`. Using `/usr/bin/python3` instead of `python3` avoids command-not-found errors.

---

### 4. How do you view a user's cron jobs?

**Answer:**

```bash
crontab -l
```

---

### 5. Where are system-wide cron jobs stored?

**Answer:**

* `/etc/crontab`
* `/etc/cron.d/`
* `/etc/cron.hourly/`
* `/etc/cron.daily/`
* `/etc/cron.weekly/`
* `/etc/cron.monthly/`

---

### 6. When would you choose `at` instead of `cron`?

**Answer:** Use `at` for a one-time future task, such as restarting a service after a planned migration or deleting temporary files the next day.

---

### 7. How do you check whether the `at` daemon is running?

**Answer:**

```bash
systemctl status atd
```

---

### 8. What advantages do systemd timers have over cron?

**Answer:** They integrate with systemd, provide centralized logging through `journalctl`, support service dependencies, allow missed jobs to run after reboot with `Persistent=true`, and offer better control over execution.

---

### 9. How do you list active systemd timers?

**Answer:**

```bash
systemctl list-timers
```

---

### 10. How do you troubleshoot a cron job that never executed?

**Answer:** Check:

* The cron daemon (`systemctl status cron` or `crond`)
* The crontab entry (`crontab -l`)
* Script permissions (`chmod +x`)
* Absolute paths
* Relevant logs (`journalctl -u cron` or `/var/log/cron` depending on the distribution)

---

### 11. A server was down during a scheduled backup. Which scheduler can automatically run the missed backup after reboot?

**Answer:** A **systemd timer** configured with `Persistent=true`.

---

### 12. Why is running scheduled jobs as the correct user important?

**Answer:** The user determines file permissions, environment variables, and access to resources. Running under the wrong account can cause permission failures or access denied errors.

---

### 13. How can you verify whether an `at` job is waiting to run?

**Answer:**

```bash
atq
```

---

### 14. Would you choose cron or a systemd timer for a production backup service? Why?

**Answer:** For simple recurring backups, cron is sufficient. For production systems requiring reliable logging, dependency management, restart behavior, and missed-job recovery, a systemd timer is generally the better choice.

---

### 15. What happens if the cron daemon is stopped?

**Answer:** Scheduled cron jobs will not execute until the daemon is started again. Existing schedules remain configured, but no new jobs are triggered while the daemon is inactive.

---

## What interviewers are really testing

For a **Mid-level DevOps Engineer**, interviewers are usually less interested in whether you remember every cron expression and more interested in whether you can **design, automate, and troubleshoot scheduled operations**. They want to know that you can:

* Choose the right scheduler (`cron`, `at`, or `systemd timer`) for the use case.
* Diagnose why a scheduled task didn't run.
* Write scripts that work in non-interactive environments.
* Understand permissions, environment variables, and service dependencies.
* Build reliable automation for production maintenance tasks such as backups, certificate renewals, log rotation, health checks, and cleanup jobs.
