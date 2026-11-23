# Threat Hunting: Introduction | TryHackMe Write-up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-212C42)
![Path](https://img.shields.io/badge/Path-SOC%20Level%202-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Introductory-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

**Author:** Vansh ([@aroravansh-cyber](https://github.com/aroravansh-cyber))  
**Completed:** 2026-11-23  
**Room:** [Threat Hunting: Introduction](https://tryhackme.com/room/threathuntingintroduction)  
**Module:** [Threat Hunting](https://tryhackme.com/module/threat-hunting)  
**Tools & References:** MITRE ATT&CK, VirusTotal, Windows Event Logs, SIEM, EDR, Splunk

---

## Overview

Threat Hunting is the systematic and proactive search for malicious activity that may have bypassed existing security detection systems.

Unlike Incident Response, which reacts to security alerts, Threat Hunting actively searches systems, logs, and networks to identify hidden threats before they cause damage.

This room covers the foundational concepts of threat hunting, including hunting approaches, targets, techniques, and a practical investigation involving APT-Serpent.

**Key Focus:**
- Threat Hunting vs Incident Response
- Dwell Time
- Threat Hunting Approaches
- Threat Hunting Targets
- Hunting Techniques
- Practical APT-Serpent Investigation

---

## Walkthrough

### 1. What Is Threat Hunting?

Threat Hunting is a proactive security practice in which analysts search for suspicious activities without waiting for automated alerts.

Traditional security tools such as firewalls, IDS, and antivirus solutions detect many threats, but sophisticated attackers may bypass these defenses.

Threat hunters investigate available data sources to identify evidence of malicious activity.

### Threat Hunting vs Incident Response

| Aspect | Threat Hunting | Incident Response |
|---|---|---|
| Approach | Proactive | Reactive |
| Trigger | Self-initiated | Security alert |
| Objective | Discover hidden threats | Contain and remove threats |
| Timeline | Before compromise becomes obvious | After threat detection |
| Outcome | Improves detection capabilities | Remediation and recovery |

Both disciplines work together to strengthen an organization's security.

### Dwell Time

Dwell Time refers to the number of days an attacker remains undetected inside a network after the initial compromise.

Longer dwell time gives attackers more opportunities to:
- Perform reconnaissance
- Steal credentials
- Move laterally
- Access sensitive information
- Exfiltrate data

Threat Hunting helps reduce dwell time by discovering suspicious activity earlier.

---

### 2. Threat Hunting Approaches

Threat hunters use three foundational approaches depending on the available intelligence and investigation requirements.

#### 2.1 Hypothesis-Driven Hunting

Begins with a question or suspicion about potential malicious activity.

**Example:** A threat hunter suspects that finance department workstations may have been compromised through phishing.

The hunter investigates:
- Suspicious PowerShell execution
- Unusual network connections
- Unexpected process spawning
- Possible credential theft activity

**When to use:** When there are specific concerns about a user, system, department, or attack scenario.

#### 2.2 Intelligence-Driven Hunting

Uses external threat intelligence to investigate threats targeting a particular industry or organization.

Threat intelligence may include:
- Attack group information
- Known campaigns
- Malware behavior
- Indicators of Compromise
- MITRE ATT&CK techniques

**MITRE ATT&CK Examples:**

| Technique | ID | Description |
|---|---|---|
| Spearphishing Link | T1566.002 | Phishing through malicious links |
| PowerShell | T1059.001 | Command execution using PowerShell |
| Process Injection | T1055 | Injecting code into processes |
| Exfiltration Over C2 Channel | T1041 | Data theft through C2 communication |

Reference: [MITRE ATT&CK Framework](https://attack.mitre.org/)

**When to use:** When threat intelligence identifies attack groups or techniques relevant to your organization.

#### 2.3 Indicator-Driven Hunting

Focuses on specific Indicators of Compromise (IOCs) obtained from threat intelligence feeds, previous incidents, and malware analysis.

Common indicators include:
- File hashes
- IP addresses
- Domain names
- Email addresses
- Malicious URLs

**Example:** Searching an organization's SIEM logs for file hashes associated with a known malware campaign.

**When to use:** When specific indicators are available and need to be checked across the environment.

---

### 3. Threat Hunting Targets

Threat hunters investigate three major categories of evidence.

#### 3.1 Known Malware

Known malware refers to malicious software that has already been discovered, analyzed, and documented by security researchers.

Threat hunters search for:
- Known malicious file hashes
- Malware signatures
- Suspicious command-line patterns
- Known C2 connections
- Malware-related behaviors

**Example:** Emotet malware.

Useful reference: [VirusTotal](https://www.virustotal.com/)

**Limitation:** Malware variants can change their hashes, signatures, and infrastructure, making older indicators less effective.

#### 3.2 Attack Residues

Attack residues are artifacts left behind by attackers while executing their attack activities.

Examples:
- Unusual process execution
- Suspicious PowerShell commands
- Unexpected network connections
- Registry modifications
- Unusual file creation
- Privilege escalation attempts

**Example:** An attacker executing commands such as `whoami`, `net user`, and `ipconfig` to gather information about a compromised system.

Attack residues are particularly useful when investigating unknown malware because different attackers may leave similar behavioral artifacts.

#### 3.3 Known Vulnerabilities

Threat hunters investigate whether attackers have exploited known software vulnerabilities.

**Example: Log4Shell (CVE-2021-44228)**

Log4Shell was a critical vulnerability affecting the Apache Log4j logging library.

Threat hunters investigate:
- Suspicious application requests
- Unexpected Java child processes
- Unusual network connections
- Unexpected file system activity

Reference: [NVD – CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)

---

### 4. Threat Hunting Techniques

Threat hunters use different techniques and data sources to identify malicious activities.

#### Data Sources

| Data Source | Purpose |
|---|---|
| Windows Event Logs | Investigating system and security events |
| EDR | Endpoint activity and process information |
| SIEM | Centralized log collection and analysis |
| Network Traffic | Investigating network communications |
| DNS Logs | Identifying suspicious domain resolutions |
| Firewall Logs | Monitoring network connections |
| Application Logs | Investigating application activities |

#### 4.1 Attack Signatures

Attack signatures are recognizable patterns associated with malicious activity.

Examples:
- Known malware file hashes
- Suspicious command-line parameters
- Unusual parent-child process relationships
- Known malicious URL patterns
- Registry modifications associated with persistence

**Example:** Detecting `cmd.exe` spawned by an unexpected parent process such as `winword.exe`.

#### 4.2 Indicators of Compromise (IOCs)

IOCs are specific artifacts that indicate possible malicious activity.

Examples:
- File hashes
- IP addresses
- Domain names
- Email addresses
- URLs
- Registry keys

Threat hunters search these indicators across SIEM, EDR, firewall, and DNS logs.

#### 4.3 Behavioral Pattern Analysis

This technique identifies suspicious sequences of events that may indicate malicious activity.

**Example:**

`Word.exe → cmd.exe → powershell.exe → External IP`

Individual events may appear legitimate, but their sequence can indicate malicious behavior.

Other examples:
- Unusual file access patterns
- Lateral movement
- Privilege escalation
- Bulk data collection
- Suspicious process chains

#### 4.4 Logical Queries and Anomaly Detection

Uses conditional queries to identify activities that deviate from normal system behavior.

Examples:
- Administrator commands executed at unusual hours
- SYSTEM accounts running unexpected interactive commands
- Users accessing unfamiliar sensitive directories
- Outbound connections to unusual ports
- Suspicious password changes

Common query platforms include:

- [Splunk SPL](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual)
- [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [Azure KQL](https://learn.microsoft.com/en-us/kusto/query/)

---

### 5. Practical – APT-Serpent Investigation

**Scenario:**

A financial services organization receives threat intelligence about APT-Serpent, a sophisticated threat group targeting companies in its industry.

No security alerts have fired, but the organization wants to proactively investigate whether the threat actor has already gained access.

The objective is to select the appropriate threat hunting approach and target.

#### Investigation Answers

| Question | Answer |
|---|---|
| Known campaigns since 2021 | 4 |
| CustomBackdoor drop phase | Execution & Persistence |
| Primary initial access vector | Spear-phishing |
| HTTPS C2 beacon interval | 300 seconds |

#### Hunting Strategy

**Approach:** Intelligence-Driven Hunting

**Target:** Known Malware

The investigation uses external threat intelligence about APT-Serpent to guide the search for associated malicious activity.

**Flag:**

`THM-APT-SERPENT-INTEL`

---

## Key Takeaways

1. Threat Hunting is proactive, whereas Incident Response is reactive.
2. Dwell Time measures how long an attacker remains undetected.
3. Hypothesis-driven, intelligence-driven, and indicator-driven hunting are the three foundational approaches.
4. Known malware, attack residues, and known vulnerabilities are the three primary hunting targets.
5. Attack signatures, IOCs, behavioral pattern analysis, and anomaly detection are important hunting techniques.
6. MITRE ATT&CK helps security analysts understand and classify adversary behavior.
7. Combining multiple data sources improves threat visibility and investigation accuracy.

---

## Conclusion

This room provided an introduction to the foundational concepts of Threat Hunting.

It covered different hunting approaches, investigation targets, and techniques used by security analysts to identify suspicious activities that automated detection systems may miss.

The practical APT-Serpent scenario demonstrated how threat intelligence can guide a proactive investigation.

Understanding these fundamentals provides a foundation for more advanced threat hunting scenarios and SOC investigations.

---

## Completion

**Room:** [Threat Hunting: Introduction](https://tryhackme.com/room/threathuntingintroduction)

**Module:** [Threat Hunting](https://tryhackme.com/module/threat-hunting)

**Learning Path:** [SOC Level 2](https://tryhackme.com/path/outline/soclevel2)

**Status:** Completed

*Written for educational purposes as part of my SOC Level 2 learning path.*
