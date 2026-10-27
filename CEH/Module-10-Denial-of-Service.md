# Module 10 – Denial-of-Service

> Status: ✅ Reference complete

## 1. Goal
Make a system/network/service unavailable to legitimate users by exhausting resources (bandwidth, CPU, disk, data structures, connection tables) — not to steal data or gain access. Effects: resource/bandwidth consumption, physical destruction, file destruction.

- **DoS** — single source floods the victim with illegitimate requests/traffic.
- **DDoS** — distributed: attacker controls many compromised systems (**botnet** of zombies) via a **C&C server**; traffic from many sources is hard to trace. Zombies may send requests with the victim's spoofed IP to **reflectors**, which reply to the victim (flood).

**Case studies:**
- **HOIC (High Orbit Ion Cannon)** — hacker hosts the tool, recruits "volunteers" via IRC/social media, commands them to flood a target (PayPal/MasterCard). Hackers advertise botnet download links; Android devices (malicious APKs) become mobile botnets.
- **HTTP/2 Rapid Reset (Sept 2023, Google Cloud)** — 398M rps (7.5× prior record); rapidly create and reset many HTTP/2 streams over one TCP connection to overload servers. **CVE-2023-44487** (CVSS 7.5).

## 2. Botnets
Networks of bot-infected systems controlled remotely. Small botnets still generate powerful attacks. Examples: Cardinal, Sopel, Eggdrop (IRC). **Organized cybercrime chart:** Boss → Underboss (C&C + crimeware) → Campaign Managers → Resellers.

**Botnet uses:** DDoS, spamming, traffic sniffing, keylogging, spreading malware, click fraud, credential stuffing, crypto mining, poll/game manipulation, mass identity theft.

**Scanning methods to find vulnerable machines:**
- **Random scanning** — infected host scans random IPs (fast at first, heavy traffic).
- **Hit-list scanning** — pre-built target list, split between infected hosts (rapid growth).
- **Topological scanning** — use info on the infected host (browser history, URLs) to find targets.
- **Local subnet scanning** — find vulnerable machines behind the same firewall.
- **Permutation scanning** — shared shuffled IP list (32-bit block cipher + key); avoids reinfection, high speed.

**Malicious-code propagation:**
- **Central source** — toolkit on a central server, pushed to victims (HTTP/FTP/RPC).
- **Back-chaining** — tools on the attacker's machine; victim connects back to fetch them (TFTP).
- **Autonomous** — infected host transfers tools itself at the moment of break-in (fastest).

## 3. Attack techniques

### Attack vectors
- **Volumetric** (measured in **bps**) — saturate bandwidth. Abuse NTP/DNS/SSDP. Sub-types: *flood* (zombies send bulk traffic) and *amplification* (messages to a broadcast IP amplify traffic). Examples: UDP flood, ICMP flood, Ping of Death, Smurf, pulse wave, NTP amplification, spoofed/malformed IP packet flood, zero-day.
- **Protocol** (measured in **pps / cps**) — exhaust connection-state tables (firewalls, load balancers, servers). Examples: SYN flood, fragmentation, spoofed session flood, ACK flood, SYN-ACK flood, TCP SACK panic, RST, TCP state exhaustion, TCP connection flood, ACK & PUSH ACK flood.
- **Application layer** (measured in **rps**) — target app code/bugs; low traffic, hard to detect. Examples: HTTP flood, Slowloris, UDP application-layer flood, DDoS extortion.

### Techniques
- **UDP flood** — flood random ports with spoofed UDP; server replies with ICMP unreachable, wasting resources.
- **ICMP flood** — mass ICMP echo requests. Defense: threshold ~1000 pps, then block.
- **Ping of Death** — oversized/malformed ping (> 65,535 bytes) crashes on reassembly.
- **Smurf** — spoof victim's IP, send ICMP ECHO to a broadcast address → all hosts reply to the victim.
- **Pulse wave** — massive bursts (300 Gbps+) at regular intervals (~every 10 min), for hours/days.
- **Zero-day DDoS** — exploit unknown, unpatched vulnerabilities.
- **NTP amplification** — spoofed `monlist` requests to NTP servers → huge replies to the victim.
  ```bash
  nmap -sU -pU:123 -Pn -n --script=ntp-monlist <target>   # retrieve monlist
  ```
  Defense: harden NTP config (disable monlist), flow control, monitor, zero-trust, firewall NTP requests.
