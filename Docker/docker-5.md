Excellent. These are among the most frequently asked Docker interview topics for **Mid-level DevOps Engineers** because Dockerfiles determine **how production applications are built, secured, optimized, and deployed**.

I'll explain each topic from a DevOps perspective rather than simply defining Dockerfile instructions.

---

# Dockerfile

A Dockerfile is simply **a blueprint for creating a Docker image.**

Just as Terraform describes infrastructure,

Dockerfile describes an application image.

```
Dockerfile
        │
docker build
        │
        ▼
Docker Image
        │
docker run
        ▼
Docker Container
```

---

# 1. CMD

## What is this?

CMD specifies the **default command** that runs when a container starts.

Example

```dockerfile
FROM python:3.12

CMD ["python","app.py"]
```

Running

```
docker run myimage
```

actually executes

```
python app.py
```

---

## Why required?

Containers must know **what process to start.**

Without CMD:

```
docker run image
```

Container starts...

then immediately exits because nothing is running.

---

## Why recruiters care?

Because beginners often confuse

* CMD
* ENTRYPOINT
* RUN

Knowing the difference shows you understand container lifecycle.

---

## Production problem solved

Suppose a Flask app.

```
CMD ["python","app.py"]
```

Now Kubernetes starts Pods automatically.

No manual command required.

---

## Troubleshooting

Container exits immediately.

Check

```
docker inspect container
```

Look at

```
Config.Cmd
```

Maybe CMD points to wrong executable.

Example

```
CMD ["python","server.py"]
```

But file is

```
app.py
```

Container instantly exits.

---

## Analogy

CMD is like

> Default destination in Google Maps.

You can change destination before starting.

---

## Test yourself

1.

Can docker run override CMD?

Answer:

Yes.

```
docker run image ls
```

replaces CMD.

---

2.

Can Dockerfile have multiple CMD?

Only last one works.

---

3.

Difference between RUN and CMD?

RUN → during image build

CMD → during container runtime

---

# 2. ENTRYPOINT

## What is this?

ENTRYPOINT defines the **main executable**.

Example

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Runtime becomes

```
python app.py
```

---

## Why required?

Sometimes you never want users to replace executable.

Example

Official Redis image.

```
ENTRYPOINT ["redis-server"]
```

User only changes options.

```
docker run redis --port 6380
```

Actual execution

```
redis-server --port 6380
```

---

## Recruiters care because

This is one of the most common Docker interview questions.

---

## Production

Imagine backup container.

You never want user to accidentally run

```
bash
```

Instead

```
ENTRYPOINT ["backup.sh"]
```

Guarantees backup script always runs.

---

## Troubleshooting

Container behaves unexpectedly.

Check

```
docker inspect image
```

Look at

```
Entrypoint
Cmd
```

---

## Analogy

ENTRYPOINT is the engine.

CMD is fuel.

Engine stays same.

Fuel changes.

---

## Questions

1.

Difference between CMD and ENTRYPOINT?

2.

How can ENTRYPOINT and CMD work together?

3.

How to override ENTRYPOINT?

```
docker run --entrypoint bash image
```

---

# 3. COPY

## What is this?

Copies files from host into image.

```
COPY app.py /app/
```

---

## Why required?

Application code must exist inside image.

---

## Production

CI builds image.

COPY places application code inside image.

---

## Troubleshooting

Application missing.

```
docker run image ls
```

Maybe COPY path wrong.

Example

```
COPY src .
```

But folder actually

```
source/
```

---

## Analogy

Packing clothes into suitcase.

---

## Questions

1.

COPY vs bind mount?

2.

Can COPY fetch URLs?

No.

3.

Does COPY preserve permissions?

Mostly yes.

---

# 4. ADD

## What is this?

Like COPY but extra features.

It can

* download URL
* extract tar automatically

```
ADD app.tar.gz /
```

Automatically extracts.

---

## Why required?

Convenience.

---

## Production

Rarely recommended.

COPY is safer.

---

## Troubleshooting

Unexpected extracted directory.

Reason

ADD automatically untars.

---

## Analogy

COPY = move box.

ADD = unpack box automatically.

---

## Questions

1.

Difference COPY vs ADD?

2.

Why COPY preferred?

Predictable.

3.

When use ADD?

Mostly tar extraction.

---

# 5. ENV

## What is this?

Sets environment variables.

```
ENV PORT=8080
```

---

## Why required?

Configuration without changing code.

---

## Production

```
ENV LOG_LEVEL=INFO
```

Application reads

```
LOG_LEVEL
```

---

## Troubleshooting

```
docker exec container env
```

Check variables.

---

