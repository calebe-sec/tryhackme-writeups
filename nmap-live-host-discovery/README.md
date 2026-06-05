# TryHackMe - Nmap Live Host Discovery

## Introduction

In this TryHackMe room, I explored live host discovery techniques using Nmap. The goal was to understand how different protocols and scanning methods can be used to identify active devices on a network before performing more advanced enumeration steps.

During the exercises, fundamental networking concepts were covered, including ARP, ICMP, TCP, and UDP, as well as the use of various Nmap options to perform host discovery efficiently.

---

## Objectives

* Understand how the ARP protocol works in local area networks.

* Understand the host discovery process using ICMP.

* Learn different host discovery techniques with Nmap.

* Use TCP and UDP scans to identify active devices.

* Perform reverse DNS resolution to obtain additional information about hosts.

---

## Task 2 – ARP Requests

This step demonstrated how the ARP (Address Resolution Protocol) is used to discover the MAC address associated with an IP address within the same local network.

### Observed Results

* Devices that viewed the ARP request: **4**
* Computer6 received the ARP request: **no**
* Computer6 responded to the ARP request: **yes**

### Concepts Learned

ARP operates only within the broadcast domain of the local network. When a host wants to communicate with another device on the same network, it first needs to discover its MAC address through an ARP Request.

--

## Task 3 – ICMP Ping and ARP

This activity analyzed the complete communication process before executing a ping.

### Observations

Before sending the ICMP Echo Request, the computer needed to:

1. Send an ARP Request.

2. Receive an ARP Response.

3. Store the IP ↔ MAC association in cache.

### Results

* Packet sent before ping: **ARP Request**
* Packet received before ping: **ARP Response**
* Computers that responded to the ping: **1**
* First device to respond to the first ARP Request: **router**
* First device to respond to the second ARP Request: **computer5**
* New ping required new ARP requests: **no**

### Concepts learned

After the IP/MAC mapping is stored in the system's ARP table, new ARP requests are usually not needed until the entry expires.

---

## Task 4 – List Scan

Nmap allows you to list targets without actually performing a scan.

### Commands Used

```bash
nmap -iL list_of_hosts.txt
nmap -sL TARGETS
```

### Results

* First IP of the 10.10.12.13/29 block:

```text
10.10.12.8
```

* Number of addresses in the range:

```text
10.10.0-255.101-125
```

Result:

```text
6400 hosts
```

### Concepts Learned

The `-sL` parameter only enumerates the targets without sending packets to the hosts, being useful for validating ranges before starting a scan.

---

## Task 5 – ARP Scan

In this activity, an ARP scan was used to identify active hosts on a local network.

### Command Used

```bash.nmap -PR -sn CONNECTION_IP/24

```

### Result

```text.1 active host found

```

### Concepts Learned

ARP-based discovery is extremely efficient in local area networks, as devices typically respond to ARP requests even when firewalls block ICMP packets.

---

## Task 6 – ICMP Host Discovery

Nmap supports different types of ICMP messages for host discovery.

### Commands Used

```bash.nmap -PE -sn 10.200.6.0/24
nmap -PP -sn 10.200.6.0/24
nmap -PM -sn 10.200.6.0/24

```

### Options Studied

| Option | Function |

| ----- | ------------------------- | | -PE | ICMP Echo Request |

| -PP | ICMP Timestamp Request |

| -PM | ICMP Address Mask Request |

### Concepts Learned

Depending on the network configuration, certain types of ICMP messages may be blocked. Knowing different methods increases the chances of identifying active hosts.

---

## Task 7 – TCP and UDP Ping Scans

In this step, discovery techniques using TCP SYN, TCP ACK, and UDP Ping were explored.

### Commands Used

```bash
nmap -PS -sn 10.200.6.0/24

sudo nmap -PA -sn 10.200.6.0/24

sudo nmap -PU -sn 10.200.6.0/24

masscan 10.200.6.0/24 -p443

masscan 10.200.6.0/24 -p80,443

masscan 10.200.6.0/24 -p22-25

```

### Results

* TCP Ping without administrative privileges:

```text TCP SYN Ping
```

* TCP Ping requiring privileges:

```text TCP ACK Ping
```

* Option to execute TCP SYN Ping on port 23:

```bash
-PS23
```

### Concepts learned

When ICMP is blocked, TCP and UDP-based techniques can be used to identify active hosts. This scenario is common in corporate environments.

---

## Task 8 – Reverse DNS Lookup

Nmap also allows you to perform reverse DNS lookups during host discovery.

### Option used

```bash
-R
```

### Concepts learned

Reverse DNS resolution can reveal the names of specific machines, servers, or functions within an infrastructure, aiding in the reconnaissance process.

Conclusion

This room provided an in-depth look at the host discovery mechanisms available in Nmap. Methods based on ARP, ICMP, TCP, and UDP were explored, as well as auxiliary techniques such as List Scan and Reverse DNS Lookup.

At the end of the activity, I gained a more solid understanding of how the reconnaissance phase works in security testing and how to choose the most appropriate host discovery technique for different network scenarios.