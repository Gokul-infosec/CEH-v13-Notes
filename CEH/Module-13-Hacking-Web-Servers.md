# Module 13 – Hacking Web Servers

> Status: ✅ Reference complete

## 1. Web server concepts
A web server is a computer that delivers websites, images, or videos over the internet using HTTP. Browser sends an HTTP request → server fetches and returns the content (or an error if not found). Client-server model: browser = client, web server = server. Servers: Apache, Nginx, IIS, Tomcat.

### Components of a web server
- **Document root** — main folder holding the site's HTML files. For `www.certifiedhacker.com` with document root `/admin/web/certroot`, a request to `/P-folio/index.html` fetches `/admin/web/certroot/P-folio/index.html`.
- **Server root** — top-level directory holding server config, logs, and executables. Sub-dirs: `conf` (config files), `logs` (access/error logs), `cgi-bin` (CGI scripts/server-side executables).
- **Virtual document tree** — storage on a different machine/disk when the original fills up; case-sensitive, provides object-level security.
- **Virtual hosting** — hosting multiple domains/sites on one server. Types: **name-based**, **IP-based**, **port-based**.
- **Web proxy** — middleman between user and server; forwards requests, hides the user's IP (anonymity, avoid IP blocking).

### Security issues & why servers are compromised
Web servers are open to the internet, so even behind firewalls/IDS/IPS a poorly configured server creates gaps. Organizational security spans **seven levels (stack 1–7)**.
- **Webmaster's view:** bugs in server software and insecure scripts (CGI) create holes.
- **Network admin's view:** poorly configured servers weaken LAN security or block legit access if too strict.
- **End-user's view:** risky content (JavaScript, WebAssembly) can bypass firewalls and harm systems.

### Common goals behind web-server hacking
Steal sensitive data (card info via phishing), use the server in botnets for DDoS, access/damage databases, get closed-source software, redirect traffic / hide malicious activity, gain privilege escalation; sometimes just for fun, curiosity, or to harm a company's image.

### Dangerous flaws / oversights that make servers vulnerable
Failing to apply latest patches; reusing admin creds everywhere; unrestricted internal/outbound traffic; unhardened apps/servers; complete error messages with server version; outdated SSL/TLS; third-party plugins; improper file/directory permissions; default settings/accounts (default or no passwords); unnecessary services (content management, remote admin); security vs ease-of-use conflicts; lack of security policy; improper external authentication; unnecessary default/backup/sample files; misconfigs in server/OS/network; bugs in software/OS/apps; misconfigured SSL certs; accessible admin/debug functions; self-signed/default certs; not using a dedicated server; excessive privileges / no least-privilege.

### Impact of web-server attacks
Compromise of user accounts; website defacement; launching secondary attacks (on other sites/users); root access to the server; data tampering (modify/delete/replace, add malware); data theft (financial info, plans, code); reputation damage.

### Apache web server architecture
Open-source HTTP server; robust, flexible, supports many web technologies. Components:
- **HTTP client** — browser/software initiating requests.
- **HTTP server (core)** — handles HTTP(S) requests/responses, interfaces with modules.
- **mod_auth** — user authentication (only authorized users access specific resources).
- **mod_ssl** — SSL/TLS encryption of client-server communication.
- **mod_rewrite** — URL rewriting, custom URLs, redirection by rules.
- **mod_proxy** — proxy/gateway, request forwarding, load balancing.
- **BMMTM extensible agent** — intercepts HTTP(S) requests/responses for transaction data (monitoring/performance).
- **Application server** — runs backend apps (PHP/Java/Python), generates dynamic content served by Apache.

**Apache vulnerabilities:** HTTP response splitting (inject headers → XSS/cache poisoning/info disclosure); HTTP/2 DoS by memory exhaustion on endless continuation frames; mod_macro buffer over-read (read beyond buffer → info leak); DoS in HTTP/2 with initial window size 0 (server waits indefinitely); HTTP/2 stream memory not reclaimed on RST (repeated create/reset → exhaustion); insecure default configuration (default admin creds → RCE); improper authorization (faulty checks → unauthorized access/privilege escalation); DNS rebinding in Allura import; environment variable injection (override config → code execution); code injection (Zeppelin JDBC/MySQL → RCE); improper certificate validation (Airflow FTP_TLS → MITM); XSS in log entries (Airflow); path-traversal (OFBiz); SQL injection (Submarine Server Core).