## Analogy

Settings menu.

---

## Questions

1.

Difference ENV vs ARG?

2.

Can ENV change at runtime?

Yes

```
docker run -e
```

3.

Where stored?

Image metadata.

---

# 6. ARG

## What is this?

Build-time variable.

```
ARG VERSION=3.12
```

---

## Why required?

Parameterize image build.

```
docker build --build-arg VERSION=3.13
```

---

## Production

Same Dockerfile.

Build

Python 3.12

or

Python 3.13

---

## Troubleshooting

Wrong version installed.

Check build args.

---

## Analogy

Cake recipe.

Choose flavor before baking.

Cannot change later.

---

## Questions

1.

ARG available at runtime?

No.

2.

ARG vs ENV?

3.

Can ARG appear before FROM?

Yes (for base image selection).

---

# 7. LABEL

## What is this?

Metadata.

```
LABEL maintainer="Sonu"
LABEL version="1.0"
```

---

## Production

Security scanners

CI/CD

Asset inventory

---

## Troubleshooting

```
docker inspect image
```

Find owner/version.

---

## Analogy

Shipping label.

---

## Questions

1.

Do LABEL affect execution?

No.

2.

How read labels?

docker inspect

3.

Why useful?

Metadata.

---

# 8. USER

## What is this?

Runs container as non-root.

```
USER appuser
```

---

## Why required?

Security.

---

## Production

Root compromise becomes much harder.

---

## Troubleshooting

Permission denied.

Check

```
whoami
```

inside container.

---

## Analogy

Admin vs guest account.

---

## Questions

1.

Why avoid root?

2.

How create user?

3.

How override?

```
docker run --user
```

---

# 9. HEALTHCHECK

## What is this?

Docker periodically checks application.

```
HEALTHCHECK CMD curl localhost:8080 || exit 1
```

---

## Production

Detect hung application.

---

## Troubleshooting

```
docker inspect
```

Shows

```
healthy
unhealthy
```

---

## Analogy

Doctor checking heartbeat.

---

## Questions

1.

Difference running vs healthy?

2.

Can process run but health fail?

Yes.

3.

Where check status?

docker ps

---

# 10. VOLUME

## What is this?

Declares persistent storage.

```
VOLUME /data
```

---

## Why?

Container filesystem disappears after deletion.

---

## Production

Database data.

Uploads.

Logs.

---

## Troubleshooting

Data missing after restart.

Check mounted volume.

---

## Analogy

External hard drive.

---

## Questions

1.

Volume vs bind mount?

2.

Why persist data?

3.

Where stored?

Docker managed volume.

---

# 11. SHELL

## What is this?

Changes default shell.

```
SHELL ["/bin/bash","-c"]
```

---

## Production

Need bash features.

Arrays.

pipefail.

---

## Troubleshooting

Build fails.

```
source command not found
```

Reason

Default shell is

```
sh
```

---

## Analogy

Changing language interpreter.

---

## Questions

1.

Default shell?

2.

Why bash?

3.

Can multiple SHELL exist?

Yes.

---

# 12. STOPSIGNAL

## What is this?

Signal Docker sends on stop.

Default

```
SIGTERM
```

Example

```
STOPSIGNAL SIGINT
```

---

## Production

Some applications need specific signal.

---

## Troubleshooting

Graceful shutdown fails.

Application ignores SIGTERM.

---

## Analogy

Different emergency alarms.

---

## Questions

1.

Default signal?

2.

Why change?

3.

Difference SIGTERM SIGKILL?

---

# 13. ONBUILD

## What is this?

Trigger instruction.

Runs only when another image inherits.

```
ONBUILD COPY . /app
```

---

## Production

Framework images.

---

## Troubleshooting

Unexpected COPY executed.

Reason

Inherited ONBUILD image.

---

## Analogy

Inheritance in OOP.

---

## Questions

1.

When ONBUILD execute?

2.

Why dangerous?

Unexpected behavior.

3.

Common use?

Base images.

---

# Build Cache

## What is it?

Docker reuses previously built layers instead of rebuilding them.

```
COPY requirements.txt .
RUN pip install

COPY . .
```

If only source code changes,

Docker skips reinstalling dependencies.

Huge speed improvement.

---

## Production impact

CI pipelines become much faster.

---

## Troubleshooting

```
docker build --no-cache
```

Fixes stale cache issues.

---

## Analogy

Cooking.

Reuse chopped vegetables instead of chopping again.

---

# Layer Creation

Every instruction creates a read-only layer.

```
FROM Ubuntu

RUN apt update

RUN apt install python

COPY app.py
```

Produces

Layer 1

↓

