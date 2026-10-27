# Module 02 – Footprinting and Reconnaissance

> Status: ✅ Reference complete

## 1. Goal
Collect information about a target system, network, or organization to identify potential weaknesses — the earliest phase of an attack or pentest.

**Two types of reconnaissance:**
- **Passive** — no direct interaction with the target. OSINT gathering, proprietary/paid databases, intel sharing with partner orgs/industry groups.
- **Active** — direct interaction with the target. DNS interrogation, social engineering, network/port scanning, user & service enumeration.

## 2. Footprinting through search engines
Attackers pull employed tech platforms, employee details, login pages, and intranet portals — feeding social engineering and later attacks.

**Google hacking** = advanced search operators chained into complex queries to surface sensitive/hidden data and vulnerable targets.
- GoogleGuide: https://www.googleguide.com
- Google Hacking Database (GHDB): https://www.exploit-db.com/google-hacking-database

### Google advanced search operators
| Operator | What it does |
|----------|--------------|
| `site:` | Restrict to a site/domain — `games site:www.certifiedhacker.com` |
| `allinurl:` | Pages with all terms in the URL — `allinurl: google career` |
| `inurl:` | Pages with the term in the URL — `inurl: copy site:www.google.com` |
| `intext:` | Keyword in the page body — `intext:"vpn configuration"` |
| `allintitle:` | All terms in the title — `allintitle: detect malware` |
| `intitle:` | Term in the title — `malware detection intitle:help` |
| `inanchor:` | Query terms in anchor text of links — `Anti-virus inanchor: Norton` |
| `allinanchor:` | All terms in anchor text — `allinanchor: best cloud service provider` |
| `cache:` | Google's cached copy — `cache:www.eff.org` |
| `link:` | Pages linking to a URL — `link:www.googleguide.com` (can't combine with keyword search) |
| `related:` | Similar sites — `related:www.microsoft.com` |
| `info:` | Info about a page — `info:gothotel.com` |
| `location:` | Info for a location — `location: 4 seasons restaurant` |
| `filetype:` | By file extension — `filetype:pdf` |
| `source:` | From a source in Google News — `Malware news source:"Hacker News"` |
| `phonebook:` | Residential/business phone numbers — `phonebook: Sundar Pichai` |
| `before:` / `after:` | Filter by publish date — `ransomware before:2020-06-29` · `site:wikipedia.org after:2023-01-01 artificial intelligence` |

### VPN footprinting via GHDB
```text
# login portals (WatchGuard)
inurl:"/sslvpn_logon.shtml" intitle:"User Authentication" "WatchGuard Technologies"
# generic SSL VPN login portals
inurl:/sslvpn/Login/Login
site:vpn.*.*/intitle:"login"
# Zyxel hardcoded-password vuln hosts
inurl:weblogin intitle:("USG20-VPN"|"USG20W-VPN"|USG40|USG40W|USG60|USG60W|USG110|USG210|USG310|USG1100|USG1900|USG2200|"ZyWALL110"|"ZyWALL 310"|"ZYWALL1100"|ATP100|ATP100W|ATP200|ATP500|ATP700|ATP800|VPN50|VPN100|VPN300|VPN000|"FLEX")
# Fortinet VPN login
intext:"Please Login" "SSL VPN" inurl:remote/login intext:FortiClient
# generic VPN login pages
site:vpn.*.*/intext:"login" intitle:"login"
# OpenVPN juicy dirs / keys / configs
intitle:"index of" /etc/openvpn/
"-----BEGIN OpenVPN Static key V1-----" ext:key
intitle:"index of" "vpn-config.*"
"Index of" /*.ovpn
# Netscaler / Citrix Gateway portals
inurl:"/vpn/tmindex.html" vpn
# Cisco ASA login pages
intitle:"SSL VPN Service" + intext:"Your system administrator provided the following information to help understand and remedy the security conditions:"
```

### FTP server dorks
```text
# juicy index-of FTP
site:.in .com | .net intitle:"index of" ftp
intitle:"index of" "*/ftp.txt"
intext:"index of" "ftp"
inurl:WS_FTP.log
intitle:index.of/cftp/robots.txt
# passwords
intitle:"Index of ftp passwords"
# detect web server
inurl:/ftp intitle:"office"
# login portals
inurl:/web-ftp.cgi
site:sftp.*.*/ intext:"login" intitle:"server login"
# ws_ftp.ini (FTP user creds)
intitle:"Index of" ws_ftp.ini
# archived email conversations
inurl:ftp -inurl:(http|https) intext:"@gmail.com" intext:subject fwd confidential|important|CARD|cvv
# CrushFTP web interface (login + password reset)
allintitle:"CrushFTP WebInterface"
# sensitive logs
"ws_ftp.log" ext:log
"index of" /ftp/logs
# Monsta FTP
intitle:"Monsta ftp" intext:"Lock session to IP"
# admin folders on FTP
intitle:"index of" inurl:ftp intext:admin
```

### FTP search engines
NAPALM FTP Indexer (https://www.searchftps.net) · FreewareWeb FTP File Search · Mamont (https://www.mmnt.ru/int) · Globalfilesearch.com

## 3. Device / OS discovery
- **Shodan** (https://www.shodan.io) — search engine for internet-connected devices; footprint VoIP, VPN, vulnerable devices/networks.
- **Censys** (https://censys.io) — device/host discovery.
- **Netcraft** (https://www.netcraft.com/tools) — determine OS and server tech.

Also: Google Advanced Search (/advanced_search), Advanced Image Search, Reverse Image Search (/imghp).

## 4. Internet research services
Online sources giving publicly accessible info on the target (domains, subdomains, hosts).
- **Netcraft** (https://www.netcraft.com) · **DNSdumpster** (https://dnsdumpster.com) · **Pentest-Tools** (https://pentest-tools.com)
- **archive.org Wayback Machine** (https://archive.org) — browse archived versions of a site.
- **People search**: Spokeo, Intelius, Pipl — location, emails, sites, contacts, addresses. Job sites (Dice, LinkedIn, Glassdoor) reveal infrastructure details.

### Dark web footprinting
Darknet = deeper layer of cyberspace for anonymous, untraceable navigation. Tools: **Tor Browser** (https://www.torproject.org — acts as default VPN, bounces IP through several servers) and **ExoneraTor**.

Advanced dark-web search parameters:
```text
filetype:pdf site:onion confidential        # confidential PDFs
inurl:config filetype:txt password           # passwords in config files
filetype:xlsx site:onion financial           # financial Excel files
filetype:sql site:onion dump                 # SQL database dumps
filetype:csv site:onion email                # email lists
intitle:"login credentials" filetype:docx    # login-cred docs
filetype:xml inurl:config server             # server configs
filetype:key site:onion private              # private keys
filetype:pdf site:onion "medical records"    # medical records
filetype:ppt site:onion "business plan"      # business plans
filetype:py site:onion "def"                 # Python source
filetype:docx site:onion "legal document"    # legal docs
filetype:pdf site:onion "bank statement"     # bank statements
filetype:pdf inurl:patent confidential       # patents / IP
filetype:txt inurl:exploit                   # exploit notes
```

## 5. Competitive intelligence gathering
Identifying/gathering/analyzing/verifying info about competitors — non-interfering and subtle.
**Sources:** company websites & job ads, social engineering of employees, search engines/online DBs, product catalogs, press releases & annual reports, analyst/regulatory reports, trade journals/conferences, customer/vendor interviews, patents/trademarks, agents/distributors/suppliers.

**Info resource sites:**
- Company intel: EDGAR (https://www.sec.gov/edgar), D&B Hoovers, LexisNexis, Business Wire, Factiva (dowjones.com)
- Company plans: MarketWatch, The Wall Street Transcript, Euromonitor, Experian, The Search Monitor, USPTO
- Expert opinions: SEMRush, ABI/INFORM Global (ProQuest), SimilarWeb, SE Ranking
- Other: Google Earth (geolocation), Google Finance, OpenCorporates (business profiles), Google Alerts (monitoring), Mention (online reputation)

## 6. Footprinting through social networking sites
- **Sherlock** (https://github.com/sherlock-project/sherlock) — Python tool; hunts a username across many social sites.
  ```bash
  sherlock "Elon Musk"
  ```
- **theHarvester** (https://www.kali.org/tools/theharvester) — OSINT in early pentest stages; enumerate LinkedIn employees/titles, emails.
  ```bash
  theharvester -d microsoft.com -l 200 -b linkedin          # up to 200 emails/usernames from LinkedIn
  theharvester -d eccouncil.org -l 200 -b baidu             # from Baidu
  theharvester -d microsoft.com -l 200 -b baidu -f Microsoft_email.xml   # save to XML
  ```
- **Social Searcher** (https://www.social-searcher.com) — real-time social content search + deep analytics; track a user across sites.
- **BuzzSumo** (https://buzzsumo.com) — most-shared content by topic/author/domain across major networks.

## 7. WHOIS footprinting
WHOIS DBs (maintained by RIRs) hold domain-owner personal info.
- Lookups: https://whois.domaintools.com · tamos.com · sabsoft.com

**WHOIS query returns:** domain name details, owner contact details, name servers, NetRange, creation date, expiry, last-updated record.
**Helps attacker:** gather info for social engineering, map the target network, obtain internal network details.

**Regional Internet Registries (RIRs):** ARIN (arin.net), AFRINIC (afrinic.net), APNIC (apnic.net), RIPE NCC (ripe.net), LACNIC (lacnic.net).

**IP geolocation** (IP2Location — https://www.ip2location.com, IP Location Finder): country, region, city, ZIP, timezone, connection speed, ISP, domain, country/area code, mobile carrier, elevation — fuels spamming/phishing.

## 8. DNS footprinting
DNS records reveal server locations and types → key hosts → social engineering.

| Record | Description |
|--------|-------------|
| A | Host's IP address |
| MX | Domain's mail server |
| NS | Host's name server |
| CNAME | Alias to a host |
| SOA | Authority for a domain |
| SRV | Service records |
| PTR | Maps IP → hostname |
| RP | Responsible person |
| HINFO | CPU type and OS |
| TXT | Unstructured text |

**Interrogation tools:** nslookup (nslookup.io), SecurityTrails, DNSChecker (dnschecker.org), zdns (github.com/zmap/zdns), dnsdumpster, **Fierce** (kali.org/tools/fierce):
```bash
fierce --domain certifiedhacker.com                                  # basic recon — subdomains & hosts
fierce --domain certifiedhacker.com --subdomains write admin mail    # check specific subdomains
fierce --domain certifiedhacker.com --subdomains mail --traverse 10  # adjacent subdomains, 10 brute steps
fierce --domain certifiedhacker.com --subdomains mail --connect      # check connectivity/open ports
fierce --domain certifiedhacker.com --wide                           # wide/deep scan
```

**Reverse DNS lookup** — resolve IP ranges back to DNS records. Tools: DNSRecon (github.com/darkoperator/dnsrecon), Reverse Lookup (mxtoolbox.com), puredns, Reverse IP Domain Check, Reverse IP Lookup.

## 9. Network and email footprinting
**ARIN.net** — find who owns an IP; map the target network without touching it.

**Traceroute** — works on ICMP + the TTL field to discover routers on the path to a host.
```bash
tracert <target>              # Windows — ICMP by default
sudo tcptraceroute <target>   # Linux — TCP (many nets block ICMP traceroute)
traceroute <target>           # UDP (Windows/Linux)
```
Tools: NetScanTools Pro (ICMP/UDP/TCP traceroute + geolocate hops), PingPlotter (graphs latency/packet loss over time), Traceroute NG (SolarWinds).

**Email tracking** — extract sender identity, mail server, sender IP, location from headers; track the path to the target mail server. Tools: eMailTrackerPro, IP2Location Email Header Tracer, MxToolbox, Holehe, DNS Checker Email Header Analyzer, Social Catfish.

## 10. Footprinting through social engineering
Non-technical manipulation to get confidential info from a person unaware they're leaking it.
- **Eavesdropping** — unauthorized interception of audio/video/text.
- **Shoulder surfing** — secretly observing to grab passwords, PINs, account/card numbers.
- **Dumpster diving** — phone bills, contacts, financial/operational info, sticky notes from trash.
- **Impersonation** — pose as a legitimate/authorized person to trick targets into revealing info.
- Others: tailgating, third-party authorization, piggybacking, reverse social engineering.

## 11. Advanced & AI-powered OSINT tools
**Classic frameworks/tools:**
- **Maltego** (maltego.com) — map real-world links between people, orgs, sites, infrastructure, documents.
- **Recon-ng** (github.com/lanmaster53/recon-ng) — modular web recon framework with DB interaction.
- **FOCA** (elevenpaths.com) — metadata/hidden info in Office/OpenOffice/PDF documents.
- **SubFinder** (github.com/projectdiscovery/subfinder) & **Sudomy** — subdomain discovery.
- **OSINT Framework** (osintframework.com) — categorized OSINT tool tree. Markers: (T) local tool, (D) Google dork, (R) needs registration, (M) edit URL manually.
- **Recon-Dog** (github.com/s0md3v/ReconDog), **BillCipher** (DNS/Whois/portscan/zone transfer/reverse IP), whatweb, Raccoon, Orb, Web Check (web-check.xyz), OSINT.SH.

**AI-powered OSINT:**
- **Taranis AI** (taranis.ai) — NLP/AI news & unstructured-article analysis into reports.
- **OSS Insight** (ossinsight.io) — analyzes 5B+ GitHub events for repo/developer insights.
- **DorkGPT** (dorkgpt.com) / **DorkGenius** (dorkgenius.com) / **Google Word Sniper** — AI-generated Google dorks.
- **Cylect.io** — multi-database AI OSINT interface.
- **ChatPDF** (chatpdf.com) — conversational extraction from PDFs.
- **Bardeen.ai** — automate OSINT data collection.
- **DarkGPT** (github.com/luijait/DarkGPT) — GPT-4-200K queries against leaked databases.
- **PenLink Cobwebs** — gather/analyze/visualize data for investigations.
- **Explore AI** — AI YouTube search engine.
- **AnyPicker** — no-code visual web scraper / AI OSINT.

> Flow: passive OSINT (search engines + dorks → Shodan/Censys/Netcraft → archive.org/people search) → WHOIS + IP geolocation → DNS (records, Fierce, reverse DNS) → network/email (traceroute, header tracing) → social engineering → pull it together in Maltego/Recon-ng. Stay passive until scope allows active interaction.
