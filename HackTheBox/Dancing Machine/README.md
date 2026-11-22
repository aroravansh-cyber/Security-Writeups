# Dancing — Hack The Box

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![OS](https://img.shields.io/badge/OS-Windows-blue?logo=windows)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Dancing |
| Platform | Hack The Box |
| OS | Windows |
| Difficulty | Very Easy |
| XP Earned | 150 |
| Creator | HTB-Bot |
| Release Date | 30 September 2021 |
| Service | SMB |
| Port | 445 |
| Accessible Share | WorkShares |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/395)

## Introduction

Dancing is a Very Easy Windows machine that introduces SMB enumeration, share discovery, anonymous access, and file transfers.

## 1. VPN Connectivity

Connect to the Hack The Box network using OpenVPN.

```bash
sudo openvpn filename.ovpn
```

Verify connectivity with the target:

```bash
ping <TARGET_IP>
```

## 2. Enumeration

Perform an Nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

### Open Ports

| Port | Service | Description |
|---|---|---|
| 445/tcp | microsoft-ds | SMB |

The scan identified the SMB service running on port 445.

## 3. SMB Enumeration

SMB stands for Server Message Block. It is a network protocol used for sharing files, printers, and other resources.

SMB commonly operates on TCP port 445.

Used `smbclient` to enumerate available SMB shares.

```bash
smbclient -L //<TARGET_IP>/
```

The server returned four available shares.

### Available Shares

| Share | Access |
|---|---|
| ADMIN$ | Restricted |
| C$ | Restricted |
| IPC$ | Restricted |
| WorkShares | Accessible |

## 4. Initial Access — Anonymous SMB

After enumerating the shares, identified `WorkShares` as accessible without a password.

Connected using:

```bash
smbclient //<TARGET_IP>/WorkShares
```

When prompted for a password, pressed Enter to authenticate with a blank password.

Successfully accessed the SMB share.

## 5. File Enumeration

Listed the available files and directories inside the SMB share.

```bash
smb: \> ls
```

The share contained user directories and accessible files.

Navigated through the available directories to locate the flag.

```bash
smb: \> cd <DIRECTORY>
smb: \> ls
```

## 6. Downloading the Flag

Used the `get` command within the SMB shell to download the discovered file.

```bash
smb: \> get <FILENAME>
```

Exit the SMB session:

```bash
smb: \> exit
```

Read the downloaded file:

```bash
cat <FILENAME>
```

**Flag:** `REDACTED`

## Vulnerabilities Identified

- Anonymous SMB share access.
- Insufficient access restrictions on shared files.
- Sensitive information exposed through accessible SMB directories.
- Weak SMB share permissions.

## Key Learnings

- Understanding SMB and Server Message Block.
- Identifying SMB services using Nmap.
- Enumerating SMB shares using smbclient.
- Understanding the `-L` flag.
- Anonymous SMB authentication.
- Accessing shared directories.
- Listing remote files and directories.
- Downloading files using the `get` command.

## Mitigation

- Disable anonymous access to SMB shares.
- Apply appropriate authentication requirements.
- Restrict share permissions using least privilege.
- Avoid storing sensitive files in publicly accessible shares.
- Monitor SMB access and authentication activity.

## Conclusion

Dancing demonstrates how improperly configured SMB shares can expose files to unauthorized users.

By enumerating SMB shares, identifying anonymous access to `WorkShares`, and downloading the accessible flag, the machine was successfully completed.

**Result: Machine Successfully Pwned.**
