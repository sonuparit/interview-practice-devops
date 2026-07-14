This is one of the most important Docker topics for a Mid-level DevOps Engineer because **every container communicates through Docker networking**. Most production issues are actually networking issues rather than application issues.

Think of Docker networking as learning **how containers talk to each other, to the host, and to the outside world.**

---

# 1. Bridge Network

## What is it?

Bridge is Docker's **default network driver**.

Whenever you run

```bash
docker run nginx
```

Docker automatically connects that container to a bridge network called

```
bridge
```

You can see it

```bash
docker network ls
```

Example

```
NETWORK ID     NAME      DRIVER
abc123         bridge    bridge
```

---

## Why is it required?

Without networking,

* containers cannot access internet
* containers cannot talk to each other
* users cannot access containers

Bridge network solves all of this.

---

## How does it work?

Docker creates a virtual Linux bridge.

Imagine

```
             Linux Host

          docker0 (Bridge)
        +----------------+
        |                |
        |                |
     Container A      Container B
     172.17.0.2      172.17.0.3
```

The bridge behaves exactly like a physical network switch.

---

## Production Use

Most single-host applications use bridge networking.

Example

```
Frontend
Backend
Redis
Postgres
```

All running on one VM.

They communicate over bridge network.

---

## Troubleshooting

Suppose

```
Frontend cannot connect Backend
```

Check

```bash
docker network inspect bridge
```

Is backend attached?

Check

```bash
docker inspect backend
```

Check IP

```
172.17.0.3
```

Ping

```bash
docker exec frontend ping 172.17.0.3
```

---

## Analogy

Imagine a WiFi router.

Every phone connected to WiFi can communicate.

Docker bridge = WiFi router

Containers = Phones

---

## Test Yourself

1.

Why is bridge the default Docker network?

---

2.

Can containers communicate without publishing ports?

Answer:

Yes.

Containers on same bridge network communicate directly.

---

3.

Can outside users access bridge network directly?

No.

Need port mapping.

---

# 2. Host Network

## What is it?

Container shares host's network stack.

No separate IP.

```
Host IP

192.168.1.10

Container also uses

192.168.1.10
```

Run

```bash
docker run --network host nginx
```

---

## Why?

Removes Docker networking overhead.

Useful for

* high performance
* monitoring
* packet capture

---

## Production

Example

Prometheus Node Exporter

Needs direct access to host networking.

Host networking is common.

---

## Troubleshooting

Problem

```
Container cannot bind port 80
```

Reason

Host already using port 80.

Check

```bash
ss -tulnp
```

---

## Analogy

Instead of renting your own office,

you work inside the company's main office.

No private room.

---

## Test Yourself

1.

Does host networking create container IP?

No.

---

2.

Can two host network containers use same port?

No.

---

3.

When is host networking preferred?

Very high performance or host-level monitoring.

---

# 3. None Network

## What?

No networking.

```
No IP

No Internet

No DNS

Nothing
```

Run

```bash
docker run --network none alpine
```

---

## Why?

Maximum isolation.

---

## Production

Useful for

* security
* offline batch jobs
* malware analysis

---

## Troubleshooting

If application cannot reach internet

Check

```bash
docker inspect container
```

Network mode may be

```
none
```

---

## Analogy

Person locked inside a room.

No phone.

No internet.

---

## Test Yourself

1.

Can it ping internet?

No.

---

2.

Does it get IP?

No.

---

3.

Why use it?

Security.

---

# 4. Overlay Network

## What?

Allows containers on different Docker hosts to communicate.

```
Server A

Container A

↓

Overlay

↓

Server B

Container B
```

---

## Why?

Bridge only works on one machine.

Overlay connects multiple machines.

---

## Production

Docker Swarm

Kubernetes (similar concept through CNI)

Microservices across multiple nodes.

---

## Troubleshooting

Container cannot reach service on another node.

Check

```bash
docker network inspect
```

Check

```
VXLAN

Firewall

Ports

4789
7946
2377
```

---

## Analogy

Bridge = Inside one building

Overlay = Metro connecting many buildings

---

## Test Yourself

1.

Does bridge work across servers?

No.

---

2.

Why overlay?

Multi-host communication.

---

3.

Which technology carries overlay traffic?

VXLAN.

---

# 5. Macvlan

## What?

Container gets its own MAC address.

Appears as physical device.

