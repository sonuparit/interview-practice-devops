Excellent. These topics are asked surprisingly often in Mid-level DevOps interviews because they deal with **running containers in production**, not just building them.

---

# 1. `docker save`

## 1. What is this?

`docker save` exports one or more Docker images into a tar archive.

Example

```bash
docker save myapp:v1 -o myapp.tar
```

Produces

```
myapp.tar
```

The tar contains

* Image layers
* Image metadata
* Tags
* Manifest

It does **NOT** include running containers.

---

## 2. Why is it required?

Sometimes production servers

* have no Internet
* cannot access Docker Hub
* are air-gapped
* require offline deployment

Instead of downloading from registry

```
Registry
    ↓
docker pull
```

you do

```
Developer
    ↓
docker save

USB / SCP

Production

docker load
```

---

## 3. Why recruiters care?

They want to know whether you understand deployment beyond Docker Hub.

Many enterprises

* don't allow Internet
* use secure environments
* move images manually

---

## 4. Production usage

Example

Government data center

No Internet allowed.

Developer machine

```
docker save payment:v2
```

Copy

```
payment.tar
```

Production

```
docker load
docker run
```

Application starts.

---

## 5. Troubleshooting

Problem

```
Cannot pull image

Connection timeout
```

Solution

Ask someone who has the image

```
docker save
```

Transfer

```
docker load
```

Problem solved.

---

## 6. Analogy

Think of

```
docker save

=

Packing an entire house into boxes for moving.
```

Everything required travels together.

---

## 7. Test yourself

Can you answer

1. Does docker save export containers?
2. Does docker save preserve tags?
3. Why would a company prefer docker save over docker pull?

---

# 2. `docker load`

---

## 1. What is this?

Imports images previously exported by `docker save`.

```
docker load -i myapp.tar
```

Docker recreates

```
Image
Layers
Tags
Manifest
```

---

## 2. Why required?

Moves images

Machine A

↓

Machine B

without registry.

---

## 3. Why recruiters care?

Production environments often

* cannot access Docker Hub
* restore backups
* migrate environments

---

## 4. Production usage

Disaster Recovery

Registry failed.

Luckily

```
image.tar
```

exists.

```
docker load
```

Entire application restored.

---

## 5. Troubleshooting

Registry corrupted.

Restore image

```
docker load
```

instead of rebuilding.

---

## 6. Analogy

Imagine unpacking moving boxes into a new house.

That's docker load.

---

## 7. Questions

1. Which command works with docker save?
2. Does docker load create containers?
3. Does docker load preserve tags?

---

# 3. `docker pause`

---

## 1. What is this?

Temporarily freezes all processes inside a container.

```
docker pause nginx
```

Container still exists.

Processes stop executing.

Uses Linux cgroups freezer.

---

## 2. Why required?

Need to temporarily stop execution

without shutting down.

---

## 3. Recruiters care because

They expect engineers to know

Difference between

```
pause

stop

kill
```

---

## 4. Production usage

Database maintenance.

Pause application

```
docker pause app
```

Perform maintenance.

Resume

```
docker unpause
```

No restart needed.

---

## 5. Troubleshooting

Container consuming resources unexpectedly.

Pause it

Observe host

Resume later.

---

## 6. Analogy

Movie

Pause button.

Movie isn't closed.

Just frozen.

---

## 7. Questions

1. Does pause stop the container?
2. Can paused container receive CPU?
3. Difference between pause and stop?

---

# 4. `docker unpause`

---

## 1. What is this?

Resumes execution.

```
docker unpause nginx
```

---

## 2. Why required?

Continue after maintenance.

---

## 3. Recruiters care

Because they expect engineers to understand lifecycle management.

---

## 4. Production usage

Resume services after

* backups
* debugging
* maintenance

---

## 5. Troubleshooting

Paused accidentally?

```
docker unpause
```

Application works again.

---

