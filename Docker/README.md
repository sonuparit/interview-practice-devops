This roadmap is specifically for a **Mid-Level DevOps Engineer** interviewing with international companies (US, Europe, Australia, remote startups). It's not just about knowing Docker commands—it's about understanding how containers behave in production and how to troubleshoot them.

---

# Docker Roadmap for Mid-Level DevOps

## 1. Container Fundamentals

Understand:

* What is a container?
* Why containers exist
* Containers vs Virtual Machines
* Docker architecture
* Docker Engine
* Docker Desktop
* Docker Daemon
* Docker Client
* Docker Registry
* OCI standards
* Container lifecycle

---

## 2. Docker Installation & Architecture

Know:

* Docker Engine
* dockerd
* containerd
* runc
* Docker CLI

Understand the flow:

```
docker CLI
     ↓
dockerd
     ↓
containerd
     ↓
runc
     ↓
Linux Kernel
```

---

## 3. Docker Images

Topics

* Image
* Layer
* Base image
* Parent image
* Scratch image
* Immutable images
* Image manifest
* Image digest
* Image tags

Commands

* docker images
* docker pull
* docker push
* docker tag
* docker inspect
* docker history
* docker save
* docker load
* docker export
* docker import
* docker commit

Understand

* Layer caching
* Copy-on-write
* Content-addressable storage

---

## 4. Docker Containers

Commands

* docker run
* docker create
* docker start
* docker stop
* docker restart
* docker pause
* docker unpause
* docker rm
* docker ps
* docker exec
* docker attach
* docker logs
* docker inspect

Understand

Container lifecycle

```
Created

↓

Running

↓

Paused

↓

Stopped

↓

Removed
```

---

## 5. Dockerfile

Must master.

Instructions

* FROM
* RUN
* CMD
* ENTRYPOINT
* COPY
* ADD
* WORKDIR
* ENV
* ARG
* LABEL
* USER
* EXPOSE
* HEALTHCHECK
* VOLUME
* SHELL
* STOPSIGNAL
* ONBUILD

Understand

* Build cache
* Layer creation
* Image optimization

---

## 6. Multi-stage Builds

Topics

* Builder stage
* Runtime stage
* Smaller images
* Removing build dependencies

Know when and why to use it.

---

## 7. Docker Build

Commands

* docker build
* docker buildx
* docker builder prune

Understand

* Build context
* .dockerignore
* Build cache
* BuildKit
* Multi-platform builds

---

## 8. Docker Volumes

Topics

* Named volumes
* Anonymous volumes
* Bind mounts
* tmpfs

Commands

* docker volume ls
* docker volume create
* docker volume inspect
* docker volume rm

Understand

* Persistent storage
* Volume lifecycle
* Data persistence

---

## 9. Bind Mounts

Understand

* Development workflow
* Production risks
* File permissions
* SELinux implications (high level)

---

## 10. Docker Networking

Very important.

Drivers

* bridge
* host
* none
* overlay
* macvlan

Commands

* docker network ls
* docker network create
* docker network inspect
* docker network connect
* docker network disconnect

Understand

* DNS inside Docker
* Service discovery
* Port mapping
* NAT
* Bridge networking
* Container-to-container communication

---

## 11. Environment Variables

Topics

* -e
* --env-file

Understand

* Runtime configuration
* Secrets vs environment variables

---

## 12. Docker Compose

Must know thoroughly.

Compose file

Topics

* services
* networks
* volumes
* depends_on
* restart
* healthcheck
* environment
* build
* image
* ports
* profiles

Commands

* docker compose up
* docker compose down
* docker compose logs
* docker compose ps
* docker compose exec
* docker compose restart

Understand

* Multi-container applications
* Dependency management
* Development workflow

---

## 13. Docker Registry

Topics

* Docker Hub
* Private Registry
* AWS ECR
* GCR
* GHCR

Commands

* docker login
* docker logout
* docker push
* docker pull

Understand

* Authentication
* Image versioning
* Registry caching

---

## 14. Image Optimization

Interview favorite.

Know

* Alpine vs Debian vs Ubuntu
* Distroless images
* Minimal images
* Multi-stage builds
* Removing unnecessary packages
* Layer optimization
* COPY ordering
* Build cache optimization

Understand

* Image size
* Startup time
* Attack surface

---

## 15. Container Security

Must know.

Topics

* Non-root containers
* USER instruction
* Capabilities
* Read-only filesystem
* Secrets
* Image scanning
* Docker Bench
* Seccomp
* AppArmor
* SELinux (high level)

---

## 16. Resource Management

Commands

* --cpus
* --memory
* --memory-swap
* --pids-limit

Understand

* CPU limits
* Memory limits
* OOM killer
* cgroups

---

## 17. Logging

Commands

* docker logs

Drivers

