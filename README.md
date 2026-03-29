# WRP - Web Recon Parser & PEP - Privesc Parser

A suite of browser-based tools for parsing and visualizing output from common recon, enumeration, and privilege escalation tools used in penetration testing and CTF challenges.

![Web Recon Output Parser screenshot](images/wrp_screenshot.png)

**Author:** melmols

## Requirements

Any modern browser (Chrome, Firefox, Edge, Safari). No install, no server, no internet connection required.

## Installation
- git clone https://github.com/melmols/WebReconParser
- cd `WebReconParser`
- open `wrp.html` — web recon parser
- open `pep.html` — linPEAS privesc parser

## Overview

Both tools are single-file, offline HTML. No server, no dependencies, no data leaves your machine. Open directly in any modern browser.

## Supported Tools

| Tab | Tool | Output Parsed |
|-----|------|---------------|
| 🤖 Auto-Detect | — | Detects tool type automatically and routes to the correct parser |
| Feroxbuster | `feroxbuster` | HTTP responses: status codes, URLs, methods |
| SQLMap | `sqlmap` | Injection verdicts, vulnerable params, DB/OS/tech fingerprint, warnings |
| Nmap | `nmap` | Hosts, open ports, services, versions, OS detection, NSE script output |
| WPScan | `wpscan` | WordPress version, themes, plugins, users, interesting findings, next-step commands |
| IP Route | `ip route` | Network topology diagram + per-route explanation table |

### Feroxbuster

Parses directory and file brute-force output from `feroxbuster`.

- Status code summary (2xx / 3xx / 4xx / 5xx counts)
- Filterable results table: filter by status code or interesting paths only
- Configurable keyword highlighting for high-value paths (`admin`, `api`, `.git`, `.env`, etc.)
- One-click URL copy to clipboard

### SQLMap

Parses SQL injection scan output from `sqlmap`.

- Injection verdict and vulnerable parameter detection
- Database, OS, and technology fingerprint extraction
- Warnings and errors surfaced separately
- Suggested next-step commands for confirmed injections

### Nmap

Parses host and port scan output from `nmap` (`-sV -sC` style).

- Host discovery with open port and service listing
- Version banners and OS detection results
- NSE script output displayed inline
- Inline attack notes for 40+ common ports (SMB, WinRM, Redis, RDP, etc.)
- Key Observations panel: CMS/framework from `http-generator`, mail hostnames from SSL cert CNs and SANs, all discovered domains
- Next Steps panel: `/etc/hosts` entries ready to copy, web URLs to browse (HTTP/HTTPS ports, mail hosts excluded)

### WPScan

Parses WordPress security scan output from `wpscan`.

- WordPress version detection with known CVE notes
- Themes and plugins enumerated with version and vulnerability status
- Users discovered by the scanner
- Interesting findings surfaced separately
- Suggested next-step commands (xmlrpc brute-force, user enumeration, exploit search)

### IP Route

Parses `ip route` output and builds a visual network topology diagram.

- SVG diagram: machine → VPN tunnel → gateway → reachable networks
- Per-route explanation table (interface, gateway, scope, protocol, metric)
- Pivot note shown when multiple /24 networks are reachable behind the VPN gateway

## Features

- **Auto-detect:** paste any tool output and it figures out the format
- **Summary cards:** instant counts for 2xx/3xx/4xx, vuln params, interesting ports
- **Filterable tables:** filter Feroxbuster results by status code or "interesting" paths
- **Interesting keyword highlighting:** configurable keyword list flags high-value paths (admin, login, api, .git, .env, etc.)
- **Nmap port notes:** known attack vectors shown inline for 40+ common ports (SMB, WinRM, Redis, etc.)
- **Key Observations panel:** surfaces CMS/framework detections (http-generator) and mail hostnames from SSL certs above the port table
- **Next Steps panel:** auto-generates `/etc/hosts` entries from discovered hostnames and lists web targets to browse based on open HTTP/HTTPS ports
- **Next steps guidance:** SQLMap results include suggested follow-up commands
- **Markdown export:** export any parsed result as a formatted Markdown table for notes/reports
- **Copy to clipboard:** one-click URL copying from Feroxbuster results

## Usage

1. Open `wrp.html` in a browser (no web server needed) - can be used offline for engagements too.
2. Either:
   - Use the **Auto-Detect** tab and paste any supported output, or
   - Select the specific tool tab and paste the raw terminal output
3. Click **Parse Output** (or **Detect & Parse** in Auto mode)
4. Use filters and tables to triage results
5. Click **Export Markdown** to copy a formatted report

---

## PEP — Privesc Parser (`pep.html`)

A standalone linPEAS output parser focused on actionable privilege escalation. Red findings only — no noise.

### Features

- **Attack Surface:** automatically identifies SUID GTFOBins, sudo NOPASSWD rules, and dangerous Linux capabilities from red-highlighted lines, with a ready-to-run exploit command and copy button for each
- **Credentials & Hashes:** surfaces only real secrets — shadow/passwd hashes, htpasswd entries, private key references, and config file password values. Filters out log noise (SSH auth lines, HTTP access logs, package manager output)
- **Red Findings table:** all linPEAS red lines organized by section with file/path split for readability
- **Markdown export:** export findings as a formatted report

### Usage

```bash
./linpeas.sh | tee linpeas_out.txt
```

Open `pep.html`, paste the contents of `linpeas_out.txt`, and click **Parse Output**. ANSI color codes must be present (they carry the severity signal).

### Attack Surface detects

| Vector | Example exploit generated |
|--------|--------------------------|
| SUID GTFOBins | `python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'` |
| Sudo NOPASSWD | `sudo find . -exec /bin/sh -p \; -quit` |
| cap_setuid | `python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'` |
| cap_dac_read_search | `python3 -c 'print(open("/etc/shadow").read())'` |
| Writable /etc/passwd | `echo 'root2::0:0:root:/root:/bin/bash' >> /etc/passwd && su root2` |

---

## Example Workflow

Run your tool and save output, then paste into the parser:

```bash
# Feroxbuster
feroxbuster -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt | tee ferox_out.txt

# SQLMap
sqlmap -u "http://10.10.10.10/login.php" --data="user=a&pass=b" | tee sqlmap_out.txt

# Nmap
nmap -sV -sC -p- 10.10.10.10 | tee nmap_out.txt

# WPScan
wpscan --url http://10.10.10.10 -e vp,vt,u | tee wpscan_out.txt

# IP Route (copy output of ip route from target)
ip route

# LinPEAS (privesc — use pep.html)
./linpeas.sh | tee linpeas_out.txt
```

Open `wrp.html` and paste any recon output → **Detect & Parse**.
Open `pep.html` and paste linPEAS output → **Parse Output**.

## Default Interesting Keywords

`admin` `login` `upload` `api` `config` `backup` `secret` `key` `token` `dashboard` `manage` `panel` `shell` `exec` `cmd` `phpinfo` `wp-` `.git` `.env` `password` `passwd` `credential` `register` `user` `flag`

Keywords can be added/removed/reset from the Feroxbuster tab.

## Nmap Coverage

Flags interesting ports including: FTP, SSH, Telnet, SMTP, DNS, HTTP/S, Kerberos, SNMP, LDAP, SMB, MSSQL, MySQL, PostgreSQL, RDP, WinRM, Redis, VNC, MongoDB, Elasticsearch, NFS, Rsync, and more.

## Privacy

All processing is done client-side in JavaScript. No data is sent anywhere.
