# Module 13 – Hacking Web Servers

> Status: ✅ Reference complete

## 1. Web server concepts
A web server delivers content over HTTP (client-server model). **Components:** document root (HTML files), server root (config/logs/cgi-bin), virtual document tree (storage on another disk), virtual hosting (name/IP/port-based — multiple sites on one server), web proxy (middleman hiding client IP). Common servers: Apache, Nginx, IIS, Tomcat.

- **Goals behind hacking:** steal data (card info via phishing), botnet/DDoS, damage databases, steal closed-source apps, redirect traffic, privilege escalation, defacement.
- **Flaws that expose servers:** unpatched software, shared admin creds, unrestricted traffic, unhardened apps, verbose error messages (leak version), outdated SSL/TLS, third-party plugins, default accounts/settings, improper permissions, self-signed/default certs, excessive privileges.
- **Impact:** account compromise, defacement, secondary attacks, root access, data tampering/theft, reputation damage.

### Server architectures & vulnerabilities
- **Apache** (open-source HTTP): core + modules (mod_auth, mod_ssl, mod_rewrite, mod_proxy) + application server. Vulns: HTTP response splitting, HTTP/2 DoS (endless continuation frames, window size 0, RST stream memory), mod_macro buffer over-read, insecure default config (RCE), improper authorization, path traversal (OFBiz), SQL injection, XSS.
- **IIS** (Microsoft): HTTP.sys (protocol listener) + WWW Service + WAS; reads ApplicationHost.config, spawns worker processes; relies on inetinfo.exe + DLLs. Vulns: trust boundary violation, auth bypass, CRLF XSS, credential exposure in logs, arbitrary file-path access, elevation of privilege, directory permissions, XSS in password manager.
- **Nginx** (high-performance, master-worker, event-driven): master process + worker processes + proxy cache + cache loader/manager + app server + memcache. Vulns: NULL pointer deref in HTTP/3, SSRF, RCE (Nginx-UI config exposure), improper cert validation, SQL injection, unauthenticated private-key access (.htaccess reliance), HTTP/2 memory/CPU exhaustion, OS command injection (nginxWebUI), default file permissions.

## 2. Web server attacks
- **DNS server hijacking** — take over the DNS server, redirect real domains to fake sites.
- **DNS amplification** — bots spoof the victim's IP in recursive DNS queries → amplified replies flood the victim (DDoS).
- **Directory traversal** — `../` in the URL to reach files outside the web root.
- **Website defacement** — change site appearance/content (often via SQL injection).
- **Web server misconfiguration** — verbose errors, anon/default creds, sample files, remote admin, unnecessary services, default SSL, directory browsing, unrestricted uploads (→ web shell/RCE).
- **HTTP response splitting** — inject CR/LF into response headers → split one response into two (redirect, XSS, cache poisoning).
- **Web cache poisoning** — trick the cache into storing malicious content served to all users (relies on response splitting).
- **SSH brute force** — guess SSH creds on TCP 22 (Nmap + Ncrack), then tunnel malware.
- **FTP brute force with AI:** `hydra -L usernames.txt -P passwords.txt ftp://10.10.1.11`.
- **HTTP/2 continuation flood** — send endless CONTINUATION frames without END_HEADERS → Apache memory/CPU exhaustion (DoS).
- **Frontjacking** — CRLF + fake Host header on misconfigured Nginx reverse proxy (`$uri`/`$document_uri`) → route to attacker server (phishing/XSS).
- **Other:** web server password cracking (guessing/dictionary/brute/hybrid — Hydra/Ncrack), DoS/DDoS, MITM, phishing, web application attacks (SSRF, parameter/cookie tampering, file injection, session hijacking, SQLi, directory traversal, XSS, buffer overflow, CSRF, command injection, source-code disclosure).

## 3. Attack methodology
Information gathering → footprinting → mirroring → vulnerability scanning → session hijacking → password hacking.

### Information gathering & footprinting
- WHOIS (who.is, Domain Dossier), robots.txt (`site.com/robots.txt` — may reveal hidden paths).
- **Banner grabbing:**
  ```bash
  nc -vv www.moviescope.com 80      # then: GET / HTTP/1.0  (Enter twice)
  telnet www.moviescope.com 80      # then: GET / HTTP/1.0
  ```
  Tools: httprecon (fingerprinting via odd HTTP requests), Uniscan, Netcraft, ID Serve, Nmap, Ghost Eye.
- **AI footprinting:** `nmap -sV 10.10.1.22 && whatweb 10.10.1.22 && nikto -h 10.10.1.22`.
- **IIS via Shodan:** `http.title:"IIS"`, `http.title:"IIS Windows Server" country:"US"`, `http.title:"IIS7" port:80`, `Ssl.cert.subject.CN:"company.in" http.title:"IIS"`.
- **Apache mod_userdir user enumeration:**
  ```bash
  nmap -p80 --script http-userdir-enum <target>
  nmap -p80 --script http-userdir-enum userdir.users=<wordlist>.txt <target>
  ```

