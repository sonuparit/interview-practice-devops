# Why does APT need GPG keys?

APT downloads packages from repositories over the Internet.

Imagine this:

```
Your Server
     |
     |  apt install nginx
     |
Internet
     |
Repository
```

How does Ubuntu know that:

* the repository is really Ubuntu's?
* nobody modified the package?
* nobody inserted malware?

The answer is:

**GPG signatures.**

APT doesn't trust repositories blindly.

Every repository signs its metadata using a **private GPG key**.

Your machine stores the corresponding **public key**.

APT verifies:

> "Was this metadata signed by the trusted owner?"

If yes:

```
Install packages
```

If no:

```
STOP
```

---

# What is a GPG key?

GPG uses **public-key cryptography**.

There are two keys.

```
Repository

Private Key
      |
      | Signs repository metadata
      V

Internet

      ^
      |
Public Key
stored on your server
```

The private key never leaves Ubuntu (or Docker, HashiCorp, Kubernetes, etc.)

Only the public key is shared.

---

# Why not sign every package?

Actually, APT mainly verifies the repository metadata.

Example:

```
Packages.gz
Release
InRelease
```

These contain:

* package names
* versions
* SHA256 hashes

Example:

```
nginx_1.28.deb

SHA256:
abc123...
```

Ubuntu signs this Release file.

APT verifies:

1. Signature
2. Hashes

Then checks the downloaded package hash.

So the package is indirectly trusted.

---

# What is NO_PUBKEY?

Suppose Ubuntu signs its repository.

Your machine receives:

```
Release
Signature
```

APT now asks:

> "Do I have Ubuntu's public key?"

If yes

```
Verify signature
```

If no

```
NO_PUBKEY
```

Example:

```
The following signatures couldn't be verified because the public key is not available:

NO_PUBKEY 871920D1991BC93C
```

That long hexadecimal number is the missing key ID.

Meaning:

> "I don't have the public key needed to verify this repository."

---

# Why does NO_PUBKEY happen?

Usually because:

### New repository

Example:

```
Docker Repository
HashiCorp Repository
Cloudflare Repository
```

You added:

```
deb https://download.docker.com/linux/ubuntu
```

But never installed Docker's public key.

---

### Ubuntu rotated its signing key

Old key:

```
AAA111
```

New key:

```
BBB222
```

Your machine only trusts the old one.

---

### Wrong repository

Example:

Ubuntu 24 repository on Ubuntu 20.

Keys don't match.

---

### Repository configuration mistake

You copied a random repository.

Forgot to install its key.

---

# What attack does GPG prevent?

This is extremely important.

Imagine there is **no signature**.

```
You

     |
     | apt install nginx
     |

Internet
```

A hacker intercepts traffic.

Instead of:

```
nginx
```

He sends:

```
evil-nginx
```

Without signatures...

APT would happily install malware.

---

GPG prevents:

```
Man-in-the-middle attack
```

Example:

```
Server
      |
      |
Attacker
      |
      |
Repository
```

Attacker modifies:

```
Release
Packages
```

But...

He **cannot produce a valid signature**, because he doesn't have Ubuntu's private key.

APT immediately detects it.

---

Another attack:

Repository compromise.

Someone uploads

```
nginx.deb
```

with malware.

But cannot sign Release.

APT rejects it.

---

Another attack:

Changing package versions.

Example:

```
openssl
```

becomes

```
old vulnerable version
```

Signature becomes invalid.

Rejected.

---

# How is it verified?

The process looks like this.

```
Repository

Release

SHA256:
nginx.deb

Signed
        |
        |
Internet
        |
        V

APT
```

APT performs:

Step 1

Download

```
InRelease
```

or

```
Release
Release.gpg
```

---

Step 2

Find public key

```
/etc/apt/keyrings/
```

or

```
trusted.gpg.d
```

---

Step 3

Verify signature.

```
Good signature?
```

If yes

Continue.

If no

Stop.

---

Step 4

Download package.

