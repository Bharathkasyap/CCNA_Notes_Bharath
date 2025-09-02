# SSH Lab Notes for CCNA Beginners

## Lab Overview

This lab configures **Secure Shell (SSH)** on a Layer 2 switch (SW2) for secure remote CLI access, aligning with **CCNA exam topic 4.8** ("Configure network devices for remote access using SSH"). Using Packet Tracer, it demonstrates console port security, management IP configuration, and SSH setup on SW2, connected to Laptop1 (console) and PC1 (SSH client). The lab secures the console with local authentication (`login local`, `exec-timeout`), sets up a management IP on VLAN1 (`interface vlan 1`, `ip default-gateway`), and configures SSH (`crypto key generate rsa`, `transport input ssh`) with an ACL to restrict access to PC1 (192.168.1.1). Verification includes ping and SSH tests from PC1 and R2, highlighting ACL restrictions. The lab reinforces the SSH lecture and prepares for exam topic 4.8.

**Lab Objectives**:
- **Secure Console Port**: Configure local authentication (`login local`) and timeout (`exec-timeout`) on SW2.
- **Configure Management IP**: Set up VLAN1 SVI and default gateway for remote access.
- **Configure SSH**: Enable SSH with RSA keys, restrict to SSH-only (`transport input ssh`), and limit access with an ACL.
- **Verify Connectivity**: Test ping and SSH from PC1 (allowed) and R2 (blocked).
- **Prepare for Exam**: Master SSH configuration and verification (topic 4.8).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
| PC1 (SSH)      |-------|      R2        |-------|      SW2       |
| 192.168.1.1    |       | G0/0: 192.168.2.254 | | VLAN1: 192.168.2.253 |
|                |       |                |       | Default-GW: 192.168.2.254 |
+----------------+       +----------------+       +----------------+
         |                       |
         |                       |
+----------------+       +----------------+
| Laptop1 (Console) |    |                |
|                |       |                |
+----------------+       +----------------+
```
- **Devices**:
  - **Laptop1**: Console access to SW2 for initial configuration.
  - **PC1**: SSH client (192.168.1.1), permitted by ACL to access SW2.
  - **R2**: Router (G0/0: 192.168.2.254), default gateway for SW2; blocked by ACL for SSH.
  - **SW2**: Layer 2 switch, management IP (VLAN1: 192.168.2.253), configured for SSH.
- **Note**: Connectivity via OSPF/static routes (assumed from Day 34). ACL restricts SSH to PC1. SW2 unconfigured initially.
- **Requirements**:
  1. Connect Laptop1 to SW2’s console, configure hostname, enable secret, user account.
  2. Set up SW2’s management IP (`interface vlan 1`, `ip default-gateway`).
  3. Secure console with `login local`, `exec-timeout`.
  4. Configure SSH (`ip domain name`, `crypto key generate rsa`, `transport input ssh`, ACL).
  5. Verify: Ping/SSH from PC1 (succeeds), R2 (ping succeeds, SSH fails).

**Analogy**: Configuring SSH on SW2 is like setting up a secure phone line (TCP 22) for remote commands from PC1, with a locked control room (console, `login local`) and a gatekeeper (ACL) allowing only PC1. The management IP is SW2’s mailbox for receiving calls.

**Lifelong Retention Tip**: Visualize PC1 SSH-ing to SW2 (192.168.2.253, TCP 22) with encrypted packets, blocked for R2 by ACL. In Packet Tracer, configure `crypto key generate rsa`, SSH from PC1, verify with `show ip ssh`. Create flashcards: “SSH port? TCP 22. Console security? `login local`. ACL command? `access-class`.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Console Connection and Basic Configuration
**Objective**: Connect Laptop1 to SW2’s console port, configure hostname, enable secret, and user account.

**Steps (on Laptop1, Console to SW2)**:
1. In Packet Tracer, select console cable (light blue), connect Laptop1’s RS232 port to SW2’s console port.
2. On Laptop1: Desktop → Terminal (default settings: OK).
3. Configure SW2:
```plaintext
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# enable secret ccna
SW2(config)# username jeremy secret ccna
```

**Verification**:
- **CLI Prompt**: Changes to `SW2(config)#` after `hostname SW2`.
- **Show Config**:
```plaintext
SW2# show running-config | include hostname|enable|username
hostname SW2
enable secret 5 $1$...
username jeremy secret 5 $1$...
```

**Observations**:
- **Console Access**: Default no password; direct CLI access via console cable.
- **Hostname**: `SW2` sets device name, required for RSA keys (SSH).
- **Enable Secret**: `ccna` (case-sensitive) required for privileged EXEC mode via SSH.
- **Username**: `jeremy` with `secret ccna` (encrypted, unlike `password`) for local authentication.
- **Packet Tracer**: Console connection mimics real-world setup; `secret` uses strong encryption.
- **Exam Note**: Basic configs (hostname, enable secret) are prerequisites for SSH (topic 4.8).

