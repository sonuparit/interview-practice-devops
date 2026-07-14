# 3. Docker Images (Mid-Level DevOps Interview)

This is one of the **most frequently asked interview topics** because **everything in Docker starts with an image**.

If containers are the **running processes**, images are the **blueprints** used to create them.

---

# 1. Image

## What is this?

A Docker image is a **read-only template** containing everything needed to run an application.

It includes

* Application code
* Runtime (Python, Java, Node)
* Libraries
* Dependencies
* Environment configuration
* Filesystem

Think of it as a packaged operating environment.

Example:

```
ubuntu:24.04
nginx:1.27
python:3.12
```

These are all Docker images.

---

## Why is it required?

Without images, every server would require manual installation.

Instead of

```
Install Ubuntu
Install Python
Install pip
Install Flask
Copy source code
Configure everything
```

You simply run

```bash
docker run my-app
```

Everything is already inside.

---

## Why recruiters care?

They want engineers who understand

* reproducible deployments
* image optimization
* versioning
* security
* CI/CD pipelines

Every Kubernetes deployment eventually pulls an image.

---

## Production problem it solves

Without images

```
Developer machine
Python 3.10

Production
Python 3.12

QA
Python 3.11
```

Everything behaves differently.

Images solve

> "Works on my machine."

---

## Troubleshooting example

Application crashes only in production.

Check

```bash
docker inspect container
```

Which image?

```bash
docker images
```

Maybe production is using

```
my-app:v1.0
```

while testing used

```
my-app:v1.2
```

Problem found.

---

## Analogy

A Docker image is like a **cake recipe frozen in time**.

Every cake baked from it should be identical.

Container = baked cake.

---

## Test yourself

1. Can an image change after being built?
2. Why are images read-only?
3. Why do containers come from images?

---

# 2. Layer

---

## What is this?

Docker images consist of multiple filesystem layers.

Example Dockerfile

```dockerfile
FROM ubuntu

RUN apt install nginx

COPY . /app

CMD ["nginx"]
```

Creates something like

```
Layer 1
Ubuntu

Layer 2
Install nginx

Layer 3
Copy application

Layer 4
Metadata
```

Every instruction creates a new layer (except a few metadata instructions).

---

## Why required?

Layers allow

* caching
* sharing
* faster builds
* smaller downloads

---

Example

100 images all use Ubuntu.

Docker stores Ubuntu only once.

Huge storage savings.

---

## Why recruiters care?

Layer ordering directly affects

* build speed
* CI pipeline duration
* registry bandwidth
* image size

---

## Production problem solved

Imagine

```
RUN apt install python
RUN apt install git
RUN apt install curl
```

Three layers.

Instead

```
RUN apt update && \
    apt install python git curl
```

One layer.

Smaller image.

---

## Troubleshooting

Image became

```
6 GB
```

Investigate

```bash
docker history image
```

You'll see

```
Layer 3
+3.8GB
```

Now you know where the problem is.

---

## Analogy

Imagine Photoshop.

Every edit is another layer.

Docker works similarly.

---

## Test yourself

1. Why do Docker layers exist?
2. Which command shows image layers?
3. Why should Dockerfiles minimize unnecessary layers?

---

# 3. Base Image

---

## What is this?

The starting image in Dockerfile.

Example

```dockerfile
FROM ubuntu
```

Ubuntu is the base image.

---

## Why required?

Every application needs some filesystem.

Without it,

there is nowhere to install software.

---

## Why recruiters care?

Choosing a bad base image causes

* vulnerabilities
* huge image size
* slower deployments

---

## Production problem

Instead of

```
ubuntu
```

Maybe use

```
alpine
```

or

```
distroless
```

Image becomes

```
900MB

↓

70MB
```

Huge improvement.

---

## Troubleshooting

Security scanner reports

```
250 vulnerabilities
```

Cause

Old Ubuntu base image.

Update

```dockerfile
FROM ubuntu:24.04
```

---

## Analogy

Building a house.

Base image = foundation.

---

## Test yourself

1. What is the first line in most Dockerfiles?
2. Why does choosing the right base image matter?
3. Can two applications share the same base image?

---

# 4. Parent Image

---

