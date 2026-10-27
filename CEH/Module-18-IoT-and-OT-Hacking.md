# Module 18 – IoT and OT Hacking

> Status: ✅ Reference complete

---
# PART A — IoT HACKING

## 1. IoT concepts
The **Internet of Things (IoT)** connects everyday physical devices via machine-to-machine communication + big data, bridging the digital and physical worlds. Devices sense, collect, and share data using built-in sensors, processors, and comms hardware (appliances, industrial machines, vehicles, wearables). Key features: connectivity, sensors, AI, real-time interaction.

### How IoT works (4 systems)
- **Sensing technology** — sensors collect environmental data (temperature, gas, health).
- **IoT gateways** — bridge devices to users/cloud for secure transmission.
- **Cloud server / data storage** — store, analyze, process; share results.
- **Remote control via mobile app** — monitor/control from anywhere (after authentication).

### IoT architecture (5 layers)
- **Edge Technology Layer** — foundation; sensors, RFID tags/readers, devices collecting real-world data.
- **Access Gateway Layer** — bridge between devices and clients (routing, data identification, subscriptions).
- **Internet Layer** — communication backbone connecting devices/cloud/gateways/back-end.
- **Middleware Layer** — data storage, filtering, analysis, device management, access control.
- **Application Layer** — user-facing IoT services (healthcare, smart homes, automobiles, security).

### Application areas
Smart home devices, healthcare & life sciences (wearables, pacemakers, ECG/EKG, telemedicine), Industrial IoT (IIoT), transportation (V2V, roadside systems), retail (smart payments, product tracking), IT & networks (printers, PBX).

### Technologies & protocols (by range)
- **Short-range:** BLE/Bluetooth Smart, Li-Fi (VLC, 224 Gbps), NFC, QR/barcodes, RFID, Thread (IPv6), Wi-Fi (802.11n, 600 Mbps/~50 m), Wi-Fi Direct, Z-Wave, ZigBee (IEEE 802.15.4, 10–100 m), ANT.
- **Medium-range:** HaLow, LTE-Advanced, 6LoWPAN, QUIC (over UDP, SSL/TLS-like security).
- **Long-range:** LPWAN (LoRaWAN, Sigfox, Neul), VSAT, Cellular, MQTT, NB-IoT.
- **Wired:** Ethernet, MoCA (over coax), Power-Line Communication (PLC).

### IoT operating systems
Windows 10 IoT, Amazon FreeRTOS, Fuchsia (Google), RIOT, Ubuntu Core (Snappy), ARM Mbed OS, Zephyr, Embedded Linux, NuttX RTOS, Integrity RTOS, Apache Mynewt, Tizen.

### IoT application protocols
CoAP (Constrained Application Protocol), Edge Computing, LWM2M, Physical Web (Bluetooth beacons broadcasting URLs), XMPP, Mihini/M3DA.

### Communication models
- **Device-to-Device** — direct exchange via ZigBee/Z-Wave/Bluetooth (smart home, wearables).
- **Device-to-Cloud** — device sends data to cloud (Wi-Fi/Ethernet/cellular); client authenticates to the cloud.
- **Device-to-Gateway** — device → intermediate gateway (hub/phone) → cloud (protocol translation + security).
- **Back-End Data-Sharing** — device data to cloud, accessible by authorized third parties for analysis.

### Challenges
Lack of security/privacy, vulnerable web interfaces, legal/regulatory gaps, default/weak/hardcoded credentials, cleartext protocols + open ports, coding errors (buffer overflow), storage issues, hard-to-update firmware, interoperability, physical theft/tampering, lack of vendor support, unstructured data, scalability, power consumption, regulatory compliance, legacy integration.

### Security problems & OWASP Top 10 IoT threats
Common issues: no/weak authentication, default credentials, no lockout, weak encryption, poor key management, poor physical security. **OWASP Top 10 IoT threats:** weak/guessable/hardcoded passwords; insecure network services; insecure ecosystem interfaces; lack of secure update mechanisms; insecure/outdated components; insufficient privacy protection; insecure data transfer/storage; lack of device management; insecure default settings; lack of physical hardening.

