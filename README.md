# 🧩 Web Vulnerability Pyramid  
### _Detection • Exploitation • Prevention_

> **A complete mapping of application vulnerabilities within the context of offensive security auditing.**

![Web Vulnerability Pyramid](./pyramide_general.png)

---

## 🎯 Project Objective

This project aims to provide a **structured overview** of web vulnerabilities and their **detection**, **exploitation**, and **prevention** cycles as part of an **offensive security audit**.

The idea is to present each phase in a **pyramidal format**, from passive steps to final detection, to understand:
- How an attack surface is identified,  
- What vulnerabilities may arise from it,  
- And what methods can be applied to fix or prevent them.

---

## 🧠 Global Vision

The **main pyramid** above represents the overall process:

1. **Passive Phase** → information gathering, content analysis, technologies, and subdomain discovery.  
2. **Active Phase** → targeted scanning, enumeration, brute forcing, vulnerability detection, and bypass techniques.  
3. **Detection Phase** → vulnerability classification according to OWASP families (XSS, SQLi, RCE, CSRF, etc.) and recommended prevention practices.

Each layer of this pyramid will be detailed in dedicated sections, each with its own explanatory diagrams.

---

## 🧩 Repository Structure

```bash
📦 Web-Vulnerability-Pyramid
 ┣ 📜 README.md
 ┣ 🖼️ pyramide_general.png
 ┣ 📂 docs/
 ┃ ┣ 📄 01-passive.md        # Passive phase – information gathering and analysis
 ┃ ┣ 📄 02-active.md         # Active phase – controlled tests and interactions
 ┃ ┗ 📄 03-detection.md      # Detection phase – vulnerability classification
 ┗ 📂 assets/
    ┗ 📂 images/             # Additional diagrams and complementary visuals
