# Module 01 – Introduction to Ethical Hacking

> Status: ✅ Reference complete

## 1. Goal
Establish the foundational language, principles, and legal/operational framework of ethical hacking — the security concepts, hacker classifications, attack models, methodologies, controls, and standards every later module builds on. The whole principle is to avoid **Theft, Tampering, and Disruption** of systems through the **CIA Triad**.

## 2. Fundamental security concepts (CIA Triad + extended)

| Property | Meaning |
|----------|---------|
| **Confidentiality** | Keeping systems and data from being accessed, seen, or read by anyone not authorized. Information is accessible only to authorized personnel. |
| **Integrity** | *Trustworthiness of data or resources.* Protect data from modification or deletion by unauthorized parties, and ensure that when authorized people make changes that shouldn't have been made, the damage can be undone. |
| **Availability** | *Accessible when required by authorized users.* Systems, access channels, and authentication mechanisms must all be working properly so the information they provide and protect is available when needed. |
| **Authenticity** | The characteristic of a communication, document, or any data that ensures the quality of being genuine. |
| **Non-Repudiation** | Assurance that someone cannot deny the validity of something. A legal concept: a service that provides proof of the **origin** of data and the **integrity** of the data. |

**Auditing & Accountability** — keep tracking of everything: who's logging in, when they're logging in, whose data they're accessing.

> Note: In addition to CIA, other properties such as authenticity, accountability, non-repudiation, and reliability can also be involved (ISO/IEC 27000:2009).

### Security, Functionality, and Usability balance
There is an interdependency between these three attributes. **When security goes up, usability and functionality come down.** Any organization should balance between these three qualities to arrive at a balanced information system.

## 3. Types of hackers

| Type | Description |
|------|-------------|
| **Black Hat** | Hackers that seek to perform malicious activities. |
| **Gray Hat** | Perform good or bad activities but do **not** have the permission of the organization they are hacking against. |
| **White Hat** | Ethical hackers; use their skills to improve security by exposing vulnerabilities before malicious hackers do. |
| **Script Kiddie / Skiddies** | Unskilled individual who uses malicious scripts or programs (e.g., a web shell) developed by others to attack systems/networks and deface websites. |
| **State-Sponsored Hacker** | Hacker hired by a government or a related entity. |
| **Hacktivist** | Someone who hacks for a cause; political agenda. |
| **Suicide Hackers** | Not afraid of going to jail or facing punishment; hack to get the job done. |
| **Cyberterrorist** | Motivated by religious or political beliefs to create fear or disruption. |

## 4. Hacking vocabulary

| Term | Meaning |
|------|---------|
| **Hack value** | Perceived value or worth of a target as seen by the attacker. |
| **Vulnerability** | A system flaw or weakness (in design, implementation, etc.). |
| **Threat** | Exploits a vulnerability. |
| **Exploit** | A way of gaining access to a system through a security flaw and taking advantage of it for benefit. |
| **Payload** | Component of an attack; the part of the private user text that could contain malware (worms, viruses) performing the malicious action — deleting data, sending spam, encrypting data. |
| **Zero-day attack** | Attack that occurs before a vendor knows of or is able to patch a flaw. |
| **Daisy Chaining / Pivoting** | Gaining access to a network/computer and then using that same information to gain access to multiple networks and computers containing desirable information. |
| **Doxxing** | Publishing PII about an individual, usually with malicious intent. |
| **EISA** | Enterprise Information Security Architecture — determines the structure and behavior of an organization's information systems through processes, requirements, principles, and models. |

## 5. Threat categories

**Network Threats**
- Information gathering
- Sniffing and eavesdropping
- DNS/ARP Poisoning
- MITM (Man-in-the-Middle Attack)
- DoS/DDoS
- Password-based attacks
- Firewall and IDS attack
- Session Hijacking

**Host Threats**
- Password cracking
- Malware attacks
- Footprinting
- Profiling
- Arbitrary code execution
- Backdoor access
- Privilege Escalation
- Code Execution

**Application Threats**
- Injection Attacks
- Improper data/input validation
- Improper error handling and exception management
- Hidden-field manipulation
- Broken session management
- Cryptography issues
- SQL injection
- Phishing
- Buffer Overflow
- Information disclosure
- Security Misconfigurations