## 2. IoT threats & attacks
- **DDoS** — IoT botnet floods a target. Steps: gain remote access → inject malware (botnet) → C&C instructs bots → target offline.
- **Attack on HVAC** — exploit internet-connected HVAC (default/shared creds). Steps: Shodan find ICS → defpass.com for default creds → access ICS → reach HVAC → control temperature/pivot.
- **Rolling code attack** — jam + sniff a key fob's one-time code (car never receives it), replay later. Tools: HackRF One, RFCrack. Steps: victim presses (attacker jams + sniffs code 1) → victim retries (sniff code 2) → attacker forwards code 1 (unlocks) → keeps code 2 to steal later.
- **BlueBorne** — exploit Bluetooth protocol flaws, no pairing/interaction. Steps: discover BT devices (even non-discoverable) → get MAC → probe for OS → exploit → RCE/MITM.
- **Jamming** — flood the radio channel with noise → DoS.
- **Remote access via backdoor** — compromised IoT device as a hidden entry point.
- **Remote access via Telnet** — open Telnet port → device info/shell.
- **Sybil attack** — one attacker fakes many identities (VANET: fake traffic jams, confuse nodes).
- **Exploit kits** — automated scripts targeting unpatched IoT flaws.
- **MITM** — intercept/relay comms (weak IoT crypto).
- **Replay** — resend captured legitimate messages (e.g. IR door-unlock pattern).
- **Forged malicious device** — swap a real device for a malicious clone with a backdoor.
- **Side-channel** — infer keys from power/timing/EM emissions.
- **Ransomware** — encrypt/lock device. Phases: phishing email w/ malicious file → user opens → malware (PowerShell/Vssadmin/cmd), C&C connect, files encrypted → ransom note (bitcoin).
- **SQL injection, SDR-based, fault injection, network pivoting, DNS rebinding, FOTA (firmware update) attacks.**

### SDR-based attacks (software-defined radio)
- **Replay** — target frequency → capture original data → segregate command sequence (URH) → inject on same frequency.
- **Cryptanalysis** — replay + reverse-engineer the protocol to recover the original signal (needs crypto/modulation skill).
- **Reconnaissance** — extract device info from public records (RF certification reports) + hardware probing (multimeter → chipset → published reports).

### Identifying & accessing local IoT devices
Victim visits a malicious page → malicious JavaScript runs.
- **Discovering local devices:** get local IP (malicious code) → request all devices → active devices reply with reset packet, inactive return timeout → detect devices.
- **DNS rebinding** (via tools like Singularity of Origin) → gain access over the router → command/control local devices, extract UIDs/BSSIDs (→ geolocation).

### Fault injection (perturbation) attacks
Force errors to reveal secrets. **Non-invasive** (nearby — radio/power glitches) vs **invasive** (physical chip access). Types: Optical/EMFI/BBI (lasers, EM pulses, high-voltage on RNGs), Power/Clock/Reset glitching (skip instructions), Frequency/Voltage tampering, Temperature attacks.

### IoT malware & case study
- **KmsdBot** (Kmsdx) — scans SSH/Telnet, logs in with password lists, recruits into a botnet; multi-architecture. Others: WailingCrab, P2PInfect, NKAbuse, IoTroop, XorDdos.
- **Case study — IZ1H9** (Mirai-based, 2021→2023): **Pre-exploitation** — target Linux, scan weak devices (Tenda G103, LB-Link, DCN DCBI-Netlog-LAB, Zyxel RCE). **Exploitation** — download/execute `lb.sh` shellscript downloader from 163.123.143[.]126, delete logs, fetch bot clients per architecture (dark.x86/mips/arm4-7/ppc/m68k/sh4...), contact C2 `dotheneedfull[.]club`; decrypt config strings with **XOR key 0xBAADF00D**; spread via HTTP/SSH/Telnet (brute-force ~100 weak creds; HTTP 4 RCE CVEs). **Persistence** — hardcoded C2 connections, attack command codes.

## 3. IoT hacking methodology
Phases: **Information Gathering → Vulnerability Scanning → Launch Attacks → Gain Remote Access → Maintain Access.** Goal: unauthorized access, build botnets for DDoS; profit via stolen data resale, ransomware, DDoS-for-hire, selling camera/system access.

### Information gathering
Collect IP addresses, wireless protocols (Zigbee/BLE/5G/IPv6LoWPAN), open ports, device type/location/manufacturer.
- **Shodan** — search internet-connected devices. Filters: `webcamxp country:US`, `webcamxp city:paris`, `webcamxp geo:-50.81,201.80`, `net:`, `os:`, `port:`, `before/after:`.
- **MultiPing** — find IoT device IPs (add gateway IP range, 255 addresses, 1-sec interval).
- **FCC ID Search** — device details via the FCC ID label (grantee ID + product ID) at fcc.gov/oet/ea/fccid → Summary/Detail (cover letter, photos, test report, manual).
- **Censys** (ZMap/ZGrab banners), **FOFA** (cyberspace mapping).
- **Sniffing a web camera:**
  ```bash
  nmap -p 80,81,8080,8081 <Target IP range>   # find insecure-HTTP IoT devices
  ifconfig                                      # identify wireless card (wlan0)
  airmon-ng start wlan0                         # monitor mode
  airodump-ng start wlan0mon                    # scan networks, note channel
  airmon-ng start wlan0mon 11                   # listen on target channel 11
  # Wireshark on wlan0mon → decrypt WEP/WPA keys
  ```
