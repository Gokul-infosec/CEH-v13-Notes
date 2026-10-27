# Module 04 – Enumeration

> Status: ✅ Reference complete

## 1. Goal
Actively extract detailed info after recon — usernames, system/host names, network shares, services, groups — to set up exploitation. Placeholders: `<target>`, `<target_ip>`.

**Techniques:**
- Extract usernames from emails (`username@domain`).
- Try default manufacturer usernames/passwords.
- Brute Active Directory using login error messages to find valid usernames, then crack.
- DNS zone transfer (if DNS misconfigured) for domain/IP details.
- Extract Windows user groups (needs a valid account).
- SNMP — guess community strings → usernames; map network resources/structure.

**Enumerable services/ports:**
| Port | Proto | Service |
|------|-------|---------|
| 53 | tcp/udp | DNS (UDP default, TCP for large/zone transfer) |
| 135 | tcp/udp | MS RPC Endpoint Mapper |
| 137 | udp | NetBIOS Name Service (NBNS) |
| 139 | tcp | NetBIOS Session Service |
| 445 | tcp/udp | SMB over TCP |
| 161 / 162 | udp | SNMP / SNMP Trap |
| 389 | tcp/udp | LDAP |
| 3268 | tcp/udp | Global Catalog Service |
| 2049 | tcp | NFS |
| 25 | tcp | SMTP |
| 500 | udp | ISAKMP/IKE (IPsec VPN) |
| 22 | tcp | SSH/SFTP |
| 5060 / 5061 | tcp/udp | SIP (unencrypted / encrypted) |
| 20 / 21 | tcp | FTP |
| 23 | tcp | Telnet (plaintext) |
| 69 | udp | TFTP |
| 179 | tcp | BGP |

## 2. NetBIOS enumeration
NetBIOS = name-based LAN comms for Windows file/printer sharing. UDP 137 (name), UDP 138 (datagram), TCP 139 (session). Targeted because it's easy to exploit and runs even when unused. Yields: domain computer list, host shares, policies & passwords. (Not supported for IPv6 name resolution.)

### nbtstat (Windows)
```text
nbtstat -n              # local registered NetBIOS names
nbtstat -c              # NetBIOS name cache contents
nbtstat -r              # name-resolution stats (broadcast + WINS)
nbtstat -R              # purge & reload remote cache from LMHOSTS
nbtstat -RR             # release + refresh name with WINS
nbtstat -s              # sessions table (by name)
nbtstat -S              # sessions table (by IP)
nbtstat -a <HOSTNAME>   # remote machine's NetBIOS name table (by name)
nbtstat -A 10.10.10.5   # same, by IP
```

### net use (Windows)
```text
net use                                          # current connections / mapped drives
net use \\<ip>\share                             # connect to a share
net use \\<ip>\share /user:<user>                # connect with creds
net use Z: \\<ip>\share                          # map to drive Z:
net use /delete Z:                               # disconnect Z:
net use * /delete                                # disconnect all
net use \\<ip>\IPC$ /user:<user>                 # hidden IPC$ share (enum / exploits)
```

### Nmap (NSE)
```bash
nmap -sU -p 137 <target>                                 # NBNS discovery
nmap -sU --script nbstat -p 137 <target>                 # NetBIOS names/user/domain/MAC
nmap -p 139,445 --open <target>                          # NetBIOS/SMB open check
nmap -p 139,445 --script smb-os-discovery <target>       # OS, computer/domain/NetBIOS name
nmap --script smb-enum-shares -p 139,445 <target>        # shares (ADMIN$, IPC$, read/write)
nmap --script smb-enum-users -p 139,445 <target>         # users
nmap --script smb-enum-groups -p 139,445 <target>        # groups (AD)
nmap --script smb-enum-processes -p 445 <target>         # processes
nmap --script smb-enum-sessions -p 445 <target>          # active sessions
nmap --script smb-security-mode -p 445 <target>          # null-session check
nmap --script smb2-security-mode -p 445 <target>         # SMBv2 signing/guest
nmap --script smb-protocols -p 445 <target>              # SMB versions
nmap --script smb2-capabilities -p 445 <target>          # SMBv2 capabilities
nmap --script "smb* and not intrusive" -p 139,445 <target>                    # safe initial sweep
nmap -Pn -n -p 137,139,445 -sS --script nbstat,smb-os-discovery <target>      # stealth combo
nmap -p 137,139,445 --script nbstat,smb-os-discovery,smb-enum-shares,smb-enum-users <target>
```
Other tools: Global Network Inventory, Advanced IP Scanner, Hyena, Nsauditor, NetBIOS Enumerator (nbtenum.sourceforge.net — extracts data on null-session hosts).

