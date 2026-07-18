This is one of the most important Docker topics for a mid-level DevOps engineer.

In production, you are **not paid for making containers work**.
You are paid for making them **small, secure, fast to build, fast to deploy, and easy to maintain.**

---

# 1. Alpine vs Debian vs Ubuntu

## What is this?

These are different Linux distributions used as the base image.

```dockerfile
FROM alpine
```

or

```dockerfile
FROM debian:bookworm-slim
```

or

```dockerfile
FROM ubuntu:24.04
```

Each provides different libraries, package managers and image sizes.

| Distribution | Size                 | Package Manager | Typical Use                |
| ------------ | -------------------- | --------------- | -------------------------- |
| Alpine       | Very small (~5 MB)   | apk             | Small containers           |
| Debian Slim  | Medium (~25-80 MB)   | apt             | Most production apps       |
| Ubuntu       | Larger (~70-150 MB+) | apt             | Development, compatibility |

---

## Why required?

Choosing the wrong base image can cause

* larger downloads
* longer deployments
* more vulnerabilities
* compatibility issues

---

## Production Example

Suppose:

100 Kubernetes nodes

Application image

```
Ubuntu image = 400MB
```

New deployment

Every node downloads

```
400 MB ×100

=40 GB
```

Now switch to

```
Debian Slim =120 MB

Total

12 GB
```

Deployment becomes much faster.

---

## Troubleshooting Example

Application crashes only on Alpine.

Reason?

Alpine uses

```
musl libc
```

instead of

```
glibc
```

Some binaries require glibc.

You inspect

```
ldd binary
```

or

```
file binary
```

and discover missing libraries.

---

## Analogy

Think of moving to another city.

Ubuntu

> Taking entire house.

Debian

> Taking important furniture.

Alpine

> Taking only one backpack.

---

## Test Yourself

1. Why is Alpine smaller?

2. Why do some applications fail on Alpine?

3. When would you choose Debian Slim over Alpine?

---

# 2. Distroless Images

## What is this?

Distroless images contain

* application
* required runtime libraries

Nothing else.

No

* bash
* apt
* curl
* package manager

Example

```
gcr.io/distroless/java
```

---

## Why required?

Smaller attack surface.

Fewer vulnerabilities.

Smaller images.

---

## Production Example

Security scan

Ubuntu

```
180 CVEs
```

Distroless

```
12 CVEs
```

Huge security improvement.

---

## Troubleshooting

Problem

```
kubectl exec

bash
```

fails.

Why?

There is no shell.

Instead

Use

```
kubectl debug
```

or attach ephemeral debug container.

---

## Analogy

Hotel room

Normal image

Everything included

Kitchen

TV

Dining

Distroless

Only bed.

Nothing else.

---

## Test Yourself

1. Why can't you use bash?

2. Why are vulnerabilities lower?

3. How do you debug distroless containers?

---

# 3. Minimal Images

## What is this?

Minimal images remove unnecessary software.

Example

```
debian:bookworm-slim
```

instead of

```
debian
```

---

## Why?

Reduce

* size
* attack surface
* pull time

---

## Production Example

CI pipeline

Normal image

```
3 minutes
```

Minimal image

```
50 seconds
```

---

## Troubleshooting

Check image history

```
docker history image
```

Large layers often indicate unnecessary packages.

---

## Analogy

Traveling with cabin luggage instead of three suitcases.

---

## Questions

1. Difference between minimal and distroless?

2. Why fewer vulnerabilities?

3. How to inspect image size?

---

# 4. Multi-stage Builds

## What is this?

Separate build stage from runtime stage.

```dockerfile
FROM golang AS builder

RUN go build

FROM alpine

COPY --from=builder app .
```

---

## Why?

Compiler isn't needed at runtime.

---

## Production Example

Without multi-stage

```
Image

900MB
```

With multi-stage

```
40MB
```

Huge savings.

---

## Troubleshooting

Large image?

Inspect

```
docker history
```

