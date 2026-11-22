# Assistant — HTB Sherlock Writeup

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![Category](https://img.shields.io/badge/Category-Digital%20Forensics-blue)
![OS](https://img.shields.io/badge/OS-Windows-blue?logo=windows)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Challenge Information

| Field | Details |
|---|---|
| Challenge | Assistant |
| Platform | Hack The Box |
| Type | Sherlock |
| Category | Digital Forensics |
| OS | Windows |
| Difficulty | Sherlock |
| Artifacts | PCA Logs |
| Status | Completed |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/sherlock/4058304/1954)

## 1. Introduction

The Assistant Sherlock challenge involved investigating Windows forensic artifacts to reconstruct suspicious activity on a compromised system.

The investigation focused on Program Compatibility Assistant (PCA) artifacts, specifically:

- `PcaAppLaunchDic.txt`
- `PcaGeneralDb0.txt`
- `PcaGeneralDb1.txt`

These files helped identify executed applications, their timestamps, application metadata, and process-related events.

## 2. Extracting the Evidence

The provided archive was password-protected. After extracting it, I located the PCA artifacts under:

```text
C:\Windows\appcompat\pca\
```

I converted the UTF-16LE database into readable UTF-8 text for analysis:

```bash
iconv -f UTF-16LE -t UTF-8 PcaGeneralDb0.txt
```

I then searched the records for application names, execution timestamps, and suspicious activity.

## 3. Identifying the Downloaded Archive Utility

The launch dictionary contained this entry:

```text
C:\Users\cyberjunkie\Downloads\7z2600-x64.exe
```

**Timestamp:**

```text
2026-04-08 11:30:37.758
```

This identified the downloaded 7-Zip installer.

The PCA database also contained installer-related events, including an initial failure and a later PCA resolution event.

## 4. Identifying the Exfiltration Tool

The launch dictionary revealed another executable:

```text
C:\Users\cyberjunkie\Downloads\rclone-v1.73.3-windows-amd64\rclone.exe
```

The PCA database identified the application as:

| Field | Details |
|---|---|
| Application | rclone |
| Version | 1.73.3 |
| Recorded Launch | 2026-04-08 11:32:00.303 |

Rclone is a command-line utility commonly used to manage and transfer files between local storage and cloud storage services.

Its presence in this investigation was particularly relevant because of its potential use in data transfer.

## 5. Investigating Process Activity

The PCA database recorded the following event for rclone:

```text
2026-04-08 11:32:08.187
Abnormal process exit with code 0x1
```

This indicated an abnormal process exit.

It was important to distinguish this from proof of a successful transfer or deliberate process termination.

## 6. Reconstructing the Timeline

The launch dictionary provided several important timestamps:

| Timestamp | Application |
|---|---|
| 11:30:37.758 | 7-Zip installer |
| 11:32:00.303 | rclone |
| 11:37:07.600 | 7-Zip File Manager |
| 11:40:16.828 | Windows Terminal |
| 11:42:11.726 | Eraser installer |
| 11:43:38.671 | Eraser |
| 12:00:25.475 | KAPE GUI |

The PCA database also recorded a 7-Zip File Manager resolution event at:

```text
2026-04-08 11:40:03.043
```

These records helped establish the sequence of suspicious application activity.

**Forensic distinction:** PCA launch timestamps and resolution events do not independently prove when data collection started, when it finished, or when data was successfully exfiltrated. Those conclusions require correlation with additional artifacts.

## 7. Identifying the Data Erasure Utility

The investigation revealed the Eraser application.

The relevant executable path was:

```text
C:\Program Files\Eraser\Eraser.exe
```

Eraser is a data-erasure utility. Its presence was relevant to the investigation because such software can be used to remove data from a system.

However, application execution alone does not establish that evidence was actually erased.

## 8. Forensic Takeaways

This Sherlock investigation reinforced several important Windows forensics concepts:

1. PCA artifacts can help reconstruct application execution timelines.
2. Application launch records and process events provide different types of evidence.
3. Rclone can be relevant when investigating potential data transfers.
4. KAPE is a forensic collection utility.
5. Eraser is a data-erasure utility.
6. Correlating multiple artifacts is essential before drawing conclusions about attacker activity.

## 9. Conclusion

The Assistant Sherlock challenge provided practical experience in analyzing Windows PCA artifacts and building an application execution timeline.

The investigation demonstrated how seemingly ordinary utilities can become important forensic indicators when examined in the context of suspicious system activity.

**Challenge completed successfully — HTB Sherlock: Assistant.**
