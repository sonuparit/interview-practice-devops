This is a very important section for a Mid-level DevOps Engineer.

If Linux is about **managing the server**, Networking is about **making servers communicate**.

In real production, **70-80% of incidents are either networking, DNS, firewall, routing, TLS, or connectivity problems.**

Many interviewers intentionally ask networking questions because they immediately reveal whether someone has actually worked with production systems.

---

# Networking Commands Overview

| Command    | Purpose                           |
| ---------- | --------------------------------- |
| ip         | Configure and inspect networking  |
| ss         | Check sockets and listening ports |
| netstat    | Old network diagnostic tool       |
| ping       | Test connectivity                 |
| traceroute | Find routing path                 |
| dig        | Advanced DNS lookup               |
| nslookup   | Basic DNS lookup                  |
| host       | Simple DNS query                  |
| curl       | HTTP client                       |
| wget       | Download files                    |
| nc         | Raw TCP/UDP testing               |
| telnet     | Old TCP connectivity testing      |
| tcpdump    | Packet capture                    |

---

# 1. ip

---

## What is this?

The modern Linux networking command.

It replaces many old commands like

* ifconfig
* route
* arp

Everything networking starts here.

Examples

```
ip addr
ip route
ip link
ip neigh
```

---

## Why is it required?

To

* see IP addresses
* configure interfaces
* troubleshoot routing
* enable/disable NIC
* check gateways

---

## Mid-Level DevOps Use

Every production server troubleshooting starts with

```
ip addr
```

and

```
ip route
```

---

## Why recruiters care?

Because without IP configuration nothing communicates.

---

## Production Problem

Application cannot reach database.

You verify

```
ip addr
```

Server has wrong IP.

Problem solved.

---

## Troubleshooting Example

Website unreachable.

Check

```
ip addr
```

Interface accidentally down.

Bring it up.

---

## Dependencies

Affects

* routing
* DNS
* Kubernetes
* Docker
* VPN
* cloud networking

---

## Analogy

Think of a house.

IP address = house number.

Without house number nobody can deliver letters.

---

## Must Answer

1. Difference between IP address and MAC?
2. Difference between ip addr and ip route?
3. Why default gateway is needed?

---

# 2. ss

(Socket Statistics)

---

## What is this?

Shows

* listening ports
* established connections
* socket states

Example

```
ss -tulnp
```

---

## Why required?

To know

Which service is listening?

Who owns port?

---

## DevOps Use

Website isn't working.

Check

```
ss -tulnp
```

Port 443 missing.

Nginx failed.

---

## Recruiters

Most production outages begin with

"Is service actually listening?"

---

## Production Example

Application says

Connection refused

```
ss -tln
```

Nothing on port 8080.

Problem identified.

---

## Dependencies

* systemd
* Docker
* Kubernetes
* Firewalls

---

## Analogy

Apartment building.

Sockets are doors.

ss tells you

Which doors are open?

---

## Must Answer

1. Difference between LISTEN and ESTABLISHED?
2. What does ss -tulnp show?
3. Why use ss instead of netstat?

---

# 3. netstat

---

## What is this?

Older networking tool.

Still appears in legacy systems.

Shows

* routing
* sockets
* statistics

---

## Why required?

Many old production servers still use it.

---

## DevOps Use

Legacy troubleshooting.

---

## Recruiters

Need compatibility knowledge.

---

## Production Example

Old CentOS server.

No ss installed.

Use

```
netstat -tulnp
```

---

## Dependencies

Same as ss.

---

## Analogy

Old map of a city.

Still works.

---

## Must Answer

1. Why replaced by ss?
2. Difference between netstat and ss?
3. When would you still use it?

---

# 4. ping

---

## What is this?

Tests reachability using ICMP.

---

## Why required?

Quick connectivity test.

---

## DevOps Use

Can server reach gateway?

Can pod reach node?

---

## Production Example

Application unreachable.

```
ping database
```

Fails.

Network issue.

---

## Dependencies

Firewall

ICMP

Routing

---

## Analogy

Calling someone's phone.

If it rings,

they're reachable.

---

## Must Answer

1. Does ping test application?
2. Why can ping fail but website still works?
3. Why is ICMP sometimes blocked?

---

# 5. traceroute

---

## What is this?

Shows every router between source and destination.

---

## Why required?

Find where packets stop.

---

## DevOps Use

Internet latency.

VPN issues.