- **Cascoda Packet Sniffer** (IEEE 802.15.4 — Thread/Zigbee/KNX-IoT): `sniffer -w <channel_number>`. Others: Suphacap (Z-Wave), IoT Inspector 2, ZBOSS Sniffer, tcpdump, Ubiqua.

### Vulnerability scanning
- **IoTSeeker** — find devices with factory-default creds: `perl iotScanner.pl <IP range>`.
- **Genzai** — scan IoT dashboards for default passwords: `./genzai <target_host> -save scan.json`.
- **Nmap:**
  ```bash
  nmap -n -Pn -sS -pT:0-65535 -v -A -oX <Name> <IP>                      # TCP full
  nmap -n -Pn -sSU -pT:0-65535,U:0-65535 -v -A -oX <Name> <IP>           # TCP+UDP
  nmap -6 -n -Pn -sSU -pT:0-65535,U:0-65535 -v -A -oX <Name> <IP>        # IPv6
  ```
- Tools: beSTORM (smart fuzzer), Metasploit, IoTsploit, IoTVAS, Enterprise IoT Security.

### Spectrum & traffic analysis
- **Gqrx** (SDR + GNU Radio) — analyze frequency bands (sensors, car keys, M-bus); `git clone ... && cmake .. && make`, launch `gqrx` (with FunCube Dongle Pro+/HackRF/RTL-SDR).
- **ONEKEY** — find IoT devices + inspect/record/replay traffic.
- **Universal Radio Hacker (URH)** — investigate unknown protocols (demodulation, crack CC1101 whitening, reverse-engineer fields, fuzzing, modulation/injection). SDR tools: BladeRF, TempestSDR, HackRF One.

### Launch attacks
- **Rolling code — RFCrack:**
  ```bash
  python RFCrack.py -i                                   # live replay
  python RFCrack.py -r -M MOD_2FSK -F 314350000          # rolling code
  python RFCrack.py -r -M MOD_2FSK -F 314350000 -U -100 -L -10   # adjust RSSI
  python RFCrack.py -j -F 314000000                      # jamming
  python RFCrack.py -k                                   # scan common frequencies
  python RFCrack.py -b -v 5000000                        # incremental scan
  python RFCrack.py -s -u ./captures/test.cap -F 315000000 -M MOD_ASK_OOK   # send payload
  ```
- **Zigbee — Open Sniffer** (Wireshark-based, 802.15.4/Zigbee) → capture/replay/spoof/DoS.
- **BlueBorne — HackRF One** (1 MHz–6 GHz, half-duplex) → replay/fuzzing/jamming.
- **Replay — HackRF One:**
  ```bash
  hackrf_transfer -r connector.raw -f [device frequency]   # record
  hackrf_transfer -t connector.raw -f [device frequency]   # replay
  ```
- **SDR — RTL-SDR** (USB dongle, 500 kHz–1.75 GHz) + **GNU Radio** tools: `uhd_ft` (spectrum analyzer), `uhd_rx_cfile` (store samples), `uhd_rx_nogui` (listen), `uhd_siggen_gui` (generate signals), `gr_plot`.
- **Side-channel — ChipWhisperer** — power analysis + glitching (capture board + target board) to recover keys.
- **Identify communication buses/interfaces** (UART/SPI/JTAG/I2C) with **BUS Auditor** (16 channels CH0–CH15) + DIVA board via the EXPLIoT framework:
  ```bash
  run busauditor.generic.uartscan -v 3.3 -p /dev/ttyACM0 -s 0 -e 1    # UART (CH0,CH1)
  run busauditor.generic.jtagscan -v 3.3 -p /dev/ttyACM0 -s 0 -e 10   # JTAG (CH0-CH8; TMS/TCK/TDI/TDO/TRST)
  run busauditor.generic.i2scan  -v 3.3 -p /dev/ttyACM0 -s 0 -e 10    # I2C (SDA/SCL)
  ```
  SPI identified via Google search of chip numbers. Tools: JTAGulator, Attify Badge, Saleae Logic Analyzer.
- **NAND glitching** (boot-time hardware attack → root):
  ```bash
  minicom -D /dev/ttyUSB0 -w -C D-link_startup.txt     # recon bootlogs via UART-USB
  # short the flash serial I/O pin to ground during boot → backup loader
  printenv                                              # view bootargs
  setenv bootargs 'noinitrd console=ttyAM0,115200 rootfstype=ubifs ubi.mtd=5 root=ubi0:rootfs rw gpmi init=/bin/sh';
  nand read ${loadaddr} app-kernel 0x00400000 && bootm ${loadaddr}   # root shell
  ```
