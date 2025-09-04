# Port Security Lab Notes for CCNA Beginners

## Lab Overview

This lab configures **port security** on Cisco switches SW1 and SW2 in **Packet Tracer**, reinforcing **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **Port Security Lecture**, **Security Fundamentals Lecture** (topic 5.1), and **Kali Linux Demo Lab** (DHCP exhaustion attack), demonstrating practical application of port security to prevent unauthorized access and mitigate attacks like DHCP starvation. The lab configures port security on SW1’s interfaces (F0/1–3) for PCs and SW2’s G0/1 (connected to SW1), tests violation modes (shutdown, restrict), and verifies configurations.

**Lab Objectives**:
- **Configure Port Security on SW1**: Enable on F0/1–3, shutdown mode, max 1 MAC, aging time 60 minutes, no sticky learning.
- **Configure Port Security on SW2**: Enable on G0/1, restrict mode, max 4 MACs (for PC1, PC2, PC3, SW1), sticky learning.
- **Test Violations**: Trigger violations on SW1 (PC1 MAC change) and SW2 (SW1 SVI ping), observe shutdown/restrict behavior.
- **Verify Configurations**: Use `show` commands to confirm settings, MAC learning, and violation counts.
- **Prepare for CCNA**: Master topic 5.7 (port security configuration/verification) and connect to topics 5.1 (security concepts) and 2.2 (DHCP).
- **Reinforce Prior Content**: Link to DHCP (Day 13), DTP (Day 22), Kali Linux Demo Lab (Day 48), and Port Security Lecture.

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      SW2       |-------|      R1        |
| MAC: A.A.A     | F0/1  | G0/0           | G0/1  | G0/0           |       | 10.0.0.254     |
| 10.0.0.1       |       |                |       |                |       | (Router)       |
+----------------+       +----------------+       +----------------+       +----------------+
|      PC2       | F0/2  |
| MAC: B.B.B     |       |
| 10.0.0.2       |       |
+----------------+       |
|      PC3       | F0/3  |
| MAC: C.C.C     |       |
| 10.0.0.3       |       |
+----------------+       |
```
- **Devices**:
  - **PC1, PC2, PC3**: Clients on SW1 (F0/1–3), MACs A.A.A, B.B.B, C.C.C, IPs 10.0.0.1–3 (DHCP or static).
  - **SW1**: Cisco switch, port security on F0/1–3 (max 1 MAC, shutdown), G0/0 to SW2.
  - **SW2**: Cisco switch, port security on G0/1 (max 4 MACs, restrict, sticky), G0/0 to R1.
  - **R1**: Router, IP 10.0.0.254, ping destination.
- **Note**: 
  - SW1’s F0/1–3: Each allows 1 MAC (A.A.A, B.B.B, C.C.C), shutdown mode, 60-min aging.
  - SW2’s G0/1: Allows 4 MACs (PC1, PC2, PC3, SW1’s G0/0), restrict mode, sticky learning.
  - Violations: PC1 MAC change (F0/1, shutdown), SW1 SVI ping (G0/1, restrict).
  - Packet Tracer limits: No ErrDisable recovery, no Syslog for restrict mode.
- **Requirements**:
  1. Configure SW1’s F0/1–3: `switchport port-security`, max 1, shutdown, aging 60 mins.
  2. Configure SW2’s G0/1: `switchport port-security`, max 4, restrict, sticky.
  3. Test pings from PC1–3 to R1, verify MAC learning.
  4. Trigger violations: Change PC1’s MAC, ping from SW1’s SVI.
  5. Verify with `show port-security`, `show mac address-table`, `show running-config`.
  6. Recover SW1’s F0/1 manually (`shutdown`, `no shutdown`).

**Analogy**: Port security is like a club with a strict guest list (MACs). SW1’s doors (F0/1–3) lock out fakes (shutdown), while SW2’s door (G0/1) logs and blocks fakes (restrict) but lets approved guests (PC1–3, SW1) pass.

**Lifelong Retention Tip**: Visualize SW1 locking F0/1 (PC1 MAC change) and SW2 logging violations (SVI ping). Create flashcards: “Port security? MAC control. Shutdown? Err-disabled. Restrict? Drops, logs. Sticky? Auto-config.” Practice in Packet Tracer: configure, test violations, recover. Review 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure Port Security on SW1
**Objective**: Enable port security on SW1’s F0/1–3 for PC1–3, max 1 MAC, shutdown mode, aging 60 minutes, no sticky learning.

**Steps (on SW1)**:
1. Enter global configuration mode:
```plaintext
SW1> enable
SW1# configure terminal
```
2. Configure interfaces F0/1–3:
```plaintext
SW1(config)# interface range f0/1 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
```
- Note: `switchport mode access` required (not dynamic auto, Day 22). Command rejected if dynamic.
3. Set aging time (default 0 mins, no aging):
```plaintext
SW1(config-if-range)# switchport port-security aging time 60
```
4. Verify defaults (max 1 MAC, shutdown mode, no sticky) are correct, no further config needed.
5. Check configuration:
```plaintext
SW1# show port-security interface f0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 1
Total MAC Addresses         : 0
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : 0000.0000.0000:0
Security Violation Count    : 0
```

**Observations**:
- **Port Security**: Enabled on F0/1–3, max 1 MAC per port (A.A.A, B.B.B, C.C.C).
- **Violation Mode**: Shutdown (err-disabled on violation).
- **Aging**: 60 minutes, absolute (default, timer runs regardless of traffic).
- **Sticky**: Disabled (default, dynamic learning).
- **Exam Note**: Topic 5.7 requires configuring `switchport port-security`, `aging time`, and static access mode.

**Comparison to Prior Content**:
- **Port Security Lecture**: Configures shutdown mode, max 1 MAC, aging (topic 5.7).
- **Security Fundamentals Lecture**: Mitigates unauthorized access (Confidentiality, topic 5.1).
- **Kali Linux Demo Lab**: Prevents DHCP exhaustion by limiting MACs.
- **Day 22 (DTP)**: Requires `switchport mode access`.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Complements dynamic ARP inspection.
- **Day 13 (DHCP)**: Protects DHCP pool access.
- **Day 10 (IP Addressing)**: Secures 10.0.0.0/24 network.

**Analogy**: SW1’s F0/1–3 are like locked doors, each allowing one key (MAC), locking on fake keys (shutdown) after 60 minutes (aging).

**Lifelong Retention Tip**: Configure F0/1–3 in Packet Tracer, verify with `show port-security`. Flashcard: “Port security? Max 1, shutdown, 60-min aging. Dynamic auto? Rejected.” Practice 3 times.

### Step 2: Configure Port Security on SW2
**Objective**: Enable port security on SW2’s G0/1, max 4 MACs (PC1, PC2, PC3, SW1), restrict mode, sticky learning.

**Steps (on SW2)**:
1. Enter global configuration mode:
```plaintext
SW2> enable
SW2# configure terminal
```
2. Configure G0/1:
```plaintext
SW2(config)# interface g0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport port-security
SW2(config-if)# switchport port-security violation restrict
SW2(config-if)# switchport port-security maximum 4
SW2(config-if)# switchport port-security mac-address sticky
```
3. Verify configuration:
```plaintext
SW2# show port-security interface g0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 4
Total MAC Addresses         : 0
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : 0000.0000.0000:0
Security Violation Count    : 0
```

**Observations**:
- **Port Security**: Enabled on G0/1, max 4 MACs (PC1, PC2, PC3, SW1’s G0/0).
- **Violation Mode**: Restrict (drops unauthorized traffic, port stays up, increments violation count).
- **Sticky Learning**: Enabled, learned MACs added to running-config.
- **Max 4 MACs**: Accounts for PC1–3 and SW1’s G0/0 (CDP frames carry SW1’s MAC).
- **Exam Note**: Topic 5.7 requires configuring `violation restrict`, `maximum`, `sticky`.

**Comparison to Prior Content**:
- **Port Security Lecture**: Configures restrict mode, sticky MACs, max >1 (topic 5.7).
- **Kali Linux Demo Lab**: Max 4 MACs prevents DHCP exhaustion on G0/1.
- **Security Fundamentals Lecture**: Restrict mode mitigates spoofing (Confidentiality, topic 5.1).
- **Day 22 (DTP)**: `switchport mode access` required.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Sticky MACs secure against ARP spoofing.
- **Day 13 (DHCP)**: Protects DHCP traffic from PC1–3.
- **Day 10 (IP Addressing)**: Secures 10.0.0.0/24 access.

**Analogy**: SW2’s G0/1 is a door allowing 4 guests (MACs), logging fakes (restrict) and saving approved IDs (sticky).

**Lifelong Retention Tip**: Configure G0/1 in Packet Tracer, enable sticky, verify with `show running-config`. Flashcard: “Restrict? Drops, logs, counts. Sticky? Auto-config MACs. Max 4? PC1–3, SW1.” Practice 3 times.

### Step 3: Test MAC Learning
**Objective**: Ping from PC1–3 to R1, verify SW1/SW2 learn MACs.

**Steps**:
1. **On PC1**:
```plaintext
C:\> ping 10.0.0.254
Reply from 10.0.0.254: bytes=32 time=1ms TTL=255
```
2. **On PC2**:
```plaintext
C:\> ping 10.0.0.254
Reply from 10.0.0.254: bytes=32 time=1ms TTL=255
```
3. **On PC3**:
```plaintext
C:\> ping 10.0.0.254
Reply from 10.0.0.254: bytes=32 time=1ms TTL=255
```
4. **On SW2**:
```plaintext
SW2# show port-security interface g0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 4
Total MAC Addresses         : 4
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 4
Last Source Address:Vlan    : C.C.C:1
Security Violation Count    : 0
```
5. Check running-config:
```plaintext
SW2# show running-config
interface GigabitEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 4
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky A.A.A
 switchport port-security mac-address sticky B.B.B
 switchport port-security mac-address sticky C.C.C
 switchport port-security mac-address sticky D.D.D
