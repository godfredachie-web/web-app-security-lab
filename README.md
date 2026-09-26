# Repository Name: `web-app-security-lab`

## Repository Description
> A hands-on web application penetration testing and threat remediation repository documenting the identification, exploitation, network traffic analysis, and secure code mitigation of OWASP Top 10 vulnerabilities (including SQL Injection, DOM-based XSS, and Broken Access Control) using OWASP Juice Shop in a Dockerized Ubuntu Linux environment.

---

# Web Application Security Assessment & Penetration Testing Lab

![Target](https://img.shields.io/badge/Target-OWASP%20Juice%20Shop-orange)
![Environment](https://img.shields.io/badge/Environment-Docker%20%7C%20Ubuntu%20VM-blue)
![Focus](https://img.shields.io/badge/Focus-OWASP%20Top%2010%20%26%20Threat%20Remediation-green)
![Analysis](https://img.shields.io/badge/Analysis-Wireshark%20Packet%20Capture-red)

A comprehensive, hands-on web application vulnerability assessment and penetration testing lab conducted against **OWASP Juice Shop** running inside a Dockerized Ubuntu Linux environment (`Cybersecurity-Lab`).

This project demonstrates practical exploitation of critical web application security flaws—including **Broken Access Control**, **SQL Injection (Authentication Bypass)**, and **DOM-based Cross-Site Scripting (XSS)**—alongside low-level HTTP network traffic analysis in Wireshark and detailed developer code remediation guidelines.

---

## 📋 Executive Summary

The primary objective of this assessment was to evaluate the security posture of an intentional vulnerability laboratory (OWASP Juice Shop), identify implementation defects, document exploitation mechanics, perform packet-level verification, and provide actionable secure coding practices to mitigate risks.

### Key Assessment Findings
* **Broken Access Control:** Unlinked directory exposure allowing unauthorized access to confidential internal documents.
* **SQL Injection (SQLi):** Direct string concatenation in the authentication module enabling complete administrative login bypass.
* **DOM-based Cross-Site Scripting (XSS):** Unsanitized client-side rendering permitting arbitrary JavaScript execution via URL parameters.
* **Network Traffic Analysis:** Raw HTTP loopback traffic captured via Wireshark during automated SQLi payload execution (`wget`), confirming plain-text authentication tokens in HTTP responses.

---

## 🛠️ Lab Architecture & Setup

* **Host Machine:** Ubuntu Virtual Machine (`Cybersecurity-Lab`)
* **Containerization Engine:** Docker Engine (`sudo docker run -d -p 3000:3000 bkimminich/juice-shop`)
* **Target Application:** OWASP Juice Shop (`http://localhost:3000`)
* **Packet Analyzer:** Wireshark (`Loopback: lo` interface, filter: `http && tcp.port == 3000`)
* **Browser / CLI Tools:** Epiphany / Mozilla Firefox / `wget`
* **Assessment Framework:** OWASP Top 10 Security Risks

---

## 📁 Repository Structure & Evidence Artifacts

```text
.
├── README.md                          # Project Documentation
├── reports/
│   ├── web_app_sec_report.html        # Formatted Interactive HTML Report
│   └── web_app_sec_report.pdf         # Printable Executive PDF Report
└── screenshots/
    ├── 01_docker_installation_success.png
    ├── 02_juiceshop_container_started.png
    ├── 03_juiceshop_web_interface.png
    ├── 04_scoreboard_unlocked.png
    ├── 05_confidential_document_solved.png
    ├── 06_admin_login_sqli_solved.png
    ├── 07_dom_xss_solved.png
    └── 08_wireshark_sqli_capture.png

```

### Artifact Index

| Step / Phase | Objective | Evidence File |
| --- | --- | --- |
| **Setup** | Docker Installation Verification | `01_docker_installation_success.png` |
| **Setup** | Juice Shop Container Execution | `02_juiceshop_container_started.png` |
| **Phase 1** | Target Web Application Access | `03_juiceshop_web_interface.png` |
| **Phase 1** | Hidden Challenge Scoreboard Access | `04_scoreboard_unlocked.png` |
| **Phase 1** | Unlinked Confidential File Retrieval | `05_confidential_document_solved.png` |
| **Phase 1** | Admin Authentication Bypass via SQLi | `06_admin_login_sqli_solved.png` |
| **Phase 2** | Client-Side DOM XSS Payload Execution | `07_dom_xss_solved.png` |
| **Phase 3** | Wireshark Packet Capture of SQLi Request & Response | `08_wireshark_sqli_capture.png` |

---

## 🔍 Detailed Vulnerability Breakdown

### 1. Broken Access Control (Sensitive Information Disclosure)

* **Severity:** `Medium`
* **Mechanism:** The application fails to restrict public access to unlinked system assets and backup directories.
* **Exploitation:** By inspecting asset references and directly navigating to public storage paths, internal corporate documentation (`acquisitions.md`) was exposed and downloaded without authentication.
* **Remediation:** Enforce role-based access control (RBAC), disable directory browsing, and store sensitive assets outside the public web root.

---

### 2. Broken Authentication via SQL Injection (SQLi)

* **Severity:** `Critical`
* **Mechanism:** The login form dynamically constructs SQL query strings using direct user string concatenation. The single quote (`'`) acts as a control character that closes the string input boundary and injects custom SQL directives.
* **Payload Used:** `' OR 1=1--`
* **Exploitation Mechanics:**

$$\text{SELECT * FROM Users WHERE email = '} + \textbf{' OR 1=1--} + \text{' AND password = ...}$$

* `'` closes the email input parameter early.
* `OR 1=1` satisfies the query condition as `TRUE` for the first record in the database (the Administrator).
* `--` comments out the remainder of the SQL query, skipping password validation entirely.

#### Code Remediation (Parameterized Queries)

Developers must separate SQL logic from user data using **Prepared Statements**.

```javascript
// ❌ VULNERABLE CODE (String Concatenation)
let query = "SELECT * FROM Users WHERE email = '" + req.body.email + "' AND password = '" + req.body.password + "'";
db.query(query);

// ✅ REMEDIATED CODE (Parameterized Query)
let query = "SELECT * FROM Users WHERE email = :email AND password = :password";
db.query(query, {
    replacements: {
        email: req.body.email,
        password: req.body.password
    }
});

```

---

### 3. DOM-based Cross-Site Scripting (XSS)

* **Severity:** `High`
* **Mechanism:** The client-side application extracts URL parameters (`#search?q=...`) and dynamically renders them into the DOM using unsafe HTML parsing methods (`innerHTML`) without input sanitization or output encoding.
* **Payload Used:**

```html
<iframe src="javascript:alert(`xss`)">

```

* **Exploitation Mechanics:** The browser parses the injected string as active DOM nodes instead of literal text, triggering an inline JavaScript execution context (`alert('xss')`).

#### Code Remediation (Safe DOM Parsing)

Developers must assign untrusted user input using properties that treat input strictly as text nodes.

```javascript
// ❌ VULNERABLE CODE (Unsafe HTML Parsing)
document.getElementById('searchResults').innerHTML = searchValue;

// ✅ REMEDIATED CODE (Safe Text Node Assignment)
document.getElementById('searchResults').textContent = searchValue;

```

---

## 📡 Network Packet Capture & Traffic Analysis

To inspect the raw payload transport and session response at the network layer, a packet analysis lab was executed using Wireshark on the Ubuntu host.

### Capture Setup & Command Execution

1. Wireshark was attached to the **Loopback interface (`lo`)** with the display filter set to `http && tcp.port == 3000`.
2. The authentication bypass payload was transmitted directly via `wget`:

```bash
wget --post-data='{"email":"'\'' OR 1=1--","password":"anything"}' \
     --header="Content-Type: application/json" \
     http://localhost:3000/rest/user/login -O -

```

### Wireshark Stream Inspection

#### Packet Analysis Observations

* **HTTP Request Frame:** Captures the `POST /rest/user/login` payload originating from `Wget/1.25.0`. The body clearly reflects the malicious JSON object containing `{"email":"' OR 1=1--","password":"anything"}`.
* **HTTP Response Frame:** Juice Shop responds with `HTTP/1.1 200 OK`, returning an authentication payload containing a administrative JSON Web Token (`token: "eyJ0eXAi..."`) for `admin@juice-sh.op`.
* **Security Insight:** Unencrypted HTTP communication allows credential and session token interception via network eavesdropping. Enforcing TLS/HTTPS alongside secure header controls (`nosniff`, `SAMEORIGIN`) is critical to protect sensitive authentication exchanges.

---

## 🛠️ How to Replicate This Lab

1. **Start the Docker Container:**
```bash
sudo docker run -d -p 3000:3000 bkimminich/juice-shop

```


2. **Launch Wireshark Capture:**
* Open Wireshark and select the **Loopback: lo** interface.
* Set display filter to: `http && tcp.port == 3000`.


3. **Execute SQL Injection & Traffic Capture:**
* Run the CLI payload:
```bash
wget --post-data='{"email":"'\'' OR 1=1--","password":"anything"}' --header="Content-Type: application/json" http://localhost:3000/rest/user/login -O -

```


* Stop the capture in Wireshark, right-click the `POST /rest/user/login` packet, and select **Follow $\rightarrow$ HTTP Stream**.


4. **Execute DOM XSS Payload:**
* Access `http://localhost:3000` in your web browser.
* In the search bar, enter: `<iframe src="javascript:alert(`xss`)">`.



---

## 👤 Author

* **Name:** Godfred Acheampong
* **Role:** Municipal ICT Coordinator | Cybersecurity & IT Professional
* **Certifications:** CompTIA Security+ | ISC2 Certified in Cybersecurity (CC)
* **GitHub:** [godfredachie-web](https://github.com/godfredachie-web?utm_source=gemini)
* **LinkedIn:** [godfred-acheampong05](https://linkedin.com/in/godfred-acheampong05?utm_source=gemini)

```

```
