
---

# **CTF Walkthrough: Reminiscent**  

### **Category**: Forensics  

---

## **Description**  
Suspicious traffic was detected from a recruiter’s virtual PC. A memory dump of the offending VM was captured before it was removed from the network for imaging and analysis. The recruiter mentioned receiving an email regarding their resume, which contained a suspicious link. A copy of the email was recovered and provided for reference.  

The goal: **Find and decode the source of the malware to retrieve the flag.**  

---

## **Detailed Walkthrough**  

### **Step 1: Analyzing the Email**  
The first step was to analyze the phishing email provided as part of the investigation. The email was opened in a sandboxed environment using [Any.Run](https://app.any.run).  

#### **Findings**:  
- The email included a link to `http://10.10.99.55:8080/resume.zip`.  
- Clicking the link triggered a download of `resume.zip`, hosted on a web server at the private IP address `10.10.99.55:8080`.  
- This indicated that a file delivery mechanism was part of the attack.  

The recruiter’s interaction with the link might have resulted in the execution of malicious content, likely related to the detected traffic anomalies.  

---

### **Step 2: Memory Analysis with Volatility**  

#### **Load the Memory Dump**  
The memory dump (`flounder-pc-memdump.elf`) was provided for analysis. Using Volatility, I first identified the appropriate profile to analyze the memory dump:  

**Command**:  
```bash
volatility -f flounder-pc-memdump.elf imageinfo
```

**Output**:  
- The suggested profile was **Win7SP1x64**, which was used for all subsequent analysis.

---

#### **List Running Processes**  
To identify active processes at the time of the dump, I listed all running processes:  

**Command**:  
```bash
volatility -f flounder-pc-memdump.elf --profile=Win7SP1x64 pslist
```

**Findings**:  
- Most of the processes appeared normal.
- A **PowerShell process** (`powershell.exe`) caught my attention, as it is often used in malicious scripts or payload execution.

---

#### **Network Connections**  
To investigate network activity, I scanned the memory dump for active or closed network connections:  

**Command**:  
```bash
volatility -f flounder-pc-memdump.elf --profile=Win7SP1x64 netscan
```

**Key Findings**:  
A TCP connection involving **10.10.99.55** (the IP address from the phishing email) was identified:  
```plaintext
0x1fc04490         TCPv4    10.10.100.43:49246             10.10.99.55:80       CLOSED           2752     powershell.exe
```

This strongly suggested that the `powershell.exe` process (PID: **2752**) was responsible for interacting with the malicious server.

---

### **Step 3: Dumping the Suspicious Process**  
To investigate the behavior of the `powershell.exe` process further, I dumped its memory:  

**Command**:  
```bash
volatility -f flounder-pc-memdump.elf --profile=Win7SP1x64 memdump -p 2752 -D myoutput/
```

**Result**:  
A file named `2752.dmp` was created, representing the memory space of the PowerShell process.

---

### **Step 4: Analyzing the Dump**  

#### **Extract Strings**  
The memory dump was processed to extract readable strings using the `strings` utility:  

**Command**:  
```bash
strings myoutput/2752.dmp > strings_output.txt
```

This generated a text file containing all human-readable content from the process's memory.

---

#### **Search for Suspicious Content**  
Using basic search techniques, I looked for:
- **PowerShell commands**: To identify potential scripts or commands executed.
- **URLs**: To find references to malicious servers.
- **Encoded Data**: Often attackers encode their payloads using Base64 or similar techniques.  

**Commands**:  
```bash
grep -i "powershell" strings_output.txt
grep -i "http" strings_output.txt
grep -i "base64" strings_output.txt
```

**Findings**:  
A **Base64-encoded string** was identified among the extracted strings. This is a common method used by attackers to obfuscate payloads.

---

### **Step 5: Decoding the Base64 String**  

The Base64 string was extracted and decoded to reveal its contents. Using the `base64` utility:  

**Command**:  
```bash
echo "<base64_string>" | base64 -d
```

**Decoded Output**:  
The payload contained the following flag:  
```plaintext
HTB{$_j0G_y0uR_M3m0rY_$}
```

---

## **Flag**  
The final flag extracted from the decoded Base64 payload:  
```plaintext
HTB{$_j0G_y0uR_M3m0rY_$}
```

---

## **Tools and Techniques Used**  

1. **Volatility Framework**: To analyze the memory dump and extract process information.  
2. **Strings**: To retrieve readable text from the memory dump.  
3. **Base64 Decoder**: To decode obfuscated payloads.  
4. **Any.Run**: For safely analyzing the email and observing its behavior.  

---

## **Key Takeaways**  

1. **Phishing is Effective**: Emails with enticing links and attachments remain a top attack vector.  
2. **Memory Analysis is Critical**: Volatility enabled detailed analysis of the system's state during the incident.  
3. **Obfuscation Techniques**: Attackers often encode malicious payloads in Base64 to bypass detection.  
4. **Proactive Investigation**: Identifying the PowerShell process and its network activity was key to unraveling the attack.  

---

## **Conclusion**  

This challenge demonstrated the importance of memory forensics in uncovering malicious activity. By carefully analyzing a memory dump, I was able to trace the attacker’s activity, decode their payload, and retrieve the flag. This exercise reinforced my skills in process analysis, network forensics, and encoded payload decoding.

---