**Comparison to Prior Content**:
- **SSH Lecture**: Lecture noted hostname and enable secret as SSH prerequisites; lab applies them.
- **Syslog Lab**: Console triggers Syslog messages (e.g., `%SYS-5-CONFIG_I`) during config.
- **SNMP Lab**: Console for initial SNMP setup; similar to SSH setup here.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Console is local (no port); vs. SSH (TCP 22).
- **Security (Section 5.0)**: `enable secret`, `username secret` align with secure authentication.
- **Extended ACLs (Day 35)**: Console unaffected by ACLs (local access).

**Analogy**: Console setup is like initializing a control panel (SW2) with a name (`hostname`) and passcode (`enable secret`, `username`) before securing the room or enabling remote access.

**Lifelong Retention Tip**: In Packet Tracer, connect console cable, configure `hostname SW2`, `enable secret ccna`, verify with `show running-config`. Flashcard: “Enable secret? Required for SSH EXEC. Username? `secret` for encryption.” Practice 3 times.

### Step 2: Configure Management IP on SW2
**Objective**: Set up VLAN1 SVI and default gateway for remote management (SSH).

**Steps (on SW2, Console)**:
1. Configure VLAN1 SVI:
```plaintext
SW2(config)# interface vlan 1
SW2(config-if)# ip address 192.168.2.253 255.255.255.0
SW2(config-if)# no shutdown
```
2. Configure default gateway:
```plaintext
SW2(config)# ip default-gateway 192.168.2.254
```

**Verification**:
- **Syslog Messages** (after `no shutdown`):
  - `Jan 1 12:00:00: %LINK-5-CHANGED: Interface Vlan1, changed state to up`
  - `Jan 1 12:00:00: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up`
- **Show Config**:
```plaintext
SW2# show running-config | section vlan 1
interface vlan 1
 ip address 192.168.2.253 255.255.255.0
ip default-gateway 192.168.2.254
```
- **Ping (from SW2)**:
```plaintext
SW2# ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
```

**Observations**:
- **SVI**: `interface vlan 1` assigns management IP (192.168.2.253); `no shutdown` enables it.
- **Default Gateway**: `ip default-gateway 192.168.2.254` (R2) routes traffic to PC1 (192.168.1.1).
- **Syslog**: Interface up triggers `%LINK-5-CHANGED`, `%LINEPROTO-5-UPDOWN` (severity 5).
- **Packet Tracer**: VLAN1 SVI shutdown by default; requires `no shutdown`.
- **Exam Note**: Management IP enables SSH access (topic 4.8).

**Comparison to Prior Content**:
- **SSH Lecture**: Lecture configured VLAN1 SVI; lab applies to SW2 for SSH.
- **Syslog Lab**: SVI up/down logged to console/buffer (e.g., `%LINK-5-CHANGED`).
- **SNMP Lab**: Management IP used for SNMP queries (UDP 161).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: SVI enables SSH (TCP 22); vs. console (local).
- **VLANs (Day 15)**: VLAN1 SVI requires active VLAN.
- **Extended ACLs (Day 35)**: ACLs can block SVI (e.g., `deny tcp any host 192.168.2.253 eq 22`).

**Analogy**: The management IP is like giving SW2 a mailbox (VLAN1, 192.168.2.253) with a post office (default gateway, 192.168.2.254) to receive remote SSH calls from PC1.

**Lifelong Retention Tip**: In Packet Tracer, configure `interface vlan 1`, `ip default-gateway`, ping PC1. Flashcard: “L2 switch IP? `interface vlan 1`. Gateway? `ip default-gateway`.” Practice 3 times.

### Step 3: Secure Console Port
**Objective**: Configure console with local authentication (`login local`) and timeout (`exec-timeout`).

**Steps (on SW2, Console)**:
1. Configure console:
```plaintext
SW2(config)# line console 0
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5
```
2. Test authentication:
```plaintext
SW2(config)# end
SW2# exit
[Press Enter]
Username: jeremy
Password: ccna
SW2> enable
Password: ccna
```

**Verification**:
- **Console Login**: Prompts for `jeremy`/`ccna` after `exit`.
- **Show Config**:
```plaintext
SW2# show running-config | section line con
line con 0
 login local
 exec-timeout 5 0
```

**Observations**:
- **Login Local**: Requires `username jeremy secret ccna` credentials.
- **Exec Timeout**: 5 minutes inactivity logs out user.
- **Packet Tracer**: Console cable connection required; `secret` encrypts password.
- **Exam Note**: Console security supports SSH setup (topic 4.8) by securing local access.