---

Step 5

Calculate SHA256.

Compare with Release file.

If hashes match

Install.

Else

Reject.

---

# Does every dependency have its own GPG key?

No.

This is a common misconception.

Example:

```
Docker repository

containerd
docker-ce
docker-cli
buildx
compose
```

All are trusted because:

```
Repository
        |
Signed once
```

The repository owns one signing key.

Not every package.

---

Same for Ubuntu.

```
vim
curl
nginx
gcc
python
git
```

They do **not** have separate keys.

Ubuntu repository signs the repository metadata.

---

# Where can I check installed GPG keys?

Modern Ubuntu stores keys here.

```
/etc/apt/keyrings/
```

Example:

```
docker.gpg
hashicorp.gpg
cloudflare.gpg
```

---

Older systems:

```
/etc/apt/trusted.gpg.d/
```

---

List them

```bash
ls /etc/apt/keyrings
```

or

```bash
ls /etc/apt/trusted.gpg.d
```

---

You can inspect a key:

```bash
gpg --show-keys /etc/apt/keyrings/docker.gpg
```

Example output:

```
pub rsa4096

Docker Release Signing Key

Fingerprint:
9DC8 5822 ...
```

---

# How can I know if a key is missing?

Simply run:

```bash
sudo apt update
```

If missing:

```
NO_PUBKEY 7EA0A9C3F273FCD8
```

APT tells you exactly which key is absent.

---

# How do we add a GPG key?

Modern method:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Then reference it in the repository configuration:

```text
deb [signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
stable
```

Now only that repository uses that specific key.

This is more secure than trusting the key globally.

---

# Why not use `apt-key`?

Older Ubuntu versions used:

```bash
apt-key add key.gpg
```

This added the key to the global trust store.

That meant **any trusted key could authenticate any repository**, which is a broader trust model than necessary.

Modern APT recommends storing repository-specific keys in `/etc/apt/keyrings/` and using the `signed-by=` option to limit trust to the intended repository.

---

# How do we remove a key?

Delete the key file.

Example:

```bash
sudo rm /etc/apt/keyrings/docker.gpg
```

or

```bash
sudo rm /etc/apt/trusted.gpg.d/docker.gpg
```

After removal:

```
apt update

NO_PUBKEY
```

will appear for repositories that depended on that key.

---

# When do we add or remove keys?

## Add

* Installing Docker
* Installing HashiCorp tools
* Installing Kubernetes packages
* Installing Cloudflare WARP
* Adding any third-party APT repository

## Remove

* Repository is no longer used
* Key has expired and is replaced
* Repository is compromised
* Security cleanup of unused trusted keys

---

# Real DevOps example

Imagine you're writing a Dockerfile:

```dockerfile
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
 | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

RUN echo \
"deb [signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu noble stable" \
> /etc/apt/sources.list.d/docker.list

RUN apt update
RUN apt install docker-ce
```

Without the key:

```
apt update

NO_PUBKEY
```

The build fails.

---

# Why is this knowledge important in DevOps?

Modern DevOps engineers routinely automate the installation of software from external repositories. Understanding APT's trust model helps you:

* **Build reliable automation:** Fix `NO_PUBKEY` and repository signature errors in CI/CD pipelines, Docker images, and cloud-init scripts.
* **Strengthen supply chain security:** Ensure only trusted repositories are used and avoid disabling signature verification.
* **Manage infrastructure safely:** Rotate, add, or remove repository keys as part of system maintenance and security updates.
* **Troubleshoot production issues:** Diagnose why `apt update` or automated deployments suddenly fail after a repository key change.
* **Follow security best practices:** Use repository-specific keyrings with `signed-by=` instead of the deprecated global `apt-key` approach.

Recruiters value this knowledge because a mid-level DevOps engineer is expected to understand **why** package installation is secure, not just how to run `apt install`. Being able to explain GPG verification, repository trust, and signature failures demonstrates competence in Linux administration and secure infrastructure management.
