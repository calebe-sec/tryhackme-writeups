# TryHackMe - Nmap Basic Port Scans

## Introduction

In this TryHackMe room, I deepened my knowledge of the main port scanning methods available in Nmap. The focus was on understanding the inner workings of TCP Connect, TCP SYN, and UDP scans, as well as understanding how Nmap interprets the different port states during the enumeration process.

These concepts are fundamental for reconnaissance activities in offensive security, allowing the identification of active services and potential attack surfaces in a system.

---

## Objectives

* Understand the port states identified by Nmap.

* Understand the operation of the TCP protocol during the connection process.

* Learn the difference between TCP Connect Scan and TCP SYN Scan.

* Explore UDP scanning techniques.

* Learn about optimization and performance control options in Nmap.

---

## Task 2 – Introduction to Port States

This activity presented fundamental concepts about TCP and UDP ports, common services, and the different states that can be identified by Nmap.

### Results

* Service that uses UDP 53 by default:

```text DNS

```

* Service that uses TCP 22 by default:

```text SSH

```

* Number of port states recognized by Nmap:

```text
6
```

* Most interesting state for a pentester:

```text Open

```

### Concepts learned

Nmap classifies ports into different states, such as:

* Open
* Closed
* Filtered
* Unfiltered
* Open|Filtered
* Closed|Filtered

Open ports are especially important during the reconnaissance phase, as they indicate accessible services that can be investigated later.

---

## Task 3 – Understanding the Three-Way Handshake

This step analyzed the TCP connection establishment process.

### Results

* Flag representing Reset:

```text RST

```

* Flag used to initiate a TCP connection:

```text SYN

```

### Concepts learned

A TCP connection is established through the famous Three-Way Handshake:

1. Client sends SYN.

2. Server responds SYN-ACK.

3. Client responds ACK.

This process is fundamental to understanding how different port scanning techniques work.

--

## Task 4 – TCP Connect Scan

The TCP Connect Scan uses the complete TCP connection process to check if a port is open.

### Command studied

```bash.nmap -sT MACHINE_IP

```

### Results

* FTP port status (21):

```text
open
```

* Service identified on port 53:

```text
domain
```

### Concepts learned

The `-sT` parameter executes a full TCP connection using operating system calls. This method does not require elevated privileges, but tends to be more easily logged compared to SYN Scan.

---

## Task 5 – TCP SYN Scan

This activity explored TCP SYN Scan, also known as Half-Open Scan.

### Studied Command

```bash.nmap -sS MACHINE_IP

```

### Results

* SYN-ACK packets successfully received:

```text
4
```

* Open ports identified:

```text
4
```

### Concepts learned

The SYN Scan sends only the initial SYN packet and analyzes the target's response.

Simplified flow:

1. Scanner sends SYN.

2. Server responds with SYN-ACK.

3. Scanner terminates the attempt with RST.

Because it does not complete the TCP connection, this technique is generally faster and less visible than the TCP Connect Scan.

---

## Task 6 – UDP Scan

This step analyzed the scanning of UDP services.

### Commands studied

```bash
nmap -sU --top-ports 10 MACHINE_IP

nmap -sU MACHINE_IP

```

### Results

* UDP port 161 state:

```text
closed
```

* Service identified:

```text
snmp

```

### Concepts learned

UDP enumeration is usually slower and more complex than TCP enumeration because many services do not respond when they receive unexpected packets.

Despite this, several critical services use UDP, including:

* DNS (53)
* DHCP (67/68)
* SNMP (161)
* NTP (123)

For this reason, UDP scans should not be ignored during security assessments.

---

## Task 7 – Port Scan Configuration

This activity explored additional options for customizing Nmap's behavior.

### Results

* Scan ports between 5000 and 5500:

```bash
-p5000-5500

```

* Ensure at least 64 parallel scans:

```bash
--min-parallelism=64

```

* Run the scan in the slowest and most discreet mode:

```bash
-T0

```

### Concepts Learned

Nmap offers several parameters to adjust the speed, parallelism, and aggressiveness of the scan.

Timing templates vary from:

* T0 (Paranoid)
* T1 (Sneaky)
* T2 (Polite)
* T3 (Normal)
* T4 (Aggressive)
* T5 (Insane)

These options allow you to adapt the enumeration strategy according to the analyzed environment.

---

## Conclusion

This room delved into the fundamental concepts related to port enumeration using Nmap. The internal mechanisms of TCP Connect, TCP SYN, and UDP scans were explored, as well as the different port states.