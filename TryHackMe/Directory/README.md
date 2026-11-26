# Directory | TryHackMe Write-up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?logo=tryhackme)
![Category](https://img.shields.io/badge/Category-DFIR-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Hard-orange)
![Focus](https://img.shields.io/badge/Focus-Network_Forensics-purple)

**Author:** [Vansh Arora](https://github.com/aroravansh-cyber)

**Room:** [Directory – TryHackMe](https://tryhackme.com/room/directorydfirroom)

**Date:** 26 November 2026

**Tools Used:** Wireshark, TShark, Hashcat, Python

---

## Overview

The Directory room is a Digital Forensics and Incident Response (DFIR) challenge involving the investigation of a compromised Windows Active Directory environment.

A small music company was targeted by a threat actor. Larry, the company's Art Director, discovered a suspicious note on his Desktop.

The only available forensic artifact was a network packet capture (`traffic.pcap`) collected using Wireshark.

The objective was to analyze the network traffic, identify the initial access method, recover compromised credentials, investigate the attacker's remote activity, and retrieve the final flag.

## Investigation Details

| Attribute | Details |
|---|---|
| Room | Directory |
| Platform | TryHackMe |
| Difficulty | Hard |
| Category | Digital Forensics |
| Artifact | traffic.pcap |
| Victim IP | 10.0.2.75 |
| Attacker IP | 10.0.2.74 |
| Main Protocols | TCP, HTTP, LDAP, Kerberos, SMB |
| Remote Access | WinRM (5985) |

## Attack Overview

The investigation revealed the following attack sequence:

1. Network reconnaissance and port scanning.
2. Active Directory username enumeration.
3. AS-REP Roasting against a domain account.
4. Offline password recovery.
5. Authentication through WinRM.
6. Registry hive extraction.
7. Creation of a note on the victim's Desktop.

---

## 1. Initial Network Reconnaissance

The first step was analyzing the PCAP to identify the machines involved and understand the initial network activity.

The attacker performed a TCP port scan against the victim machine.

### Identifying the attacker and victim

The conversation statistics in Wireshark revealed two primary IP addresses:

- Attacker: `10.0.2.74`
- Victim: `10.0.2.75`

### Identifying open ports

A TCP SYN-ACK response indicates that a TCP port is open.

We can use TShark to extract the responding ports from the initial scan.

```bash
tshark -r traffic.pcap -c 3610 \
-Y "tcp.flags.syn == 1 && tcp.flags.ack == 1 && ip.src == 10.0.2.75 && ip.dst == 10.0.2.74" \
-T fields -e tcp.srcport | sort -n | uniq | paste -sd ','
```

### Results

The attacker initially discovered the following open ports:

```text
53,80,88,135,139,389,445,464,593,636,3268,3269,5357
```

Port 5985 was observed later during the WinRM connection, not during the initial scan.

---

## 2. Active Directory Username Enumeration

After reconnaissance, the attacker performed username enumeration against the Active Directory environment using Kerberos.

Kerberos authentication traffic can reveal whether a username exists within a domain.

### Understanding Kerberos responses

Two important error codes were observed:

| Error Code | Meaning |
|---|---|
| 6 | KDC_ERR_C_PRINCIPAL_UNKNOWN |
| 25 | KDC_ERR_PREAUTH_REQUIRED |

An unknown principal indicates that the username does not exist. A pre-authentication-required response indicates that the account exists but normally requires pre-authentication.

The attacker identified four valid usernames:

```text
john.doe
ranith.kays
joan.ray
larry.doe
```

However, the account `larry.doe` returned an AS-REP response without requiring Kerberos pre-authentication.

### Username Enumeration

The following command extracts Kerberos usernames and domain names:

```bash
tshark -r traffic.pcap \
-Y "kerberos" \
-T fields \
-e kerberos.CNameString \
-e kerberos.crealm
```

To display them in domain\username format:

```bash
tshark -r traffic.pcap -Y "kerberos" \
-T fields -e kerberos.CNameString -e kerberos.crealm \
| awk 'NF==2 {print $2 "\\" $1}'
```

### Finding

The vulnerable account was:

```text
DIRECTORY.THM\larry.doe
```

---

## 3. AS-REP Roasting and Hash Extraction

The account `larry.doe` did not require Kerberos pre-authentication.

This configuration allowed the attacker to request an AS-REP response containing encrypted authentication data that could be subjected to offline password guessing.

This technique is known as **AS-REP Roasting**.

### Extracting the Kerberos response

The following TShark command extracts AS-REP messages:

```bash
tshark -r traffic.pcap \
-Y "kerberos.msg_type == 11" \
-T fields \
-e kerberos.CNameString \
-e kerberos.etype \
-e kerberos.encryptedKDCREPData_cipher
```

The captured encrypted data was reconstructed into a format suitable for password recovery.

The relevant encryption type was:

```text
23 - RC4-HMAC
```

### Hash identification

The last 30 characters of the captured hash were:

```text
55616532b664cd0b50cda8d4ba469f
```

---

## 4. Password Recovery

After extracting the AS-REP data, the next step was attempting offline password recovery.

Hashcat supports AS-REP hashes through mode `18200`.

### Password recovery command

```bash
hashcat -m 18200 -a 0 larry.hash rockyou.txt
```

Where:

| Option | Description |
|---|---|
| `-m 18200` | AS-REP etype 23 hash mode |
| `-a 0` | Dictionary attack |
| `larry.hash` | Extracted AS-REP hash |
| `rockyou.txt` | Password dictionary |

### Result

The password was successfully recovered:

```text
Password1!
```

The attacker now had valid credentials for the domain account.

---

## 5. WinRM Traffic Analysis

After obtaining the credentials, the attacker established a remote session with the victim through Windows Remote Management (WinRM).

WinRM commonly uses TCP port 5985 for HTTP-based remote management.

### Identifying WinRM

The PCAP contained HTTP traffic on port 5985.

The relevant traffic used:

```text
HTTP
multipart/encrypted
NTLMSSP
SPNEGO
```

The PowerShell Remoting messages were encrypted, so their contents could not be directly interpreted from the packet payload.

### Decrypting the traffic

Since the user's password had been recovered, it was possible to derive the NTLM session key used for message protection.

The investigation required examining the NTLM authentication exchange and decrypting the sealed WinRM messages.

An important observation was that the command connection and authentication setup used separate TCP streams. They needed to be analyzed independently while preserving message order and the RC4 sealing state.

After decrypting the WinRM traffic, the attacker's commands could be recovered.

---

## 6. Post-Exploitation Activity

The decrypted PowerShell Remoting traffic revealed that the attacker executed several commands on the victim machine.

### Commands identified

**First command:**

```powershell
whoami
```

**Second command:**

```powershell
reg save HKLM\SYSTEM C:\SYSTEM
```

**Third command:**

```powershell
reg save HKLM\SAM C:\SAM
```

**Fourth command:**

```powershell
reg save HKLM\SECURITY C:\SECURITY
```

The attacker saved important Windows registry hives.

These files are valuable during credential analysis because they contain information associated with local account credentials and security secrets.

### Creating the suspicious note

The decrypted traffic also revealed the following commands:

```powershell
cd ..\Desktop\
```

The attacker then created a note on the Desktop:

```powershell
echo "THM{Ya_G0t_R0aSt3d!}" > note.txt
```

This explained the suspicious note discovered by Larry.

---

## 7. Question Answers

| No. | Question | Answer |
|---|---|---|
| 1 | Initially open ports | `53,80,88,135,139,389,445,464,593,636,3268,3269,5357` |
| 2 | Username | `DIRECTORY.THM\larry.doe` |
| 3 | Last 30 hash characters | `55616532b664cd0b50cda8d4ba469f` |
| 4 | User password | `Password1!` |
| 5 | Second and third commands | `reg save HKLM\SYSTEM C:\SYSTEM,reg save HKLM\SAM C:\SAM` |
| 6 | Flag | `THM{Ya_G0t_R0aSt3d!}` |

---

## 8. Key Takeaways

- Network packet captures can provide evidence of an entire intrusion when endpoint logging is unavailable.
- Kerberos responses can expose accounts configured without pre-authentication.
- AS-REP Roasting allows offline password recovery against vulnerable domain accounts.
- Recovered credentials can help investigate other encrypted network sessions.
- WinRM traffic can reveal remote command execution when the session-protection mechanism is properly analyzed.
- Windows registry hives are important artifacts during credential-related investigations.

## References

- [TryHackMe – Directory](https://tryhackme.com/room/directorydfirroom)
- [Directory – Packet Capture Analysis](https://classroom.anir0y.in/post/thm-room-directory/)
- [Directory – Additional Walkthrough](https://0xb0b.gitbook.io/writeups/tryhackme/2025/directory)

---

## Completion Summary

**Room:** Directory

**Platform:** TryHackMe

**Category:** Digital Forensics and Incident Response

**Key Skills Practiced:** Wireshark, TShark, Kerberos, AS-REP Roasting, Hashcat, NTLM, WinRM, and Windows Registry Analysis.

**Final Flag:**

```text
THM{Ya_G0t_R0aSt3d!}
```