## 3. User-account & shared-resource enumeration (Windows)
**PsTools suite** (Sysinternals, no client install needed; `-?` for help): PsExec (run processes remotely), PsFile (remote-opened files), PsGetSid (SID), PsInfo (system info), PsPing, PsKill, PsList, PsLoggedOn, PsLogList (event logs), PsPasswd, PsService, PsShutdown, PsSuspend.

**net view** (shared resources):
```text
net view                             # visible computers in domain/network
net view \\10.10.10.5                # shares on a host
net view /domain                     # available domains
net view /workgroup                  # workgroups
net config workstation               # NetBIOS name + domain/workgroup
net config server                    # server service config / shares
net share                            # local shares
net share ShareName                  # details of a local share
net use \\10.10.10.5\ShareName /user:Administrator   # connect with creds
net share NewShare=c:\folder /grant:everyone,full    # create a share (caution)
```

## 4. SNMP enumeration
SNMP manages/monitors devices (routers, switches, servers, printers). Enumerate network resources (hosts/routers/shares) + info (ARP/routing tables, traffic stats) via default community strings. Default string is `public`; also try `private`, `admin`. UDP 161 (v1/v2c unencrypted; v3 is auth+encrypted).

### Key OIDs
| OID | Purpose |
|-----|---------|
| 1.3.6.1.2.1.1 | System info (sysName/Descr/Location) |
| 1.3.6.1.2.1.2.2 | Network interfaces |
| 1.3.6.1.2.1.4.20 | IP addresses |
| 1.3.6.1.2.1.25.4.2 | Running processes |
| 1.3.6.1.2.1.25.6.3 | Installed software |
| 1.3.6.1.2.1.6.13 | TCP connections |

### snmpwalk
```bash
snmpwalk -v1 -c public <target>                          # all info, SNMPv1
snmpwalk -v2c -c public <target>                         # all info, SNMPv2c
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.1           # system info
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.25.1.1      # uptime
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.25.4.2      # running processes
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.25.6.3      # installed software
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.6.13        # active TCP connections
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.4.20        # interface IPs
snmpwalk -v2c -c public <target> 1.3.6.1.2.1.2.2.1.2     # interface names
snmpwalk -v2c -c public <target> 1.3.6.1.4.1             # private enterprise MIB tree
snmpwalk -v2c -c public <target> hrSWInstalledName       # installed software (MIB label)
snmpwalk -v2c -c public <target> hrMemorySize            # memory size
snmpwalk -v2c -c public <target> hrSWRunName             # process names (MIB label)
snmpwalk -v2c -c private <target>                        # try a different community string
snmpwalk -v2c -c public -t 5 -r 2 <target>               # timeout 5s, 2 retries
snmpwalk -v2c -c public -O e <target>                    # numeric OIDs
snmpwalk -v2c -c public -On -Oq <target>                 # minimal numeric output (parsing)
snmpwalk -v2c -c public -m ALL <target>                  # load all MIBs (needs MIBs installed)
```
> Note: snmpwalk is read-only — it cannot set values (use snmpset for that).

### nmap + snmp-check
```bash
nmap -sU -p 161 --script=snmp-processes <target>         # running processes
nmap -sU -p 161 --script=snmp-sysdescr <target>          # system/OS description
nmap -sU -p 161 --script=snmp-win32-software <target>    # installed software (Windows)

snmp-check <target>                   # default enum (public, v1)
snmp-check -c private <target>        # private community
snmp-check -v2c -c public <target>    # SNMPv2c
snmp-check -t <target> --os           # OS info only
snmp-check -t <target> --hardware     # CPU/mem/storage
snmp-check -t <target> --software     # installed software
snmp-check -t <target> --processes    # processes
snmp-check -t <target> --tcp-connections   # TCP connections
snmp-check -t <target> --network-interfaces
snmp-check -t <target> --users        # local users
snmp-check -t <target> --routing-table
snmp-check -t <target> --all          # full enumeration
snmp-check -t <target> -o output.txt  # save output
```
> Workflow: `snmp-check` for quick high-level → `snmpwalk` for deep OID brute-forcing → `onesixtyone` for community-string discovery. Other tools: SoftPerfect Network Scanner, SolarWinds Network Performance Monitor, OpUtils, PRTG, Engineer's Toolset.

