Here is the updated **`README.md`** text with the repository name (**`web-app-security-lab`**) and project description clearly placed right at the top:

```markdown
# Repository Name: `web-app-security-lab`

## Repository Description
> A hands-on web application penetration testing and threat remediation repository documenting the identification, exploitation, and secure code mitigation of OWASP Top 10 vulnerabilities (including SQL Injection, DOM-based XSS, and Broken Access Control) using OWASP Juice Shop in a Dockerized Ubuntu Linux environment.

---

# Web Application Security Assessment & Penetration Testing Lab

![Target](https://img.shields.io/badge/Target-OWASP%20Juice%20Shop-orange)
![Environment](https://img.shields.io/badge/Environment-Docker%20%7C%20Ubuntu%20VM-blue)
![Focus](https://img.shields.io/badge/Focus-OWASP%20Top%2010%20%26%20Threat%20Remediation-green)

A comprehensive, hands-on web application vulnerability assessment and penetration testing lab conducted against **OWASP Juice Shop** running inside a Dockerized Ubuntu Linux environment (`Cybersecurity-Lab`).

This project demonstrates practical exploitation of critical web application security flaws—including **Broken Access Control**, **SQL Injection (Authentication Bypass)**, and **DOM-based Cross-Site Scripting (XSS)**—alongside detailed developer code remediation guidelines.

---

## 📋 Executive Summary

The primary objective of this assessment was to evaluate the security posture of an intentional vulnerability laboratory (OWASP Juice Shop), identify implementation defects, document exploitation mechanics, and provide actionable secure coding practices to mitigate risks.

### Key Assessment Findings
* **Broken Access Control:** Unlinked directory exposure allowing unauthorized access to confidential internal documents.
* **SQL Injection (SQLi):** Direct string concatenation in the authentication module enabling complete administrative login bypass.
* **DOM-based Cross-Site Scripting (XSS):** Unsanitized client-side rendering permitting arbitrary JavaScript execution via URL parameters.

---

## 🛠️ Lab Architecture & Setup

* **Host Machine:** Ubuntu Virtual Machine (`Cybersecurity-Lab`)
* **Containerization Engine:** Docker Engine (`sudo docker start juiceshop`)
* **Target Application:** OWASP Juice Shop (`http://localhost:3000`)
* **Browser Client:** Epiphany / Mozilla Firefox
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
    └── 07_dom_xss_solved.png

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

## 🛠️ How to Replicate This Lab

1. **Start the Docker Service:**
```bash
sudo docker start juiceshop

```


2. **Access the Application:**
Open your browser and navigate to `http://localhost:3000`.
3. **Execute SQL Injection Bypass:**
Go to the Login page and input `' OR 1=1--` as the email address with any password.
4. **Execute DOM XSS Payload:**
Click the search icon and enter `<iframe src="javascript:alert(`xss`)">`.

---

## 👤 Author

* **Name:** Godfred Acheampong
* **Role:** Municipal ICT Coordinator | Cybersecurity & IT Professional
* **Certifications:** CompTIA Security+ | ISC2 Certified in Cybersecurity (CC)
* **GitHub:** [godfredachie-web](https://github.com/godfredachie-web)
* **LinkedIn:** [godfred-acheampong05](https://linkedin.com/in/godfred-acheampong05)
```

```
