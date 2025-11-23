

![plan1](./etapes_active.png)

# 🧭 Active Phase: Subdomain Enumeration

This guide provides all the **copy-paste** commands needed to discover subdomains of a target. It includes:
- Setting up the working environment,
- Installing the required tools,
- Step-by-step execution commands.

---

## 📁 Initialize Variables and Directory Structure

```bash
export DOMAIN="example.com"
export OUTDIR="out/$DOMAIN/01-subdomains"
mkdir -p "$OUTDIR"/{{raw,clean,live,screens,tmp}}
```

---

## 🛠️ Install Required Tools

```bash
# Go tools
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/tomnomnom/assetfinder@latest
go install -v github.com/owasp-amass/amass/v4/...@master
go install github.com/d3mondev/puredns/v2@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/sensepost/gowitness@latest
go install -v github.com/projectdiscovery/mapcidr/cmd/mapcidr@latest

# MassDNS
git clone https://github.com/blechschmidt/massdns.git
cd massdns && make && cd ..

# Python tool
pip install dnsgen

# jq (for parsing JSON)
sudo apt install jq -y
```

---

# 🌐 Dnsdumpster — Passive Recon via Web Interface

`Dnsdumpster` is an online tool that allows passive enumeration of public DNS information (subdomains, servers, IPs, network map) **without actively probing the target**.

---

## 🔗 Access the Tool