Layer 2

↓

Layer 3

↓

Layer 4

Containers add one writable layer on top.

---

## Production

Layer sharing reduces storage and download size.

---

## Troubleshooting

```
docker history image
```

Shows layer creation history.

Useful for identifying large layers.

---

## Analogy

Photoshop layers.

---

# Image Optimization

Main goal

Smaller

Safer

Faster

Common techniques

* Multi-stage builds
* Small base images (`alpine`, `distroless`, `al2023-minimal`)
* Combine related `RUN` commands to reduce unnecessary intermediate files
* Remove package caches
* Use `.dockerignore`
* Copy dependencies before source code to maximize cache reuse
* Run as non-root (`USER`)
* Avoid unnecessary packages

Example

Bad

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Good

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

Changing application code won't invalidate the dependency installation layer, making rebuilds much faster.

---

# 15 Important Mid-Level DevOps Interview Questions

### 1. Explain the difference between CMD and ENTRYPOINT.

**Answer:** `CMD` provides default arguments or a default command that can be overridden by `docker run`. `ENTRYPOINT` defines the primary executable that always runs unless explicitly overridden with `--entrypoint`. They are often combined so `ENTRYPOINT` specifies the executable and `CMD` supplies default arguments.

---

### 2. Why is COPY preferred over ADD?

**Answer:** `COPY` has a single, predictable purpose: copying local files into the image. `ADD` has extra behaviors (automatic tar extraction and URL downloads), which can introduce unexpected results. Use `ADD` only when those extra features are intentionally needed.

---

### 3. What is the difference between ARG and ENV?

**Answer:** `ARG` is available only during image build and is not available in running containers unless explicitly persisted. `ENV` is stored in the image and is available at runtime.

---

### 4. Why should containers avoid running as root?

**Answer:** Running as a non-root user follows the principle of least privilege and reduces the impact of a container compromise. It is a common security best practice in production and Kubernetes environments.

---

### 5. How does Docker build cache work?

**Answer:** Docker reuses previously built layers when the instruction and its inputs have not changed. A change in one layer invalidates that layer and all layers after it.

---

### 6. Why should `requirements.txt` or `package.json` be copied before the application source?

**Answer:** It allows dependency installation to be cached. Changes to application code won't force dependencies to be reinstalled, significantly speeding up builds.

---

### 7. How can you troubleshoot a container that exits immediately?

**Answer:** Check logs (`docker logs`), inspect the image (`docker inspect`) to verify `CMD` and `ENTRYPOINT`, and run an interactive shell (`docker run --entrypoint sh image`) to investigate.

---

### 8. What is the purpose of a HEALTHCHECK?

**Answer:** It verifies whether an application is actually functioning, not just whether its process is running. This helps orchestrators detect unhealthy containers.

---

### 9. What happens if you define multiple CMD instructions?

**Answer:** Only the last `CMD` instruction is effective. Earlier ones are discarded.

---

### 10. What is a Docker image layer?

**Answer:** Each Dockerfile instruction creates an immutable, read-only layer. Containers add a writable layer on top of these shared layers.

---

### 11. How do you reduce Docker image size?

**Answer:** Use multi-stage builds, minimal base images, `.dockerignore`, remove package caches, avoid unnecessary packages, and structure the Dockerfile to maximize cache reuse.

---

### 12. When would you use VOLUME?

**Answer:** When application data must survive container recreation, such as database files, uploaded content, or persistent application state.

---

### 13. What is STOPSIGNAL used for?

**Answer:** It specifies which Unix signal Docker sends to the main process when stopping a container, enabling graceful shutdown for applications that expect a specific signal.

---

### 14. How can you inspect how an image was built?

**Answer:** Use `docker history` to view the image's layer history and `docker inspect` to examine metadata such as environment variables, labels, entrypoint, and command.

---

### 15. Why is understanding Dockerfile optimization important for a DevOps engineer?

**Answer:** Efficient Dockerfiles reduce CI/CD build times, decrease image pull times, lower storage and network costs, improve security by minimizing the attack surface, and make deployments faster and more reliable across development, testing, and production environments.

---

If you're targeting **international mid-level DevOps roles**, mastering these topics is important, but I'd add three advanced Dockerfile subjects that are now commonly expected in interviews:

1. **Multi-stage builds** (one of the most frequently asked Dockerfile topics)
2. **`.dockerignore`** and build context optimization
3. **BuildKit features** such as `RUN --mount=type=cache` and `RUN --mount=type=secret` for faster, more secure builds

These topics often distinguish mid-level candidates from junior ones because they directly impact CI/CD performance, image security, and production deployment efficiency.