**Comparison to Prior Content**:
- **SSH Lecture**: Lecture used `login local` for console; lab applies to SW2.
- **Syslog Lab**: Console configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SNMP Lab**: Console for secure SNMP setup; similar to SSH here.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Console local vs. SSH (TCP 22).
- **Security (Section 5.0)**: `login local` aligns with AAA authentication.
- **Extended ACLs (Day 35)**: Console unaffected by ACLs.

**Analogy**: Console security is like locking SW2’s control room with an ID check (`login local`) and a timer (`exec-timeout`) to eject idle users.

**Lifelong Retention Tip**: In Packet Tracer, configure `login local`, `exec-timeout 5`, test login (`jeremy`/`ccna`). Flashcard: “Console security? `login local`. Timeout? `exec-timeout`.” Practice 3 times.

### Step 4: Configure SSH on SW2
**Objective**: Enable SSH with RSA keys, restrict to SSH-only, and limit access to PC1 with an ACL.

**Steps (on SW2, Console)**:
1. Configure domain name and RSA keys:
```plaintext
SW2(config)# ip domain name jeremysitlab.com
SW2(config)# crypto key generate rsa
The name for the keys will be: SW2.jeremysitlab.com
Choose the size of the key modulus [512]: 2048
```
2. Configure VTY lines:
```plaintext
SW2(config)# access-list 1 permit host 192.168.1.1
SW2(config)# line vty 0 15
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5
SW2(config-line)# transport input ssh
SW2(config-line)# access-class 1 in
```

**Verification**:
- **Check SSH**:
```plaintext
SW2# show ip ssh
SSH Enabled - version 1.99
Authentication timeout: 120 secs
Authentication retries: 3
```
- **Show Config**:
```plaintext
SW2# show running-config | section line vty
line vty 0 15
 access-class 1 in
 login local
 exec-timeout 5 0
 transport input ssh
```
- **From PC1**:
```plaintext
ping 192.168.2.253
Reply from 192.168.2.253: bytes=100 time=1ms TTL=255
ssh -l jeremy 192.168.2.253
Password: ccna
SW2> enable
Password: ccna
SW2#
```
- **From R2**:
```plaintext
R2# ping 192.168.2.253
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.253:
!!!!!
R2# ssh -l jeremy 192.168.2.253
Connection refused by remote host
```

**Observations**:
- **Domain Name**: `ip domain name jeremysitlab.com` required for RSA keys (FQDN: SW2.jeremysitlab.com).
- **RSA Keys**: `crypto key generate rsa` (2048 bits) enables SSH; ≥768 bits for SSHv2.
- **VTY Config**: `transport input ssh` restricts to SSH (no Telnet); `login local` mandates username (`jeremy`/`ccna`).
- **ACL**: `access-class 1 in` permits only PC1 (192.168.1.1); blocks R2.
- **Packet Tracer**: ARP delays may cause initial ping failures; SSH uses TCP 22.
- **Syslog**: `%SSH-5-ENABLED` logged when keys generated.
- **Exam Note**: SSH config (`crypto key generate rsa`, `transport input ssh`) is critical for topic 4.8.

**Comparison to Prior Content**:
- **SSH Lecture**: Lab mirrors lecture’s SSH config (hostname, domain, RSA, VTY).
- **Syslog Lab**: SSH setup triggers Syslog (e.g., `%SYS-5-CONFIG_I`, `%SSH-5-ENABLED`).
- **SNMP Lab**: SSH for secure SNMP config vs. Telnet’s insecurity.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: SSH TCP 22 vs. Telnet TCP 23.
- **Security (Section 5.0)**: SSH encryption, `login local` align with secure access.
- **Extended ACLs (Day 35)**: `access-class` uses standard ACL (`permit host 192.168.1.1`).

**Analogy**: SSH setup is like installing a secure phone line (TCP 22) on SW2, requiring a key (RSA), ID (`login local`), and a gatekeeper (ACL) to allow only PC1’s calls.

**Lifelong Retention Tip**: In Packet Tracer, configure `crypto key generate rsa`, `transport input ssh`, SSH from PC1, verify R2 is blocked. Flashcard: “SSH key? `crypto key generate rsa`. Port? TCP 22.” Practice 3 times.

## Common Issues and Troubleshooting

- **Console Login Fails**:
  - **Issue**: Wrong username/password after `exit`.
  - **Cause**: Incorrect `username jeremy secret ccna` or `login local`.
  - **Fix**: Verify `show running-config | section line con`, reconfigure `username`, `login local`.
- **RSA Key Generation Fails**:
  - **Issue**: `crypto key generate rsa` rejected.
  - **Cause**: Missing `hostname` or `ip domain name`.
  - **Fix**: Configure `hostname SW2`, `ip domain name jeremysitlab.com`, retry (`show ip ssh`).
