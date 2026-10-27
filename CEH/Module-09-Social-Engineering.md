# Module 09 – Social Engineering

> Status: ✅ Reference complete

## 1. Social engineering concepts
Technique to trick people into revealing confidential information (passwords, access details) by exploiting **human behavior, not technical flaws**. The attacker first gathers intel from company websites, ads, blogs, and forums to learn about employees and the organization, then manipulates people (impersonation, tailgating, reverse SE). Victims often don't realize they shared sensitive info — answering a casual question, replying to a spam email. Attackers study the environment (security staff, receptionists, help desks) and use social cues (uniforms, polite behavior) to avoid suspicion. Goal: exploit human trust and error for unauthorized access.

### Common targets
- **Receptionists & help-desk staff** — tricked by posing as a customer/employee needing help; trained to assist, so may leak passwords/contact details.
- **Technical support executives** — attacker poses as senior staff/client/vendor requesting confidential technical info.
- **System administrators** — targeted to steal admin credentials, OS details, and data for deeper attacks.
- **Users & clients** — attacker poses as tech support / trusted entity.
- **Vendors** — exploited for intel on internal systems / business operations.
- **Senior executives** (Finance, HR, leadership) — hold highly sensitive data; impersonate trusted contacts to trick them.

### Impact on an organization
Economic losses (stolen business secrets/strategies), damage to goodwill/reputation, loss of privacy (exposed customer/partner data), terrorist threats (intel for physical/cyber attack), lawsuits & arbitration, temporary or permanent shutdown.

### Behaviors vulnerable to attacks (psychological levers)
- **Authority** — pose as someone important (network admin) to extract passwords.
- **Intimidation** — pressure/threats ("something bad happens unless you act now").
- **Consensus / social proof** — fake reviews/testimonials for malware disguised as helpful software.
- **Scarcity** — fake urgency ("limited-time iPhone offer") → click harmful links.
- **Urgency** — rush victims (ransomware countdown timers) so they panic.
- **Familiarity / liking** — act friendly/familiar to bypass security.
- **Trust** — pose as expert/company rep, guide victim into downloading malware.
- **Greed** — offer fake rewards/money (contests, bribes) for confidential info.

### Factors that make companies vulnerable
Insufficient security training; unregulated access to information (too many people, no limits); several organizational units (multi-location = harder to manage); lack of security policies (passwords, data sharing, user access).

### Why social engineering is effective
Humans are the most susceptible variable despite policies; attempts are hard to detect; no method guarantees complete security; no hardware/software fully safeguards against it; it's cheap (often free) and easy to implement.

### Phases of a social engineering attack
1. **Research the target company** — website, business type, employees (dumpster diving, online searches).
2. **Select a target** — a specific person, often a disgruntled or vulnerable employee who's easier to trick.
3. **Develop a relationship** — build trust posing as a friend/colleague/trusted contact.
4. **Exploit the relationship** — manipulate the victim into sharing sensitive data (account info, technologies used, future plans).

### Three categories
**Human-based** (phone scams, in-person), **computer-based** (phishing emails, pop-ups), **mobile-based** (fake apps, SMS scams).

## 2. Human-based social engineering
Direct interaction with employees while posing as someone trustworthy (e.g. IT support tech). Techniques: impersonation, vishing, eavesdropping, shoulder surfing, dumpster diving, reverse SE, piggybacking, tailgating, diversion theft, honey trap, baiting, quid pro quo, elicitation, bait and switch.

### Impersonation
Pretend to be someone trusted (courier, technician, visitor) — in person, by phone, or other channels. Once inside: look for passwords on desks, search bins, eavesdrop, shoulder surf. Exploits trust, fear, and moral obligation. Types:
1. **Posing as a legitimate end user** — pretend to be a regular employee (finance team) asking for help ("forgot" password); uses friendliness/sympathy.
2. **Posing as an important user** — pretend to be a manager/director; uses authority + urgency, sometimes threats.
3. **Posing as a technical support agent** — claim a system issue, use jargon to confuse, ask for login creds to "fix" it.
4. **Posing as an internal employee/client/vendor** — physically enter as a contractor/visitor; harvest desk passwords, conversations, trash.
5. **Posing as a repairman** — dress as electrician/phone tech; access restricted areas, search for data or plant spying devices.

### Vishing (voice phishing)
Phone/VoIP calls tricking people into giving bank details, card numbers, login creds; often **caller-ID spoofing** to appear as a bank. Victim gets a fake SMS/email/call asking them to call a number; pre-recorded "official" messages ask for personal info. Tricks:
- **Abusing help-desk over-helpfulness** — pose as a pressured employee (missed deadline) with an emotional story → staff reset passwords without verifying.
- **Third-party authorization** — claim to be sent by an unavailable senior employee (on vacation); name-drop confidently.
- **Tech support** — claim a technical issue, ask for creds to "fix" it.

