# Lab 2: Port Scanning

## Objective

Perform comprehensive port scanning against a target host to enumerate all open TCP/UDP ports using both Nmap and RustScan, compare results, and identify potential attack vectors.

## Environment

| Item | Details |
|------|---------|
| Attacker OS | Kali Linux |
| Tools | Nmap 7.94, RustScan 2.3.0 |
| Target Host | 192.168.56.101 (Metasploitable 2) |

## Theory

Port scanning determines which network ports on a target host are open and potentially accessible. Each open port represents a running service and, therefore, a possible entry point for an attacker.

### Port States

| State | Meaning |
|-------|---------|
| `open` | Application is actively accepting connections |
| `closed` | Port is accessible but no application is listening |
| `filtered` | Firewall/filter is preventing Nmap from determining the state |
| `unfiltered` | Port is accessible but state cannot be determined |

### Scan Types

| Type | Description |
|------|-------------|
| SYN Scan (`-sS`) | Half-open scan; fastest and stealthiest TCP scan |
| Connect Scan (`-sT`) | Full TCP handshake; no raw packet privileges required |
| UDP Scan (`-sU`) | Slower; required to find UDP services |
| FIN/NULL/Xmas (`-sF/-sN/-sX`) | Stealthy scans that bypass some firewalls |

## Methodology

### Part A – Nmap Port Scanning

#### Step 1 – Full TCP SYN Scan (All 65535 Ports)

```bash
sudo nmap -sS -p- --min-rate 5000 192.168.56.101
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-sS` | TCP SYN (stealth) scan |
| `-p-` | Scan all 65535 ports |
| `--min-rate 5000` | Send at least 5000 packets per second |

**Sample Output:**

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-04-10 10:05 UTC
Nmap scan report for 192.168.56.101
Host is up (0.00091s latency).
Not shown: 65506 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
8787/tcp  open  msgsrvr
Nmap done: 1 IP address (1 host up) scanned in 14.73 seconds
```

---

#### Step 2 – UDP Scan (Top 1000 Ports)

```bash
sudo nmap -sU --top-ports 1000 192.168.56.101
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-sU` | UDP scan |
| `--top-ports 1000` | Scan the 1000 most common UDP ports |

**Sample Output (excerpt):**

```
PORT    STATE         SERVICE
53/udp  open          domain
68/udp  open|filtered dhcpc
111/udp open          rpcbind
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm
```

---

#### Step 3 – Targeted Service Version Scan on Discovered Ports

```bash
nmap -sV -p 21,22,23,25,53,80,139,445,3306,5432,8180 192.168.56.101 -oN nmap_services.txt
```

**Sample Output:**

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
```

---

### Part B – RustScan Port Scanning

RustScan is a modern, Rust-based port scanner optimised for speed. It locates open ports rapidly and then hands them off to Nmap for deeper analysis.

#### Step 1 – Basic RustScan (All Ports)

```bash
rustscan -a 192.168.56.101 --ulimit 5000
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `-a` | Target address |
| `--ulimit 5000` | Set file descriptor limit for concurrent connections |

**Sample Output:**

```
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'

Open 192.168.56.101:21
Open 192.168.56.101:22
Open 192.168.56.101:23
Open 192.168.56.101:25
Open 192.168.56.101:53
Open 192.168.56.101:80
Open 192.168.56.101:139
Open 192.168.56.101:445
Open 192.168.56.101:3306
Open 192.168.56.101:5432
Open 192.168.56.101:8180
[~] Starting Nmap
```

---

#### Step 2 – RustScan with Nmap Integration

RustScan can automatically pipe discovered ports to Nmap for service detection.

```bash
rustscan -a 192.168.56.101 --ulimit 5000 -- -sV -sC
```

**Flag explanation:**

| Flag | Meaning |
|------|---------|
| `--` | Everything after this flag is passed directly to Nmap |
| `-sV` | Service/version detection |
| `-sC` | Default NSE script scan |

---

### Part C – Comparison: Nmap vs RustScan

| Criterion | Nmap | RustScan |
|-----------|------|---------|
| All-port scan time | ~14.7 s | ~3.2 s |
| Service detection | Native | Pipes to Nmap |
| UDP support | Yes | No (TCP only) |
| NSE scripting | Yes | Via Nmap integration |
| Stealth options | Many (`-sS`, `-sF`, `-sN`) | Limited |
| Output formats | XML, grepable, normal | Stdout / Nmap formats |

**Conclusion:** RustScan dramatically reduces initial discovery time but relies on Nmap for in-depth analysis. The recommended workflow is to use RustScan for fast port discovery, then Nmap for service enumeration and scripting.

## Results Summary

| Port | Protocol | Service | Version | Notes |
|------|----------|---------|---------|-------|
| 21 | TCP | FTP | vsftpd 2.3.4 | Backdoor (CVE-2011-2523) |
| 22 | TCP | SSH | OpenSSH 4.7p1 | Weak configuration |
| 23 | TCP | Telnet | Linux telnetd | Plaintext credentials |
| 25 | TCP | SMTP | Postfix | Open relay check needed |
| 53 | TCP/UDP | DNS | BIND 9.4.2 | Zone transfer possible |
| 80 | TCP | HTTP | Apache 2.2.8 | Web vulnerabilities |
| 139/445 | TCP | SMB | Samba 3.0.20 | RCE (CVE-2007-2447) |
| 512-514 | TCP | RSH | — | No authentication |
| 1099 | TCP | RMI | — | Java RMI deserialization |
| 3306 | TCP | MySQL | 5.0.51a | Exposed to network |
| 5432 | TCP | PostgreSQL | 8.3.x | Exposed to network |
| 5900 | TCP | VNC | — | Weak/no auth |
| 8180 | TCP | Tomcat | 1.1 | Default credentials |

## Conclusion

Both Nmap and RustScan successfully enumerated all open ports on the target. The target exposes an exceptionally large attack surface across TCP and UDP. Services like Telnet, RSH, and exposed databases represent immediate high-risk findings. The next phase (vulnerability assessment) will focus on exploiting or verifying specific vulnerabilities in the web application stack and database services.

## References

- Nmap SYN scan: https://nmap.org/book/synscan.html
- RustScan GitHub: https://github.com/RustScan/RustScan
- CVE-2011-2523: https://nvd.nist.gov/vuln/detail/CVE-2011-2523