### IIS web server architecture
Microsoft's web server; supports HTTP, HTTPS, FTP, FTPS, SMTP, NNTP. Uses HTML for UI and compiled Visual Basic to process requests. Components: **Protocol listeners (HTTP.sys)**, **WWW Publishing Service (WWW service)**, **Windows Process Activation Service (WAS)**. Responsibilities: listen to requests, manage processes, read config files.

**How components function:** HTTP request → intercepted by **HTTP.sys** → HTTP.sys asks **WAS** for data from **ApplicationHost.config** (root config file) → WAS requests config (site + app pool) which goes to **WWW Service** → WWW Service configures HTTP.sys → WAS starts a **worker process** for the app pool → worker processes the request → response returned to HTTP.sys → client browser receives response. IIS relies on DLLs with the main server process **inetinfo.exe** (content indexing, server-side scripting, web-based printing).

**IIS vulnerabilities:** trust boundary violation (Telerik Report Server — unauthenticated access to restricted functionality); authentication bypass (insecure endpoint → arbitrary code execution); CRLF XSS (SiteMinder Web Agent for IIS); C•CURE passwords exposed in logs; arbitrary file-path access (Aquaforest TIFF Server — directory traversal/auth bypass); IIS elevation of privilege (improper handling of user requests); file/directory permissions (Hitachi JP1/Performance Management defaults); TYPO3 XSS (unfiltered PATH_INFO); MailEnable (file-path handling → arbitrary code execution); XSS in password manager (/isapi/PasswordManager.dll ResultURL).

### Nginx web server architecture
High-performance, scalable web server / reverse proxy / load balancer; master-worker architecture, single-threaded event-driven async non-blocking model. Components:
- **Master process** — controls/manages workers, reads config, handles sockets.
- **Worker processes** — handle client requests with non-blocking I/O (thousands of connections).
- **Proxy cache** — stores web-content copies to serve faster, reduce backend load.
- **Cache loader** — loads cache info into memory at startup.
- **Cache manager** — deletes expired/unused cache data.
- **Web server** — handles HTTP, serves static files, passes dynamic requests to app servers.
- **Application server** — runs backend scripts/apps for dynamic content.
- **Memcache** — RAM data store for fast access, reduced DB load.

