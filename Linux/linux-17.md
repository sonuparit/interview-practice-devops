This is an excellent topic for a Mid-level DevOps interview because **logs are usually the first place engineers look when production breaks**.

A surprising number of candidates know Kubernetes, Terraform, and AWS well, but when the interviewer asks:

> "The application stopped working. Where would you start?"

many don't know how Linux logging actually works.

---

# Logging Architecture in Linux

Before learning the commands, understand the flow.

```
Application
     │
     │ writes log
     ▼
systemd-journald
     │
     ├── Stores binary logs
     │
     └─────────────► rsyslog (optional)
                         │
                         ▼
                 /var/log/*.log
                         │
                         ▼
               Log rotation / SIEM / ELK / Loki
```

There are **two logging systems** in modern Linux.

1. **systemd-journald**

   * Modern
   * Binary logs
   * Accessed using `journalctl`

2. **rsyslog**

   * Traditional
   * Plain text logs
   * Stores logs inside `/var/log`

Most modern Linux servers use **both together**.

---

# 1. journalctl

## 1. What is this?

`journalctl` is the command used to read logs collected by **systemd-journald**.

Think of it as:

> "Database viewer for Linux system logs."

Unlike text log files, journald stores logs in a binary format.

Examples:

```
journalctl
```

See everything.

```
journalctl -u nginx
```

Only nginx.

```
journalctl -u docker
```

Docker logs.

```
journalctl -b
```

Current boot.

```
journalctl -xe
```

Recent important errors.

---

## 2. Why is it required?

Every service generates logs.

Examples

```
SSH login

Docker

Kubernetes kubelet

systemd

NetworkManager

cron

kernel

nginx
```

Instead of every service inventing its own log storage,

systemd centralizes everything.

---

## 3. How does this help Mid-level DevOps?

Suppose:

```
nginx won't start
```

Instead of guessing,

```
journalctl -u nginx
```

Output

```
Address already in use
```

Immediately you know port 80 is occupied.

Without logs,

you may waste 30 minutes.

---

## 4. Why recruiters care?

Because production debugging starts with logs.

Anyone can restart a service.

A DevOps engineer should answer:

> "Why did it fail?"

---

## 5. Production example

EC2 suddenly becomes unreachable.

You SSH after reboot.

Run

```
journalctl -b -1
```

Shows previous boot logs.

Output

```
Kernel panic
Out of memory
```

Problem solved.

---

## 6. Troubleshooting example

Docker service won't start.

```
systemctl status docker
```

Shows

```
Failed.
```

Need details.

```
journalctl -u docker
```

Output

```
daemon.json syntax error
```

You fix JSON.

Docker starts.

---

## 7. Dependencies

journalctl depends on

```
systemd-journald
```

If journald stops,

logs stop being collected.

Many services send logs directly to journald.

---

## 8. Analogy

Imagine a hospital.

Every doctor writes patient notes.

Instead of separate notebooks,

everything goes into one central digital database.

journalctl is the computer used to read that database.

---

## 9. Three questions

Can you answer these?

1.

Difference between

```
journalctl

cat /var/log/messages
```

2.

How do you view logs from previous boot?

3.

How do you view logs of one service?

---

# 2. syslog

## 1. What is this?

Syslog is a **logging standard**, not a command.

It defines

* log format
* severity
* facility
* transport

Think of it as

> "The language Linux uses for logging."

---

Example log

```
Jul 12 09:12:15 nginx:
Failed to bind port 80
```

Different programs can all produce logs using the same format.

---

## 2. Why required?

Imagine every application used different log formats.

Impossible to automate.

Syslog standardizes logging.

---

## 3. DevOps benefit

When learning ELK,

Loki,

Splunk,

Graylog,

CloudWatch,

all understand syslog.

---

## 4. Why recruiters care?

Because enterprise infrastructure depends on standardized logging.

Firewalls

Switches

Routers

Load balancers

Linux

Databases

All can send Syslog.

---

## 5. Production example

Firewall blocks traffic.

Firewall sends syslog.

SIEM receives

```
Dropped packet
```

Engineer immediately identifies firewall issue.

---

## 6. Troubleshooting

Customer

```
Website unavailable.
```

Syslog shows

```
nginx

Permission denied

SELinux blocked access
```

Problem solved quickly.

---

## 7. Dependencies

Many services write using syslog API.

Examples

```
Apache

Nginx

SSH

Cron

Kernel

Cisco devices

Firewalls
```

---

## 8. Analogy

Like English.

Every country speaks different language.

English allows everyone to communicate.

Syslog allows every application to log consistently.

---

## 9. Three questions

1.

What is Syslog?

2.

Difference between Syslog and journalctl?

3.

What are Syslog severity levels?

---

# Syslog Severity

```
0 Emergency

1 Alert

2 Critical

3 Error

4 Warning

5 Notice

6 Info

7 Debug
```

Interviewers love this.

---

# Syslog Facilities

Examples

```
auth

daemon

kern

cron

mail

user

local0-local7
```

Facility tells

> who generated the log.

---

# 3. rsyslog

## 1. What is this?

rsyslog is a **Syslog server**.

It receives logs and stores them.

Usually inside

```
/var/log
```

Examples

```
/var/log/syslog

/var/log/messages

/var/log/auth.log

/var/log/cron

/var/log/secure
```

---

## 2. Why required?

journalctl stores binary logs.

Sometimes companies want

* plain text
* remote logging
* forwarding
* filtering

rsyslog does all this.

---

## 3. Mid-level DevOps benefit