Maybe compiler is still inside runtime image.

---

## Analogy

Construction workers build a house.

After construction,

Workers leave.

Only house remains.

---

## Questions

1. Why use multiple FROM statements?

2. What gets copied?

3. Why smaller?

---

# 5. Removing Unnecessary Packages

## What is this?

Delete temporary packages after installation.

Instead of

```dockerfile
RUN apt update

RUN apt install gcc

RUN build app
```

Do

```dockerfile
RUN apt update && \
apt install gcc && \
build app && \
apt remove gcc && \
apt clean
```

---

## Why?

Temporary build tools don't belong in production.

---

## Production Example

Removing

* gcc
* make
* cache

Reduced image

```
700MB

→

180MB
```

---

## Troubleshooting

Inspect

```
docker history
```

Huge layers often come from package installation.

---

## Analogy

Buying furniture.

Throw away cardboard boxes after unpacking.

---

## Questions

1. Why remove gcc?

2. Why clean apt cache?

3. Does deleting later remove previous layers?

---

Answer:

No.

Because Docker layers are immutable.

Need to remove in same RUN.

---

# 6. Layer Optimization

## What is this?

Each Docker instruction creates a layer.

```
FROM

RUN

COPY

ENV

ADD
```

---

## Why?

Smaller images

Better caching

Faster builds

---

Bad

```dockerfile
RUN apt update

RUN apt install git

RUN apt clean
```

Better

```dockerfile
RUN apt update && \
apt install git && \
apt clean
```

---

## Troubleshooting

View

```
docker history image
```

Find huge layers.

---

## Analogy

Saving PowerPoint.

Each save creates a version.

Fewer meaningful saves.

---

## Questions

1. Which instructions create layers?

2. Why combine RUN commands?

3. How inspect layers?

---

# 7. COPY Ordering

## What is this?

Docker cache depends on previous layers.

Bad

```dockerfile
COPY .

RUN npm install
```

Every code change

↓

npm install again.

---

Better

```dockerfile
COPY package.json .

RUN npm install

COPY .
```

Now dependency cache remains.

---

## Production Example

Build

Before

```
6 min
```

After

```
45 sec
```

---

## Troubleshooting

Why build always slow?

Probably

COPY . placed too early.

---

## Analogy

Cooking.

First buy ingredients.

Then cook.

Don't buy everything every time.

---

## Questions

1. Why copy package.json first?

2. What invalidates Docker cache?

3. Which files change most frequently?

---

# 8. Build Cache Optimization

## What is this?

Docker reuses unchanged layers.

---

If

```dockerfile
RUN npm install
```

already exists

Docker skips it.

---

## Why?

Much faster CI.

---

## Production Example

Pipeline

Without cache

```
8 min
```

With cache

```
1 min
```

---

## Troubleshooting

Build always rebuilding?

Check

* COPY ordering
* Changed ENV
* Changed ARG
* Build context
* `.dockerignore`

---

## Analogy

Building Lego.

Reuse completed blocks.

Don't rebuild entire castle.

---

## Questions

1. When is cache invalidated?

2. How inspect cached layers?

3. Why is `.dockerignore` important?

---

# Relationship Between All Topics

```
                    Docker Image
                          │
          ┌───────────────┼─────────────────┐
          │               │                 │
     Base Image      Build Process      Runtime
          │               │                 │
          │               │                 │
 Alpine/Debian      Multi-stage        Distroless
 Ubuntu             Layer Order        Minimal Image
                    COPY Order
                    Build Cache
                    Remove Packages
```

---

# Production Troubleshooting Flow

Suppose deployment suddenly becomes slow.

```
Pods starting slowly
        │
        ▼
Image pull taking long?
        │
        ▼
docker image inspect
        │
        ▼
Image grew from

180MB

to

950MB
        │
        ▼
docker history
        │
        ▼
Large apt install layer
        │
        ▼
Compiler accidentally included
        │
        ▼
Move compiler to build stage
        │
        ▼
Use multi-stage build
        │
        ▼
Image back to 180MB
        │
        ▼
Deployment fast again
```

