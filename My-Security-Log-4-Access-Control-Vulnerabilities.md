# My-Security-Log-4-Access-Control-Vulnerabilities
This my public notes regarding my experiences with the OS Command Injection topic on PortSwigger Academy.

**Date:** January 5, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
Access control vulnerabilities occur when an application fails to prevent users from accessing resources or performing actions that are outside their intended permissions. This essentially means an attacker can walk through "doors" that should be locked—such as accessing an admin panel directory that lacks proper authentication or authorization checks.

### 1. Insecure Direct Object References (IDOR)
IDOR is a sub-type of access control where an application uses user-supplied input to access objects directly without verifying ownership.
* **Mechanism:** Changing a parameter (like `id`, `user_id`, or `orderNum`) in the URL or HTTP request body to access data belonging to another user.
* **Examples:**
    * **URL-based:** `GET /user/profile?id=123` -> change to `id=124`.
    * **Body-based:** Pushing unexpected JSON data in a POST request:
      ```json
      {
        "email": "attacker@evil.com",
        "roleid": 2
      }
      ```
* **Detection:** Always stay vigilant regarding any parameters requested in the URL or HTTP body that look like database identifiers.

### 2. Broken Trust in Client-Side Data
Sometimes developers rely on data provided by the client to determine privileges, which is a critical flaw.
* **Cookie Manipulation:** Cases where a cookie like `admin=false` can be manually changed to `admin=true` to grant administrative access.
* **Method Switching:** If a `POST` request to an admin function returns an "Unauthorized" error, switching the method to `GET` (or vice versa) might bypass poorly configured filters that only protect specific methods.

### 3. Header-Based Bypasses (Frontend vs. Backend)
This technique is typical for architectures using a **Frontend Proxy (Nginx/HAProxy/Cloudflare)** and a **Backend Server**.
* **Mechanism:** The frontend blocks access to `/admin`, but the backend uses specific headers to determine the "real" requested path. By using headers like `X-Original-Url: /admin/delete`, the frontend sees a safe request (e.g., `GET /`), while the backend processes the path provided in the header, bypassing the filter.
* **Key Headers:** `X-Original-Url`, `X-Rewrite-Url`, `X-Forwarded-For`, `X-Custom-IP-Authorization`, `X-Remote-IP`.
* **Identification:** * **Status Codes:** If `/admin` gives `403 Forbidden` (from Proxy) but `/non-existent` gives `404 Not Found` (from Backend), `/admin` is likely blacklisted at the proxy level.
    * **Server Headers:** Compare `Server` or `X-Powered-By` headers in blocked vs. normal responses to see which layer is rejecting the request.

### 4. Real-World Scenarios
* **Legacy Systems:** CRM or router panels often check permissions only on the "Main" admin page. Direct paths to functions (e.g., `/admin/delete_user?id=1`) often remain unprotected for any authenticated user.
* **Modern APIs:** A website's frontend might hide buttons, but the underlying API (e.g., `POST /api/v1/make_me_admin`) often lacks its own authorization check, assuming the frontend handles security.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> Access control vulnerabilities are often not just technical bugs, but "human-logic" failures. A developer might think: "A regular user would never know how to execute this specific HTTP request, so we don't need to check their permissions." This is gross negligence. 
> Looking at escalation vectors, I would prioritize gaining access to administrative tools that allow reading logs or, ideally, uploading files. The chain: **IDOR -> Admin File Upload -> RCE -> Reverse Shell -> Full System Compromise.**

---

## Future Roadmap
* **SSRF (Server-Side Request Forgery):** I remember that's critical for clouds. Just repeat.

---
*Notes compiled based on PortSwigger Academy research materials.*
