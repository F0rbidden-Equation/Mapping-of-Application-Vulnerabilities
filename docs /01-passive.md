
![plan1](./etape_passive.png)
# Web Application Analysis (Passive Phase)

## 🧩 Objectives
<p align="center">
  <img src="./diagpassiveEN.png" alt="Application Plan" width="200">
</p>

- Understand how the target website technically works (e.g. `website.com`)
- Collect visible information through the browser **without actively interacting with the server**
- Identify frameworks, technologies, redirections, and sensitive elements publicly accessible

---

## 🔍 Analysis Points

| Target Element                | Objective                                                                 |
|-------------------------------|---------------------------------------------------------------------------|
| **HTML Source Code**          | Search for **comments**, **API keys**, **JS libraries**                   |
| **JavaScript Files**          | Identify critical functions, API endpoints, frontend logic                |
| **URLs and Redirections**     | Detect parameters, internal/external redirects                            |
| **HTTP Headers**              | Identify technologies, cookies, redirections, security policies           |
| **Detected Frameworks**       | Determine versions (React, WordPress, Laravel, etc.)                      |
| **Blocked Pages**             | Bypass JS walls or hidden content via inspector                           |
| **Dynamic Behaviors**         | Check if the site loads content dynamically (AJAX/fetch)                  |
| **External Files**            | Scripts, stylesheets, fonts, trackers (Analytics, etc.)                   |
| **User Information Displayed**| Identify sensitive data exposed in DOM or JS                              |

---

## 🛠️ Tools to Use

| DevTools Panel     | Main Function                                                         |
|--------------------|-----------------------------------------------------------------------|
| DOM Inspector       | Read HTML, review comments, analyze document structure               |
| Console             | Detect JavaScript errors, logs, injections, suspicious behaviors     |
| Sources             | Access all JS / CSS / source files                                   |
| Network             | Requests, cookies, headers, AJAX/XHR/fetch endpoints                 |
| Storage             | Analyze LocalStorage, SessionStorage, IndexedDB                      |
| Application         | View manifest, service workers, cache, persistent data               |
| Security            | HTTPS, TLS info, secure cookies, mixed content detection             |

---

## 🧩 Additional Techniques to Explore

- **.map file analysis**: try accessing `main.js.map` to reconstruct unminified JS  
- **Cookie analysis**: detect JWTs, sessions, or sensitive data  
- **Monitoring AJAX requests**: identify internal endpoints like `/api`, `/v1/`, etc.  
- **Directory reconstruction**: `/admin/`, `/login/`, `/dashboard/` revealed inside scripts  
- **Framework detection**: manual review (`wp-content`, `_next`, `csrf_token`, etc.)  
- **Security header analysis**: CSP, X-Frame-Options, X-XSS-Protection, etc.  

---

## 📎 Useful Sources

- Visible page source (right-click → *View Page Source*)  
- Chrome DevTools or Firefox Developer Edition  
- Wappalyzer extension (automatic technology fingerprinting)  
- Wayback Machine (if JS or structure changed over time)

---

## 📌 Summary

This phase allows you to **map the application environment** of the target without any intrusive action.  
It is essential for better targeting the next steps (enumeration, vulnerabilities, fuzzing).

---

# 🔎 Web Content Discovery (Passive Phase)

## 🎯 Objectives
<p align="center">
  <img src="./decouverte_contenus.PNG" alt="Content Plan" width="200">
</p>

- Reveal files, pages and directories not listed in the website navigation  
- Identify technologies, frameworks, CMS, endpoints, and historical content  
- Cross-reference findings with archives, hashes, and fingerprinting  

---

## 🧭 Discovery Steps

### 🟣 **Manual Phase**

| Element | Description |
|--------|-------------|
| **robots.txt** | May reveal intentionally hidden pages (`/admin`, `/private`, etc.) |
| **favicon.ico** | Framework signature; compute **MD5 hash** to identify CMS |
| **Hash command:** | `curl https://example.com/favicon.ico | md5sum` |
| **OWASP favicon hash DB:** | https://wiki.owasp.org/index.php/owasp_favicon.database |
| **sitemap.xml** | Often lists all indexable pages/files |
| **HTTP headers analysis** | Server info, versions, protections |
| **Header inspection command:** | `curl -I http://target_ip` or `curl -v http://target_ip` |
| **HTML source analysis** | Comments, framework mentions (`<!-- Powered by X -->`) |

---

### 🔴 **Automated Phase**

| Tool / Technique | Function |
|------------------|----------|
| **Google Dorks** | Targeted dorks: `site:example.com inurl:admin` |
| **Wappalyzer** | Tech detection: CMS, JS, servers, etc. |
| **Wayback Machine** | Old pages or abandoned site structures |
| **GitHub** | Search for `.env`, `credentials`, leaked code |
| **S3 Buckets** | Explore `bucket.s3.amazonaws.com` or `example.s3.amazonaws.com` |
| **Automated directory discovery** | Hidden path brute-forcing using wordlists |