* json-file
* journald
* syslog
* fluentd
* awslogs

Understand

* Log rotation
* Centralized logging

---

## 18. Health Checks

Topics

Dockerfile

```
HEALTHCHECK
```

Compose

```
healthcheck:
```

Understand

* Healthy
* Unhealthy
* Starting

---

## 19. Docker Inspect

Must master.

Commands

* docker inspect

Know how to inspect

* Networks
* Volumes
* Mounts
* IP
* Environment
* Labels
* Health
* Entrypoint

---

## 20. Debugging Containers

Commands

* docker exec
* docker logs
* docker inspect
* docker stats
* docker top
* docker diff
* docker events

Know how to debug

* Container exits immediately
* CrashLoop
* Network issues
* Missing files
* Permission denied
* DNS failure

---

## 21. Docker Storage

Understand

* Storage drivers
* overlay2
* Copy-on-write
* Writable layer
* Read-only layers

---

## 22. Docker Cleanup

Commands

* docker system prune
* docker image prune
* docker container prune
* docker volume prune
* docker network prune

Understand

* Dangling images
* Unused resources

---

## 23. Docker Performance

Topics

* Build speed
* Image size
* Layer caching
* Startup time
* Volume performance

---

## 24. Docker Internals

Mid-level interview topic.

Understand

* Namespaces
* cgroups
* OverlayFS
* Union filesystem
* PID namespace
* Network namespace
* Mount namespace
* IPC namespace

Understand how Docker uses Linux features.

---

## 25. Docker Security Best Practices

Know

* Don't run as root
* Use minimal images
* Pin image versions
* Scan images
* Keep images updated
* Read-only filesystem
* Limit capabilities
* Secrets management
* Drop unnecessary privileges

---

## 26. Docker in CI/CD

Know

* Build images
* Tag strategy
* Push to registry
* Versioning
* Cache optimization
* Multi-stage pipeline

---

## 27. Production Docker

Understand

* Immutable deployments
* Stateless containers
* Externalized configuration
* Persistent storage
* Health checks
* Graceful shutdown
* Restart policies
* Logging strategy
* Monitoring

---

## 28. Docker Troubleshooting

This is where many mid-level interviews focus.

Be able to troubleshoot:

* Container won't start
* Container exits immediately
* Port already allocated
* Image pull failure
* Registry authentication failure
* Disk full
* High CPU
* High memory
* OOMKilled
* Container networking issues
* DNS resolution failure
* Volume mount problems
* Permission denied
* Read-only filesystem
* Slow image builds
* Large image size
* Build cache not working
* Health check failures
* Restart loops
* Missing environment variables
* Broken bind mounts

---

## 29. Docker Best Practices

Know why these matter:

* One process per container (general guideline)
* Use `.dockerignore`
* Multi-stage builds
* Pin image versions
* Avoid `latest` tag in production
* Keep images small
* Use non-root users
* Minimize layers where appropriate
* Cache dependencies effectively
* Separate build-time and runtime dependencies
* Avoid storing secrets in images

---

## 30. Advanced Docker Concepts (Mid-Level)

Understand conceptually:

* BuildKit
* Docker Buildx
* Multi-architecture images
* Manifest lists
* Rootless Docker
* Docker contexts
* Docker API
* Docker socket (`/var/run/docker.sock`) and its security implications

---

# Priority for Mid-Level DevOps Interviews

## ⭐⭐⭐⭐⭐ Master these

1. Dockerfile
2. Image Layers & Build Cache
3. Container Lifecycle
4. Docker Networking
5. Volumes & Bind Mounts
6. Docker Compose
7. Multi-stage Builds
8. Production Troubleshooting
9. Container Security Basics
10. Image Optimization

---

## ⭐⭐⭐⭐ Know well

11. Docker Internals
12. BuildKit
13. Registries (especially AWS ECR)
14. Logging
15. Health Checks
16. Resource Limits
17. Docker Storage
18. CI/CD Integration

---

## ⭐⭐⭐ Understand conceptually

19. Rootless Docker
20. Multi-architecture Builds
21. Custom Network Drivers
22. Docker API
23. Storage Drivers
24. Manifest Lists

---

### Tailored to your experience

Based on the projects you've already built, you're ahead of many mid-level candidates in several areas. You already have hands-on experience with:

* Reverse-engineering and writing production-ready Dockerfiles.
* Optimizing images (e.g., reducing image size significantly using minimal base images).
* Creating and managing multi-service applications with Docker Compose.
* Integrating Docker into GitHub Actions and GitOps workflows.
* Using Docker as the foundation for Kubernetes deployments.

For interviews, I'd recommend shifting your preparation from "How do I use Docker?" to "How do I diagnose and fix Docker problems in production?" That's the level of discussion international mid-level DevOps interviews typically expect.