- **Exploit cameras — CamOver** (CCTV/GoAhead/Netwave): `camover -a <Camera IP>`, `camover -t --shodan <API Key>`.

### Gain remote access (Telnet)
Scan for open Telnet (Shodan/Censys), probe model/software; no auth = immediate access, else default logins (`root/root`) or brute force → read/alter data, lateral movement, malware.

### Maintain access (firmware exploitation)
Clear logs, update/modify firmware, install backdoors/Trojans. Tools: **Firmware Mod Kit** (unpack/modify/rebuild TRX/uImage + SquashFS/CramFS → add backdoors → reflash), IoTVAS, Firmware Analysis Toolkit, Firmwalker.

### Firmware analysis & reverse engineering
Uncover passwords, API tokens, endpoints, backdoor accounts, configs, private keys.
```bash
file xyz.bin                                        # file type
cat *.md5 ; md5sum xyz.bin                          # verify MD5
strings -n 10 xyz.bin > strings.out ; less strings.out
hexdump -C -n 512 xyz.bin > hexdump.out            # identify build type
binwalk xyz.bin                                     # identify filesystem
dd if=xyz.bin bs=1 skip=922460 count=2522318 of=xyz.squashfs   # extract fs
mkdir rootfs ; sudo mount -t ext2 {filename} rootfs            # mount
# analyze: etc/passwd, etc/shadow, etc/ssl; grep -rnw '/path/' -e "password|admin|root"
# find . -name '*.conf' (also *.pem, *.crt, *.cfg, .sh, .bin) ; run Firmwalker
# emulate (dynamic testing):
file / readelf     # identify CPU architecture
qemu-mipsel -L <prefix> <binary>    # user-mode emulation
qemu-arm   -L <prefix> <binary>
chroot ~/<filename> /bin/            # cross-architecture chroot (qemu-<arch>-static in /usr/bin/)
```
Tools: QEMU, Firmware Analysis Toolkit. IoT hacking tools: CatSniffer, KillerBee (ZigBee/802.15.4), JTAGULATOR, PENIOT, RouterSploit.

## 4. IoT countermeasures
### Defend against IoT hacking
Strong authentication + password policy (8–10+ chars, lockout/CAPTCHA); regular updates/patching; network segmentation + firewalls; disable default/unused features (Telnet/UPnP); encryption + VPN + PKI; access control + zero trust (IAM/IDS/IPS/NAC); secure boot + trusted hardware (TPM/HSM/SAM); centralized device management; physical/environmental security; cloud + anti-DDoS.

### Prevent SDR-based attacks
Secure the signal (encryption); avoid command repetition (rolling technique); synchronization + preamble nibbles (anti de-Bruijn brute force); anti-jamming; frequency hopping (FHSS); secure key management (HSM); secure OTA updates (cryptographic signatures).

### OWASP Top 10 IoT — solutions
| Vulnerability | Solutions |
|---------------|-----------|
| Weak/hardcoded passwords | APM, strong complex passwords, no hardcoded |
| Insecure network services | close open ports, disable UPnP, encrypt before TLS |
| Insecure ecosystem interfaces | account lockout, periodic assessment, sanity/output filtering, 2FA |
| Lack of secure update | verify source/integrity, encrypt endpoints, notify users |
| Insecure/outdated components | monitor unmaintained, remove unused deps, avoid compromised supply chain |
| Insufficient privacy | minimize + anonymize data, user control |
| Insecure data transfer/storage | encrypt endpoints, SSL/TLS, no proprietary crypto |
| Lack of device management | blacklist malicious, validate assets, secure decommissioning |
| Insecure default settings | change defaults, custom privacy/security, disable remote access |
| Lack of physical hardening | BIOS/firmware password, boot order, minimize USB ports |

### IoT framework security (4 parts)
**Edge** (cross-platform, encryption, no default creds, updates), **Gateway** (strong encryption, mutual auth, central updates), **Cloud Platform** (restricted authenticated access, encryption, hardened web interface, auto updates), **Mobile** (strong auth, lockout, safe local storage, encrypted channels).

### IoT hardware & secure development
Hardware best practices: limit entry points, tamper protection + GPS, secure boot (TPM), patches, interface management, avoid open hardware access, secure auth keys, event logging, anti-malware, protect credentials, isolate power (anti-rowhammer), root-of-trust, modern gateways for legacy, secure wireless, secure debug (JTAG/UART), hardware root of trust (TEE/enclave), secure sensor data, hardware IDS. TPM countermeasures: BitLocker, TPM bind key (RSA), sealing/unsealing, HMAC, symmetric-key for small data, RSA signatures, Root of Trust, perfect forward secrecy, certificate-based auth, remote attestation. Secure dev: secure boot, secure API endpoints, threat modeling, secure coding, security testing, secure OTA, device identity management, hardware security, code signing, runtime protection, secure cloud integration, MQTT with TLS/SSL.

