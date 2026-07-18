The **12-Factor App** is a set of **12 best practices** for building **cloud-native**, **portable**, **scalable**, and **maintainable** applications.

It was introduced by engineers at **Heroku** in 2011 because developers were facing common problems:

* "Works on my machine."
* Difficult deployments.
* Configuration hardcoded in code.
* Scaling applications was difficult.
* Different environments (dev, staging, production) behaved differently.

Today, even though it was written years ago, it is still one of the most important principles behind:

* Docker
* Kubernetes
* AWS ECS/EKS
* CI/CD pipelines
* Microservices
* Cloud-native applications

If you're preparing for a **Mid-Level DevOps interview**, recruiters expect you to understand **why Docker and Kubernetes work so well with applications designed according to the 12-Factor principles.**

---

# Why was 12-Factor created?

Imagine a company.

Developer writes:

```
DATABASE_HOST=localhost
```

inside Python code.

Everything works.

Now application goes to production.

Production database is

```
database.company.internal
```

Now developer edits code.

Deploys again.

Tomorrow staging has another database.

Again edit code.

Tomorrow QA server changes.

Again edit code.

Soon everyone is changing source code just because configuration changes.

Huge mess.

12-Factor says:

> Application code should never change between environments.

Only configuration should change.

---

# What problems does it solve?

It solves problems like:

* Configuration management
* Scaling
* CI/CD
* Cloud deployment
* Docker deployment
* Kubernetes deployment
* Microservices communication
* Faster onboarding
* Easier disaster recovery
* Consistent environments

---

# The 12 Factors

---

# 1. Codebase

> One codebase tracked in version control.

Example:

```
Git Repository

Retail-App
    Cart
    Product
    Payment
```

Good

```
GitHub

Retail-App
```

Bad

```
Cart-Code
Cart-Code-New
Cart-Code-Latest
Cart-Final
Cart-Production
```

---

## DevOps importance

CI/CD always knows

```
Which repository to build?
```

Simple.

---

# 2. Dependencies

Every dependency should be declared explicitly.

Python

```
requirements.txt
```

```
Flask==3.0
requests==2.32
boto3==1.36
```

NodeJS

```
package.json
```

Java

```
pom.xml
```

Never assume

> "Python package already installed."

---

Docker example

Instead of

```
pip install flask
pip install requests
pip install boto3
```

Use

```dockerfile
COPY requirements.txt .

RUN pip install -r requirements.txt
```

---

Benefits

* reproducible builds
* easier Docker images
* easier CI

---

# 3. Config

Configuration should be stored outside code.

BAD

```python
DB_HOST="localhost"
DB_PASSWORD="admin123"
```

Good

```python
DB_HOST=os.getenv("DB_HOST")
```

Docker

```
docker run \
-e DB_HOST=mysql \
-e DB_USER=root \
-e DB_PASSWORD=secret
```

Kubernetes

```
ConfigMap
Secret
```

AWS

```
Secrets Manager

SSM Parameter Store
```

---

Why?

Same image can run everywhere.

```
Dev

DB_HOST=dev-db
```

Production

```
DB_HOST=prod-db
```

No rebuild required.

---

Interview question

**Why shouldn't configuration be stored inside Docker images?**

Answer

Because image should remain immutable.

Configuration changes frequently.

Image should not.

---

# 4. Backing Services

Treat databases, caches and queues as attached resources.

Example

```
Postgres

Redis

RabbitMQ

Kafka

S3
```

Application should connect through configuration.

Instead of

```
localhost
```

Use

```
DATABASE_URL
```

Example

```
DATABASE_URL=postgres://...
```

Tomorrow

Replace PostgreSQL with RDS.

Application shouldn't change.

Only environment variable changes.

---

# 5. Build, Release, Run

These three stages should always remain separate.

Example

## Build

```
docker build
```

Produces

```
myapp:v1
```

---

Release

Combine

```
Image

Environment Variables

Secrets

ConfigMaps
```

Now

```
Release v1
```

---

Run

Container starts.

```
docker run

kubectl apply
```

---

Why?

If release fails

You don't rebuild image.

Just rollback release.

---

CI/CD

GitHub Actions

```
Build

↓

Push ECR

↓

Deploy

↓

Run
```

Exactly follows 12-factor.

---

# 6. Processes

Application should be stateless.

Never store important data inside container.

Bad

```
/tmp/orders.db
```

Container dies

Everything gone.

Good

```
Postgres

S3

Redis
```

Containers become disposable.

---

Docker connection

This is why we use

Volumes

PVC

EBS

RDS

instead of writing data inside container filesystem.

---

# 7. Port Binding

Application should expose service through a port.

Example

Flask

```python
app.run(port=5000)
```

Node

```
3000
```

Java

```
8080
```

Docker

```
5000
```

Then

```
docker run -p 8080:5000
```

Container exposes service itself.

Not through Apache installed separately.

---

# 8. Concurrency

Scale by adding more processes.

Instead of

One huge server

```
Users
      |
   App
```

Use

```
Users

↓

Load Balancer

↓

Container 1

Container 2

Container 3

Container 4
```