## 5. LDAP enumeration
LDAP = directory-service access (e.g. Active Directory) over TCP 389 (LDAPS 636), using BER encoding. Unsecured LDAP leaks usernames, emails, departments, server info without auth.

### Manual (Python ldap3)
```python
pip3 install ldap3
python3
>>> import ldap3
>>> server = ldap3.Server('Target_IP', port=389, get_info=ldap3.ALL)   # use_ssl=True for LDAPS
>>> connection = ldap3.Connection(server)
>>> connection.bind()                 # True = success
>>> server.info                       # domain info / naming context
>>> connection.search(search_base='DC=example,DC=com', search_filter='(&(objectClass=*))', search_scope='SUBTREE', attributes='*')
>>> connection.entries
>>> connection.search(search_base='DC=example,DC=com', search_filter='(&(objectClass=person))', search_scope='SUBTREE', attributes='userPassword')
>>> connection.entries
```

### ldapsearch
Flags: `-x` simple bind · `-h` host · `-H` URI · `-D` bind DN · `-w` password · `-b` base DN · `-s base|one|sub` scope · `-LLL` clean output.
```bash
ldapsearch -x -h <target> -s base                                   # server responding?
ldapsearch -x -h <target> -s base namingcontexts                    # base DNs
ldapsearch -x -h <target> -b "dc=example,dc=com" "(objectClass=*)"  # all objects
ldapsearch -x -h <target> -b "dc=example,dc=com" "(objectClass=person)"   # users
ldapsearch -x -h <target> -b "dc=example,dc=com" "(objectClass=group)"    # groups
ldapsearch -x -h <target> -b "dc=example,dc=com" "(cn=*)" cn        # common names
ldapsearch -x -h <target> -b "dc=example,dc=com" -LLL dn            # DNs only
ldapsearch -x -h <target> -b "dc=example,dc=com" "(!(objectClass=computer))"   # humans/groups only
ldapsearch -x -h <target> -b "dc=example,dc=com" "(userAccountControl=512)"    # enabled accounts
ldapsearch -x -h <target> -b "dc=example,dc=com" "(adminCount=1)"              # privileged accounts
ldapsearch -x -h <target> -b "dc=example,dc=com" "(servicePrincipalName=*)"    # service accts (Kerberoasting)
ldapsearch -x -h <target> -b "dc=example,dc=com" "(sAMAccountName=*)" sAMAccountName   # usernames
ldapsearch -x -h <target> -b "dc=example,dc=com" "(&(objectClass=person)(memberOf=CN=Domain Admins,CN=Users,DC=example,DC=com))"   # domain admins
ldapsearch -x -H ldap://<target> -D "CN=user,DC=example,DC=com" -w 'Password123' -b "dc=example,dc=com" "(objectClass=*)"           # authenticated full enum
```

### Automated (nmap) & tools
```bash
nmap -p 389 --script ldap-brute --script-args ldap.base='"cn=users,dc=CEH,dc=com"' <target>
```
Tools: Softerra LDAP Administrator, AD Explorer, LDAP Admin Tool, LDAP Account Manager, LDAP Search.

## 6. NTP enumeration
NTP syncs clocks over UDP 123. Query to learn connected hosts, client IPs/names/OSs, internal IPs (if NTP in DMZ). Tools: ntpdate, ntptrace, ntpdc, ntpq.

