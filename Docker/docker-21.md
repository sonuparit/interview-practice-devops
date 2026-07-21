# Docker Storage

Docker storage is one of the most misunderstood topics in interviews. Many engineers know how to build images, but don't understand **how Docker actually stores files on disk**.

Understanding this helps you solve:

* Containers running out of disk
* Slow image builds
* Large image sizes
* Data disappearing after container deletion
* Performance issues
* Image optimization

---

# 1. Storage Drivers

## What is it?

A **storage driver** is the component inside Docker that decides **how Docker stores image layers and container files on the host machine.**

Think of it as the filesystem implementation used by Docker.

Examples:

* overlay2 (default)
* aufs (old)
* btrfs
* zfs
* devicemapper (old)

Today, **overlay2** is used almost everywhere.

---

## Why is it required?

Docker images consist of many layers.

Docker needs a way to:

* Store them
* Share them
* Reuse them
* Merge them together
* Save disk space

The storage driver does all of this.

Without a storage driver,

Docker would have to duplicate the whole filesystem for every container.

That would waste enormous storage.

---

## Production Problem

Suppose:

100 containers run nginx.

Without storage drivers:

# 100 × 200MB image

20 GB

With storage driver:

Only one image is stored.

All containers reuse it.

Huge disk savings.

---

## Troubleshooting Example

Container suddenly becomes very slow.

You check:

```bash
docker info
```

Output:

```
Storage Driver: overlay2
```

Good.

If you see:

```
vfs
```

This driver copies every file.

Performance becomes terrible.

Root cause found.

---

## Analogy

Imagine:

One apartment blueprint.

100 apartments are built using the same blueprint.

No need to redraw it 100 times.

Storage driver works exactly like this.

---

## Questions

1.

What is Docker Storage Driver?

2.

Why is overlay2 preferred?

3.

What happens if Docker used no storage driver?

---

# 2. overlay2

This is the most important storage driver.

Interviewers love asking this.

---

## What is overlay2?

overlay2 uses Linux OverlayFS.

It combines multiple directories into one unified filesystem.

It stacks layers together.

```
Layer 5
Layer 4
Layer 3
Layer 2
Layer 1

↓

Merged View
```

Container only sees:

```
/
```

It never sees separate layers.

---

## Why required?

Imagine Ubuntu image.

Layer 1

```
Kernel libraries
```

Layer 2

```
apt packages
```

Layer 3

```
Python
```

Layer 4

```
Application
```

overlay2 combines all these into one filesystem.

Without copying them.

---

## Production Benefit

Suppose:

Ubuntu image

300MB

Python image

350MB

They both share Ubuntu layers.

Docker stores Ubuntu once.

Huge storage savings.

---

## Troubleshooting Example

Developer says:

Image size exploded.

You inspect:

```
docker history image
```

Find:

```
RUN apt update
RUN apt install
RUN rm
```

Many unnecessary layers.

overlay2 stores every layer.

You optimize Dockerfile.

Problem solved.

---

## Analogy

Imagine transparent sheets.

Each sheet has some drawing.

Stack them together.

You see one complete picture.

OverlayFS works exactly like this.

---

## Questions

1.

How does overlay2 work?

2.

Why is overlay2 faster?

3.

Why doesn't Docker duplicate common files?

---

# 3. Copy-on-Write (CoW)

One of Docker's core concepts.

---

## What is it?

Docker never copies files unless modification happens.

Read:

Shared.

Write:

Copy first.

Then modify.

Hence:

Copy-on-Write.

---

Example

Image contains

```
/etc/config.txt
```

100 containers use it.

Storage:

One file.

Now one container edits it.

Docker copies only that file.

Other containers still share original.

---

## Why required?

Imagine:

Ubuntu image

500MB

100 containers.

Without CoW:

50GB

With CoW:

Only changed files occupy space.

Massive savings.

---

## Production Benefit

Suppose application only changes

```
app.log
```

Everything else remains shared.

Container consumes very little additional storage.

---

## Troubleshooting Example

Container disk usage increases.

Check:

```
docker ps -s
```

Example:

```
SIZE

8GB
```

Container wrote huge log files.

Copy-on-write layer became huge.

Root cause found.

---

## Analogy

Think of Google Docs.

Everyone views one document.

Only when someone edits,

their own version gets changes.

Original remains.

---

## Questions

1.

What is Copy-on-Write?

2.

Why is it efficient?

3.

When does Docker copy files?

---

# 4. Writable Layer

Every running container gets one writable layer.

---

## What is it?

Images are read-only.

Docker adds one writable layer above them.

```
Writable Layer

↓

Image Layer

↓

Image Layer

↓

Image Layer
```

All writes go here.

---

## Why required?

Containers need to:

* create logs
* create temp files
* modify configs

Without writable layer,

container couldn't function.

---

## Production Benefit

Application writes

```
/tmp
```

or

```
/var/log
```

Those changes remain only in writable layer.

Image remains untouched.

---

## Troubleshooting Example

Application crashes.

Disk full.

Run:

```
docker ps -s
```

Shows

```
Writable layer

12GB
```

Logs filled writable layer.

Solution:

Move logs to volumes.

---

## Analogy

Think of laminated paper.

Original paper cannot be written on.

You write using a transparent sheet placed above it.

That's writable layer.

---

## Questions

1.

Where are container changes stored?

2.

Why is image never modified?

3.

What happens after container deletion?

Answer:

Writable layer disappears.

---

# 5. Read-only Layers

Every Docker image consists of immutable (read-only) layers.

---

## What is it?

Each Dockerfile instruction creates a layer.

