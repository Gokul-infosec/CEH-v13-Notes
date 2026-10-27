# Module 03 – Scanning Networks

> Status: ✅ Reference complete

## 1. Goal
Gather detailed info about the target with aggressive active recon — discover exploitable channels, probe listeners, track responsive ones. Identifies hosts, ports, and services to profile the target. Tools: Nmap, Hping3, Metasploit, NetScanTools Pro.

**Types of scanning:**
- **Port scanning** — lists open ports and services.
- **Network scanning** — lists active hosts and IP addresses.
- **Vulnerability scanning** — shows known weaknesses.

**Objectives:** live hosts / IPs / open ports → OS & architecture → running services → vulnerabilities.

## 2. Scanning tools
- **Nmap** (https://nmap.org) — network discovery, admin, security auditing, inventory, uptime monitoring. Syntax: `nmap <option> <target>`.
- **Hping3** (https://salsa.debian.org) — crafts custom TCP/UDP/ICMP packets; port scan, firewall detection, OS fingerprint, can bypass some filters.
- **Metasploit** (https://www.metasploit.com) — find/test/exploit weaknesses; mix exploits + payloads, post-access, reporting.
- **NetScanTools Pro** (https://www.netscantools.com) — IPs, ports, domains, emails; many tools in one.
- Others: sx, RustScan, MegaPing, SolarWinds Engineer's Toolset, PRTG Network Monitor.

### Hping3 commands
```bash
hping3 -1 <target>                                 # ICMP echo (ping) — is host up
hping3 -A <target> -p <port>                       # TCP ACK — test firewall filtering
hping3 -2 <target> -p <port>                       # UDP — open/closed UDP port
hping3 <target> -Q -p <port>                       # TCP sequence-number prediction (spoofing)
hping3 -S <target> -p <port> --tcp-timestamp       # SYN + timestamp — OS fingerprint
hping3 -8 <Port_Range> -S <target> -v              # SYN scan over a port range, verbose
hping3 -F -P -U <target> -p <port>                 # FIN+PSH+URG (Xmas) scan
hping3 -1 <target> --rand-dest -I eth0             # ICMP sweep to random IPs on eth0
hping3 -9 HTTP -I eth0                             # listen on eth0, show packets containing "HTTP"
hping3 -S <target> -a <Spoof_IP> -p <port> --flood # SYN flood with spoofed IP (DoS)
```

## 3. Host discovery
Find which hosts are alive — usually the first scanning step.
```bash
nmap -sn -PR <target>          # ARP ping scan (default on local net)
nmap -sn -PU 53 <target>       # UDP ping to port 53 (DNS)
nmap -sn -PE <target>          # ICMP echo ping
nmap -sn -PE -PS80 <target>    # ICMP echo + SYN ping sweep
nmap -sn -PP <target>          # ICMP timestamp ping
nmap -sn -PM <target>          # ICMP address-mask ping
nmap -sn -PS80 <target>        # TCP SYN ping to port 80
nmap -sn -PA443 <target>       # TCP ACK ping to port 443 (bypass firewalls)
nmap -sn -PO <target>          # IP protocol ping
nmap -sn --disable-arp-ping -PE <target>   # disable default ARP ping, use another method
```
`-sn` disables the port scan. Nmap uses ARP ping by default on the local net — `--disable-arp-ping` to force another ping type.

**Ping sweep tools:** Angry IP Scanner, SolarWinds Engineer's Toolset, NetScanTools Pro, Colasoft Ping Tool, Advanced IP Scanner, OpUtils — send ICMP ECHO to a whole range at once.

## 4. Port and service discovery
Find open ports and the services behind them → what's exposed/exploitable.

### Well-known ports
| Port | Proto | Service | Port | Proto | Service |
|------|-------|---------|------|-------|---------|
| 7 | tcp/udp | echo | 161/162 | tcp/udp | snmp / snmp-trap |
| 9 | tcp/udp | discard | 163/164 | tcp/udp | cmip manager/agent |
| 11 | tcp | systat | 194 | tcp/udp | irc |
| 13 | tcp/udp | daytime | 201–208 | tcp/udp | appletalk (at-*) |
| 15 | tcp/udp | netstat | 213 | tcp/udp | ipx (novell) |
| 17 | tcp/udp | qotd | 220 | tcp/udp | imap3 |
| 19 | tcp/udp | chargen | 387 | tcp/udp | aurp (appletalk) |
| 20 | tcp | ftp-data | 396 | tcp/udp | netware-ip |
| 21 | tcp | ftp (command) | 411 | tcp/udp | rmt |
| 22 | tcp | ssh | 443 | tcp | www-https |
| 23 | tcp | telnet | 445 | tcp/udp | microsoft-ds / kerberos-ds |
| 25 | tcp | smtp | 500 | udp | isakmp/ike |
| 37 | tcp/udp | time | 510 | tcp | fcp (first class) |
| 39 | tcp/udp | rlp | 512 | tcp/udp | exec / comsat-biff |
| 53 | tcp/udp | domain (DNS) | 513 | tcp/udp | login-rlogin / who |
| 66 | tcp/udp | Oracle SQL*net | 514 | tcp/udp | shell-rsh / syslog |
| 67/68 | udp | bootp server/client | 515 | tcp/udp | printer (lpd) |
| 69 | udp | tftp | 517/518 | udp | talk / ntalk |
| 70 | tcp | gopher | 532 | tcp/udp | netnews |
| 79 | tcp | finger | 540 | tcp/udp | uucp |
| 80 | tcp/udp | www-http | 543/544/545 | tcp | klogin / kshell / ekshell |
| 88 | tcp/udp | kerberos | 600 | tcp | pcserver |
| 109/110 | tcp | pop2 / pop3 | 635/640/650 | udp | nfs mount / pcnfs / bwnfs |
| 111 | tcp/udp | sunrpc portmapper | 744 | tcp/udp | flexlm |
| 113 | tcp/udp | auth/ident | 749–754 | tcp/udp | kerberos admin/master/prop |
| 114 | tcp/udp | audionews | 999 | udp | applix |
| 119 | tcp | nntp (usenet) | 1080 | tcp/udp | socks proxy |
| 123 | udp | ntp | 1109 | tcp | kpop |
| 137/138/139 | tcp/udp | netbios ns/dgm/ssn | 1433/1434 | tcp/udp | ms-sql-s / ms-sql-m |
| 143 | tcp/udp | imap | 1723 | tcp/udp | pptp |
| 150 | tcp/udp | sql-net | 2049 | tcp/udp | nfs |
| 156 | tcp/udp | sqlsrv | 2105–2120 | tcp | kerberos rlogin/kinit/kx/kauth |
| | | | 4894 | tcp | lyskom |
| | | | 5060 | tcp/udp | sip |
| | | | 6000–6063 | tcp/udp | x11 |
| | | | 6667 | tcp | irc |

### Nmap scan types
```bash
nmap -sT <target>              # TCP Connect / full-open (full 3-way handshake)
nmap -sS <target>              # Stealth / half-open (SYN, no full handshake)
nmap -sN <target>              # TCP Null scan
nmap -sF <target>              # FIN scan
nmap -sX <target>              # Xmas scan (FIN+URG+PSH)
nmap -sM <target>              # Maimon scan (FIN/ACK)
nmap -sA <target>              # ACK flag probe — map firewall rulesets
nmap -sA --ttl 128 <target>    # TTL-based ACK probe — filtered vs unfiltered
nmap -sA --reason <target>     # window-based ACK probe — stateful vs stateless
nmap -sI <zombie_ip> <target>  # IDLE/IPID scan — via zombie host (stealth)
nmap -sU <target>              # UDP scan
nmap -sY <target>              # SCTP INIT scan
nmap -sZ <target>              # SCTP COOKIE-ECHO scan (stealthier)
nmap -sL --script=ssdp-discover <target>   # list scan + SSDP (IoT enum)
nmap -6 <ipv6-address>         # IPv6 scan
nmap -sV <target>              # service version detection
```

## 5. OS discovery (banner grabbing / OS fingerprinting)
- **Banner grabbing** — read service banners (FTP/HTTP/SSH) that leak OS name/version.
- **OS fingerprinting** — analyze how the stack responds to specific traffic to guess the OS.
  - **Active** — send crafted/unusual packets and compare responses to a known DB.
  - **Passive** — sniff existing traffic (no packets sent), infer OS from stack quirks — quiet, no alert.

**TTL / TCP window size** in the first returned packet hint at OS — e.g. **TTL 128 ≈ Windows, TTL 64 ≈ Linux**. Read these in Wireshark (https://www.wireshark.org) from the first TCP packet the target sends back.

```bash
nmap -O <target>                 # OS detection
nmap -sC <target>                # run default NSE scripts
nmap -6 -O <target>              # OS discovery via IPv6 fingerprinting
nmap --script <script> <target>  # custom NSE script
unicornscan -Iv <target_ip>      # Unicornscan
```

## 6. Scanning beyond IDS and firewall
Techniques to get probe packets to the target while evading IDS/firewall:

- **Packet fragmentation** — split the TCP header across several small fragments so filters can't reassemble intent.
  ```bash
  nmap -sS -T4 -A -f -v <target>
  ```
- **Source routing** — specify a partial/full route that avoids IDS/firewall-configured routers.
- **Source port manipulation** — use a common allowed port (HTTP/DNS/FTP) as source.
  ```bash
  nmap -g 80 <target>
  nmap --source-port 21 <target>
  ```
- **IP address decoy** — mix real scan with decoy source IPs so the IDS can't tell who's really scanning.
  ```bash
  nmap -D RND:10 <target>                    # random decoys
  nmap -D decoy1,decoy2,decoy3,... <target>  # manual decoys
  ```
- **IP address spoofing** — forge the source IP (replies go to the spoofed host).
  ```bash
  hping3 <target> -a <spoofed_IP>
  ```
- **MAC address spoofing** — impersonate a legitimate host's MAC.
  ```bash
  nmap -sT -Pn --spoof-mac 0 <target>         # random MAC
  nmap -sT -Pn --spoof-mac <Vendor> <target>  # random MAC for a vendor (DELL/HP/CISCO)
  nmap -sT -Pn --spoof-mac <new MAC> <target> # set a specific MAC
  ```
- **Custom packets** — craft TCP packets with tools like **Colasoft Packet Builder** (Packet List / Decode Editor / Hex Editor views) or NetScanTools Pro.
- **Randomize host order:** `nmap --randomize-hosts <target>`
- **Bad checksums:** `nmap --badsum <target>` — bogus TCP/UDP checksums slip past some rulesets.

### Proxies & anonymizers (hide source)
- **Proxy tools:** Proxy Switcher, CyberGhost VPN, Burp Suite, Tor, Hotspot Shield, Proxifier, IPRoyal Residential Proxy — chain SOCKS/HTTP proxies to hide IP.
- **Anonymizers:** Whonix (Tor-routed Debian VMs), Psiphon, TunnelBear, I2P, Bright Data Proxy API — SSH/VPN/HTTP proxying, encrypt traffic from ISP.
- **Censorship circumvention:** AstrillVPN (no traffic/DNS logging), Tails (live amnesic OS from USB/SD, leaves no trace).

## 7. Scanning detection & prevention tools
ExtraHop · Splunk Enterprise Security · Scanlogd (github.com/openwall/scanlogd) · Vectra Detect · IBM Security QRadar XDR · Cynet 360 AutoXDR.

> Flow: host discovery (`-sn` ping scans) → port/service scan (`-sS`/`-sV`, pick scan type by firewall behavior) → OS fingerprint (`-O`, TTL/window in Wireshark) → if IDS/firewall in the way, layer evasion (fragment, decoy, source-port, spoof-mac, proxies). Active scanning is noisy — only in scope.