Cloud routing.

---

## Production Example

Website slow.

Traceroute shows delay at ISP.

---

## Dependencies

Routers

Gateways

Internet

---

## Analogy

Google Maps route.

---

## Must Answer

1. What is a hop?
2. Why can traceroute stop midway?
3. Difference from ping?

---

# 6. dig

---

## What is this?

Professional DNS lookup tool.

---

## Why required?

Verify DNS records.

---

## DevOps Use

```
dig google.com
```

```
dig MX
```

```
dig TXT
```

---

## Production Example

Domain not resolving.

Check DNS.

---

## Dependencies

DNS servers.

---

## Analogy

Phone directory lookup.

---

## Must Answer

1. Difference between recursive and authoritative answer?
2. What records can dig query?
3. Why preferred over nslookup?

---

# 7. nslookup

---

## What is this?

Simple DNS query tool.

---

## Why required?

Quick DNS checks.

---

## DevOps Use

Verify hostname resolution.

---

## Production Example

API cannot reach DB hostname.

nslookup reveals wrong IP.

---

## Dependencies

DNS.

---

## Analogy

Simple dictionary.

---

## Must Answer

1. Difference from dig?
2. What does it query?
3. Why is dig preferred?

---

# 8. host

---

## What is this?

Very lightweight DNS lookup.

---

## Why required?

Quick hostname resolution.

---

## DevOps Use

Scripts.

Automation.

---

## Production Example

```
host google.com
```

---

## Dependencies

DNS.

---

## Analogy

Pocket dictionary.

---

## Must Answer

1. Difference from dig?
2. When use host?
3. What output does it provide?

---

# 9. curl

---

## What is this?

HTTP client.

Can call

REST APIs

HTTPS

Headers

Authentication

---

## Why required?

DevOps uses APIs everywhere.

---

## DevOps Use

Health checks.

Webhook testing.

Kubernetes API.

Cloud APIs.

---

## Production Example

```
curl http://localhost:8080/health
```

Returns

```
200 OK
```

Application healthy.

---

## Dependencies

HTTP

TLS

DNS

Networking

---

## Analogy

Sending a letter and reading reply.

---

## Must Answer

1. Difference between curl and ping?
2. How check only headers?
3. How send POST request?

---

# 10. wget

---

## What is this?

Downloads files.

---

## Why required?

Automation.

Software installation.

---

## DevOps Use

CI/CD.

Download artifacts.

---

## Production Example

```
wget backup.tar.gz
```

---

## Dependencies

HTTP

FTP

HTTPS

---

## Analogy

Delivery truck.

---

## Must Answer

1. Difference between curl and wget?
2. Resume interrupted download?
3. Recursive download?

---

# 11. nc (Netcat)

---

## What is this?

Swiss Army Knife of networking.

---

## Why required?

Test TCP/UDP ports.

---

## DevOps Use

```
nc -zv host 443
```

---

## Production Example

Database unreachable.

Port 5432 closed.

---

## Dependencies

TCP

UDP

Firewall

---

## Analogy

Knocking on every door.

---

## Must Answer

1. What does -z mean?
2. Why use nc instead of ping?
3. Can nc test UDP?

---

# 12. telnet

---

## What is this?

Old remote terminal.

Still useful for port testing.

---

## Why required?

Legacy servers.

---

## DevOps Use

```
telnet host 25
```

SMTP test.

---

## Production Example

Email server debugging.

---

## Dependencies

TCP.

---

## Analogy

Old telephone.

---

## Must Answer

1. Why insecure?
2. Why still used?
3. Difference from SSH?

---

# 13. tcpdump

---

## What is this?

Captures packets directly from network interface.

One of the most powerful troubleshooting tools.

---

## Why required?

When everything else looks correct, inspect the packets themselves.

---

## DevOps Use

* Debug Kubernetes networking
* Verify requests reach a service
* Check DNS queries
* Diagnose TLS handshake issues
* Analyze packet loss

---

## Production Problem

Users report they cannot access an API.

* `ss` shows the application is listening.
* Firewall rules look correct.
* `curl` from the same host works.

Run:

```bash
sudo tcpdump -i eth0 port 443
```

If no packets arrive, the issue is upstream (load balancer, firewall, routing, etc.).

---

## Troubleshooting Example

DNS resolution is failing.

Capture DNS traffic:

```bash
sudo tcpdump -i eth0 port 53
```

