![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | DoubleTake |
| Platform | Hack The Box |
| OS | Linux |
| Difficulty | Easy |
| XP Earned | 450 |
| Creator | TheCyberGeek |
| Release Date | 12 November 2026 |
| Initial User | alpine |
| User | bob |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/1018)

## Enumeration

Started with an Nmap scan to identify open ports and running services.

```bash
nmap doubletake.htb -sC -sV -O -oN nmap_scan
```

### Open Ports

| Port | Service | Version |
|---|---|---|
| 22 | SSH | OpenSSH 10.6 |
| 80 | HTTP | Nginx 1.24.0 |

## HTTP Enumeration

Visited the web application on port 80.

```bash
curl -i http://doubletake.htb/
```

The application returned an HTTP 401 Unauthorized response with the authentication realm:

```text
Zentara
```

Performed directory enumeration using Gobuster.

```bash
gobuster dir -u http://doubletake.htb \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-x env,md,txt,config,json,php,html
```

Discovered several endpoints, including:

- `/docs`
- `/files`
- `/status`
- `/usage`
- `/settings`

All discovered endpoints required authentication.

## Initial Access — SSH Authentication Bypass

Identified that the machine was potentially vulnerable to **CVE-2026-123158 (SSHole)**.

This vulnerability affects certain OpenSSH configurations using musl libc without PAM.

The issue involves password authentication incorrectly accepting a specially sized password under specific conditions.

The vulnerability was tested against the `alpine` account.

```bash
ssh alpine@doubletake.htb
```

After the authentication bypass, obtained initial access as:

```bash
whoami
```

```text
alpine
```

## Lateral Movement — Firefox Credential Discovery

Enumerated the filesystem to identify other users and potentially exposed credentials.

A Firefox profile was discovered inside the Alpine user's home directory.

Located the saved login information:

```bash
cat ~/.mozilla/firefox/zentara.default/logins.json
```

The file contained Base64-encoded authentication information.

Decoded the username and password fields:

```bash
echo "Ym9i" | base64 -d
```

The recovered username was:

```text
bob
```

The saved credentials provided access to the Zentara application and another SSH account.

### SSH Access — bob

```bash
ssh bob@doubletake.htb
```

Successfully authenticated as `bob`.

### User Flag

```bash
cat /home/bob/user.txt
```

**User Flag:** `REDACTED`

## Web Application Enumeration — Zentara

Used the recovered credentials to authenticate to the web application.

Explored the Zentara dashboard and discovered the `/status` endpoint.

```text
http://doubletake.htb/status
```

The status information revealed that the application was running inside a Docker container.

### Docker Information

| Component | Details |
|---|---|
| Engine | Docker |
| Version | 29.5.0 |
| Environment | Linux |
| API | Docker API |

Further investigation identified **CVE-2026-41567**, an archive decompression vulnerability affecting the Docker environment.

## Privilege Escalation — Docker Archive Decompression

The vulnerability involves Docker resolving archive decompression binaries from the container filesystem instead of the expected host environment.

This can allow an attacker to execute a modified decompression binary with Docker daemon privileges.

### Vulnerability Analysis

The relevant binary identified inside the container was:

```text
/usr/bin/xz
```

The exploitation involved:

1. Identifying the vulnerable Docker version.
2. Investigating the archive deployment functionality.
3. Understanding the decompression binary resolution issue.
4. Demonstrating the impact of a modified `xz` executable.
5. Triggering the vulnerable archive processing functionality.

### Archive Deployment

The application exposed a deployment endpoint:

```text
/deploy
```

Archive uploads were processed by the Docker-backed deployment functionality.

The vulnerable archive handling mechanism provided a path from container-level access to host-level privileges.

## Root Access

Successfully obtained root-level access through the Docker archive decompression vulnerability.

Verified privileges:

```bash
whoami
```

Expected output:

```text
root
```

### Root Flag

```bash
cat /root/root.txt
```

**Root Flag:** `REDACTED`

## Vulnerabilities Identified

- OpenSSH authentication bypass — CVE-2026-123158.
- Exposed Firefox saved credentials.
- Insecure credential storage.
- Docker archive decompression vulnerability — CVE-2026-41567.
- Container-to-host privilege escalation.

## Key Learnings

- Nmap service enumeration.
- HTTP Basic Authentication analysis.
- Directory enumeration using Gobuster.
- OpenSSH authentication vulnerabilities.
- Firefox profile and credential analysis.
- Base64 decoding.
- Credential reuse and lateral movement.
- Docker architecture and API enumeration.
- Container escape concepts.
- Privilege escalation through vulnerable archive processing.

## Conclusion

DoubleTake demonstrates a multi-stage compromise involving an OpenSSH authentication bypass, exposed browser credentials, and a Docker archive decompression vulnerability.

The attack path moves from initial access as `alpine` to lateral movement as `bob`, followed by Docker-based privilege escalation to root.

**Result: Machine Successfully Pwned.**
