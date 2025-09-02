# SSH Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **Secure Shell (SSH)**, a protocol for secure remote CLI access to network devices, aligning with **CCNA exam topic 4.8** ("Configure network devices for remote access using SSH"). It also introduces console port security, Layer 2 switch management IP configuration, and Telnet as a less secure alternative to SSH. The lecture explains how to secure the console port (`line console 0`, `password`, `login local`), configure a management IP on a Layer 2 switch (`interface vlan`, `ip default-gateway`), set up Telnet (`transport input telnet`), and configure SSH (`crypto key generate rsa`, `transport input ssh`). A comparison of Telnet and SSH highlights SSH’s encryption and security advantages. A bonus practice question from Boson Software’s ExSim for CCNA reinforces key concepts.

**Lecture Objectives**:
- **Secure Console Port**: Configure password or local authentication (`login local`) and timeout (`exec-timeout`).
- **Configure Layer 2 Switch Management IP**: Set up SVI (`interface vlan 1`) and default gateway (`ip default-gateway`).
- **Understand Telnet**: Explain its insecurity (plain-text, TCP 23) and configuration.
- **Configure SSH**: Set up SSHv2 (`crypto key generate rsa`, `ip ssh version 2`) for secure remote access.
- **Compare Telnet and SSH**: Highlight SSH’s encryption and authentication (TCP 22).
- **Prepare for Exam**: Master SSH configuration and related concepts (topic 4.8).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
| PC2 (Admin)    |-------|      SW1       |-------|      R1        |
| 192.168.1.2    |       | VLAN1: 192.168.1.1 | | G0/0: 192.168.1.254 |
| (Telnet/SSH)   |       | Default-GW: 192.168.1.254 |   |
+----------------+       +----------------+       +----------------+
         |                       |
         |                       |
+----------------+       +----------------+
| PC1 (Console)  |       |      R2        |
|                |       | 192.168.2.1    |
|                |       | (Blocked Telnet) |
+----------------+       +----------------+
```
- **Devices**:
  - **PC2**: Admin PC (192.168.1.2), connects to SW1/R1 via Telnet (TCP 23) or SSH (TCP 22).
  - **PC1**: Console access to SW1/R1 (local CLI).
  - **SW1**: Layer 2 switch, management IP (VLAN1: 192.168.1.1), default gateway (192.168.1.254).
  - **R1**: Router (G0/0: 192.168.1.254), routes for SW1, supports Telnet/SSH.
  - **R2**: External device (192.168.2.1), blocked by ACL for Telnet/SSH to SW1.
- **Note**: Connectivity via OSPF/static routes (assumed from Day 34). ACL restricts Telnet/SSH to PC2.
- **Requirements**:
  1. Secure R1/SW1 console with password or local authentication (`login local`, `exec-timeout`).
  2. Configure SW1 management IP (`interface vlan 1`, `ip default-gateway`).
  3. Set up Telnet on SW1 (`transport input telnet`, ACL).
  4. Configure SSH on SW1 (`crypto key generate rsa`, `ip ssh version 2`, `transport input ssh`).
  5. Verify with `show ip ssh`, `show running-config`, and SSH from PC2 (`ssh -l jeremy 192.168.1.1`).

**Analogy**: SSH is like a secure phone line (encrypted, TCP 22) for remotely configuring a device (SW1), while Telnet is an unsecured line (plain-text, TCP 23). Console access is like directly speaking to the device’s control panel, secured with a passcode (`password`) or ID (`login local`).

**Lifelong Retention Tip**: Visualize PC2 SSH-ing to SW1 (192.168.1.1, TCP 22) with encrypted packets, unlike Telnet’s plain-text. In Packet Tracer, configure `crypto key generate rsa`, SSH from PC2, and verify with `show ip ssh`. Create flashcards: “SSH port? TCP 22. Telnet? TCP 23. RSA keys? `crypto key generate rsa`.” Practice 3 times daily for a week.

## Console Port Security

**Purpose**: Secure the console port to require authentication, preventing unauthorized CLI access, and set timeouts for inactivity.

**Key Points**:
- **Default Behavior**: No password required for console access; anyone with physical access can configure the device.
- **Security Options**:
  - **Password**: Set a console-specific password (`password`, `login`).
  - **Local Authentication**: Require username/password (`login local`).
  - **Timeout**: Log out inactive sessions (`exec-timeout`).
- **Console Line**: `line console 0` (single console port, one user at a time).

**Commands (on R1/SW1)**:
```plaintext
R1(config)# line console 0
R1(config-line)# password ccna
R1(config-line)# login
R1(config-line)# exec-timeout 3 30
```
- **Alternative (Local Authentication)**:
```plaintext
R1(config)# username jeremy secret ccnp
R1(config)# line console 0
R1(config-line)# login local
R1(config-line)# exec-timeout 3 30
```

**Verification**:
- **Password**: After `exit`, console prompts for password (`ccna`).
- **Login Local**: Prompts for username (`jeremy`) and password (`ccnp`).
- **Show Config**:
```plaintext
R1# show running-config | section line con
line con 0
 password ccna
 login
 exec-timeout 3 30