```
6. Check MAC address table:
```plaintext
SW2# show mac address-table
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    A.A.A             STATIC      G0/1
   1    B.B.B             STATIC      G0/1
   1    C.C.C             STATIC      G0/1
   1    D.D.D             STATIC      G0/1
```
7. Overview:
```plaintext
SW2# show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)      (Count)
---------------------------------------------------------------------------
Gi0/1              4              4            0           Restrict
---------------------------------------------------------------------------
```

**Observations**:
- **SW1**: F0/1–3 learn A.A.A, B.B.B, C.C.C (dynamic, max 1 each).
- **SW2**: G0/1 learns A.A.A, B.B.B, C.C.C (PC1–3), D.D.D (SW1’s G0/0 via CDP), sticky, type STATIC in MAC table.
- **Pings**: All succeed, MACs learned correctly.
- **Exam Note**: Verify sticky MACs in running-config, `show mac address-table` (topic 5.7).

**Comparison to Prior Content**:
- **Port Security Lecture**: Sticky MACs auto-added to running-config, restrict mode allows authorized traffic.
- **Kali Linux Demo Lab**: Max 4 MACs on G0/1 prevents DHCP exhaustion.
- **Security Fundamentals Lecture**: Secures Confidentiality (authorized MACs).
- **Day 13 (DHCP)**: Ensures PC1–3 get IPs securely.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Sticky MACs protect against ARP spoofing.
- **Day 10 (IP Addressing)**: Secures 10.0.0.0/24 access.
- **Day 15 (TCP/UDP)**: Protects ping (ICMP) traffic.

**Analogy**: Pings are like guests checking in, SW2’s G0/1 saves their IDs (sticky MACs) up to 4.

**Lifelong Retention Tip**: Simulate pings in Packet Tracer, verify sticky MACs in `show running-config`. Flashcard: “Sticky MACs? Running-config, STATIC. Max 4? PC1–3, SW1.” Practice 3 times.

### Step 4: Trigger Violations
**Objective**: Trigger violations on SW1 (PC1 MAC change, shutdown) and SW2 (SW1 SVI ping, restrict).

**Steps**:
1. **On SW1 (SVI Violation)**:
   - Configure VLAN 1 SVI:
```plaintext
SW1(config)# interface vlan 1
SW1(config-if)# ip address 10.0.0.10 255.255.255.0
SW1(config-if)# no shutdown
```
   - Ping R1:
```plaintext
SW1# ping 10.0.0.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.254, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```
2. **On SW2 (Verify Violation)**:
```plaintext
SW2# show port-security interface g0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 4
Total MAC Addresses         : 4
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 4
Last Source Address:Vlan    : E.E.E:1
Security Violation Count    : 5
```
3. **On PC1 (MAC Change)**:
   - Change MAC from A.A.A to A.A.B (Packet Tracer: Config → FastEthernet0 → MAC Address).
   - Ping R1:
