This is a very important topic for a **Mid-Level DevOps Engineer** because almost every production deployment (Docker, Kubernetes, ECS, Nomad, GitHub Actions, Jenkins, etc.) relies on environment variables.

---

# 1. `-e` (Environment Variable)

## What is it?

`-e` passes an environment variable into a running container.

```bash
docker run \
-e DB_HOST=mysql \
-e DB_PORT=3306 \
-e LOG_LEVEL=debug \
myapp
```

Inside the container

```bash
echo $DB_HOST
```

Output

```
mysql
```

Python

```python
import os

db = os.getenv("DB_HOST")
```

---

## Why is it required?

Without environment variables, every environment would require rebuilding the Docker image.

Example

Instead of

```
Development Image
Production Image
Testing Image
```

You build only one image.

```
myapp:v1
```

Run it differently

Development

```
docker run -e DB_HOST=dev-db myapp:v1
```

Production

```
docker run -e DB_HOST=prod-db myapp:v1
```

Same image.

Different configuration.

This follows the **12-Factor App** principle.

---

## Production Problems it Solves

Suppose

```
Image:
mycompany/payment:v4
```

Need to deploy

* Development
* QA
* Staging
* Production

Without env variables

```
payment-dev
payment-qa
payment-stage
payment-prod
```

Four images.

With env variables

One image.

Different runtime configuration.

Huge operational simplification.

---

## Troubleshooting Example

Application cannot connect to database.

Check

```
docker inspect container_name
```

or

```
docker exec container env
```

Output

```
DB_HOST=wronghost
```

Immediately found the issue.

---

## Analogy

Think of a TV.

The TV is the Docker Image.

The remote control changes

* volume
* brightness
* language

without changing the TV itself.

Environment variables are like the remote.

---

## Test Yourself

Can you answer?

1. Why shouldn't we rebuild an image just to change the database hostname?
2. Where do environment variables exist, image or running container?
3. How can you check all environment variables inside a running container?

---

# 2. `--env-file`

Instead of

```bash
docker run \
-e DB_HOST=mysql \
-e DB_PORT=3306 \
-e USER=admin \
-e PASSWORD=abc123 \
-e REDIS_HOST=redis \
-e API_KEY=xxxxx
```

Use

```
.env
```

```
DB_HOST=mysql
DB_PORT=3306
USER=admin
PASSWORD=abc123
REDIS_HOST=redis
API_KEY=xxxxx
```

Run

```bash
docker run --env-file .env myapp
```

---

## Why Required?

Applications often have

```
20
40
100
```

environment variables.

Managing them on the command line becomes impossible.

---

## Production Problem Solved

Imagine

```
API_URL
JWT_SECRET
SMTP_SERVER
DB_HOST
DB_PORT
REDIS
LOG_LEVEL
FEATURE_FLAGS
```

Keeping them in one file

```
production.env
```

is much easier.

---

## Troubleshooting

Suppose application crashes.

Compare

```
production.env
staging.env
```

Immediately notice

```
REDIS_HOST missing
```

Problem solved.

---

## Analogy

Think of

Recipe Book

Instead of remembering

50 ingredients

everything is written in one notebook.

---

## Test Yourself

1. Why use env-file instead of many `-e`?
2. How does Docker parse env-file?
3. Can comments exist in env-file?

---

# 3. Runtime Configuration

This is one of the most important DevOps concepts.

---

Instead of hardcoding

```
Database = localhost
```

Read it from environment

```
DB_HOST
```

Same application.

Different runtime behavior.

---

Example

Development

```
DB_HOST=localhost
```

Testing

```
DB_HOST=test-db
```

Production

```
DB_HOST=rds.amazonaws.com
```

No rebuild.

---

## Production Benefits

Imagine

100 Kubernetes Pods

Need to switch database.

Only update ConfigMap or Secret.

Pods restart.

Done.

No image rebuild.

---

## Troubleshooting

Application connects to wrong database.

Check

```
kubectl describe pod
```

or

```
docker inspect
```

Find incorrect variable.

---

## Analogy

Phone SIM card.