```
- **Login Local**:
```plaintext
line con 0
 login local
 exec-timeout 3 30
```

**Observations**:
- **Password**: `login` enforces password; hidden when typed.
- **Login Local**: Uses device’s user database (`username jeremy secret ccnp`); overrides console password.
- **Exec Timeout**: 3 minutes, 30 seconds inactivity logs out user.
- **Exam Note**: Console security not in topic 4.8 but foundational for device access.

**Comparison to Prior Content**:
- **Syslog Lab**: Console logs (`%SYS-5-CONFIG_I`) show configuration changes; console security limits who can trigger them.
- **SNMP Lab**: Console access for configuring `snmp-server community`, secured with password.
- **Syslog Lecture**: Console logs enabled by default; secure console to restrict access.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Console is local (no port); vs. SSH (TCP 22), Telnet (TCP 23).
- **Security (Section 5.0)**: Console password/login local aligns with authentication practices.
- **Extended ACLs (Day 35)**: Console access unaffected by ACLs (local).

**Analogy**: Console security is like locking a control room door (console port) with a passcode (`password ccna`) or ID check (`login local`), timing out if the operator leaves (`exec-timeout`).

**Lifelong Retention Tip**: In Packet Tracer, configure `line console 0`, `password ccna`, `login`, test login, verify with `show running-config | section line con`. Flashcard: “Console security? `password`, `login` or `login local`.” Practice 3 times.

## Layer 2 Switch Management IP

**Purpose**: Assign an IP address to a Layer 2 switch for remote management (Telnet/SSH), using a Switch Virtual Interface (SVI) and default gateway.

**Key Points**:
- **Layer 2 Switch**: Forwards frames, no routing table; needs SVI for management.
- **SVI**: `interface vlan 1` assigns IP for remote access (e.g., 192.168.1.1).
- **Default Gateway**: `ip default-gateway` routes traffic to external networks (e.g., R1, 192.168.1.254).
- **Use Case**: Enables SSH/Telnet from PC2 (192.168.1.2) to SW1.

**Commands (on SW1)**:
```plaintext
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config)# ip default-gateway 192.168.1.254
```

**Verification**:
- **Ping**:
```plaintext
SW1# ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
```
- **Show Config**:
```plaintext
SW1# show running-config | section vlan 1
interface vlan 1
 ip address 192.168.1.1 255.255.255.0
