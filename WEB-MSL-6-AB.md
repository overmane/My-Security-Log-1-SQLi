# My-Security-Log-6-Authentication-Bypass
This my public notes regarding my experiences with the Authentication Bypass topic on PortSwigger Academy.

**Date:** January 6, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
Authentication bypass occurs when an attacker exploits vulnerabilities in the login process to gain access to accounts or administrative panels without providing valid credentials. This often involves abusing flawed logic, inadequate rate limiting, or predictable session management.

### 1. Brute-Force & Response Anomalies
The most basic entry point is brute-forcing usernames and passwords via **Burp Intruder**. 
* **Username Enumeration:** Look for subtle differences in error messages (e.g., "Invalid username" vs "Invalid password").
* **Response Timing:** Sometimes the server takes slightly longer to process a correct username, even if the password is wrong.
* **Status Codes:** A `200 OK` vs. a `403 Forbidden` can reveal if a user exists.

### 2. Flawed 2FA Implementations
A common logic error occurs when the server handles authentication in two disconnected stages:
* **Stage 1 (Login):** You enter correct credentials. The server sets a session cookie with `logged_in: true`.
* **Stage 2 (MFA):** The server redirects you to a 2FA code page, expecting to set `2fa_verified: true`.
* **The Vulnerability:** If the protected page (e.g., `/my-account`) only checks for the `logged_in` flag and ignores the 2FA status, an attacker can skip the 2FA page entirely and access the account.

### 3. Rate Limit Bypassing (The "Chess" Strategy)
Modern servers block IPs after a few failed attempts. However, logic flaws can reset these counters:
* **X-Forwarded-For (XFF) Spoofing:** If the server trusts this header, you can rotate the IP address in every request to bypass IP-based blocking.
* **Account-Based Reset:** Some servers reset the failure counter if you successfully log into *your own* account.
    * **Strategy:** Attempt to crack `carlos` (Error #1) -> Attempt `carlos` (Error #2) -> **Log into your own account** (Counter resets to 0) -> Repeat.

### 4. Logic Flaws in Token/Parameter Handling
* **Unbound Tokens:** Submitting a password reset request where the `username=carlos` is passed alongside a `temp-token`, but the server fails to verify if that specific token actually belongs to Carlos.
* **Parameter Pollution (JSON):** If the backend accepts JSON, you can try submitting an array: `"password": ["123456", "password", "qwerty"]`. The server might iterate through the list and grant access if any one matches.
* **Verification Spoofing:** Passing parameters like `verify=carlos` in a 2FA request. If the server takes this "on faith" without checking the current session, you can brute-force the MFA code for any user.

### 5. Predictable Session Tokens
If session cookies are generated using a formula (e.g., `Base64(username + ":" + MD5(password))`), they can be brute-forced.
* **Intruder Setup:**
    1.  **Payload:** Wordlist of common passwords.
    2.  **Processing:** Hash (MD5) -> Add prefix `carlos:` -> Encode (Base64).
    3.  **Result:** You generate valid session tokens without ever knowing the cleartext password.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> This category of vulnerabilities is built entirely on abusing authorization logic. Based on my research in the Academy, I've gathered a few core realizations:
>
> As a developer, the goal is to create "mute" authentication pages. They shouldn't give away any hints—no "user exists" vs "user doesn't exist" games. Using credentials (like MD5 of a password) to generate cookies is a massive "no-go"; stick to pure randomness.
>
> The "Golden Standards" for defense:
> * **Blind Responses:** Use generic error messages like "Invalid credentials" for both wrong usernames and wrong passwords.
> * **Multi-Layer Rate Limiting:** Apply limits not just by IP (which can be spoofed via XFF), but also by username.
> * **Minimalism:** Accept as few arguments as possible in a request. The more parameters you accept, the more surface area for logic manipulation.
> * **Atomic Sessions:** Don't grant "half-logged-in" states. Ensure the entire auth flow (Password + MFA) is completed before any session privileges are granted.
>
> It’s fascinating how these "simple" logic errors can bypass the most complex encryption. It's not about breaking the lock; it's about finding out the back door was left on a latch.

---

## Future Roadmap
* **File Upload Vulnerabilities:** Reverse Shell one love.

---
*Notes compiled based on PortSwigger Academy research materials.*