```bash
# ntpdate
ntpdate -q <target>          # query only (offset/delay), no time set
ntpdate -d <target>          # debug (packet exchange)
ntpdate -u <target>          # unprivileged ports (firewall bypass)
sudo ntpdate <target>        # actually sync system time
ntpdate -q <target> | grep offset

# ntptrace — trace the NTP server chain to the primary time source
ntptrace [-n] [-m maxhosts] [server/IP]

# ntpdc (legacy daemon control)
ntpdc -n -c monlist <target>     # recent clients (DDoS amplification vector)
ntpdc -n -c peers <target>       # peers + reachability
ntpdc -n -c sysinfo <target>     # system peer, stratum, precision
ntpdc -n -c version <target>     # NTP software version
ntpdc -n -c reslist <target>     # restriction/ACL list
ntpdc -n -c iostats <target>     # I/O stats

# ntpq (monitor/control)
ntpq -p <target>                 # peers: reachability/delay/offset/jitter
ntpq -c rv <target>              # system variables (time, version, stats)
ntpq -c "rv 0" <target>          # daemon runtime variables
ntpq -c assoc <target>           # association IDs
ntpq -c sysinfo <target>         # uptime/stratum/offset/precision
```
> On many Linux distros ntpd is now Chrony (chronyd). Tools: Nmap, Wireshark, udp-proto-scanner, NTP Server Scanner, PRTG.

## 7. NFS enumeration
NFS shares files over the network (ports 111 + 2049 tcp/udp), relying on RPC/portmap. Check for weak permissions/misconfig → unauthorized read/write/execute. Tools: RPCScan, SuperEnum.
```bash
python3 rpc-scan.py <host/range> --rpc              # list RPC services
python3 rpc-scan.py <host/range> --mounts           # list mountpoints
python3 rpc-scan.py <host/range> --nfs --recurse 3  # recursive NFS share listing
```
**SuperEnum** — automates service enum; checks misconfigured NFS shares, lists exports, tries mounting.

## 8. SMTP enumeration
SMTP sends mail over TCP 25/2525/587 (with POP3/IMAP for retrieval, MX via DNS). Built-in commands: **VRFY** (validate a user), **EXPN** (alias delivery address), **RCPT TO** (recipient address).

```bash
# Nmap
nmap --script smtp-brute -p 25 <target>              # brute SMTP auth
nmap --script smtp-commands -p 25 <target>           # supported commands (fingerprint)
nmap --script smtp-enum-users -p 25 <target>         # users via VRFY
nmap --script smtp-ntlm-info -p 25 <target>          # NTLM info
nmap --script smtp-open-relay -p 25 <target>         # open relay check
nmap --script smtp-strangeport -p <port> <target>    # SMTP on non-standard port
nmap --script smtp-vuln-cve2010-4344 -p 25 <target>  # Exim buffer overflow
nmap --script smtp-vuln-cve2011-1720 -p 25 <target>  # Postfix memory corruption
nmap --script smtp-vuln-cve2011-1764 -p 25 <target>  # Exim DoS
```
```text
# Metasploit auxiliary/scanner/smtp/
smtp_enum        # valid email users
smtp_version     # server version fingerprint
smtp_open_relay  # open relay check
smtp_ntlm_info   # NTLM details
smtp_vrfy        # VRFY user validation
smtp_brute       # brute auth (set USER_FILE / PASS_FILE)
# each: set RHOSTS <target_ip> ; run
```
```bash
# smtp-user-enum  (-M mode: VRFY|EXPN|RCPT, -U userlist, -t host, -p port)
smtp-user-enum -M VRFY -U users.txt -t <target_ip>
smtp-user-enum -M EXPN -U users.txt -t <target_ip>
smtp-user-enum -M RCPT -U users.txt -t <target_ip>
smtp-user-enum -M VRFY -u alice -t <target_ip>                  # single user
smtp-user-enum -M VRFY -U users.txt -t <target_ip> -p 587       # non-standard port
smtp-user-enum -M VRFY -U users.txt -t <target_ip> -v           # verbose
```
Tools: NetScanTools Pro, smtp-user-enum (pentestmonkey).

## 9. DNS enumeration
Zone transfer (AXFR) copies the zone file from primary→secondary DNS; if unsecured, an attacker posing as a client pulls hostnames, IPs, usernames. Tools: nslookup, dig, DNSRecon, ldns-walk, Amass.

### dig
```bash
dig example.com                      # A record (default)
dig example.com ANY                  # all records
dig example.com MX / NS / SOA / TXT / CNAME
dig example.com A +short             # just the IPv4
dig @8.8.8.8 example.com             # query a specific resolver
dig -x 93.184.216.34                 # reverse (PTR)
dig example.com +trace               # full resolution path
dig +noall +answer example.com       # answer section only
dig example.com AXFR @ns1.example.com   # zone transfer attempt
dig example.com +nssearch            # SOA consistency across NS
dig ns <domain>                      # find name servers
dig @<NS> <domain> axfr              # zone transfer
```

