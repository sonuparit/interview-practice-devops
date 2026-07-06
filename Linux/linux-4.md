This is one of the **highest-value Linux topics** for a Mid-level DevOps Engineer.

Why?

Because almost every production issue eventually becomes an identity and permission problem.

For example:

* Jenkins cannot deploy because of permissions.
* Kubernetes node cannot mount EBS because IAM/user mapping.
* Docker container cannot access mounted volume.
* SSH login suddenly stops working.
* Service account cannot read secrets.
* Cron job fails.
* Application cannot write logs.
* Sudo access broken after server migration.

Almost every one of these starts with **"Which user is running this?"**

---

# Mental Model First

Before learning commands, understand Linux's security model.

Imagine a company.

```
Company
│
├── Employees (Users)
│      │
│      ├── Sonu
│      ├── Rahul
│      ├── Jenkins
│      ├── nginx
│      └── postgres
│
├── Departments (Groups)
│      │
│      ├── Developers
│      ├── Finance
│      ├── Docker
│      └── sudo
│
└── HR Rules
       │
       ├── Who can enter where
       ├── Salary
       ├── Password
       ├── Promotion
       └── Access permissions
```

Linux works almost exactly like this.

Everything is based on

* Users
* Groups
* Permissions

---

# 1. useradd

## What is it?

Creates a new Linux user.

Example

```bash
sudo useradd sonu
```

---

## Why required?

Linux is multi-user.

Every human

Every service

Every application

Every automation

should run under a separate identity.

Examples

```
nginx
mysql
jenkins
gitlab-runner
prometheus
grafana
```

These are Linux users.

---

## DevOps importance

Suppose

```
Jenkins
```

runs as

```
jenkins
```

instead of

```
root
```

If Jenkins gets hacked,

attacker only gets

```
jenkins
```

permissions.

Not full server.

This is called

**Principle of Least Privilege.**

---

## Recruiter expectation

They want to know

* Why services run under different users
* Security best practices
* User isolation

---

## Production example

Application writes logs.

Instead of

```
root
```

Create

```
appuser
```

Run application as

```
appuser
```

If compromised,

damage stays limited.

---

## Troubleshooting

Problem

```
Permission denied
```

Check

```bash
ps -ef
```

See

```
Which user is running?
```

---

## Dependencies

Affects

* SSH
* sudo
* cron
* systemd
* Docker
* Jenkins
* Nginx

Because every process belongs to some Linux user.

---

## Analogy

Creating a new employee in a company.

---

## Questions

1. Why shouldn't applications run as root?
2. What happens if you delete a service user?
3. Why create separate users for different services?

---

# 2. usermod

## What is it?

Modify existing users.

Example

```bash
sudo usermod -aG docker sonu
```

Adds user to docker group.

---

## Why required?

People change roles.

Need new permissions.

Need new groups.

Need new shell.

Need home directory changes.

---

## DevOps usage

Very common.

Example

```
Developer cannot use Docker.
```

Solution

```bash
sudo usermod -aG docker developer
```

---

## Production problem

Jenkins cannot execute Docker.

Check

```
groups jenkins
```

Missing

```
docker
```

Add group.

Restart Jenkins.

Done.

---

## Troubleshooting

Check

```bash
id jenkins
```

---

## Dependencies

* Docker
* Kubernetes
* GitLab Runner
* SSH
* sudo

---

## Analogy

Employee transferred to another department.

---

## Questions

1. Difference between `-G` and `-aG`?
2. Why must the user log out after changing groups?
3. Why does removing a group sometimes break applications?

---

# 3. userdel

## What is it?

Deletes users.

Example

```bash
sudo userdel sonu
```

Delete including home

```bash
sudo userdel -r sonu
```

---

## Why required?

Employees leave.

Need cleanup.

---

## DevOps importance

Security.

Unused users

=

Unused attack surface.

---

## Production example

Former employee still has SSH access.

Delete account immediately.

---

## Troubleshooting

Check

```
last
```

See if deleted user recently logged in.

---

## Dependencies

SSH

Cron

Running services

---

## Analogy

Employee exits company.

Deactivate ID card.

---

## Questions

1. Difference between `userdel` and `userdel -r`?
2. What happens if deleted user owns files?
3. Can running processes continue after deleting the user?

---

# 4. passwd

## What is it?

Sets or changes passwords.

```bash
passwd sonu
```

---

## Why required?

Authentication.

---

## DevOps importance

Emergency password reset.

Lock compromised account.

Expire passwords.

---

## Production

Developer forgot password.

Reset quickly.

---

## Troubleshooting

Cannot login.

Check

```bash
passwd -S sonu
```

---

## Dependencies

SSH

PAM

Login

sudo

---

