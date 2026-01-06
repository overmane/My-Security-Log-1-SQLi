# My-Security-Log-5-SSRF
This my public notes regarding my experiences with the Server-Side Request Forgery (SSRF) topic on PortSwigger Academy.

**Date:** January 6, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
Server-Side Request Forgery (SSRF) is a vulnerability where an attacker induces the server-side application to make requests to an unintended location. The server often has access to internal resources that are not reachable from the outside world, making this a high-impact flaw.

### 1. Identifying Entry Points
SSRF occurs whenever a server fetches a resource from a URL or internal path based on user input. 
* **Common Parameters to Target:**
    `?url=`, `?redirect=`, `?next=`, `?path=`, `?returnTo=`, `?api_url=`, `?view=`, `?feed=`
* **Mechanism:** If the server takes an argument containing an internal address/URL and executes it without proper validation, you have found an SSRF vector.

### 2. Host Discovery & Fuzzing
Once an SSRF entry point is found, it can be used to map the internal network (Intranet).
* **Intruder Method:** Use a payload like `http://192.168.0.§1§:8080/admin` to brute-force the final octet.
* **FFUF CLI Examples:**
    * **Host Discovery:** `ffuf -u 'https://target.com/api?url=http://192.168.0.FUZZ' -w <(seq 1 255) -m 2`
    * **Port Scanning:** `ffuf -u 'https://target.com/api?url=http://192.168.0.42:FUZZ' -w ports.txt`
    * **Directory Fuzzing:** `ffuf -u 'https://target.com/api?url=http://192.168.0.42:8080/FUZZ' -w common.txt`

### 3. Bypass Techniques (Filter Evasion)
Applications often use blacklists or whitelists that can be bypassed using encoding or URI confusion:
* **URL Shortcuts:** Using `127.1` or `2130706433` instead of `127.0.0.1`.
* **Obfuscation:** Using Double or Triple URL Encoding (e.g., `admin` -> `%25%36%31dmin`).
* **URL Confusion (The @ trick):** * `http://localhost:80%2523@stock.weliketoshop.net/admin`
    * **Logic:** The filter sees the trusted domain after the `@`, but the backend decodes `%2523` into `#`, causing it to ignore everything after the fragment and request `localhost:80`.

### 4. Open Redirects vs. SSRF
A simple "Open Redirect" (e.g., `site.com/login?redirect=https://google.com`) might seem low impact, but when combined with a backend SSRF check, it can be used to bypass domain whitelists. If the server trusts its own domain, you can point the SSRF to an Open Redirect on the same server, which then points to the internal target.

### 5. The Cloud Threat (IMDS)
For cloud environments (AWS, Azure, GCP), SSRF is critical due to the **Instance Metadata Service (IMDS)**.
* **Vulnerability:** Accessing `http://169.254.169.254/` (v1) allows an attacker to steal temporary security credentials (IAM roles).
* **Real-world Impact:** This was the primary vector in the Capital One breach, leading to the theft of data from 100 million customers.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> SSRF is very similar to IDOR, but instead of "tricks" with arguments like `id=123`, we "play around" with paths — for example, shifting from `?path=script.py` to `?path=http://127.0.0.1/admin`. 
>
> A realistic modern exploitation scenario: an attacker finds an SSRF -> probes the internal IP perimeter -> discovers an internal-only script -> sends a request via SSRF with a malicious payload for that script -> the server executes it -> **Boom! RCE.** The attacker gains a reverse shell, leading to server compromise or full domain escalation. 
>
> The issue isn't just that internal scripts are written with a "nobody from the outside can get here" mindset (neglecting checks), but also how we defend it. To fix this:
> * **Strict Whitelisting:** Allow only specific protocols (http/https) and domains (e.g., `images.unsplash.com`).
> * **ID Mapping:** Don't let users provide raw URLs. Use IDs instead. **Bad:** `status.php?server=http://192.168.0.10`. **Good:** `status.php?server_id=1` (where the IP is hardcoded in the DB).
> * **Deep IP Validation:** Resolve domains to IPs on the backend and verify they don't fall into private ranges (`127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.169.254`).
> * **Network Isolation:** Use a dedicated egress proxy for outgoing requests and configure firewalls to block the application server from hitting internal IPs.
>
> This "Castle-and-Moat" mentality—tough outside, soft inside—is a major flaw. IDOR and SSRF remain the most interesting vulnerabilities for manual "poking" with Burp Suite. It's always great to revisit the basics.

---

## Future Roadmap
* **Authentication bypass:** Next classic thing.

---
*Notes compiled based on PortSwigger Academy research materials.*
