# Lab 1: Network Reconnaissance

## Objective

Perform passive and active network reconnaissance against a target network to discover live hosts, map the network topology, and gather operating system and service information using Nmap.

## Environment

| Item | Details |
|------|---------|
| Attacker OS | Kali Linux |
| Tool | Nmap 7.94 |
| Target Network | 192.168.56.0/24 (isolated lab network) |
| Target Host | 192.168.56.101 (Metasploitable 2) |

## Theory

Network reconnaissance is the first phase of a penetration test (following the PTES methodology). The goal is to collect as much information as possible about the target without triggering intrusion detection systems. Reconnaissance can be:

- **Passive** – collecting information from public sources (WHOIS, DNS, Shodan) without directly touching the target.
- **Active** – sending packets to the target to enumerate live hosts, open ports, and running services.

Nmap (Network Mapper) is the de-facto standard for active reconnaissance. It uses raw IP packets to determine:

1. Which hosts are available on the network.
2. What services (application name and version) those hosts offer.
3. What operating systems they are running.
4. What type of packet filters/firewalls are in use.

## Methodology

### Step 1 – Host Discovery (Ping Sweep)

Identify live hosts on the target subnet without performing a full port scan.

```bash
nmap -sn 192.168.56.0/24
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-sn` | Ping scan – disable port scan, only check host availability |

**Sample Output:**

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-10 09:15 UTC
Nmap scan report for 192.168.56.1
Host is up (0.00023s latency).
Nmap scan report for 192.168.56.101
Host is up (0.00087s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.43 seconds
```

**Findings:** Two hosts are alive on the /24 subnet — the gateway (`.1`) and the target (`.101`).

---

### Step 2 – OS Detection and Service Version Scan

Perform a comprehensive scan of the target to identify operating system details and service versions.

```bash
nmap -sV -O -A 192.168.56.101
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-sV` | Probe open ports to determine service/version info |
| `-O` | Enable OS detection |
| `-A` | Aggressive scan: OS detection + version detection + script scanning + traceroute |

**Sample Output:**

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-10 09:18 UTC
Nmap scan report for 192.168.56.101
Host is up (0.00091s latency).
Not shown: 977 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
OS details: Linux 2.6.9 - 2.6.33
```

**Findings:**

- **OS:** Linux 2.6.x kernel (consistent with Ubuntu 8.04)
- Multiple outdated and vulnerable services running (vsftpd 2.3.4, OpenSSH 4.7, Samba 3.0.20)
- Web servers exposed on port 80 (Apache) and 8180 (Tomcat)
- Database services exposed: MySQL (3306) and PostgreSQL (5432)

---

### Step 3 – NSE Script Scan (Default Scripts)

Run Nmap's built-in scripting engine to gather additional information.

```bash
nmap -sC 192.168.56.101
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-sC` | Run default NSE scripts equivalent to `--script=default` |

**Sample Output (excerpt):**

```
PORT    STATE SERVICE
21/tcp  open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.56.100
|      Logged in as ftp
|_    vsFTPd 2.3.4 - secure, fast, stable
```

**Findings:** Anonymous FTP login is allowed — a significant misconfiguration that could allow unauthenticated file access.

---

### Step 4 – Save Output for Reporting

Save scan results in all formats for documentation.

```bash
nmap -sV -O -A 192.168.56.101 -oA recon_output
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-oA <basename>` | Output in all formats: `.nmap` (normal), `.xml`, `.gnmap` (grepable) |

## Results Summary

| Port | Service | Version | Risk |
|------|---------|---------|------|
| 21 | FTP | vsftpd 2.3.4 | **Critical** – backdoor vulnerability (CVE-2011-2523) |
| 22 | SSH | OpenSSH 4.7p1 | Medium – outdated version |
| 23 | Telnet | Linux telnetd | **High** – plaintext authentication |
| 25 | SMTP | Postfix smtpd | Low – mail relay check recommended |
| 80 | HTTP | Apache 2.2.8 | **High** – outdated, multiple CVEs |
| 445 | SMB | Samba 3.0.20 | **Critical** – CVE-2007-2447 (username map script) |
| 3306 | MySQL | 5.0.51a | **High** – exposed to network without auth restrictions |

## Conclusion

Active reconnaissance with Nmap revealed a target host running an outdated Linux distribution with numerous critical vulnerabilities across multiple services. The attack surface is large, and several services have known public exploits. This information forms the foundation for the subsequent port scanning and vulnerability assessment phases.

## References

- Nmap documentation: https://nmap.org/docs.html
- PTES Technical Guidelines: http://www.pentest-standard.org/
- CVE-2011-2523 (vsftpd backdoor): https://nvd.nist.gov/vuln/detail/CVE-2011-2523
- CVE-2007-2447 (Samba): https://nvd.nist.gov/vuln/detail/CVE-2007-2447
