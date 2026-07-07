This is one of the **highest priority topics** for a Mid-level DevOps Engineer.

Why?

Because almost every production Linux server today (Ubuntu, Debian, RHEL, Rocky, AlmaLinux, Amazon Linux 2/2023, etc.) uses **systemd**. Whenever an application is installed (Nginx, Docker, Jenkins, PostgreSQL, Prometheus, Grafana, SSH, etc.), you will almost always manage it through **systemd**.

Think of systemd as the **Operations Manager of Linux**.

It decides:

* What starts during boot
* What order services start
* Which services depend on others
* What happens if a service crashes
* Where logs are stored
* Whether a service should restart automatically

---

# Before learning commands

## What is systemd?

systemd is the **init system** of modern Linux.

It is the **first userspace process** started by the kernel.

```
Kernel
   │
   ▼
systemd (PID 1)
   │
   ├── ssh
   ├── nginx
   ├── docker
   ├── mysql
   ├── cron
   └── every other service
```

Everything eventually comes under systemd.

You can verify:

```bash
ps -p 1
```

Output

```
PID TTY      TIME CMD
1 ?        00:00:03 systemd
```

PID 1 is special.

If PID 1 dies...

Linux crashes.

---

# 1. systemctl

---

## What is it?

systemctl is the command used to communicate with systemd.

Think of it as the **remote control** for Linux services.

Examples

```bash
systemctl start nginx
```

```bash
systemctl stop docker
```

```bash
systemctl restart ssh
```

```bash
systemctl status mysql
```

---

## Why required?

Without systemctl you'd have to manually

* start processes
* stop them
* restart them
* configure boot
* inspect status

systemctl centralizes everything.

---

## Mid-Level DevOps Importance

Suppose Jenkins failed.

Instead of

```
kill
ps
nohup
```

You simply

```bash
systemctl restart jenkins
```

Or

```bash
systemctl status jenkins
```

Production engineers use this constantly.

---

## Why recruiters care?

Interviewer thinks

> If production breaks,
>
> can this person safely restart a service?

systemctl is the standard answer.

---

## Production problem

Example

Website is down.

Check

```bash
systemctl status nginx
```

Output

```
failed
```

Restart

```bash
systemctl restart nginx
```

Problem solved.

---

## Troubleshooting

Example

```
Docker won't start.
```

Run

```bash
systemctl status docker
```

You may see

```
Dependency failed
```

Now you know the problem is not Docker.

It's another dependency.

---

## Dependencies affected

Almost everything.

Examples

* Docker
* SSH
* nginx
* PostgreSQL
* Jenkins
* Prometheus
* Grafana
* Redis

---

## Analogy

systemctl is the TV remote.

systemd is the TV.

The services are channels.

---

## Interview Questions

1.

Difference between

```
service nginx restart
```

and

```
systemctl restart nginx
```

---

2.

How do you know whether a service is running?

---

3.

How do you restart Docker?

---

# 2. journalctl

---

## What is it?

journalctl reads logs stored by systemd.

Instead of checking many log files,

systemd stores logs in one central journal.

```
Application

↓

systemd journal

↓

journalctl
```

---

## Why required?

Logs are everything in production.

Without logs

You are guessing.

With logs

You know exactly what failed.

---

## DevOps Importance

Whenever a service crashes

First command

```bash
journalctl -u nginx
```

or

```bash
journalctl -xe
```

---

## Recruiters care because

Troubleshooting always begins with logs.

---

## Production problem

Jenkins won't start.

Run

```bash
journalctl -u jenkins
```

Output

```
Port already in use
```

Immediately you know another application uses port 8080.

---

## Troubleshooting

Only today's logs

```bash
journalctl --since today
```

Live logs

```bash
journalctl -f
```

Specific service

```bash
journalctl -u docker
```

Previous boot

```bash
journalctl -b -1
```

---

## Dependencies

Every service managed by systemd logs here.

---

## Analogy

Think of journalctl as CCTV footage.

Something happened yesterday?

Watch the recording.

---

## Interview Questions

1.

How do you see logs of nginx?

---

2.

How do you watch logs in real time?

---

3.

How do you check previous boot logs?

---

# 3. Service Units

---

## What is it?

A service unit is simply a configuration file telling systemd

How to run an application.

Example

```
/etc/systemd/system/nginx.service
```

Inside

```ini
[Unit]
Description=Nginx

[Service]
ExecStart=/usr/sbin/nginx

[Install]
WantedBy=multi-user.target
```

---

## Why required?

Instead of remembering

```
/usr/bin/java
-Xmx2G
...
```

systemd stores everything in one place.

---

## DevOps Importance

You'll often create your own service.

Example

Python API

NodeJS API

Go application

Spring Boot

---

## Production problem

Your Python app dies after logout.

Instead,

Create

```
myapp.service
```

systemd keeps it alive.

---

## Troubleshooting

```bash
systemctl cat myapp
```

Check

```
ExecStart
WorkingDirectory
User
Environment
```

---

## Dependencies

A service unit may require

```
network.target

postgresql.service

docker.service
```

---

## Analogy

Service unit = Employee job description.

systemd = Manager.

---

## Interview Questions

1.

Where are service unit files?

---

2.

What is ExecStart?

---

3.

How do you reload systemd after editing?

```
systemctl daemon-reload
```

---

# 4. Target

---

## What is it?

Target is a collection of services.

Old Linux

```
Runlevels
```

Modern Linux

```
Targets
```

Example

```
multi-user.target
```

means

Boot into command line.

---

```
graphical.target
```

means

Boot GUI.

---

## Why required?

Instead of starting hundreds of services manually,

systemd starts an entire group.

---

## Production