Same phone.

Insert another SIM.

Works in another network.

---

## Test Yourself

1. Why should images be immutable?
2. Why is runtime configuration preferred?
3. Which changes require rebuilding image?

---

# 4. Secrets vs Environment Variables

Interviewers LOVE this question.

---

Environment Variable

```
LOG_LEVEL=INFO
```

Fine.

---

Password

```
DB_PASSWORD=admin123
```

Not fine.

---

Why?

Environment variables

* visible using

```
docker inspect
```

* can leak into logs
* inherited by child processes
* sometimes visible to debugging tools

---

Better Options

Docker Secrets

Kubernetes Secrets

AWS Secrets Manager

Hashicorp Vault

Azure Key Vault

---

## Production Example

Bad

```
docker run \
-e DB_PASSWORD=mysecret
```

Better

```
AWS Secrets Manager
↓

Application fetches secret

↓

Memory only
```

---

## Troubleshooting

Secret not working.

Check

```
Is secret mounted?

Does application read correct path?

Does IAM permission exist?

Is secret updated?
```

---

## Analogy

Environment variable

Writing ATM PIN on sticky note.

Secret Manager

Keeping ATM PIN inside bank locker.

---

## Test Yourself

1. Why shouldn't passwords be stored as env variables?
2. What alternatives exist?
3. What happens if someone runs `docker inspect`?

---

# 5. `RUN --mount=type=cache`

This is BuildKit.

Not runtime.

Only during image build.

---

Without cache

```
RUN apt update
RUN apt install python3
```

Every build

↓

Downloads everything again.

Slow.

---

With cache

```dockerfile
# syntax=docker/dockerfile:1.7

RUN --mount=type=cache,target=/var/cache/apt \
    apt update && apt install -y curl
```

Now

```
Package cache reused.
```

---

Python Example

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
pip install -r requirements.txt
```

Next build

```
No redownload.
```

---

Go Example

```dockerfile
RUN --mount=type=cache,target=/go/pkg/mod \
go build
```

---

Rust Example

```
cargo cache
```

---

Node

```
npm cache
```

---

## Production Benefits

CI builds become

```
15 minutes

↓

3 minutes
```

Huge savings.

---

## Troubleshooting

Pipeline suddenly slow.

Check

```
Was BuildKit disabled?

Was cache invalidated?

Did cache mount disappear?
```

---

## Analogy

Cooking.

Instead of buying vegetables every day,

store them in refrigerator.

---

## Test Yourself

1. Does cache become part of final image?
2. Is cache available at runtime?
3. Why is cache mount faster?

---

# 6. `RUN --mount=type=secret`

One of BuildKit's best features.

Suppose build needs

```
GitHub Token
```

Never do

```dockerfile
ARG TOKEN
```

because

```
Image History
```

may expose it.

---

Instead

```dockerfile
RUN --mount=type=secret,id=github \
git clone https://private-repo
```

Build

```bash
docker build \
--secret id=github,src=token.txt .
```

Inside build

```
/run/secrets/github
```

exists temporarily.

After build

Gone.

---

## Production Benefits

Need

* private Git repo
* private npm
* private pip
* AWS credentials

No secrets stored in image.

---

## Troubleshooting

Secret unavailable.

Check

```
Did BuildKit enable?

Correct id?

Correct mount path?

Secret file exists?

