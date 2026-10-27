# Module 16 – Hacking Wireless Networks

> Status: ✅ Reference complete

## 1. Wireless concepts
Wireless networking uses radio-frequency (electromagnetic) signals instead of cables to connect devices — mobile, portable, flexible. Wi-Fi (IEEE 802.11) is the most common; operates at the physical layer using DSSS, FHSS, infrared, and OFDM.

### Terminology
- **GSM** — worldwide mobile-communication standard (calls, texts, mobile data).
- **Bandwidth** — max data per unit time (bps); higher = faster.
- **Access Point (AP)** — bridges wireless devices to a wired network (entry point to a WLAN).
- **BSSID** — the unique MAC address of a specific AP.
- **ISM band** — unlicensed radio frequencies (research/healthcare/industrial); Wi-Fi & Bluetooth use these.
- **Hotspot** — a public Wi-Fi location (café, airport, library).
- **Association** — the process of a device formally connecting to an AP (authenticate + exchange credentials).
- **SSID** — the network name; case-sensitive, up to 32 chars; shared identifier between APs and clients.
- **OFDM** — splits data across many non-interfering frequencies (high-speed; Wi-Fi/4G/5G).
- **MIMO-OFDM** — multiple antennas + OFDM (4G LTE, 5G backbone).
- **DSSS** — spreads the signal over a wide band with a pseudo-random code (interference-resistant).
- **FHSS** — rapidly hops between channels in a pseudo-random order (hard to intercept/jam; Bluetooth).

### Advantages / disadvantages
- **+** fast/easy install (no wiring), connectivity where cabling is hard, access anywhere in AP range, public WLANs.
- **–** security may fall short, bandwidth drops as devices increase, upgrades may need new cards/APs, electronic interference.

### Types of wireless networks
1. **Extension to a wired network** — APs bridge wireless clients to a wired LAN. *Software APs (SAPs)* run on a PC with a wireless NIC; *Hardware APs (HAPs)* support most wireless features. The AP acts as a switch.
2. **Multiple access points** — overlapping AP ranges enable roaming; extension points act as wireless relays.
3. **LAN-to-LAN wireless** — APs interconnect two+ LANs over wireless links (complex to configure/secure).
4. **3G/4G/5G hotspot** — provides Wi-Fi to enabled devices (tablets, cameras, netbooks).

### Wireless standards (802.11 family)
| Amendment | Frequency (GHz) | Modulation | Speed (Mbps) | Range (m) |
|-----------|-----------------|------------|--------------|-----------|
| 802.11 (Wi-Fi) | 2.4 | DSSS, FHSS | 1, 2 | 20–100 |
| 802.11a | 5 / 3.7 | OFDM | 6–54 | 35–100 / 5000 |
| 802.11b | 2.4 | DSSS | 1, 2, 5.5, 11 | 35–140 |
| 802.11g | 2.4 | OFDM | 6–54 | 38–140 |
| 802.11n | 2.4, 5 | MIMO-OFDM | 54–600 | 70–250 |
| 802.11ac (Wi-Fi 5) | 5 | — | — | — |
| 802.11ax (Wi-Fi 6) | 2.4–5 | 1024-QAM / OFDMA | 2400 (up to 9.6 Gbps) | 240 |
| 802.11be (Wi-Fi 7) | 2.4, 5, 6 | QAM | 3000 (up to 30 Gbps) | 120 |
| 802.15.1 (Bluetooth) | 2.4 | GFSK, π/4-DPSK, 8DPSK | 25–50 | 10–240 |
| 802.15.4 (ZigBee) | 0.868, 0.915, 2.4 | O-QPSK, GFSK, BPSK | 0.02, 0.04, 0.25 | 1–100 |
| 802.16 (WiMAX) | 2–11 | SOFDMA | 34–1000 | 1–6 miles |

Non-rate amendments: **802.11d** (regulatory domains/global portability), **802.11e** (QoS for voice/VoIP/video at MAC layer), **802.11i** (TKIP + AES; defines WPA2-Enterprise/Personal), **802.11ah** (Wi-Fi HaLow, 900 MHz, IoT long range), **802.11ad** (60 GHz, very high speed, short range), **802.12** (demand priority, 100 Mbps), **802.15** (WPAN), **802.15.5** (mesh), **802.16** (WiMAX MAN, point-to-multipoint).