**Nginx vulnerabilities:** NULL pointer dereference in HTTP/3 (QUIC module → worker termination, DoS/RCE); SSRF (anything-llm upload link → internal port scan/file deletion/LFI); RCE (Nginx-UI exposed config → privilege escalation/info disclosure); improper cert validation (Nginx-UI import → arbitrary file writes); SQL injection (unsanitized params); unauthenticated private-key access (.htaccess reliance on servers that don't support it); excessive memory/CPU exhaustion in HTTP/2; OS command injection (nginxWebUI upload feature); default file permissions (Nginx Management Suite → modify sensitive files).

## 2. Web server attacks
- **DNS server hijacking** — take control of a DNS server and change its settings so real domains resolve to attacker sites (steal info / spread malware).
- **DNS amplification attack** — abuse **recursive DNS queries**: bots spoof the victim's IP and send DNS queries; the recursion (root → .com/TLD → authoritative) produces replies that flood the victim's DNS server (DDoS).
- **Directory traversal** — use `../` in URLs/input to reach restricted folders and sensitive files outside the web root (weak input validation / misconfig).
- **Website defacement** — break in and change the site's look/content (images, text, popups, full replacement), often via SQL injection; can show propaganda or infect visitors.
- **Web server misconfiguration** — exploit config weaknesses: verbose debug/error messages, anonymous/default users/passwords, sample config/script files, remote administration, unnecessary services, misconfigured/default SSL certs, directory browsing enabled, unrestricted file uploads (→ web shell/RCE), missing Nginx root location, unsanitized user input in variables.
- **HTTP response-splitting attack** — inject newline (CR/LF) chars into response headers so the server splits one response into two; the first (attacker-controlled) can redirect to fake sites or launch XSS/CSRF/SQLi. Root cause: poor input validation.
- **Web cache poisoning** — send a crafted request that gets cached, so other users requesting the same URL get the poisoned version (relies on response splitting); persists until the cache is cleared.
- **SSH brute force** — scan for open SSH (TCP 22), then use Nmap + Ncrack to try many password combos; once in, use the encrypted SSH tunnel to send malware undetected.
- **FTP brute force with AI** — ChatGPT prompt "Attempt FTP login on target IP 10.10.1.11 with hydra using usernames and passwords from wordlists" → `hydra -L /usr/share/wordlists/ftp-usernames.txt -P /usr/share/wordlists/ftp-passwords.txt ftp://10.10.1.11` (`-L` username list, `-P` password list).
- **HTTP/2 continuation flood** — send a HEADERS frame then endless CONTINUATION frames without END_HEADERS; Apache keeps waiting and using memory/CPU → slows/crashes (DoS).
- **Frontjacking** — on misconfigured Nginx reverse proxies in shared hosting (unsafe `$uri`/`$document_uri`), inject a fake Host header via CRLF → proxy routes to attacker server → fake content/phishing/XSS. Combines CRLF injection, header injection, XSS.
- **Web server password cracking** — guess/steal login passwords for SMTP, FTP, SSH, web shares, or login forms using weak/common passwords. Techniques: **guessing**, **dictionary attack**, **brute-force**, **hybrid** (THC Hydra, Ncrack).
- **DoS/DDoS** — overload the server with fake requests targeting network bandwidth, server memory, exception handling, CPU, disk space, DB space.
- **MITM/sniffing** — intercept (and possibly alter) user↔server communication by posing as a trusted proxy.
- **Phishing** — fake email/link → fake site → steal creds/bank info entered by the victim.
- **Web application attacks** — insecure app code gives a way in even behind firewalls: SSRF, parameter/form tampering, cookie tampering, unvalidated input/file injection, session hijacking, SQL injection, directory traversal, DoS, XSS, buffer overflow, CSRF, command injection, source-code disclosure.

## 3. Web server attack methodology
Stages: **information gathering → web server footprinting → website mirroring → vulnerability scanning → session hijacking → web server password hacking.**

### Information gathering
Collect domain name, IP, server info using **Whois / who.is** (domain whois search, whois IP lookup, whois DB search). Tools: Whois Lookup (domaintools), Whois.com, Domain Dossier (centralops), Subdomain Finder (pentest-tools).
- **robots.txt** — tells search engines what to ignore but can reveal hidden/sensitive areas. View at `site.com/robots.txt` or download with Wget.

### Web server footprinting / banner grabbing
Collect software version, OS, server name, running services. Tools: Telnet, httprecon, Uniscan, Netcraft.
- **Netcat:** `nc -vv www.moviescope.com 80` [Enter], then `GET / HTTP/1.0` [Enter twice].
- **Telnet:** `telnet www.moviescope.com 80` [Enter], then `GET / HTTP/1.0` [Enter twice] → server returns version info.
- **httprecon** — web-server fingerprinting; sends normal + unusual HTTP requests. Test cases: legitimate GET for an existing resource; exceedingly long GET (URI > 1024 bytes); common GET for a non-existing resource; common HEAD for an existing resource; OPTIONS (allowed); DELETE (usually not permitted); TEST (undefined); protocol version HTTP/9.8 (nonexistent); GET with attack patterns (`../`, `%%`).
- **Uniscan** — basic commands (ping, traceroute, nslookup) + deep scans; Google/Bing searches for IPs; full report.
- **Additional:** Netcraft, ID Serve, Nmap, Ghost Eye, Skipfish.
- **AI footprinting:** prompt "Perform webserver footprinting on target IP 10.10.1.22" → `nmap -sV 10.10.1.22 && whatweb 10.10.1.22 && nikto -h 10.10.1.22`. With Netcat: `nc -v 10.10.1.22 80` then `HEAD / HTTP/1.1` + `Host:` header.
- **IIS info via Shodan filters:** `http.title:"IIS"`; `Ssl:"Company Inc." http.title:"IIS"`; `Ssl.cert.subject.CN:"company.in" http.title:"IIS"`; `http.title:"IIS Windows Server" country:"US"`; `http.title:"IIS7" port:80`; `http.title:"IIS7" net:"<IP>/24"`; `http.title:"Internet Information Services"`.

### Enumeration
- **Abusing Apache mod_userdir** (user dirs via `http://site/~username/`):
  ```bash
  nmap -p80 --script http-userdir-enum <target>                                  # default wordlist usernames.lst
  nmap -p80 --script http-userdir-enum userdir.users=<Wordlist>.txt <target>     # custom list
  nmap -p80 --script http-brute --script-args http.useragent="<User_Agent>" <target>  # custom UA to evade detection
  ```
- **Nmap NSE web enumeration:**
  ```bash
  nmap --script hostmap-bfk <host>                   # virtual domains
  nmap --script http-trace -p80 localhost            # vulnerable TRACE method
  nmap --script http-google-email <host>             # harvest emails
  nmap -p80 --script http-userdir-enum localhost     # enumerate users
  nmap --script=http-waf-fingerprint -p80,443 <host> # fingerprint WAF
  nmap --script http-enum -p80 <host>                # common web apps
  nmap -p80 --script http-robots.txt <host>          # robots.txt
  nmap -p80 --script http-waf-detect --script-args="http-waf-detect.uri=/testphp.vulnweb.com/artists.php,http-waf-detect.detectBodyChanges" www.modsecurity.org
  nmap -sV -O -p <ports> <TargetIP>                  # service versions + OS
  nmap -sV --script http-enum <TargetIP>             # web directories/apps
  nmap <TargetIP> -p 80 --script=http-frontpage-login   # FrontPage login pages
  nmap --script http-passwd --script-args http-passwd.root=/ <TargetIP>   # try /etc/passwd
  ```
- **Default credentials** — consult admin-interface docs, scan with Metasploit's built-in DB, use cirt.net/passwords, FortyPoundHead.com, defaultpassword.com, default-password.info, routerpasswords.com; password-guess/brute-force.
- **Default content** — admin debug/test functionality, sample/demo scripts, publicly accessible powerful functions (exploit with Metasploit), server installation manuals (find with Nikto2).
- **Directory brute forcing** — a directory request returns a default resource (index.html), an error (403), or a **directory listing** (exposes files). Tools: Dirhunt, Sitechecker. AI prompt: "Perform a directory traversal on target url https://certifiedhacker.com".

### Vulnerability scanning
Check for weaknesses/misconfigs; scan open ports, services, known vulns; sniff traffic. 
- **Acunetix WVS** — finds SQLi, XSS even on login/CAPTCHA pages; supports HTML5, SOAP, AJAX; detects app language/server type; compliance reports.
- **Additional:** OpenText Fortify WebInspect, Tenable.io, ImmuniWeb, Invicti.
- **NginxPwner** (Python) — scans Nginx for weak HTTP methods, wrong file perms, outdated versions, unsafe defaults: `nano /tmp/pathlist` then `python3 nginxpwner.py <target_URL> /tmp/pathlist`.
- **Finding exploits** — Packet Storm, Exploit Database. AI: `sudo apt install nmap -y && nmap -sV -O 10.10.1.19 -oX nmap_scan.xml && sudo apt install exploitdb -y && searchsploit --nmap nmap_scan.xml`.

### Session hijacking
Steal a valid session ID to control a user's session without logging in. Methods: session prediction, replay, fixation, sidejacking, XSS. Tools: **Burp Suite** (Sequencer tests token randomness → predict next ID), JHijack, Ettercap.

### Web server password hacking
Techniques: guessing, dictionary, brute-force, rainbow tables — automated with **Hashcat** (multi-hash MD4/5, SHA-224/256/384/512, RIPEMD-160; modes straight/combination/brute-force/hybrid), **THC Hydra** (FTP/SSH/Telnet/HTTP/RDP/MySQL…), Ncrack, RainbowCrack, Wfuzz, Wireshark.

### Other methodology techniques
- **Using the app server as a proxy** — abuse forwarding/reverse proxies via GET/CONNECT to attack other systems, reach internal devices, or connect to services on the proxy.
- **Path traversal via misconfigured Nginx alias** — `alias` without trailing `/` (`location /i { alias /data/w3/images/; }`) lets `/i/../secret.txt` escape the folder. **Kyubi** (`kyubi -v <target_URL>`) automates detection by recursively brute-forcing the URL path.
- **Attack tools:** Immunity CANVAS (hundreds of exploits, client-side exploitation, privilege escalation, HTTP-tunneled privesc, remote kernel exploitation, backdoors, web attacks), OpenVAS, THC Hydra, HULK DoS, MPack.

## 4. Countermeasures
### Placement
Put web servers in a **DMZ** — a separate segment between the public internet and internal network. Isolates them, lets firewalls control internet↔DMZ↔internal traffic; even if the web server is hacked, the internal network isn't easily reached.

### Patches and updates
Scan & patch regularly; test before deployment (non-production); review/peer-review patch docs; have a backup/recovery plan; avoid default configs; use virtual patching (detect/log before official patches); automate (SolarWinds); set patch alerts; inventory all systems/services/dependencies; integrate patching into the SDLC.

### Protocols
Block unused ports/ICMP/insecure protocols; apply patches; secure remote access (SSH/VPN tunneling + encryption); use TLS/SSL (HTTPS not HTTP); redirect HTTP→HTTPS; enable HSTS (stop downgrade attacks); use FTPS for file transfers; isolate critical services (LDAP on a separate subnet + firewall); hide server info (banners); automate SSL/TLS cert renewal.

### Accounts
Least privilege; disable unused/default accounts; strong password policy; 2FA/MFA; account lockout policy; password hashing (bcrypt/Argon2); audit login attempts; remove unused modules; secure account recovery; CAPTCHA on logins.

### Files and directories
Run web server with least privileges; disable directory listings; file integrity checkers; store web files on a separate drive/partition from OS/logs; filter user input (block `../`, `<`, `>`); secure config files (.htaccess, web.config not public); scan uploaded files for malware and store outside web root; WAF; avoid serving unnecessary files (backups/text/archives); use SFTP instead of FTP.

### Detecting hacking attempts
**Website Change Detection System (WDS)** monitors files (HTML/PHP/JS) by comparing current hash values to originals; alerts on any added/changed file (e.g. DirectoryMonitor sends email alerts).

### Defense details
- **Ports** — monitor all ports; restrict public access to 80/443; encrypt/limit intranet traffic; block spoofed/suspicious IPs via logs + firewall rules or blackhole routing.
- **Server certificates** — validate directly (don't rely solely on third parties); use secure protocols; let sites verify their own certs; strong crypto; valid dates and correct purpose; confirm not revoked and public key trusted to the root CA.
- **machine.config (.NET)** — block access to sensitive resources, remove unused modules, disable debugging/tracing, hide detailed errors, secure session settings (applies machine-wide).
- **Code Access Security** — secure coding (stop input tampering/source leaks), strict permissions on internet/internal code, block `../`, lock down system tools with ACLs + patches.
- **Other:** restricted ACLs, NTFS permissions, remove unnecessary ISAPI filters/script mappings/shares, dedicated non-DC web server, network segmentation (VPN + HTTPS), RBAC, centralized log monitoring, IDPS, physical security, turn off directory browsing/unused scripting.

### HTTP response-splitting & cache poisoning defense
Remove CR/LF from input before adding to headers (esp. cookies); use security testing tools for CRLF flaws; keep app engines updated; UDP source-port randomization; limit recursive DNS; increase TTL. **Server admin:** latest web-server software, patch OS/server, run a web vuln scanner. **Developers:** restrict app access to unique IPs, disallow CR (`%0d`/`\r`) and LF (`%0a`/`\n`), comply with RFC 2616, parse all inputs before HTTP headers. **Proxy servers:** don't share incoming TCP connections among clients, use different TCP connections per virtual host, implement "maintain request host header" correctly.

### DNS hijacking defense
DNSSEC; trusted ICANN-accredited registrar + REGISTRAR-LOCK; strong passwords + MFA; constant DNS monitoring; secure router (change defaults, update firmware); DNS filtering / secure DNS (Cloudflare, Google DNS); DoH/DoT; restrict access with ACLs; avoid suspicious downloads (codecs/downloaders); reputable VPN.

### Security scanners & tools
- **Web application scanners:** Syhunt Hybrid, N-Stalker X, Invicti, Burp Suite, Wapiti, WebScarab, WPSec, Tinfoil, Skipfish, Detectify, Fortify On Demand, OWASP ZAP, SonarQube, Arachni.
- **Web server security scanners:** Qualys Community Edition, Mozilla Observatory, WordPress Security Scan, Web Vulnerability Scanner (pentest-tools), Nikto, ImmuniWeb.
- **Malware infection monitoring:** QualysGuard Malware Detection, Sucuri SiteCheck, SiteLock, Quttera, Web Inspector, SiteGuarding.
- **Web server security tools:** OpenText Fortify WebInspect, Acunetix WVS, NetIQ Secure Configuration Manager, SAINT Security Suite, Sophos Intercept X for Server, UpGuard.
- **Pen testing tools:** CORE Impact, Cobalt Strike, Fuxploider, mitmproxy.

## 5. Patch management
**Patches** = software updates fixing bugs, security flaws, or performance issues (close known vulns). **Hotfixes** = quick fixes for urgent problems in live environments, often shared directly with affected users; multiple hotfixes can be bundled into a **service pack**.

**Patch management** = acquiring, testing, and installing patches; defense against vulnerabilities. Tasks: choose/verify/test/apply patches; update previously applied patches; list previously applied patches; record patch repositories/depots; assign and deploy patches.

**Automated process:** **Detect** (tools find missing patches) → **Assess** (issue + severity) → **Acquire** (download for testing) → **Test** (install on a test machine) → **Deploy** (to computers, ensure apps unaffected) → **Maintain** (subscribe to vulnerability notifications).

**Installation:**
- **Sources** — create a patch management plan per business needs; download patches only from official vendor sites; register on vendor sites for update/security alerts.
- **Methods** — manual (download + install) or automatic (auto-update feature).
- **Verify** — verify the source; use a patch management program to validate file versions and checksums; monitor patched systems; check for updates regularly.

**Best practices:** define clear roles/procedures (incl. emergencies); maintain an updated asset inventory; prioritize critical patches; group/classify systems; use automation; test before deployment; create a regular schedule; back up before patching; track/verify patch status; include third-party apps (Adobe, Java).

**Tools:** GFI LanGuard (Windows/macOS/Linux + third-party, auto-download missing patches, patch rollback), Symantec Client Management Suite, SolarWinds Patch Manager, Kaseya Patch Management, Flexera Software Vulnerability Manager, Ivanti Patch for Endpoint Manager.

> Flow: footprint the server (banner grab via nc/telnet, WHOIS, robots.txt, Shodan) → enumerate (Nmap NSE, mod_userdir, default creds/content, directory brute) → scan for vulns (Acunetix/NginxPwner) → exploit (searchsploit, Kyubi for Nginx alias, Hydra for passwords, Burp for session hijacking). Defend with DMZ placement, disciplined patch management, hardened protocols/accounts/files, CRLF filtering, DNSSEC, and continuous scanning + change detection.