### nslookup
```text
nslookup google.com                  # forward lookup
nslookup                             # interactive mode:
  server <dns_ip>
  set type=any|mx|ns|txt|soa|a|aaaa
  ls -d domain.com                   # zone transfer (if allowed)
  ls domain.com > output.txt
nslookup -type=any domain.com <dns_server_ip>   # non-interactive, all records
# zone transfer (interactive): set querytype=soa ; ls -d <NS>
```

### DNSRecon
```bash
dnsrecon -d example.com                 # default enum
dnsrecon -d example.com -t std          # standard records
dnsrecon -d example.com -t brt          # brute subdomains (built-in list)
dnsrecon -d example.com -t axfr         # zone transfer on all NS
dnsrecon -d example.com -t zonewalk     # DNSSEC NSEC walking
dnsrecon -d example.com -t rvl          # reverse lookup
dnsrecon -d example.com -n 8.8.8.8      # custom resolver
dnsrecon -d example.com -a              # full sweep
dnsrecon -d example.com -j output.json  # JSON output
dnsrecon -d example.com -D /path/wordlist.txt -t brt   # custom wordlist brute
```

### DNSSEC zone walking (ldns-walk)
Misconfigured NSEC records expose the full record list.
```bash
ldns-walk example.com                  # NSEC zone walk
ldns-walk -v example.com               # verbose
ldns-walk -f output.txt example.com    # save
ldns-walk -t MX example.com            # filter record type
ldns-walk -n 8.8.8.8 example.com       # custom resolver
ldns-walk @<DNS_IP> <target_domain>    # walk via a given DNS server
```

### OWASP Amass
```bash
amass enum -d example.com                 # passive + active
amass enum -d example.com -passive        # passive only (stealth)
amass enum -d example.com -active -brute  # active + brute
amass enum -d example.com -brute -w wordlist.txt   # custom wordlist
amass enum -d example.com -ip             # subdomains + IPs
amass enum -d example.com -asn            # + ASN info
amass enum -d example.com -src            # show data source
amass enum -d example.com -o subdomains.txt
amass intel -d example.com                # related domains/netblocks
amass track -d example.com                # diff runs
amass viz -d3 -dir amass4owasp            # D3 graph visualization
```

### Nmap DNS
```bash
nmap -sU -p 53 --script=dns-recursion <target>       # recursion (amplification risk)
nmap -T4 -p 53 --script dns-brute <target>           # brute subdomains
nmap --script=dns-zone-transfer <target>             # AXFR
nmap --script=dns-nsid -p 53 <target>                # DNS server fingerprint (NSID)
nmap --script=dns-srv-enum --script-args 'dns-srv-enum.domain=example.com' <target>   # SRV records
nmap --script=broadcast-dns-service-discovery        # mDNS services on LAN
# DNSSEC:
nmap -sU -p 53 --script dns-nsec-enum --script-args dns-nsec-enum.domains=eccouncil.org <target>
nmap -p 53 --script=dns-nsec3-enum <target>          # NSEC3 enum
```
Other tools: Knock, Raccoon, Subfinder, Turbolist3r.

## 10. Other enumeration
### IPsec (ISAKMP/IKE, UDP 500)
```bash
nmap -sU -p 500 <target_ip>                               # IKE service
nmap -sU -p 500 --script ike-version <target_ip>          # IKEv1/IKEv2
nmap -sU -p 500,4500 <target_ip>                          # IKE + NAT-T
# ike-scan
ike-scan <target_ip>                                      # detect VPN responder
ike-scan -M <target_ip>                                   # Main Mode + transforms
ike-scan -A <target_ip>                                   # Aggressive Mode (grab PSK hash)
ike-scan -A --id=examplegroup --pskcrack <target_ip>      # PSK hash for pskcrack
```