```plaintext
C:\> ping 10.0.0.254
Request timed out.
```
4. **On SW1 (Verify Violation)**:
```plaintext
SW1# show port-security interface f0/1
Port Security              : Enabled
Port Status                : Secure-shutdown
Violation Mode             : Shutdown
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 1
Total MAC Addresses         : 0
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : A.A.B:1
Security Violation Count    : 1
```

**Observations**:
- **SW2 (Restrict)**:
  - SW1’s SVI (MAC E.E.E) pings trigger violation (not in G0/1’s 4 MACs).
  - Port stays secure-up, violation count = 5 (one per ping).
  - No Syslog in Packet Tracer (limitation, real devices show `%PORT_SECURITY-2-PSECURE_VIOLATION`).
- **SW1 (Shutdown)**:
  - PC1’s new MAC (A.A.B) triggers violation on F0/1.
  - Port enters secure-shutdown (err-disabled), violation count = 1.
  - Syslog limited in Packet Tracer.
- **Exam Note**: Topic 5.7 tests violation modes (shutdown: err-disabled, restrict: drops, counts).

**Comparison to Prior Content**:
- **Port Security Lecture**: Shutdown (err-disabled), restrict (drops, counts) behaviors verified.
- **Kali Linux Demo Lab**: Max MACs prevent spoofed MAC attacks.
- **Security Fundamentals Lecture**: Mitigates spoofing (Confidentiality, topic 5.1).
- **Syslog Lab**: Violations trigger `%PORT_SECURITY-2-PSECURE_VIOLATION` (not in Packet Tracer).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Prevents MAC spoofing (e.g., A.A.B).
- **Day 10 (IP Addressing)**: Secures 10.0.0.0/24 access.
- **Day 15 (TCP/UDP)**: Protects ICMP (ping) traffic.

