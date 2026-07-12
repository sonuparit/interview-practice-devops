This is one of the most important topics for a Mid-Level DevOps Engineer.

Many candidates know how to **deploy applications**, but companies pay mid-level engineers because they know **how to secure production infrastructure without breaking it**.

A production DevOps engineer constantly asks:

> **"Can I make this service work while exposing the minimum possible attack surface?"**

---

# 1. SSH Hardening

## 1. What is this?

SSH Hardening means making the SSH server more secure so attackers cannot easily gain access.

Instead of using SSH's default configuration, you modify it to reduce attack possibilities.

Common hardening steps:

* Disable root login
* Disable password authentication
* Use SSH keys
* Change idle timeout
* Restrict allowed users/groups
* Disable unused authentication methods
* Use modern ciphers
* Limit login attempts

Main configuration file:

```
/etc/ssh/sshd_config
```

---

## 2. Why is it required?

SSH is usually exposed to the Internet.

That makes it one of the biggest attack targets.

Without hardening:

* brute force attacks
* password guessing
* stolen passwords
* root compromise

become much easier.

---

## 3. DevOps benefit

Almost every Linux server is managed through SSH.

Examples:

Deploying

```
GitHub Actions
        ↓
SSH
        ↓
EC2
```

or

```
Ansible
        ↓
SSH
        ↓
100 servers
```

If SSH is compromised,

everything is compromised.

---

## 4. Why recruiters care?

They want engineers who don't deploy insecure servers.

They expect you to know:

* key authentication
* ssh-agent
* authorized_keys
* ssh-copy-id
* sshd_config

---

## 5. Production example

Company launches 500 EC2 instances.

By mistake:

```
PermitRootLogin yes
PasswordAuthentication yes
```

Bots immediately begin trying millions of passwords.

You harden SSH.

Attack surface drops dramatically.

---

## 6. Troubleshooting

SSH suddenly stops working.

You investigate:

```
systemctl status ssh

journalctl -u ssh

sshd -t
```

Maybe:

```
AllowUsers

```

is wrong.

Or

```
PasswordAuthentication no
```

while users don't have SSH keys.

---

## 7. Dependencies

Depends on

* sshd service
* firewall
* user accounts
* SSH keys
* PAM
* network

---

## 8. Analogy

Imagine your house.

Weak SSH:

Front door unlocked.

Strong SSH:

* fingerprint scanner
* security guard
* camera
* visitor whitelist

---

## 9. Must answer

1.

Why should root login be disabled?

2.

Why are SSH keys safer than passwords?

3.

What happens if PasswordAuthentication is disabled?

---

# 2. File Permissions