## What is this?

The image from which another image inherits.

Example

```
ubuntu
        ↓
python:3.12
        ↓
my-company-python
        ↓
inventory-service
```

Every image has a parent except `scratch`.

---

## Why required?

Avoid rebuilding common software.

Inheritance.

---

## Recruiters care because

Large organizations build

```
company-base-image

↓

all internal applications
```

Consistency.

---

## Production problem

Company updates OpenSSL.

Only update

```
company-base-image
```

Rebuild all applications.

Done.

---

## Troubleshooting

Application vulnerable.

Check parent image.

Maybe it inherited vulnerable packages.

---

## Analogy

Family tree.

Children inherit traits.

Images inherit layers.

---

## Test yourself

1. What's the difference between base and parent image?
2. Why do companies create custom parent images?
3. How does inheritance reduce duplication?

---

# 5. Scratch Image

---

## What is this?

`scratch` is an **empty image**.

Literally nothing inside.

No Linux.

No shell.

No libraries.

Example

```dockerfile
FROM scratch
```

---

## Why required?

For statically compiled applications.

Example

Go

Rust

C

Tiny containers.

---

## Recruiters care because

Very secure.

Tiny attack surface.

Common in production.

---

## Production problem

Normal Ubuntu image

```
220MB
```

Scratch

```
12MB
```

Huge savings.

---

## Troubleshooting

Container starts then

```
bash not found
```

Of course.

Scratch has no shell.

---

## Analogy

Empty USB drive.

Nothing exists until you copy something.

---

## Test yourself

1. Does scratch contain Linux?
2. Why are scratch images secure?
3. Which languages commonly use scratch?

---

# 6. Immutable Images

---

## What is this?

Once built,

an image never changes.

If something changes,

create a new image.

Never edit existing ones.

---

## Why required?

Guarantees

```
Development

QA

Production
```

all use identical images.

---

## Recruiters care

Immutable infrastructure is a DevOps principle.

---

## Production problem

Bad practice

```
docker exec

apt install vim
```

Container works.

Tomorrow it disappears.

Correct

Rebuild image.

---

## Troubleshooting

Someone says

"I installed package manually."

Impossible to reproduce.

Immutable images avoid this.

---

## Analogy

PDF document.

You don't edit it.

You generate a new version.

---

## Test yourself

1. Why shouldn't production containers be modified?
2. Why rebuild instead of patching?
3. What is immutable infrastructure?

---

# 7. Image Manifest

---

## What is this?

JSON metadata describing an image.

Contains

* layers
* architecture
* OS
* config
* digest

Registry sends manifest before downloading layers.

---

## Why required?

Docker must know

"What files make this image?"

---

## Recruiters care

Multi-platform builds rely on manifests.

---

## Production problem

Mac

Linux

ARM

AMD64

Same image tag.

Manifest decides which image to download.

---

## Troubleshooting

Image won't pull on ARM.

Inspect manifest.

Wrong architecture.

---

## Analogy

Book index.

Before reading,

you know what's inside.

---

## Test yourself

1. What information does manifest contain?
2. Why is it downloaded first?
3. How does multi-architecture work?

---

# 8. Image Digest

---

## What is this?

A SHA256 hash uniquely identifying an image.

Example

```
sha256:89df9af...
```

---

## Why required?

Guarantees exact image.

Unlike tags,

digest never changes.

---

## Recruiters care

Production deployments often use digests.

Security.

Reproducibility.

---

## Production problem

Someone overwrites

```
latest
```

Digest still points to original image.

---

## Troubleshooting

Production image

```
latest
```

Developer

```
latest
```

Different.

Compare digest

```bash
docker inspect
```

Found mismatch.

---

## Analogy

Fingerprint.

Unique forever.

---

## Test yourself

1. Can two images have same digest?
2. Why are digests safer than tags?
3. What algorithm generates image digest?

---

# 9. Image Tags

---

## What is this?

Human-readable names pointing to images.

Example

```
nginx:latest

python:3.12

ubuntu:24.04
```

---

## Why required?

Humans can't remember SHA256 hashes.

---

## Recruiters care

Proper tagging strategy is critical in CI/CD.

---

## Production problem

Bad

