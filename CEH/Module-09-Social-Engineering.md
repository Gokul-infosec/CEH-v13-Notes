# Module 09 – Social Engineering

> Status: ✅ Reference complete

## 1. Goal
Manipulate human trust/error (not technical flaws) to extract confidential info or access. Attacker first gathers intel (websites, ads, blogs, forums), then uses impersonation/tailgating/reverse SE, blending in with social cues (uniforms, politeness).

- **Common targets:** receptionists/help-desk, technical support, system admins, users/clients, vendors, senior executives (high-value data or easy to manipulate).
- **Impact:** economic loss, goodwill damage, loss of privacy, terrorist threats, lawsuits, shutdown.
- **Behaviors exploited:** authority, intimidation, consensus (social proof), scarcity, urgency, familiarity/liking, trust, greed.
- **Company vulnerabilities:** insufficient security training, unregulated access, several org units, lack of security policies.
- **Why effective:** humans are the weakest link; hard to detect; no method/hardware/software fully protects; cheap and easy.
- **Phases:** 1. Research target company → 2. Select a target (often a disgruntled/vulnerable employee) → 3. Develop a relationship → 4. Exploit the relationship.
- **Three types:** human-based, computer-based, mobile-based.

## 2. Human-based techniques
Direct interaction, posing as someone trusted.

- **Impersonation** — pose as a trusted person. 5 variants: legitimate end user, important user (authority/urgency), technical support agent (jargon → ask creds), internal employee/client/vendor (physical entry), repairman (restricted-area access + plant devices).
- **Vishing** (voice phishing) — phone/VoIP + caller-ID spoofing. Tricks: abuse help-desk over-helpfulness, third-party authorization (name-dropping an absent senior), tech support (ask creds to "fix" an issue).
- **Eavesdropping** — secretly listen to/read private comms.
- **Shoulder surfing** — watch someone enter a password/PIN (binoculars/cameras).
- **Dumpster diving** — search trash for phone lists, org charts, email printouts, policy manuals, calendars/logs.
- **Reverse social engineering** — attacker poses as a helper: sabotage (break the system) → marketing (leave contact info) → support (keep "helping" so victim never suspects).
- **Piggybacking** — authorized person knowingly lets attacker in ("forgot my badge").
- **Tailgating** — slip in behind an authorized person (unaware), fake ID.
- **Diversion theft** — mislead a courier/driver to deliver goods/data to the wrong place ("Round the Corner Game").
- **Honey trap** — fake online romance with an insider to extract info.
- **Baiting** — leave a malware USB ("Employee Salary Info 2024") for the curious to plug in.
- **Quid pro quo** — offer fake help (tech support) in exchange for info/running commands.
- **Elicitation** — extract info through casual, friendly conversation.
- **Bait and switching** — lure with a fake offer (cheaper product pop-up) → malicious site/download.

## 3. Computer-based techniques
### Phishing
Fake emails/links → fake site → steal credentials/card details. **Types:**
- **Spear phishing** — targeted at a specific person/small group.
- **Whaling** — targets executives (CEO/CFO), politicians, celebrities.
- **Pharming** — silently redirect even with the correct URL ("phishing without a lure"). Two methods: *DNS cache poisoning* (fake IP in DNS server), *host file modification* (malicious code edits local hosts file).
- **Spimming** — spam over IM (bots harvest usernames).
- **Clone phishing** — copy a real email/site, swap in a malicious link/attachment.
- **E-wallet phishing** — fake digital-wallet login pages.
- **Tabnabbing / reverse tabnabbing** — a background tab silently morphs into a fake login page.
- **Consent phishing** — OAuth-based: fake app requests permissions → account access without a password.
- **Search engine phishing** — SEO-manipulated fake sites ranked high in results.

**Tools:** ShellPhish, Social-Engineer Toolkit (SET — Python, email/web/USB vectors), Dark-Phish, BLACKEYE, SocialFish, Modlishka, Trape, Gophish, King Phisher, Zphisher.

### AI-assisted social engineering
- **ChatGPT phishing emails** — generate realistic, customized urgent emails (fake Microsoft support, IT software-install notice) that bypass basic filters.
- **Impersonate writing style** — train AI on someone's public messages to mimic tone/grammar for fake urgent requests (e.g. fake "father had a heart attack, send money").
- **Deepfake videos** — AI/ML clone a face/voice. Tools: DeepFaceLab, Vidnoz, Deepfakesweb, Synthesia, DeepBrain AI, Hoodem. Needs ML knowledge, programming, training data, video-editing/VFX skills.
- **Voice cloning** — collect recordings → train CNN/RNN → text-to-fake-speech → impersonate in calls. Tools: VEED.IO, Murf.AI, Resemble.AI, ElevenLabs, PlayHT, voice.ai.

### Other computer-based
Pop-up windows (fake alerts → malware), hoax letters, chain letters, instant chat messenger, spam email, scareware (fake AV panic).

### Impersonation on social networks
- **Angler phishing** — pose as a company's support, reply to complainers with malicious links.
- **Catfishing** — steal an identity, build fake relationships for manipulation/money. Signs: avoids direct contact, same profile pic for years, many opposite-gender friends, requests money.
- **Corporate SN threats:** data theft, involuntary data leakage, targeted attacks, network vulnerability, spam/phishing, content modification, malware spread, reputation damage, higher costs, loss of productivity.

### Identity theft
Illegal use of someone's PII (name, SSN, bank/card, DOB, passport, biometric, medical) for fraud. *Identity Theft and Assumption Deterrence Act of 1998.*
- **Types:** child, criminal, financial, driver's-license, insurance, medical, tax, identity cloning/concealment, synthetic (mix real + fake data), social-security.
- **Techniques to obtain PII:** phishing, social engineering, skimming, pretexting, keyloggers/password stealers, dumpster diving & shoulder surfing, social media mining, OSINT searches, wardriving, dark-web data trading.
- **Indications:** unfamiliar charges, missing statements, creditors calling about unknown accounts, traffic violations you didn't commit, charges for medical services you never received, multiple tax returns filed, denied account access, missing utility bills.

## 4. Mobile-based techniques
- **Publishing malicious apps** — fake app with a popular name on app stores → exfiltrates creds/contacts.
- **Repackaging legitimate apps** — add malware to a real app, upload to third-party stores.
- **Fake security applications** — after infecting the PC, a fake bank pop-up pushes a "security app" that steals banking creds + intercepts 2FA SMS.
- **SMiShing (SMS phishing)** — fake urgent texts (bank security team) → call a number / click a link / share details.
- **QRLJacking** — clone a site's login QR code onto a phishing page; victim scans it → login + device info (GPS/IMEI/SIM) sent to attacker → account takeover. Tools: QR TIGER (clone static/dynamic QR + track scans), QR Code Generator, SOTI MobiControl.

> Defense (general): security-awareness training is the only real control — verify identities, enforce policies for password resets and data sharing, shred sensitive documents, badge discipline (no tailgating/piggybacking), scrutinize unsolicited urgent requests, and treat AI-perfect emails/voices as untrusted until verified through a known channel. Authorization in writing before any SE engagement.