**Analogy**: SW1 locks the door (F0/1, shutdown) on fake IDs (A.A.B), SW2 logs fakes (G0/1, restrict) but lets approved guests pass.

**Lifelong Retention Tip**: Simulate violations in Packet Tracer (MAC change, SVI ping). Flashcard: “Shutdown? Err-disabled, count 1. Restrict? Drops, counts, secure-up.” Practice 3 times.

### Step 5: Recover SW1’s F0/1
**Objective**: Manually re-enable SW1’s F0/1 after PC1’s violation (Packet Tracer lacks ErrDisable recovery).

**Steps (on SW1)**:
1. Restore PC1’s MAC to A.A.A (Packet Tracer: Config → FastEthernet0 → MAC Address).
2. Re-enable F0/1:
```plaintext
SW1(config)# interface f0/1
SW1(config-if)# shutdown
SW1(config-if)# no shutdown
```
3. Verify:
```plaintext
SW1# show port-security interface f0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 1
Total MAC Addresses         : 0
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : 0000.0000.0000:0
Security Violation Count    : 0
```
4. Test PC1 ping:
```plaintext
C:\> ping 10.0.0.254
Reply from 10.0.0.254: bytes=32 time=1ms TTL=255
```

**Observations**:
- **Recovery**: `shutdown`, `no shutdown` re-enables F0/1 (secure-up).
- **Violation Count**: Resets to 0.
- **PC1**: MAC A.A.A restored, ping succeeds.
- **Exam Note**: Topic 5.7 requires manual recovery (`shutdown`, `no shutdown`), ErrDisable recovery not in Packet Tracer.