You discover the server is sending DNS queries but receiving no responses. The problem is likely with the DNS server or the network path to it.

---

## Dependencies

* Network interfaces
* TCP/IP
* UDP
* DNS
* Firewalls
* Load balancers

---

## Analogy

Imagine a highway with many vehicles (packets). `tcpdump` is a camera placed beside the highway that records every vehicle passing by.

---

## Must Answer

1. What does `tcpdump` capture?
2. Why is `tcpdump` useful when `curl` fails?
3. How would you capture only traffic on port 443?

---

# 15 Important Mid-Level DevOps Interview Questions

These are representative questions that interviewers commonly ask to assess practical networking knowledge.

### 1. A website is down. What steps would you take to troubleshoot it?

**Answer (high-level flow):**

1. Check IP configuration (`ip addr`)
2. Verify routing (`ip route`)
3. Test basic connectivity (`ping`)
4. Check DNS (`dig`/`nslookup`)
5. Confirm the service is listening (`ss -tulnp`)
6. Test the application (`curl`)
7. Check firewall/security groups
8. Capture packets with `tcpdump` if necessary

---

### 2. `ping` works, but `curl` fails. Why?

Because ICMP (used by `ping`) and HTTP (used by `curl`) are different protocols. The network path may be fine, but the web service may be down, the port may be closed, TLS may be failing, or a firewall may be blocking HTTP/HTTPS.

---

### 3. How would you determine whether a service is listening on port 8080?

```bash
ss -tulnp | grep 8080
```

If no process is listening, clients will receive "Connection refused."

---

### 4. What is the difference between `curl` and `wget`?

* `curl` is primarily for interacting with services and APIs (GET, POST, headers, authentication, etc.).
* `wget` is optimized for downloading files and can resume interrupted downloads or mirror websites.

---

### 5. Why is `ss` preferred over `netstat`?

`ss` is faster, uses the kernel's socket diagnostics directly, consumes fewer resources, and is actively maintained. `netstat` is largely kept for compatibility with older systems.

---

### 6. What is the difference between `dig`, `nslookup`, and `host`?

* `dig`: Detailed DNS diagnostics, preferred by administrators.
* `nslookup`: Simpler interactive DNS queries.
* `host`: Minimal, quick hostname/IP lookups.

---

### 7. A hostname resolves to the wrong IP. Which tool would you use first?

Use:

```bash
dig hostname
```

Then inspect DNS records and determine whether the problem is due to DNS configuration, propagation, or caching.

---

### 8. When would you use `tcpdump`?

When connectivity appears correct but packets are not behaving as expected—for example, intermittent failures, TLS issues, missing responses, or suspected firewall/routing problems.

---

### 9. How do you verify whether a remote TCP port is reachable?

Use:

```bash
nc -zv server.example.com 443
```

This confirms whether a TCP connection can be established to that port.

---

### 10. What is the difference between "Connection refused" and "Connection timed out"?

* **Connection refused:** The destination host is reachable, but nothing is accepting connections on that port (or it actively rejects them).
* **Connection timed out:** No response is received, often because of a firewall, routing issue, or unreachable host.

---

### 11. What information does `ip route` provide?

It displays the routing table, including the default gateway and the paths the kernel uses to send packets to different networks.

---

### 12. What does `traceroute` tell you that `ping` does not?

`traceroute` shows each intermediate router (hop) between the source and destination and where delays or packet drops occur, while `ping` only measures end-to-end reachability and latency.

---

### 13. Why might `ping` fail even though the website is accessible?

Many organizations block ICMP for security reasons, while still allowing TCP ports such as 80 and 443. The web server can therefore be fully functional even if `ping` receives no replies.

---

### 14. How would you troubleshoot a Kubernetes service that cannot reach a database?

A systematic approach would be:

* Verify Pod IP and routes.
* Check DNS resolution inside the Pod.
* Test the database port with `nc`.
* Verify the application is listening on the database server (`ss`).
* Review NetworkPolicies, security groups, and firewalls.
* Use `tcpdump` if packets are disappearing.

---

### 15. Which networking commands do you use most often during production incidents?

A common sequence is:

```bash
ip addr
ip route
ping
dig
ss -tulnp
curl
nc
tcpdump
```

This progression moves from basic network configuration, to name resolution, to service availability, and finally to packet-level analysis. Mastering this workflow demonstrates the troubleshooting methodology expected from a mid-level DevOps engineer.
