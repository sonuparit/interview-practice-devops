This is one of the highest-value Docker topics for a Mid-level DevOps Engineer.

Many production incidents are **not caused by application bugs**, but by **containers having more privileges than they actually need**.

The security principle behind almost every topic here is:

> **Principle of Least Privilege (PoLP): Give the container only the permissions it absolutely needs—and nothing more.**

---

# How Linux Protects Containers

Before learning each topic, understand the security layers.

```
                     Application
                          │
                    Linux User (UID)
                          │
               Linux Capabilities
                          │
         Read-only Filesystem / Mounts
                          │
         Seccomp (Allowed System Calls)
                          │
      AppArmor / SELinux (Resource Access)
                          │
              Linux Kernel (Host)
```

Each layer removes another attack vector.

---

# 1. Non-root Containers

## What is this?

By default Docker containers run as:

```
UID = 0
```

which is

```
root
```

inside the container.

Instead, run

```dockerfile
USER 1001
```

or

```dockerfile
USER appuser
```

---

## Why required?

If an attacker compromises the application,

they become

```
root
```

inside the container.

That increases the damage they can do.

Running as a normal user greatly limits privileges.

---

## Production Problem

Imagine your web application has Remote Code Execution.

Attacker executes

```bash
rm -rf /
```

If running as root

Huge damage.

If running as non-root

Permission denied on many files.

Much safer.

---

## Troubleshooting

Container fails

```
Permission denied
```

Check

```bash
kubectl exec pod -- id
```

or

```bash
whoami
```

Maybe application expects root-owned directories.

Fix

```
chown

chmod

securityContext
```

---

## Analogy

A hotel employee.

Root

↓

Master key opens every room.

Normal user

↓

Only assigned room.

---

## Test Yourself

1. Why shouldn't containers run as root?

2. How do you verify current user?

3. Why does a non-root container sometimes fail?

---

# 2. USER Instruction

## What is this?

Dockerfile instruction

```dockerfile
USER appuser
```

Changes default runtime user.

---

## Why?

Prevents accidental root execution.

---

## Production Example

CI pipeline checks Dockerfile.

Rejects images without

```
USER
```

because company policy requires non-root.

---

## Troubleshooting

Application suddenly cannot write logs.

Check

```
ls -l
```

Maybe log directory belongs to root.

---

## Analogy

Employee ID card.

Before entering office,

company assigns access level.

---

## Questions

1. Difference between USER and root?

2. Can USER be changed?

3. What if USER doesn't exist?

---

# 3. Linux Capabilities

## What is this?

Root privileges are divided into smaller pieces.

Examples

```
CAP_NET_ADMIN

CAP_SYS_ADMIN

CAP_CHOWN

CAP_KILL
```

Instead of giving full root,

give only required capabilities.

---

## Why?

Much safer.

Example

Web server needs

```
CAP_NET_BIND_SERVICE
```

to bind port 80.

It doesn't need

```
CAP_SYS_ADMIN
```

---

## Production Example

Drop everything

```
--cap-drop ALL
```

Then add only

```
NET_BIND_SERVICE
```

---

## Troubleshooting

Application suddenly cannot

```
ping
```

Needs

```
CAP_NET_RAW
```

Inspect

```
capsh --print
```

---

## Analogy

Instead of giving house keys,

give only garage key.

---

## Questions

1. What are Linux capabilities?

2. Why not give CAP_SYS_ADMIN?

3. How inspect capabilities?

---

# 4. Read-only Filesystem

## What is this?

Container filesystem becomes immutable.

Docker

```bash
docker run --read-only
```

Kubernetes

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

---

## Why?

Malware cannot modify application.

Cannot install tools.

Cannot replace binaries.

---

## Production Example

Ransomware enters container.

Attempts

```
encrypt application
```

Filesystem

↓

Read-only.

Attack fails.

---

## Troubleshooting

Application crashes

```
Read-only filesystem
```

Need writable volume

```
/tmp

/logs

/cache
```

Mount

```
emptyDir

PVC

tmpfs
```

---

## Analogy

Museum.

Visitors can see paintings.

Cannot modify them.

---

## Questions

1. Why read-only?

2. Which directories remain writable?

3. How fix write errors?

---

# 5. Secrets

## What is this?

Sensitive information

* passwords
* API keys
* certificates
* tokens

Never hardcode inside image.

---

Docker

```
Docker Secrets
```

Kubernetes

```
Secret
```

AWS

```
Secrets Manager
```

Vault

```
HashiCorp Vault
```

---

## Production Example

Bad

```dockerfile
ENV PASSWORD=admin123
```

Password now exists forever inside image.

Good

