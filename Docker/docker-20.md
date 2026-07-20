This is one of the most practical DevOps interview topics. Many Kubernetes/Docker production incidents boil down to **"the container isn't running"**. Interviewers often don't care if you know every Docker command—they want to know **how you think while debugging**.

---

# 1. Container exits immediately

## What is it and why is it required?

Normally, a container keeps running because its **main process (PID 1)** is alive.

When that process finishes, the container stops.

Example:

```dockerfile
CMD ["echo", "Hello"]
```

Output:

```
Hello
```

Container exits.

Another example:

```dockerfile
CMD ["python", "script.py"]
```

If script.py finishes in 2 seconds:

```
Container Started
↓
Script Finished
↓
Container Stopped
```

This is expected behavior.

---

## Why is this important in DevOps?

Very common production issue.

Imagine your company deploys

```
Backend API
```

Expected:

```
Running 24x7
```

Instead

```
Running
↓

Exit
↓

Restart
↓

Exit
↓

Restart
```

Application never becomes available.

---

## How do you troubleshoot?

Check status

```bash
docker ps -a
```

Example

```
Exited (0)
```

means successful exit.

```
Exited (1)
```

means application crashed.

See logs

```bash
docker logs my-container
```

Example

```
Database connection failed
```

or

```
Configuration file missing
```

Inspect

```bash
docker inspect my-container
```

Check

* Entrypoint
* CMD
* ExitCode

---

### Example

Developer forgot to start Node app.

Dockerfile

```dockerfile
CMD ["npm"]
```

Instead of

```dockerfile
CMD ["npm","start"]
```

Container immediately exits.

---

## Analogy

Think of a taxi.

Driver starts engine.

Immediately reaches destination.

Turns engine off.

Taxi isn't broken.

Its job simply finished.

Container behaves the same way.

---

## Questions

Can a container exit with code 0?

**Yes.**
It means successful completion.

---

What keeps a container alive?

The main process (PID 1).

---

How do you know why it exited?

```
docker logs

docker inspect

docker ps -a
```

---

# 2. CrashLoop (CrashLoopBackOff)

This is Kubernetes.

---

## What is it?

Application starts.

Crashes.

Kubernetes restarts it.

Crashes again.

Keeps repeating.

```
Start

↓

Crash

↓

Restart

↓

Crash

↓

Restart
```

Eventually

```
CrashLoopBackOff
```

---

## Why is it required?

Kubernetes tries to recover failed applications automatically.

Instead of giving up,

it retries.

---

## Production example

Application requires

```
DATABASE_URL
```

Developer forgot environment variable.

App starts

↓

Cannot connect

↓

Exit

↓

Restart

↓

Exit

↓

CrashLoopBackOff

---

## Troubleshooting

Check

```bash
kubectl get pods
```

```
CrashLoopBackOff
```

Logs

```bash
kubectl logs pod-name
```

Previous logs

```bash
kubectl logs pod-name --previous
```

Describe

```bash
kubectl describe pod pod-name
```

See

* Exit code
* Events
* OOMKilled
* Failed Mount
* Failed Scheduling

---

### Example

```
panic:

config.yml not found
```

Problem solved by mounting ConfigMap.

---

## Analogy

Imagine an employee.

Boss says

```
Go start machine.
```

Employee starts machine.

Machine immediately explodes.

Boss says

```
Try again.
```

Same thing happens.

Eventually boss stops asking.

That's CrashLoopBackOff.

---

## Questions

Difference between Error and CrashLoopBackOff?

Error means container failed once.

CrashLoopBackOff means repeated restart failures.

---

How do you see previous crash logs?

```bash
kubectl logs pod --previous
```

---

Common causes?

* Missing env
* Missing Secret
* Missing ConfigMap
* OOMKilled
* Wrong command
* Application bug

---

# 3. IP

---

## What is it?

Every container gets an IP.

Docker

```
172.x.x.x
```

Kubernetes

Pod IP

```
10.x.x.x
```

Used for communication.

---

## Why important?

Microservices communicate using IP/network.

Example

```
Frontend

↓

Backend

↓

Redis

↓

Database
```

Wrong IP

↓

Connection fails.

---

## Troubleshooting

Find IP

Docker

```bash
docker inspect container
```