### SSID
Case-sensitive, ≤32 alphanumeric chars; in the frame header of packets. **Not a security mechanism** — transmitted in plaintext in probe requests/responses, easily sniffed; hidden SSIDs are revealed once clients connect; blank/"any" SSID configs are vulnerable. Use WPA2/WPA3, not SSID hiding, for security. Default vendor-name SSIDs are a risk.

### Wi-Fi authentication process
- **PSK mode** (WPA/WPA2-PSK) — single shared password on router + device; simple (home/small office); security depends entirely on password strength.
- **Centralized mode** (WPA/WPA2-Enterprise, 802.1X) — a **RADIUS** server authenticates each user individually (username/password/certificate); strong, for large networks.

### Wireless antennas
- **Directional** — transmits/receives from one direction (better range, less interference).
- **Omnidirectional** — 360° horizontal radiation (radio-station style).
- **Parabolic grid** — wire-grid dish, highly focused beams up to ~10 miles; captures weak signals → more data/bandwidth/power for Layer-1 DoS or MITM.
- **Yagi (Yagi-Uda)** — unidirectional, 10 MHz–UHF/VHF, high gain; reflector + dipole + directors → end-fire pattern.
- **Dipole** — half-wavelength balanced conductor (doublet).
- **Reflector** — parabolic surface focusing signals to a point (satellite comms); bigger = higher gain.

## 2. Wi-Fi discovery
First step: find a target wireless network. **Footprinting** identifies the BSS of an AP via its SSID, then attempts association.
- **Passive footprinting** — listen to airwaves without interacting (sniff packets → identify APs/SSIDs); stealthy, hard to detect.
- **Active footprinting** — send probe requests to APs; include the SSID (AP responds) or send an empty SSID (most APs reply with the real one, unless configured to ignore empty requests).

### Wi-Fi chalking
- **WarWalking** — walk around with a Wi-Fi laptop + discovery tool.
- **WarChalking** — draw symbols in public to advertise open Wi-Fi.
- **WarFlying** — use drones to find open networks.
- **WarDriving** — drive around mapping open networks.

### Discovery tools
inSSIDer (signal/channels/export GPS), Sparrow-wifi (2.4/5 GHz GUI; integrates HackRF/Ubertooth/GPS/mavlink), Wi-Fi Scanner, Acrylic WiFi Heatmaps, WirelessMon, Ekahau, NetSpot, AirMagnet Survey PRO, NetSurveyor. Mobile: WiFi Analyzer, Opensignal, Network Signal Info Pro, Net Signal Pro, WiFiman.

### Finding WPS-enabled APs (Wash)
`wash` (Reaver package) identifies WPS-enabled APs and whether they're locked (most lock after 5 wrong attempts; unlock manually in admin UI). Key args:
```
-i, --interface=<iface>   # capture interface
-a, --all                 # show all APs, even WPS-disabled
-f, --file [FILES]        # read from captured files
-c, --channel=<num>       # channel to listen
-o, --out-file=<file>     # write output
-n, --probes=<num>        # max probes per AP in scan mode
-5, --5ghz                # use 5 GHz channels
-s, --scan                # scan mode
-u, --survey              # survey mode (default)
```
```bash
sudo wash -i wlan0        # discover AP, ESSID, BSSID
```

## 3. Wireless traffic analysis
Study traffic before attacking to find insecure auth, weak encryption, misconfigured APs, and victims. Wi-Fi is Layer 2, not serialized → easy to intercept over the air. Sniffing reveals SSIDs, number of APs, hidden SSIDs (recoverable), encryption type (WEP/WPA/WPA2), and auth mechanisms. Requires **monitor mode** on a compatible card (not all cards/Windows support it; check secwiki.org Npcap list).

Tools: Wireshark (monitor mode → management/control/data frames), CommView for Wi-Fi (802.11 a/b/g/n; decrypts with WPA-PSK keys), Kismet, Riverbed Packet Analyzer, OmniPeek, AirMagnet G3 Pro, airgeddon.

### Choosing the optimal Wi-Fi card
Needs a chipset that supports **monitor mode + packet injection** (for Aircrack-ng etc.). Windows can listen only; Linux can listen and inject. Steps: determine requirements (listen/inject, PCMCIA/USB) → learn card capabilities (brand *and* chipset) → determine the chipset (internet search, Windows driver filenames, manufacturer page, view the chip, **FCC ID search**, account for card revisions) → verify chipset/OS compatibility → determine drivers/patches → pick a card from a compatible list.

