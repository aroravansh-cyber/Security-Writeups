# NahamStore | TryHackMe Write-up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Web%20Security-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Status](https://img.shields.io/badge/Status-Completed-success)

**Author:** [Vansh Arora](https://github.com/aroravansh-cyber)  
**Date:** 28 November 2026  
**Room:** [NahamStore](https://tryhackme.com/room/nahamstore)

## Overview

NahamStore is a web application security lab covering common vulnerabilities such as XSS, IDOR, CSRF, SSRF, XXE, LFI, RCE and SQL Injection.

## Task 3 – Recon

**Enumeration**

```bash
nmap -sC -sV -p- nahamstore.thm
```

| Question | Answer |
|---|---|
| Jimmy Jones SSN | `521-61-6392` |

## Task 4 – XSS

**Enumeration**

```bash
gobuster dir -u http://nahamstore.thm -w /usr/share/wordlists/dirb/common.txt
```

| Question | Answer |
|---|---|
| Vulnerable endpoint | `http://marketing.nahamstore.thm/?error=` |
| Stored XSS header | `User-Agent` |
| Product page HTML tag | `title` |
| JavaScript variable | `search` |
| Hidden parameter | `q` |
| Returns page HTML tag | `textarea` |
| H1 value | `Page Not Found` |
| Other hidden parameter | `discount` |

## Task 5 – Open Redirect

**Testing**

```bash
curl -I "http://nahamstore.thm/"
```

| Question | Answer |
|---|---|
| Open Redirect One | `r` |
| Open Redirect Two | `redirect_url` |

## Task 6 – CSRF

**Inspect request**

```bash
curl -i http://nahamstore.thm/account/settings/password
```

| Question | Answer |
|---|---|
| Vulnerable URL | `http://nahamstore.thm/account/settings/password` |
| Field to remove | `csrf_protect` |
| Encoding used | `base64` |

## Task 7 – IDOR

**Inspect HTTP requests**

```bash
curl -i http://nahamstore.thm/
```

| Question | Answer |
|---|---|
| First line of address | `160 Broadway` |
| Order ID 3 date and time | `22/02/2021 11:42:13` |

## Task 8 – LFI

**Testing**

```bash
curl -i "http://nahamstore.thm/"
```

| Question | Answer |
|---|---|
| LFI Flag | `{7ef60e74b711f4c3a1fdf5a131ebf863}` |

## Task 9 – SSRF

**Inspect server-side requests and responses.**

| Question | Answer |
|---|---|
| Jimmy Jones Credit Card | `5190216301622131` |

## Task 10 – XXE

**XML inspection**

```bash
curl -X POST http://nahamstore.thm/ \
-H "Content-Type: application/xml" \
-d '<root>test</root>'
```

| Question | Answer |
|---|---|
| XXE Flag | `{9f18bd8b9acaada53c4c643744401ea8}` |
| Blind XXE Flag | `{d6b22cb3e37bef32d800105b11107d8f}` |

## Task 11 – RCE

**Inspect application responses and input handling.**

| Question | Answer |
|---|---|
| First RCE Flag | `{b42d2f1ff39874d56132537be62cf9e3}` |
| Second RCE Flag | `{93125e2a845a38c3e1531f72c250e676}` |

## Task 12 – SQL Injection

**Inspect HTTP requests and database responses.**

| Question | Answer |
|---|---|
| Flag 1 | `{d890234e20be48ff96a2f9caab0de55c}` |
| Flag 2 (Blind) | `{212ec3b036925a38b7167cf9f0243015}` |

## Key Learnings

- Web Enumeration
- XSS and Open Redirect
- CSRF and IDOR
- LFI and SSRF
- XXE and RCE
- SQL Injection

**Status:** Completed