### Other human-based techniques
- **Eavesdropping** — secretly listen to/read private conversations (calls, emails, chats) to steal passwords, business details, phone numbers, addresses.
- **Shoulder surfing** — watch someone enter a password/PIN directly or via binoculars/hidden cameras.
- **Dumpster diving** — search trash for passwords, bank statements, source code, personal details; may pose as repair/cleaning staff. Finds: phone lists (impersonation targets), org charts (structure, server/restricted zones), email printouts/notes/faxes/memos (passwords, processes), policy manuals (system rules), event notes/calendars/logs (when users are active/away → best time to attack).
- **Reverse social engineering** — attacker first creates/sabotages a problem, then poses as the helpful expert so the victim contacts *them*. Techniques: **sabotage** (break the system), **marketing** (place contact info on a fake error message/business card), **support** (keep "helping" after stealing data so the victim never realizes).
- **Piggybacking** — entry with the authorized person's consent ("forgot my ID badge", common courtesy opens the door).
- **Tailgating** — sneak in by following closely behind an authorized person (exploit door-holding politeness); may wear a fake ID.
- **Diversion theft** ("Round the Corner Game") — mislead a delivery worker/driver into sending goods to a fake location; online, trick someone into sending files to the wrong person.
- **Honey trap** — pose online as an attractive person, start a fake relationship to extract info from an insider who holds critical data.
- **Baiting** — leave a tempting malware-laden USB ("Employee Salary Information 2024") in elevators/parking lots; exploits curiosity/greed.
- **Quid pro quo** — offer fake help (pose as tech support, random calls) in exchange for info; trick the victim into running commands / installing malware.
- **Elicitation** — extract info through casual friendly conversation, steering toward usernames/passwords without raising suspicion.
- **Bait and switch** — lure with fake offers (links/downloads "too good to be true") → malware/data theft/fake site.

## 3. Computer-based social engineering
Uses computers and Internet systems. Techniques: phishing, spam mail, instant chat messenger, pop-up windows, scareware, deepfake videos, voice cloning.

### Phishing
Fake emails/links posing as trusted sites to steal personal info; a fake website that looks real captures card numbers, addresses, etc. Works because people don't notice the fake site, ignore warnings, or lack awareness. Types:
- **Spear phishing** — targeted at a specific person/small group with convincing emails from a "trusted source" (manager/colleague); higher success rate.
- **Whaling** — targets high-level individuals (CEOs, CFOs, politicians, celebrities) with carefully crafted emails/sites.
- **Pharming** ("phishing without a lure") — silently redirect the victim to a fake site even after they type the correct URL; via **DNS cache poisoning** (modify the DNS server so the real URL resolves to the attacker's IP) or **host-file modification** (malicious email attachment edits local hosts file). Can also use Trojans/worms.
- **Spimming** — spam over Instant Messaging; a *spimmer* uses bots to collect IM usernames and push ads/malicious links.
- **Clone phishing** — copy a real email/site the victim already received, swap the link/attachment for a malicious one.
- **E-wallet phishing** — fake emails from "trusted e-wallet providers" → fake sites steal wallet login/financial details.
- **Tabnabbing / reverse tabnabbing** — exploit multiple open tabs. *Tabnabbing*: a malicious page waits until the user switches tabs, then rewrites itself as a real login page. *Reverse tabnabbing*: a link opens a new tab while the original tab is replaced with a fake page.
- **Consent phishing** — abuse OAuth; a fake but legit-looking app asks the victim to grant permissions → attacker gets account data (emails, contacts) without a password.
- **Search engine phishing** — SEO manipulation / keyword stuffing pushes fake sites high in results.

**Phishing tools:** generate fake login pages, send spoofed emails, capture IP + session cookies. ShellPhish (Instagram/Facebook/Twitter/LinkedIn creds + victim IP/browser/host/geo), **Social-Engineer Toolkit (SET)** (open-source Python pentest tool; categorizes email/web/USB attack vectors). Others: Dark-Phish, BLACKEYE, SocialFish, Modlishka, Trape, Gophish, King Phisher, LUCY Security, Zphisher.

### AI-assisted social engineering
- **Crafting phishing emails with ChatGPT** — AI mimics tone/style to produce convincing, filter-evading emails using urgency/offers. (Example prompts: pose as Microsoft support warning of a suspicious login + reset link; pose as IT admin requiring a security-software install by a deadline.)
- **Impersonate writing style** — train AI on a target's public emails/posts to copy tone, word choice, grammar, then send fake messages (e.g. a fake "father had a heart attack, send money urgently" message in the target's style). AI is consistent and scalable but lacks true human understanding.
- **Deepfake videos** — AI/ML (CNNs, GANs) clone a face/voice from old clips/recordings for disinformation, impersonation, blackmail. Needs ML knowledge, programming, training data, data prep, video-editing + VFX skills (color grading, motion tracking, rotoscoping). Tools: DeepFaceLab, Vidnoz, Deepfakesweb, Synthesia, DeepBrain AI, Hoodem.
- **Voice cloning** — speech synthesis + neural nets copy a voice (tone, style). Steps: collect voice recordings from public sources → train deep models (CNN/RNN) → generate text-to-speech samples → impersonate in calls/messages. Tools: VEED.IO, Murf.AI, Resemble.AI, ElevenLabs, PlayHT, voice.ai.

