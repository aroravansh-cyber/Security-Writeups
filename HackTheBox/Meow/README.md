![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Meow |
| Platform | Hack The Box |
| OS | Linux |
| Difficulty | Very Easy |
| XP Earned | 150 |
| Creator | HTB-Bot |
| Release Date | 30 September 2021 |
| Service | Telnet |
| Username | root |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/394)

## Introduction

Meow is a Very Easy Linux machine designed for beginners. It introduces basic networking concepts, VPN connectivity, port scanning, and remote access through Telnet.

## 1. VPN Connectivity

First, connect to the Hack The Box network using OpenVPN.

```bash
sudo openvpn filename.ovpn
```

Verify connectivity with the target:

```bash
ping <TARGET_IP>
```

Successful ICMP responses indicate that the target is reachable.

## 2. Enumeration

Performed an Nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

### Open Ports

| Port | Service |
|---|---|
| 23/tcp | Telnet |

The scan identified Telnet running on port 23.

## 3. Initial Access — Telnet

Telnet is an unencrypted remote access protocol that allows users to interact with a remote system through a command-line interface.

Connected to the target using:

```bash
telnet <TARGET_IP>
```

During authentication, discovered that the `root` username allowed login with a blank password.

Successfully obtained root access.

Verify the current user:

```bash
whoami
```

Expected output:

```text
root
```

## 4. Root Flag

Since access was obtained as root, navigated to the root home directory.

```bash
cd /root
ls -la
```

Read the flag:

```bash
cat root.txt
```

**Root Flag:** `REDACTED`

## Vulnerabilities Identified

- Telnet exposed to the network.
- Root account accessible without a password.
- Insecure remote authentication.
- Lack of secure remote administration.

## Key Learnings

- Understanding Virtual Machines.
- Terminal and command-line basics.
- Connecting to HTB using OpenVPN.
- Testing connectivity using Ping.
- Network enumeration using Nmap.
- Identifying open ports and services.
- Understanding Telnet.
- Identifying weak authentication.
- Obtaining root access.

## Mitigation

- Disable Telnet and use SSH.
- Never allow root login with an empty password.
- Enforce strong authentication.
- Restrict remote access using firewall rules.
- Regularly audit exposed network services.

## Conclusion

Meow demonstrates the importance of basic network security and secure authentication.

An exposed Telnet service combined with an unrestricted root account allowed complete system access.

**Result: Machine Successfully Pwned.**