## Analogy

Changing ATM PIN.

---

## Questions

1. Where is password stored?
2. Is password stored in plain text?
3. How do password hashes work?

---

# 5. groupadd

## What is it?

Creates groups.

```bash
groupadd developers
```

---

## Why?

Permissions become easier.

Instead of

100 users

Give permission to

one group.

---

## DevOps

Shared deployment directory.

```
developers
```

gets access.

---

## Production

Multiple developers deploy application.

Just add users to group.

---

## Troubleshooting

Application cannot read files.

Check group ownership.

---

## Dependencies

ACL

Permissions

Docker

---

## Analogy

Create HR department.

---

## Questions

1. Why groups instead of individual permissions?
2. Primary vs supplementary groups?
3. Why are groups more scalable?

---

# 6. groups

## What is it?

Shows groups user belongs to.

```bash
groups sonu
```

---

## DevOps

First command when permission issues occur.

---

## Production

Docker denied.

Check

```
groups
```

Missing docker group.

---

## Troubleshooting

Very common.

---

## Analogy

Departments employee belongs to.

---

## Questions

1. Why check groups first?
2. Why can missing groups cause permission denied?
3. Difference from id?

---

# 7. id

## What is it?

Shows

* UID
* GID
* Groups

```bash
id sonu
```

---

## DevOps

Extremely useful.

Example

Container volume ownership mismatch.

Need UID.

---

## Production

NFS permissions fail.

Need matching UID.

---

## Troubleshooting

```bash
id
```

Immediately tells

Who am I?

---

## Dependencies

NFS

Docker

Kubernetes

Volumes

---

## Analogy

Employee ID card.

---

## Questions

1. Difference between UID and username?
2. Why UID matters more than username?
3. What happens if two users share the same UID?

---

# 8. who

## What is it?

Shows currently logged-in users.

```bash
who
```

---

## DevOps

Know who is using server.

---

## Production

Unexpected login.

Investigate immediately.

---

## Troubleshooting

Unexpected reboot.

See active users.

---

## Analogy

Visitors register.

---

## Questions

1. Difference from `w`?
2. What information does it show?
3. Why useful before maintenance?

---

# 9. w

## What is it?

Shows

* Logged users
* Commands
* CPU usage
* Login time

---

## DevOps

Know what users are doing.

---

## Production

Before reboot

Run

```bash
w
```

See

```
Someone running database migration.
```

Don't reboot.

---

## Troubleshooting

High load.

See which session causing it.

---

## Analogy

Live CCTV.

---

## Questions

1. Difference between `who` and `w`?
2. Why use before reboot?
3. How identify long-running sessions?

---

# 10. last

## What is it?

Login history.

```bash
last
```

---

## DevOps

Security audits.

---

## Production

Server compromised.

Need login history.

---

## Troubleshooting

Who logged in before outage?

Use

```
last
```

---

## Analogy

Visitor logbook.

---

## Questions

1. Where does `last` get its information?
2. Difference between current users and historical users?
3. Why useful after security incidents?

---

# 11. `/etc/passwd`

## What is it?

User database.

Each line represents one user.

Example:

```text
sonu:x:1000:1000:Sonu:/home/sonu:/bin/bash
```

Fields (colon-separated):

1. Username (`sonu`)
2. Password placeholder (`x`; actual hash is in `/etc/shadow`)
3. UID (`1000`)
4. Primary GID (`1000`)
5. GECOS/comment (e.g., full name)
6. Home directory (`/home/sonu`)
7. Login shell (`/bin/bash`)

---

## Why required?

Linux needs a central mapping of usernames to UIDs, home directories, and login shells.

---

## DevOps importance

Many services and scripts resolve users through this file. A missing or incorrect entry can prevent logins or services from starting.

---

## Production problem

A service fails because its configured user doesn't exist.

Check:

```bash
grep nginx /etc/passwd
```

---

## Troubleshooting

Verify:

* Does the user exist?
* Is the shell valid?
* Is the home directory correct?

---

## Dependencies

* SSH
* Login
* systemd services
* Cron

---

## Analogy

Company employee directory.

---

## Questions

1. What does each field mean?
2. Why is there an `x` instead of a password?
3. Can a user exist without a home directory?

---

# 12. `/etc/shadow`

## What is it?

Stores password hashes and password-aging information.

Readable only by `root`.

---

## Why required?

Passwords must not be visible to normal users.

---

## DevOps importance

Essential for secure authentication and enforcing password policies.

---

## Production problem

A user's password expires unexpectedly and they cannot log in.

---

## Troubleshooting

Check password status:

```bash
passwd -S sonu
```

View aging information:

```bash
sudo chage -l sonu
```

---

## Dependencies

* PAM
* SSH
* Login