```
Router

↓

Container

Own MAC

Own IP
```

---

## Why?

Some legacy applications require real LAN identity.

---

## Production

Industrial equipment

IoT

DHCP servers

Network appliances

---

## Troubleshooting

Container unreachable.

Check

Switch supports multiple MACs.

Check VLAN configuration.

---

## Analogy

Instead of apartment number,

container gets its own house.

---

## Test Yourself

1.

Does macvlan create separate MAC?

Yes.

---

2.

Can router see container?

Yes.

---

3.

Common use?

Legacy network applications.

---

# 6. DNS Inside Docker

## What?

Docker provides built-in DNS.

Instead of

```
172.18.0.5
```

Containers use

```
redis
```

---

Example

```
Frontend

↓

redis

↓

Docker DNS

↓

172.18.0.4
```

---

Why?

IPs change.

Names remain.

---

Production

```
DATABASE_URL

postgres:5432
```

instead of

```
172.18.0.8
```

---

Troubleshooting

Check

```bash
docker exec app nslookup redis
```

or

```bash
cat /etc/resolv.conf
```

---

Analogy

Phone contacts.

You remember

```
Mom
```

not

```
9876543210
```

---

Test Yourself

1.

Who provides DNS?

Docker.

---

2.

Why use names?

IPs change.

---

3.

Can default bridge resolve container names?

Only user-defined bridge networks provide automatic container-name DNS resolution. The built-in default `bridge` network does **not** automatically resolve container names unless you use the legacy `--link` feature.

---

# 7. Service Discovery

## What?

Finding services automatically.

Instead of

```
10.0.2.15
```

Use

```
database
```

---

Production

Microservices

```
API

↓

Redis

↓

MySQL
```

Everything discovers automatically.

---

Troubleshooting

```bash
ping database
```

or

```bash
nslookup database
```

---

Analogy

Google Maps.

Search restaurant by name.

Not GPS coordinates.

---

Test Yourself

1.

Why service discovery?

Dynamic infrastructure.

---

2.

Benefit?

No hardcoded IPs.

---

3.

Docker uses?

Embedded DNS.

---

# 8. Port Mapping

## What?

Expose container port.

```
docker run -p 8080:80 nginx
```

Meaning

```
Host

8080

↓

Container

80
```

---

Production

Users access application.

Without mapping,

container stays private.

---

Troubleshooting

```bash
docker ps
```

Shows

```
0.0.0.0:8080->80
```

---

Analogy

Apartment building.

Reception

↓

Apartment

---

Test Yourself

1.

Meaning of

```
8080:80
```

Host:Container

---

2.

Without mapping internet access?

No.

---

3.

Can multiple containers use same host port?

No.

---

# 9. NAT

## What?

Docker performs Network Address Translation.

Container

```
172.17.0.2
```

becomes

```
192.168.1.100
```

outside.

---

Why?

Private IPs cannot route internet.

---

Production

Thousands of containers share one host IP.

---

Troubleshooting

Check

```bash
iptables -t nat -L
```

or

```bash
nft list ruleset
```

