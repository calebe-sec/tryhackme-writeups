# TryHackMe - Nmap Advanced Port Scans

## Introduction

In this TryHackMe room, I explored advanced port scanning techniques using Nmap. The main focus was understanding alternative enumeration methods that can help evade firewalls, reduce detection, and gather information in more restrictive environments.

Scans based on TCP flag manipulation, spoofing techniques, packet fragmentation, and the Idle Scan — considered one of the most discreet techniques available in Nmap — were studied.

---

## Objectives

* Understand how advanced TCP scans work.
* Learn evasion techniques using flag manipulation.
* Explore ACK, Window, and Maimon scans.
* Understand the concepts of IP Spoofing and Decoy Scanning.
* Study packet fragmentation.
* Learn how Idle Scan works.
* Interpret detailed results using advanced Nmap features.

---

## Task 2 – Null, FIN, and Xmas Scans

This activity explored three classic enumeration techniques based on TCP flag manipulation.

### Commands Studied

```bash
sudo nmap -sN MACHINE_IP

sudo nmap -sF MACHINE_IP

sudo nmap -sX MACHINE_IP
```

### Results

* Flags activated in a Null Scan:

```text
0
```

* Flags activated in a FIN Scan:

```text
1
```

* Flags activated in a Xmas Scan:

```text
3
```

* Ports identified as open|filtered during the FIN Scan:

```text
9
```

* Ports identified as open|filtered during the Null Scan:

```text
9
```

### Concepts Learned

#### Null Scan (-sN)

No TCP flags are activated. The packet is sent virtually empty.

#### FIN Scan (-sF)

Only the FIN flag is sent.

#### Xmas Scan (-sX)

The FIN, PSH, and URG flags are activated simultaneously, making the packet resemble a "Christmas tree" due to the number of active flags.

These techniques exploit differences in TCP stack implementation to identify open ports without using the traditional TCP connection process.

---

## Task 3 – Maimon Scan

Maimon Scan is a less common variation of TCP flag-based scans.

### Command Studied

```bash
sudo nmap -sM MACHINE_IP
```

### Result

* Number of flags activated:

```text
2
```

### Concepts Learned

Maimon Scan uses the following flags simultaneously:

* FIN
* ACK

This technique was developed to exploit specific behaviors observed in certain Unix and BSD systems. Although it currently has more limited applications, it remains an important technique for understanding how Nmap's advanced scans work.

---

## Task 4 – ACK, Window, and Custom Scans

This step explored scans mainly used for filtering analysis and firewall behavior.

### Commands Studied

```bash
sudo nmap -sA MACHINE_IP

sudo nmap -sW MACHINE_IP

sudo nmap --scanflags RST MACHINE_IP
```

### Results

* Number of flags used by Window Scan:

```text
1
```

* Flag used in a custom scan with `--scanflags`:

```text
RST
```

* Ports identified as unfiltered during the ACK Scan:

```text
5
```

* New port discovered:

```text
443
```

* Existence of associated service:

```text
yes
```

### Concepts Learned

#### ACK Scan (-sA)

Used to determine whether ports are filtered by firewalls.

#### Window Scan (-sW)

Based on analyzing the returned TCP window value to infer the port state.

#### Custom Scans (--scanflags)

Allows you to manually define which TCP flags will be used during enumeration.

These approaches are especially useful when the goal is to map network filtering rules and identify services hidden by firewalls.

---

## Task 5 – IP Spoofing and Decoy Scan

This activity studied techniques for hiding the scan origin.

### Commands Studied

```bash
nmap -S SPOOFED_IP MACHINE_IP

nmap -e NET_INTERFACE -Pn -S SPOOFED_IP MACHINE_IP

nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME MACHINE_IP
```

### Results

* Set a fake source IP address:

```bash
-S 10.10.10.11
```

* Use multiple fake source addresses:

```bash
-D 10.10.20.21,10.10.20.28,ME
```

### Concepts Learned

#### Source IP Spoofing

Allows you to change the apparent IP address used during the scan.

#### Decoy Scan

Adds multiple fake addresses to the scan to make it harder to identify the true originator of the activity.

These techniques can make log analysis more difficult and increase discretion during security assessments.

---

## Task 6 – Packet Fragmentation

This step explored IP packet fragmentation.

### Command Studied

```bash
sudo nmap -sS -p80 -f MACHINE_IP
```

### Result

If a TCP segment has 64 bytes and the `-ff` option is used:

```text
4 fragments
```

### Concepts Learned

Fragmentation divides larger packets into multiple smaller IP fragments. In some scenarios, this technique can help evade traffic inspection mechanisms or firewalls that do not perform proper packet reassembly.

---

## Task 7 – Idle Scan

Idle Scan is considered one of the most discreet techniques available in Nmap.

### Command Studied

```bash
nmap -sI ZOMBIE_IP MACHINE_IP
```

### Result

* Using a network printer as a zombie:

```bash
-sI 10.10.5.5
```

### Concepts Learned

Idle Scan uses a third device, called a zombie host, to perform enumeration indirectly.

Characteristics:

* Completely hides the scanner's IP address.
* Reduces traces on the target.
* Requires a predictable and rarely used zombie host.

It is a classic anonymization technique during reconnaissance processes.

---

## Task 8 – Interpreting Nmap Results

This activity explored features that provide additional information about scan results.

### Commands Studied

```bash
sudo nmap -sS MACHINE_IP

sudo nmap -sS --reason MACHINE_IP

sudo nmap -sS -vv MACHINE_IP
```

### Result

* Reason presented by Nmap for considering a port open:

```text
syn-ack
```

### Concepts Learned

#### --reason

Displays the exact reason why Nmap classified a port in a given state.

#### -vv

Increases the verbosity of the output, providing more detailed information during execution.

These features are extremely useful for validating results and understanding how Nmap arrived at its conclusions.

---

## Conclusion

This room significantly deepened my understanding of advanced port enumeration techniques using Nmap. Less conventional methods were explored, including Null Scan, FIN Scan, Xmas Scan, Maimon Scan, ACK Scan, Window Scan, Idle Scan, and spoofing techniques.

Furthermore, I learned important concepts related to firewall evasion, scan anonymization, and detailed interpretation of the results generated by the tool.

This knowledge broadens the ability to perform reconnaissance in complex environments and reinforces the understanding of the inner workings of network protocols used during security testing.