**Comparison to Prior Content**:
- **Port Security Lecture**: Manual recovery (`shutdown`, `no shutdown`) vs. ErrDisable recovery.
- **Security Fundamentals Lecture**: Restores Confidentiality/Availability (topic 5.1).
- **Kali Linux Demo Lab**: Recovery ensures legitimate access post-attack.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Restores 10.0.0.1 connectivity.
- **Day 15 (TCP/UDP)**: Enables ICMP pings.
- **Day 18 (ARP)**: Ensures valid MAC for ARP.

**Analogy**: Re-enabling F0/1 is like unlocking the door after removing the fake guest (A.A.B), letting A.A.A back in.

**Lifelong Retention Tip**: Simulate recovery in Packet Tracer, use `shutdown`, `no shutdown`. Flashcard: “Recover? `shutdown`, `no shutdown`. Err-disabled? Violation.” Practice 3 times.

## Common Issues and Troubleshooting

- **Port Security Not Enabled**:
  - **Issue**: `switchport port-security` rejected on F0/1–3.
  - **Cause**: Dynamic auto mode (DTP).
  - **Fix**: `switchport mode access`.
  - **Verify**: `show interfaces f0/1 switchport`.
- **Interface Err-Disabled**:
  - **Issue**: F0/1 secure-shutdown after PC1 MAC change.
  - **Cause**: Unauthorized MAC (A.A.B).
  - **Fix**: Restore MAC to A.A.A, `shutdown`, `no shutdown`.
  - **Verify**: `show port-security interface f0/1`.
- **Violation Count Increments (SW2)**:
  - **Issue**: G0/1 violation count >0, no Syslog.
  - **Cause**: SW1 SVI MAC (E.E.E) unauthorized, Packet Tracer Syslog limitation.
  - **Fix**: Stop SVI pings, verify authorized MACs.
  - **Verify**: `show port-security interface g0/1`.
- **Sticky MACs Lost**:
  - **Issue**: SW2’s G0/1 MACs gone after reload.
  - **Cause**: Running-config not saved.
  - **Fix**: `write memory`.
  - **Verify**: `show running-config`.
- **Insufficient Max MACs**:
  - **Issue**: SW2’s G0/1 violations with PC1–3, SW1.
  - **Cause**: Max <4.
  - **Fix**: `switchport port-security maximum 4`.
  - **Verify**: `show port-security`.

**Analogy**: Troubleshooting is like fixing a club’s entry system: set the right door mode (access), unlock after fakes (recovery), save the guest list (sticky), and allow enough guests (max 4).

**Lifelong Retention Tip**: Simulate issues in Packet Tracer (MAC change, dynamic mode), fix and verify. Checklist: “Err-disabled? Check MAC, mode, recovery. Sticky lost? Save config.” Practice 3 times.

## Additional CCNA-Relevant Information

**Why This Matters for CCNA**:
- **Exam Topic 5.7**: Configures/verifies port security (`switchport port-security`, `violation`, `maximum`, `sticky`, `aging`).
- **Exam Topic 5.1**: Mitigates unauthorized access (Confidentiality) and spoofing/DoS (Availability, Kali Linux Demo Lab).
- **Exam Topic 2.2**: Complements DHCP security by limiting MACs.
- **Packet Tracer Limitations**: No ErrDisable recovery, limited Syslog for restrict mode. Real devices show `%PORT_SECURITY-2-PSECURE_VIOLATION`.

