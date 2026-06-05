# TryHackMe — Nmap Series: Write-Up
**Author:** [Calebe Araújo]  
**Platform:** TryHackMe  
**Module:** Nmap — Complete Series (Live Host Discovery · Basic Port Scans · Advanced Port Scans · Post-Port Scanning)  
**Category:** Network Reconnaissance / Enumeration  
**Difficulty:** Easy–Medium  

---

## Table of Contents

1. [Overview](#overview)
2. [Module 1 — Live Host Discovery](#module-1--live-host-discovery)
3. [Module 2 — Basic Port Scans](#module-2--basic-port-scans)
4. [Module 3 — Advanced Port Scans](#module-3--advanced-port-scans)
5. [Module 4 — Post-Port Scanning](#module-4--post-port-scanning)
6. [Key Takeaways](#key-takeaways)

---

## Overview

This write-up documents the methodology, commands, and concepts covered across the four-module Nmap series on TryHackMe. The series progresses from foundational host discovery to advanced evasion techniques and post-enumeration analysis using the Nmap Scripting Engine (NSE).

These modules cover core skills for the reconnaissance phase of a penetration test or red team engagement: identifying live hosts, mapping open ports, fingerprinting services and operating systems, and evading common network filtering mechanisms.

---

## Module 1 — Live Host Discovery

### Objectives

- Understand the role of ARP, ICMP, TCP, and UDP in host discovery.
- Enumerate active hosts prior to port scanning.
- Apply reverse DNS resolution to gather additional host context.

---

### Task 2 — ARP Scanning

#### Concepts

ARP (Address Resolution Protocol) operates exclusively within the local broadcast domain and resolves IP addresses to MAC addresses. When a host initiates communication with another device on the same subnet, it must first resolve the destination's MAC address via an ARP Request.

Key behavioral observations from lab exercises:

| Parameter | Result |
|---|---|
| Devices that received the ARP request | 4 |
| Did Computer6 receive the broadcast? | No |
| Did Computer6 respond? | Yes |

#### Command

```bash
nmap -PR -sn <TARGET_CIDR>
```

ARP-based discovery is particularly reliable in LAN environments because hosts typically respond to ARP requests even when ICMP traffic is filtered by host-based firewalls.

---

### Task 3 — ICMP Echo Discovery

#### Concepts

Before an ICMP Echo Request can be transmitted, the scanning host must perform an ARP exchange to resolve the gateway or target's MAC address. Once the ARP table entry is cached, subsequent probes to the same host do not require additional ARP resolution until the cache entry expires.

Lab observations:

| Event | Detail |
|---|---|
| Packet sent before ICMP ping | ARP Request |
| Packet received before ICMP ping | ARP Response |
| Number of hosts responding to ping | 1 |
| First device responding to ARP #1 | Router |
| First device responding to ARP #2 | Computer5 |
| Did new pings require new ARP? | No |

---

### Task 4 — List Scan (`-sL`)

#### Command

```bash
nmap -iL list_of_hosts.txt
nmap -sL <TARGETS>
```

The `-sL` flag enumerates the specified targets without transmitting any packets to the hosts. This is useful for validating IP ranges and CIDR blocks before executing an active scan.

| Range Query | Result |
|---|---|
| First IP in 10.10.12.13/29 | 10.10.12.8 |
| Total addresses in 10.10.0-255.101-125 | 6,400 hosts |

---

### Task 5 — ARP Host Discovery

#### Command

```bash
sudo nmap -PR -sn <CONNECTION_IP>/24
```
this command is used to discovevr all the live systems on the same subnet

**question** How many hosts are found alive after scanning the CONNECTION_IP/24?

**Result:** 1 active host identified.

---

### Task 6 — ICMP Host Discovery Variants

#### Commands

```bash
nmap -PE -sn 10.200.6.0/24   # ICMP Echo Request
nmap -PP -sn 10.200.6.0/24   # ICMP Timestamp Request
nmap -PM -sn 10.200.6.0/24   # ICMP Address Mask Request
```

| Flag | ICMP Type |
|---|---|
| `-PE` | Echo Request |
| `-PP` | Timestamp Request |
| `-PM` | Address Mask Request |

Each probe type may produce different results depending on the network's ingress/egress filtering policy. Using multiple techniques increases the probability of successful host discovery in hardened environments.

---

### Task 7 — TCP and UDP Ping Scans

#### Commands

```bash
nmap -PS -sn 10.200.6.0/24           # TCP SYN Ping (no privileges needed)
sudo nmap -PA -sn 10.200.6.0/24      # TCP ACK Ping (requires root)
sudo nmap -PU -sn 10.200.6.0/24      # UDP Ping (requires root)

# Masscan alternatives
masscan 10.200.6.0/24 -p443
masscan 10.200.6.0/24 -p80,443
masscan 10.200.6.0/24 -p22-25
```

| Technique | Privilege Required | Flag |
|---|---|---|
| TCP SYN Ping | No | `-PS` |
| TCP ACK Ping | Yes | `-PA` |
| TCP SYN on port 23 | No | `-PS23` |

In environments where ICMP is blocked at the perimeter, TCP- and UDP-based host discovery provides viable alternatives for identifying live systems.

---

### Task 8 — Reverse DNS Lookup

#### Flag

```bash
-R
```

Reverse DNS resolution maps IP addresses back to hostnames, potentially exposing machine names, service roles, or infrastructure ownership — valuable intelligence during the reconnaissance phase.

---

## Module 2 — Basic Port Scans

### Objectives

- Understand the six port states recognized by Nmap.
- Distinguish between TCP Connect, SYN, and UDP scan techniques.
- Configure scan timing and parallelism for different engagement contexts.

---

### Task 2 — Port States

Nmap classifies ports into six states:

| State | Description |
|---|---|
| Open | A service is actively accepting connections |
| Closed | The port is accessible but no service is listening |
| Filtered | A firewall or ACL is preventing Nmap from determining the state |
| Unfiltered | The port is accessible, but its open/closed state cannot be determined |
| Open\|Filtered | Nmap cannot determine whether the port is open or filtered |
| Closed\|Filtered | Nmap cannot determine whether the port is closed or filtered |

From a penetration testing perspective, **Open** ports are the primary targets of interest as they indicate accessible, potentially exploitable services.

Additional findings:

| Query | Result |
|---|---|
| Default protocol for port 53 | UDP (DNS) |
| Default service on TCP 22 | SSH |
| Most valuable state for a pentester | Open |

---

### Task 3 — TCP Three-Way Handshake

Standard TCP connection establishment:

```
Client ──SYN──────────────► Server
Client ◄──────────SYN-ACK── Server
Client ──ACK──────────────► Server
```

| Flag | Function |
|---|---|
| SYN | Initiates the connection |
| RST | Terminates the connection abnormally (Reset) |

Understanding the handshake is fundamental to interpreting how different scan techniques interact with target systems.

---

### Task 4 — TCP Connect Scan (`-sT`)

#### Command

```bash
nmap -sT <TARGET_IP>
```

TCP Connect Scan completes the full three-way handshake using the operating system's native socket calls. It does not require elevated privileges but tends to generate more log entries on the target, making it more detectable than SYN Scan.

| Finding | Result |
|---|---|
| FTP (port 21) state | Open |
| Service on port 53 | domain (DNS) |

---

### Task 5 — TCP SYN Scan (`-sS`) — Half-Open Scan

#### Command

```bash
nmap -sS <TARGET_IP>
```

The SYN Scan (also called a Half-Open Scan) transmits only the initial SYN packet and analyzes the server's response without completing the TCP connection:

```
Scanner ──SYN──► Target
Scanner ◄──SYN-ACK── Target   (port open)
Scanner ──RST──► Target        (connection torn down — not logged by most services)
```

| Finding | Result |
|---|---|
| SYN-ACK responses received | 4 |
| Open ports identified | 4 |

Because the connection is never fully established, SYN Scan is significantly faster and generates fewer application-layer log entries than a full TCP Connect Scan.

---

### Task 6 — UDP Scan (`-sU`)

#### Commands

```bash
nmap -sU <TARGET_IP>
nmap -sU --top-ports 10 <TARGET_IP>
```

UDP enumeration is inherently slower and less reliable than TCP scanning because many services silently discard unexpected UDP packets without responding.

| Finding | Result |
|---|---|
| UDP port 161 state | Closed |
| Service identified on port 161 | SNMP |

Notable services running over UDP that should not be overlooked during assessments:

| Port | Service |
|---|---|
| 53 | DNS |
| 67/68 | DHCP |
| 123 | NTP |
| 161 | SNMP |

---

### Task 7 — Scan Configuration and Timing

#### Useful Flags

```bash
-p5000-5500           # Scan only ports 5000–5500
--min-parallelism=64  # Enforce at least 64 parallel probes
-T0                   # Paranoid timing (slowest, most covert)
```

Nmap timing templates:

| Template | Name | Use Case |
|---|---|---|
| `-T0` | Paranoid | IDS evasion — extremely slow |
| `-T1` | Sneaky | Low-noise covert scans |
| `-T2` | Polite | Reduced bandwidth consumption |
| `-T3` | Normal | Default behavior |
| `-T4` | Aggressive | Fast scans on reliable networks |
| `-T5` | Insane | Maximum speed — sacrifices accuracy |

Timing selection should reflect the assessment's rules of engagement and the sensitivity of the target environment.

---

## Module 3 — Advanced Port Scans

### Objectives

- Leverage TCP flag manipulation to enumerate ports while evading detection.
- Understand IP spoofing, Decoy Scanning, and packet fragmentation.
- Apply the Idle Scan technique for anonymous reconnaissance.

---

### Task 2 — Null, FIN, and Xmas Scans

These three techniques exploit TCP stack behavior differences to infer port states without completing a standard connection. When a closed port receives these packets, RFC 793-compliant stacks respond with a RST. An open port typically sends no response, resulting in an `open|filtered` classification.

#### Commands

```bash
sudo nmap -sN <TARGET_IP>   # Null Scan  — no flags set
sudo nmap -sF <TARGET_IP>   # FIN Scan   — FIN flag only
sudo nmap -sX <TARGET_IP>   # Xmas Scan  — FIN + PSH + URG
```

| Scan Type | Flags Active | Flag Count |
|---|---|---|
| Null Scan (`-sN`) | None | 0 |
| FIN Scan (`-sF`) | FIN | 1 |
| Xmas Scan (`-sX`) | FIN, PSH, URG | 3 |

Lab results:

| Metric | Result |
|---|---|
| Ports identified as `open\|filtered` — FIN Scan | 9 |
| Ports identified as `open\|filtered` — Null Scan | 9 |

> **Note:** These techniques are ineffective against Windows hosts, as Microsoft's TCP/IP stack responds with RST to all malformed flag combinations regardless of port state, causing all ports to appear closed.

---

### Task 3 — Maimon Scan (`-sM`)

#### Command

```bash
sudo nmap -sM <TARGET_IP>
```

The Maimon Scan activates two TCP flags simultaneously — **FIN** and **ACK** — for a total flag count of **2**. Originally designed to exploit specific behavior in certain BSD-derived TCP stacks, it has limited applicability in modern environments but remains relevant for understanding Nmap's flag-based scan capabilities.

---

### Task 4 — ACK, Window, and Custom Scans

These techniques are primarily used to map firewall rule sets rather than to identify open services.

#### Commands

```bash
sudo nmap -sA <TARGET_IP>             # ACK Scan
sudo nmap -sW <TARGET_IP>             # Window Scan
sudo nmap --scanflags RST <TARGET_IP> # Custom flag scan
```

| Scan Type | Flags Used | Primary Purpose |
|---|---|---|
| ACK Scan (`-sA`) | ACK (1 flag) | Firewall rule mapping |
| Window Scan (`-sW`) | ACK (1 flag) | Open/closed inference via TCP window value |
| Custom (`--scanflags`) | User-defined | Flexible enumeration |

Lab results:

| Metric | Result |
|---|---|
| Flags used in Window Scan | 1 (ACK) |
| Custom scan flag specified | RST |
| Ports classified as unfiltered — ACK Scan | 5 |
| New port discovered | 443 |
| Associated service present | Yes |

**ACK Scan:** Returns `unfiltered` when ACK packets reach the target, and `filtered` when a stateful firewall drops them. Does not differentiate between open and closed ports — strictly used for firewall characterization.

**Window Scan:** Uses the TCP Window field in RST responses to infer port state. Useful on systems where certain legacy or non-standard TCP stacks return non-zero window values for open ports.

---

### Task 5 — IP Spoofing and Decoy Scanning

These techniques obscure or misrepresent the true origin of the scan.

#### Commands

```bash
# Source IP spoofing — requires attacker to be on the same network segment
nmap -S <SPOOFED_IP> <TARGET_IP>
nmap -e <NET_INTERFACE> -Pn -S <SPOOFED_IP> <TARGET_IP>

# Decoy scan — inserts multiple fake source IPs into probe packets
nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME <TARGET_IP>
```

Lab syntax examples:

```bash
-S 10.10.10.11                         # Spoof a specific source IP
-D 10.10.20.21,10.10.20.28,ME          # Decoy scan with two decoys + real IP
```

**Source IP Spoofing (`-S`):** Replaces the scanner's real IP with a specified address. For results to be received, the attacker must be positioned within the same broadcast domain as the spoofed address or have access to intercept return traffic. Primarily useful for testing, not for receiving results remotely.

**Decoy Scan (`-D`):** Injects multiple forged source addresses alongside the actual scan packets. From the target's perspective, the port sweep appears to originate from several different hosts simultaneously, making it significantly harder to identify the true attacker IP in firewall and IDS logs.

---

### Task 6 — Packet Fragmentation

#### Command

```bash
sudo nmap -sS -p80 -f <TARGET_IP>      # Fragment into 8-byte chunks
sudo nmap -sS -p80 -ff <TARGET_IP>     # Fragment into 16-byte chunks
```

Fragmentation divides IP packets into smaller units. Some older or misconfigured stateless firewalls and packet filters do not perform complete fragment reassembly before applying inspection rules, potentially allowing fragmented scan probes to bypass filtering.

**Fragmentation calculation:**

A 64-byte TCP segment with the `-ff` flag (16-byte fragments) produces **4 fragments**.

---

### Task 7 — Idle Scan (`-sI`) — Zombie Scan

The Idle Scan is considered the most covert active scanning technique available in Nmap. It leverages a third-party host — referred to as a "zombie" — to perform reconnaissance indirectly, ensuring the scanner's IP address never appears in the target's logs.

#### Command

```bash
nmap -sI <ZOMBIE_IP> <TARGET_IP>
```

Lab example:

```bash
nmap -sI 10.10.5.5 <TARGET_IP>   # Using a network printer as the zombie host
```

#### How the Idle Scan Works

1. Scanner queries the zombie's IP ID (IPID) sequence to establish a baseline.
2. Scanner sends a SYN packet to the target, forging the zombie's IP as the source.
3. If the target port is **open**, it sends SYN-ACK to the zombie, which responds with RST, incrementing the zombie's IPID.
4. If the target port is **closed**, the target sends RST to the zombie, which is ignored — IPID does not increment.
5. Scanner queries the zombie's IPID again and compares to the baseline. An increment indicates an open port.

**Zombie host requirements:**

- Must be idle (minimal network traffic to avoid IPID pollution).
- Must use a predictable, incremental IPID sequence (many modern systems use randomized IPIDs — this limits usable zombie hosts).

---

### Task 8 — Interpreting Nmap Output

#### Commands

```bash
sudo nmap -sS <TARGET_IP>
sudo nmap -sS --reason <TARGET_IP>     # Show reason for each port state
sudo nmap -sS -vv <TARGET_IP>          # Increase output verbosity
```

| Flag | Purpose |
|---|---|
| `--reason` | Displays the specific packet response that determined a port's state |
| `-vv` | Increases output verbosity for detailed real-time scan information |

Lab result:

| Query | Result |
|---|---|
| Reason Nmap classified a port as open | `syn-ack` |

`--reason` is particularly valuable when validating scan results in complex network environments, as it provides explicit evidence for each classification decision rather than requiring the analyst to infer from context.

---

## Module 4 — Post-Port Scanning

### Objectives

- Fingerprint service versions with `--sV`.
- Perform OS detection via TCP/IP stack analysis.
- Use integrated traceroute for network topology mapping.
- Store scan results in multiple output formats.
- Leverage the Nmap Scripting Engine (NSE) for advanced enumeration and vulnerability detection.

---

### Task 2 — Service Version Detection (`-sV`)

#### Command

```bash
sudo nmap -sV --version-light <TARGET_IP>
```

The `-sV` flag instructs Nmap to establish full TCP connections with discovered services to collect version banners and service metadata.

| Flag | Behavior |
|---|---|
| `--version-light` | Fewer probes; faster but less thorough |
| `--version-all` | Maximum probe intensity; most complete results |

> **Important:** Because `-sV` requires completing the TCP handshake, it is incompatible with SYN Scan (`-sS`). Enabling service detection automatically forces full TCP Connect behavior.

Lab results:

| Finding | Result |
|---|---|
| Version identified on port 143 | Pombal imapd |
| Service without version (using `--version-light`) | rpcbind |

---

### Task 3 — OS Detection and Traceroute

#### Commands

```bash
sudo nmap -sS -O <TARGET_IP>
nmap -sS --traceroute <TARGET_IP>
```

**OS Detection (`-O`):** Nmap infers the target's operating system by analyzing multiple characteristics of the TCP/IP stack response, including default TTL values, TCP sequence number behavior, and packet flag handling. Even without an exact match, Nmap typically identifies the OS family.

**Integrated Traceroute (`--traceroute`):** Maps the network path between the scanner and the target, identifying intermediate hops and aiding in infrastructure topology analysis.

Lab results:

| Finding | Result |
|---|---|
| Closest OS match identified | Linux |
| Distance to target | 3 hops |

---

### Task 4 — Output Formats

Nmap supports multiple output formats suitable for different post-scan workflows.

#### Commands

```bash
nmap -oN scan.nmap <TARGET_IP>     # Normal (human-readable)
nmap -oG scan.gnmap <TARGET_IP>    # Grepable
nmap -oX scan.xml <TARGET_IP>      # XML (machine-parseable)
nmap -oA scan <TARGET_IP>          # All three simultaneously
```

| Flag | Format | Best Use Case |
|---|---|---|
| `-oN` | Normal | Manual analysis; human-readable review |
| `-oG` | Grepable | Rapid filtering with `grep`, `awk`, `cut` |
| `-oX` | XML | Integration with automated tools (Metasploit, Faraday, Dradis) |
| `-oA` | All | Standard practice during penetration tests and audits |

Lab results:

| Query | Result |
|---|---|
| Flag for grepable output | `-oG` |
| XML output supported | Yes |

> **Operational Recommendation:** Use `-oA` as the default during any engagement to ensure all output formats are captured simultaneously. Output files can be ingested into reporting frameworks or searched post-engagement without re-running scans.

---

### Task 5 — Nmap Scripting Engine (NSE)

The NSE extends Nmap's capabilities from port enumeration into full-service interrogation, vulnerability detection, and automated exploitation support.

#### Script Location

```bash
/usr/share/nmap/scripts
```

#### Commands

```bash
sudo nmap -sS -sC <TARGET_IP>                          # Run default scripts
sudo nmap -sS -n --script "http-robots.txt" <TARGET_IP> # Run specific script
sudo nmap -sS --script "http*" <TARGET_IP>              # Run all http-* scripts
```

#### NSE Script Categories

| Category | Purpose |
|---|---|
| `auth` | Credential and authentication testing |
| `broadcast` | LAN-based service discovery |
| `brute` | Brute-force credential attacks |
| `default` | Run with `-sC`; standard enumeration scripts |
| `discovery` | Network and service enumeration |
| `dos` | Denial-of-service testing |
| `exploit` | Active exploitation modules |
| `external` | Queries external databases (Shodan, etc.) |
| `fuzzer` | Sends malformed/unexpected input |
| `intrusive` | Active, potentially disruptive probes |
| `malware` | Checks for malware indicators or backdoors |
| `safe` | Non-disruptive, low-risk information gathering |
| `version` | Enhanced service version detection |
| `vuln` | Checks for known CVEs and vulnerabilities |

Lab results:

| Query | Result |
|---|---|
| Script checking `robots.txt` disallow entries | `http-robots.txt` |
| Script checking CVE-2015-1635 (MS15-034) | `http-vuln-cve2015-1635` |
| Page title discovered on port 80 | Welcome to nginx on Debian! |
| SSH key algorithm (SHA2-512) identified | `rsa-sha2-512` |

---

## Key Takeaways

This series builds a complete Nmap-based reconnaissance workflow applicable to real-world penetration testing engagements:

| Phase | Technique | Primary Tool/Flag |
|---|---|---|
| Host Discovery | ARP, ICMP, TCP/UDP Ping | `-PR`, `-PE`, `-PS`, `-PA`, `-PU` |
| Port Scanning | SYN, Connect, UDP | `-sS`, `-sT`, `-sU` |
| Evasion | Fragmentation, Decoy, Idle Scan | `-f`, `-D`, `-sI` |
| Firewall Analysis | ACK, Window, Xmas, FIN, Null | `-sA`, `-sW`, `-sX`, `-sF`, `-sN` |
| Service Fingerprinting | Version detection, NSE | `-sV`, `-sC`, `--script` |
| OS Fingerprinting | TCP/IP stack analysis | `-O` |
| Output & Reporting | Multi-format export | `-oA`, `-oN`, `-oG`, `-oX` |

> **Note:** All techniques described in this write-up should only be applied to systems for which explicit written authorization has been obtained. Unauthorized scanning constitutes a violation of computer crime laws in most jurisdictions.

---

*Write-up produced as part of an ongoing offensive security portfolio. All exercises were conducted within the TryHackMe learning platform's isolated lab environments.*