#### 🔧 Bruteforce Tools & Commands (common.txt):

```bash
# FFUF - Fast web fuzzer
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://SERVER_IP/FUZZ

# DIRB
dirb http://SERVER_IP /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt

# GoBuster
gobuster dir -u http://SERVER_IP -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt

```

---
## 🧩 Optional Complementary Tools

| Tool | Purpose |
|-------|---------|
| **GAU (GetAllUrls)** | Lists all known URLs for a domain |
| `gau example.com` | Requires `go install github.com/lc/gau` |
| **Hakrawler** | Fast crawling of URLs from an entry point |
| **Arjun** | Discovery of **GET/POST parameters** on a URL |

---

## 📌 Notes

- These techniques are **100% passive or semi-passive**, with no direct vulnerability scanning involved.  
- Results from this phase feed into the **active scanning / targeted fuzzing** phase.

---

# 🕵️ OSINT Phase – Information Gathering (Passive)

## 🎯 Objective
<p align="center">
  <img src="./OSINT.PNG" alt="OSINT Plan" width="200">
</p>

Gather as much information as possible about a target **without any intrusive interaction**.  
This phase relies exclusively on public and open sources (Open Source Intelligence).

---

## 🧩 Part 1: Primary Information Gathering

| Data Type | Tools / Commands / Sites |
|-----------|---------------------------|
| **WHOIS** (domain, DNS, email info) | `whois example.com`<br>Website: https://whois.domaintools.com |
| **SSL/TLS** (certificates, CN, SAN, dates) | ``echo \| openssl s_client -connect example.com:443``<br>Manual certificate analysis |
| **Passive DNS** (historical subdomains, resolutions) | - https://crt.sh<br>- https://securitytrails.com<br>- https://dnsdumpster.com<br>- https://shodan.io<br>- https://spyse.com |
| **Social networks** (employees, technologies, leaks) | Searches via `LinkedIn`, `Twitter`, Google Dorks: `site:linkedin.com company +tech` |
| **Data breaches** (compromised emails/passwords) | https://haveibeenpwned.com<br>https://dehashed.com<br>https://intelx.io |
| **Exposed technologies** | https://builtwith.com<br>https://netcraft.com |
| **GitHub repositories** (leaked .env, credentials, etc.) | `site:github.com "example.com"` or tools like `github-subdomains` |

---

## 🧩 Part 2: Additional Information Gathering

| Type | Tools / Sites |
|------|----------------|
| **Google Analytics / AdSense trackers** | https://spyonweb.com<br>https://publicwww.com |
| **Emails linked to the domain** | https://hunter.io<br>https://emailrep.io |
| **Special files** (policies, security info) | `http://example.com/security.txt`<br>`http://example.com/humans.txt` |
| **Favicon Hash** | `curl https://example.com/favicon.ico | md5sum`<br>Search via Shodan: `http.favicon.hash:<hash>` |
| **TLS fingerprints** | https://censys.io<br>https://crt.sh |
| **ASN / hosting lookup** | https://bgpview.io<br>https://securitytrails.com |
| **Reverse image search** | https://images.google.com |

---

## 💡 Tips

- Combine results from tools like `crt.sh`, `ffuf`, `subfinder`, etc., to expand subdomain discovery.  
- Emails collected can be tested via HaveIBeenPwned or EmailRep to check for compromise and reputation.  
- PublicWWW and SpyOnWeb are excellent for correlating websites sharing the **same Analytics or AdSense ID**.

---

## 📌 Ethical Notice

All these methods fall under **passive OSINT**.  
They generate **no alerts** and **no malicious traffic** on target systems.  
They must only be used for analysis, documentation, or **authorized penetration testing**.

---

## 📋 OSINT Template (Passive Collection)

Before starting the active phase (scanning, fuzzing, exploitation), it is essential to **centralize all information collected during the passive OSINT phase**.  
This file serves as a template to fill out for each target to prepare the next steps efficiently.

🔗 **Download the template:**  
➡️ [OSINT_Template_Passive.txt](./OSINT_Template_Passive.txt)

### 🧠 Template Contents

- General information (domain, IP, ASN, hosting provider)  
- SSL/TLS certificates  
- Passive subdomains (crt.sh, DNSDumpster, etc.)  
- Special files (robots.txt, sitemap.xml, security.txt, etc.)  
- Domain-related emails (Hunter.io, EmailRep, etc.)  
- Detected technologies (CMS, JS frameworks, etc.)  
- Social networks, data leaks  
- Site history via Wayback Machine  
- Favicon hash (Shodan fingerprint)  
- S3 buckets, GitHub leaks  
- Pre-active-phase checks  

✅ **Purpose:** Ensure nothing is missed and enter the active phase with a fully informed view of publicly exposed weaknesses.