Kubernetes

```bash
kubectl get pod -o wide
```

or

```bash
kubectl describe pod
```

---

Example

Application connects to

```
10.10.5.3
```

Database moved.

Now IP changed.

Connection fails.

Solution

Never hardcode IP.

Use DNS/service.

---

## Analogy

IP is like a house number.

Without it,

postman cannot deliver letters.

---

## Questions

Do Pods keep same IP forever?

No.

---

Should applications hardcode Pod IP?

Never.

---

How do Pods usually communicate?

Using Kubernetes Services (DNS).

---

# 4. Network issues

---

## What is it?

Application cannot communicate.

Example

```
Frontend

↓

Backend

×

Connection refused
```

---

## Why important?

Nearly every production issue involves networking.

---

## Troubleshooting

Check

Container running?

```bash
docker ps
```

Ping

```bash
ping
```

DNS

```bash
nslookup
```

Port

```bash
nc
```

or

```bash
telnet
```

Listening ports

```bash
ss -tulnp
```

Curl

```bash
curl
```

Kubernetes

```
Service

Endpoints

NetworkPolicy

Ingress

```

---

Example

Service listens on

```
8080
```

Ingress forwards

```
80
```

Wrong targetPort.

Traffic never reaches app.

---

## Analogy

Telephone line exists.

Person exists.

Wrong phone number.

Call never reaches.

---

## Questions

Difference between timeout and connection refused?

Timeout

No response.

Connection refused

Target reachable but nothing listening.

---

What command checks listening ports?

```
ss -tulnp
```

---

Why use curl?

To test HTTP communication.

---

# 5. Missing files

---

## What is it?

Application expects

```
config.yml
```

File absent.

Application crashes.

---

## Production example

Secret wasn't mounted.

ConfigMap missing.

PVC failed.

---

## Troubleshooting

Inside container

```bash
docker exec -it container sh
```

or

```bash
kubectl exec -it pod -- sh
```

Check

```bash
ls

pwd

find
```

Verify mount

```bash
mount
```

or

```bash
kubectl describe pod
```

---

Example

```
/etc/config/app.yml
```

Missing.

Reason

ConfigMap typo.

---

## Analogy

Imagine cooking.

Recipe says

```
Use sugar.
```

Kitchen has no sugar.

Recipe fails.

---

## Questions

How do you verify mounted files?

```
kubectl exec

ls
```

---

Where do ConfigMaps appear?

Mounted as files or environment variables.

---

How to verify volume mounted?

```
kubectl describe pod
```

---

# 6. Permission denied

---

## What is it?

Application lacks permission.

Example

```
Permission denied
```

Very common.

---

## Production example

Container runs

```
UID 1001
```

Mounted volume owned by

```
root
```

Application cannot write.

---

## Troubleshooting

Check

```bash
ls -l
```

Owner

Permissions

User

```bash
id
```

Dockerfile

```
USER
```

Kubernetes

```
securityContext
```

---

Example

```
chmod 600 config
```

Application user cannot read.

---

## Analogy

Locked office.

You have employee card.

Door requires manager card.

Permission denied.

---

## Questions

How do you check current user?

```bash
id
```

---

How to see permissions?

```bash
ls -l
```

---

Common Kubernetes cause?

securityContext/fsGroup mismatch.

---

# 7. DNS failure

---

## What is it?

Application cannot resolve hostname.

Example

```
database.company.local
```

↓

Unknown host.

---

## Why important?

Modern systems communicate using DNS.

Without DNS

Nothing connects.

---

## Troubleshooting

Check

```bash
nslookup
```

or

```bash
dig
```

Check

```
/etc/resolv.conf
```

Docker

```bash
docker exec
```

Kubernetes

```bash
kubectl exec
```

Test

```bash
ping google.com
```

or

```bash
curl
```

---

Example

Application tries

```
postgres.default.svc.cluster.local
```

CoreDNS crashed.

DNS resolution fails.

---

## Analogy

You know someone's name.

But phone directory is missing.

You cannot find phone number.

DNS is that directory.

---

## Questions

How to test DNS?

```
nslookup

dig
```

---

Where does Linux keep DNS configuration?

```
/etc/resolv.conf
```

---

Can application communicate without DNS?

Only if IP is known.