## 6. Attack vectors
Path by which a hacker can gain access to a host in order to deliver a payload or malicious outcome.

- **APT (Advanced Persistent Threats)** — a stealthy threat actor, typically a nation state or state-sponsored group, that gains unauthorized access to a network and remains undetected for an extended period; typically uses zero-day attacks.
- **Cloud computing / Cloud-based technologies** — a flaw in one client's application could allow an attacker to access another client's data.
- **Viruses, worms, and malware** — the most prevalent networking threat; capable of infecting a network within seconds.
- **Ransomware** — restricts access to the system's files/folders and demands an online ransom payment to remove the restrictions.
- **Mobile Device threats**
- **Botnets** — huge network of compromised systems used by an intruder to perform various network attacks (used for DDoS).
- **Insider attacks** — a disgruntled employee can damage assets from inside.
- **Phishing attacks**
- **Web Application Threats** — SQL injection, XSS (Cross-site scripting), etc.
- **IoT Threats**

## 7. Attack types
1. **Operating System** — attacks targeting OS flaws or security issues such as guest accounts or default passwords.
   - Vectors: Buffer overflows, protocol implementations, software defects, patch levels, authentication schemes.
2. **Application Level** — attacks on programming code and software logic.
   - Vectors: Buffer overflows, bugs, XSS, DoS, SQL injection, MitM.
3. **Misconfiguration** — takes advantage of systems misconfigured due to improper or default configuration.
   - Examples: Improper permissions of SQL users; access-list permit all.
4. **Shrink-Wrap Code** — exploiting holes in unpatched or poorly configured software.
   - Examples: Software defect in version 1.0; defect in example CGI scripts; default passwords.

## 8. Vulnerabilities

**Scoring & databases**
- **CVSS – Common Vulnerability Scoring System** — places a numerical score based on severity.
- **CVE – Common Vulnerabilities and Exposures** — a list of publicly disclosed vulnerabilities and exposures, maintained by **MITRE**.
- **NVD – National Vulnerability Database** — a database maintained by **NIST**, fully synchronized with the MITRE CVE list; the US Gov. vulnerabilities repository.

**Vulnerability categories**
- **Misconfiguration** — improperly configuring a service or application.
- **Default installation** — failure to change default settings in an application.
- **Buffer overflow** — code execution flaw.
- **Missing patches** — systems that have not been patched.
- **Design flaws** — flaws inherent to system design such as encryption and data validation.
- **Operating System Flaws** — flaws specific to each OS.
- **Default passwords** — leaving default passwords that ship with a system/application.

## 9. Pen Test phases (CEH)
- **Pre-Attack Phase** — reconnaissance and data-gathering.
- **Attack Phase** — attempts to penetrate the network and execute attacks.
- **Post-Attack Phase** — cleanup to return a system to the pre-attack condition and deliver reports.

> ⚠️ For the exam, EC-Council brings its own methodology and that's all you need for the exam. For professional pentesting beyond certification content, the **OSSTMM** (Open Source Security Testing Methodology Manual) is recommended.

## 10. The Five Stages of Ethical Hacking
1. **Reconnaissance** — gathering evidence about targets. Two types:
   - **Passive Reconnaissance** — gain information without direct interaction (Google search, public records, news releases, social media, wardriving to scan networks around).
   - **Active Reconnaissance** — direct interaction with the target (phone call, job interview; tools like Nmap, Nessus, OpenVAS, Nikto, and Metasploit count as active recon).
2. **Scanning & Enumeration** — obtaining more in-depth information (network scanning, port scanning, which versions of services are running).
3. **Gaining Access** — attacks are leveled to gain access.
   - Can be done locally (offline), over a LAN, or over the internet.
   - Spoofing to exploit the system by pretending to be a legitimate user/different system; send a data packet containing a bug to exploit a vulnerability.
   - Techniques: command injection, buffer overflow, DoS, brute forcing credentials, social engineering, misconfigurations, etc.
4. **Maintaining Access** — items put in place to ensure future access (rootkit, trojan, backdoor).
5. **Covering Tracks** — conceal success and intrusion to avoid being noticed (clear the logs; obfuscate trojans or malicious backdoor programs).