---

## Analogy

The company's secure vault containing employee PIN hashes.

---

## Questions

1. Why are password hashes stored here instead of `/etc/passwd`?
2. Why are hashes used instead of plain-text passwords?
3. What password-aging information is stored?

---

# 13. `/etc/group`

## What is it?

Stores group definitions and memberships.

---

## Why required?

Linux needs to know which users belong to which groups.

---

## DevOps importance

Group membership controls access to shared resources such as Docker, logs, or deployment directories.

---

## Production problem

A developer can't use Docker because they're not in the `docker` group.

---

## Troubleshooting

Check:

```bash
grep docker /etc/group
```

or

```bash
groups sonu
```

---

## Dependencies

* File permissions
* Docker
* Shared directories
* sudo (through the `sudo` or `wheel` group on some distributions)

---

## Analogy

Department membership list.

---

## Questions

1. What information does `/etc/group` contain?
2. Why are groups preferable to assigning permissions one user at a time?
3. How do primary and supplementary groups differ?

---

# 14. sudo

## What is it?

Allows a permitted user to execute commands as another user (most commonly `root`) without logging in as that user.

Example:

```bash
sudo systemctl restart nginx
```

---

## Why required?

Provides controlled administrative access while maintaining accountability.

---

## DevOps importance

Almost every infrastructure task—installing packages, restarting services, editing system files, viewing restricted logs—uses `sudo`.

---

## Production problem

A deployment script fails because the executing user lacks privileges to restart a service.

---

## Troubleshooting

Check:

* Is the user allowed to use `sudo`?
* Does the command require elevated privileges?
* Are there policy restrictions?

---

## Dependencies

* PAM
* `sudoers`
* system administration tasks

---

## Analogy

A manager temporarily authorizing an employee to access a restricted control room.

---

## Questions

1. Why is using `sudo` safer than logging in directly as `root`?
2. How does `sudo` improve auditing?
3. What happens if a user isn't authorized to use `sudo`?

---

# 15. sudoers

## What is it?

The policy that defines who may use `sudo` and which commands they may execute. It is typically managed with:

```bash
visudo
```

---

## Why required?

Not every administrator should have unrestricted root access.

---

## DevOps importance

Enables least-privilege administration, allowing automation accounts or teams to run only the commands they need.

---

## Production problem

A CI/CD service account needs permission to restart an application but should not be able to modify firewall rules.

---

## Troubleshooting

If `sudo` reports "user is not in the sudoers file," verify the user's group membership and the `sudoers` configuration.

---

## Dependencies

* `sudo`
* PAM

---

## Analogy

A company policy describing exactly which managers can approve which actions.

---

## Questions

1. Why should `visudo` be used instead of editing the file directly?
2. How can you grant permission for only one command?
3. Why is least privilege important in `sudoers`?

---

# 16. PAM basics (Pluggable Authentication Modules)

## What is it?

PAM is Linux's authentication framework. Instead of every application implementing authentication independently, applications ask PAM to authenticate users.

Examples of applications that use PAM:

* SSH
* Login
* `sudo`
* Screen lockers
* Display managers

---

## Why required?

Provides a centralized, consistent, and configurable authentication mechanism.

---

## DevOps importance

When login or authentication suddenly fails across multiple services, PAM is often the common component involved.

---

## Production problem

After changing PAM policies, no one can log in through SSH even though the SSH service is running.

---

## Troubleshooting

Check:

* PAM configuration under `/etc/pam.d/`
* Authentication logs (location varies by distribution, such as `/var/log/auth.log` or `/var/log/secure`)
* Whether account, password, or session modules are failing

---

## Dependencies

* SSH
* Login
* `sudo`
* Password changes
* Password complexity and expiration policies

---

## Analogy

Imagine every office entrance, VPN, and secure room asking the same centralized security desk whether someone should be allowed in. PAM is that security desk.

---

## Questions

1. What problem does PAM solve?
2. Which common Linux services rely on PAM?
3. How can an incorrect PAM configuration affect an entire server?

---

# What Mid-level DevOps Interviewers Usually Expect

By the end of this topic, you should be able to confidently explain and troubleshoot scenarios such as:

* Why services should run under dedicated, non-root users.
* How Linux maps usernames, UIDs, groups, and permissions.
* The roles of `/etc/passwd`, `/etc/shadow`, and `/etc/group`.
* How `sudo` and `sudoers` enforce least privilege.
* How PAM provides centralized authentication.
* How to diagnose login failures, `Permission denied` errors, missing group memberships, broken `sudo` access, and service startup failures related to users and permissions.

Mastering these concepts will prepare you for a large percentage of Linux administration and production troubleshooting questions in mid-level DevOps interviews.