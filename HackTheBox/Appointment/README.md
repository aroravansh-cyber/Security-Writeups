# Appointment — HTB Machine Writeup

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![Category](https://img.shields.io/badge/Category-Web%20Security-blue)
![OS](https://img.shields.io/badge/OS-Linux-blue?logo=linux)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Appointment |
| Platform | Hack The Box |
| Type | Starting Point |
| Category | Web Security |
| OS | Linux |
| Difficulty | Very Easy |
| Vulnerability | SQL Injection |
| Service | Apache HTTP Server |
| Port | 80/tcp |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/402)

## 1. Introduction

The Appointment machine focused on identifying and exploiting a SQL Injection vulnerability in a web application's authentication mechanism.

The investigation involved network reconnaissance, web service identification, understanding SQL queries, and analyzing how improperly handled user input could affect authentication.

The primary objectives were:

- Identify open ports and running services.
- Enumerate the web application.
- Understand SQL Injection vulnerabilities.
- Identify the authentication bypass condition.
- Retrieve the root flag.

## 2. Network Reconnaissance

I started by scanning the target machine using Nmap to identify open ports and running services.

```bash
nmap -sC -sV TARGET_IP
```

The scan identified the following service:

| Port | State | Service | Version |
|---|---|---|---|
| 80/tcp | Open | HTTP | Apache httpd 2.4.38 ((Debian)) |

Port 80 is the default port used by HTTP.

The discovery of Apache HTTP Server established the web application as the primary attack surface.

## 3. Web Application Enumeration

After identifying the HTTP service, I accessed the target through a web browser.

```text
http://TARGET_IP
```

The website presented a login page containing username and password fields.

This indicated that the application used an authentication mechanism to control access to its protected content.

I identified the login functionality as an important area for further security assessment.

## 4. Understanding SQL Injection

SQL Injection is a web application vulnerability that occurs when user-controlled input is incorporated into SQL queries without proper handling.

It can allow an attacker to interfere with database queries and potentially:

- Bypass authentication.
- Access unauthorized information.
- Modify database records.
- Compromise application data.

The vulnerability demonstrated in Appointment falls under:

**OWASP Top 10 (2021): A03 — Injection**

## 5. Identifying the Authentication Vulnerability

The login functionality was vulnerable because user input was incorporated into the SQL authentication query without adequate protection.

A simplified example of a vulnerable query is:

```sql
SELECT * FROM users
WHERE username = 'INPUT'
AND password = 'INPUT';
```

The application failed to properly separate user input from SQL query logic.

This created an opportunity to manipulate the query structure through the login form.

## 6. Understanding MySQL Comments

MySQL supports the `#` character as a single-line comment marker.

Everything following this character on the same line is treated as a comment.

In the context of this challenge, the comment character could interfere with the application's password validation condition.

This demonstrated why untrusted input must never be directly concatenated into SQL statements.

## 7. Authentication Bypass

During the assessment, I identified that the login functionality could be bypassed using a comment-based SQL Injection technique.

The vulnerable application failed to properly handle special characters in the username field.

By exploiting this behavior, authentication could be completed as the administrator without knowing the actual password.

The application subsequently displayed a successful login page.

The first word displayed on the returned webpage was:

```text
Congratulations
```

This confirmed that the authentication bypass had succeeded.

## 8. Flag Retrieval

After successfully bypassing the login mechanism, the application displayed the root flag on the webpage.

The flag was retrieved directly from the authenticated page.

This confirmed successful completion of the Appointment machine.

## 9. Security Findings

The primary vulnerability identified during this assessment was SQL Injection in the application's login functionality.

**Potential Security Impacts:**

- Authentication bypass.
- Unauthorized access to administrator accounts.
- Exposure of sensitive database information.
- Compromise of application data.
- Violation of confidentiality and integrity.

## 10. Security Recommendations

To prevent SQL Injection vulnerabilities, the following security measures are recommended:

1. Use parameterized queries and prepared statements.
2. Avoid directly concatenating user input into SQL queries.
3. Implement server-side input validation.
4. Apply least-privilege permissions to database accounts.
5. Handle authentication securely.
6. Avoid exposing detailed database errors to users.
7. Perform regular security assessments of web applications.

## 11. Key Takeaways

This machine reinforced several important cybersecurity concepts:

1. Network reconnaissance using Nmap.
2. Identifying exposed HTTP services.
3. Web application enumeration.
4. Understanding SQL Injection vulnerabilities.
5. Understanding MySQL comment syntax.
6. Identifying authentication bypass conditions.
7. Understanding OWASP A03:2021 — Injection.
8. Learning the importance of prepared statements and secure database interaction.

## 12. Conclusion

The Appointment machine provided practical experience in identifying and understanding SQL Injection vulnerabilities in web applications.

The assessment demonstrated how improperly handled user input can alter SQL query behavior and bypass authentication mechanisms.

By identifying the exposed web service and analyzing the vulnerable login functionality, I successfully bypassed authentication and retrieved the root flag.

The challenge highlighted the importance of secure coding practices, parameterized queries, and proper input handling in web application security.

**Machine completed successfully — HTB Starting Point: Appointment.**
