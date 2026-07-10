This is an excellent list. For a **Mid-level DevOps Engineer**, networking is one of the biggest differentiators. Many people know how to deploy Kubernetes, but they struggle when the application is unreachable, DNS fails, or packets disappear.

The good news is that from our previous conversations, you've already developed a solid understanding of networking fundamentals (subnet masks, routing tables, MAC addresses, EKS networking, ICMP, DNS, etc.). These topics tie everything together.

---

# 1. TCP/IP

## 1. What is this?

TCP/IP is the communication protocol suite used by the Internet.

Think of it as a complete set of rules.

IP handles **where data should go**.

TCP handles **how data reaches safely**.

Example:

```
Browser
    ↓
TCP
    ↓
IP
    ↓
Network
```

---

## 2. Why is it required?

Without TCP/IP:

* computers wouldn't know destination
* data could arrive corrupted
* packets might never arrive
* applications couldn't communicate

---

## 3. How does this help in DevOps?

Almost every tool uses TCP/IP.

Examples:

* SSH
* Kubernetes API
* Docker Registry
* GitHub
* Prometheus
* Grafana
* Jenkins
* PostgreSQL
* Redis
* AWS APIs

---

## 4. Why recruiters care?

Because production systems communicate using TCP/IP.

If services can't communicate,
production is down.

---

## 5. Production problem solved

Example:

```
Application cannot connect to database.
```

Instead of blaming PostgreSQL,

you investigate:

* IP reachable?
* TCP port open?
* firewall?
* routing?
* security groups?

---

## 6. Troubleshooting

Commands:

```
ping

ss -tulnp

netstat

tcpdump

curl

nc
```

Example:

```
curl timeout

↓

ping works

↓

TCP port blocked

↓

Security Group issue
```

---

## 7. Dependencies

Everything.

SSH

HTTPS

DNS

Kubernetes

AWS APIs

---

## 8. Analogy

IP = Address on parcel

TCP = Courier ensuring parcel arrives correctly.

---

## 9. Three questions

✔ Difference between TCP and IP?

✔ Why can IP connectivity exist while TCP fails?

✔ Why does TCP use sequence numbers?

---

# 2. UDP

## What is it?

Connectionless protocol.

No guarantee.

No retransmission.

Very fast.

---

## Why required?

Sometimes speed matters more than reliability.

Examples

* DNS
* VoIP
* Video call
* Live streaming
* Online gaming

---

## DevOps importance

CoreDNS

Prometheus exporters

Service discovery

Monitoring

---

## Recruiters care

Need to know why some protocols intentionally avoid TCP.

---

## Production problem

DNS lookup intermittently failing.

Firewall blocks UDP 53.

---

## Troubleshooting

```
dig

tcpdump udp

ss -u
```

---

## Dependencies

DNS

DHCP

Streaming

---

## Analogy

Throwing newspapers.

No confirmation.

---

## Questions

Why doesn't UDP retransmit?

Why DNS uses UDP?

When should TCP replace UDP?

---

# 3. DNS

## What?

Converts

```
google.com

↓

142.250.x.x
```

---

## Why required?

Humans remember names.

Machines need IPs.

---

## DevOps

Every Kubernetes Service uses DNS.

```
postgres.default.svc.cluster.local
```

AWS

ALB

Route53

CoreDNS

Ingress

Everything depends on DNS.

---

## Recruiters care

Most outages are "DNS issue."

---

## Production problem

Pods cannot resolve database hostname.

Application fails.

---

## Troubleshooting

```
dig

nslookup

host

cat /etc/resolv.conf

kubectl exec

dig kubernetes.default
```

---

## Dependencies

Everything using hostname.

---

## Analogy

Phone contacts.

You know "Mom"

Phone knows number.

---

## Questions

Recursive vs authoritative?

TTL?

Why DNS cache exists?

---

# 4. Routing

## What?

Routing decides

"Which path should packet take?"

---

## Why?

Networks are interconnected.

Packets need directions.

---

## DevOps

AWS Route Tables

Linux routing table

Kubernetes routing

VPN

Transit Gateway

---

## Production

Pod cannot reach Internet.

Wrong route table.

---

## Troubleshooting

```
ip route

traceroute

tracepath
```