### Other computer-based techniques
- **Pop-up windows** — fake alerts/login prompts → install Trojans/keyloggers/spyware.
- **Hoax letters** — fake virus warnings that waste time/resources.
- **Chain letters** — promise rewards / threaten bad luck unless forwarded.
- **Instant chat messenger** — casual chat to extract personal info.
- **Spam email** — collect info / deliver malicious attachments (disguised with long filenames).
- **Scareware** — fake antivirus pop-ups create panic → download harmful software.

### Impersonation on social networking sites
Fake profiles exploit the personal info people share. Two forms:
- **Angler phishing** — pose as a company's customer support, reply to complaint posts with "helpful" malicious links.
- **Catfishing** — steal an identity from Facebook/Instagram, build fake relationships for bullying, manipulation, or money scams. Signs: avoids direct communication (no calls/webcam, emergency excuses), keeps one profile picture for years, many opposite-gender "friends", requests for money.

**Social networking threats to corporate networks:** data theft, involuntary data leakage, targeted attacks, network vulnerability (platform bugs), spam & phishing, content modification (page/profile spoofing), malware spread, reputation damage, higher monitoring costs, loss of productivity.

### Identity theft
Illegally using another person's PII (name, SSN, bank details) to commit fraud. (Identity Theft and Assumption Deterrence Act of 1998.) **PII stolen:** name, address, SSN, phone, DOB, passport #, biometric data, bank account #, card info, credit report, driving-license #, medical/health-insurance info. **Uses:** open credit cards / phone accounts, obtain utility services, open bank accounts for bogus checks, clone ATM/debit cards, obtain loans in the victim's name.

**Types:** child, criminal, financial, driver's-license, insurance, medical, tax, identity cloning & concealment, synthetic (mix real + fake data), social-security.

**Techniques to obtain PII:** phishing, social engineering, skimming (ATM/card-reader skimmers), pretexting, keyloggers/password stealers, dumpster diving & shoulder surfing, social-media mining, OSINT internet searches, wardriving (unsecured Wi-Fi), dark-web data trading.

**Indications of identity theft:** unfamiliar card charges; statements stop arriving; creditors call about unknown accounts; traffic violations you didn't commit; charges for medical services you never received; more than one tax return under your name; denied access to your own accounts/loans; utility bills stop arriving (stolen mail).

## 4. Mobile-based social engineering
Fake versions of popular apps uploaded to stores with similar names. Techniques:
- **Publishing malicious apps** — e.g. a gaming app with attractive features under a popular name; once installed, sends creds/contacts to the attacker.
- **Repackaging legitimate apps** — add hidden malware to a real app, upload to third-party stores; silently collects data.
- **Fake security applications** — after infecting the victim's computer, a fake bank pop-up tells them to install a "security app"; it steals banking creds and intercepts the SMS 2FA code → account takeover.
- **SMiShing (SMS phishing)** — urgent fake texts from "banks/companies" → click a malicious link, call a fake number, or share info (e.g. fake "XIM Bank security team" asking to call and share card number/password); also fake prize messages.
- **QRLJacking** — target QR-code login systems. Attacker grabs a real login QR, clones it onto an auto-updating phishing page; victim scans it → login details + device info (GPS, IMEI, SIM) sent to attacker → full account access. Tools: **QR TIGER** (clones static/dynamic QR codes, tracks scans/time/location/device), QR Code Generator, Soti MobiControl.

## 5. Countermeasures
- **People:** security-awareness training on all three categories; verify caller/visitor identity through a known channel (call back a listed number); never share passwords; escalate unusual "urgent" requests.
- **Help desk / password resets:** strict identity verification before any reset; no password resets or disclosure over the phone without callback verification.
- **Policy:** classify data and enforce least-privilege access; clear password, data-sharing, and acceptable-use policies; shred sensitive documents (anti–dumpster diving); clean-desk policy.
- **Physical:** badge-controlled access, anti-tailgating (mantraps/turnstiles), challenge unescorted strangers, secure disposal bins.
- **Technical:** email/anti-phishing gateways + SPF/DKIM/DMARC; MFA to blunt stolen credentials (prefer phishing-resistant factors); block/flag external email; endpoint protection; disable autorun to counter baiting USBs; URL/attachment sandboxing.
- **Social media / OSINT:** limit what employees publish about roles/projects; corporate social-media account hardening.
- **Identity theft:** monitor statements and credit reports, fraud alerts/credit freezes, secure disposal of PII, report indications quickly.
- **Detection & response:** phishing-simulation exercises, incident reporting culture (no blame), monitor for impersonation/typosquatting domains.

> Core idea: social engineering attacks the human, not the firewall. Defense is layered — awareness training + verification procedures + least privilege + MFA + physical controls — because no single tool stops a convincing human (or AI-generated) pretext.