## 6. Analogy

Press Play.

---

## 7. Questions

1. Does unpause restart?
2. Does PID change?
3. Does application lose memory?

---

# 5. `docker attach`

---

## 1. What is this?

Connects your terminal to the main process inside a running container.

```
docker attach container1
```

You're connected to

PID 1.

---

## 2. Why required?

Interact directly with application output.

---

## 3. Recruiters care

Need to know

Difference

```
attach

exec
```

Very common interview question.

---

## 4. Production usage

Observe

Interactive Python

Node

Java

applications.

---

## 5. Troubleshooting

Application waiting for input.

Attach

Provide input.

Continue execution.

---

## 6. Analogy

Like plugging your monitor directly into a running computer.

---

## 7. Questions

1. Difference between attach and exec?
2. Which process do you connect to?
3. Can multiple users attach?

---

# Important Difference

```
docker attach
```

Connects

```
PID 1
```

Whereas

```
docker exec -it container bash
```

Starts

another process

```
bash
```

inside container.

Interviewers LOVE this question.

---

# 6. `docker logs`

---

## 1. What is this?

Displays stdout and stderr produced by the container.

```
docker logs nginx
```

Live logs

```
docker logs -f nginx
```

Last 100 lines

```
docker logs --tail 100 nginx
```

---

## 2. Why required?

Containers don't have traditional log files by default.

Logs are usually

```
stdout

stderr
```

Docker captures them.

---

## 3. Recruiters care

Because first debugging step is

```
docker logs
```

---

## 4. Production usage

Customer reports

```
500 Internal Server Error
```

You immediately

```
docker logs payment
```

See

```
Database connection refused
```

Root cause found.

---

## 5. Troubleshooting

Examples

Application crashed

```
docker logs
```

OOM

```
Killed
```

Python traceback

```
ImportError
```

Permission

```
Permission denied
```

Port

```
Address already in use
```

All visible.

---

## 6. Analogy

Security camera footage.

You rewind and see

what happened.

---

## 7. Questions

1. Where do docker logs come from?
2. Difference between logs and exec?
3. Which option follows logs live?

---

# 7. `docker inspect`

---

## 1. What is this?

Shows detailed JSON metadata.

```
docker inspect nginx
```

Returns

* IP
* Mounts
* Volumes
* Network
* Labels
* Image
* Environment variables
* Restart policy
* PID
* Healthcheck

Everything.

---

## 2. Why required?

Need detailed information unavailable from

```
docker ps
```

---

## 3. Recruiters care

Real troubleshooting depends on inspect.

---

## 4. Production usage

Application cannot connect.

Inspect

```
IPAddress
```

Network

```
Bridge
```

Volumes

Environment

Restart policy

Healthcheck

Everything visible.

---

## 5. Troubleshooting

Examples

Wrong environment variable

```
docker inspect
```

Missing volume

```
docker inspect
```

Wrong port mapping

```
docker inspect
```

Healthcheck failing

```
docker inspect
```

---

## 6. Analogy

Hospital MRI.

Outside looks normal.

Inspect shows everything inside.

---

## 7. Questions

1. What information does inspect provide?
2. Why use inspect instead of ps?
3. Is inspect human readable or JSON?

---

# 8. Content-addressable Storage

This is one of the most important concepts recruiters ask.

---

## 1. What is this?

Docker stores layers using

their content hash.

Example

```
Layer

↓

SHA256

↓

sha256:98f9c...
```

If content changes

Hash changes.

---

## 2. Why required?

Avoid duplicates.

Suppose

Ubuntu layer

```
120 MB
```

used by

```
Python image

Node image

Java image
```

Docker stores

only one copy.

Huge disk savings.

---

## 3. Recruiters care

Shows understanding of

* image layers
* caching
* immutability
* storage optimization

---

## 4. Production usage

Server has

100 containers.

Ubuntu base exists.

Stored

once.