### Spectrum analysis
Use spectrum analyzers to detect networks, measure signal strength, identify transmissions, and isolate sources. Tools: RF Explorer, Chanalyzer, AirCheck G3 Pro, Spectraware S1000, RSA306B, RFXpert, Signal Hound, FieldSENSE.

## 4. Launch wireless attacks

### Aircrack-ng suite
Detector, sniffer, WEP + WPA/WPA2-PSK cracker, analysis (Linux/Windows):
- **airbase-ng** — capture WPA/WPA2 handshake, act as ad-hoc AP.
- **aircrack-ng** — de facto WEP + WPA/WPA2 PSK cracker.
- **airdecap-ng** — decrypt WEP/WPA/WPA2, strip wireless headers.
- **airdrop-ng** — targeted rule-based de-authentication.
- **aireplay-ng** — gather WEP IVs and WPA handshakes (injection).
- **airgraph-ng** — client-AP relationship/probe graphs from airodump files.
- **airmon-ng** — switch managed ↔ monitor mode.
- **airodump-ng** — capture raw 802.11 frames, collect WEP IVs.
- **airolib-ng** — store/manage ESSID + password lists for WPA/WPA2.
- **airtun-ng** — virtual tunnel to monitor encrypted traffic + inject.

### Detect hidden SSIDs (aircrack-ng + mdk3)
```bash
airmon-ng start <wireless interface>      # monitor mode (airmon-ng check kill if needed)
airodump-ng <interface>                   # hidden SSID shows blank
mdk3 <interface> p -b 1 -c <channel> -t <target BSSID>   # brute-force the hidden ESSID
# p = probing/ESSID brute-force, -b = beacon flood, -c = channel, -t = target BSSID
```

### Denial-of-Service
WLANs carry VoIP/DB/internet → disruption causes downtime. Forged frames force disconnects:
- **Disassociation attack** — destroy connectivity between AP and client (target unavailable).
- **De-authentication attack** — flood stations with forged deauth/disassoc frames to disconnect users.

### Man-in-the-Middle
Active attack intercepting/reading/altering traffic between two hosts (wireless + wired).
- **Eavesdropping** — capture data frames over the air; weak with WEP (crackable); POP/IMAP in cleartext is readable. Defend: IPsec/SSL/SSH.
- **Manipulation** — receive encrypted data, alter it, retransmit; change destination address.

**MITM steps:** sniff the victim's wireless params (MAC, ESSID/BSSID, channels) → send a DEAUTH with the AP's spoofed source → victim de-authenticates and searches channels → attacker sets a forged AP on a new channel with the original BSSID/ESSID → victim associates to the forged AP → attacker spoofs the victim to the real AP → sits in the middle.

**MITM with aircrack-ng:** `airmon-ng` monitor → `airodump-ng` discover → `aireplay-ng` deauth the client → `aireplay-ng` fake association with the AP.

### MAC spoofing
- **AP MAC spoofing** — rogue AP copies a legit AP's MAC (and SSID); deauth users off the real AP so they reconnect to the fake one → intercept creds/data.
- **Client MAC spoofing** — change the device MAC to a legit user's to bypass MAC filtering (`ifconfig` as root, or Technitium MAC Address Changer / LizardSystems Change MAC Address).

