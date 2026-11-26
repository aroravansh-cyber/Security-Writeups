Bypass Disable Functions | TryHackMe Write-up

"Platform" (https://img.shields.io/badge/Platform-TryHackMe-212C42)
"Category" (https://img.shields.io/badge/Category-Web%20Exploitation-red)
"Difficulty" (https://img.shields.io/badge/Difficulty-Info-blue)
"Status" (https://img.shields.io/badge/Status-Completed-brightgreen)

Author: "Vansh Arora" (https://github.com/aroravansh-cyber)

Completion Date: 26 November 2026

Room Link: "TryHackMe – Bypass Disable Functions" (https://tryhackme.com/room/bypassdisablefunctions)

---

Overview

This room focuses on bypassing PHP security restrictions designed to prevent operating system command execution and process creation.

The challenge demonstrates how insecure file upload functionality, combined with incomplete PHP restrictions, can potentially lead to remote code execution.

The primary tool introduced in this room is Chankro, which demonstrates abusing PHP's "putenv()" and "mail()" functions alongside Linux's "LD_PRELOAD" mechanism.

Tools Used

- Nmap
- cURL
- Gobuster
- Chankro
- Netcat
- AttackBox
- Browser Developer Tools

---

1. Reconnaissance

Started by scanning the target machine to identify open ports and running services.

Command:

nmap -sV -sC 10.49.157.60

Results:

Port| Service| Version
22| SSH| OpenSSH 7.2p2
80| HTTP| Apache 2.4.18

The target was running Ubuntu Linux with an Apache web server.

The website title was identified as:

Ecorp - Jobs

2. Web Enumeration

Opened the web application:

http://10.49.157.60

The homepage contained a navigation option named Apply Job.

The application page was identified as:

http://10.49.157.60/cv.php

Retrieved the page source using cURL:

curl -s http://10.49.157.60/cv.php

The page displayed the following information:

Send your cv in image format

This indicated that the application provided a file upload feature.

3. Identifying the File Upload Functionality

The HTML source revealed the following form:

<form enctype="multipart/form-data" action="" method="POST">
    <input name="file" type="file" />
    <input type="submit" value="Subir archivo" />
</form>

Form Analysis:

Parameter| Value
Endpoint| "/cv.php"
HTTP Method| POST
Encoding| multipart/form-data
File Parameter| "file"
Expected Format| Image

The upload functionality became the primary attack surface for further investigation.

4. Understanding PHP Disabled Functions

PHP provides several functions that can execute operating system commands or launch external processes.

Examples include:

Function| Purpose
"system()"| Executes system commands
"shell_exec()"| Executes shell commands
"exec()"| Executes external programs
"passthru()"| Executes an external command
"putenv()"| Modifies environment variables
"mail()"| Sends email
"dl()"| Dynamically loads PHP extensions

Administrators commonly disable potentially dangerous functions using the "disable_functions" directive in "php.ini".

However, disabling selected functions does not necessarily eliminate every possible execution mechanism.

5. Chankro – PHP Restrictions Bypass

Chankro is a tool developed by Tarlogic Security.

It demonstrates a technique involving PHP functions and Linux dynamic library loading.

The technique uses:

- "putenv()" to modify environment variables.
- "LD_PRELOAD" to influence shared library loading.
- Shared object libraries (".so") to intercept library functions.
- "mail()" as part of the execution mechanism.

Important Concepts

LD_PRELOAD

A Linux environment variable that allows shared libraries to be loaded before other shared libraries.

Function Hooking

A technique that intercepts an existing function and changes its behaviour.

Dropper

A script that creates or deploys additional files required for another operation.

Chankro Installation

The room introduces the following installation process:

git clone https://github.com/TarlogicSecurity/Chankro.git

cd Chankro

python2 chankro.py --help

Chankro accepts several parameters:

Parameter| Description
"--arch"| Target architecture (32 or 64 bit)
"--input"| Input file
"--output"| Generated PHP file
"--path"| Absolute path of the PHP file on the target

Example syntax from the room:

python chankro.py --arch 64 --input c.sh --output tryhackme.php --path /var/www/html

The generated PHP file acts as a dropper that prepares the required components on the server.

6. Upload Validation Analysis

The application expected an image file.

An important concept demonstrated by this challenge is the difference between checking a file's extension and validating its actual contents.

A weak upload validation mechanism may rely on file signatures without properly checking the complete file structure or preventing server-side execution.

A GIF signature commonly begins with:

GIF89a;

This illustrates why file signature checks alone are insufficient to secure upload functionality.

7. Understanding the Attack Flow

The overall technique can be represented as follows:

File Upload Functionality
          |
          v
Upload Validation Analysis
          |
          v
PHP Restrictions Identified
          |
          v
Chankro Dropper
          |
          v
LD_PRELOAD Mechanism
          |
          v
Function Hooking
          |
          v
PHP Restriction Bypass
          |
          v
Server Compromise
          |
          v
Flag Retrieval

8. Flag

Objective: Compromise the target machine and locate "flag.txt".

Flag:

thm{bypass_d1sable_functions_1n_php}

<details>
<summary>Completion Details</summary>- Platform: TryHackMe
- Room: Bypass Disable Functions
- Target Service: Apache HTTP
- Application: PHP
- Main Technique: Chankro
- Result: Challenge completed

</details>---

9. Key Takeaways

- File upload vulnerabilities can expose web applications to server-side execution risks.
- File extensions and MIME types should not be trusted independently.
- PHP "disable_functions" is not a complete security boundary.
- "LD_PRELOAD" influences Linux dynamic library loading.
- Function hooking can alter the behaviour of existing library functions.
- Chankro demonstrates the risks of relying exclusively on PHP function restrictions.
- Upload directories should not permit execution of uploaded content.

10. Security Recommendations

Secure File Uploads

- Validate file extensions, MIME types and file signatures.
- Re-encode uploaded images.
- Store uploaded files outside the web root.

PHP Hardening

- Minimize unnecessary PHP functionality.
- Avoid relying exclusively on "disable_functions".
- Restrict application permissions and process capabilities.

Server Hardening

- Apply appropriate filesystem permissions.
- Use AppArmor or SELinux where applicable.
- Restrict access to sensitive configuration information.
- Monitor unexpected process execution.

References

- "TryHackMe – Bypass Disable Functions" (https://tryhackme.com/room/bypassdisablefunctions)
- "Chankro – Tarlogic Security" (https://github.com/TarlogicSecurity/Chankro)
- "PHP Documentation" (https://www.php.net/manual/en/)
- "PHP putenv()" (https://www.php.net/manual/en/function.putenv.php)
- "PHP mail()" (https://www.php.net/manual/en/function.mail.php)
- "Linux Dynamic Linker Documentation" (https://man7.org/linux/man-pages/man8/ld.so.8.html)

---

Status: Completed

Category: Web Security | PHP Security | File Upload Vulnerabilities