## 11. Three types of active defense
- **Annoyance** — tracking a hacker and leading him into a fake server, wasting his time and making him easy to detect.
- **Attribution** — identify an attacker; use tools to trace the source of an attack back to a location or an individual.
- **Attack** — the most controversial. To "hack back," a company accesses an alleged hacker's computer to delete its data or take revenge. Both of these steps are considered **illegal**.

## 12. Information Assurance (IA)
Assurance of the Integrity, Availability, Confidentiality, and Authenticity of information and information systems during usage, processing, storage, and transmission.

**Processes that help achieve IA:**
- Developing local policy, process, and guidance.
- Designing network and user authentication strategy.
- Identifying network vulnerabilities and threats (vulnerability assessments outline the security posture of the network).
- Identifying problems and resource requirements.
- Creating a plan for identified resource requirements.
- Applying appropriate IA controls.
- Performing the **C&A (Certification and Accreditation)** process of information systems to trace vulnerabilities and implement safety measures.
- Providing information assurance training to all personnel in federal and private orgs.

## 13. Information Security Management Program
Combination of policies, processes, procedures, standards, and guidelines to establish the required level of information security. Designed to ensure the business operates in a state of reduced risk; encompasses all organizational and operational processes and participants relevant to information security.

> ⚠️ **IA** focuses on risk assessment and the mitigation side of things. ⚠️ **InfoSec** focuses on actually implementing security measures to safeguard systems.

### EISA – Enterprise Information Security Architecture
Set of requirements, processes, principles, and models that determines the structure and behavior of an organization's information systems.

**Goals of EISA:**
- Help in monitoring and detecting network behaviors.
- Detect and recover from security breaches.
- Prioritize resources of an organization.
- Help perform risk assessment of an organization's IT assets.
- Cost perspective when incorporated in security provisions such as incident response, disaster recovery, event correlation, etc.

## 14. Physical security controls

| Control | Purpose | Examples |
|---------|---------|----------|
| **Preventive** | Deters the actor from performing the threat | Fence, server locks, mantraps |
| **Detective** | Recognizes an actor's threat | Background check, CCTV |
| **Deterrent** | Deters the actor from attempting the threat | Warning sign |
| **Recovery** | Mitigates the impact of a manifested threat | Backups |
| **Compensating** | Provides alternative fixes to any of the above functions | — |

Most security controls are **preventive** phase controls.

> ⚠️ **Defense in Depth** — multiple layers of security controls; provides redundancy in the event of a control failure.

### Types of security controls (by nature)
| Type | Examples |
|------|----------|
| **Physical** | Guards, lights, cameras, fire extinguishers, flood protection |
| **Administrative** | Training awareness, policies, procedures, and guidelines for infosec |
| **Technical** | IDS/IPS, firewall, encryption, smart cards, access control lists |

### Types of security controls (by function)
| Type | Examples |
|------|----------|
| **Preventative** | Authentication, alarm bells |
| **Detective** | Audits, backups |
| **Corrective** | Restore operations |

## 15. Managing the risk
Risk = probability of the occurrence of a threat or event that may damage, cause loss, or have other negative impact, from internal or external liabilities.

**Risk matrix** — used during risk assessment to define the level of risk by considering the category of probability/likelihood against the category of consequence severity. A simple mechanism to increase visibility of risks and assist management decision-making.

### Phases of Risk Management
- **Risk Identification** — identifies the sources, causes, and consequences of internal and external risks.
- **Risk Assessment** — assesses the org's risk and estimates the likelihood and impact of the risk.
- **Risk Treatment** — selects and implements appropriate controls on the identified risks.
- **Risk Tracking** — ensures appropriate controls are implemented and identifies the chance of a new risk occurring.
- **Risk Review** — evaluates the performance of the implemented risk management strategies.

## 16. Threat modeling
A risk assessment approach for analyzing the security of an application by capturing, organizing, and analyzing all information that affects its security.