Correct docker build command?
```

---

## Analogy

Imagine

A delivery person gives you a key.

You open the room.

Return the key.

Nobody else can use it later.

---

## Test Yourself

1. Why is ARG unsafe for secrets?
2. Does secret remain in final image?
3. Where is secret mounted?

---

# Cache vs Secret Mount

| Feature           | Cache Mount     | Secret Mount           |
| ----------------- | --------------- | ---------------------- |
| Purpose           | Speed up builds | Protect sensitive data |
| Runtime Available | ❌               | ❌                      |
| Stored in image   | ❌               | ❌                      |
| Temporary         | Yes             | Yes                    |
| Used by BuildKit  | Yes             | Yes                    |

---

# Environment Variable vs Secret

| Environment Variable | Secret                                             |
| -------------------- | -------------------------------------------------- |
| Configuration        | Sensitive information                              |
| Visible in inspect   | Usually yes                                        |
| Good for DB host     | Yes                                                |
| Good for Password    | No                                                 |
| Kubernetes ConfigMap | Yes                                                |
| Kubernetes Secret    | Yes                                                |
| AWS Secrets Manager  | No (it's a secret store, not an env var mechanism) |

---

# 15 Mid-Level DevOps Interview Questions

### 1. Why should Docker images be immutable?

**Answer:** Images should not change between environments. Configuration should be injected at runtime using environment variables, ConfigMaps, or Secrets so the same tested image can be promoted from development to production.

---

### 2. What is the difference between `ARG` and `ENV`?

**Answer:** `ARG` is available only during image build (unless copied into `ENV`) and is mainly for build-time configuration. `ENV` becomes part of the image metadata and is available when the container runs.

---

### 3. When would you use `-e` instead of rebuilding an image?

**Answer:** When only runtime configuration changes, such as database endpoints, API URLs, feature flags, or log levels.

---

### 4. What is the benefit of `--env-file`?

**Answer:** It centralizes many environment variables into a file, making deployments easier to manage and reducing long `docker run` commands.

---

### 5. Why are secrets different from normal environment variables?

**Answer:** Secrets contain sensitive information like passwords or API keys. Environment variables can be exposed through inspection or logging, while dedicated secret management solutions provide better protection and access control.

---

### 6. How do you inspect environment variables in a running container?

**Answer:**

```bash
docker exec <container> env
```

or

```bash
docker inspect <container>
```

---

### 7. What is BuildKit?

**Answer:** BuildKit is Docker's modern build engine. It improves build performance, supports advanced features such as cache mounts and secret mounts, enables parallel build execution, and provides more efficient layer caching.

---

### 8. Why use `RUN --mount=type=cache`?

**Answer:** To reuse dependency caches between builds, reducing build times in local development and CI/CD pipelines.

---

### 9. Does a cache mount become part of the final image?

**Answer:** No. It exists only during the build process and is not included in the resulting image.

---

### 10. Why use `RUN --mount=type=secret`?

**Answer:** To securely provide temporary credentials during image builds without embedding them into image layers or build history.

---

### 11. Can someone recover secrets passed through `ARG`?

**Answer:** Potentially yes. Build arguments can be exposed in image history, build logs, or layers if used improperly, making them unsuitable for sensitive data.

---

### 12. How would you troubleshoot an application using the wrong configuration?

**Answer:** Inspect the container environment (`docker exec env` or `docker inspect`), verify the `.env` file, confirm deployment manifests, and check application logs to ensure the expected values are being read.

---

### 13. How would you speed up Docker builds in a CI/CD pipeline?

**Answer:** Enable BuildKit, use `RUN --mount=type=cache`, optimize layer ordering, use multi-stage builds, and leverage remote or local build caches.

---

### 14. How are runtime configuration and build-time configuration different?

**Answer:** Build-time configuration affects how the image is created (`ARG`), while runtime configuration affects how the application behaves after the container starts (`ENV`, `-e`, ConfigMaps, Secrets).

---

### 15. In Kubernetes, what is the equivalent of Docker environment variables?

**Answer:** Kubernetes provides **ConfigMaps** for non-sensitive configuration and **Secrets** for sensitive data. These can be injected as environment variables or mounted as files inside Pods.

---

## What interviewers are really testing

At the mid-level, interviewers aren't looking for someone who can just use `docker run -e`. They want to know whether you understand:

* **Image immutability** (build once, deploy everywhere)
* **Separation of code and configuration**
* **Secure handling of secrets** (avoid baking credentials into images)
* **Build optimization** with BuildKit and cache mounts
* **Troubleshooting configuration issues** by inspecting environment variables and build settings
* **How these concepts translate to Kubernetes** using ConfigMaps, Secrets, and GitOps workflows

Being able to explain **why** these practices matter in production is often more important than remembering the exact command syntax.