- **SSH Connection Refused**:
  - **Issue**: `ssh -l jeremy 192.168.2.253` fails.
  - **Cause**: Missing `transport input ssh`, no RSA keys, or ACL block.
  - **Fix**: Verify `transport input ssh`, `show ip ssh`, ACL (`show access-lists`).
- **Ping Fails from PC1**:
  - **Issue**: `ping 192.168.2.253` fails.
  - **Cause**: VLAN1 down, wrong default gateway, or network issue.
  - **Fix**: Check `interface vlan 1` (`no shutdown`), `ip default-gateway 192.168.2.254`, ping again.
- **R2 SSH Allowed**:
  - **Issue**: R2 connects via SSH.
  - **Cause**: Missing or incorrect ACL (`access-class 1 in`).
  - **Fix**: Verify `access-list 1 permit host 192.168.1.1`, `access-class 1 in` (`show running-config | section line vty`).

**Analogy**: Troubleshooting SSH is like ensuring a secure phone line (TCP 22) has a valid key (RSA), correct ID (`login local`), and a gatekeeper (ACL) allowing only PC1, with a working mailbox (VLAN1 IP).

**Lifelong Retention Tip**: Simulate SSH failure (remove `ip domain name`), retry after fixing, verify with `show ip ssh`. Checklist: “No SSH? Check hostname, domain, RSA, VTY, ACL.” Practice 3 times.

## Practice Commands

- **Basic Configuration (SW2)**:
```plaintext
SW2(config)# hostname SW2
SW2(config)# enable secret ccna
SW2(config)# username jeremy secret ccna
```
- **Management IP (SW2)**:
```plaintext
SW2(config)# interface vlan 1
SW2(config-if)# ip address 192.168.2.253 255.255.255.0
SW2(config-if)# no shutdown
SW2(config)# ip default-gateway 192.168.2.254
```
- **Console Security (SW2)**:
```plaintext
SW2(config)# line console 0
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5
```
- **SSH Configuration (SW2)**:
```plaintext
SW2(config)# ip domain name jeremysitlab.com
SW2(config)# crypto key generate rsa
SW2(config)# access-list 1 permit host 192.168.1.1
SW2(config)# line vty 0 15
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5
SW2(config-line)# transport input ssh
SW2(config-line)# access-class 1 in
```
- **Verification**:
```plaintext
SW2# show ip ssh
SW2# show running-config | section line vty
SW2# show running-config | section line con
SW2# ping 192.168.1.1
```
- **From PC1**:
```plaintext
ping 192.168.2.253
ssh -l jeremy 192.168.2.253
```
- **From R2**:
```plaintext
R2# ping 192.168.2.253
R2# ssh -l jeremy 192.168.2.253
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - SSH Steps: “Hostname, Domain, RSA, VTY, SSH” (`hostname`, `ip domain name`, `crypto key`, `line vty`, `transport input ssh`).
   - Ports: “SSH 22, Telnet 23” (S before T, 22 before 23).
2. **Visualize Topology**:
   - Draw PC1 → SW2 (SSH, TCP 22) with ACL blocking R2. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “SSH port? TCP 22. RSA command? `crypto key generate rsa`. ACL? `access-class`.”
4. **Practice in Packet Tracer**:
   - Configure `crypto key generate rsa`, `transport input ssh`, SSH from PC1, verify R2 blocked (`show ip ssh`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: SSH (TCP 22, encrypted) vs. Syslog (UDP 514, logs).
   - Quiz: “SSH port? TCP 22. Console security? `login local`.”
6. **Troubleshooting Practice**:
   - Remove `ip domain name`, try SSH, fix, verify. Checklist: “No SSH? Check hostname, domain, keys, VTY, ACL.”
7. **Teach Someone**:
   - Explain SSH config and ACL to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, ports, SSH steps. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Console Security**:
   - `login local`, `exec-timeout 5` secure console access.
2. **Management IP**:
   - `interface vlan 1`, `ip default-gateway` enable remote access.
3. **SSH Configuration**:
   - `hostname`, `ip domain name`, `crypto key generate rsa`, `transport input ssh`, `login local`.
4. **ACL Restriction**:
   - `access-list 1 permit host 192.168.1.1`, `access-class 1 in` limits SSH to PC1.
5. **Verification**:
   - `show ip ssh`, `show running-config`, ping/SSH from PC1 (succeeds), R2 (SSH fails).
6. **CCNA Focus**:
   - Topic 4.8: Master SSH setup (`crypto key generate rsa`, `transport input ssh`).

These notes provide a complete foundation for the SSH lab, reinforcing CCNA exam topic 4.8. Practice configurations in Packet Tracer, verify SSH access from PC1, and use analogies/retention tips to master SSH for life!