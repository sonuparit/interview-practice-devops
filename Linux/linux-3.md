This is one of the **highest ROI Linux topics** for a Mid-level DevOps Engineer.

A surprising number of production incidents are caused by **incorrect file permissions**, not bugs in the application.

Examples:

* Nginx can't read SSL certificate.
* Docker container can't write logs.
* Jenkins can't access workspace.
* Kubernetes Pod crashes because mounted volume isn't writable.
* SSH refuses to use a private key because its permissions are too open.
* Application cannot read configuration files after deployment.

Let's cover every topic from a production perspective.

---

# 1. chmod

## 1. What is this?

`chmod` means **Change Mode**.

It changes file or directory permissions.

Example

```bash
chmod 644 file.txt
chmod +x script.sh
chmod -R 755 website/
```

---

## 2. Why is it required?

Linux follows security principles.

Every file must define

* Who can read
* Who can write
* Who can execute

Without chmod, everyone could modify everything.

---

## 3. DevOps Importance

You'll use it every day.

Examples

Deployment script

```bash
chmod +x deploy.sh
```

Nginx

```bash
chmod 600 private.key
```

SSH

```bash
chmod 600 ~/.ssh/id_rsa
```

Docker entrypoint

```bash
chmod +x entrypoint.sh
```

---

## 4. Why recruiters ask?

Because permissions are responsible for many production outages.

If you understand chmod, you're trusted to deploy safely.

---

## 5. Production Problem

Problem

```
Permission denied
```

Fix

```bash
chmod +x app
```

---

## 6. Troubleshooting

User cannot execute

```bash
./backup.sh
```

Error

```
Permission denied
```

Check

```bash
ls -l
```

Output

```
-rw-r--r--
```

No execute permission.

Fix

```bash
chmod +x backup.sh
```

---

## 7. Services affected

Almost everything.

Examples

* systemd
* nginx
* apache
* ssh
* docker
* kubernetes
* cron

---

## 8. Analogy

Think of a hotel room.

You decide

* Who can enter
* Who can clean
* Who can modify

chmod changes those permissions.

---

## 9. Interview Questions

* Why chmod 777 is dangerous?
* Difference between chmod +x and chmod 755?
* Why does "Permission denied" occur?

---

# 2. chown

## 1. What is this?

Changes file owner.

```bash
chown sonu file.txt

chown nginx:nginx index.html
```

---

## 2. Why required?

Linux checks ownership before allowing access.

---

## 3. DevOps Importance

Applications usually run as dedicated users.

Examples

```
nginx

mysql

postgres

jenkins

www-data
```

If ownership is wrong

Application fails.

---

## 4. Recruiters care because

Ownership issues are very common.

---

## 5. Production Problem

Nginx

```
403 Forbidden
```

Reason

Owner

```
root
```

Instead of

```
www-data
```

Fix

```bash
chown www-data:www-data index.html
```

---

## 6. Troubleshooting

Check

```bash
ls -l
```

Fix

```bash
chown -R app:app /var/www
```

---

## 7. Services affected

* nginx
* mysql
* postgres
* docker
* kubernetes volumes

---

## 8. Analogy

House ownership.

Even if door is unlocked

Only owner has rights.

---

## 9. Questions

* Difference between chmod and chown?
* Why use dedicated users?
* When should ownership be changed?

---

# 3. chgrp

## 1. What is this?

Changes group ownership.

```bash
chgrp developers file.txt
```

---

## 2. Why required?

Allows teams to share files.

---

## 3. DevOps Importance

Shared deployment directory.

Developers

Operations

CI/CD

Need common access.

---

## 4. Recruiters care

Shows understanding of Linux multi-user systems.

---

## 5. Production

Jenkins and developers both need access.

Instead of changing owner repeatedly

Use groups.

---

## 6. Troubleshooting

Check

```bash
ls -l
```

Fix

```bash
chgrp devops app.log
```

---

## 7. Services

* Jenkins
* GitLab Runner
* Shared storage
* NFS

---

## 8. Analogy

A WhatsApp group.

Everyone in the group shares access.

---

## 9. Questions

* Difference between owner and group?
* Why use groups?
* How to change group?

---

# 4. umask

## 1. What is this?

Default permission mask.

Controls permissions of newly created files.

Example

```
umask 022
```

---

## 2. Why required?

Avoid insecure default permissions.

---

## 3. DevOps Importance

Every deployment creates files.

Need safe defaults.

---

## 4. Recruiters care

Security knowledge.

---

## 5. Production

Developer accidentally creates

```
777
```

Anyone can modify.

Better

```
644
```

---

## 6. Troubleshooting

Check

```bash
umask
```

Unexpected permissions?

Wrong umask.

---

## 7. Services

* shell
* scripts
* deployment tools

---

## 8. Analogy

Factory default settings.

---

## 9. Questions

* What does umask 022 mean?
* Why isn't default 777?
* How does umask affect new files?

---

# 5. rwx Permissions

## What is this?

Three permissions

```
r Read

w Write

x Execute
```

Applied to

Owner

Group

Others

```
rwxr-xr--
```

