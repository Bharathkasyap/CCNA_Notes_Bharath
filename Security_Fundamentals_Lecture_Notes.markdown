# Security Fundamentals Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture introduces **security fundamentals** for the **CCNA exam** (topic 5.1: "Describe key security concepts"), covering essential network and system security concepts for IT professionals. Although the CCNA is not a cybersecurity certification, understanding these fundamentals is critical. The lecture defines the **CIA triad**, common **attacks**, **passwords** and **multi-factor authentication (MFA)**, **AAA (Authentication, Authorization, Accounting)**, and **security program elements**. It provides a broad overview of security topics, introducing new terms for beginners to memorize and apply.

**Lecture Objectives**:
- **Understand CIA Triad**: Learn Confidentiality, Integrity, Availability as the foundation of security.
- **Identify Attacks**: Recognize common attacks (DoS, spoofing, MITM, reconnaissance, malware, social engineering, password attacks) and their impact on CIA.
- **Grasp Authentication**: Explore passwords, MFA (something you know, have, are), and digital certificates.
- **Learn AAA Framework**: Differentiate Authentication, Authorization, and Accounting, including RADIUS and TACACS+.
- **Understand Security Programs**: Cover user awareness, training, and physical access control.
- **Prepare for Exam**: Master topic 5.1 concepts for CCNA (no configuration required).
- **Reinforce Prior Content**: Connect to TCP (Day 15), DHCP (Day 13), ARP (Day 18).

**Network Topology** (for attack examples):
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      SRV1      |
| 10.0.0.11      |       |                |       | 10.0.0.1       |
| (Client)       |       |                |       | (Server)       |
+----------------+       +----------------+       +----------------+
                       |
                       |       +----------------+
                       +-------|   Attacker     |
                               | 10.0.0.99     |
                               +----------------+