### Device management & tools
Solutions: Azure IoT Central, Oracle Fusion Cloud IoT, Golioth, AWS IoT Device Management, IBM Watson IoT, openBalena. Security tools: SeaCat.io, Armis Centrix, FortiNAC, Microsoft Defender for IoT, Cisco Industrial Threat Defense, AWS IoT Device Defender, Forescout, Azure Sphere.

---
# PART B — OT HACKING

## 5. OT concepts
**Operational Technology (OT)** = hardware + software to monitor/control industrial devices (pumps, sensors, valves, robots, cameras). Includes SCADA, PLCs, RTUs, DCS. Historically isolated from the internet; OT/IT convergence now exposes it to IT vulnerabilities.

### Essential terminology
- **Assets** — physical (sensors, actuators, servers, PLCs) + logical (process graphics, program logic, DB, firmware, firewall rules).
- **Zones and conduits** — network segregation for access control.
- **Industrial vs business network** — automated control systems vs enterprise information infrastructure.
- **Industrial protocols** — proprietary (S7, CDA, SRTP) or open (Modbus, OPC, DNP3, CIP); application-layer.
- **Network/electronic security perimeter** — boundary between secure/insecure zones.
- **Critical infrastructure** — systems whose failure severely impacts security/safety/economy/health.

### ICS (Industrial Control Systems)
Combination of control systems, devices, networks, software. Types: SCADA, DCS, BPCS, SIS, HMI, PLC, RTU, IED. Modes: **open loop** (no feedback), **closed loop** (adjusts on output), **manual loop** (human-controlled).
- **DCS (Distributed Control System)** — central supervisory + multiple local controllers (power plants, refineries); feedback/feedforward loops, redundancy, scalability.
- **SCADA** — centralized monitoring/control over large areas (pipelines, grids); collects real-time data from PLCs/RTUs/IEDs.
- **PLC (Programmable Logic Controller)** — rugged digital computer; **3 modules:** CPU (processor + RAM/ROM + retentive memory), Power Supply (AC→DC; 5V CPU, 24V sensors), I/O (Digital, Analog [ADC], Communication).
- **BPCS (Basic Process Control System)** — first protection layer; trending/alarm logging, HMI, batch sequencing, recipes, safety interlocks.
- **SIS (Safety Instrumented System)** — independent; detects abnormal conditions → safe state. 3 components: **sensors** (measure T/P/flow), **logic solvers** (decide actions), **final control elements** (valves). Examples: fire & gas, shutdown, interlock systems.

### IT/OT convergence (IIoT)
Integration of IT (data/cybersecurity) + OT (industrial ops) = Industry 4.0. Benefits: enhanced decision-making/automation, faster output, lower expenses, risk mitigation, agility, predictive maintenance, quality control, compliance, scalability.

### The Purdue model (PERA, ISA-95)
Layered reference architecture; zones separated by a DMZ.
- **Enterprise Zone (IT):** **Level 5** (Enterprise Network — B2B/B2C, internet), **Level 4** (Business Logistics — schedules/planning, app/file/DB servers).
- **Industrial DMZ (IDMZ, Level 3.5)** — buffer (domain controllers, DB replication, proxy servers).
- **Manufacturing Zone (OT):** **Level 3** (Operational Systems/Site Ops — scheduling, batch, QA, historians, MES/MOMS), **Level 2** (Control Systems/Area Supervisory — DCS/SCADA/HMI/PLC line control), **Level 1** (Basic Controls/Intelligent Devices — sensors, IEDs, PLCs, RTUs, PID, VFDs), **Level 0** (Physical Process / Equipment Under Control — sensors, actuators).

### OT protocols (by Purdue level)
- **L4/5:** DCOM, FTP/SFTP, GE-SRTP, IPv4/IPv6, OPC UA, TCP/IP, SMTP, HTTP/HTTPS, Wi-Fi.
- **L3:** CC-Link, HSCP, ICCP (IEC 60870-6), IEC 61850, IEC 60870-5-104, ISA/IEC 62443, Modbus, NTP, Profinet, SuiteLink, Tase-2, ControlNet, Profibus PA/DP.
- **L2:** 6LoWPAN, DNP3, DNS/DNSSEC, FTE, HART-IP, IEC 60870-5-101/104, SOAP, DeviceNet, AS-i.
- **L0/1:** BACnet, EtherCAT, CANopen, Crimson, DeviceNet, Zigbee, ISA100, MELSEC-Q, Niagara Fox, Omron Fins, PCWorx, Profibus, Sercos II, S7 Communication, WiMax, FOUNDATION Fieldbus.

