# 23. Package Management (Mid-Level DevOps Interview)

Package management is one of the most important Linux skills for a DevOps Engineer.

Almost every production server requires you to:

* Install software
* Update software
* Remove software
* Verify software authenticity
* Resolve dependency issues
* Configure repositories
* Automate package installation

If package management is broken, deployments stop.

---

# 1. apt

## What is this?

**APT (Advanced Package Tool)** is the package manager used by Debian and Ubuntu.

It manages software installation, upgrades, removal, searching and dependency resolution.

Think of apt as a **software management system**, not just an installer.

It works on top of another tool called **dpkg**.

```
User
   │
   ▼
 apt
   │
 resolves dependencies
   │
   ▼
 dpkg
   │
 installs .deb packages
```

Common commands

```bash
sudo apt update

sudo apt install nginx

sudo apt remove nginx

sudo apt purge nginx

sudo apt autoremove

sudo apt upgrade

sudo apt full-upgrade

apt search nginx

apt show nginx
```

---

## Why is it required?

Imagine nginx needs

* OpenSSL
* libc
* PCRE
* systemd files

Installing all manually would be impossible.

APT automatically:

* downloads packages
* downloads dependencies
* verifies signatures
* installs in correct order

---

## How does it help in Mid-Level DevOps?

Daily tasks include

* Install Docker
* Install Kubernetes tools
* Upgrade OS
* Install monitoring agents
* Install CI/CD runners
* Security patching

Example

```bash
sudo apt install docker.io
```

Instead of manually downloading dozens of libraries.

---

## Why recruiters care?

Because production Linux servers constantly need

* patches
* upgrades
* package rollback
* automation

Recruiters expect you to know

* apt update
* apt upgrade
* apt-cache
* package pinning
* repository management

---

## Production Problem Example

Production server has OpenSSL vulnerability.

Security team says

"Upgrade OpenSSL immediately."

You run

```bash
sudo apt update
sudo apt install openssl
```

or

```bash
sudo apt upgrade
```

without reinstalling the OS.

---

## Troubleshooting Example

Problem

```
E: Unable to locate package docker-ce
```

Possible causes

* repository missing
* apt update not run
* wrong repository
* spelling mistake

Steps

```bash
apt update

apt search docker

cat /etc/apt/sources.list

ls /etc/apt/sources.list.d/
```

---

## Services affected

Everything.

Examples

* nginx
* Docker
* Kubernetes
* PostgreSQL
* Jenkins
* Grafana
* Prometheus

Because nearly all Linux software is distributed as packages.

---

## Analogy

APT is like Amazon.

You order

"Install nginx"

Amazon automatically sends

* nginx
* screws
* manuals
* accessories

instead of asking you to order every part separately.

---

## Test Yourself

Can you answer?

1. Difference between apt and dpkg?
2. Why do we run `apt update` before `apt install`?
3. Difference between remove, purge and autoremove?

---

# 2. Repositories

## What is this?

A repository is a server that stores software packages.

Example

Ubuntu repository

```
nginx
docker
git
curl
python
```

APT downloads packages from repositories.

---

## Why required?

Without repositories,

every user would need to download software manually.

Repositories provide

* latest versions
* security updates
* dependency metadata
* verified packages

---

## Mid-Level DevOps Usage

Very common.

Examples

Adding Docker repository

```bash
echo "deb ..." > docker.list
```

Installing Kubernetes repository

Installing HashiCorp repository

Installing GitHub CLI repository

---

## Recruiters care because

Companies rarely use only Ubuntu repositories.

Production often includes

* Docker repo
* Hashicorp repo
* Grafana repo
* Elastic repo
* Kubernetes repo

You must know how to configure them.

---

## Production Problem

Need Docker 28.

Ubuntu repository has Docker 24.

Solution

Add Docker Official Repository.

---

## Troubleshooting

Problem

```
Package not found
```

Check

```bash
cat /etc/apt/sources.list

ls /etc/apt/sources.list.d

apt update
```

---

## Services affected

Every package downloaded from repository.

---

## Analogy

Repository = App Store.

APT = Play Store app.

Packages = Apps.

---

## Test Yourself

1. What is a repository?
2. Why add third-party repositories?
3. Where are repository files stored?

---

# 3. GPG Keys

## What is this?

Every repository signs its packages using a **GPG (GNU Privacy Guard)** key.

Before installation,

APT verifies

```
Repository

Signs Package

↓

APT verifies signature

↓

If valid → install

If invalid → reject
```

---

## Why required?

Prevents

* tampered packages
* malware
* fake repositories
* man-in-the-middle attacks

Without signatures,

attackers could replace packages.

---

## Mid-Level DevOps Usage

You'll often add repository keys.

Example

Docker

Hashicorp

Grafana

Cloudflare

GitHub CLI

You actually experienced this.

Remember:

```
NO_PUBKEY
```

That happened because the repository key was missing.

---

## Recruiters care

Supply chain attacks are increasing.

Companies expect engineers to know

* repository trust
* key verification
* signed packages

---

## Production Problem

```
NO_PUBKEY XXXXX
```

APT refuses updates.

Production deployment stops.

Fix

Import the correct public key.

---

## Troubleshooting

Error

```
The following signatures couldn't be verified
```

Check

```bash
apt update
```

Look for

```
NO_PUBKEY
```

Verify key installation.

---

## Services affected

Every package installation.

Without trusted keys,

APT refuses installation.

---

## Analogy

Restaurant analogy.

You receive medicine.

Would you trust

* sealed medicine
* or opened medicine?

GPG seal proves authenticity.

---

## Test Yourself

1. Why does APT need GPG keys?
2. What is NO_PUBKEY?
3. What attack does GPG prevent?

---

# 4. Package Dependencies

## What is this?