Disk usage reduced dramatically.

---

## 5. Troubleshooting

Problem

```
Disk full
```

Engineer wonders

```
Why only 40 GB?

I have 300 images.
```

Answer

Many share layers.

Content-addressable storage prevents duplication.

---

## 6. Analogy

Google Drive.

Upload same file twice.

It stores

one copy

Both references point to it.

---

## 7. Questions

1. Why does Docker use hashes?
2. Why don't identical layers consume duplicate disk?
3. What happens if layer content changes?

---

# Mid-Level Interview Questions (Must Know)

## 1.

What is the difference between docker save and docker export?

**Answer**

* save → images
* export → container filesystem only

---

## 2.

Difference between docker save and docker commit?

**Answer**

save exports an image.

commit creates a new image from a container.

---

## 3.

Difference between docker attach and docker exec?

**Answer**

attach connects to the existing main process (PID 1). `exec` starts a new process inside the running container (for example, `bash`). `exec` is generally preferred for debugging because it doesn't interfere with the application's primary process.

---

## 4.

When should docker logs be your first troubleshooting command?

**Answer**

Almost always when an application is failing to start, crashing, or returning errors, because it quickly reveals startup exceptions, stack traces, permission problems, missing configuration, and runtime errors.

---

## 5.

Difference between pause and stop?

**Answer**

pause freezes running processes without terminating them. stop sends a signal (SIGTERM, then SIGKILL if necessary) to gracefully terminate the container.

---

## 6.

Can docker load recreate containers?

**Answer**

No. It restores images only. You still need `docker run` (or Docker Compose/Kubernetes) to create containers.

---

## 7.

What is stored inside docker save?

**Answer**

Image layers, image configuration, tags, and the image manifest.

---

## 8.

How does Docker avoid duplicate image storage?

**Answer**

Using content-addressable storage. Identical layers have the same SHA-256 digest and are stored only once.

---

## 9.

How do you inspect environment variables of a running container?

**Answer**

```bash
docker inspect <container>
```

Then look at the `Config.Env` section (or use a Go template such as `docker inspect -f '{{json .Config.Env}}' <container>`).

---

## 10.

How do you continuously watch logs?

**Answer**

```bash
docker logs -f container
```

---

## 11.

A container cannot connect to another service. Which Docker command helps you investigate the networking configuration?

**Answer**

```bash
docker inspect
```

Check the network, IP address, DNS settings, and attached networks.

---

## 12.

Why is `docker exec` preferred over `docker attach` for production debugging?

**Answer**

Because `exec` starts a separate shell or command without attaching to the application's main process, making it safer and less disruptive.

---

## 13.

Can a paused container still consume CPU?

**Answer**

Its application processes are frozen and should not consume CPU time while paused, though the container still exists and occupies memory and other resources.

---

## 14.

Can you move Docker images between two offline servers?

**Answer**

Yes:

```bash
docker save myimage -o image.tar
scp image.tar server2:/tmp
docker load -i image.tar
```

---

## 15.

What is the purpose of image digests (SHA-256 hashes) in Docker?

**Answer**

They uniquely identify image content, verify integrity, enable content-addressable storage, and ensure that pulling an image by digest always retrieves the exact same image.

---

# High-Value Interview Tip

Among all these topics, interviewers most frequently explore these four because they reveal practical production experience:

1. **`docker logs`** – your first tool for diagnosing container failures.
2. **`docker inspect`** – essential for investigating networking, environment variables, volumes, mounts, health checks, and runtime configuration.
3. **`docker attach` vs `docker exec`** – a classic interview question testing understanding of container processes.
4. **Content-addressable storage** – demonstrates understanding of image layers, storage efficiency, caching, immutability, and image digests.

Being able to explain not just *what* these commands do, but *when* and *why* you would use them during a production incident, is what typically distinguishes a mid-level DevOps engineer from someone who has only learned Docker commands by rote.