---

## Why required?

Controls access.

---

## DevOps

Without understanding rwx

You cannot troubleshoot permission problems.

---

## Production

Container cannot write logs.

Reason

No write permission.

---

## Troubleshooting

```
ls -l
```

Look at rwx.

---

## Services

Everything.

---

## Analogy

Book

Read

Write

Perform

---

## Questions

* Difference between read and execute on directories?
* Why execute needed for directories?
* Can a file run without x?

---

# 6. Numeric Permissions

Example

```
755

644

600

777
```

Numbers

```
Read=4

Write=2

Execute=1
```

Example

```
7=4+2+1

5=4+1

6=4+2
```

---

DevOps

Used everywhere.

Dockerfiles

Scripts

CI/CD

Terraform

---

Production

```
chmod 600 id_rsa
```

SSH works.

---

Questions

* Why 755?
* Why 644?
* Why 600 for SSH?

---

# 7. Symbolic Permissions

Example

```bash
chmod u+x

chmod g+w

chmod o-r

chmod a+x
```

---

Useful when changing one permission.

---

Questions

* Difference from numeric?
* What does u mean?
* What does a+x do?

---

# 8. SUID (Set User ID)

## What is this?

When executable runs

It uses owner's privileges.

Not user's.

Example

```
-rwsr-xr-x
```

Notice

```
s
```

---

Example

```
passwd
```

Normal users can change passwords because passwd runs as root.

---

Why required?

Some programs need temporary elevated privilege.

---

Production

Without SUID

Users couldn't update passwords.

---

Troubleshooting

Check

```bash
ls -l
```

Look for

```
s
```

---

Services

Authentication utilities

---

Analogy

Receptionist borrowing manager's authority.

---

Questions

* Why passwd uses SUID?
* Why dangerous?
* How to detect SUID?

---

# 9. SGID (Set Group ID)

## What is this?

Files

Program runs with group privileges.

Directories

New files inherit directory group.

---

Useful

Shared project folders.

---

Production

Developers create files.

Everyone automatically belongs to same group.

---

Troubleshooting

```
ls -ld
```

Look for

```
s
```

---

Analogy

Everyone joining a team automatically gets the team's badge.

---

Questions

* SGID on directory?
* SGID vs SUID?
* Why useful for teams?

---

# 10. Sticky Bit

## What is this?

On directories

Only owner can delete own files.

Even if directory is writable.

```
drwxrwxrwt
```

Notice

```
t
```

---

Example

```
/tmp
```

---

Production

Multiple users share directory.

Prevent deleting each other's files.

---

Troubleshooting

```
ls -ld /tmp
```

---

Analogy

Shared notice board.

Everyone can pin papers.

Only owner removes own paper.

---

Questions

* Why /tmp has sticky bit?
* Difference between sticky and SGID?
* What happens without sticky bit?

---

# 11. Permission Inheritance

## What is this?

New files inherit

* ownership rules
* group (with SGID)
* default permissions (via umask)
* default ACLs (if configured)

---

Why important?

Keeps permissions consistent.

---

Production

Shared project directory.

Every new deployment automatically gets correct group.

---

Troubleshooting

Unexpected permissions?

Check

```
umask

SGID

ACLs

Parent directory
```

---

Analogy

Children inherit family surname.

---

Questions

* What controls inherited permissions?
* Does chmod inherit?
* What affects inheritance?

---

# 12. Default Permissions

Default file permissions before applying `umask` are:

* Files: `666` (`rw-rw-rw-`) — no execute bit by default.
* Directories: `777` (`rwxrwxrwx`) — directories need execute permission to be traversable.

The effective permissions are calculated by subtracting the `umask`.

Example:

```
umask = 022

New file:
666 - 022 = 644

New directory:
777 - 022 = 755
```

---

Production

Company policy

```
umask 027
```

Prevents other users from reading sensitive files.

---

Troubleshooting

New files always have unexpected permissions.

Check

```
umask
```

---

Questions

* Why are new files not executable by default?
* How does `umask` affect default permissions?
* Why do directories start with `777` while files start with `666`?

---

# Production Troubleshooting Flow (Very Common in Interviews)

When you encounter a **"Permission denied"** error:

1. Check the permissions:

   ```bash
   ls -l <file>
   ```
2. Verify the owner and group:

   ```bash
   stat <file>
   ```
3. Confirm which user the process is running as:

   ```bash
   ps -ef | grep <process>
   ```
4. Check the parent directory permissions (directory traversal requires execute permission):

   ```bash
   ls -ld <parent-directory>
   ```
5. Verify the current user's groups:

   ```bash
   id
   groups
   ```
6. Check for ACLs (which can override basic permissions):

   ```bash
   getfacl <file>
   ```
7. If SELinux or AppArmor is enabled, verify they aren't blocking access.
8. Apply the appropriate fix (`chmod`, `chown`, `chgrp`, ACL changes, or policy updates) rather than granting overly broad permissions like `777`.

This systematic approach is exactly what interviewers look for in a mid-level DevOps engineer: identifying the root cause instead of applying insecure permission changes.