(depending on the host's firewall backend)

---

Analogy

Receptionist forwarding calls.

Outside world sees office number.

---

Test Yourself

1.

Why NAT?

Private IPs.

---

2.

Who performs NAT?

Docker configures Linux networking (iptables/nftables) on the host.

---

3.

Benefit?

Many containers share one IP.

---

# 10. Bridge Networking

This is the implementation of the bridge driver.

Linux bridge

↓

Virtual switch

↓

Connects containers

It is the underlying mechanism behind bridge networking.

---

Troubleshooting

```bash
ip link
```

Shows

```
docker0
```

---

Analogy

Ethernet switch.

---

Questions

1.

What is docker0?

Linux bridge.

---

2.

Is it Layer 2?

Yes.

---

3.

Who creates it?

Docker Engine.

---

# 11. Container-to-Container Communication

## What?

Containers exchange traffic directly.

```
App

↓

Redis

↓

Postgres
```

---

Production

Every microservice architecture depends on this.

---

Troubleshooting

```bash
docker exec app ping redis
```

```bash
curl http://backend:8080
```

```bash
docker network inspect
```

---

Analogy

Employees talking inside office.

No need to go outside.

---

Test Yourself

1.

Need published ports?

No.

Internal traffic doesn't require port publishing.

---

2.

Need same network?

Yes.

---

3.

Best practice?

Use service names, not IPs.

---

# 15 Important Mid-Level DevOps Interview Questions

## 1. What is the difference between the default bridge network and a user-defined bridge network?

**Answer:** A user-defined bridge provides automatic DNS-based service discovery (containers can reach each other by name), better isolation, and easier network management. The default `bridge` network does not automatically resolve container names.

---

## 2. Why shouldn't applications use container IP addresses?

**Answer:** Container IPs are ephemeral and can change whenever a container is recreated. Using service names through Docker DNS makes applications resilient.

---

## 3. Explain how `docker run -p 8080:80 nginx` works.

**Answer:** Docker configures NAT and port forwarding so traffic arriving on the host's port 8080 is forwarded to port 80 inside the container.

---

## 4. What is the difference between `EXPOSE` and `-p`?

**Answer:** `EXPOSE` is metadata that documents which ports the application expects to use. It does **not** publish ports. The `-p` option actually publishes a container port to the host.

---

## 5. When would you use host networking?

**Answer:** For workloads needing maximum network performance or direct access to the host network stack, such as monitoring agents or packet capture tools.

---

## 6. Why is overlay networking needed?

**Answer:** It enables communication between containers running on different hosts in a cluster, something bridge networking cannot do.

---

## 7. How does Docker DNS work?

**Answer:** Docker runs an embedded DNS server for user-defined networks. Containers register their names, allowing other containers on the same network to resolve those names to IP addresses.

---

## 8. How would you troubleshoot if one container cannot communicate with another?

**Answer:**

1. Check they are on the same network (`docker network inspect`).
2. Verify DNS resolution (`nslookup` or `getent hosts` inside the container).
3. Test connectivity (`ping` if available or `curl`/application-specific client).
4. Verify the destination service is listening on the expected port.
5. Review application logs.

---

## 9. What is NAT in Docker?

**Answer:** Docker uses Linux NAT rules so containers with private IP addresses can communicate with external networks and receive traffic via published ports.

---

## 10. Can containers on different bridge networks communicate directly?

**Answer:** No. Separate bridge networks are isolated from each other unless you explicitly connect a container to both networks or configure routing.

---

## 11. What happens if two containers try to publish the same host port?

**Answer:** The second container fails to start with a port binding error because only one process can listen on the same host IP and port combination.

---

## 12. Why do microservices rely on service discovery?

**Answer:** Service instances are frequently created, destroyed, and rescheduled. Service discovery lets applications communicate using stable names instead of changing IP addresses.

---

## 13. What is `docker0`?

**Answer:** `docker0` is the Linux bridge interface Docker creates on the host for the default bridge network. It acts like a virtual Layer-2 switch connecting containers.

---

## 14. What is the difference between bridge and macvlan?

**Answer:** Bridge networking gives containers private IP addresses behind the host, while macvlan gives each container its own MAC address and typically its own LAN IP, making it appear as a physical device on the network.

---

## 15. Why is container networking knowledge important for a DevOps engineer?

**Answer:** Many production outages involve connectivity rather than application code—failed service communication, DNS issues, incorrect port publishing, firewall rules, NAT problems, or multi-host networking. Understanding Docker networking enables faster diagnosis and resolution of these issues.

---

# Commands You Should Know for Interviews

```bash
# List networks
docker network ls

# Inspect a network
docker network inspect bridge

# Create a bridge network
docker network create my-network

# Run a container on a network
docker run --network my-network nginx

# Connect an existing container to another network
docker network connect my-network my-container

# Disconnect a container
docker network disconnect my-network my-container

# Show published ports
docker port my-container

# Inspect container networking
docker inspect my-container

# List host network interfaces (shows docker0)
ip addr

# Show bridge interfaces
bridge link

# Show routing table
ip route

# Test connectivity from a container
docker exec app curl http://backend:8080

# Resolve a service name
docker exec app getent hosts backend

# View NAT rules (iptables-based systems)
sudo iptables -t nat -L -n -v

# View NAT rules (nftables-based systems)
sudo nft list ruleset
```

If you're preparing for **international mid-level DevOps interviews**, mastering these networking concepts will also make Kubernetes networking (CNI, Services, kube-proxy, Ingress, NetworkPolicies, and Service Mesh) much easier, because Docker networking provides the foundational concepts they build upon.