You can

Forward logs

```
Server A
      │
      ▼
Central Log Server
```

Very common.

---

## 4. Recruiters care because

Large companies rarely inspect logs manually.

They centralize logs.

rsyslog is often the first forwarding step.

---

## 5. Production example

One hundred servers.

Disk crashes.

Logs lost.

Instead,

rsyslog forwarded logs every second.

Logs already exist on central server.

Incident investigation succeeds.

---

## 6. Troubleshooting example

Developer says

```
Application crashed yesterday.
```

Application logs deleted.

Central rsyslog still has them.

Root cause found.

---

## 7. Dependencies

Depends on

```
Syslog protocol
```

Can receive logs from

Linux

Routers

Switches

Firewalls

Applications

Can forward to

ELK

Graylog

Splunk

SIEM

Remote servers

---

## 8. Analogy

Imagine a courier company.

Everyone writes letters (logs).

Courier collects them.

Delivers to archive.

rsyslog is the courier.

---

## 9. Three questions

1.

Difference between rsyslog and Syslog?

2.

Why use rsyslog if journalctl exists?

3.

How do companies centralize logs?

---

# journalctl vs syslog vs rsyslog

| Topic           | journalctl | Syslog           | rsyslog        |
| --------------- | ---------- | ---------------- | -------------- |
| What is it?     | Log viewer | Logging standard | Logging daemon |
| Storage         | Binary     | No storage       | Text files     |
| Reads logs      | Yes        | No               | Yes            |
| Writes logs     | No         | Standard only    | Yes            |
| Network logging | Limited    | Defines protocol | Yes            |
| Used today      | Yes        | Yes              | Yes            |

---

# Real Production Flow

```
Docker crashes

↓

systemd captures event

↓

journald stores binary logs

↓

rsyslog forwards logs

↓

ELK / Loki

↓

Grafana Dashboard

↓

Engineer investigates
```

---

# 15 Important Interview Questions

## 1. What is the difference between journald and rsyslog?

**Answer:** journald collects and stores logs in a binary journal managed by systemd. rsyslog is a Syslog daemon that can store logs as text files, filter them, and forward them to remote servers. Many systems use journald to collect logs and rsyslog to archive or forward them.

---

## 2. What is Syslog?

**Answer:** Syslog is a standardized logging protocol and message format that allows operating systems, applications, and network devices to generate logs consistently.

---

## 3. Why are journald logs binary?

**Answer:** Binary storage provides faster indexing, structured metadata (such as service name, PID, boot ID), efficient compression, and integrity checks. `journalctl` is used to query these logs.

---

## 4. How do you see logs for a service?

```
journalctl -u nginx
```

---

## 5. How do you see logs since the last reboot?

```
journalctl -b
```

To see the previous boot:

```
journalctl -b -1
```

---

## 6. How do you continuously watch logs?

```
journalctl -f
```

Similar to:

```
tail -f
```

---

## 7. How would you investigate a failed service?

1. `systemctl status <service>`
2. `journalctl -u <service>`
3. Check configuration files.
4. Restart the service and verify the logs again.

---

## 8. Where are traditional Linux logs stored?

Common locations include:

* `/var/log/syslog` (Debian/Ubuntu)
* `/var/log/messages` (RHEL/CentOS)
* `/var/log/auth.log`
* `/var/log/secure`
* `/var/log/kern.log`

---

## 9. Why centralize logs?

To retain logs even if a server fails, simplify troubleshooting across multiple servers, improve security auditing, and meet compliance requirements.

---

## 10. What is a Syslog facility?

A facility identifies the source of a log message (for example, `auth`, `cron`, `daemon`, or `kern`), making filtering and routing easier.

---

## 11. What is a Syslog severity level?

A severity indicates how important a message is, ranging from `Emergency (0)` to `Debug (7)`.

---

## 12. How does journald interact with rsyslog?

On many Linux systems, journald collects logs first. rsyslog can then read from journald (or receive Syslog messages directly) and write them to text files or forward them to remote logging systems.

---

## 13. When would you use `journalctl` instead of reading `/var/log`?

Use `journalctl` for services managed by systemd, when you need structured filtering (by unit, boot, priority, time, or PID), or when logs are stored only in the journal.

---

## 14. Why is remote logging important?

It preserves logs if a server is compromised or its disk fails, enables centralized monitoring, and allows correlation of events across many systems.

---

## 15. An application crashed overnight. How would you investigate?

A strong answer would include:

1. Check whether the service is running with `systemctl status`.
2. Inspect service logs using `journalctl -u <service>`.
3. Review logs from the previous boot if the server rebooted (`journalctl -b -1`).
4. Check kernel logs for OOM kills, hardware, or filesystem issues.
5. Correlate application logs with system logs to identify the root cause.

---

# Mid-level DevOps Interview Tip

For a mid-level DevOps role, don't stop at saying **"I check the logs."** Explain **which logs** and **why**.

For example:

> "If a systemd-managed service fails, I first run `systemctl status` to get the high-level failure reason. Then I use `journalctl -u <service>` to inspect detailed logs, checking for configuration errors, permission issues, missing dependencies, or startup failures. If the problem started after a reboot, I'd compare the current and previous boot logs using `journalctl -b` and `journalctl -b -1`. If multiple servers are involved, I'd correlate those logs with the centralized logging platform such as Loki, ELK, or Splunk."

That answer demonstrates not just familiarity with the commands, but a structured production troubleshooting process—the level of reasoning interviewers expect from a mid-level DevOps engineer.