### OT challenges & vulnerabilities
Challenges: lack of visibility, plain-text passwords, network complexity, legacy tech, no antivirus, skill shortage, rapid change, outdated systems, insecure connections, rogue devices, IT convergence, vulnerable protocols (Modbus/Profinet), remote-management protocols (RDP/VNC/SSH), insufficient segmentation, physical security, vendor dependencies, no encryption, data integrity issues.

| OT Vulnerability | Solution (from countermeasures) |
|------------------|--------------------------------|
| Publicly accessible OT systems | MFA, enterprise firewall/remote access, incident response plans |
| Insecure remote connections | MFA + password policy, patching, RBAC |
| Missing security updates | sandbox testing, firewall + hardening |
| Weak passwords | separate IT/OT usernames, change defaults, audits |
| Insecure firewall config | secure config, ACLs |
| OT in corporate IT network | segregate IT/OT, DMZ, monitor DMZ |
| Insufficient IT protection from OT | restrict access by need, secure gateway, risk assessment |
| Lack of OT segmentation | separate critical/non-critical, zoning + defense-in-depth, zero trust |
| No wireless encryption/auth | strong wireless crypto, standard algorithms, audits |
| Unrestricted outbound internet | risk assessment, segregate, separate update repo |

### MITRE ATT&CK for ICS (tactics)
- **Initial Access** — drive-by compromise, exploit public-facing app, exploit remote services, external remote services, internet-accessible devices, removable media, rogue master, spear-phishing attachment, supply-chain compromise, transient cyber assets, wireless compromise.
- **Execution** — change operating mode, CLI, execution through APIs, GUI, hooking, modify controller tasking, native API, scripting, user execution.
- **Persistence** — modify program, module firmware, project file infection, system firmware, valid accounts.
- **Privilege Escalation** — exploit software, hooking.
- **Evasion** — remove indicators, rootkits, change operator mode, exploit software vulns, masquerading, spoofed reporting messages.
- **Discovery** — enumerate network connection, network sniffing, identify remote systems, remote system info discovery, wireless sniffing.
- **Lateral Movement** — default credentials, program download, remote services, lateral tool transfer, valid accounts.
- **Collection** — automated collection, information repositories, I/O image, detect operating mode, MITM, monitor process state, point/tag identification, program upload, screen capture, wireless sniffing.
- **Command and Control** — frequently used ports (80/443), connection proxy, standard application-layer protocol (HTTPS/Telnet/RDP).
- **Inhibit Response Function** — activate firmware update mode, block command/reporting messages, alarm suppression, block serial COM, data destruction, DoS, device restart/shutdown, rootkit, service stop.
- **Impair Process Control** — I/O brute-forcing, alter parameters, module firmware, spoofed reporting, unauthorized command messages.
- **Impact** — damage to property, loss of availability/control/view/safety/productivity, denial of control/view, manipulation of control/view, theft of operational info.

### OT threats
Maintenance/admin threat (zero-days), data leakage, protocol abuse (Modbus/CAN spoofing), destruction of ICS resources, reconnaissance, DoS (malformed CIP), HMI-based attacks, exploiting enterprise tools (SIS), spear phishing, malware, unpatched vulns, side-channel, buffer overflow, exploiting RF remote controllers.

### HMI-based attacks
Memory corruption, credential management (hardcoded/plaintext), lack of authorization/auth + insecure defaults, code injection (SQL/OS/HMI-specific like Gamma EvalExpression), buffer overflow, path traversal (`../etc/passwd`).

### Side-channel attacks
- **Timing analysis** — measure auth response time per character to reconstruct passwords (defend: constant-time comparison, random noise, throttling).
- **Power analysis** — measure power consumption with probes/oscilloscope to recover keys; stealthy.

### Hacking PLCs
- **PLC rootkit (ghost) attack:** Step 1 — inject rootkit, control-flow attack to guess default password → root. Step 2 — map I/O modules + memory locations. Step 3 — manipulate I/O init sequence → full control. (PLC CPUs have program mode + run mode; malware uploaded replaces original program.)
- **Evil PLC attack:** Step 1 — find vulnerable PLC (Shodan/Censys). Step 2 — exploit firmware, weaponize by changing logic via download. Step 3 — infected PLC runs arbitrary code on connected workstations during upload.