- **Identify Objectives** — determine how much effort to put into subsequent steps.
- **Application Overview** — identify the components, data flows, and trust boundaries.
- **Decompose Application** — find more relevant details on threats.
- **Identify Threats** — identify threats relevant to the control scenario and context using the info from the previous two steps.
- **Identify Vulnerabilities** — identify weaknesses related to the threats found, using vulnerability categories.

## 17. Security policies
**Policies** — high-level statements about protecting information; business rules to safeguard the CIA triad. Can be applied to Users, Systems, Partners, Networks, and Providers.

**Common security policy examples:**
- **Password Policy** — meet complexity requirements (e.g., min 8 chars, upper and lower case, alphanumeric).
- **Wireless Security Policy**
- **AUP – Acceptable Use Policy** — how to properly use company assets (do's and don'ts with company computers).
- **Data Retention Policy** — keep the data for X time.
- **Access Control Policies** — accessing servers; firewalls.

**Procedures** — set of detailed steps to accomplish a goal; instructions for implementation.
**Guidelines** — advice on actions given a situation; recommended, not mandatory.

### Security policy examples (detailed)
- **Access Control Policy** — defines the resources being protected and the rules that control access to them.
- **Remote Access Policy** — who can have remote access; defines access medium and remote access security controls.
- **Firewall Management Policy** — access, management, and monitoring of firewalls.
- **Network Connection Policy** — who can install new resources, approve new devices, document network changes.
- **Password Policy** — guidelines for strong password protection.
- **User Account Policy** — account creation process, authority, rights, and responsibility of user accounts.
- **Information Protection Policy** — sensitivity levels of information, who may access, how it is stored/transmitted/deleted.
- **Special Access Policy** — terms and conditions of granting special access to system resources.
- **Email Security Policy** — governs proper usage of corporate email.
- **Acceptable Use Policy** — defines the acceptable use of system resources.

### Security policy types
- **Promiscuous Policy** — no restrictions on usage of system resources.
- **Permissive Policy** — begins wide open; only known dangerous services/attacks/behaviors are blocked. Must be updated regularly to stay effective.
- **Prudent Policy** — maximum security while allowing known but necessary dangers. Blocks all services; only safe/necessary services are enabled individually. Everything is logged.
- **Paranoid Policy** — forbids everything. No Internet connection or severely restricted Internet usage allowed.

### Security policy – creation steps
1. Perform a risk assessment.
2. Use security standards and frameworks as a guide.
3. Get management and staff input.
4. Enforce the policy. Use penalties for non-compliance.
5. Publish the final draft to the entire org.
6. Have all staff read/sign that they understood the policy.
7. Employ tools to help enforce the policy.
8. Staff training.
9. Review and update regularly.

## 18. Incident management process
An **incident** is an event that could lead to loss of, or disruption to, an organization's operations, services, or functions. **Incident management** describes the activities of an organization to identify, analyze, and correct hazards to prevent future re-occurrence.

- **Preparation** — select people, assign rules, define tools to handle the incident.
- **Detection & Analysis** — determine an incident has occurred (IDS, SIEM, AV, someone reporting, etc.).
- **Classification and Prioritization**
- **Notification** — identify minor vs major incident; who and how to notify.
- **Containment** — limit the damage; isolate hosts; contact system owners.
- **Forensic Investigation** — investigate the root cause using forensic tools; system logs, real-time memory, network device logs, application logs.
- **Eradicate & Recovery** — remove the cause; patch if needed. Recovery: get back into production; monitor affected systems.
- **Post-incident Activities** — document what happened and why; transfer knowledge.

### Incident Response Team duties
- Managing security issues by taking a proactive approach toward the customer's security vulnerabilities.
- Developing or reviewing processes and procedures that must be followed.
- Managing the response to an incident and ensuring all procedures are followed correctly to minimize and control damage.
- Identifying and analyzing what happened during an incident, including impact and threat.
- Providing a single point of contact for reporting security incidents and issues.
- Reviewing changes in legal and regulatory requirements to ensure all processes/procedures are valid.
- Reviewing existing controls and recommending steps and technologies to prevent future incidents.
- Establishing relationships with local law enforcement, gov. agencies, key partners, and suppliers.