**Key CCNA Concepts Reinforced**:
- **Port Security**:
  - Shutdown: Err-disables port, single Syslog, violation count 1.
  - Restrict: Drops unauthorized traffic, port up, counts violations, Syslog.
  - Sticky: Auto-adds learned MACs to running-config, type STATIC.
- **DTP (Day 22)**: `switchport mode access` required for port security.
- **MAC Address Table**: Sticky MACs appear as STATIC, dynamic as DYNAMIC.
- **Syslog**: Violations trigger `%PORT_SECURITY-2-PSECURE_VIOLATION` (not in Packet Tracer).

**Additional Configurations (Not in Lab)**:
- Static MAC:
```plaintext
SW1(config-if)# switchport port-security mac-address A.A.A
```
- Aging Type:
```plaintext
SW1(config-if)# switchport port-security aging type inactivity
```
- Static Aging:
```plaintext
SW1(config-if)# switchport port-security aging static
```

**Practice in Packet Tracer**:
1. Replicate lab: Configure SW1 (F0/1–3, shutdown, max 1), SW2 (G0/1, restrict, max 4, sticky).
2. Test violations: Change PC1’s MAC, ping from SW1’s SVI.
3. Recover F0/1 manually, verify MACs in `show running-config`.
4. Experiment: Try `violation protect`, static MACs, aging 30 mins.

**Additional Resources**:
- **Cisco Documentation**: Review port security (topic 5.7) in CCNA study guide.
- **Boson ExSim**: Practice questions on port security, violation modes, recovery.
- **Cloudflare Articles**: Search “MAC address spoofing” or “port security Cisco” for real-world context.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Violation Modes: “Shutdown, Restrict, Protect” (SRP).
   - Config Steps: “Access, Enable, Max, Sticky, Violation” (AEMSV).
   - Recovery: “Shutdown, No Shutdown” (SNS).
2. **Visualize Topology**:
   - Draw PC1–3 → SW1 (F0/1–3) → SW2 (G0/1) → R1. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Shutdown? Err-disabled, count 1. Restrict? Drops, counts, Syslog. Sticky? Running-config, STATIC. Max 4? PC1–3, SW1.”
4. **Practice in Packet Tracer**:
   - Configure port security, test shutdown/restrict, recover. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Shutdown (err-disabled) vs. Restrict (drops, counts) vs. Protect (drops silently).
   - Quiz: “Restrict? Drops, counts, secure-up. Sticky? Auto-config MACs.”
6. **Troubleshooting Practice**:
   - Simulate MAC change, dynamic mode error, recover. Checklist: “Err-disabled? Check MAC, mode, recovery.”
7. **Teach Someone**:
   - Explain port security setup and violation modes to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`switchport port-security`), modes, recovery. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Port Security on SW1**:
   - F0/1–3: Max 1 MAC (A.A.A, B.B.B, C.C.C), shutdown mode, 60-min aging, dynamic learning.
   - Prevents unauthorized access (e.g., PC1 MAC A.A.B triggers err-disabled).
2. **Port Security on SW2**:
   - G0/1: Max 4 MACs (PC1–3, SW1), restrict mode, sticky learning.
   - Drops unauthorized traffic (e.g., SW1 SVI), counts violations, stays up.
3. **Verification**:
   - `show port-security`, `show port-security interface`, `show running-config`, `show mac address-table`.
   - Sticky MACs in running-config, STATIC in MAC table.
4. **Recovery**:
   - Manual: `shutdown`, `no shutdown` (Packet Tracer limitation, no ErrDisable recovery).
5. **CCNA Focus**:
   - Topic 5.7: Configure/verify port security (`maximum`, `violation`, `sticky`, `aging`).
   - Topic 5.1: Mitigates spoofing/DoS (Confidentiality, Availability).
   - Topic 2.2: Complements DHCP security.
6. **Packet Tracer**: Limited Syslog, no ErrDisable recovery, but sufficient for topic 5.7.

These notes provide a comprehensive foundation for the Port Security Lab (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/modes, and apply analogies/retention tips to master concepts for life!