### RF remote controller attacks
TX/RX radio commands. Threats: **replay** (record + replay RF packets), **command injection** (reverse-engineer RF packets, craft new commands), **abusing E-stop** (flood emergency-stop → DoS), **re-pairing with malicious RF controller**, **malicious reprogramming** (inject malware into firmware for persistence).

### OT supply chain attacks
Third-party software compromise, hardware manipulation, service provider breach, injection of malicious components, exploitation of trusted relationships.

### OT malware & COSMICENERGY case study
- **Fuxnet** — ICS malware; modifies data, blocks sensor gateways, damages sensors (rewrites memory chips), malformed packets. Others: Stuxnet, INDUSTROYER.V2, Kapeka, COSMICENERGY, Pipedream.
- **COSMICENERGY** — targets OT/ICS via **IEC 60870-5-104 (IEC-104)** devices (RTUs). **Stage 1 Recon** — find MSSQL servers with OT access, IEC-104 device IPs. **Stage 2 Initial Access** — connect to remote MSSQL. **Stage 3 Attack** — two components: **PIEHOP** (Python; connects to MSSQL, uploads files, issues RTU commands via `r3_iec104_control.py` + `iec104_mssql_lib`), **LIGHTWORK** (C++; crafts IEC-104 ASDU messages: `<ip> <port> <command ON=1/OFF=0>`; sends `C_IC_NA_1` station interrogation → `C_SC_NA_1` single command per IOA → `C_CS_NA_1` clock sync) → toggles power-line switches/breakers → outages. **Stage 4** — no traditional lateral movement; delete PIEHOP/LIGHTWORK traces.

## 6. OT hacking methodology
Phases: **Information Gathering → Vulnerability Scanning → Launch Attacks → Gain Remote Access → Maintain Access.** Profit: full control/damage, shut down production (DDoS), reprogram assembly (faulty products), injure employees, install malware/ransomware.

### Information gathering
- **Shodan (ICS/SCADA):** by port (`port:502` Modbus), by PLC name (`Schneider Electric`), by geolocation (`SCADA Country:"US"`). SCADA ports: Modbus 502, Fieldbus 1089–1091, DNP3 19999/20000, Ethernet/IP 2222, PROFINET 34962–34964, EtherCAT 34980.
- **CIRT.net** — default password database for ICS/routers/switches.
- **Nmap (ICS/SCADA):**
  ```bash
  nmap -Pn -sT --scan-delay 1s --max-parallelism 1 -p 80,102,443,502,530,593,789,1089-1091,1911,1962,2222,2404,4000,4840,4843,4911,9600,19999,20000,20547,34962-34964,34980,44818,46823,46824,55000-55003 <IP>   # ICS protocol recon
  nmap -Pn -sT -p 46824 <IP>                                   # HMI (Sielco Sistemi Winlog)
  nmap -Pn -sT -p 102 --script=s7-info <IP>                    # Siemens SIMATIC S7
  nmap -Pn -sT -p 502 --script modbus-discover <IP>            # Modbus (add --script-args='modbus-discover.aggressive=true')
  nmap -Pn -sU -p 47808 --script bacnet-info <IP>              # BACnet
  nmap -Pn -sU -p 44818 --script enip-info <IP>                # Ethernet/IP
  nmap -Pn -sT -p 1911,4911 --script fox-info <IP>             # Niagara Fox
  nmap -Pn -sT -p 20547 --script proconos-info <IP>            # ProConOS
  nmap -Pn -sT -p 9600 --script omron-info <IP>                # Omron FINS
  nmap -Pn -sT -p 1962 --script pcworx-info <IP>               # PCWorx
  ```
- Tools: Kamerka-GUI (Shodan-based ICS mapping + heat maps), SearchDiggity, Zeek, Criminal IP, ZoomEye, ONYPHE. **NetworkMiner** (passive forensics), **Wireshark** (Modbus/TCP cleartext), **Malcolm** (ICS protocol discovery, OpenSearch/Arkime).

### Vulnerability scanning
- **Nessus** — Basic Network Scan template, port range 0–1000, SCADA plugins → SCADA vulns.
- **Skybox Vulnerability Control** — combined OT+IT, attack-path analysis.
- **SmartRF Packet Sniffer** (ZigBee/EasyLink/BLE), **Microsoft Defender for IoT**.
- **Fuzzing ICS protocols — Fuzzowski:**
  ```bash
  python -m fuzzowski 127.0.0.1 47808 -p udp -f bacnet -rt 0.5 -m BACnetMon   # BACnet
  python -m fuzzowski 127.0.0.1 502 -p tcp -f modbus -rt 1 -m modbusMon       # Modbus
  python -m fuzzowski printer1 631 -f ipp -r get_printer_attribs --restart smartplug   # IPP
  ```