## 19. SIEM – Security Information and Event Management
Collects data points from the network — log files, traffic captures, SNMP messages, etc. — from every host, into one centralized location and correlates it for analysis to look for security and performance issues and negative trends, all in real time.

- **Aggregation** — collecting data from disparate sources and organizing it into a single format. Any device that collects data is called a **collector** or **aggregator**.
- **Correlation** — the logic that looks at data from disparate sources and makes determinations about events on the network (in-band or out-of-band, depending on placement of the NIDS/NIPS).
  - **Alerts** — notification if something goes bad.
  - **Triggering** — exceeding thresholds.
- **Normalization** — creates multiple tables / organizes data so it becomes more efficient and analysis/report tools work better.
- **WORM (Write Once Read Many)** — log files are precious; for archival viewing, optical media like WORM drives store them so they can't be altered.

**Most popular SIEM tools:** Splunk · ArcSight · ELK (Elasticsearch, Logstash, Kibana — open source).

## 20. Identity and Access Management (IAAA)
Identification, Authentication, Authorization, and Accounting work together to manage assets securely.

1. **Identification** — credentials identify the user (name, username, ID number, employee number, SSN).
2. **Authentication** — "prove you are the legitimate user." Should always be done with Multifactor Authentication.
   - **Authentication factors:**
     - Something you **know** (password)
     - Something you **have** (smart card)
     - Something you **are** (fingerprint)
     - Something you **do** (android pattern; manual signature)
     - Somewhere you **are** (geolocation)
   - 🛑 MFA generally uses **two** of these, **never from the same category** (e.g., something you know + something you have).
3. **Authorization** — what you are allowed to access. Uses Access Control models; implementation depends on the org and its security goals.
   - **Permissions** — applied to resources.
   - **Rights / Privileges** — assigned at system level.
   - **Authorization strategies:** Least privilege, Separation of Duties.
4. **Accounting** — trace an action to a subject's identity; prove who/what performed a given action (non-repudiation); logging.

### Access control models
- **Mandatory Access Control (MAC)** — every object gets a label (Confidential, Secret, Top Secret). The administrator decides who gets access to what security level; users cannot change these settings. Used on old systems (e.g., Top Secret Gov. information).
- **Discretionary Access Control (DAC)** — used in most OSes. Owner of the data defines access. Very flexible; very weak security.
- **Role-Based Access Control (RBAC)** — access defined by rules based on a role in the org/job function (Manager, Director). Admins provide access based on the user's role; rights are gained implicitly instead of explicitly. In Windows, use Groups to provide RBAC (Admin Group → rights/perms, Sales Group → rights/perms).

> ⚠️ Access is defined by **ACL (Access Control List)**. ⚠️ **Implicit deny** prevents access unless specifically permitted.

## 21. Data Loss Prevention (DLP)
The practice of detecting and preventing data breaches, exfiltration, or unwanted destruction of sensitive data. Organizations use DLP to protect/secure data and comply with regulations; it defends against both data **loss** and data **leakage**.

**Organizations typically use DLP to:**
- Protect PII and comply with relevant regulations.
- Protect Intellectual Property critical to the organization.
- Achieve data visibility in large organizations.
- Secure a mobile workforce and enforce security in BYOD environments.
- Secure data on remote cloud systems.

## 22. Data backup
Data backup plays a crucial role in business continuity, helping orgs recover from IT disasters, security breaches, application failures, human error, etc. Regulatory compliance (COBIT, SSAE, SOC II, PCI-DSS, HIPAA, SOX, FINRA, FISMA, GDPR, etc.) requires businesses to maintain backups of critical data for a specified duration.

**Backup strategies:**
- Identify the critical business data.
- Select the backup media.
- Select a backup technology.
- Select the appropriate RAID levels.
- Select an appropriate backup method.

### 3 backup site methods
- **Cold backup 🔵** — empty site, no hardware, no data, no people. Takes weeks to bring online. Basic office space; no operational equipment. Cheapest recovery site.
- **Warm backup 🟡** — between cold and hot (big room with rack space, you bring the hardware). Hardware ready and waiting; you bring software and data. Takes days to bring online; operational equipment but little or no data.
- **Hot backup 🔴** — exact replica of production systems; applications/software constantly updated. Flip a switch and everything moves. Takes hours to bring online; real-time synchronization; almost all data ready. Very expensive.