### Wireless ARP poisoning
Forged ARP replies corrupt the ARP cache (ARP doesn't verify responses) → IP maps to the wrong MAC → intercept/modify/disrupt. APs act as transparent bridges, so the whole subnet is exposed. Example: spoof the victim's MAC, authenticate to AP1 (arpspoof) → routers/switches update tables → backbone traffic to the victim goes to AP1 instead of AP2.

**ARP poisoning with Ettercap:** Sniff → Unified Sniffing (set interface) → Hosts → Scan for Hosts → Hosts List → View → Connections → Targets → Current targets → MITM → ARP poisoning → "Sniff remote connections" → sniff cleartext (non-HTTPS) creds.

### Rogue APs
An unauthorized AP installed without admin management, not security-configured → backdoor into the network. Install scenarios: compact pocket AP into an Ethernet port; rogue AP over a Wi-Fi link (needs credentials); USB-based rogue AP on a network machine (shares its network access); software-based rogue AP on an embedded Wi-Fi adapter. **Deploy:** pick a location for coverage → disable SSID broadcast (silent) + management features → place behind a firewall → run for a short period.

**Rogue AP with MANA Toolkit:** edit `hostapd-mana.conf` (set interface wlan0 + BSSID/SSID "Free Internet") → edit `start-nat-simple.sh` (set `phy` = wlan0, upstream = eth0) → `bash <path>/mana-toolkit/run-mana/start-nat-simple.sh` → victim connects to the open "Free Internet" → capture with tcpdump/Wireshark. (MANA also bypasses HTTPS/HSTS.)

### Evil twin
A fake AP copying a legit SSID to trick users; tools like **KARMA** listen for probe requests and mimic trusted SSIDs. Devices auto-reconnect to familiar SSIDs with strong signals → intercept traffic/creds. **Set up a fake hotspot:** enable Internet Connection Sharing (Windows) / Internet Sharing (macOS) → broadcast the Wi-Fi + run a sniffer. (A failed login just looks like a random failure to the victim.)

### KRACK (Key Reinstallation Attack)
Exploits the WPA2 **four-way handshake** by forcing **nonce reuse** — capture the victim's ANonce, replay handshake messages. Works against WPA and WPA2, personal and enterprise, WPA-TKIP/AES-CCMP/GCMP. Steals card numbers, passwords, chats, emails, photos. Affects Android, Linux, Windows, Apple, OpenBSD, MediaTek.

### Jamming signal attack
Flood the RF spectrum with interference (looks like background noise) → devices can't transmit → DoS. 802.11 uses CSMA/CA (needs silence before transmit), so a high-gain amplifier that drowns out the AP blocks logins/disconnects users. **Wi-Fi jammers** (specs):
| Device | Range | Antennas | Bands | Working time |
|--------|-------|----------|-------|--------------|
| PCB-4510 | 50–150 m | 10 | 10 (GSM/3G/UMTS/4G LTE/Wi-Fi/GPS/5G) | 1–2 h |
| CPB-2920 | 10–40 m | 20 | 20 (CDMA/DCS/PCS/3G/4G/Wi-Fi/WiMAX/5G/GPS/Lojack/VHF/UHF) | no limit |
| CPB-2612H-5G | 20–60 m | 12 | 12 (5G/4G/GSM/DCS/3G/Wi-Fi/Lojack/UHF/VHF) | no limit |
| CPB-2080-5G | 10–40 m | 8 | 8 (5G/4G/GSM900/DCS/3G/Wi-Fi) | no limit |
| PCB-2112 | 20–50 m | 12 | 12 (CDMA/DCS/3G/Wi-Fi/4G/5G/GPS/VHF/UHF) | 60–80 min |
| PCB-1016 | 10–30 m | 16 | 16 (CDMA/DCS/PCS/3G/4G/5G/Wi-Fi/WiMAX/GPS/UHF/5G LTE) | 3 h |

### aLTEr attack (LTE/4G)
Targets LTE using **AES-CTR** (encrypted but no integrity protection). A fake tower between user and real tower manipulates data at the data-link layer → hijack sessions, spoof DNS, redirect to malicious sites. **Steps:** install a malicious tower → determine user position, relay a valid request to the real tower → real tower responds → attacker redirects the user to harmful sites. **Phases:** *information gathering* (passive — identity mapping, website fingerprinting), *attack* (active — DNS spoofing/redirection, record creds).

### Wi-Jacking attack
Steal Wi-Fi details saved in the victim's browser (router login over HTTP). **Conditions:** ≥1 active client; client previously joined an open network with auto-reconnect; Chromium-based browser; browser stores the router admin creds; router uses unencrypted HTTP for config. **Steps:** deauth the victim (aireplay-ng) → KARMA attack (hostapd-wpe) → inject a malicious URL (dnsmasq + Python) matching the router admin origin → victim's router updates/restarts → browser auto-loads stored creds if same-origin + matching fields → keep the victim on the page, stop KARMA → use XMLHttpRequest to log in to the router and extract the WPA2 PSK.

### RFID cloning
Capture a legit RFID tag's data and clone it to a new chip (change the TID, keep form factor/data). Tools: iCopy-X (standalone Proxmark-like), RFIDler, RFID Mifare Cloner, Flipper Zero, Boscloner Pro.

## 5. Wireless encryption

### Standards overview
- **802.11i** — security mechanisms amendment for 802.11.
- **WEP** — old RC4 standard; easily cracked.
- **EAP** — supports multiple auth methods (token cards, Kerberos, certificates). **LEAP** — Cisco's proprietary EAP. **PEAP** — EAP inside an encrypted TLS tunnel.
- **WPA** — TKIP + MIC; 48-bit IV, 32-bit CRC.
- **TKIP** — WPA's replacement for WEP (per-packet key, Michael MIC, rekeys every 10,000 packets).
- **WPA2** — AES + CCMP. **WPA2-Enterprise** — EAP + WPA2.
- **AES** — symmetric key in WPA2 (replaces TKIP). **CCMP** — WPA2 encryption/auth protocol.
- **RADIUS** — centralized auth/authorization.
- **WPA3** — GCMP-256 encryption + HMAC-SHA-384 auth.

### WEP
RC4 stream cipher at the data-link layer; goal = wired-equivalent confidentiality. Keys: 64-bit (40-bit key), 128-bit (104-bit key), 256-bit (232-bit key). **How it works:** CRC-32 ICV (integrity) → 24-bit IV + secret key = seed → RC4 keystream → keystream XOR (data+ICV) = ciphertext → IV + PAD + KID prepended → MAC frame. **Flaws:** no key-distribution method (PSKs rarely changed); RC4 reused with the same key; vulnerable key-scheduling; crack with Fern Wifi Cracker / WEP-key-break.

### WPA
TKIP + 128-bit keys + per-packet key mixing + rekeying + MIC (Michael); PSK and enterprise modes. **Temporal keys (TK)** derived from the PMK (WEP) or generated in the four-way handshake (WPA/WPA2). **TK install:** AP sends ANonce → client builds PTK → client sends SNonce + MIC → AP sends GTK + sequence + MIC → client confirms. **How WPA works:** TK + transmit address + TSC → RC4 keystream; MSDU + MIC via Michael → fragmented to MPDU → 32-bit ICV → XOR keystream → + IV → MAC frame.

### WPA2
802.11i; **AES-CCMP**. Modes: **WPA2-Personal** (PSK from 8–63-char passphrase + SSID → 128-bit keys), **WPA2-Enterprise** (EAP/RADIUS, unique keys per system). **How it works:** generate AAD from the MAC header (not encrypted, tamper-protected) → Packet Number (PN) increments per frame → PN + MAC header = Nonce → AES-CCMP(temporal keys, plaintext, AAD) → MIC (encrypted) → combine encrypted data + MIC + MAC header + CCMP header → MAC frame. PN prevents replay.

### WPA3
2018; WPA3-Personal + WPA3-Enterprise; AES, Protected Management Frames (PMF), no weak legacy protocols.
- **WPA3-Personal** — **SAE (Simultaneous Authentication of Equals / Dragonfly)** replaces PSK: resistant to offline dictionary attacks, key recovery (forward secrecy), allows natural passwords, easy accessibility.
- **WPA3-Enterprise** — authenticated encryption **GCMP-256**; key derivation/validation **HMAC-SHA-384**; key establishment **ECDH + ECDSA** (384-bit curve); frame protection **BIP-GMAC-256**; 192-bit+ session keys.
- **Enhancements:** mandatory SAE/Dragonfly handshake; **Wi-Fi Easy Connect** (DPP, QR-code onboarding for IoT); **OWE** (Opportunistic Wireless Encryption — replaces open auth on public hotspots); bigger session keys.

### Comparison
| Encryption | Algorithm | IV size | Key length | Key management | Integrity check |
|-----------|-----------|---------|------------|----------------|-----------------|
| WEP | RC4 | 24-bit | 40/104-bit | None | CRC-32 |
| WPA | RC4, TKIP | 48-bit | 128-bit | 4-way handshake | Michael + CRC-32 |
| WPA2 | AES-CCMP | 48-bit | 128-bit | 4-way handshake | CBC-MAC |
| WPA3 | AES-GCMP 256 | arbitrary (1–2^64) | 192-bit | ECDH + ECDSA | BIP-GMAC-256 |

### Issues
- **WEP:** CRC-32 doesn't ensure integrity (bit-flipping); 24-bit IV too small (exhausts in ~5 h at 11 Mbps); known-plaintext (IV collisions reconstruct keystream); dictionary attacks; DoS (unauthenticated assoc/disassoc); reconstructable decryption table (~24 GB); no central key management; no unique-IV requirement; RC4 one-time-cipher misuse; shared key; no replay protection; **FMS attack** (RC4 key-scheduling weakness, statistical key recovery); one-way auth only. **Weak-IV causes:** predictable IVs, IV reuse, FMS risk, key leakage, PRGA emulation, weak CRC/ICV, no key updates.
- **WPA:** weak passwords; no forward secrecy (captured PSK decrypts all); packet spoofing/decryption (TKIP → TCP hijack); predictable GTK (weak RNG); IP-address guessing.
- **WPA2:** weak passwords; no forward secrecy; **Hole196** (shared GTK → MITM/DoS); predictable GTK; **KRACK**; replay-detection DoS (forged group frames with large PN); insecure WPS PIN recovery.
- **WPA3:** implementation/compatibility challenges; limited adoption; resource-intensive; config errors; **timing attacks** on SAE; **cache-based side-channel**; **transition-mode weakness** (WPA2 fallback is still KRACK-vulnerable); hardware requirements.

## 6. Wi-Fi encryption cracking

### WPA/WPA2 cracking
Capture the four-way handshake (be near the AP briefly), then crack offline. Techniques: WPA PSK (per-packet key, but dictionary-attackable), offline attack (capture handshake → crack key offline), de-authentication attack (force reconnect to capture handshake containing PMK data), brute-forcing (compute-intensive, hours–weeks).

**Cracking WPA/WPA2 with aircrack-ng:**
```bash
airmon-ng start <wireless interface>                 # monitor mode (airmon-ng check kill if needed)
airodump-ng <wireless interface>                     # list APs + clients, pick target
airodump-ng --bssid <BSSID> -c 1 -w <ESSID> <interface>   # capture target packets
aireplay-ng -0 11 -a <BSSID> -c <client MAC> <interface>  # deauth to force handshake
# wait for "WPA handshake: <BSSID>" in airodump; packets saved to .cap
aircrack-ng -a2 <BSSID> -w password.txt <captured>.cap    # dictionary crack → KEY FOUND!
```
Other tools: hashcat, EAPHammer, Portable Penetrator, WepCrackGui, Wifite.

**Fern Wifi Cracker (WPA brute force):** `sudo fern-wifi-cracker` → enable monitor mode → scan for APs → pick WPA/WPA2 target → Attack (deauth → capture handshake) → choose wordlist (e.g. rockyou.txt in `/usr/share/wordlists/`) → Start WPA Attack.

### WPA3 cracking (Dragonblood)
WPA3 replaces the PSK handshake with **Dragonfly/SAE** but is still crackable. **Dragonblood** = vulnerabilities allowing key recovery, downgrade, info-theft. Tools: Dragonslayer, Dragonforce, Dragondrain, Dragontime.
- **Downgrade attacks** (client + AP support both WPA3 and WPA2): exploit backward compatibility (rogue WPA2-only AP forces the 4-way handshake), or exploit the Dragonfly handshake (masquerade as AP, claim no WPA3 → suggest WPA2).
- **Side-channel attacks:** timing-based (analyze Dragonfly encoding time → shortlist passwords), cache-based (inject JS → observe memory access patterns).

**Cracking WPA3 with aircrack-ng + hashcat:**
```bash
airmon-ng start <interface>
airodump-ng wlan0mon                                             # or: --bssid <BSSID> --channel <CH> --write capture wlan0mon
aireplay-ng --deauth 10 -a <BSSID> -c <client MAC> wlan0mon      # capture handshake
hcxpcapngtool -o capture.hccapx <capture>.cap                    # convert
hashcat -m 22000 capture.hccapx </path/to/wordlist.txt>          # crack
```

### Cracking WPS (Reaver)
Reaver brute-forces WPS registrar PINs to recover the WPA/WPA2 passphrase:
```bash
airmon-ng start wlan0                      # monitor mode
wash -i mon0                               # detect WPS-enabled devices (or airodump-ng wlan0mon)
reaver -i wlan0mon -b B4:75:0E:89:00:60 -vv   # brute-force the WPS PIN
```

## 7. Wireless threats

### Access-control attacks
Evade WLAN access controls (MAC filters, port access). **MAC spoofing** (mimic an authorized AP/device; SMAC); **AP misconfiguration** (default SSID → brute-force, SSID-as-password, config errors, SSID broadcast leaks); **ad-hoc associations** (direct device-to-device, no auth/encryption); **promiscuous client** (fake AP, same SSID, stronger signal — like evil twin); **client mis-association** (connect to a rogue/overlapping AP → MITM/EAP dictionary); **unauthorized association** (malicious soft APs, or accidental overlap).

### Integrity attacks
| Attack | Description | Tools |
|--------|-------------|-------|
| Data-frame injection | Forged 802.11 frames | Airpwn-ng, Wperf |
| WEP injection | Forged WEP keys | WEP crack+inject |
| Bit-flipping | Flip payload bits, fix ICV | — |
| Extensible AP replay | Capture 802.1X EAP for replay | capture+inject |
| Data replay | Capture 802.11 frames for modified replay | capture+inject |
| IV replay | Derive keystream via plaintext | — |
| RADIUS replay | Capture Access-Accept/Reject | Ethernet capture+inject |
| Wireless viruses | Compromise APs easily | — |

### Confidentiality attacks
Eavesdropping (Wireshark/Ettercap/Kismet), traffic analysis (Wireshark/Snort), cracking WEP key (Aircrack-ng/WEPCrack, FMS), evil twin AP (Hostapd/Wifiphisher), honeypot AP (SSID matching), session hijacking, masquerading (stolen IDs), MITM (dsniff/Ettercap/aLTEr).

### Availability attacks
AP theft (physical), disassociation, EAP-Failure, beacon flood (fake beacons hide the real AP), DoS (CSMA/CA CCA → channel appears busy), deauth flood, routing attacks (RIP/AODV/DSR via wormhole/sinkhole), authentication flood (forged auth frames fill the AP association table), ARP cache poisoning, power-saving (spoofed TIM/DTIM), TKIP MIC exploit (exceed MIC error threshold → suspend WLAN).

### Authentication attacks
| Attack | Description | Tools |
|--------|-------------|-------|
| PSK cracking | WPA PSK from handshake via dictionary | Aircrack-ng, Fern |
| EAP cracking | NT hash from EAP packets | Asleap, THC-LEAPCracker |
| VPN login cracking | PPTP/IPsec PSK brute force | ike-scan, IKECrack, THC-pptp-bruter |
| Domain login cracking | NetBIOS hash cracking | John, L0phtCrack, THC-Hydra |
| KRACK | WPA2 4-way handshake nonce reuse | nonce reuse |
| Identity theft | 802.1X Identity Response in cleartext | packet capture |
| Shared-key guessing | Default/guessed WEP keys | packet capture |
| Session hijacking | Replay 802.1X identities | packet capture |
| App login theft | Cleartext app protocols | Ace Password Sniffer, Wi-Jacking |

### Other threats
- **Honeypot AP** — rogue AP, same SSID, stronger signal → clients auto-connect → capture creds.
- **Wormhole** — malicious node tunnels RREQ/RREP (AODV/DSR) to appear as the shortest route → control all traffic (drop/alter/delay); no node compromise needed.
- **Sinkhole** — malicious node advertises itself as the best path to the base station → attract traffic → sniff/drop/alter; worse with a wormhole.
- **Inter-chip privilege escalation (wireless co-existence)** — exploit shared resources in combo Wi-Fi/Bluetooth chips (attack the weaker Bluetooth interface → pivot to Wi-Fi chip) → bypass higher-level security.

## 8. Countermeasures

### Wireless security layers (6)
1. **Wireless signal security** — monitor network + RF spectrum with a **WIDS** (rogue APs, unusual bandwidth/interference).
2. **Connection security** — per-frame/packet authentication (anti-MITM).
3. **Device security** — vulnerability + patch management.
4. **Data protection** — WPA3/WPA2/AES encryption.
5. **Network protection** — strong authentication.
6. **End-user protection** — personal firewalls on WLAN clients.

### Defense against WPA/WPA2/WPA3 cracking
Strong passwords (12–16 chars, mixed); WPA2 with AES/CCMP only; validate the server + regenerate keys per connection; VPN (remote/extranet/intranet) + IPsec/SSL/TLS; NAC/NAP; **disable TKIP** (AES only); MAC filtering; **upgrade to WPA3**; disable remote management; **disable WPS**; update router firmware; reduce signal range (central placement); monitor activity; **enable WPA3-SAE**; **disable transition mode** when all devices support WPA3.

### Defense against KRACK
Patch routers + Wi-Fi devices; auto-update firmware; avoid public Wi-Fi; browse HTTPS only; audit IoT; HTTPS Everywhere; 2FA; VPN; WPA3; disable fast roaming + repeater mode; EAPOL-key replay counter; backup wired/mobile connection; third-party routers if ISP lacks patches; network segmentation; disable 802.11r if unused; 802.1X with RADIUS for enterprise.

### Defense against aLTEr
Encrypt DNS queries (DoH/DoT, e.g. Cisco Security Connectors → Cisco Umbrella); HTTPS-only sites; **HSTS**; **DNSSEC**; AES-256; mutual authentication; firmware/patch updates; secure SIM/eSIM; VPN with integrity protection; upgrade to 5G.

### Detecting & blocking rogue APs
- **Detect:** RF scanning (RF sensors on the wired network), AP scanning (neighbor detection via MIB/web), wired-side inputs (Telnet/SNMP/CDP), compare against an authorized-AP list (AirMagnet WiFi Analyzer), signal-strength analysis (Ekahau), MAC filtering (Cisco WLC rogue detection).
- **Block:** DoS the rogue AP, block the switch port / physically remove it, **WIPS** automated blocking, ACLs for authorized MACs, 802.1X auth, network segmentation, disable open-SSID broadcast, whitelist authorized MACs.

### Best practices
- **Configuration:** change default SSID; set router password + firewall; disable SSID broadcast; disable remote login/wireless admin; MAC filtering; encryption + frequent passphrase changes; close unused ports; segregate guest network; closed networks (push SSID to employees); disable DHCP (static IPs); disable/limit SNMP; change default router IP; WPA3 (or WPA2-AES); turn off WPS; VLANs/separate SSIDs; limit transmission power; router firewall; separate guest network.
- **SSID settings:** SSID cloaking; no company/network name in passphrases; firewall between AP and intranet; limit signal strength to premises; regular config checks; IPsec over wireless; unique SSID (not default); separate guest SSID; zone segmentation; hidden SSID broadcast; WPA3/WPA2-AES minimum; periodic SSID + password changes.
- **Authentication:** WPA3 (or WPA2-AES); 802.1X with RADIUS for enterprise; MFA; manage digital certificates; disable the network when unused; secure AP placement; update drivers; centralized auth server; server verification on the client (anti-MITM); deploy rogue-AP detection / WIPS.

### WIPS
A network device monitoring the RF spectrum to detect (and auto-respond to) unauthorized APs/attacks. **Cisco WIPS components:** APs in monitor mode (channel scanning + attack detection), Mobility Services Engine (alarm aggregation + forensics), local-mode APs (service + time-sliced scanning), Wireless LAN controllers (forward attack info to MSE, push config), Wireless Control System (configure/push/view alarms, forensics, threat encyclopedia). Auditing tools: Cisco Adaptive Wireless IPS, RFProtect, Fern Wifi Cracker, OSWA-Assistant, BoopSuite, Wifite. Wi-Fi IPSs: WatchGuard Wi-Fi Cloud WIPS, Extreme AirDefense, Arista WIPS, SonicWall Wireless Network Manager, Cisco Meraki, FortiGate NGFW.

> Flow: discover (footprint/chalking → inSSIDer/wash) → analyze traffic in monitor mode (Wireshark/Kismet) → attack (deauth/MITM, rogue AP/evil twin, jamming, KRACK/aLTEr/Wi-Jacking) → crack encryption (capture the 4-way handshake → aircrack/hashcat dictionary; WPS via Reaver; WPA3 via Dragonblood/downgrade). Defend in layers: WPA3-SAE (or WPA2-AES), disable WPS/TKIP/transition mode, 802.1X+RADIUS, MAC filtering, reduced signal range, rogue-AP detection + WIPS, and patch everything against KRACK.
