Security Testing & Vulnerability Assessment Report: DVWA Lab

Target Environment: Damn Vulnerable Web Application (DVWA) hosted on Metasploitable2 (192.168.56.102)

Executive Summary
This security assessment evaluates the vulnerability posture of the Damn Vulnerable Web Application (DVWA) across multiple security tiers (Low, Medium, High). Standard web application security flaws—including SQL Injection, Reflected and Stored Cross-Site Scripting (XSS), Cross-Site Request Forgery (CSRF), and Local File Inclusion (LFI)—were successfully identified, exploited, and patched using secure coding practices and server hardening techniques.

1. Vulnerability Assessment & Exploitation Evidence
A. SQL Injection (SQLi)
Description: The application fails to sanitize or parameterize user-supplied input within authentication and data retrieval queries.

Exploitation: Bypassed authentication controls on the low-security login form using payload ' OR '1'='1 to dump database table records.

Mitigation: Implement parameterized queries and Prepared Statements using PHP PDO:

PHP
$stmt = $pdo->prepare('SELECT user, password FROM users WHERE user = ?');
$stmt->execute([$username]);
B. Cross-Site Scripting (XSS - Reflected & Stored)
Description: User input is reflected or stored without proper output encoding or HTML entity escaping.

Exploitation: Executed JavaScript alert popups (<script>alert(document.cookie)</script>) via vulnerable GET parameters and guestbook input fields.

Mitigation: Apply context-aware output encoding using htmlspecialchars() before rendering untrusted data in the Document Object Model (DOM).

C. Cross-Site Request Forgery (CSRF)
Description: State-changing operations (such as password modification) lack validation tokens, allowing unauthorized command execution via third-party contexts.

Exploitation: Executed direct GET request parameter changes on low security, bypassed Referer checks on medium security, and extracted per-session dynamic tokens (user_token) on high security.

Mitigation: Enforce cryptographically secure, per-session anti-CSRF tokens validated server-side on all state-changing requests.

D. Local File Inclusion (LFI)
Description: Unsanitized file path inputs allow directory traversal outside the intended web root.

Exploitation: Traversed directory paths using ../../../../etc/passwd to dump system user accounts.

Mitigation: Restrict file inclusions to a whitelist of approved filenames and disable allow_url_include in php.ini.

E. Burp Suite Advanced Fuzzing
Description: Automated brute-force vulnerability testing against authentication endpoints.

Exploitation: Captured HTTP requests using Burp Proxy, configured Intruder payload injection points (§username§ and §password§), and analyzed response length variances to isolate valid login credentials.

2. Web Server Hardening & Security Headers
To mitigate client-side attacks, clickjacking, and MIME-sniffing vulnerabilities, defensive HTTP response headers were configured in the Apache global configuration (/etc/apache2/apache2.conf):

Apache
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-XSS-Protection "1; mode=block"
Header always set X-Content-Type-Options "nosniff"
