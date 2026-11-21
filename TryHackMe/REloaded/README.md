# REloaded | TryHackMe Write-up

![Platform](https://img.shields.io/badge/Platform-TryHackMe-212C42)
![Path](https://img.shields.io/badge/Path-Reverse%20Engineering-blue)
![Levels](https://img.shields.io/badge/Levels-5-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

**Author:** Vansh ([@aroravansh-cyber](https://github.com/aroravansh-cyber))  
**Completed:** 2026-11-21  
**Room:** [REloaded](https://tryhackme.com/room/reloaded?utm_campaign=social_share&utm_medium=social&utm_content=room&utm_source=copy&share_date=2026-11-21&sharerId=69c7be8df8f1cb2be36ec757)  
**Tools:** Kali Linux, Ghidra, x64dbg, Immunity Debugger, Binary Ninja

---

## Overview

REloaded is a beginner-friendly Reverse Engineering room on TryHackMe. It introduces different reverse engineering concepts through five executable-based challenges.

The challenges cover basic file enumeration, string extraction, hexadecimal conversion, authentication analysis, conditional jump instructions, XOR transformations, debugging and encryption analysis.

**Objectives:**
- Analyse Windows executable files.
- Extract useful information from binaries.
- Understand program logic using Ghidra.
- Analyse authentication mechanisms.
- Inspect memory and execution using a debugger.
- Understand basic encryption and decryption concepts.

---

## Walkthrough

### Level 0 – Basic Enumeration

**Objective:** Find the flag hidden inside the executable.

**Step 1. Identify the executable.**

```bash
file Level-0.exe
```

**Step 2. Extract readable strings.**

```bash
strings Level-0.exe
```

**Step 3. Filter potentially interesting strings.**

```bash
strings Level-0.exe | grep -E '.{10,100}'
```

**Step 4. Identify the flag.**

Inspect the extracted strings and locate the challenge flag.

**Answer:**

```text
L3v3lZ340_is_D02e
```

**Key Learning:**
- File identification
- String extraction
- Basic binary enumeration

---

### Level 1 – Find the Lucky Number

**Objective:** Recover a number hidden inside the executable.

**Step 1. Run strings.**

```bash
strings Level-1.exe
```

**Step 2. Open the executable in Ghidra.**

Import `Level-1.exe` and allow Ghidra to analyse the binary.

**Step 3. Search for the reference string.**

```text
Thats your lucky number !!!
```

**Step 4. Find its cross-reference (XREF).**

Inspect the function associated with the reference string.

**Step 5. Identify the hexadecimal comparison.**

```text
0x6ad
```

**Step 6. Convert hexadecimal to decimal.**

```python
print(int("6ad", 16))
```

**Answer:**

```text
1709
```

**Key Learning:**
- Cross-reference analysis
- Decompiled function inspection
- Hexadecimal-to-decimal conversion

---

### Level 2 – Authentication Bypass

**Objective:** Analyse the authentication mechanism and identify the conditional jump instruction.

**Step 1. Open the executable in Ghidra.**

Import `Level-2.exe`.

**Step 2. Search for the reference string.**

```text
Get Ready For L4 ;)
```

**Step 3. Follow the XREF.**

Locate the authentication function and inspect the comparison between the supplied input and the stored value.

**Step 4. Identify the flag.**

```text
L3_1s_20t_Th3_L131t
```

**Step 5. Inspect the assembly listing.**

Locate the conditional jump controlling the authentication result.

The original instruction is:

```asm
JNZ
```

The challenge walkthrough identifies the patched instruction as:

```asm
JZ
```

**Answers:**

| Question | Answer |
|---|---|
| Flag | `L3_1s_20t_Th3_L131t` |
| Which instruction did you modify? | `JNZ` |

**Key Learning:**
- Authentication logic analysis
- Conditional jump instructions
- Assembly inspection
- Binary patching concepts

---

### Level 3 – Recover the Master Password

**Objective:** Understand the XOR transformation and recover the master password.

**Step 1. Open the executable in Ghidra.**

Import `Level-3.exe`.

**Step 2. Search for the reference string.**

```text
Rooted !!!
```

**Step 3. Follow the XREF.**

Locate the validation function and inspect the password-generation logic.

**Step 4. Identify the stored string.**

```text
THMctf-L4
```

The transformation uses XOR with:

```text
0x07
```

**Step 5. Locate the XOR function.**

```text
00401410
```

**Step 6. Open the executable in a debugger.**

Use x64dbg or Immunity Debugger to inspect the relevant function.

**Step 7. Set a breakpoint.**

```text
00401410
```

**Step 8. Inspect the execution.**

Run the application and inspect the relevant registers and memory values while analysing the XOR transformation.

The XOR-transformed bytes are intermediate data, so they should not be confused with the recovered answer.

**Answer:**

```text
THMctf-L4
```

**Key Learning:**
- XOR transformations
- Password-generation logic
- Memory inspection
- Dynamic analysis

---

### Level 4 – Encryption and Decryption

**Objective:** Analyse the encryption routine and recover the original plaintext.

**Step 1. Open the executable in Ghidra.**

Import `Level-4.exe`.

**Step 2. Search for the printf function.**

Locate its cross-references to identify where the transformed string is printed.

**Step 3. Identify the relevant function.**

```text
FUN_00401453
```

**Step 4. Analyse the function.**

The function contains:

- String length calculation
- A preliminary condition check
- An XOR-based transformation
- Output of the transformed string

**Step 5. Locate the function address.**

```text
00401453
```

**Step 6. Inspect the executable using a debugger.**

Set a breakpoint and analyse the input buffer and relevant memory values during execution.

**Step 7. Recover the original plaintext.**

The recovered answer is:

```text
Alan Turing Was a Geniuse
```

**Key Learning:**
- Encryption routine analysis
- XOR-based transformations
- Function cross-reference analysis
- Debugger-assisted inspection

---

## Final Answer Sheet

| Level | Question | Answer |
|---|---|---|
| Level 0 | Flag | `L3v3lZ340_is_D02e` |
| Level 1 | Flag | `1709` |
| Level 2 | Flag | `L3_1s_20t_Th3_L131t` |
| Level 2 | Modified instruction | `JNZ` |
| Level 3 | Flag | `THMctf-L4` |
| Level 4 | Flag | `Alan Turing Was a Geniuse` |

---

## Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux | Linux analysis environment |
| Ghidra | Static analysis and decompilation |
| x64dbg | Dynamic debugging |
| Immunity Debugger | Runtime inspection |
| Binary Ninja | Binary analysis |

---

## Key Takeaways

1. The `strings` utility helps extract readable information from executable files.
2. Ghidra makes it easier to understand functions, references and program logic.
3. Conditional jump instructions are important when analysing authentication mechanisms.
4. XOR transformations can be investigated through static and dynamic analysis.
5. Debuggers help inspect registers, memory and program execution.
6. Combining static and dynamic analysis provides a better understanding of binary behaviour.

---

## References

- [TryHackMe – REloaded Official Room](https://tryhackme.com/room/reloaded?utm_campaign=social_share&utm_medium=social&utm_content=room&utm_source=copy&share_date=2026-11-21&sharerId=69c7be8df8f1cb2be36ec757)
- [REloaded – InfoPhreak Walkthrough](https://infophreak.com/tryhackme-reloaded-writeup/)
- [REloaded – Additional Analysis](https://sovayle.github.io/ctfs/tryhackme)

---

*Written for educational purposes as part of my Reverse Engineering and Cybersecurity learning journey.*
