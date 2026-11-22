# Fawn — Hack The Box

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Fawn |
| Platform | Hack The Box |
| OS | Linux |
| Difficulty | Very Easy |
| XP Earned | 150 |
| Creator | HTB-Bot |
| Release Date | 30 September 2021 |
| Service | FTP |
| FTP Version | vsftpd 3.0.3 |
| Username | anonymous |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/393)

## Introduction

Fawn is a Very Easy Linux machine that introduces FTP enumeration, anonymous authentication, and file transfers.

## 1. VPN Connectivity

Connect to the Hack The Box network using OpenVPN.

```bash
sudo openvpn filename.ovpn
```

Check connectivity with the target:

```bash
ping <TARGET_IP>
```

## 2. Enumeration

Perform an Nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

### Open Ports

| Port | Service | Version |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.3 |

The scan also identified the operating system type as Unix.

## 3. FTP Enumeration

FTP stands for File Transfer Protocol. It commonly operates on TCP port 21.

Display the FTP client help menu:

```bash
ftp -?
```

FTP transmits data without encryption. SFTP (SSH File Transfer Protocol) is a secure alternative based on SSH.

## 4. Initial Access — Anonymous FTP

Connected to the FTP service:

```bash
ftp <TARGET_IP>
```

At the login prompt, used the anonymous username:

```text
Name: anonymous
Password:
```

The server returned:

```text
230 Login successful.
```

This confirmed that anonymous FTP authentication was enabled.

## 5. File Enumeration

Listed the available files on the FTP server:

```bash
ls
```

The directory listing revealed a file available for download.

## 6. Downloading the Flag

Used the FTP `get` command to download the discovered file.

```bash
get <FILENAME>
```

Exit the FTP session:

```bash
bye
```

Read the downloaded file:

```bash
cat <FILENAME>
```

**Flag:** `REDACTED`

## Vulnerabilities Identified

- Anonymous FTP authentication enabled.
- Unrestricted access to sensitive files.
- FTP communication without encryption.
- Sensitive information exposed through file downloads.

## Key Learnings

- Understanding FTP and SFTP.
- Identifying FTP services using Nmap.
- FTP service version enumeration.
- Anonymous FTP authentication.
- Understanding FTP response codes.
- Listing remote directories.
- Downloading files using the FTP client.
- Basic Linux command-line operations.

## Mitigation

- Disable anonymous FTP access unless explicitly required.
- Restrict access to sensitive files.
- Use SFTP instead of unencrypted FTP.
- Apply appropriate authentication and permissions.
- Monitor FTP access and file transfers.

## Conclusion

Fawn demonstrates how anonymous FTP access can expose files to unauthorized users.

By identifying the FTP service, authenticating anonymously, and downloading an accessible file, the machine was successfully completed.

**Result: Machine Successfully Pwned.**