- **SYN flood** — send SYN with spoofed IP, never send final ACK → half-open connections fill the queue. Defense: SYN cookies, SynAttackProtect, lower SYN-RECEIVED timeout, limit/disable retransmissions.
- **SYN-ACK flood** — flood SYN-ACK packets (abuse handshake stage 2).
- **ACK & PUSH ACK flood** — spoofed ACK/PUSH ACK packets exhaust the target.
- **Fragmentation** — large fragmented packets (> 1500 bytes) pass firewalls/IDS, drain memory on reassembly.
- **Spoofed session flood** — fake TCP sessions (SYN/ACK/RST/FIN) bypass firewalls. Variants: multiple SYN-ACK, multiple ACK (skips SYN → evades SYN-filtering firewalls).
- **HTTP GET/POST** (Layer 7) — GET: delayed/incomplete headers; POST: partial body → server waits, resources exhausted. Variants: single-session, single-request, recursive GET, random recursive GET.
- **Slowloris** — incomplete HTTP requests keep connections open until the pool is exhausted.
- **UDP application-layer flood** — abuse CHARGEN, SNMPv2, QOTD, RPC, SSDP, CLDAP, TFTP, NetBIOS, NTP, Quake, Steam, VoIP.
- **Multi-vector** — combine volumetric + protocol + application, switching vectors to confuse defenders.
- **Peer-to-peer (P2P)** — abuse DC++/P2P networks to redirect many users to the victim (no botnet). Defense: block P2P on port 80.
- **Permanent DoS (PDoS / phlashing)** — push corrupted firmware updates to "brick" hardware (routers/printers).
- **TCP SACK panic** — crafted SACK packets with tiny MSS overflow the Linux socket buffer (>17 segments) → kernel panic. Defense: patch, firewall-block lowest-MSS packets.
- **DRDoS (reflection)** — zombies send spoofed requests (victim's IP) to reflectors → reflectors flood the victim with SYN/ACK; hard to trace. Defense: disable CHARGEN, patch servers.
- **DDoS extortion / Ransom DDoS (RDDoS)** — threaten a DDoS unless ransom (crypto) is paid. Defense: DDoS defense tools, report to LEA, risk assessment, BGP/DNS swing + always-on protection.

**Tools:** ISB ("I'm so bored"), UltraDDOS-v2, HOIC, LOIC, HULK, Slowloris, UFONet, Packet Flooder Tool.

## 4. Countermeasures

### Detection techniques
- **Activity profiling** — monitor average packet rate per flow; spikes/entropy increases signal attack.
- **Sequential change-point detection** — CUSUM algorithm flags sudden flow-rate changes (also catches worm scanning).
- **Wavelet-based signal analysis** — decompose traffic into frequency bands; attacks spike high-frequency components.

### Strategies
Absorbing (add bandwidth/capacity), degrading services (keep only critical), shutting down services (last resort).

### DDoS countermeasures
- **Protect secondary victims** — users keep AV/anti-Trojan updated, patch, disable unused services; providers use dynamic pricing to encourage responsibility.
- **Detect & neutralize handlers** — fewer handlers than agents; take them down, spot spoofed source IPs.
- **Prevent attacks:** *egress filtering* (block spoofed packets leaving), *ingress filtering* (ISP-level, block spoofed sources), *TCP intercept* (router verifies connections before the server), *rate limiting*.
- **Deflect with honeypots** — low-interaction (basic services) vs high-interaction (honeynets); Blumira, KFSensor, Cowrie, StingBox.
- **Mitigate:** load balancing (replicated servers), throttling (smart routers slow traffic), drop requests (puzzles for senders).
- **Post-attack forensics:** traffic-pattern analysis, packet traceback, event-log analysis.

### Botnet defense
1. **RFC 3704 filtering** — drop packets from bogon (unused) IPs.
2. **Cisco IPS source-IP reputation filtering** — block known-bad IPs.
3. **Black-hole filtering** — null-route bad traffic (with ISP).
4. **ISP/cloud DDoS prevention** — clean traffic in the cloud; IP Source Guard blocks forged IPs.

### Additional
Keep software updated, SYN cookies, block malicious inbound ports, firewall-block ICMP, strong encryption (WPA3/AES-256), AI/ML anomaly detection, input validation, third-party protection (Cloudflare/Akamai), limit broadcasting, run DoS/DDoS simulations.

### ISP level & Cisco TCP Intercept
ISPs offer "clean pipes"; cloud services (Imperva, VeriSign) filter and forward safe traffic.
```text
access-list <n> {deny | permit} tcp any <dest> <dest-wildcard>   # 1. define extended ACL
ip tcp intercept list <n>                                        # 2. enable TCP intercept
ip tcp intercept mode {intercept | watch}                        # set mode
```
Modes: *active intercept* (default — router answers the client, forwards only verified connections), *passive watch* (monitors; resets connections not completed in 30s).

### Appliances & services
Appliances: FortiDDoS, Check Point Quantum DDoS Protector, Huawei AntiDDoS1000, A10 Thunder TPS. Tools: Anti DDoS Guardian, DDoS-GUARD, Radware DefensePro X, F5. Services: Cloudflare, Akamai, Stormwall PRO, Imperva, Nexusguard.

> Flow: attacker builds a botnet (scan → propagate) → launches volumetric (bps), protocol (pps), or app-layer (rps) floods, often multi-vector. Defend in layers: ingress/egress filtering + SYN cookies + TCP intercept + rate limiting at the edge, honeypots to deflect, and cloud/ISP scrubbing for volume beyond local capacity.