## 23. Penetration test – basics
An authorized simulated cyberattack on a computer system, performed to evaluate the security of the system. ⚠️ Not to be confused with a **vulnerability assessment**. A clearly defined, full-scale test of security controls.

**Phases:**
- **Preparation** — contracts and team determined.
- **Assessment** — all hacking phases (reconnaissance, scanning, attacks, etc.).
- **Post-Assessment** — reports & conclusions.

**Types:**
- **Black Box** — done without any knowledge of the system or network.
- **White Box** — attacker has complete knowledge of the system, provided by the owner/target.
- **Gray Box** — attacker has some knowledge of the system and/or network.

## 24. Law categories
- **Criminal** — laws that protect public safety; usually have jail time attached.
- **Civil** — private rights and remedies.
- **Common** — laws based on societal customs.

## 25. Laws and standards

### OSSTMM compliance
"Open Source Security Testing Methodology Manual," maintained by **ISECOM**; defines three types of compliance:
- **Legislative** — government regulations (SOX, HIPAA).
- **Contractual** — industry/group requirement (PCI DSS).
- **Standards based** — practices that must be followed by members of a group/organization (ITIL, ISO, OSSTMM itself).

### OSSTMM controls
**Class A – Interactive Controls**
- **Authentication** — identification and authorization based on credentials.
- **Indemnification** — contractual protection against loss or damages.
- **Subjugation** — ensures interactions occur according to processes defined by the asset owner.
- **Continuity** — maintains interactivity with assets if corruption/failure occurs.
- **Resilience** — protects assets from corruption and failure.

**Class B – Process Controls**
- **Non-repudiation** — prevents participants from denying their actions.
- **Confidentiality** — ensures only participants know of an asset.
- **Privacy** — ensures only participants have access to the asset.
- **Integrity** — ensures participants know when assets and processes change.
- **Alarm** — notifies participants when interactions occur.

### Regulations & frameworks reference