Official site: [https://dnsdumpster.com](https://dnsdumpster.com)

No installation required.

---

## 🧭 How to Use

1. Visit: [https://dnsdumpster.com](https://dnsdumpster.com)
2. Enter the target domain (e.g. `example.com`)
3. Solve the captcha (if any)
4. Launch the scan
5. Download the results:
   - `dnsdumpster.csv` (tabular results)
   - `dnsdumpster.svg` (visual network map)

---

## 💾 Organize the Results Locally

```bash
export DOMAIN="example.com"
export OUTDIR="out/$DOMAIN/01-subdomains"
mkdir -p "$OUTDIR/raw/dnsdumpster"

# Manually copy the files from your Downloads folder
mv ~/Downloads/dnsdumpster.csv "$OUTDIR/raw/dnsdumpster/results.csv"
mv ~/Downloads/dnsdumpster.svg "$OUTDIR/raw/dnsdumpster/map.svg"
```

---

## 🧪 (Optional) Extract Subdomains from CSV

```bash
cut -d',' -f1 "$OUTDIR/raw/dnsdumpster/results.csv" | grep -v '^$' | sort -u > "$OUTDIR/clean/dnsdumpster_subdomains.txt"
```
## 🔁 Merge Subdomains into Global List

```bash
cat "$OUTDIR/clean/dnsdumpster_subdomains.txt" >> "$OUTDIR/clean/passive_uniq.txt"
sort -u "$OUTDIR/clean/passive_uniq.txt" -o "$OUTDIR/clean/passive_uniq.txt"
```

---

## ✅ Summary

| Generated File | Description |
|----------------|-------------|
| `results.csv` | Raw results exported from the interface |
| `map.svg` | Visual DNS and IP network map |
| `dnsdumpster_subdomains.txt` | Extracted subdomains (from CSV) |
| `passive_uniq.txt` | Final enriched list of passive subdomains |

---
---

## 1️⃣ Subfinder

```bash
subfinder -d "$DOMAIN" -all -silent | tee "$OUTDIR/raw/subfinder.txt"
```

## 2️⃣ Assetfinder

```bash
assetfinder --subs-only "$DOMAIN" | tee "$OUTDIR/raw/assetfinder.txt"
```

## 3️⃣ Amass (Passive Mode)

```bash
amass enum -passive -d "$DOMAIN" -silent | tee "$OUTDIR/raw/amass_passive.txt"
```

## 4️⃣ Merge Passive Results

```bash
cat "$OUTDIR"/raw/*.txt | sort -u > "$OUTDIR/clean/passive_uniq.txt"
```

---

## 5️⃣ Puredns (Bruteforce + Resolution)

```bash
puredns bruteforce /path/to/wordlists/subdomains.txt "$DOMAIN" \
  -r resolvers.txt --wildcard-tests 5 --threads 50 \
  --write "$OUTDIR/raw/puredns_bruteforce.txt"

puredns resolve "$OUTDIR/raw/puredns_bruteforce.txt" -r resolvers.txt \
  --write "$OUTDIR/clean/bruteforce_resolved.txt"
```

---

## 6️⃣ MassDNS (Massive Resolution)

```bash
awk -v d="$DOMAIN" '{print $0"."d}' /path/to/wordlists/subdomains.txt > "$OUTDIR/tmp/fqdn.txt"

massdns/bin/massdns -r resolvers.txt -t A -o S -w "$OUTDIR/raw/massdns.out" \
  -s 10000 -q "$OUTDIR/tmp/fqdn.txt"

grep -Eo "^[^ ]+" "$OUTDIR/raw/massdns.out" | sed 's/\.$//' | sort -u \
  > "$OUTDIR/clean/massdns_resolved.txt"
```

---

## 7️⃣ DNSGen + MassDNS (Permutations)

```bash
dnsgen "$OUTDIR/clean/passive_uniq.txt" --wordlist /path/to/words.txt \
  | sed 's/[[:space:]]//g' | sort -u > "$OUTDIR/tmp/dnsgen.txt"

massdns/bin/massdns -r resolvers.txt -t A -o S -w "$OUTDIR/raw/dnsgen_massdns.out" \
  -s 10000 -q "$OUTDIR/tmp/dnsgen.txt"

grep -Eo "^[^ ]+" "$OUTDIR/raw/dnsgen_massdns.out" | sed 's/\.$//' | sort -u \
  > "$OUTDIR/clean/perms_resolved.txt"
```

---

## 8️⃣ DNSx (Resolution + IP Addresses)

```bash
cat "$OUTDIR"/clean/*resolved*.txt | sort -u > "$OUTDIR/clean/all_subdomains_resolved.txt"

dnsx -silent -a -aaaa -resp -l "$OUTDIR/clean/all_subdomains_resolved.txt" \
  -o "$OUTDIR/clean/subs_with_ips.txt"
```
## 9️⃣ HTTPx (Active Web Targets)

```bash
httpx -l "$OUTDIR/clean/all_subdomains_resolved.txt" -silent \
  -ports 80,443,8080,8443,8000,5000,3000 \
  -status-code -title -tech-detect -follow-redirects -json -threads 50 \
  -o "$OUTDIR/live/httpx_live.json"

jq -r '.url' "$OUTDIR/live/httpx_live.json" | sort -u > "$OUTDIR/live/urls.txt"
```

---

## 🔟 GoWitness (Automatic Screenshots)

```bash
gowitness file -f "$OUTDIR/live/urls.txt" -P "$OUTDIR/screens" --timeout 10
```

---

## 🔢 MapCIDR (Summarizing IPs to CIDR Blocks)

```bash
awk '{print $2}' "$OUTDIR/clean/subs_with_ips.txt" | sed 's/,/\n/g' | sort -u > "$OUTDIR/tmp/ips.txt"
mapcidr -aggregate -silent -l "$OUTDIR/tmp/ips.txt" -o "$OUTDIR/clean/cidrs.txt"
```

---

## 📦 Summary of Generated Files

| File | Description |
|------|-------------|
| `subfinder.txt`, `assetfinder.txt`, `amass_passive.txt` | Passive enumeration results |
| `bruteforce_resolved.txt`, `massdns_resolved.txt` | DNS resolution results |
| `all_subdomains_resolved.txt` | Full merged list |
| `subs_with_ips.txt` | Subdomain IPs |
| `urls.txt`, `httpx_live.json` | HTTP(s) targets |
| `screens/` | Screenshots |
| `cidrs.txt` | Aggregated IP networks (CIDRs) |

---

# 🚪 Active Phase — Port Scanning

This phase aims to identify active services on previously discovered hosts by probing open ports. Effective tools like `nmap`, `rustscan`, and `naabu` are used.

---

## 📁 Initialization

```bash
export DOMAIN="example.com"
export OUTDIR="out/$DOMAIN/02-portscan"
mkdir -p "$OUTDIR"/{raw,clean}
```

You should already have a file containing the IPs to scan, for example:

```bash
cat out/$DOMAIN/01-subdomains/clean/subs_with_ips.txt | awk '{print $2}' | sed 's/,/\n/g' | sort -u > "$OUTDIR/targets.txt"
```
## 🛠️ Tools to Install

```bash
# Nmap (classic)
sudo apt install nmap -y

# Rustscan (fast)
cargo install rustscan

# Naabu (passive TCP scanner)
go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest

# Masscan (ultra-fast, noisy)
sudo apt install masscan -y
```

## ⚡ Rustscan — Fast TCP Port Scanner

```bash
rustscan -a "$OUTDIR/targets.txt" --ulimit 5000 -b 1500 -- -sS -Pn -n -T4 -oA "$OUTDIR/raw/rustscan_output"
```

> Option `-b 1500` = number of simultaneous IP batches.

## 🔍 Nmap — Deep Port Scan

```bash
nmap -iL "$OUTDIR/targets.txt" -p- -T4 -sS -n -Pn -oA "$OUTDIR/raw/nmap_full_tcp"
```

### Scan with Service Detection (after identifying open ports)

```bash
nmap -iL "$OUTDIR/targets.txt" -p 21,22,23,25,80,443,445,3306,8080,8443   -sV -sC -A -T4 -Pn -n -oA "$OUTDIR/raw/nmap_detect_services"
```

> You can adjust the port list based on Rustscan/Naabu results.

## 🛰️ Naabu — Lightweight TCP Discovery

```bash
naabu -list "$OUTDIR/targets.txt" -top-ports 1000 -rate 5000 -o "$OUTDIR/clean/naabu_ports.txt"
```

## ⚙️ Masscan — High-Speed Scanning (Use With Caution)

```bash
masscan -iL "$OUTDIR/targets.txt" -p1-65535 --rate 10000 -oG "$OUTDIR/raw/masscan.gnmap"
```

> ⚠️ Be cautious with `--rate` to avoid bans/detection. Masscan generates heavy noise.

## ✅ Merging & Cleaning

```bash
cat "$OUTDIR"/clean/*.txt "$OUTDIR"/raw/*.gnmap "$OUTDIR"/raw/*.nmap | grep -Eo '[0-9]{1,5}/open' | cut -d'/' -f1 | sort -un > "$OUTDIR/clean/ports_all.txt"
```

## 📦 Summary of Port Scan Files

| File | Description |
|------|-------------|
| `targets.txt` | Target IPs |
| `rustscan_output.*` | Fast scan results |
| `nmap_full_tcp.*` | Full TCP port scan |
| `nmap_detect_services.*` | Service details |
| `naabu_ports.txt` | Naabu discovered ports |
| `masscan.gnmap` | Masscan result |
| `ports_all.txt` | Final aggregated port list |

---

# 🧬 Active Phase — Fingerprinting & Service Analysis

```bash
export DOMAIN="example.com"
export OUTDIR="out/$DOMAIN/03-fingerprint"
mkdir -p "$OUTDIR"/{raw,clean}
cp out/$DOMAIN/01-subdomains/live/urls.txt "$OUTDIR/targets_http.txt"
```

## Tool Installation

```bash
sudo apt install whatweb -y
npm install -g wappalyzer
sudo apt install nmap -y
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

---

## WhatWeb — CMS/Tech Detection

```bash
whatweb -i "$OUTDIR/targets_http.txt" --log-verbose="$OUTDIR/raw/whatweb_results.txt"
```

---

## Wappalyzer CLI — Web Stack Analysis

```bash
cat "$OUTDIR/targets_http.txt" | while read url; do echo "[*] $url" && wappalyzer "$url"; done > "$OUTDIR/raw/wappalyzer_output.txt"
```

---

## Nmap — Service Version Detection

```bash
nmap -iL "$OUTDIR/../02-portscan/targets.txt" -p 21,22,80,443,445,3306,8080   -sV -sC -Pn -T4 -oA "$OUTDIR/raw/nmap_services"
```

---

## Nuclei — Vulnerability Detection

```bash
nuclei -l "$OUTDIR/targets_http.txt" -severity critical,high,medium   -o "$OUTDIR/raw/nuclei_output.txt"
```

---

## 📦 Final File Summary

| File | Description |
|------|-------------|
| `whatweb_results.txt` | CMS/Tech detection |
| `wappalyzer_output.txt` | Web stack info |
| `nmap_services.*` | Version detection |
| `nuclei_output.txt` | Vulnerabilities found |

---