### VoIP (SIP, UDP/TCP 2000/2001/5060/5061)
```bash
# svmap (SIPVicious)
svmap 192.168.1.0/24                 # find SIP devices (port 5060)
svmap -p 5060 192.168.1.0/24         # custom port
svmap -v -p 5061 192.168.1.0/24      # SIP over TLS
svmap -o results.txt 192.168.1.0/24  # save
```
```text
# Metasploit auxiliary/scanner/sip/
options        # open SIP ports (set RHOSTS range, THREADS)
enumerator     # valid SIP usernames (set USER_FILE)
sip_brute      # brute SIP creds (set USERNAME, PASS_FILE)
sip_options    # SIP banner/endpoints
# auxiliary/voip/: sip_invite_spoof, sip_digest_leak
```

### RPC (portmapper, port 111)
```bash
nmap -sV -p 111 <target_ip>                               # RPC services + versions
nmap -p 111 --script=rpcinfo <target_ip>                  # registered RPC programs
nmap -p 111 --script=rpc-grind <target_ip>                # enumerate services
nmap -p 111 -sT --script=rpcdump <target_ip>             # dump RPC info (TCP)
```

### Unix/Linux user enumeration
```bash
# rusers (rusersd service)
rusers <target_ip>           # logged-in users
rusers -l <target_ip>        # detailed (idle/login time)
rusers -a <target_ip>        # include down hosts
# rwho (rwhod)
rwho                         # users across the network
rwho -a                      # include idle >1h
rwho | grep <username>       # find a user
# finger (port 79 — deprecated/risky but powerful if open)
finger <username>@<target_ip>    # query a user
finger @<target_ip>              # all logged-in users
finger -l <username>@<target_ip> # long format (.plan/.project)
```

### SMB (TCP 445, or NetBIOS 137/138/139)
```bash
# Nmap
nmap -sV -p 139,445 <target_ip>                           # SMB versions
nmap --script=smb-os-discovery -p 445 <target_ip>         # OS info
nmap --script=smb-protocols -p 445 <target_ip>            # SMBv1/2/3
nmap --script=smb-security-mode -p 445 <target_ip>        # signing/auth
nmap --script=smb-enum-shares -p 445 <target_ip>          # shares (hidden/admin)
nmap --script=smb-enum-users -p 445 <target_ip>           # users
nmap --script=smb-enum-groups -p 445 <target_ip>          # groups
nmap --script=smb-enum-domains -p 445 <target_ip>         # domains
nmap --script=smb-vuln-ms17-010 -p 445 <target_ip>        # EternalBlue
nmap --script=smb-vuln-ms08-067 -p 445 <target_ip>        # MS08-067
nmap --script=smb-vuln-* -p 445 <target_ip>               # all SMB vuln scripts
nmap --script "smb-*" -p 139,445 <target_ip>              # all SMB enum scripts
```
```bash
# SMBMap
smbmap -H <target_ip>                                     # list shares
smbmap -H <target_ip> -u <user> -p <password>             # authenticated
smbmap -H <target_ip> -u '' -p ''                         # anonymous/null
smbmap -H <target_ip> -u <user> -p <password> --shares    # shares + access levels
smbmap -H <target_ip> -u <user> -p <password> -R          # recursive listing
smbmap -H <target_ip> -u <user> -p <password> -r <share>  # one share's contents
smbmap -H <target_ip> -u <user> -p <password> --download  # pull files
smbmap -H <target_ip> -u <user> -p <password> --upload <file>   # push file to writable share
smbmap -H <target_ip> -u <user> -p <password> --execute 'cmd.exe /c whoami'   # run command (if allowed)
```
```bash
# enum4linux
enum4linux <target_ip>                      # basic (users/shares/OS)
enum4linux -U <target_ip>                   # users (RID cycling/SAMR)
enum4linux -S <target_ip>                   # shares
enum4linux -P <target_ip>                   # password policy
enum4linux -G <target_ip>                   # groups
enum4linux -r <target_ip>                   # RID cycling (guest access)
enum4linux -a <target_ip>                   # all modules (most thorough)
enum4linux -A <target_ip>                   # all but NetBIOS shares (faster)
enum4linux -u <user> -p <password> <target_ip>   # authenticated
enum4linux -o output.txt <target_ip>        # save output
```
Also for SMB: nullinux, SMBeagle, NetScanTools Pro.

> Flow per service: open-port scan → pick the enumerator (NetBIOS/SMB → users+shares, SNMP → `public` strings, LDAP → AD users/groups/SPNs, SMTP → VRFY users, DNS → AXFR + subdomains). Try null/anonymous sessions everywhere first. Feed valid usernames into password attacks (Module 06).