---

## Dependencies

NAT

Gateway

VPC

VPN

---

## Analogy

Google Maps.

---

## Questions

Default route?

Longest prefix match?

Static vs dynamic routing?

---

# 5. Gateway

## What?

Gateway is the next hop leaving current network.

Usually router.

---

## Why?

Local devices don't know remote networks.

Gateway forwards packets.

---

## DevOps

Default Gateway

Internet Gateway

NAT Gateway

Transit Gateway

---

## Production

Wrong gateway.

Entire server loses Internet.

---

## Troubleshooting

```
ip route

ping gateway
```

---

## Dependencies

Routing

Internet access

---

## Analogy

Exit gate of apartment.

---

## Questions

Default gateway?

Difference from router?

Can gateway fail while LAN works?

---

# 6. MTU

## What?

Maximum packet size.

Usually

```
1500 bytes
```

---

## Why?

Large packets may fragment.

---

## DevOps

VPN

Docker

Kubernetes overlay networks

VXLAN

WireGuard

Cloudflare WARP

---

## Production

VPN drops packets.

Large uploads fail.

---

## Troubleshooting

```
ping -M do

tracepath

ip link
```

---

## Dependencies

TCP

VPN

Overlay network

---

## Analogy

Truck size limit on bridge.

---

## Questions

Why fragmentation happens?

Why VPN reduces MTU?

How PMTU works?

---

# 7. ARP

## What?

Maps

IP

↓

MAC

inside local LAN.

---

## Why?

Ethernet sends frames to MAC.

Not IP.

---

## DevOps

Node communication

VM networking

Bridge networking

---

## Production

Duplicate IP

Wrong MAC

ARP cache stale

---

## Troubleshooting

```
arp -a

ip neigh
```

---

## Dependencies

Ethernet

LAN

---

## Analogy

Looking up apartment number from person's name.

---

## Questions

Why ARP only works locally?

What is ARP cache?

What is Gratuitous ARP?

---

# 8. ICMP

## What?

Network diagnostic protocol.

Used by

```
ping

traceroute
```

---

## Why?

Reports errors.

---

## DevOps

Health checks

Monitoring

Latency

---

## Production

Ping blocked.

Website still works.

---

## Troubleshooting

```
ping

traceroute

mtr
```

---

## Dependencies

Network diagnostics.

---

## Analogy

Postal service returning

"Address not found."

---

## Questions

Why website works when ping fails?

TTL exceeded?

Destination unreachable?

---

# 9. CIDR

## What?

CIDR defines network size.

Example

```
10.0.0.0/24
```

---

## Why?

Efficient IP allocation.

---

## DevOps

VPC

Subnets

EKS

Docker

Kubernetes

---

## Production

Subnet overlap.

Routing failure.

---

## Troubleshooting

Know subnet boundaries.

---

## Dependencies

Routing

VPC

Firewall

---

## Analogy

Apartment building floors.

---

## Questions

Difference between /24 and /16?

Hosts in /26?

Why CIDR replaced classful networking?

---

# 10. NAT

## What?

Network Address Translation.

Changes IP.

Private

↓

Public

---

## Why?

Millions of private devices share one public IP.

---

## DevOps

AWS NAT Gateway

Docker

Kubernetes SNAT

Home router

---

## Production

Private subnet loses Internet.

NAT Gateway deleted.

---

## Troubleshooting

```
curl ifconfig.me

traceroute

iptables

conntrack
```

---

## Dependencies

Internet

Private subnet

Containers

---

## Analogy

Company receptionist.

Everyone calls using one company number.

---

## Questions

SNAT?

DNAT?

Why private subnet needs NAT?

---

# 15 Important Interview Questions (Mid-Level DevOps)

These are the kinds of networking questions that frequently come up in DevOps interviews because they test whether you can diagnose real production issues rather than just define networking terms.

### 1. Explain the difference between TCP and UDP.

**Answer:** TCP is connection-oriented, reliable, uses acknowledgments and retransmissions. UDP is connectionless, faster, and does not guarantee delivery. TCP is used for SSH, HTTPS, and databases; UDP is commonly used for DNS, DHCP, streaming, and VoIP.

---

### 2. A website opens in the browser, but `ping` fails. Why?