### Nmap NSE web enumeration
```bash
nmap --script hostmap-bfk <host>                   # virtual domains
nmap --script http-trace -p80 localhost            # TRACE method vuln
nmap --script http-google-email <host>             # harvest emails
nmap -p80 --script http-userdir-enum localhost     # enumerate users
nmap --script=http-waf-fingerprint -p80,443 <host> # fingerprint WAF
nmap --script http-enum -p80 <host>                # common web apps
nmap -p80 --script http-robots.txt <host>          # robots.txt
nmap -p80 --script http-waf-detect --script-args="http-waf-detect.uri=/..." <host>
nmap --script http-passwd --script-args http-passwd.root=/ <host>   # try /etc/passwd
nmap <host> -p80 --script=http-frontpage-login     # FrontPage login
```

### Later stages
- **Default credentials:** cirt.net/passwords, defaultpassword.com, Metasploit DB.
- **Default content:** debug/test functions, sample scripts, admin functions, install manuals (Nikto2).
- **Directory brute forcing** (server returns default resource / 403 / directory listing): Dirhunt, Sitechecker.
- **Vulnerability scanning:** Acunetix WVS, Fortify WebInspect, Tenable.io, Invicti, **NginxPwner** (`python3 nginxpwner.py <target_URL> /tmp/pathlist`).
- **Finding exploits:** Exploit-DB, Packet Storm; AI: `nmap -sV -O <ip> -oX scan.xml && searchsploit --nmap scan.xml`.
- **Session hijacking:** Burp Suite Sequencer (token randomness), JHijack, Ettercap.
- **Password hacking:** Hashcat, THC Hydra, Ncrack, RainbowCrack, Wfuzz.
- **App server as proxy** — abuse GET/CONNECT to reach internal systems.
- **Path traversal via misconfigured Nginx alias** (`alias` without trailing `/` → `/i/../secret.txt`): **Kyubi** (`kyubi -v <target_URL>`).
- **Attack tools:** Immunity CANVAS, OpenVAS, THC Hydra, HULK DoS, MPack.

## 4. Countermeasures
- **Placement** — put web servers in a **DMZ** between internet and internal network.
- **Patches & updates** — scan & patch regularly, test before deploy, review docs, backup plan, avoid default configs, virtual patching, automate (SolarWinds), patch alerts, inventory, integrate into SDLC.
- **Protocols** — block unused ports/ICMP/insecure protocols, TLS/SSL, redirect HTTP→HTTPS, HSTS, FTPS, isolate critical services (LDAP), hide server banners, auto-renew certs.
- **Accounts** — least privilege, disable unused accounts, strong password policy, 2FA/MFA, account lockout, password hashing (bcrypt/Argon2), audit logins, CAPTCHA.
- **Files & directories** — least privilege, disable directory listing, file integrity checkers, store web files separate from OS, filter input (`../`, `<`, `>`), secure .htaccess/web.config, scan uploads (store outside web root), WAF, SFTP.
- **Detection** — Website Change Detection System (WDS) comparing file hashes (DirectoryMonitor).
- **Defenses** — monitor/restrict ports, validate server certificates, secure machine.config (.NET), Code Access Security (block `../`, ACLs, patches), NTFS permissions, remove unused ISAPI/features, dedicated server, RBAC, centralized logging, IDPS, physical security.
- **HTTP response splitting / cache poisoning** — strip CR (`%0d`/`\r`) and LF (`%0a`/`\n`) from input, comply with RFC 2616, parse inputs before headers, UDP source-port randomization, limit recursive DNS, increase TTL.
- **DNS hijacking** — DNSSEC, trusted registrar + REGISTRAR-LOCK, strong passwords + MFA, DNS monitoring, secure router, DNS filtering (Cloudflare/Google), DoH/DoT, ACLs, VPN.
- **Scanners/tools:** Syhunt Hybrid, N-Stalker X, Invicti, Burp, OWASP ZAP, Arachni (web app); Qualys, Nikto, Mozilla Observatory (web server); Sucuri, Quttera (malware); Fortify WebInspect, UpGuard (security); CORE Impact, Cobalt Strike, Fuxploider (pentest).

## 5. Patch management
**Patches** fix bugs/security flaws; **hotfixes** are urgent fixes for live environments (bundled into service packs). **Patch management** = scan for missing patches, test, deploy. Automated process: **Detect → Assess → Acquire → Test → Deploy → Maintain**.

- **Sources** — download only from official vendor sites; register for alerts.
- **Install** — manual or automatic (auto-update); verify source, validate checksums/versions before deploying.
- **Best practices** — clear roles, asset inventory, prioritize critical patches, group/classify systems, automation, test first, regular schedule, backup before patching, verify status, include third-party apps.
- **Tools:** GFI LanGuard (Windows/macOS/Linux + third-party, rollback), Symantec Client Management, SolarWinds Patch Manager, Kaseya, Ivanti.

> Flow: footprint the server (banner grab, WHOIS, robots.txt, Shodan) → enumerate (Nmap NSE, mod_userdir, default creds/content, dir brute) → scan for vulns (Acunetix/NginxPwner) → exploit (searchsploit, Kyubi for Nginx alias, Hydra for passwords, session hijacking). Defend with DMZ placement, disciplined patch management, hardened protocols/accounts/files, CRLF filtering, DNSSEC, and continuous scanning + change detection.
