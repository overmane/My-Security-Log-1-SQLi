# My-Security-Log-7-File-Upload-Vulnerabilities
This my public notes regarding my experiences with the File Upload Vulnerabilities topic on PortSwigger Academy.

**Date:** January 6, 2026  
**Source:** PortSwigger Academy & Personal Study Notes

---

## Research Overview
File upload vulnerabilities arise when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size. Failing to properly enforce restrictions on these can mean that even a basic image upload function can be used to upload potentially dangerous arbitrary files, including server-side executable scripts that enable Remote Code Execution (RCE).

### 1. Flawed Content-Type Validation
Many servers attempt to filter uploads by checking the `Content-Type` header provided in the POST request. Since this header is controlled by the user, it is trivial to spoof.
* **The Bypass:** If the server rejects `application/octet-stream` but allows `image/jpeg`, simply intercept the request and change the header while keeping your malicious `.php` extension.
* **Example:**
  ```http
  Content-Disposition: form-data; name="avatar"; filename="exploit.phtml"
  Content-Type: image/jpeg

  <?php echo file_get_contents('/home/carlos/secret'); ?>

### 2. Path Traversal & Execution Context
Sometimes a file is successfully uploaded but the server prevents it from executing in the current directory. Attackers can attempt to move the file to a different, more "permissive" directory using path traversal.
* **Encoding:** Use URL encoding for the directory separators (`..%2f`) to bypass basic path filters.
* **Result:** `filename="..%2fexploit.php"` may move the shell from `/avatars/` to a parent directory where the server configuration allows script execution.

### 3. Overriding Server Configuration (.htaccess)
On Apache servers, the `.htaccess` file provides directory-level configuration. If an attacker can upload this file, they can change how the server treats specific file extensions.
* **The "AddType" Attack:**
    1. Upload a `.htaccess` file containing: `AddType application/x-httpd-php .shell`
    2. This tells the server: "Treat any file ending in `.shell` as a PHP script."
    3. Upload `shell.shell` containing your PHP payload.
* **Outcome:** The server bypasses its extension blacklist (since `.shell` isn't blocked) but still executes the file as PHP.

### 4. Obfuscation & Polyglots
When simple filters are in place, attackers use various "camouflage" techniques to hide the nature of the file:
* **Null Byte Injection:** Using `shell.php%00.jpg`. The validation logic sees `.jpg`, but the filesystem stops at the Null byte, saving it as `shell.php`.
* **Polyglots (Image Metadata):** Hiding payloads inside legitimate image files to bypass "MIME-type" or "Magic Byte" checks.
    * **The Strategy:** Use a tool like `exiftool` to inject a PHP payload into the `Comment` field of a JPEG.
    * **Command:** `exiftool -comment="<?php echo file_get_contents('/home/carlos/secret'); ?>" photo.jpg -o shell321.php`
    * **Effect:** The file maintains a valid `image/jpeg` signature, but when called by the server, the PHP engine executes the embedded comment.

---

## Personal Insights & Conclusions

> **Author's Perspective:**
> Working with File Upload vulnerabilities is arguably the most engaging part. You're no longer just trying to abuse the frontend like in XSS or scamming authorization logic—you're trying to push for RCE. The filter pushes back, and you try every which way to shove your `<?php...?>` through the gap.
>
> When it comes to the danger of these vulnerabilities—it’s no joke; it’s a severe security breach. Any kid with a laptop these days will eventually try to drop a shell into your upload form, and you’d better have a filter more robust than just checking if `.php` becomes `.PhP` before saying "Welcome aboard!"
>
> **Remediation Strategies:**
> * **a) No blacklists, obviously—only whitelists:** Always validate only the final extension after the very last dot.
> * **b) Filename regeneration:** If you receive `photo.jpg`, save it as something like `a7f2...56b.jpg`.
> * **c) Don't trust Content-Type headers:** Never trust the header from the request; the server must read the file's first bytes to determine its true type. In PHP, the `Fileinfo` extension is built for this.
> * **d) Image re-processing:** After upload, the server should pass the image through a graphics library (like GD or ImageMagick) to create a copy—all unnecessary metadata and any hidden PHP code in comments will be wiped.
> * **e) Isolated storage:** Store files on a separate server (cloud) or in a directory outside the web root (e.g., `/var/www/uploads` instead of `/var/www/html/uploads`). 
> * **f) Execution prevention:** In the server configuration (Apache/Nginx), strictly define the upload folder as "no-execution" for any scripts.
> * 
---

## Future Roadmap
* **New Repository — Active Directory:** The Big One.

---
*Notes compiled based on PortSwigger Academy research materials.*