ip default-gateway 192.168.1.254
```

**Observations**:
- **SVI**: `interface vlan 1` must be up (VLAN1 active, `no shutdown`).
- **Default Gateway**: Routes traffic to R1 (192.168.1.254) for external access.
- **Use**: Enables Telnet/SSH to 192.168.1.1; ping confirms connectivity.
- **Exam Note**: Management IP supports topic 4.8 (SSH access to switches).

**Comparison to Prior Content**:
- **Syslog Lab**: Management IP allows Syslog server access (UDP 514 to SRV1).
- **SNMP Lab**: SVI IP used for SNMP queries (UDP 161 to 192.168.1.1).
- **DHCP Lab**: Switch can use DHCP for SVI IP (not covered here).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: SVI IP enables SSH (TCP 22), Telnet (TCP 23).
- **VLANs (Day 15)**: SVI tied to VLAN1; requires active VLAN.
- **Extended ACLs (Day 35)**: ACLs can block SVI access (e.g., `deny tcp any host 192.168.1.1 eq 22`).

**Analogy**: The management IP is like giving a switch a mailbox (VLAN1 IP) and a post office address (default gateway) to receive remote commands (SSH) from an admin (PC2).

**Lifelong Retention Tip**: In Packet Tracer, configure `interface vlan 1`, `ip address 192.168.1.1`, `ip default-gateway 192.168.1.254`, ping PC2. Flashcard: “L2 switch IP? `interface vlan 1`. Gateway? `ip default-gateway`.” Practice 3 times.

## Telnet

**Purpose**: Telnet enables remote CLI access but is insecure (plain-text, TCP 23), largely replaced by SSH.

**Key Points**:
- **Function**: Connects to device CLI remotely (client to server, TCP 23).
- **Security**: Sends data in plain-text (e.g., passwords visible in Wireshark).
- **Configuration**: Requires VTY lines (`line vty 0 15`), `login local`, or `password`/`login`.
- **Client-Server**: Client (PC2) initiates; server (SW1) listens on TCP 23.

**Commands (on SW1)**:
```plaintext
SW1(config)# enable secret ccnp
SW1(config)# username jeremy secret ccnp
SW1(config)# access-list 1 permit 192.168.1.2
SW1(config)# line vty 0 15
SW1(config-line)# login local
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# transport input telnet
SW1(config-line)# access-class 1 in
```

**Verification**:
- **From PC2**:
```plaintext
telnet 192.168.1.1
Username: jeremy
Password: ccnp
SW1> enable
Password: ccnp
SW1#
```
- **From R2**:
  - `telnet 192.168.1.1`: “Connection refused” (ACL blocks).
- **Show Config**:
```plaintext
SW1# show running-config | section line vty
line vty 0 15
 access-class 1 in
 login local
 exec-timeout 5 0
 transport input telnet
```

**Observations**:
- **Insecurity**: Plain-text passwords (e.g., `ccnp`) visible in Wireshark.
- **VTY Lines**: `line vty 0 15` allows 16 simultaneous connections.
- **Transport Input**: `telnet` restricts to Telnet; `all` allows other protocols.
- **ACL**: `access-class 1 in` limits connections to PC2 (192.168.1.2).
- **Exam Note**: Telnet not in topic 4.8 but contrasts with SSH’s security.

**Comparison to Prior Content**:
- **Syslog Lab**: Telnet enables VTY logging (`terminal monitor`); Syslog logs Telnet sessions.
- **SNMP Lab**: Telnet to configure SNMP vs. SSH’s secure access.
- **Syslog Lecture**: Telnet sessions logged (e.g., `%SYS-5-CONFIG_I`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Telnet TCP 23 vs. SSH TCP 22.
- **Security (Section 5.0)**: Telnet’s plain-text insecure vs. SSH encryption.
- **Extended ACLs (Day 35)**: `access-class` uses ACL to restrict Telnet (e.g., `permit 192.168.1.2`).

**Analogy**: Telnet is like shouting commands across an open field (plain-text, TCP 23), easily overheard, unlike SSH’s secure phone call.

**Lifelong Retention Tip**: In Packet Tracer, configure `transport input telnet`, Telnet from PC2, verify ACL blocks R2. Flashcard: “Telnet port? TCP 23. Secure? No, plain-text.” Practice 3 times.

## SSH

**Purpose**: SSH provides secure remote CLI access (encrypted, TCP 22), replacing Telnet, with RSA keys for encryption and authentication.

**Key Points**:
- **Security**: Encrypts data (unlike Telnet’s plain-text); uses RSA keys.
- **Versions**: SSHv1 (1995), SSHv2 (2006, more secure), 1.99 (supports both).
- **Port**: TCP 22 (vs. Telnet TCP 23).
- **Requirements**:
  - K9 IOS image (supports crypto, unlike NPE).
  - Hostname and domain name for RSA keys.
  - RSA keys (`crypto key generate rsa`, ≥768 bits for SSHv2).
  - VTY configuration (`transport input ssh`, `login local`).

**Commands (on SW1)**:
```plaintext
SW1(config)# hostname SW1
SW1(config)# ip domain name jeremysitlab.com
SW1(config)# crypto key generate rsa
The name for the keys will be: SW1.jeremysitlab.com
Choose the size of the key modulus [512]: 2048
SW1(config)# ip ssh version 2
SW1(config)# enable secret ccnp
SW1(config)# username jeremy secret ccnp
SW1(config)# access-list 1 permit 192.168.1.2
SW1(config)# line vty 0 15
SW1(config-line)# login local
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# transport input ssh
SW1(config-line)# access-class 1 in
```

**Verification**:
- **Check SSH**:
```plaintext
SW1# show ip ssh
SSH Enabled - version 1.99
Authentication timeout: 120 secs
Authentication retries: 3
```
- **Show Config**:
```plaintext
SW1# show running-config | section line vty
line vty 0 15
 access-class 1 in
 login local
 exec-timeout 5 0
 transport input ssh