---

# Production Troubleshooting Flow

Whenever a container is failing:

```
Container Running?

↓

docker ps
kubectl get pods

↓

Logs

docker logs
kubectl logs

↓

Inspect

docker inspect
kubectl describe pod

↓

Exec inside

docker exec
kubectl exec

↓

Check

Processes

Files

Permissions

Environment variables

↓

Network

IP

DNS

Ports

Connectivity

↓

Resources

CPU

Memory

OOMKilled

↓

Application configuration
```

This systematic approach prevents random guessing and helps isolate the root cause efficiently.

---

# 15 Mid-Level DevOps Interview Questions with Answers

### 1. Why does a container stop when the application exits?

Because the container's lifecycle is tied to its main process (PID 1). When that process exits, Docker considers the container finished and stops it.

---

### 2. What is the difference between `Exited (0)` and `Exited (1)`?

* **Exit code 0:** The process completed successfully.
* **Non-zero exit code (e.g., 1):** The process terminated due to an error.

---

### 3. What is `CrashLoopBackOff`?

A Kubernetes state where a container repeatedly starts, crashes, and is restarted. Kubernetes gradually increases the delay between restart attempts (backoff).

---

### 4. Which commands do you use first when a Pod is in `CrashLoopBackOff`?

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

---

### 5. Why shouldn't applications use Pod IPs directly?

Pod IPs are ephemeral and change when Pods are recreated. Applications should communicate through Kubernetes Services and DNS names.

---

### 6. How do you determine a Pod's IP address?

```bash
kubectl get pods -o wide
```

or

```bash
kubectl describe pod <pod-name>
```

---

### 7. What's the difference between a connection timeout and "Connection refused"?

* **Timeout:** The client received no response (network issue, firewall, routing, or dropped packets).
* **Connection refused:** The destination is reachable, but no process is listening on the target port.

---

### 8. How do you verify that a process is listening on the expected port?

```bash
ss -tulnp
```

or

```bash
netstat -tulnp
```

(when available).

---

### 9. How do you inspect files inside a running container?

```bash
docker exec -it <container> sh
```

or

```bash
kubectl exec -it <pod> -- sh
```

---

### 10. A Pod fails because a configuration file is missing. What could cause this?

Common causes include:

* ConfigMap not mounted.
* Secret not mounted.
* Incorrect `mountPath`.
* Typo in the volume or volumeMount definition.
* Volume mount failure.

---

### 11. How do you troubleshoot a "Permission denied" error?

Check:

* Current user (`id`)
* File ownership (`ls -l`)
* File permissions
* `USER` directive in the Dockerfile
* Kubernetes `securityContext` (e.g., `runAsUser`, `fsGroup`)

---

### 12. Why do containers often run as non-root users?

For security. Running as a non-root user limits the impact of a compromise and follows the principle of least privilege.

---

### 13. How do you troubleshoot DNS resolution failures inside a container?

Run:

```bash
nslookup <hostname>
```

or

```bash
dig <hostname>
```

Inspect `/etc/resolv.conf`, verify CoreDNS (in Kubernetes), and test connectivity to known domains.

---

### 14. What information does `kubectl describe pod` provide that `kubectl logs` does not?

`kubectl describe pod` includes:

* Events
* Scheduling failures
* Image pull errors
* Volume mount failures
* Liveness/readiness probe failures
* Exit codes
* OOMKilled status

`kubectl logs` only shows the application's stdout/stderr output.

---

### 15. Walk me through your troubleshooting process when a production Pod isn't working.

A strong answer is:

1. Check Pod status (`kubectl get pods`).
2. Read application logs (`kubectl logs`, `--previous` if needed).
3. Inspect Pod events (`kubectl describe pod`).
4. Verify resource issues (OOMKilled, CPU/memory limits).
5. Check environment variables, ConfigMaps, and Secrets.
6. Verify mounted volumes and required files.
7. Confirm permissions and the runtime user.
8. Test networking (Service, Endpoints, ports).
9. Verify DNS resolution.
10. Reproduce the issue by executing into the container if necessary.
11. Identify the root cause, apply the fix, and monitor the rollout to ensure stability.

This structured methodology demonstrates the systematic debugging approach expected from a mid-level DevOps engineer.