Inject during runtime.

---

## Troubleshooting

Application cannot connect DB.

Check

```
kubectl describe secret
```

```
kubectl get secret
```

```
env
```

Maybe secret missing.

---

## Analogy

Don't print ATM PIN on bank card.

Store separately.

---

## Questions

1. Why not ENV?

2. Secret vs ConfigMap?

3. Runtime injection?

---

# 6. Image Scanning

## What is this?

Scan image for

* CVEs
* outdated packages
* malware

Popular tools

```
Trivy

Grype

Docker Scout

Snyk
```

---

## Why?

Old package

↓

Known exploit.

---

## Production Example

Pipeline

```
Build

↓

Trivy scan

↓

Critical CVE?

↓

Deployment blocked.
```

---

## Troubleshooting

Security team reports

```
OpenSSL CVE
```

Run

```bash
trivy image myimage
```

Upgrade package.

---

## Analogy

Airport security scanner.

---

## Questions

1. What is CVE?

2. Why scan image?

3. When scan?

---

# 7. Docker Bench

## What is this?

Security auditing tool.

Checks Docker host.

Examples

* daemon config
* TLS
* logging
* permissions
* containers

---

## Why?

Verifies Docker follows CIS Benchmark.

---

## Production Example

Docker API exposed publicly

```
2375
```

Docker Bench detects it.

Huge security issue.

---

## Troubleshooting

Company audit fails.

Run

```
docker-bench-security
```

Review failed checks.

---

## Analogy

Vehicle inspection before registration.

---

## Questions

1. What does Docker Bench scan?

2. What standard?

3. Host or image?

---

# 8. Seccomp

## What is this?

Filters Linux system calls.

Application only allowed approved syscalls.

Example

```
open()

read()

write()
```

Dangerous ones

```
ptrace()

mount()

reboot()
```

blocked.

---

## Why?

Even compromised application cannot execute dangerous kernel operations.

---

## Production Example

Attacker

```
mount()
```

Seccomp

↓

Denied.

---

## Troubleshooting

Application

```
Operation not permitted
```

Check

```
dmesg

audit logs
```

Maybe seccomp blocked syscall.

---

## Analogy

Airport security.

Passengers

↓

Cannot enter cockpit.

---

## Questions

1. What does seccomp filter?

2. Where enforced?

3. Default Docker profile?

---

# 9. AppArmor

## What is this?

Linux Security Module.

Controls

```
Files

Capabilities

Signals

Networking
```

per application.

---

## Why?

Limits application behavior.

---

## Production Example

Container compromised.

Attempts

```
cat /etc/shadow
```

AppArmor

↓

Denied.

---

## Troubleshooting

Ubuntu

```
aa-status
```

```
journalctl
```

```
dmesg
```

Look for DENIED entries.

---

## Analogy

Office security guard.

Even employees

↓

Cannot enter restricted rooms.

---

## Questions

1. What controls?

2. How inspect status?

3. Difference from seccomp?

---

# 10. SELinux (High Level)

## What is this?

Mandatory Access Control.

Everything gets labels.

Example

```
Container A

↓

Cannot access

Container B
```

even as root.

---

## Why?

Extra isolation.

Mostly

RHEL

Fedora

OpenShift.

---

## Production Example

Compromised container

↓

Cannot access host files because labels mismatch.

---

## Troubleshooting

Permission denied

But permissions look correct.

Run

```
ls -Z
```

or

```
getenforce
```

Maybe SELinux label mismatch.

---

## Analogy

Airport baggage.

Each bag has label.

Wrong label

↓

Cannot board aircraft.

---

## Questions

1. Difference from chmod?

2. What are labels?

3. Which OS commonly uses SELinux?

---

# Comparison Table

| Feature        | Protects What?            | Example                           |
| -------------- | ------------------------- | --------------------------------- |
| Non-root       | User privileges           | Prevent root access               |
| USER           | Default runtime identity  | Run as appuser                    |
| Capabilities   | Specific root privileges  | Only allow binding to port 80     |
| Read-only FS   | File modifications        | Prevent malware changing binaries |
| Secrets        | Sensitive data            | Database passwords                |
| Image Scanning | Vulnerable packages       | Detect OpenSSL CVEs               |
| Docker Bench   | Docker host configuration | CIS Benchmark checks              |
| Seccomp        | Linux system calls        | Block `mount()`, `ptrace()`       |
| AppArmor       | Process resource access   | Restrict file and network access  |
| SELinux        | Mandatory access control  | Label-based isolation             |

---

# Production Troubleshooting Flow

Suppose your application starts failing after a security hardening update.

