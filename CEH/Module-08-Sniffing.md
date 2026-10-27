# Module 08 – Sniffing

> Status: ✅ Reference complete

## 1. Goal
Capture and monitor network packets to steal sensitive data (passwords, emails, cards). Easy on hub/shared networks (broadcast); on switched networks requires promiscuous mode + active techniques.

### How a sniffer works
Ethernet uses MAC + IP. A host resolves a destination MAC via its **ARP cache** or an ARP broadcast. **Shared Ethernet** (hub) — all devices see all traffic; sniffing is trivial and stealthy. **Switched Ethernet** — switch forwards by MAC table, so sniffing needs:
- **ARP spoofing** — send fake ARP replies so the victim believes the attacker is the gateway → MITM.
- **MAC flooding** — flood the CAM table (macof) → switch fails open and broadcasts to all ports → NIC in **promiscuous mode** captures everything (Wireshark/tcpdump).

### Types
- **Passive sniffing** — silently capture, no injection; works on hubs/shared networks; stealthy.
- **Active sniffing** — inject traffic (ARP) on switched networks; harder, easier to detect.

### Attack steps
1. Connect to a switch port → 2. map topology (discovery tools) → 3. identify victim → 4. ARP spoof → 5. divert victim traffic to attacker (MITM) → 6. extract creds/PINs from packets.

### Protocols vulnerable to sniffing (plaintext)
Telnet & Rlogin, HTTP, SMTP, NNTP, POP, FTP, IMAP, TFTP — all send credentials unencrypted.

### Other concepts
- Sniffing happens at the **Data Link Layer (Layer 2)** — higher layers don't detect it.
- **Hardware protocol analyzers** — physical devices (Xgig 1000, SierraNet M1288) capture at line rate with timestamps; costly.
- **SPAN port** (port mirroring) — Cisco feature copying source port/VLAN traffic to one destination port for an analyzer.
- **Wiretapping** — secret monitoring of comms. *Active* = MITM (monitor + alter/inject); *passive* = eavesdrop only. Illegal without warrant/consent.
- **Lawful Interception (LI)** — legal LEA monitoring via ISPs; tap/access switch → E-Detective (ED) decodes POP3/SMTP/FTP/Telnet → Centralized Management Server (CMS).

## 2. Sniffing tools & Wireshark
Tools: **Wireshark** (WinPcap; Ethernet/802.11/Bluetooth/USB…), Capsa, OmniPeek, RITA, Observer Analyzer, PRTG, Xplico.

**Follow TCP Stream** — Analyze → Follow → TCP Stream: shows the full conversation (ASCII/hex/C-array/raw) — pull creds from telnet, see full data exchanges.

**Display filters:**
```text
arp  http  tcp  udp  dns  ip                     # by protocol
tcp.port==23                                      # by port
ip.addr==192.168.1.100 && tcp.port==23
ip.addr == 10.0.0.4 or ip.addr == 10.0.0.5        # multiple IPs
ip.src==205.153.63.30 or ip.dst==205.153.63.30
tcp.flags.reset == 1                              # TCP resets
udp contains 33:27:58                             # hex payload match
http.request                                      # HTTP GET requests
tcp.analysis.retransmission                       # retransmissions
tcp contains traffic                              # keyword in TCP
!(arp or icmp or dns)                             # exclude noise
tcp.port eq 25 or icmp                            # SMTP + ping
ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16 # LAN only
ip.src != 10.10.10.1 && ip.dst != 10.10.10.1 && sip  # SIP, excluding an IP
```

## 3. MAC attacks
**MAC address** — 48-bit (12 hex), first 6 = manufacturer OUI, last 6 = device serial. **CAM table** — switch stores MAC↔port↔VLAN; forwards only to the right port. CAM has limited memory.

### MAC flooding (macof)
Flood CAM with fake MACs (macof, dsniff suite — up to 131,000/min) → table full → switch **fails open** (acts like a hub, broadcasts to all ports) → attacker sniffs in promiscuous mode.

### Switch port stealing
Flood the switch with fake ARP packets using the victim's MAC → switch keeps re-learning the victim's MAC on the attacker's port → attacker temporarily "steals" the port and receives the victim's traffic.

### Defend — Port Security (Cisco)
```text
Switch> configure terminal
Switch(config)> interface gigabitethernet 3/1
Switch(config-if)> switchport mode access
Switch(config-if)> switchport port-security
Switch(config-if)> switchport port-security maximum <value>        # 1–3072, default 1
Switch(config-if)> switchport port-security violation {restrict | shutdown}
Switch(config-if)> switchport port-security mac-address <mac>      # static
Switch(config-if)> switchport port-security mac-address sticky     # auto-learn
Switch(config-if)> switchport port-security aging time 2
Switch(config-if)> switchport port-security aging type inactivity
Switch(config-if)> end
Switch> show port-security address
```
Violation triggers: MAC limit reached, or unknown MAC connects → switch blocks/locks port + SNMP alert.

## 4. DHCP attacks
**How DHCP works:** client broadcasts DHCPDISCOVER → server DHCPOFFER → client DHCPREQUEST → server DHCPACK (DORA). DHCPv6: Solicit/Advertise/Request/Reply. Other messages: Release, Decline, Inform, NAK, Reconfigure, Relay-Forward/Reply.

**IPv4 DHCP packet fields:** Opcode (1=client, 2=server), hardware type/length, hops, XID (transaction ID), seconds, flags, CIADDR, YIADDR (assigned address), SIADDR, GIADDR (relay agent), CHADDR (client MAC), SNAME, File, DHCP Options.