| Standard | Description |
|----------|-------------|
| **PCI-DSS** | Payment Card Industry Data Security Standard — for organizations handling credit cards, ATM cards, and other POS cards. |
| **ISO 27001** | Requirements for establishing, implementing, maintaining, and continually improving an ISMS. |
| **ISO 27002 / 17799** | Based on BS7799; focuses on security objectives and provides security controls based on industry best practice. |
| **HIPAA** | Health Insurance Portability and Accountability Act — privacy standards to protect patient medical records and health info shared between doctors, hospitals, and insurers. |
| **SOX** | Sarbanes-Oxley Act — requires publicly traded companies to submit to independent audits and properly disclose financial information. |
| **DMCA** | Digital Millennium Copyright Act (1998, US) — implements two 1996 WIPO treaties; criminalizes production/dissemination of tech intended to circumvent access controls on copyrighted works. |
| **FISMA** | Federal Information Security Modernization Act of 2002 (updated 2004) — codifies DHS authority over information security policy implementation (for gov. agencies). |
| **NIST-800-53** | Catalogs security and privacy controls for federal information systems; helps implement FISMA. |
| **FITARA** | Federal Information Technology Acquisition Reform Act (2013) — changes the framework for how the US Gov purchases technology. |
| **COBIT** | Control Objectives for Information and Related Technology — IT governance framework/toolset created by ISACA and ITGI. |
| **GLBA** | US Gramm-Leach-Bliley Act — protects confidentiality and integrity of personal information collected by financial institutions. |
| **CSIRT** | Computer Security Incident Response Team — single point of contact when reporting computer security incidents. |
| **ITIL** | Information Technology Infrastructure Library — operational framework (developed in the '80s) that standardizes IT management procedures. |

## 26. Essential knowledge — OSI & TCP/IP models
The **OSI Model** is a reference/logical model describing communication functions by dividing the procedure into smaller, simpler components. The **TCP/IP model** is a concise version with four layers instead of seven.

| Layer | Device Type | OSI Layer | TCP/IP Model | TCP/IP (actual) | Protocols | PDU |
|-------|-------------|-----------|--------------|-----------------|-----------|-----|
| 7 | Gateway | Application | Application | Application | HTTP, FTP, POP, SMTP, DNS, RIP | Data |
| 6 | - | Presentation | Application | Application | HTTP, FTP, POP, SMTP, DNS, RIP, MIME | Data |
| 5 | - | Session | Application | Application | HTTP, FTP, POP, SMTP, DNS, RIP, SCP | Data |
| 4 | - | Transport | Transport | Transport | TCP/UDP | Segments |
| 3 | Router | Network | Internet | Network | IP, ARP, ICMP, IGMP | Packets |
| 2 | Switch/bridge | Data Link | Link | Data Link | Ethernet, Token Ring | Frames |
| 1 | Hubs/Repeater | Physical | Link | Physical | Ethernet, Token Ring | Bits |

## 27. TCP handshake

**✅ TCP connection establishment (three-way handshake)**
1. Host A sends a **SYN** packet with a proposed initial sequence number to Host B.
2. Host B receives the SYN and returns a packet with both **SYN and ACK** flags set.
3. Host A receives the SYN-ACK and sends back an **ACK** packet.
4. Host B receives the ACK — the connection is **ESTABLISHED**.

**❌ TCP connection termination**
1. Host A sends a **FIN** flag, indicating it has finished sending data.
2. Host B receives the FIN, enters a "passive close" (**CLOSE_WAIT**) state, and sends an ACK for the FIN back to Host A.
3. Host A enters a **TIME_WAIT** state and sends an ACK back to Host B.
4. Host B gets the ACK and closes the connection.

> ⚠️ Sequence numbers increase on new communication. Example: A increments B's sequence number; A never increments its own sequence.

### TCP flags
| Flag | Name | Function |
|------|------|----------|
| **SYN** | Synchronize | Set during initial communication; negotiates parameters and sequence numbers. |
| **ACK** | Acknowledgment | Set as acknowledgement to the SYN flag; always set after initial SYN. |
| **RST** | Reset | Forces termination of a connection (in both directions). |
| **FIN** | Finish | Ordered close to communications. |
| **PSH** | Push | Forces delivery of data without concern for buffering. |
| **URG** | Urgent | Data inside is sent out of band (e.g., cancelling a message). |

## 28. Port numbers
**IANA** (Internet Assigned Numbers Authority) maintains the Service Name and Transport Protocol Port Number Registry.

**Ranges:**
- **Well-known ports** — 0–1023
- **Registered ports** — 1024–49,151
- **Dynamic ports** — 49,152–65,535

| Port | Protocol | Transport |
|------|----------|-----------|
| 20/21 | FTP | TCP |
| 22 | SSH | TCP |
| 23 | Telnet | TCP |
| 25 | SMTP | TCP |
| 53 | DNS | TCP/UDP |
| 67 | DHCP | UDP |
| 69 | TFTP | UDP |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 135 | RPC | TCP |
| 137-139 | NetBIOS | TCP/UDP |
| 143 | IMAP | TCP |
| 161/162 | SNMP | UDP |
| 389 | LDAP | TCP/UDP |
| 443 | HTTPS | TCP |
| 445 | SMB | TCP |
| 514 | SYSLOG | UDP |

- A service is **listening** for a port when it has that specific port open.
- Once a service has made a connection, the port is in an **established** state.

**Netstat command:**
```
netstat -an    # displays connections in numerical form
netstat -b     # displays executables tied to the open port (admin only)
```

## 29. Subnetting

**IPv4 main address types:**
- **Unicast** — acted on by a single recipient.
- **Multicast** — acted on by members of a specific group.
- **Broadcast** — acted on by everyone on the network.
  - **Limited** — delivered to every system in the domain (255.255.255.255).
  - **Directed** — delivered to all devices on a subnet using that broadcast address.

**Subnet mask** — determines how many addresses are available on a specific subnet. Represented three ways:
- **Decimal** — 255.240.0.0
- **Binary** — 11111111.11110000.00000000.00000000
- **CIDR** — x.x.x.x/12

- If all bits in the host field are **1s**, the address is the **broadcast**.
- If they are all **0s**, it's the **network address**.
- Any other combination indicates an address in the range.