```
Container crashes
       │
       ▼
Permission denied?
       │
       ▼
Check current user
(id)
       │
       ▼
Non-root issue?
       │
       ▼
Check file ownership
(ls -l)
       │
       ▼
Still failing?
       │
       ▼
Read-only filesystem?
       │
       ▼
Needs writable /tmp?
       │
       ▼
Still failing?
       │
       ▼
Check capabilities
(capsh --print)
       │
       ▼
Still failing?
       │
       ▼
Check Seccomp/AppArmor/SELinux logs
(dmesg, journalctl, aa-status, ls -Z)
       │
       ▼
Identify blocked syscall or policy
```

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. Why is running a container as root considered a security risk?

**Answer:** If the application is compromised, the attacker gains root privileges inside the container. While container isolation limits the impact, container escape vulnerabilities or access to mounted host resources can make this dangerous. Running as a non-root user reduces the attack surface.

---

### 2. What does the `USER` instruction do?

**Answer:** It sets the default user (UID/GID) for all subsequent `RUN` instructions during build and for the container at runtime unless overridden. It helps enforce the principle of least privilege.

---

### 3. What are Linux capabilities?

**Answer:** Capabilities split root privileges into smaller, independent permissions. Instead of granting full root access, you grant only the specific privileges an application needs, such as `CAP_NET_BIND_SERVICE` for binding to privileged ports.

---

### 4. Why is `CAP_SYS_ADMIN` often called the "new root"?

**Answer:** It grants a very broad set of administrative operations, including mounting filesystems and namespace management. Because of its extensive privileges, it should rarely be granted to containers.

---

### 5. What is the purpose of a read-only root filesystem?

**Answer:** It prevents applications and attackers from modifying the container's root filesystem, reducing persistence opportunities and protecting application binaries from tampering.

---

### 6. If your application needs to write temporary files, how can you use a read-only filesystem?

**Answer:** Keep the root filesystem read-only and mount writable volumes such as `emptyDir`, `tmpfs`, or persistent volumes only where write access is required (for example, `/tmp` or `/var/log`).

---

### 7. Why shouldn't secrets be stored in Docker images or environment variables?

**Answer:** Images are immutable and may be widely distributed. Environment variables are also easy to expose through debugging commands, crash dumps, or process inspection. Dedicated secret management systems provide better access control, rotation, and auditing.

---

### 8. What is the difference between a Kubernetes Secret and a ConfigMap?

**Answer:** Both store configuration data, but Secrets are intended for sensitive information. Kubernetes stores Secret values as Base64-encoded objects (not encrypted by default in etcd unless encryption at rest is enabled), whereas ConfigMaps are for non-sensitive configuration.

---

### 9. What is image scanning?

**Answer:** It is the process of analyzing container images for known vulnerabilities (CVEs), outdated packages, misconfigurations, and sometimes embedded secrets before deployment.

---

### 10. At which stages should image scanning be performed?

**Answer:** Ideally at multiple stages: during development, in CI/CD pipelines before publishing, when pushing to the registry, and periodically afterward because new CVEs can be discovered in existing images.

---

### 11. What is Docker Bench Security?

**Answer:** It is a tool that checks a Docker host against the CIS Docker Benchmark, validating daemon configuration, host settings, container configuration, logging, and other security best practices.

---

### 12. What is Seccomp?

**Answer:** Seccomp restricts the Linux system calls a process can make. Docker ships with a default Seccomp profile that blocks many high-risk syscalls while allowing those commonly required by applications.

---

### 13. What is the difference between Seccomp and AppArmor?

**Answer:** Seccomp controls **which system calls** a process may execute. AppArmor controls **what resources** (files, networking, capabilities, etc.) a process may access. They protect different layers and are commonly used together.

---

### 14. What is SELinux?

**Answer:** SELinux is a Mandatory Access Control (MAC) framework that uses security labels and policies to determine whether a process can access a resource, even if normal UNIX permissions would allow it.

---

### 15. How would you secure a production container from the start?

**Answer:** I would:

* Use a trusted minimal or distroless base image.
* Run as a non-root user with the `USER` instruction.
* Drop all unnecessary Linux capabilities.
* Enable a read-only root filesystem and mount writable volumes only where needed.
* Store secrets in a dedicated secret management solution.
* Scan images with tools like Trivy or Docker Scout in CI/CD.
* Apply the default or a custom Seccomp profile.
* Enforce AppArmor or SELinux policies where supported.
* Regularly audit the Docker host with Docker Bench Security and keep images updated with the latest security patches.

These concepts are central to production container hardening and are frequently discussed in mid-level DevOps and Kubernetes interviews because they demonstrate an understanding of both Linux security fundamentals and practical operational security.