Dependencies are packages another package needs.

Example

Docker requires

```
containerd

iptables

libseccomp

system libraries
```

Installing Docker automatically installs them.

---

## Why required?

Software shares libraries.

Instead of shipping every library,

packages reuse existing ones.

---

## Mid-Level DevOps Usage

Very important.

Installing

* Jenkins
* Docker
* Grafana
* Kubernetes

all involve dependency resolution.

---

## Recruiters care

Dependency problems are extremely common.

Example

Broken upgrade.

Version conflict.

Held packages.

---

## Production Problem

Installing package fails

```
Depends:
libssl3

but it isn't installable
```

Need dependency troubleshooting.

---

## Troubleshooting

Commands

```bash
apt --fix-broken install

apt-cache depends nginx

apt-cache rdepends nginx

dpkg -l

apt policy
```

---

## Services affected

Almost every service.

One missing library

↓

Service fails to start.

---

## Analogy

Making tea.

Need

* tea
* water
* milk
* sugar

Tea depends on all of them.

Missing milk

↓

Tea incomplete.

---

## Test Yourself

1. What is a dependency?
2. Why do dependency conflicts happen?
3. How do you fix broken dependencies?

---

# Relationships Between All Topics

```
                    User
                      │
                      ▼
                    apt
                      │
                      ▼
             Repository List
                      │
                      ▼
             Download Packages
                      │
                      ▼
             Verify GPG Signature
                      │
              Signature Valid?
               │            │
             Yes           No
               │            │
               ▼            ▼
      Resolve Dependencies  Reject Package
               │
               ▼
          Install Software
```

---

# Production Troubleshooting Flow

```
Package install failed

        │

Is internet working?

        │

Run apt update

        │

Repository exists?

        │

GPG key valid?

        │

Dependency satisfied?

        │

Enough disk space?

        │

Broken package database?

        │

Service starts?
```

Useful commands:

```bash
apt update

apt install

apt search

apt show

apt policy

apt-cache depends

apt-cache rdepends

apt --fix-broken install

dpkg -l

dpkg -i package.deb

journalctl -xe
```

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What is the difference between `apt` and `dpkg`?

**Answer:** `dpkg` installs and manages local `.deb` packages but does not resolve dependencies. `apt` is a higher-level tool that downloads packages from repositories, resolves dependencies, verifies signatures, and then uses `dpkg` to perform the installation.

---

### 2. Why do we run `apt update` before `apt install`?

**Answer:** `apt update` refreshes the local package index from configured repositories. Without it, the system may not know about the latest package versions or newly added packages.

---

### 3. What is a repository?

**Answer:** A repository is a server that hosts software packages and metadata. APT downloads packages and dependency information from these repositories.

---

### 4. Where are APT repository configurations stored?

**Answer:** The main repository list is in `/etc/apt/sources.list`, while additional repository files are typically stored in `/etc/apt/sources.list.d/`.

---

### 5. What is the purpose of GPG keys in APT?

**Answer:** GPG keys verify that packages and repository metadata are signed by a trusted source, protecting against tampering and supply-chain attacks.

---

### 6. What does the `NO_PUBKEY` error mean?

**Answer:** It means APT cannot verify a repository's signature because the required public GPG key is missing from the system's trusted keyring.

---

### 7. What are package dependencies?

**Answer:** Dependencies are other packages or libraries required for a package to install and function correctly. APT automatically installs required dependencies when possible.

---

### 8. How do you fix broken package dependencies?

**Answer:** Common steps include:

```bash
sudo apt --fix-broken install
sudo dpkg --configure -a
sudo apt update
```

Then retry the installation and inspect any remaining dependency conflicts.

---

### 9. What is the difference between `apt remove` and `apt purge`?

**Answer:** `apt remove` removes the package but leaves configuration files. `apt purge` removes both the package and its configuration files.

---

### 10. What does `apt autoremove` do?

**Answer:** It removes packages that were automatically installed as dependencies but are no longer required by any installed package.

---

### 11. How would you troubleshoot "Unable to locate package"?

**Answer:** Check that:

* `apt update` has been run.
* The correct repository is configured.
* The package name is correct.
* The repository supports your OS version and architecture.

---

### 12. How do you see which repository will provide a package?

**Answer:**

```bash
apt policy nginx
```

This shows the installed version, candidate version, and the repositories from which the package is available.

---

### 13. How can you inspect package dependencies?

**Answer:**

```bash
apt-cache depends nginx
```

To see reverse dependencies (what depends on a package):

```bash
apt-cache rdepends nginx
```

---

### 14. A package installs successfully but the service won't start. What would you do?

**Answer:**

1. Check service status:

   ```bash
   systemctl status <service>
   ```
2. View logs:

   ```bash
   journalctl -u <service>
   ```
3. Verify configuration files.
4. Confirm all runtime dependencies are installed and compatible.

---

### 15. Why is package management important in automated DevOps pipelines?

**Answer:** Infrastructure automation relies on reproducible software installation. Tools like Ansible, Terraform (via provisioners), cloud-init, and CI/CD pipelines use package managers to install exact software versions consistently, securely, and with proper dependency handling across environments.

---

## Mid-Level DevOps Tip

As a Mid-level DevOps Engineer, go beyond memorizing commands. Be able to explain the entire package installation lifecycle:

> **APT reads the configured repositories, downloads the latest package metadata (`apt update`), verifies repository signatures using trusted GPG keys, resolves all required dependencies, downloads the necessary packages, and then invokes `dpkg` to install them in the correct order. If installation fails, I troubleshoot by checking repository configuration, GPG key validity, dependency conflicts, package policies, service logs, and the package database state.**

If you can explain that flow confidently in an interview, you'll demonstrate a practical understanding rather than just familiarity with commands.
