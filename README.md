# FristiLeaks: 1 - CTF Walkthrough

This repository contains my personal notes, technical methodology, and step-by-step walkthrough for solving the **FristiLeaks: 1** Capture The Flag (CTF) challenge from VulnHub. 

The main objective of this lab is to practice network enumeration, web application exploitation, custom script reverse engineering, and privilege escalation techniques in a Linux environment.

---

## 🛠️ Skills & Concepts Covered
* **Network & Service Enumeration:** Aggressive scanning with Nmap and web vulnerability assessment with Nikto.
* **Web Exploitation:** Directory brute-forcing, authentication bypass, and restricting file upload filters using double extensions.
* **Privilege Escalation:** Explaining insecure Cron Jobs (Scheduled Tasks) and exploiting misconfigured SUID binaries.
* **Reverse Engineering:** Analyzing a custom Python encryption script and writing a decryption tool to recover administrative credentials.

---

## 🚀 Technical Phases & Methodology

### Phase 1: Reconnaissance & Web Enumeration
* **Host Discovery:** Identified the target IP address at `10.0.2.9` within the local lab environment using its MAC address.
* **Port Scanning:** Executed an aggressive Nmap scan (`nmap -T4 -A -v`), which revealed:
  * Port 80/tcp: Apache httpd 2.2.15 running on CentOS with an outdated PHP version (5.3.3).
* **Vulnerability Assessment:** Used Nikto to discover an active HTTP TRACE method, multiple decoy directories in `robots.txt` (`/cola`, `/sisi`, `/beer`), and directory indexing.
* **Directory Brute-forcing:** Discovered a hidden administrative login interface located at `/fristi`.

### Phase 2: Exploitation & Initial Access
* **Authentication Bypass:** Logged into the `/fristi` administrative panel using credentials found during the enumeration phase (`eezeepz` / `keKkeKkeKkeKkEkEk`).
* **File Upload Bypass:** Identified a weak file extension filter in the upload section. Bypassed server-side validation using a double extension technique (`shell.php.png`).
* **Gaining a Shell:** Uploaded a PHP reverse shell, set up a Netcat listener (`nc -lvnp 1234`), and executed the payload by navigating to `/fristi/uploads/shell.php.png`. This granted initial low-privileged access as the `apache` user.

### Phase 3: Privilege Escalation & Lateral Movement
* **System Enumeration:** Discovered a note from a user named "Jerry" in the `/home/admin` directory.
* **Insecure Cron Job Exploitation:** The note revealed a scheduled task running with admin privileges that executes any command written in `/tmp/runthis` every minute.
* **Hijacking the Task:** Transferred a custom Python reverse shell script (`python-shell.py`) to the target machine and directed the cron job to execute it:  
  `echo "/usr/bin/python /tmp/python-shell.py" > /tmp/runthis`
* **Access Escalation:** Captured the incoming connection on Netcat (`nc -lvnp 4444`) to successfully escalate privileges to the `admin` user.

### Phase 4: Reverse Engineering & Local Enumeration
* **Code Analysis:** Found a custom Python script (`crypt.py`) used by administrators to obfuscate passwords. The logical flow was: `Plaintext` ➡️ `Base64 Encode` ➡️ `String Reversal` ➡️ `ROT13 Cipher`.
* **Credential Recovery:** Wrote a counter-script (`decrypt.py`) to reverse the operations (`ROT13` ➡️ `String Reversal` ➡️ `Base64 Decode`). Decrypted the encoded strings found in `whoisyourgodnow.txt` to uncover a sensitive password.

### Phase 5: Final Root Escalation
* **SUID Binary Exploitation:** Identified a custom binary named `doCom` with the SUID bit set, owned by root. The `admin` user could execute this binary as the `fristi` user via `sudo`.
* **Execution:** Leveraged the `doCom` binary to execute a Python reverse shell script with elevated root contexts:  
  `sudo -u fristi ./doCom /usr/bin/python /tmp/pythonshell.py`
* **Outcome:** Received a root interactive shell, captured the final flag in `/root/`, and achieved 100% full system compromise.

---

## 📌 Key Takeaways
1. **Broken Access Control & Upload Validation:** Relying only on front-end or weak extension checks allows attackers to gain remote code execution (RCE).
2. **Security Through Obscurity:** Using custom, reversible encryption logic to store administrative credentials provides a false sense of security.
3. **Insecure Task Scheduling:** Giving write access to files executed by privileged cron jobs allows instant privilege escalation.

---
*Note: This repository is for educational and portfolio purposes only.*