Kubernetes

```
ReplicaSet

Deployment

Horizontal Pod Autoscaler
```

This directly follows 12-factor.

---

# 9. Disposability

Containers should

* Start fast
* Stop gracefully

Why?

Kubernetes constantly creates and destroys Pods.

Application should not need

10 minutes to shut down.

Python

Handle

```
SIGTERM
```

Gracefully.

Instead of

```
kill -9
```

---

Docker

```
STOPSIGNAL

HEALTHCHECK
```

exist because of this.

---

# 10. Dev/Prod Parity

Development and production should be as similar as possible.

Bad

Developer

```
SQLite
```

Production

```
PostgreSQL
```

Bad

Developer

Windows

Production

Linux

Bad

Developer

Python 3.10

Production

Python 3.13

Good

Docker

Same image

Everywhere.

---

This is why containers became so popular.

---

# 11. Logs

Application should never manage log files itself.

Instead

Write logs to

```
stdout

stderr
```

Docker captures

```
docker logs
```

Kubernetes captures

```
kubectl logs
```

Then

Promtail

↓

Loki

↓

Grafana

or

Fluent Bit

↓

CloudWatch

---

Don't do

```
app.log
```

inside container.

Container may disappear.

---

# 12. Admin Processes

Administrative tasks should run as one-off processes using the same codebase and environment.

Example

Database migration

```
python migrate.py
```

Instead of SSHing into the server and manually changing the database, run a one-off process using the same application image and configuration. Examples include:

```bash
docker run --rm myapp python migrate.py
```

or in Kubernetes:

```bash
kubectl create job db-migration --image=myapp:v1
```

This ensures migrations, data fixes, or batch jobs run with the same dependencies and environment as the application.

---

# How Docker supports the 12-Factor App

| 12-Factor Principle | Docker Feature                                     |
| ------------------- | -------------------------------------------------- |
| Codebase            | Git repository + Dockerfile                        |
| Dependencies        | Docker image packages dependencies                 |
| Config              | `-e`, `--env-file`, Secrets                        |
| Backing Services    | Environment variables for DB, Redis, Kafka         |
| Build/Release/Run   | `docker build` → push image → `docker run`         |
| Processes           | Stateless containers, volumes for persistent data  |
| Port Binding        | `EXPOSE`, `-p`                                     |
| Concurrency         | Multiple container replicas                        |
| Disposability       | Fast startup/shutdown, `STOPSIGNAL`, health checks |
| Dev/Prod Parity     | Same image in all environments                     |
| Logs                | `stdout`/`stderr`, `docker logs`                   |
| Admin Processes     | One-off containers or Kubernetes Jobs              |

# How Kubernetes supports the 12-Factor App

| Principle         | Kubernetes Feature                                                                |
| ----------------- | --------------------------------------------------------------------------------- |
| Config            | ConfigMaps, Secrets                                                               |
| Processes         | Pods are treated as ephemeral; persistent data lives in PVCs or external services |
| Concurrency       | Deployments, ReplicaSets, HPA                                                     |
| Logs              | `kubectl logs`, centralized logging stacks                                        |
| Disposability     | Liveness/Readiness probes, graceful termination, rolling updates                  |
| Dev/Prod Parity   | Same container image across dev, staging, and production                          |
| Build/Release/Run | CI builds the image, CD deploys manifests or Helm charts                          |

# Interview questions you may be asked

**Q1. Why are environment variables preferred over hardcoded configuration?**

**Answer:** They keep the application image immutable, allow the same image to run in different environments, and avoid rebuilding the application when only configuration changes.

**Q2. Why should containers be stateless?**

**Answer:** Containers are ephemeral and may be rescheduled or replaced at any time. Persistent data should be stored in external databases, object storage, or mounted volumes so application instances can scale and recover safely.

**Q3. Why should applications log to `stdout` instead of files?**

**Answer:** Containers can be deleted at any time, taking local log files with them. Logging to `stdout`/`stderr` lets Docker and Kubernetes collect logs and forward them to centralized systems like Loki, CloudWatch, or Elasticsearch.

**Q4. Why separate Build, Release, and Run?**

**Answer:** It produces immutable artifacts, enables repeatable deployments, simplifies rollbacks, and keeps the deployment process predictable across environments.

---

## How this relates to your DevOps preparation

The projects you've been building already align with many of these principles:

* **Environment variables, ConfigMaps, and Secrets** → **Factor 3 (Config)**
* **AWS Secrets Manager + External Secrets Operator** → **Factors 3 & 4**
* **Helm + ArgoCD deployments** → **Factor 5 (Build, Release, Run)**
* **EBS-backed Persistent Volumes for PostgreSQL** → **Factor 6 (Processes)**
* **Kubernetes Deployments and ReplicaSets** → **Factor 8 (Concurrency)**
* **Prometheus/Loki/Grafana log and metrics collection** → **Factor 11 (Logs)**

Understanding *why* these practices exist—not just how to configure them—is exactly what interviewers look for in mid-level DevOps engineers.