(You've already learned chmod, chown, umask, SUID, SGID, Sticky Bit.)

From a security perspective:

Permissions enforce **least privilege**.

Example:

Application only needs read access.

Don't give write permission.

---

### Production

Suppose:

```
/etc/shadow

```

becomes world readable.

Every user can read password hashes.

Huge security issue.

---

### Troubleshooting

Application says:

```
Permission denied
```

Check:

```
ls -l

namei -l file

stat file
```

---

### Dependencies

* filesystem
* users
* groups
* ACLs
* SELinux/AppArmor

---

### Analogy

Office filing cabinet.

Only HR has the key.

Everyone else can't open it.

---

### Must answer

* Difference between owner/group/others
* Difference between chmod and chown
* Why 777 is dangerous

---

# 3. sudo

## What?

Allows users to execute commands as another user (usually root).

Configuration:

```
/etc/sudoers
```

or

```
/etc/sudoers.d/
```

---

## Why?

Never log in as root.

Instead:

```
sudo apt update
```

---

## DevOps usage

CI/CD

Automation

Scripts

Deployment

---

## Production

Developer needs to restart nginx.

Don't give root.

Instead:

```
sudo systemctl restart nginx
```

Only that command.

---

## Troubleshooting

```
sudo -l
```

Shows allowed commands.

Logs:

```
journalctl

/var/log/auth.log
```

---

## Dependencies

* PAM
* users
* groups
* systemd
* authentication

---

## Analogy

A hotel master key.

Staff can open only rooms they are authorized for.

---

## Must answer

* Why use sudo instead of root?
* What is sudoers?
* How to allow only one command?

---

# 4. Fail2Ban (High Level)

## What?

Monitors logs.

Blocks IPs showing malicious behavior.

Example:

Repeated SSH failures.

Automatically:

```
iptables

```

or

```
firewalld

```

rule added.

---

## Why?

Stops brute-force attacks.

---

## DevOps usage

Protects:

SSH

FTP

Apache

Nginx

Mail

---

## Production

Bot:

```
5000 failed SSH attempts
```

Fail2Ban bans IP.

---

## Troubleshooting

```
fail2ban-client status

fail2ban-client status sshd
```

---

## Dependencies

* logs
* sshd
* firewall
* regex filters

---

## Analogy

Security guard.

Three failed attempts.

You're kicked out.

---

## Must answer

* Does Fail2Ban replace firewall?
* How does it detect attacks?
* What happens after repeated failures?

---

# 5. firewalld

## What?

Dynamic firewall manager.

Uses zones.

Backend:

iptables/nftables

---

## Why?

Easier firewall management.

---

## DevOps

Production servers often use:

```
firewall-cmd
```

instead of raw iptables.

---

## Troubleshooting

```
firewall-cmd --list-all
```

---

## Dependencies

Kernel firewall

systemd

network interfaces

---

## Analogy

Airport security with different gates.

Public gate.

Internal gate.

Management gate.

---

## Must answer

* What are zones?
* Difference between runtime and permanent?
* Why reload?

---

# 6. UFW

## What?

Uncomplicated Firewall.

Simplified firewall.

Mostly Ubuntu.

---

## Commands

```
ufw enable

ufw allow 22

ufw deny 80

ufw status
```

---

## Production

Quick firewall configuration.

---

## Troubleshooting

```
ufw status verbose
```

---

## Dependencies

iptables/nftables

---

## Analogy

Simple remote control.

Instead of manually controlling electrical circuits.

---

## Must answer

* Why use UFW?
* Difference from iptables?
* How to allow SSH?

---

# 7. iptables (Basics)

## What?

Linux kernel firewall.

Processes packets.

Rules:

```
INPUT

OUTPUT

FORWARD
```

---

## Why?

Controls network traffic.

---

## Production

Allow:

443

Block:

23

Drop:

malicious IP

---

## Troubleshooting

```
iptables -L

iptables-save
```

---

## Dependencies

Kernel networking stack

Netfilter

---

## Analogy

Security checkpoint.

Every packet is inspected.

---

## Must answer

* INPUT vs OUTPUT vs FORWARD
* ACCEPT vs DROP
* Rule order importance

---

# 8. SELinux (Basics)

## What?

Mandatory Access Control (MAC).

Even root can be restricted.

---

Unlike permissions:

Permissions say

> User owns file.

SELinux asks

> Is this process allowed to access this type of file?

---

## Why?

Limits damage after compromise.

---

## Production

Apache compromised.

SELinux prevents Apache reading

```
/home

```

even if permissions allow.

---

## Troubleshooting

Common symptom:

```
Permission denied
```

Permissions look correct.

Check:

```
getenforce

sestatus

ausearch

restorecon
```

---

## Dependencies

Filesystem labels

Kernel

Policies

---

## Analogy

Employee has office key.

Security guard still checks whether they may enter that room.

---

## Must answer

* Difference from chmod?
* Modes?
* Why disable only as last resort?

---

# 9. AppArmor (Basics)

## What?

Another MAC system.

Profile-based.

Mostly Ubuntu.

---

## Why?

Restricts applications.

---

## Production

Compromised process cannot access random files.

---

## Troubleshooting

```
aa-status
```

Check profile.

---

## Dependencies

Kernel

Profiles

Filesystem

---

## Analogy

Each employee receives a list:

"You may enter rooms A, B, C only."

---

## Must answer

* Difference from SELinux?
* What is a profile?
* Why use AppArmor?

---

# firewalld vs UFW vs iptables

| Feature  | iptables             | firewalld          | UFW                   |
| -------- | -------------------- | ------------------ | --------------------- |
| Level    | Low                  | Medium             | High                  |
| Ease     | Difficult            | Easier             | Easiest               |
| Used on  | All Linux            | RHEL/CentOS        | Ubuntu                |
| Backend  | Netfilter            | iptables/nftables  | iptables/nftables     |
| Best For | Fine-grained control | Enterprise servers | Simple administration |

---

# SELinux vs AppArmor

| SELinux                | AppArmor         |
| ---------------------- | ---------------- |
| Label-based            | Path-based       |
| More powerful          | Easier           |
| Common on RHEL         | Common on Ubuntu |
| Steeper learning curve | Simpler profiles |

---

# Overall Production Flow

```
Internet
    │
    ▼
Firewall (iptables/UFW/firewalld)
    │
    ▼
SSH Hardening
    │
    ▼
Fail2Ban blocks attackers
    │
    ▼
User logs in
    │
    ▼
sudo provides least privilege
    │
    ▼
File permissions protect files
    │
    ▼
SELinux/AppArmor restrict process behavior
    │
    ▼
Application
```

# 15 Important Mid-Level DevOps Interview Questions (with Answers)

### 1. Why are SSH keys preferred over passwords?

**Answer:** SSH keys use asymmetric cryptography and are far more resistant to brute-force attacks than passwords. They also support secure automation for tools like Ansible and CI/CD pipelines.

---

### 2. Why should `PermitRootLogin` usually be disabled?

**Answer:** Logging in directly as `root` provides attackers with a known high-privilege target and removes accountability. It's safer to log in as a regular user and elevate privileges with `sudo`.

---

### 3. What is the Principle of Least Privilege?

**Answer:** Every user, process, and service should have only the minimum permissions required to perform its task. This limits the impact of mistakes or compromises.

---

### 4. Why is using `chmod 777` on production files considered dangerous?

**Answer:** It grants read, write, and execute permissions to everyone, allowing unauthorized users or compromised processes to modify or execute files, creating a significant security risk.

---

### 5. What is the difference between authentication and authorization?

**Answer:** Authentication verifies **who** you are (for example, via an SSH key or password). Authorization determines **what** you're allowed to do (for example, through `sudo` rules or file permissions).

---

### 6. How does `sudo` improve security compared to logging in as `root`?

**Answer:** `sudo` provides controlled privilege escalation, supports command-level restrictions, and records actions in logs for auditing and accountability.

---

### 7. How does Fail2Ban protect a Linux server?

**Answer:** It monitors log files for repeated authentication failures or other suspicious patterns and temporarily blocks the offending IP address by adding firewall rules.

---

### 8. What is the difference between `iptables`, `firewalld`, and `ufw`?

**Answer:** `iptables` provides low-level packet filtering. `firewalld` is a dynamic firewall manager commonly used on RHEL-based systems. `ufw` is a simplified firewall interface commonly used on Ubuntu.

---

### 9. What is the difference between `DROP` and `REJECT` in firewall rules?

**Answer:** `DROP` silently discards packets without responding. `REJECT` actively informs the sender that the connection was refused. `DROP` reveals less information to potential attackers.

---

### 10. Explain the `INPUT`, `OUTPUT`, and `FORWARD` chains in `iptables`.

**Answer:** `INPUT` filters traffic destined for the local machine, `OUTPUT` filters traffic generated by the local machine, and `FORWARD` filters traffic passing through the machine between different networks.

---

### 11. What problem does SELinux solve?

**Answer:** SELinux enforces mandatory access control policies, restricting what processes can access even if traditional Linux permissions would otherwise allow it. This limits the impact of a compromised application.

---

### 12. What is the difference between traditional Linux file permissions and SELinux?

**Answer:** File permissions control access based on user, group, and other ownership. SELinux adds an additional policy layer based on security contexts, allowing or denying access regardless of ownership.

---

### 13. Why might an application receive "Permission denied" even when file permissions look correct?

**Answer:** Security frameworks such as SELinux or AppArmor may be denying access based on their policies. Logs and security status should be checked before changing permissions.

---

### 14. A web server is reachable locally but not from another machine. What would you check?

**Answer:** Verify that the service is listening (`ss -tulnp`), confirm firewall rules (`ufw`, `firewalld`, or `iptables`), ensure the application is bound to the correct interface (not just `127.0.0.1`), and check any cloud security groups or network ACLs.

---

### 15. How do these security layers work together in production?

**Answer:** Firewalls control network access, SSH hardening secures remote administration, Fail2Ban blocks abusive clients, `sudo` enforces controlled privilege escalation, file permissions protect resources, and SELinux or AppArmor constrain process behavior. Together, they implement defense in depth, so if one layer is bypassed, additional layers continue to protect the system.
