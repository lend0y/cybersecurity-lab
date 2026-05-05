# 🔐 Cybersecurity Lab — Ahmad Qayyim

Praktikum Keamanan Jaringan | Teknik Informatika  
Universitas Muhammadiyah Malang | 2026

---

## 📋 Daftar Modul

### 🟥 Module 1 — Introduction to Stages of Hacking
**Tools:** Gobuster, WhatWeb, Nuclei, FFuF, Nikto, Hydra  
**Highlight:** Vulnerability discovery pada DVWA menggunakan 6 tools reconnaissance. Ditemukan 12 vulnerability termasuk SQLi, XSS, File Upload, dan default SSH credentials (CRITICAL).

---

### 🟧 Module 2 — Port Scanning & Network Vulnerability Assessment
**Tools:** RustScan, Nmap, NSE (Nmap Scripting Engine)  
**Highlight:** Scan 11 port terbuka pada 9 container Docker. Identifikasi versi layanan (MySQL 5.7.44, OpenSSH 7.6p1, nginx 1.21.6, dll). Klasifikasi 9 vulnerability dari CRITICAL hingga LOW.

| Severity | Jumlah | Contoh Temuan |
|----------|--------|---------------|
| CRITICAL | 1 | Default SSH credentials root/root |
| HIGH | 4 | OpenSSH lama, MySQL EOL, PostgreSQL & Modbus exposed |
| MEDIUM | 3 | HTTP TRACE enabled, MQTT tanpa autentikasi |
| LOW | 1 | nginx versi lama |

---

### 🟨 Module 3 — SQL Injection & Brute Force
**Tools:** SQLMap, WPScan  
**Highlight:** Eksploitasi SQL Injection pada plugin WordPress vulnerable. Berhasil dump tabel `wp_users` dan crack password hash. Brute Force via XML-RPC berhasil dalam 17 detik.

---

### 🟦 Phishing Simulation — Credential Harvesting (Red Team)
**Tools:** Zphisher, LocalXpose, Kali Linux  
**Highlight:** Simulasi serangan phishing menggunakan halaman login tiruan TikTok. Berhasil capture username, password, IP, dan User-Agent target.

---

## 🛠️ Environment
- OS: Kali Linux (Virtual Machine)
- Platform: Docker / Docker Compose
- Target: DVWA, WordPress, SSH Server, MySQL, PostgreSQL, MQTT, RabbitMQ, Modbus

## 👤 Author
**Ahmad Qayyim** — 202310370311286  
Teknik Informatika, Universitas Muhammadiyah Malang