```
FROM Ubuntu

↓

Layer 1

RUN apt install nginx

↓

Layer 2

COPY app

↓

Layer 3

CMD

↓

Metadata
```

These layers never change.

---

## Why required?

If images changed,

Docker couldn't:

* cache builds
* share layers
* verify hashes
* download efficiently

Read-only layers make images reusable and predictable.

---

## Production Benefit

10 microservices all use

```
FROM ubuntu
```

Ubuntu layer downloaded once.

Everything reused.

---

## Troubleshooting Example

Developer modifies

```
/usr/bin/python
```

inside container.

After restart:

Changes gone.

Reason:

Image never changed.

Only writable layer changed.

---

## Analogy

Think of a printed book.

You cannot modify printed pages.

You write notes using sticky notes.

Sticky notes = writable layer.

Book = read-only layers.

---

## Questions

1.

Why are Docker image layers read-only?

2.

How does Docker cache builds?

3.

Why are image downloads fast?

---

# Layer Structure

```
             Container

        Writable Layer
-------------------------------
        App Layer
-------------------------------
     Python Layer
-------------------------------
    Ubuntu Packages
-------------------------------
      Base Ubuntu
-------------------------------
          Disk
```

Only the **top writable layer** changes.

Everything below remains immutable.

---

# Real Production Scenario

Your monitoring alerts:

```
Host disk 95% full.
```

You investigate:

```bash
docker system df
```

Shows:

```
Containers
18GB

Images
4GB

Volumes
2GB
```

Check container size:

```bash
docker ps -s
```

```
mysql

Writable layer

15GB
```

Reason:

Application writes logs inside container.

Instead,

Mount:

```
/var/log

↓

Docker Volume
```

Now writable layer remains small.

Problem solved.

---

# Commands Every DevOps Engineer Should Know

```bash
docker info
```

Storage driver

---

```bash
docker system df
```

Docker disk usage

---

```bash
docker image inspect IMAGE
```

Inspect image metadata and layers

---

```bash
docker history IMAGE
```

Layer history

---

```bash
docker ps -s
```

Writable layer size

---

```bash
docker system prune
```

Remove unused objects

---

```bash
docker image prune
```

Remove unused images

---

```bash
docker volume ls
```

Volumes

---

```bash
docker volume inspect VOLUME
```

Volume location

---

```bash
du -sh /var/lib/docker
```

Host disk usage

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. What is a Docker storage driver?

A storage driver determines how Docker stores and manages image layers and writable container layers on the host. It enables efficient sharing, layering, and space optimization. On modern Linux systems, `overlay2` is the default and recommended driver.

---

### 2. Why is `overlay2` the default storage driver?

It uses Linux OverlayFS to merge multiple read-only image layers with a writable container layer into a single unified filesystem. It offers good performance, efficient disk usage, and broad kernel support.

---

### 3. What is Copy-on-Write (CoW)?

Copy-on-Write allows containers to share read-only image files. A file is copied into the container's writable layer only when it is modified, reducing storage usage and improving efficiency.

---

### 4. What is the writable layer?

The writable layer is the top layer added to every running container. All file creations, modifications, and deletions made by the container are stored there. It is removed when the container is deleted unless data is stored in a volume.

---

### 5. Why are Docker image layers read-only?

Read-only layers make images immutable, allowing layer reuse, build caching, content verification, and efficient image distribution without modifying the original image.

---

### 6. What happens when you modify `/etc/hosts` inside a container?

The file is copied from the shared read-only layer into the container's writable layer (if not already there), and the modification is made only in that container. Other containers remain unaffected.

---

### 7. Why are Docker images built in layers?

Each Dockerfile instruction creates a separate layer. This enables Docker to reuse unchanged layers during future builds, making builds faster and reducing network and storage usage.

---

### 8. How do you identify why Docker is consuming too much disk space?

Use:

* `docker system df` to view disk usage by images, containers, volumes, and build cache.
* `docker ps -s` to identify containers with large writable layers.
* `du -sh /var/lib/docker` to inspect Docker's storage on the host.

---

### 9. Why should logs not be stored in a container's writable layer?

Logs continuously grow and increase the writable layer size. This can fill the host disk and degrade performance. Store logs in Docker volumes or use centralized logging solutions instead.

---

### 10. What happens if you delete a container?

The container's writable layer is deleted. Any data stored only in that layer is lost. Data stored in volumes or bind mounts remains intact.

---

### 11. How does Docker avoid downloading the same base image multiple times?

Docker identifies layers by their content hash. If a required layer already exists locally, it is reused instead of downloaded again.

---

### 12. Why is `docker history` useful?

It displays the image's layer history, including the Dockerfile instructions that created each layer. It helps identify unnecessary layers, oversized commands, and optimization opportunities.

---

### 13. What is the difference between image layers and a container layer?

* **Image layers:** Immutable, shared across containers, created during image build.
* **Container layer:** Writable, unique to each container, stores runtime changes.

---

### 14. How can poor Dockerfile design increase storage usage?

Creating unnecessary layers, installing temporary files without cleaning them in the same `RUN` instruction, and copying large files early in the build can increase image size and reduce cache efficiency.

---

### 15. A container's writable layer grows to 20 GB. How would you troubleshoot it?

A good troubleshooting approach is:

1. Check the writable layer size with `docker ps -s`.
2. Inspect the container for large files (logs, caches, temporary files).
3. Determine whether application data should be moved to a Docker volume.
4. Configure log rotation or external logging if logs are the cause.
5. Remove unnecessary temporary files and rebuild the image if needed.

This demonstrates an understanding of both Docker's storage model and practical production troubleshooting, which is what interviewers typically look for in a mid-level DevOps engineer.