```
- **From PC2**:
```plaintext
ssh -l jeremy 192.168.1.1
Password: ccnp
SW1> enable
Password: ccnp
SW1#
```
- **From R2**:
  - `ssh -l jeremy 192.168.1.1`: “Connection refused” (ACL blocks).

**Observations**:
- **Prerequisites**: Non-default hostname (`SW1`), domain name (`jeremysitlab.com`), K9 IOS (`show version`).
- **RSA Keys**: `crypto key generate rsa` creates keys (SW1.jeremysitlab.com, 2048 bits); ≥768 bits for SSHv2.
- **SSHv2**: `ip ssh version 2` enforces v2 (more secure).
- **Login Local**: Requires username/password (`jeremy/ccnp`); `login` not supported for SSH.
- **ACL**: `access-class 1 in` restricts to PC2 (192.168.1.2).
- **Syslog**: `%SSH-5-ENABLED` logged when keys generated.
- **Exam Note**: Topic 4.8 requires SSH configuration (`crypto key generate rsa`, `transport input ssh`).

**Comparison to Prior Content**:
- **Syslog Lab**: SSH sessions logged (e.g., `%SYS-5-CONFIG_I`); `terminal monitor` shows logs.
- **SNMP Lab**: SSH for secure SNMP configuration vs. Telnet’s insecurity.
- **Syslog Lecture**: SSH login triggers Syslog messages (UDP 514).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: SSH TCP 22 vs. Telnet TCP 23.
- **Security (Section 5.0)**: SSH encryption aligns with AAA, secure authentication.
- **Extended ACLs (Day 35)**: `access-class` restricts SSH (e.g., `deny tcp any host 192.168.1.1 eq 22`).

**Analogy**: SSH is like a secure, encrypted walkie-talkie (TCP 22) for configuring SW1, requiring a key (RSA) and ID (username), unlike Telnet’s open megaphone.

**Lifelong Retention Tip**: In Packet Tracer, configure `crypto key generate rsa`, `ip ssh version 2`, SSH from PC2, verify with `show ip ssh`. Flashcard: “SSH port? TCP 22. Key command? `crypto key generate rsa`.” Practice 3 times.

## Telnet vs. SSH

**Purpose**: Compare Telnet (insecure, plain-text) and SSH (secure, encrypted) for remote CLI access.

**Comparison**:
| Feature           | Telnet                              | SSH                               |
|-------------------|-------------------------------------|------------------------------------|
| **Port**          | TCP 23                            | TCP 22                            |
| **Security**      | Plain-text, no encryption          | Encrypted (RSA keys)              |
| **Authentication** | Password or `login local`         | `login local` only                |
| **Developed**     | 1969                              | 1995 (v1), 2006 (v2)             |
| **Use Case**      | Legacy, insecure                  | Modern, secure remote access      |
| **Command**       | `transport input telnet`          | `transport input ssh`             |
| **IOS Support**   | All IOS images                    | K9 IOS images only                |

**Observations**:
- **Telnet**: Passwords visible in Wireshark; insecure for modern networks.
- **SSH**: Encrypts data; requires RSA keys, hostname, domain name.
- **VTY Lines**: Both use `line vty 0 15`; SSH mandates `login local`.
- **Exam Note**: Topic 4.8 emphasizes SSH; Telnet for context.

**Comparison to Prior Content**:
- **Syslog Lab**: Both trigger Syslog logs (`%SYS-5-CONFIG_I`) via VTY.
- **SNMP Lab**: SSH for secure SNMP config; Telnet risks exposure.
- **Syslog Lecture**: Telnet/SSH sessions logged to console/buffer/server.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Telnet TCP 23 vs. SSH TCP 22.
- **Security (Section 5.0)**: SSH aligns with encryption; Telnet insecure.
- **Extended ACLs (Day 35)**: `access-class` restricts both (e.g., `permit 192.168.1.2`).

**Analogy**: Telnet is an open letter (plain-text, TCP 23), readable by anyone; SSH is a sealed, coded message (encrypted, TCP 22), only readable with the right key (RSA).

**Lifelong Retention Tip**: Compare Telnet (`telnet 192.168.1.1`, plain-text) vs. SSH (`ssh -l jeremy 192.168.1.1`, encrypted). Flashcard: “Telnet vs. SSH? Telnet plain-text (23), SSH encrypted (22).” Practice 3 times.

## Common Issues and Troubleshooting

- **Console Login Fails**:
  - **Issue**: No access after `exit`.
  - **Cause**: Wrong password or `login local` misconfigured.
  - **Fix**: Verify `password ccna`, `login`, or `username jeremy secret ccnp`, `login local` (`show running-config | section line con`).
- **SSH Fails (Key Error)**:
  - **Issue**: `crypto key generate rsa` rejected.
  - **Cause**: Missing hostname or domain name.
  - **Fix**: Configure `hostname SW1`, `ip domain name jeremysitlab.com`, retry (`show ip ssh`).
- **SSH Connection Refused**:
  - **Issue**: `ssh -l jeremy 192.168.1.1` fails.
  - **Cause**: Missing `transport input ssh`, non-K9 IOS, or ACL block.
  - **Fix**: Verify `transport input ssh`, K9 IOS (`show version`), ACL (`show access-lists`).
- **Telnet Connection Refused**:
  - **Issue**: `telnet 192.168.1.1` fails from R2.
  - **Cause**: ACL (`access-class 1 in`) blocks R2.
  - **Fix**: Check ACL (`show access-lists`); allow R2 if needed (`access-list 1 permit 192.168.2.1`).
- **No VTY Access**:
  - **Issue**: Neither Telnet nor SSH works.
  - **Cause**: `transport input none` or missing `login local`.
  - **Fix**: Configure `transport input ssh` or `telnet`, `login local` (`show running-config | section line vty`).

**Analogy**: Troubleshooting SSH is like ensuring a secure phone line (TCP 22) has a valid key (RSA), correct ID (username), and open line (VTY config), unlike Telnet’s open channel.

**Lifelong Retention Tip**: In Packet Tracer, simulate SSH failure (remove `ip domain name`), retry after fixing, verify with `show ip ssh`. Checklist: “No SSH? Check hostname, domain, RSA keys, `transport input ssh`.” Practice 3 times.

## Practice Commands

- **Console Security (R1/SW1)**:
```plaintext
R1(config)# username jeremy secret ccnp
R1(config)# line console 0
R1(config-line)# login local
R1(config-line)# exec-timeout 3 30
```
- **Layer 2 Switch Management IP (SW1)**:
```plaintext
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config)# ip default-gateway 192.168.1.254
```
- **Telnet (SW1)**:
```plaintext
SW1(config)# enable secret ccnp
SW1(config)# username jeremy secret ccnp
SW1(config)# access-list 1 permit 192.168.1.2
SW1(config)# line vty 0 15
SW1(config-line)# login local
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# transport input telnet
SW1(config-line)# access-class 1 in
```
- **SSH (SW1)**:
```plaintext
SW1(config)# hostname SW1
SW1(config)# ip domain name jeremysitlab.com
SW1(config)# crypto key generate rsa
SW1(config)# ip ssh version 2
SW1(config)# enable secret ccnp
SW1(config)# username jeremy secret ccnp
SW1(config)# access-list 1 permit 192.168.1.2
SW1(config)# line vty 0 15
SW1(config-line)# login local
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# transport input ssh
SW1(config-line)# access-class 1 in
```
- **Verification**:
```plaintext
SW1# show ip ssh
SW1# show running-config | section line vty
SW1# show running-config | section line con
SW1# ping 192.168.1.2
```

**From PC2**:
```plaintext
ssh -l jeremy 192.168.1.1
telnet 192.168.1.1
```

## Quiz Answers and Explanations

1. **Why is `crypto key generate rsa` rejected? (Select two)**  
   - **A**: Hostname not configured  
   - **B**: Enable secret not configured  
   - **C**: VTY lines not configured  
   - **D**: IOS image is NPE  
   - **E**: DNS domain name not configured  
   - **Answer**: A, E (Requires hostname and domain name for FQDN).  

2. **Which commands allow both Telnet and SSH to VTY lines? (Select two)**  
   - **A**: `transport input default`  
   - **B**: `transport input none`  
   - **C**: `transport input telnet ssh`  
   - **D**: `transport input all`  
   - **Answer**: C, D (`telnet ssh` or `all` allow both).  

3. **Allow only 192.168.1.1 to SSH to R1. Which configuration?**  
   - **A**: Incorrect (e.g., wrong port or command)  
   - **B**: `access-list 1 permit 192.168.1.1`, `line vty 0 15`, `access-class 1 in`, `transport input ssh`  
   - **C**: Incorrect (e.g., wrong ACL command)  
   - **D**: Incorrect (e.g., missing `access-class`)  
   - **Answer**: B (Correct ACL, VTY config, TCP 22).  

4. **Which statements about SSH are true? (Select two)**  
   - **A**: RSA keys not required  
   - **B**: K9 IOS images support SSH  
   - **C**: SSH v1.99 is an actual version  
   - **D**: SSH sends plain-text data  
   - **E**: NPE IOS images support SSH  
   - **F**: Key length ≥768 bits for SSHv2  
   - **Answer**: B, F (K9 supports SSH; ≥768 bits for v2).  

5. **Admin on PC1 uses SSH to SW1. SW1’s role?**  
   - **A**: SSH client  
   - **B**: SSH server  
   - **C**: Telnet client  
   - **D**: Telnet server  
   - **Answer**: B (SW1 is the SSH server).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: SSH uses which port?
    - **A**: TCP 23
    - **B**: TCP 22
    - **C**: UDP 22
    - **D**: UDP 23
    - **Answer**: B (SSH uses TCP 22).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “SSH port? TCP 22. RSA keys? Required.”). Practice questions 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Ports: “SSH 22, Telnet 23” (S before T, 22 before 23).
   - SSH Steps: “Hostname, Domain, RSA, VTY, SSH” (for `hostname`, `ip domain name`, `crypto key`, `line vty`, `transport input ssh`).
2. **Visualize Topology**:
   - Draw PC2 → SW1 (SSH, TCP 22) with RSA keys. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “SSH port? TCP 22. Telnet? TCP 23. RSA command? `crypto key generate rsa`.”
4. **Practice in Packet Tracer**:
   - Configure `crypto key generate rsa`, `transport input ssh`, SSH from PC2, verify (`show ip ssh`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: SSH (TCP 22, encrypted) vs. Telnet (TCP 23, plain-text).
   - Quiz: “SSH server port? TCP 22. Telnet insecure? Plain-text.”
6. **Troubleshooting Practice**:
   - Remove `ip domain name`, try `crypto key generate rsa`, fix, verify. Checklist: “No SSH? Check hostname, domain, keys, VTY.”
7. **Teach Someone**:
   - Explain SSH vs. Telnet and console security to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, ports, SSH steps. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Console Security**:
   - Secure with `password`, `login` or `login local`, `exec-timeout`.
2. **Layer 2 Switch Management IP**:
   - `interface vlan 1`, `ip address`, `ip default-gateway` for remote access.
3. **Telnet**:
   - Insecure (plain-text, TCP 23), configured with `transport input telnet`.
4. **SSH**:
   - Secure (encrypted, TCP 22), requires `hostname`, `ip domain name`, `crypto key generate rsa`, `transport input ssh`.
5. **Telnet vs. SSH**:
   - SSH encrypts data; Telnet doesn’t. SSH uses TCP 22, Telnet TCP 23.
6. **CCNA Focus**:
   - Topic 4.8: Master SSH configuration (`crypto key generate rsa`, `line vty`, `transport input ssh`).

These notes provide a complete foundation for the SSH lecture, preparing you for CCNA exam topic 4.8. Practice configurations in Packet Tracer, verify SSH access, and use analogies/retention tips to master SSH for life!