Servers usually use

```
multi-user.target
```

No GUI.

Less RAM.

---

## Troubleshooting

Current target

```bash
systemctl get-default
```

---

Switch

```bash
systemctl isolate rescue.target
```

---

## Analogy

Target = Startup mode of your phone.

Safe Mode.

Recovery Mode.

Normal Mode.

---

## Interview Questions

1.

Difference between target and service?

---

2.

What replaced runlevels?

---

3.

Why do servers use multi-user.target?

---

# 5. Socket Activation

---

## What is it?

A service doesn't start until someone connects.

Normally

```
Boot

↓

Start SSH

↓

Wait...
```

Socket activation

```
Boot

↓

Wait

↓

User connects

↓

Start SSH
```

---

## Why required?

Faster boot.

Less RAM.

---

## DevOps

Useful for infrequently used services.

---

## Production

Hundreds of services.

Most aren't used often.

Don't waste RAM.

---

## Troubleshooting

```bash
systemctl list-sockets
```

---

## Dependencies

Works with

```
.socket

↓

.service
```

---

## Analogy

Restaurant.

Chef doesn't cook until customer orders.

---

## Interview Questions

1.

What is socket activation?

---

2.

Benefits?

---

3.

Which unit activates it?

---

# 6. Timers

---

## What is it?

systemd replacement for cron.

Runs tasks at scheduled times.

---

Instead of

```
cron
```

Use

```
.timer
```

---

## Why required?

More reliable.

Integrated with logging.

Dependencies.

---

## DevOps

Backup

Cleanup

Log rotation

Health check

---

## Troubleshooting

```bash
systemctl list-timers
```

---

## Production

Database backup every night.

---

## Analogy

Alarm clock.

---

## Interview Questions

1.

Difference between cron and timers?

---

2.

How do you list timers?

---

3.

Why prefer timers?

---

# 7. Dependencies

---

## What is it?

Some services need other services.

Example

Web application

↓

Needs

Database

↓

Needs

Network

---

```
network

↓

postgres

↓

backend

↓

nginx
```

---

systemd manages this order.

---

## Why required?

Without dependency management

Application starts

Database isn't ready

Application crashes.

---

## DevOps

Critical.

Almost every production app has dependencies.

---

## Troubleshooting

```bash
systemctl list-dependencies nginx
```

---

Example

```
Dependency failed.

```

Now investigate the required service.

---

## Analogy

You cannot drive before starting the engine.

---

## Interview Questions

1.

What is `After=`?

2.

Difference between `Requires=` and `Wants=`?

3.

How do you see dependencies?

---

# 8. Enable

---

## What is it?

Start automatically during boot.

```bash
systemctl enable nginx
```

---

Without enable

Reboot

↓

Service never starts.

---

Production

Always enable important services.

---

Interview

Difference between

```
start

enable
```

must be crystal clear.

---

# 9. Disable

---

Stops automatic startup.

```
systemctl disable nginx
```

Manual start still works.

---

Useful

Testing

Temporary shutdown

---

# 10. Mask

---

Most misunderstood interview question.

Mask means

> **Nobody can start this service.**

Not manually.

Not automatically.

Not another service.

Example

```bash
systemctl mask bluetooth
```

Even

```bash
systemctl start bluetooth
```

fails.

Internally it points the service to `/dev/null`, making it impossible for systemd to load the unit.

---

## Why production uses mask

Suppose an old service conflicts with a new one.

Disable is insufficient because someone (or another service) could still start it.

Mask guarantees it stays off.

---

## Interview

Difference

```
disable

mask
```

---

# 11. Restart

---

Stop

↓

Start again.

```bash
systemctl restart nginx
```

Use after configuration changes that require a full restart.

---

Production

Memory leak

Configuration update

Crash recovery

---

# 12. Reload

---

Reload configuration **without stopping** the process.

```bash
systemctl reload nginx
```

Nginx re-reads its configuration while continuing to serve requests.

Not every service supports reload. If a service has no reload capability, you'll need to restart it instead.

---

## Production

Update

```
nginx.conf
```

No downtime.

Perfect.

---

# Restart vs Reload (Very Important Interview Question)

| Restart                                                  | Reload                                                          |
| -------------------------------------------------------- | --------------------------------------------------------------- |
| Stops the service and starts it again                    | Keeps the service running                                       |
| Existing connections may be interrupted                  | Existing connections are usually preserved                      |
| Reloads binaries and configuration                       | Reloads configuration only                                      |
| Causes a brief interruption in many cases                | Typically avoids downtime                                       |
| Use when the application crashes or needs a full restart | Use after changing configuration if the service supports reload |

---

# Production Scenario (Interview Favorite)

Imagine your production website suddenly returns **502 Bad Gateway** after a deployment.

A structured approach would be:

1. Check the web server:

   ```bash
   systemctl status nginx
   ```
2. If it is running, inspect its logs:

   ```bash
   journalctl -u nginx --since "10 minutes ago"
   ```
3. If Nginx reports it cannot reach the backend, check the backend service:

   ```bash
   systemctl status myapp
   ```
4. If the backend failed because PostgreSQL is unavailable, inspect:

   ```bash
   systemctl status postgresql
   ```
5. If PostgreSQL failed after a reboot, examine dependency ordering:

   ```bash
   systemctl list-dependencies postgresql
   ```
6. After fixing the root cause, use:

   ```bash
   systemctl restart postgresql
   systemctl restart myapp
   ```
7. If you only changed the Nginx configuration:

   ```bash
   systemctl reload nginx
   ```

This demonstrates a production mindset: follow service dependencies, use logs to identify the root cause, and choose **reload** over **restart** when possible to minimize downtime.