---

# 15 Mid-Level DevOps Interview Questions (with Answers)

### 1. Why is image size important in Kubernetes?

**Answer:** Smaller images reduce registry storage, image pull time, deployment duration, network usage, and startup latency. This leads to faster rolling updates and quicker recovery during autoscaling or node replacement.

---

### 2. When would you choose Alpine over Debian Slim?

**Answer:** Alpine is suitable when the application is compatible with `musl` and image size is the highest priority. Debian Slim is preferred when better library compatibility (`glibc`) and easier debugging are needed.

---

### 3. Why do some applications fail on Alpine?

**Answer:** Alpine uses `musl libc` instead of `glibc`. Applications or third-party binaries compiled against `glibc` may fail unless compatibility packages are added or the application is rebuilt.

---

### 4. What is a multi-stage build?

**Answer:** A Docker build pattern that separates the build environment from the runtime environment. Build tools remain in the builder stage, and only the compiled application is copied into the final image, producing a smaller and more secure runtime image.

---

### 5. Why are distroless images considered more secure?

**Answer:** They contain only the application and required runtime libraries, removing shells, package managers, and other utilities. This reduces the attack surface and typically results in fewer reported CVEs.

---

### 6. How do you debug a distroless container?

**Answer:** Since it has no shell, use `kubectl debug` with an ephemeral container, inspect logs, monitor metrics, or reproduce the issue locally with a debug-friendly image.

---

### 7. Why should package installation and cleanup happen in the same `RUN` instruction?

**Answer:** Docker layers are immutable. If installation and cleanup occur in separate layers, the deleted files still exist in earlier layers and continue to increase the image size.

---

### 8. How does Docker build cache work?

**Answer:** Docker reuses a cached layer when the instruction and everything it depends on are unchanged. If a layer changes, all subsequent layers are rebuilt.

---

### 9. Why is `COPY package.json` typically placed before `COPY .` in Node.js projects?

**Answer:** Dependency files change less frequently than application code. Copying them first allows Docker to reuse the cached dependency installation layer, significantly speeding up rebuilds.

---

### 10. How can `.dockerignore` improve build performance?

**Answer:** It excludes unnecessary files (such as `.git`, logs, test data, and `node_modules`) from the build context, reducing data transfer to the Docker daemon and preventing unnecessary cache invalidation.

---

### 11. How do you identify which layer is making an image large?

**Answer:** Use `docker history <image>` to inspect the size of each layer and determine which instruction contributed the most.

---

### 12. A Docker build suddenly becomes much slower. What would you check first?

**Answer:** I would check recent Dockerfile changes, `COPY` ordering, build cache usage, `.dockerignore`, dependency changes, and whether a previously cached layer is now being invalidated.

---

### 13. Can deleting a file in a later Docker layer reduce the final image size?

**Answer:** No. The file still exists in the earlier immutable layer. To avoid storing it, create and remove it within the same `RUN` instruction or use a multi-stage build.

---

### 14. What is the difference between a minimal image and a distroless image?

**Answer:** A minimal image still includes a basic Linux userspace (and often a package manager), making it easier to debug. A distroless image removes almost everything except the runtime and application, maximizing security and minimizing size.

---

### 15. How would you reduce the size of a 1 GB Docker image?

**Answer:** I would:

* Use a smaller base image (for example, Debian Slim instead of Ubuntu, where appropriate).
* Convert the Dockerfile to a multi-stage build.
* Remove unnecessary build tools and packages.
* Clean package manager caches in the same `RUN` instruction.
* Optimize Docker layers and `COPY` ordering.
* Use `.dockerignore` to exclude unnecessary files.
* Inspect the image with `docker history` to identify oversized layers and optimize them systematically.

These are the kinds of answers interviewers expect from a mid-level DevOps engineer because they combine Docker internals with practical production considerations and troubleshooting.