```
- **Devices**:
  - **PC1**: Client (10.0.0.11), sends traffic to SRV1 (e.g., ARP requests, HTTP).
  - **SW1**: Switch, forwards traffic (e.g., broadcasts ARP requests).
  - **SRV1**: Server (10.0.0.1), responds to PC1 (e.g., ARP replies, DHCP offers).
  - **Attacker**: Malicious device (10.0.0.99), performs attacks (e.g., ARP spoofing, SYN flood).
- **Note**: Topology used for MITM (ARP spoofing), DoS (SYN flood, DHCP exhaustion), etc. Attacker compromises CIA (Confidentiality, Integrity, Availability).
- **Requirements**:
  1. Define CIA triad and map attacks to CIA components.
  2. Understand vulnerabilities, exploits, threats, mitigations.
  3. Explain MFA and AAA (Authentication, Authorization, Accounting).
  4. Identify security program elements (awareness, training, physical access).
  5. Verify with flashcards (e.g., CIA, attack types, AAA protocols).

**Analogy**: Security is like protecting a house: **CIA triad** (lock doors for confidentiality, secure valuables for integrity, ensure access for availability), **attacks** (burglars breaking windows, faking IDs), **MFA** (key + PIN), **AAA** (verify ID, grant room access, log entries), and **security programs** (train residents, install locks).

**Lifelong Retention Tip**: Visualize PC1 → SRV1 with Attacker intercepting (MITM). Create flashcards: “CIA? Confidentiality, Integrity, Availability. SYN flood? DoS, availability. MFA? Know, have, are.” Simulate ARP spoofing in Packet Tracer. Review 3 times daily for a week.

## CIA Triad and Security Concepts

**Purpose**: Establish the foundation of security (CIA triad) and define key terms (vulnerability, exploit, threat, mitigation).

**Key Points**:
- **CIA Triad**:
  - **Confidentiality**: Only authorized users access data (e.g., restrict SRV1’s sensitive files to PC1, not Attacker).
  - **Integrity**: Data remains unmodified by unauthorized users (e.g., Attacker can’t alter PC1’s messages to SRV1).
  - **Availability**: Systems are accessible to authorized users (e.g., SRV1 is operational for PC1’s requests).
- **Security Concepts** (CCNA topic 5.1):
  - **Vulnerability**: Potential weakness (e.g., open ports on SRV1, weak passwords).
  - **Exploit**: Method to leverage vulnerability (e.g., malware exploiting open ports).
  - **Threat**: Possibility of exploit (e.g., Attacker using malware to access SRV1).
  - **Mitigation**: Protection against threats (e.g., firewall on SRV1, strong passwords).
- **Key Notes**:
  - No system is 100% secure; aim to reduce risk (e.g., antivirus on PC1 reduces malware risk but not to 0).
  - Mitigations apply to devices (PC1, SRV1, SW1) and physical access (secure server rooms).
  - Example: A window (vulnerability) can be broken by a rock (exploit); a burglar using the rock (threat) is stopped by bars (mitigation).

**Demonstration**:
- **Topology**: PC1 (10.0.0.11) → SW1 → SRV1 (10.0.0.1), Attacker (10.0.0.99).
- **Scenario**:
  - Attacker tries to access SRV1’s confidential files (breach Confidentiality).
  - Attacker modifies PC1’s data to SRV1 (breach Integrity).
  - Attacker floods SRV1, blocking PC1’s access (breach Availability).
- **Mitigations**:
  - Firewall on SRV1 (blocks unauthorized access).
  - Data validation (ensures integrity).
  - Anti-DoS measures (maintains availability).
- **Verification** (conceptual, no config for CCNA):
  - Check SRV1’s access logs for unauthorized attempts.
  - Simulate in Packet Tracer: Attacker sends malicious traffic, mitigated by firewall.

**Comparison to Prior Content**:
- **Day 15 (TCP/UDP)**: SYN flood (DoS) exploits TCP handshake; UDP vulnerable to reflection attacks.
- **Day 13 (DHCP)**: DHCP exhaustion (spoofing) depletes SRV1’s IP pool.
- **Day 18 (ARP)**: ARP spoofing (MITM) alters PC1’s ARP table.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Spoofing uses fake IPs (e.g., Attacker fakes 10.0.0.1).
- **Day 30 (Ports)**: Reconnaissance probes open ports (e.g., TCP 80 on SRV1).
- **Day 44 (NAT)**: Firewalls with NAT mitigate external attacks.

**Analogy**: CIA triad is like a safe: lock it (Confidentiality), ensure contents are unchanged (Integrity), and keep it accessible to owners (Availability). Vulnerabilities (weak lock), exploits (lockpick), threats (thief), and mitigations (alarm) protect it.

**Lifelong Retention Tip**: Draw CIA triad (C-I-A triangle) and map attacks to each. Flashcards: “Confidentiality? Authorized access. Threat? Exploit of vulnerability. Mitigation? Firewalls, antivirus.” Simulate TCP SYN flood in Packet Tracer. Review 3 times.

## Common Attacks

**Purpose**: Identify major attack types (DoS, spoofing, reflection/amplification, MITM, reconnaissance, malware, social engineering, password attacks) and their impact on CIA.

**Key Points**:
- **Denial-of-Service (DoS)**:
  - **Target**: Availability (CIA’s A).
  - **Example**: **TCP SYN Flood**:
    - Attacker sends SYN packets to SRV1, spoofing IP (e.g., 1.2.3.4).
    - SRV1 responds with SYN-ACK, waits for ACK (never sent).
    - SRV1’s TCP connection table fills, blocking PC1’s connections.
  - **DDoS**: Distributed DoS, uses botnet (multiple infected PCs) to amplify attack.
  - **Impact**: SRV1 unavailable to PC1.
- **Spoofing**:
  - **Target**: Availability (e.g., DHCP exhaustion) or Confidentiality/Integrity (e.g., MITM).
  - **Example**: **DHCP Exhaustion**:
    - Attacker spoofs MAC addresses, sends DHCP Discover to SRV1.
    - SRV1’s DHCP pool (e.g., 250 IPs) depletes, denying PC1 an IP.
  - **Impact**: Availability (DoS) or Confidentiality/Integrity (if combined with MITM).
- **Reflection/Amplification**:
  - **Target**: Availability.
  - **Example**: **DNS Amplification**:
    - Attacker spoofs SRV1’s IP (10.0.0.1), sends small DNS query to reflector (8.8.8.8).
    - Reflector sends large DNS reply to SRV1, overwhelming it.
  - **Impact**: SRV1 flooded, unavailable to PC1.
- **Man-in-the-Middle (MITM)**:
  - **Target**: Confidentiality, Integrity.
  - **Example**: **ARP Spoofing**:
    - PC1 sends ARP request for SRV1’s MAC (10.0.0.1).
    - SW1 broadcasts; SRV1 and Attacker receive.
    - SRV1 replies with its MAC; Attacker sends fake ARP reply with Attacker’s MAC.
    - PC1’s ARP table maps 10.0.0.1 to Attacker’s MAC.
    - PC1’s traffic to SRV1 goes to Attacker, who reads/modifies it.
  - **Impact**: Attacker reads (Confidentiality) or alters (Integrity) PC1–SRV1 traffic.
- **Reconnaissance**:
  - **Target**: Information gathering for future attacks (not direct CIA impact).
  - **Example**: NSLOOKUP (jeremysitlab.com → 10.0.0.1), WHOIS (company contacts), port scanning (open TCP 80).
  - **Impact**: Enables targeted attacks (e.g., phishing using WHOIS data).
- **Malware**:
  - **Target**: Confidentiality, Integrity, Availability.
  - **Types**:
    - **Virus**: Infects software (e.g., SRV1’s app), spreads via sharing.
    - **Worm**: Standalone, self-spreading (e.g., congests network).
    - **Trojan**: Disguised as legitimate (e.g., fake app on PC1).
  - **Impact**: Corrupts files (Integrity), steals data (Confidentiality), or slows network (Availability).
- **Social Engineering**:
  - **Target**: People (Confidentiality, Integrity).
  - **Examples**:
    - **Phishing**: Fake email (e.g., “Amazon account locked”) tricks PC1 user into entering credentials.
    - **Spear Phishing**: Targeted (e.g., company employees).
    - **Whaling**: Targets executives (e.g., SRV1 admin).
    - **Vishing**: Phone-based (e.g., fake IT call to PC1 user).
    - **Smishing**: SMS-based phishing.
    - **Watering Hole**: Malicious link on trusted site (e.g., jeremysitlab.com).
    - **Tailgating**: Attacker follows authorized user into secure area.
  - **Impact**: Steals credentials (Confidentiality) or tricks user into actions (Integrity).
- **Password Attacks**:
  - **Target**: Confidentiality, Integrity.
  - **Types**:
    - **Guessing**: Manual guess (rare, weak passwords).
    - **Dictionary Attack**: Tries common words/passwords.
    - **Brute Force**: Tries all combinations (time-intensive, needs strong computing).
  - **Impact**: Attacker accesses PC1/SRV1 accounts (Confidentiality, Integrity).
- **Strong Passwords**:
  - ≥8 characters, mix of uppercase/lowercase, numbers, special characters (e.g., P@ssw0rd123!).
  - Change regularly (e.g., every 90 days).

**Demonstration**:
- **Topology**: PC1 → SW1 → SRV1, Attacker (10.0.0.99).
- **Scenarios**:
  - **SYN Flood**: Attacker floods SRV1 with SYN packets, fills TCP table.
  - **ARP Spoofing**: Attacker overwrites PC1’s ARP table, intercepts traffic.
  - **Phishing**: Attacker emails PC1 user, steals SRV1 credentials.
- **Mitigations**:
  - SYN Flood: Firewall filters excessive SYNs.
  - ARP Spoofing: Dynamic ARP Inspection (DAI) on SW1.
  - Phishing: User awareness training.
- **Verification** (conceptual):
  - Packet Tracer: Simulate ARP spoofing, observe Attacker intercepting traffic.
  - Check SRV1 logs for failed logins (password attacks).

**Comparison to Prior Content**:
- **Day 15 (TCP/UDP)**: SYN flood exploits TCP handshake; reflection uses UDP (e.g., DNS, NTP).
- **Day 13 (DHCP)**: DHCP exhaustion targets SRV1’s pool.
- **Day 18 (ARP)**: ARP spoofing enables MITM.
- **Syslog Lab**: Attacks trigger Syslog (e.g., `%SEC-6-IPACCESSLOGP` for port scans).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Spoofing fakes IPs/MACs.
- **Day 30 (Ports)**: Reconnaissance scans ports (e.g., TCP 80).
- **Day 44 (NAT)**: Firewalls with NAT block external attacks.

**Analogy**: Attacks are like burglary tactics: DoS (block driveway), spoofing (fake ID), MITM (eavesdrop via window), reconnaissance (scout house), malware (plant bugs), social engineering (trick owner), password attacks (pick lock).

**Lifelong Retention Tip**: Simulate ARP spoofing in Packet Tracer, map attacks to CIA (e.g., SYN flood → Availability). Flashcards: “SYN flood? DoS, Availability. ARP spoofing? MITM, Confidentiality/Integrity. Phishing? Social engineering.” Review 3 times.

## Passwords and Multi-Factor Authentication (MFA)

**Purpose**: Enhance security beyond passwords using MFA and digital certificates.

**Key Points**:
- **Passwords**:
  - Weakness: Usernames (e.g., email) easy to guess; passwords vulnerable to attacks.
  - **Strong Password Rules**:
    - ≥8 characters (preferably >8).
    - Mix uppercase/lowercase, numbers, special characters (e.g., S3cur!ty2025).
    - Change regularly (e.g., every 90 days).
  - **Attacks**: Guessing, dictionary, brute force (mitigated by strong passwords).
- **Multi-Factor Authentication (MFA)**:
  - Requires ≥2 factors:
    - **Something You Know**: Password, PIN.
    - **Something You Have**: Phone app notification, badge.
    - **Something You Are**: Biometrics (fingerprint, retina, face scan).
  - Example: Login with password (know) + phone app (have).
  - **Not MFA**: Two biometrics (e.g., retina + fingerprint, both “are”).
  - Benefit: Blocks attackers even with stolen passwords.
- **Digital Certificates**:
  - Verify website identity (e.g., jeremysitlab.com).
  - Process: Website sends CSR to CA; CA issues signed certificate.
  - Browser shows lock icon for valid certificate (e.g., SRV1’s HTTPS site).
  - **Impact**: Prevents MITM by ensuring SRV1 is legitimate.

**Demonstration**:
- **Topology**: PC1 → SRV1 (HTTPS server).
- **Scenario**:
  - PC1 accesses https://jeremysitlab.com (10.0.0.1).
  - Browser verifies SRV1’s certificate (lock icon).
  - PC1 user logs in with password + phone app (MFA).
- **Attack**:
  - Attacker tries phishing to steal PC1’s password; MFA blocks access.
  - Attacker fakes SRV1’s site; certificate mismatch alerts PC1.
- **Verification**:
  - Packet Tracer: Simulate HTTPS, check certificate (conceptual).
  - Observe MFA prompt (password + app).

**Comparison to Prior Content**:
- **Day 15 (TCP/UDP)**: HTTPS (TCP 443) uses certificates.
- **QoS Part 2 Lab**: HTTPS prioritized (AF31); MFA secures it.
- **Syslog Lab**: Failed logins trigger Syslog (e.g., `%SEC-6-LOGIN_FAILED`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: HTTPS (443) for certificate-based authentication.
- **Day 44 (NAT)**: Firewalls protect certificate servers.
- **Day 10 (IP Addressing)**: Certificates tied to IPs (e.g., 10.0.0.1).

**Analogy**: Passwords are like a single key; MFA adds a second lock (badge, fingerprint). Certificates are like an ID card proving SRV1’s identity.

**Lifelong Retention Tip**: Simulate HTTPS login in Packet Tracer, note lock icon. Flashcards: “MFA? Know, have, are. Certificate? Verifies website. Strong password? ≥8, mixed characters.” Practice 3 times.

## AAA Framework

**Purpose**: Control and monitor system access using Authentication, Authorization, Accounting.

**Key Points**:
- **AAA Definitions**:
  - **Authentication**: Verifies user identity (e.g., PC1 user’s password + MFA).
  - **Authorization**: Grants permissions (e.g., PC1 user accesses SRV1 files, not admin settings).
  - **Accounting**: Logs actions (e.g., PC1 user’s login time, file changes).
- **AAA Servers**:
  - Example: Cisco ISE (Identity Services Engine).
  - Centralize AAA for devices (PC1, SRV1, SW1).
- **Protocols**:
  - **RADIUS**: Open standard, UDP 1812 (authentication), 1813 (accounting).
  - **TACACS+**: Cisco proprietary, TCP 49.
  - **Exam Note**: Memorize protocol names and ports.
- **Use Case**:
  - PC1 user logs into SRV1: Authentication (password + MFA), Authorization (file access), Accounting (logs login time).

**Demonstration**:
- **Topology**: PC1 → SRV1, with ISE server (hypothetical).
- **Scenario**:
  - PC1 user authenticates via RADIUS to ISE (password + phone app).
  - ISE authorizes file access, denies admin access.
  - ISE logs login time, accessed files.
- **Verification** (conceptual):
  - Check ISE logs for PC1’s actions.
  - Packet Tracer: Simulate login, observe UDP 1812 traffic.

**Comparison to Prior Content**:
- **Day 15 (TCP/UDP)**: RADIUS (UDP 1812/1813), TACACS+ (TCP 49).
- **Syslog Lab**: AAA accounting logs trigger Syslog (e.g., `%AAA-3-ACCT`).
- **QoS Part 2 Lab**: AAA secures prioritized traffic (e.g., HTTPS).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: RADIUS (1812/1813), TACACS+ (49).
- **Day 10 (IP Addressing)**: AAA server has dedicated IP.
- **Day 44 (NAT)**: AAA servers behind NAT for security.

**Analogy**: AAA is like a building security system: Authentication (ID check), Authorization (room access), Accounting (visitor log).

**Lifelong Retention Tip**: Simulate RADIUS login in Packet Tracer. Flashcards: “AAA? Authentication, Authorization, Accounting. RADIUS? UDP 1812/1813. TACACS+? TCP 49.” Review 3 times.

## Security Program Elements

**Purpose**: Implement policies to secure systems and educate staff (user awareness, training, physical access control).

**Key Points**:
- **User Awareness**:
  - Educates employees on threats (e.g., phishing).
  - Example: Fake phishing emails to test PC1 users, inform failures.
- **User Training**:
  - Formal sessions on security policies, strong passwords, threat avoidance.
  - Conducted at onboarding and regularly (e.g., quarterly).
- **Physical Access Control**:
  - Restricts access to devices (e.g., SW1, SRV1 in secure room).
  - Uses MFA (e.g., badge + fingerprint).
  - Example: SW1 in locked closet, badge access for IT staff only.
- **Exam Note**: Know these elements for CCNA topic 5.1.

**Demonstration**:
- **Topology**: PC1, SRV1, SW1 in secure data center.
- **Scenario**:
  - PC1 user receives fake phishing email, learns to avoid real ones.
  - IT staff trained on strong passwords (e.g., S3cur!ty2025).
  - SW1/SRV1 access requires badge + fingerprint.
- **Verification** (conceptual):
  - Check phishing test results (awareness).
  - Confirm badge logs for data center access.

**Comparison to Prior Content**:
- **QoS Part 2 Lecture**: Trust boundaries (trust IP phones, not PCs) align with physical access control.
- **Syslog Lab**: Physical access violations trigger Syslog (e.g., `%SEC-6-DOOR_OPEN`).
- **Day 18 (ARP)**: Physical access control prevents Attacker tampering with SW1.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Firewalls block unauthorized access, complementing physical controls.
- **Day 44 (NAT)**: Physical access control protects NAT-configured devices.
- **Day 10 (IP Addressing)**: Secure devices have restricted IPs.

**Analogy**: Security programs are like school rules: awareness (posters on bullying), training (classes on safety), physical access (locked staff rooms).

**Lifelong Retention Tip**: Simulate phishing test in Packet Tracer (conceptual email). Flashcards: “User awareness? Phishing tests. Physical access? Badge + fingerprint. Training? Password policies.” Review 3 times.

## Common Issues and Troubleshooting

- **DoS Attack**:
  - **Issue**: SRV1 unavailable (SYN flood).
  - **Cause**: Attacker floods TCP SYNs.
  - **Fix**: Firewall filters excessive SYNs (conceptual).
- **MITM Attack**:
  - **Issue**: PC1’s traffic to SRV1 intercepted.
  - **Cause**: ARP spoofing by Attacker.
  - **Fix**: Enable Dynamic ARP Inspection on SW1.
- **Phishing Success**:
  - **Issue**: PC1 user leaks credentials.
  - **Cause**: Lack of awareness.
  - **Fix**: Conduct user awareness training.
- **Weak Passwords**:
  - **Issue**: Attacker guesses PC1’s password.
  - **Cause**: Short, simple password (e.g., “password”).
  - **Fix**: Enforce ≥8 characters, mixed types, regular changes.
- **Unauthorized Physical Access**:
  - **Issue**: Attacker accesses SW1.
  - **Cause**: No badge/fingerprint lock.
  - **Fix**: Implement MFA locks on data center.

**Analogy**: Troubleshooting is like fixing a leaky house: patch windows (DoS), check IDs (MITM), teach residents (phishing), use strong locks (passwords), and secure doors (physical access).

**Lifelong Retention Tip**: Simulate ARP spoofing, apply DAI in Packet Tracer. Checklist: “System down? Check DoS, MITM, passwords, access.” Practice 3 times.

## Quiz Answers and Explanations

1. **Which part of the CIA triad ensures systems are running and accessible?**  
   - **A**: Confidentiality.  
   - **B**: Integrity.  
   - **C**: Authentication.  
   - **D**: Availability.  
   - **E**: Authorization.  
   - **F**: Accounting.  
   - **Answer**: D (Availability ensures systems are operational for authorized users).  

2. **Which term refers to the real possibility that a potential weakness is taken advantage of?**  
   - **A**: Threat.  
   - **B**: Vulnerability.  
   - **C**: Exploit.  
   - **D**: Mitigation.  
   - **Answer**: A (Threat is the possibility of exploiting a vulnerability).  

3. **Door locks requiring a badge and pass code are an example of? (Select two)**  
   - **A**: Confidentiality.  
   - **B**: User awareness.  
   - **C**: Physical access control.  
   - **D**: Multi-factor authentication.  
   - **Answer**: C, D (Badge + pass code = MFA; restricts access to secure areas).  

4. **Which is NOT multi-factor authentication?**  
   - **A**: Password + phone app.  
   - **B**: Badge + PIN.  
   - **C**: Retina scan + fingerprint scan.  
   - **D**: Password + fingerprint.  
   - **Answer**: C (Both scans are “something you are,” not MFA).  

5. **Which is considered Accounting in AAA?**  
   - **A**: Granting file access.  
   - **B**: Verifying user identity.  
   - **C**: Restricting admin access.  
   - **D**: Logging login time.  
   - **Answer**: D (Accounting logs user actions, e.g., login time).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: What does the CIA triad ensure?
    - **A**: Authentication, Authorization, Accounting.  
    - **B**: Confidentiality, Integrity, Availability.  
    - **C**: Firewalls, antivirus, encryption.  
    - **D**: User awareness, training, access control.  
    - **Answer**: B (CIA triad = Confidentiality, Integrity, Availability).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “CIA? Confidentiality, Integrity, Availability. Threat? Exploit possibility. MFA? ≥2 categories”). Practice in Packet Tracer (e.g., ARP spoofing). Review 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - CIA: “Confidentiality, Integrity, Availability” (CIA).
   - Attacks: “DoS, Spoof, MITM, Recon, Malware, Social, Password” (DSMRMSP).
   - AAA: “Authenticate, Authorize, Account” (AAA).
2. **Visualize Topology**:
   - Draw PC1 → SW1 → SRV1 with Attacker (MITM, DoS). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “CIA? C=authorized access, I=unmodified data, A=system uptime. MFA? Know, have, are. RADIUS? UDP 1812/1813.”
4. **Practice in Packet Tracer**:
   - Simulate ARP spoofing, SYN flood; apply conceptual mitigations (DAI, firewall). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: CIA vs. AAA vs. Attacks (e.g., SYN flood → Availability, ARP spoofing → Confidentiality/Integrity).
   - Quiz: “CIA? Confidentiality, Integrity, Availability. Accounting? Log actions.”
6. **Troubleshooting Practice**:
   - Simulate phishing, weak password; apply training, MFA. Checklist: “Breach? Check CIA, attacks, AAA.”
7. **Teach Someone**:
   - Explain CIA, MFA, AAA to a peer.
8. **Spaced Repetition**:
   - Use Anki for CIA, attacks, AAA protocols, ports. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **CIA Triad**:
   - Confidentiality: Authorized access only.
   - Integrity: Unmodified data.
   - Availability: Systems accessible.
2. **Security Concepts**:
   - Vulnerability (weakness), Exploit (method), Threat (possibility), Mitigation (protection).
3. **Attacks**:
   - DoS/DDoS (Availability), Spoofing (Availability/Confidentiality), MITM (Confidentiality/Integrity), Reconnaissance (prep), Malware (CIA), Social Engineering (Confidentiality/Integrity), Password Attacks (Confidentiality/Integrity).
4. **MFA**:
   - ≥2 factors (know, have, are); certificates verify websites.
5. **AAA**:
   - Authentication (verify), Authorization (permissions), Accounting (logs).
   - RADIUS (UDP 1812/1813), TACACS+ (TCP 49).
6. **Security Programs**:
   - User awareness (phishing tests), training (policies), physical access (MFA locks).
7. **CCNA Focus**:
   - Topic 5.1: Know CIA, attacks, MFA, AAA, security programs (no config).

These notes provide a comprehensive foundation for Security Fundamentals (CCNA topic 5.1). Practice attack simulations in Packet Tracer, use flashcards for CIA/AAA, and apply analogies/retention tips to master concepts for life!