```
latest
```

Good

```
v2.3.1

2026-07-13

git-7f1c92e
```

---

## Troubleshooting

Production deployed wrong version.

Check

```
kubectl describe deployment
```

Image tag

```
latest
```

That's probably why.

---

## Analogy

A contact name in your phone.

Tag = person's name.

Digest = Aadhaar fingerprint.

---

## Test yourself

1. Can multiple tags point to one image?
2. Can one tag point to different images over time?
3. Why should production avoid `latest`?

---

# Relationship Between These Concepts

```
                 Docker Image
                      │
        ┌─────────────┴──────────────┐
        │                            │
    Built From                   Has Metadata
        │                            │
    Base Image                  Manifest
        │                            │
        │                        Digest
        │
    Parent Images
        │
    Multiple Layers
        │
Immutable After Build
        │
Referenced Using Tags
```

---

# Production Scenario

Your Kubernetes deployment says

```yaml
image: company/payment:latest
```

Pods restart after deployment.

Developers say

> "Nothing changed."

Investigation:

```
docker manifest inspect
```

Shows a new image.

Digest changed.

Someone pushed a new image using the same `latest` tag.

Solution:

Deploy

```
company/payment@sha256:abc123...
```

or use immutable version tags like

```
payment:v1.8.4
```

This guarantees every pod runs the exact same image.

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What is the difference between a Docker image and a container?

**Answer:** An image is a read-only template containing the application, runtime, dependencies, and configuration. A container is a running instance of that image with a writable layer added on top.

---

### 2. Why are Docker images made of layers?

**Answer:** Layers enable caching, layer reuse across images, faster image builds, reduced storage usage, and faster downloads because unchanged layers are reused instead of rebuilt or re-downloaded.

---

### 3. What command shows the layers of an image?

**Answer:**

```bash
docker history <image>
```

---

### 4. What is the purpose of a base image?

**Answer:** It provides the initial filesystem and runtime environment on which the rest of the application image is built.

---

### 5. What is the difference between a base image and a parent image?

**Answer:** The **base image** is the image referenced by the `FROM` instruction in the current Dockerfile. A **parent image** is any image from which another image inherits. Every base image (except `scratch`) is also a child of another parent image.

---

### 6. What is the `scratch` image?

**Answer:** `scratch` is an empty starting point with no operating system files, shell, or libraries. It's commonly used for statically compiled applications to produce extremely small and secure images.

---

### 7. What does it mean that Docker images are immutable?

**Answer:** Once an image is built, its contents never change. Any modification requires building a new image instead of editing the existing one.

---

### 8. Why shouldn't production use the `latest` tag?

**Answer:** Because `latest` can point to different images over time, making deployments unpredictable and difficult to reproduce. Versioned tags or image digests provide consistent deployments.

---

### 9. What is an image digest?

**Answer:** A digest is a SHA-256 hash that uniquely identifies a specific image. Unlike tags, it never changes for that image.

---

### 10. What is an image manifest?

**Answer:** The manifest is a JSON document that describes an image's layers, configuration, operating system, architecture, and other metadata. Docker uses it to determine what to download.

---

### 11. How do Docker layers improve CI/CD performance?

**Answer:** Docker reuses cached layers if they haven't changed. By placing frequently changing instructions (such as `COPY .`) near the end of the Dockerfile, most earlier layers remain cached, significantly reducing build time.

---

### 12. How would you investigate why an image suddenly became much larger?

**Answer:** Use:

```bash
docker history <image>
```

to identify which layer introduced the size increase, then inspect the corresponding Dockerfile instruction.

---

### 13. How can image digests improve Kubernetes deployments?

**Answer:** Deploying by digest ensures every node pulls the exact same image, even if someone later changes the associated tag.

---

### 14. A security scan reports hundreds of vulnerabilities in your image. Where would you investigate first?

**Answer:** Start with the base image and parent images. Outdated operating system packages or runtimes inherited from those images are a common source of vulnerabilities.

---

### 15. Why do large organizations maintain their own internal base images?

**Answer:** To standardize security patches, approved packages, certificates, monitoring agents, and configuration across all application images, ensuring consistency, compliance, and simpler maintenance.