### Launch attacks
- **Hacking ICS hardware** — hardware tools (signal analyzer, multimeter, microcontroller/memory programmer, oscilloscope, soldering, microscope, JTAG, screwdrivers, tweezers/UART converters) + software (GDB, OpenOCD [GDB on port 3333 / telnet 4444], Binwalk, Fritzing, Radare2, Ghidra, IDA Pro).
- **Modbus slaves — Metasploit:** `auxiliary/scanner/scada/modbus_findunitid` (scan slaves), `auxiliary/scanner/scada/modbusclient` (read/write registers + coils).
- **PLC — modbus-cli** (Schneider TM221, Modbus/TCP):
  ```bash
  # Shodan: TM221ME16R → find Internet-connected TM221 PLCs
  gem install modbus-cli
  modbus read  <IP> %MW100 10        # Schneider address (read 10 words)
  modbus read  <IP> 400101 10        # Modicon address
  modbus write <IP> %MW100 2 2 2 2 2 2 2 2      # write registers
  modbus read  <IP> 101 10           # read coils (Boolean 1/0)
  modbus write <IP> %M100 1 1 1 1 1 1 1 1 1 1   # turn ON coils
  modbus read --output SCADAregisters.txt <IP> 400101 200   # capture registers
  modbus read --output SCADAcoils.txt <IP> 101 100          # capture coils
  ```

### Gain & maintain remote access (DNP3)
Exploit poorly secured networks / default creds. Shodan scans for **DNP3 on port 20000** → login interface → default creds/brute force. Maintain access: clear logs, modify firmware, install rootkits. OT tools: **mbtget** (Perl Modbus TCP/RTU), CSET, Attkfinder, ICSREF, ICSFuzz, ISF.

## 7. OT countermeasures
### Defend against OT hacking
Regular risk assessment; patch/update systems; secure remote access (VPN/MFA/encryption/firewalls); continuous monitoring; strong passwords (replace defaults, hashed); network segmentation (DMZ); OT-aware IDS/IDPS; threat intelligence + red teaming; employee training; secure industrial protocols (encrypt Modbus/DNP3).

### Purdue-model security controls
| Zone | Level | Attack vectors | Security controls |
|------|-------|----------------|-------------------|
| Enterprise | 5 & 4 | spear phishing, ransomware | firewalls, IPS, anti-bot, URL filtering, SSL inspection, antivirus, DLP |
| Industrial DMZ | 3.5 | DoS | anti-DoS, IPS, anti-bot, application control, ALF |
| Manufacturing | 3 | ransomware, bot, unsecured USB | anti-bot, IPS, sandboxing, app control, traffic encryption, port protection |
| Manufacturing | 2 & 1 | DDoS, unencrypted protocols, default creds, app/OS vulns | IPS, firewall, IPsec, security gateways, authorized RTU/PLC commands |
| Manufacturing | 0 | physical breach | point-to-point comms, MAC auth, security gateways at L1/L0 |

### Zero-trust for ICS/SCADA
1. **Define the network** (attack surface, assets, vulnerable data, critical apps).
2. **Map the traffic** (flows, device interactions).
3. **Architect the network** (NGFW + segmentation gateway).
4. **Develop a ZT policy** (whitelist users/devices — who/why/when/what).
5. **Monitor and maintain** (continuous traffic monitoring + updates).

### Organizations, solutions & tools
Orgs: OTCC, OT-ISAC, NERC, IISF, ISA/IEC-62443. Solutions: firewalls (FortiGate Rugged, OTIFYD), unified identity + OT access management (Claroty, MetaDefender IT-OT Access), asset inventory + device authorization (SCADAfence, OTbase, Guardian, Dragos), OT network monitoring + anomaly detection (iSID, Rhebo), decoys/honeypots (Attivo ThreatDefend, Conpot, GasPot). Tools: Flowmon, Tenable OT Security, Nozomi Networks, Forescout, FortiGuard, RAM2.

> Flow (both): Information Gathering (Shodan/Censys/FCC for IoT; Shodan ports/Nmap ICS scripts for OT) → Vulnerability Scanning (IoTSeeker/Genzai/Nmap; Nessus/Fuzzowski) → Launch Attacks (SDR replay/rolling-code/BlueBorne/bus+NAND glitching for IoT; Modbus Metasploit/modbus-cli/PLC rootkit/Evil PLC for OT) → Gain Remote Access (Telnet for IoT, DNP3 for OT) → Maintain Access (firmware mod, rootkits). Defend with segmentation + zero trust, no default credentials, secure boot/hardware root of trust, encrypted protocols, and OT-aware monitoring (WIPS/IDS, Purdue-model controls, ISA/IEC-62443).
