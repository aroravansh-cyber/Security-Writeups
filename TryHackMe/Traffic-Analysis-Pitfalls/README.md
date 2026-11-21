# Traffic Analysis Pitfalls | TryHackMe Write-up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-212C42)
![Path](https://img.shields.io/badge/Path-SOC%20Level%202-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

**Author:** Vansh ([@aroravansh-cyber](https://github.com/aroravansh-cyber))
**Completed:** 2026-11-21
**Room:** [Traffic Analysis Pitfalls](https://tryhackme.com/room/trafficanalysispitfalls)
**Tools:** Splunk, FortiGate, Zeek, Sysmon, NetFlow

---

## Overview

A SIEM alert fired at 02:47 UTC: **847 MB** sent from `WKST-FINANCE-04` (10.10.15.44) to `172.67.153.42:443`. Every normal pivot came back empty:

| Pivot | Result |
|---|---|
| IP reputation | Clean |
| DNS lookup | No record |
| TLS SNI | Encrypted |
| Proxy log | None |

The destination is a Cloudflare IP. CDNs, QUIC, DoH and ECH have removed the IP, DNS and SNI signals that older detection relied on. This room shows what to look at instead.

**Verdict:** confirmed data exfiltration over QUIC through Cloudflare, done by `EXCEL.EXE` after about 16 days of beaconing.

**Lab:** Splunk with `index=lab` and time range **All Time**. Data sources: FortiGate, Zeek, Sysmon (EventID 3), NetFlow, plus an `asn_lookup` table.

---

## Walkthrough

### 1. CDN visibility: IP reputation is useless here

Cloudflare (AS13335) serves millions of domains from shared IPs, so no feed will flag it and it cannot be blocked. I enriched every destination with ASN data to see what *kind* of organization runs it.

```spl
index=lab sourcetype=fortigate_traffic
| lookup asn_lookup network AS dstip OUTPUT dst_asn_org dst_category dst_risk
| stats count by dst_category dst_asn_org
| sort -count
```

**Why:** CDN and cloud traffic is the expected bulk. Bulletproof Hosting and Tor exit destinations are the outliers that IP reputation missed.

Next, I checked the alert connection for beaconing:

```spl
index=lab sourcetype=fortigate_traffic srcip=10.10.15.44 dstip=172.67.153.42
| eval hour=strftime(_time,"%Y-%m-%d %H")
| stats count as per_hour min(sentbyte) as min_b max(sentbyte) as max_b by hour
| sort -per_hour | head 15
```

**Finding:** about 17 connections every hour for weeks, with a keepalive of about 800 bytes and occasional large bursts. That means fixed-interval beaconing, asymmetric volume and off-hours activity. No browser behaves like this.

---

### 2. QUIC: watching the right port on the wrong transport

HTTP/3 runs over QUIC on **UDP/443**. A rule like `alert TCP any any -> any 443` never fires on it, and proxies that inspect TLS only handle TCP.

```spl
index=lab sourcetype=zeek_conn "id.resp_p"=443
| eval transport=if(proto="udp","QUIC (UDP/443)","HTTPS (TCP/443)")
| stats count by transport
```

**Finding:** about 40% of port 443 traffic (6,946 sessions) is QUIC, so a TCP-only rule misses roughly two-fifths of encrypted outbound traffic.

```spl
index=lab sourcetype=zeek_conn service=quic
| where duration > 3600
| table _time "id.orig_h" "id.resp_h" duration orig_bytes resp_bytes
```

**Finding:** one session lasted 15,180 seconds (4h 12m) with about 847 MB out and 2 MB in. This is the original alert, and it explains why there was no proxy log.

**Fix:** use an NGFW that inspects QUIC, or block outbound UDP/443 so browsers fall back to HTTP/2 over TCP.

---

### 3. DoH: the silent host pattern

DNS over HTTPS hides queries inside HTTPS. I can no longer see what a host resolves, but I can see that its plaintext DNS **stopped**.

```spl
index=lab sourcetype=zeek_conn "id.resp_p"=53 proto=udp
| timechart span=1d count by "id.orig_h"
```

**Finding:** `WKST-MKTG-07` (10.10.12.23) drops to zero UDP/53 queries from **2026-03-27**.

```spl
index=lab sourcetype=zeek_conn "id.orig_h"="10.10.12.23" "id.resp_p"=443
| search "id.resp_h"="1.1.1.1" OR "id.resp_h"="8.8.8.8"
| timechart span=1d count by "id.resp_h"
```

**Why:** connections to `1.1.1.1:443` begin the same day UDP/53 stops, so the two signals confirm DoH. For ECH (hidden SNI), the workarounds are JA4 fingerprints, Certificate Transparency logs and Sysmon EventID 22.

---

### 4. Choosing the right log source

| Source | Answers | Limit |
|---|---|---|
| NetFlow | Who talked to whom, how much | Metadata only |
| FortiGate | Allowed or blocked, bytes, protocol | No process info |
| Zeek | Application protocol (`service=quic`) | No process info |
| **Sysmon EID 3** | **Which process made the connection** | Needs an agent on each host |
| PCAP | Exact bytes | Too costly to retain |

FortiGate, NetFlow and Zeek all agreed on the flow, but none could name the process. Only Sysmon could:

```spl
index=lab sourcetype=sysmon DestinationIp=172.67.153.42
| stats count by Image
| sort -count
```

**Finding:** `EXCEL.EXE` has thousands of connections, while browsers have tens. That gap is the signal.

---

### 5. ASN enrichment finds what reputation misses

Enrichment runs **at query time**, so today's intelligence applies to old logs without re-ingesting.

```spl
index=lab sourcetype=fortigate_traffic
| lookup asn_lookup network AS dstip OUTPUT dst_asn dst_asn_org dst_category dst_risk
| where dst_risk="high" OR dst_risk="critical"
| table _time srcip srcname dstip dstport proto sentbyte rcvdbyte dst_asn_org dst_category dst_risk
| sort _time
```

This cut about 17,000 rows to two incidents:

| Host | Destination | Observation |
|---|---|---|
| **WKST-HR-02** (10.10.11.18) | Tor relay `185.220.101.34` (AS205100) | 6 connections at regular 2-minute intervals, which points to automation |
| **WKST-SALES-11** (10.10.13.67) | Bulletproof hosting `185.213.154.88` (AS44477) | 4 TCP/443 connections in one day |

**Why it matters:** both IPs had clean reputation. Without ASN enrichment these would have closed as false positives.

---

### 6. Closing the case

| Step | Source | Result |
|---|---|---|
| IP and ASN | FortiGate + lookup | Cloudflare CDN, clean. IP triage is done. |
| History | FortiGate | Small beacons from Mar 17, then a spike on Apr 2 |
| Exfil event | FortiGate | 2026-04-02 02:47 UTC, QUIC, about 847 MB out and 2 MB in |
| Process | Sysmon | `EXCEL.EXE` (PID 4812), 6,108 connections |
| Transport | Zeek | QUIC confirmed, proxy bypass confirmed |
| DNS | Zeek DNS | Host uses DNS normally but **never resolved the C2 IP**, so Excel used a hardcoded IP |
| Duration | Sysmon | 2026-03-17 07:59:38 to 2026-04-06 21:55:44 |

```spl
index=lab sourcetype=sysmon Image="*EXCEL.EXE" ProcessId=4812 SourceIp=10.10.15.44
| stats min(_time) as first_seen max(_time) as last_seen count
| eval first_seen=strftime(first_seen,"%Y-%m-%d %H:%M:%S")
| eval last_seen=strftime(last_seen,"%Y-%m-%d %H:%M:%S")
```

**Why this is the key finding:** Excel is not a QUIC client, and a legitimate session does not beacon like a machine clock for weeks. Only endpoint telemetry showed what was responsible.

---

## Incident Timeline

| Date (2026) | Event |
|---|---|
| Mar 17 | `EXCEL.EXE` (PID 4812) starts beaconing about every 4 minutes |
| Mar 22 | WKST-HR-02 connects to a Tor exit relay |
| Mar 25 | WKST-SALES-11 connects to bulletproof hosting |
| Mar 27 | WKST-MKTG-07 switches to DoH |
| Apr 2, 02:47 UTC | About 847 MB exfiltrated over a 4h 12m QUIC session |
| Apr 6 | Last recorded activity from PID 4812 |

---

## Indicators of Compromise

| Type | Value |
|---|---|
| Compromised host | `10.10.15.44` (WKST-FINANCE-04) |
| C2 IP (shared CDN, do not block by IP) | `172.67.153.42` (AS13335, Cloudflare) |
| Process | `C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE` (PID 4812) |
| Beacon interval | 240 seconds |
| Transport | UDP/443 (QUIC) |
| Tor relay contacted | `185.220.101.34` (AS205100), by WKST-HR-02 |
| Bulletproof host contacted | `185.213.154.88` (AS44477), by WKST-SALES-11 |

---

## MITRE ATT&CK Mapping

Only T1090.004 is named in the room. The rest is my own mapping of the observed behavior.

| Technique | ID | Where |
|---|---|---|
| Domain Fronting | [T1090.004](https://attack.mitre.org/techniques/T1090/004/) | CDN abuse discussed in the room |
| Multi-hop Proxy | [T1090.003](https://attack.mitre.org/techniques/T1090/003/) | Tor use by WKST-HR-02 |
| Web Protocols | [T1071.001](https://attack.mitre.org/techniques/T1071/001/) | C2 over HTTPS/QUIC |
| Encrypted Channel | [T1573](https://attack.mitre.org/techniques/T1573/) | Encrypted C2 traffic |
| Exfiltration Over C2 Channel | [T1041](https://attack.mitre.org/techniques/T1041/) | 847 MB over the beacon channel |

---

## Detection and Mitigation

- Add **ASN enrichment** and alert on Bulletproof Hosting and Tor categories.
- Log all **UDP/443**, and inspect QUIC on the NGFW or block it.
- Alert on **fixed-interval beaconing** and long sessions with a high outbound-to-inbound ratio.
- Hunt for hosts whose **UDP/53 drops to zero** and watch TCP/443 to known DoH resolvers.
- Deploy **Sysmon or EDR** so every connection can be tied to a process.
- Review allowlists for `*.trycloudflare.com` and `*.workers.dev`.

---

## Key Takeaways

1. A clean reputation on a CDN IP means nothing, so look at behavior.
2. TCP/443 rules do not cover QUIC.
3. A host that suddenly stops sending DNS is a signal in itself.
4. Network logs show a connection exists. Only the endpoint shows which process made it.
5. Correlate at least two independent sources before closing an alert.

---

## Completion

Room completed: https://tryhackme.com/room/trafficanalysispitfalls

<!-- ![Room completed](screenshots/completion.png) -->

*Written for educational purposes as part of my SOC Level 2 learning path.*