### DHCP starvation
Flood the server with DHCP requests using spoofed MACs (Yersinia, Hyenae, Gobbler, DHCPig) → exhaust the IP pool → DoS (legit users can't get leases).

### Rogue DHCP server
After starving the real server, stand up a fake DHCP server → clients accept its config → attacker becomes default gateway → MITM/redirect/DoS.

### Defend
- **Starvation → port security** (limit MACs per port):
  ```text
  Switch(config)# interface fastEthernet 0/1
  Switch(config-if)# switchport port-security
  Switch(config-if)# switchport port-security maximum 1
  Switch(config-if)# switchport port-security violation restrict
  Switch(config-if)# switchport port-security mac-address sticky
  ```
- **Rogue server → DHCP snooping** (only the trusted port answers DHCP):
  ```text
  Switch(config)# ip dhcp snooping
  Switch(config)# ip dhcp snooping vlan 4,104
  Switch(config)# interface GigabitEthernet0/1
  Switch(config-if)# ip dhcp snooping trust               # uplink to real server
  Switch(config)# interface range GigabitEthernet0/2 - 24
  Switch(config-if-range)# ip dhcp snooping limit rate 10
  Switch# show ip dhcp snooping
  ```
  All ports untrusted by default. Also: Juniper MAC limiting (`set interface ge-0/0/1 mac-limit 3 action drop`), DHCP filtering.

## 5. ARP poisoning
**ARP** maps IP→MAC; it's stateless and trusts any reply (even unsolicited). **ARP spoofing** = send forged ARP replies → victim's ARP cache poisoned → attacker intercepts/alters traffic (cache poisoning → MITM/DoS/session hijacking).

**Threats:** packet sniffing, session hijacking, VoIP tapping, data manipulation, MITM, data interception, connection hijacking/resetting, password theft, DoS.

**Tools:** arpspoof (`arpspoof -i <iface> -t <target>` — reply both directions for MITM), Habu, bettercap, Ettercap, RITM, ARP Spoofer.

### Defend — Dynamic ARP Inspection (DAI)
DAI validates ARP packets against the DHCP snooping binding table. **DHCP snooping must be enabled first** (static IPs need static IP-MAC mappings).
```text
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config)# interface FastEthernet0/5
Switch(config-if)# ip dhcp snooping trust
Switch# show ip dhcp snooping binding
Switch(config)# ip arp inspection vlan 10
Switch# show ip arp inspection
```
Detection tools: Capsa, Wireshark, OpUtils, XArp, ARP-GUARD.

## 6. Spoofing attacks
### MAC spoofing/duplicating
Copy an active user's MAC to impersonate them (bypasses AP MAC filtering). **Windows** — NIC "Network Address" in adapter Advanced properties, or registry `HKLM\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-...}` → add `NetworkAddress` REG_SZ. Tools: SMAC, TMAC, macchanger, AMC.

### IRDP spoofing
Send fake ICMP Router Discovery advertisements (no auth) with high preference/lifetime → victim uses attacker as default route → sniff/MITM/DoS. Defense: disable IRDP.

### VLAN hopping
- **Switch spoofing** — attacker's rogue switch negotiates a trunk link (works if the real port is dynamic auto/desirable/trunk) → sees all VLANs.
- **Double tagging** — frame with two VLAN tags; first switch strips the outer (native) tag, inner tag delivers to target VLAN.

**Defend:** access ports `switchport mode access` + `switchport mode nonegotiate`; for double tagging set native VLAN to an unused ID (`switchport trunk native vlan 999`) + `vlan dot1q tag native`; private VLANs; audits.

### STP attack
Inject BPDUs with a very low priority → rogue switch becomes **root bridge** → all traffic routes through attacker. **Defend:**
```text
spanning-tree portfast bpduguard      # BPDU Guard — errdisable port on BPDU
spanning-tree guard root              # Root Guard — loop-inconsistent state on superior BPDU
spanning-tree guard loop              # Loop Guard
udld { enable | disable | aggressive }  # Unidirectional Link Detection
```

### Defend MAC spoofing (general)
Servers behind routers (route by IP not MAC), port security, DHCP snooping, DAI, IP Source Guard, 802.1X, AAA/NAC, WPA3, rate limiting, audits.

## 7. DNS poisoning
Manipulate DNS so the victim resolves a domain to the attacker's IP → fake site. Techniques:
- **Intranet DNS spoofing** — on the LAN, ARP-poison + sniff DNS requests, reply with a fake IP before the real server (arpspoof + dnsspoof).
- **Internet DNS spoofing** — Trojan changes the victim's DNS to a rogue server.
- **Proxy server DNS poisoning** — Trojan changes the victim's proxy settings to a malicious proxy.
- **DNS cache poisoning** — inject fake records into a resolver's cache. **SAD DNS** = modern variant exploiting side-channel flaws in dnsmasq/unbound/BIND.

Tools: DerpNSpoof, deserter, PolarDNS, Ettercap, Evilgrade, DNS Goisoner.

**Defend:** DNSSEC, SSL/TLS, resolve to a local DNS server, block external DNS, firewall external lookups, IDS, random source port per query, restrict recursion, NXDOMAIN rate limiting.

> Flow: get onto the switch → turn the switch into a hub (macof) or MITM a target (ARP spoof) → capture in Wireshark, Follow TCP Stream for creds → or poison DHCP/DNS to redirect. Defenses stack at Layer 2: port security → DHCP snooping → DAI → IP Source Guard, plus DNSSEC and encryption so sniffed traffic is useless.