**Answer:** `ping` uses ICMP, while browsers typically use TCP (HTTP/HTTPS). ICMP may be blocked by firewalls or security groups even though TCP ports 80 or 443 remain open.

---

### 3. What happens when you type `google.com` into a browser?

**Answer:** The browser checks local DNS caches, queries a DNS resolver if needed, receives Google's IP address, performs a TCP handshake with that IP, establishes a TLS session (for HTTPS), sends the HTTP request, and receives the response.

---

### 4. What is the purpose of a default gateway?

**Answer:** It is the next-hop router used when the destination IP is outside the local subnet. Without it, a host can communicate only within its own network.

---

### 5. What is ARP and when is it used?

**Answer:** ARP resolves an IPv4 address to a MAC address on the local network. Before sending an Ethernet frame to a local destination or the default gateway, the sender uses ARP to discover the destination MAC address.

---

### 6. How would you troubleshoot a pod that cannot reach the internet?

**Answer:** Verify the pod IP, inspect routing, check DNS resolution, confirm network policies, validate NAT gateway or egress configuration, inspect security groups or firewalls, and use tools like `curl`, `ping`, `traceroute`, `tcpdump`, and `kubectl exec`.

---

### 7. What is MTU and why can an incorrect MTU cause problems?

**Answer:** MTU is the maximum transmission unit. If packets exceed the supported MTU and cannot be fragmented appropriately, they may be dropped, leading to issues like hanging connections, failed uploads, or VPN problems.

---

### 8. What is CIDR and why is it important in AWS?

**Answer:** CIDR defines network ranges (for example, `10.0.0.0/16`). AWS uses CIDR blocks for VPCs and subnets, route matching, and IP allocation. Poor CIDR planning can lead to overlapping networks and routing conflicts.

---

### 9. What is NAT and why do private subnets need it?

**Answer:** NAT translates private IP addresses to a public IP so instances in private subnets can access the internet without being directly reachable from it. In AWS, this is commonly provided by a NAT Gateway.

---

### 10. What is the difference between routing and DNS?

**Answer:** DNS translates names to IP addresses. Routing determines the path packets take to reach those IP addresses. DNS answers "where is it?" while routing answers "how do I get there?"

---

### 11. How do you troubleshoot DNS problems on Linux?

**Answer:** Check `/etc/resolv.conf`, use `dig` or `nslookup`, verify the DNS server is reachable, test hostname resolution from the affected application or pod, and inspect DNS caches if applicable.

---

### 12. Why do TCP connections use a three-way handshake?

**Answer:** It synchronizes sequence numbers, confirms both endpoints are reachable, and ensures both sides are ready to exchange data reliably before application traffic begins.

---

### 13. What tools would you use to troubleshoot a networking issue on Linux?

**Answer:** Common tools include `ip`, `ss`, `ping`, `traceroute`, `tracepath`, `dig`, `curl`, `nc`, `tcpdump`, `arp`, `ip neigh`, and `mtr`. Each helps isolate a different layer of the networking stack.

---

### 14. How does a packet travel from a pod in Kubernetes to the internet?

**Answer:** The pod sends the packet to its virtual interface, the node forwards it according to its routing rules, network plugins handle overlay or bridge networking as needed, the packet exits through the node's network interface, is typically source-NATed (SNAT), reaches the VPC router, and then exits through an Internet Gateway or NAT Gateway depending on the subnet design.

---

### 15. An application cannot connect to a database. What is your troubleshooting approach?

**Answer:** Confirm the application is using the correct hostname and port, verify DNS resolution, test IP reachability, check if the TCP port is listening, inspect firewalls, security groups, network ACLs, routing, database availability, and finally capture packets with `tcpdump` if needed to determine where the connection fails.

---

## Recommendation for Mid-level DevOps Interviews

For international remote DevOps roles, these networking topics are necessary but not sufficient. The next step is to understand how they combine in real production systems. A strong mental model is:

> **Application → DNS → TCP/UDP → IP → Routing → ARP → Gateway → NAT → Remote Service**

If you can trace a packet end-to-end through those layers and explain where failures can occur and how to diagnose each one, you'll be well prepared for the networking portion of a mid-level DevOps interview.
