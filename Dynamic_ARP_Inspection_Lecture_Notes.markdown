# Dynamic ARP Inspection (DAI) Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **Dynamic ARP Inspection (DAI)**, a Cisco switch security feature that protects against ARP-based attacks, as outlined in **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **DHCP Snooping Lecture**, **Port Security Lecture**, **Security Fundamentals Lecture** (topic 5.1), **ARP Lecture** (Day 18), and **Kali Linux Demo Lab** (DHCP-related attacks). The lecture explains DAI’s purpose, operation, mitigation of ARP poisoning, configuration, and troubleshooting, preparing you for CCNA exam tasks.

**Lecture Objectives**:
- **Understand DAI**: Define DAI and its role in filtering ARP messages.
- **Explain Benefits**: Prevent ARP poisoning (Confidentiality, topic 5.1).
- **Configure DAI**: Enable per VLAN, set trusted ports, configure rate-limiting, ARP ACLs, and optional validations.
- **Verify and Troubleshoot**: Use `show` commands, recover from rate-limit violations.
- **Prepare for CCNA**: Master topic 5.7 (DAI configuration/verification) and connect to topics 5.1 (security concepts) and 2.4 (ARP).
- **Reinforce Prior Content**: Link to ARP (Day 18), DHCP Snooping, Port Security, Kali Linux Demo Lab.

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      SW2       |-------|      R1        |
| 192.168.1.10   | G0/1  | G0/0           | G0/1  | G0/0           |       | 192.168.1.1    |
| (Client)       |       |                |       |                |       | (Gateway)      |
+----------------+       +----------------+       +----------------+       +----------------+
|      PC2       | G0/2  |
| (Attacker)     |       |
+----------------+       |
|      SRV1      | G0/3  |
| 192.168.1.100  |       |
| (Static IP)    |       |
+----------------+       |
```
- **Devices**:
  - **PC1**: DHCP client on SW1 (G0/1, untrusted), IP 192.168.1.10.
  - **PC2**: Attacker on SW1 (G0/2, untrusted), sends spoofed ARP messages.
  - **SRV1**: Static IP server on SW1 (G0/3, untrusted), IP 192.168.1.100.
  - **SW1**: Cisco switch, DAI enabled, G0/1–3 untrusted (to PC1, PC2, SRV1), G0/0 trusted (to SW2).
  - **SW2**: Cisco switch, DAI enabled, G0/1 untrusted (to SW1), G0/0 trusted (to R1).
  - **R1**: Default gateway, IP 192.168.1.1.
- **Note**:
  - **Untrusted Ports**: SW1’s G0/1–3 (PC1, PC2, SRV1), SW2’s G0/1 (SW1), inspected for ARP messages.
  - **Trusted Ports**: SW1’s G0/0 (to SW2), SW2’s G0/0 (to R1), no inspection.
  - **VLAN**: Single VLAN 1 (192.168.1.0/24).
  - **Dependencies**: Relies on DHCP snooping binding table; ARP ACLs for static IPs (SRV1).
- **Requirements**:
  1. Configure DAI on SW1/SW2 for VLAN 1, trust G0/0 ports.
  2. Set rate-limiting on SW1’s G0/1–3 to prevent ARP floods.
  3. Configure ARP ACL for SRV1 (static IP).
  4. Test PC1’s ARP requests, PC2’s spoofed ARP, and SRV1’s ARP.
  5. Verify with `show ip arp inspection`, recover from rate-limit violations.

**Analogy**: DAI is like a club’s security guard (SW1/SW2) checking IDs (ARP messages) at untrusted doors (G0/1–3), verifying against a guest list (DHCP snooping table or ARP ACLs), trusting VIP doors (G0/0), and limiting entry speed (rate-limiting).

**Lifelong Retention Tip**: Visualize PC2’s spoofed ARP (claiming R1’s IP) dropped by SW1. Create flashcards: “DAI? Filters ARP untrusted ports. Trusted? Uplinks. Rate-limit? 15 pps default.” Practice in Packet Tracer: configure, test spoofing, verify. Review 3 times daily for a week.

## DAI Fundamentals

**Purpose**: Define DAI and its role in filtering ARP messages (topic 5.7).

**Key Points**:
- **Definition**: Cisco switch feature that filters ARP messages (requests, replies, gratuitous) on untrusted ports to prevent attacks.
- **Scope**: Filters only ARP messages, not other traffic (e.g., ICMP, DHCP).
- **Port Trust**:
  - **Untrusted (default)**: Downlink ports (e.g., SW1’s G0/1–3, SW2’s G0/1), inspected.
  - **Trusted**: Uplink ports (e.g., SW1’s G0/0, SW2’s G0/0), no inspection.
- **Operation**:
  - **Trusted Ports**: Forward ARP messages without checks.
  - **Untrusted Ports**: Check sender MAC/IP against DHCP snooping binding table or ARP ACLs.
    - Match: Forward.
    - No match: Discard (e.g., PC2’s spoofed ARP).
- **Dependencies**:
  - **DHCP Snooping**: Binding table (MAC, IP, interface, VLAN) validates ARP messages.
  - **ARP ACLs**: Manual mappings for static IP devices (e.g., SRV1).
- **Binding Table Example**:
```plaintext
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               192.168.1.10     86400       dhcp-snooping  1     GigabitEthernet0/1
```
- **Exam Note**: Topic 5.7 requires configuring/verifying DAI, understanding trusted/untrusted ports, and DHCP snooping dependency.

**Demonstration**:
- **Topology**: PC1 → SW1 (G0/1 untrusted) → SW2 (G0/1 untrusted, G0/0 trusted) → R1.
- **Scenario**:
  - PC1 sends ARP request (sender MAC A.A.A, IP 192.168.1.10) for R1 (192.168.1.1).
  - SW1 (G0/1 untrusted) checks A.A.A/192.168.1.10 against binding table, forwards.
  - SW2 (G0/1 untrusted) forwards (trusted G0/0 to R1).
  - R1’s reply (MAC D.D.D, IP 192.168.1.1) passes trusted G0/0 (SW2), G0/0 (SW1).
  - PC2 sends gratuitous ARP (MAC B.B.B, IP 192.168.1.1). SW1 (G0/2 untrusted) discards (no binding table match).
- **Verification**:
```plaintext
SW1# show ip arp inspection
Source Mac Validation      : Enabled
Destination Mac Validation : Enabled
IP Address Validation      : Enabled
Vlan  Configuration    Operation   ACL Match   Static ACL
----  -------------    ---------   ---------   ----------
  1   Enabled          Active      ARP-ACL-1   No
Vlan  ACL Logging   DHCP Logging   Probe Logging
----  -----------   ------------   -------------
  1   Deny          Deny           Off
Vlan  Forwarded  Dropped  DHCP Drops  ACL Drops  DHCP Permits  ACL Permits
----  ---------  -------  ----------  ---------  ------------  -----------
  1       10        4         4          0           10           1
```

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: DAI relies on binding table (topic 5.7).
- **Port Security Lecture**: Complements MAC limiting (topic 5.7).
- **Security Fundamentals Lecture**: Mitigates ARP poisoning (Confidentiality, topic 5.1).
- **ARP Lecture (Day 18)**: Builds on ARP request/reply, introduces gratuitous ARP.
- **Kali Linux Demo Lab**: Prevents spoofing attacks.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 ARP mappings.
- **Day 15 (TCP/UDP)**: Protects ARP for IP communications.
- **Day 2 (Ethernet)**: Validates Ethernet frame MACs.

**Analogy**: DAI is a gatekeeper checking ARP IDs (sender MAC/IP) against a guest list (binding table) at untrusted doors (G0/1–3), trusting VIP doors (G0/0).

**Lifelong Retention Tip**: Simulate PC2’s spoofed ARP in Packet Tracer, verify drop. Flashcard: “DAI? Checks ARP untrusted ports. Binding table? MAC/IP match. Trusted? G0/0.” Practice 3 times.

## ARP-Based Attacks and Mitigation

**Purpose**: Explain ARP poisoning and how DAI mitigates it (topics 5.1, 5.7).

**Key Points**:
- **ARP Poisoning**:
  - **Attack**: Attacker (PC2, G0/2) sends gratuitous ARP (MAC B.B.B, IP 192.168.1.1, R1’s IP), broadcast to all devices.
  - **Impact**: PC1 updates ARP table (192.168.1.1 → B.B.B), sends traffic to PC2 (MITM, Confidentiality, topic 5.1).
  - **Process**:
    - PC2’s GARP floods network (destination MAC FFFF.FFFF.FFFF).
    - PC1 maps 192.168.1.1 to B.B.B, sends external traffic (e.g., DNS to 8.8.8.8) to PC2.
    - PC2 inspects/modifies, forwards to R1 (192.168.1.1, MAC D.D.D).
    - R1 ignores GARP (own IP).
  - **Prior Content**: ARP Lecture (Day 18) introduced gratuitous ARP; DHCP Snooping Lecture showed similar poisoning.
- **DAI Mitigation**:
  - **Untrusted Ports**: SW1’s G0/2 checks PC2’s GARP (B.B.B, 192.168.1.1) against binding table, discards (no match).
  - **Trusted Ports**: SW1’s G0/0, SW2’s G0/0 allow R1’s legitimate ARP replies.
  - **Binding Table**: Validates PC1 (A.A.A, 192.168.1.10), not PC2 (B.B.B, no IP).
  - **ARP ACLs**: Allow SRV1 (static IP 192.168.1.100, no binding table entry).
- **Exam Note**: Topic 5.7 tests DAI mitigation of ARP poisoning; topic 5.1 covers MITM.

**Demonstration**:
- **Topology**: PC2 (G0/2) sends GARP (B.B.B, 192.168.1.1).
- **Poisoning**:
  - Without DAI: PC1 updates ARP table, sends traffic to PC2 (MITM).
  - With DAI: SW1 (G0/2 untrusted) discards GARP (no binding table match).
- **SRV1 (Static IP)**:
  - ARP request (192.168.1.100) dropped (no binding table entry).
  - Fix: ARP ACL permits SRV1’s MAC/IP.
- **Verification**:
```plaintext
SW1# show ip arp inspection
Vlan  Forwarded  Dropped  DHCP Drops  ACL Drops  DHCP Permits  ACL Permits
----  ---------  -------  ----------  ---------  ------------  -----------
  1       10        4         4          0           10           1
```
- **Syslog**: `%SW_DAI-4-DHCP_SNOOPING_DENY: 1 Invalid ARPs (Req) on G0/2, vlan 1`.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Similar to DHCP poisoning (MITM), uses binding table.
- **Port Security Lecture**: Complements MAC limiting.
- **Security Fundamentals Lecture**: Mitigates MITM (Confidentiality, topic 5.1).
- **ARP Lecture (Day 18)**: GARP used in poisoning.
- **Kali Linux Demo Lab**: Prevents spoofing attacks.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Protects 192.168.1.0/24 ARP mappings.
- **Day 15 (TCP/UDP)**: Secures ARP for IP traffic (e.g., DNS).
- **Day 2 (Ethernet)**: Validates Ethernet frame MACs.

**Analogy**: DAI stops fake IDs (PC2’s GARP) at untrusted gates (G0/2), checking against a guest list (binding table), ensuring only valid guests (PC1, R1) pass.

**Lifelong Retention Tip**: Simulate PC2’s GARP in Packet Tracer, verify drop. Flashcard: “ARP poisoning? GARP spoofs IP. DAI? Drops untrusted, no binding match.” Practice 3 times.

## ARP Message Types and DAI Operation

**Purpose**: Review ARP message types and explain DAI filtering (topics 5.7, 2.4).

**Key Points**:
- **ARP Message Types**:
  - **Request**: Broadcast (FFFF.FFFF.FFFF), asks for MAC of target IP (e.g., PC1: A.A.A, 192.168.1.10 → 192.168.1.1).
  - **Reply**: Unicast, provides MAC for requested IP (e.g., R1: D.D.D, 192.168.1.1 → PC1).
  - **Gratuitous ARP (GARP)**: Unsolicited reply, broadcast, updates ARP tables (e.g., PC2’s spoofed GARP).
- **DAI Operation**:
  - **Trusted Ports**: Forward all ARP messages (e.g., SW2’s G0/0 to R1).
  - **Untrusted Ports**:
    - Check sender MAC/IP (ARP message) against DHCP snooping binding table or ARP ACLs.
    - Match: Forward (e.g., PC1’s A.A.A, 192.168.1.10).
    - No match: Discard (e.g., PC2’s B.B.B, 192.168.1.1).
  - **Optional Validations**:
    - **Source MAC**: Ethernet frame source MAC vs. ARP sender MAC.
    - **Destination MAC**: Ethernet frame destination MAC vs. ARP target MAC (replies only).
    - **IP**: Sender/target IP not invalid (e.g., 0.0.0.0, 255.255.255.255, multicast).
- **Wireshark Example**:
  - ARP Request (PC1):
    - Ethernet: Source MAC A.A.A, Destination MAC FFFF.FFFF.FFFF.
    - ARP: Sender MAC A.A.A, IP 192.168.1.10; Target IP 192.168.1.1.
  - ARP Reply (R1):
    - Ethernet: Source MAC D.D.D, Destination MAC A.A.A.
    - ARP: Sender MAC D.D.D, IP 192.168.1.1; Target MAC A.A.A, IP 192.168.1.10.
- **Exam Note**: Topic 5.7 requires knowing ARP message types and DAI checks; topic 2.4 covers ARP.

**Demonstration**:
- **Topology**: PC1 (G0/1) sends ARP request, PC2 (G0/2) sends GARP.
- **Operation**:
  - PC1’s request (A.A.A, 192.168.1.10): Matches binding table, forwarded.
  - PC2’s GARP (B.B.B, 192.168.1.1): No binding match, discarded.
  - SRV1’s request (C.C.C, 192.168.1.100): Dropped, fixed with ARP ACL.
- **Verification**:
```plaintext
SW1# show ip arp inspection
Vlan  Forwarded  Dropped  DHCP Drops  ACL Drops  DHCP Permits  ACL Permits
----  ---------  -------  ----------  ---------  ------------  -----------
  1       10        4         4          0           10           1
```

**Comparison to Prior Content**:
- **ARP Lecture (Day 18)**: Covers request/reply, GARP.
- **DHCP Snooping Lecture**: Similar untrusted port filtering, binding table.
- **Port Security Lecture**: Complements MAC checks.
- **Security Fundamentals Lecture**: Protects Confidentiality (MITM, topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Validates 192.168.1.0/24 IPs.
- **Day 15 (TCP/UDP)**: Enables ARP for IP traffic.
- **Day 2 (Ethernet)**: Checks Ethernet frame MACs.

**Analogy**: DAI checks ARP tickets (requests, replies, GARP) at untrusted gates, matching IDs (MAC/IP) to a guest list (binding table/ACLs).

**Lifelong Retention Tip**: Memorize ARP types (request, reply, GARP). Simulate in Packet Tracer, verify drops. Flashcard: “DAI checks? Sender MAC/IP vs. binding table. GARP? Broadcast, spoof risk.” Practice 3 times.

## DAI Configuration

**Purpose**: Configure and verify DAI, including trusted ports, rate-limiting, validations, and ARP ACLs (topic 5.7).

**Key Commands**:
```plaintext
SW1# configure terminal
SW1(config)# ip arp inspection vlan 1
SW1(config)# interface g0/0
SW1(config-if)# ip arp inspection trust
SW1(config)# interface range g0/1 - 3
SW1(config-if-range)# ip arp inspection limit rate 25 burst-interval 2
SW1(config)# errdisable recovery cause arp-inspection
SW1(config)# ip arp inspection validate src-mac dst-mac ip
SW1(config)# arp access-list ARP-ACL-1
SW1(config-arp-acl)# permit ip host 192.168.1.100 mac host C.C.C
SW1(config)# ip arp inspection filter ARP-ACL-1 vlan 1
```

**Key Points**:
- **Enable DAI**:
  - Per VLAN: `ip arp inspection vlan 1` (no global command, unlike DHCP snooping).
- **Trusted Ports**:
  - Default: All ports untrusted.
  - Configure: `ip arp inspection trust` on uplinks (SW1’s G0/0, SW2’s G0/0).
- **Rate-Limiting**:
  - Default: 15 packets/sec (pps) on untrusted ports, disabled on trusted.
  - Configure: `ip arp inspection limit rate 25 burst-interval 2` (25 packets/2 sec).
  - Exceed: Port err-disabled, Syslog `%SW_DAI-4-DHCP_SNOOPING_RATE_LIMIT_EXCEEDED`.
  - Recover: `shutdown`, `no shutdown` or `errdisable recovery cause arp-inspection`.
- **Optional Validations**:
  - Configure: `ip arp inspection validate src-mac dst-mac ip` (all in one command).
  - **Source MAC**: Ethernet source MAC vs. ARP sender MAC.
  - **Destination MAC**: Ethernet destination MAC vs. ARP target MAC (replies).
  - **IP**: No invalid IPs (0.0.0.0, 255.255.255.255, multicast).
- **ARP ACLs**:
  - For static IPs (e.g., SRV1: 192.168.1.100, C.C.C).
  - Configure: `arp access-list`, `permit ip host`, apply with `ip arp inspection filter`.
  - Static ACL (beyond CCNA): Implicit deny if enabled, overrides binding table.
- **Verification Commands**:
```plaintext
SW1# show ip arp inspection
SW1# show ip arp inspection interfaces
SW1# show errdisable recovery
SW1# show running-config | include validate
SW1# show arp access-list
```

**Demonstration**:
- **Topology**: SW1 (G0/0 trusted, G0/1–3 untrusted, rate-limit 25/2 sec), SW2 (G0/0 trusted).
- **Scenario 1 (Legitimate ARP)**:
  - Configure: `ip arp inspection vlan 1`, trust G0/0, validate src-mac dst-mac ip.
  - PC1’s ARP request (A.A.A, 192.168.1.10) matches binding table, forwarded.
  - Verify:
```plaintext
SW1# show ip arp inspection
Vlan  Forwarded  Dropped  DHCP Drops  ACL Drops  DHCP Permits  ACL Permits
----  ---------  -------  ----------  ---------  ------------  -----------
  1       10        4         4          0           10           0
```
- **Scenario 2 (ARP Poisoning)**:
  - PC2’s GARP (B.B.B, 192.168.1.1) dropped (no binding match).
  - Syslog: `%SW_DAI-4-DHCP_SNOOPING_DENY`.
- **Scenario 3 (Static IP)**:
  - SRV1’s ARP (C.C.C, 192.168.1.100) dropped.
  - Fix: ARP ACL `ARP-ACL-1` permits C.C.C/192.168.1.100.
  - Verify: `ACL Permits: 1`.
- **Scenario 4 (Rate-Limit)**:
  - PC2 floods ARPs (>25/2 sec). G0/2 err-disabled.
  - Recover: `errdisable recovery cause arp-inspection`.
  - Verify:
```plaintext
SW1# show errdisable recovery
ErrDisable Reason            Timer Status
-----------------            --------------
arp-inspection               Enabled
Timer interval: 300 seconds
Interfaces that will be enabled at the next timeout:
Interface       Errdisable reason       Time left(sec)
---------       ---------------------   --------------
Gi0/2           arp-inspection          280
```

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: DAI uses binding table, similar untrusted port filtering.
- **Port Security Lecture**: Complements MAC limiting.
- **Security Fundamentals Lecture**: Mitigates MITM (topic 5.1).
- **ARP Lecture (Day 18)**: Validates ARP request/reply/GARP.
- **Kali Linux Demo Lab**: Prevents spoofing.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 mappings.
- **Day 15 (TCP/UDP)**: Enables ARP for IP traffic.
- **Day 2 (Ethernet)**: Checks Ethernet frame MACs.

**Analogy**: Configuring DAI is like setting a club’s security system: activate for VLAN 1, trust VIP doors (G0/0), check extra ID fields (validations), and allow special guests (ARP ACLs).

**Lifelong Retention Tip**: Configure DAI in Packet Tracer, test GARP drop, ARP ACL for SRV1. Flashcard: “DAI? VLAN 1, trust G0/0, validate src/dst/ip. Rate-limit? 15 pps default.” Practice 3 times.

## Common Issues and Troubleshooting

- **ARP Requests Dropped**:
  - **Issue**: PC1’s ARP request dropped.
  - **Cause**: G0/1 trusted (misconfigured) or DHCP snooping disabled.
  - **Fix**: Remove `ip arp inspection trust` from G0/1, enable `ip dhcp snooping vlan 1`.
  - **Verify**: `show ip arp inspection`, `show ip dhcp snooping binding`.
- **Static IP Dropped (SRV1)**:
  - **Issue**: SRV1’s ARP (192.168.1.100) dropped.
  - **Cause**: No binding table entry, no ARP ACL.
  - **Fix**: `arp access-list ARP-ACL-1`, `permit ip host 192.168.1.100 mac host C.C.C`, `ip arp inspection filter ARP-ACL-1 vlan 1`.
  - **Verify**: `show ip arp inspection` (ACL Permits).
- **Port Err-Disabled (Rate-Limit)**:
  - **Issue**: G0/2 err-disabled after PC2’s ARP flood.
  - **Cause**: Exceeds `ip arp inspection limit rate 25 burst-interval 2`.
  - **Fix**: `shutdown`, `no shutdown` or `errdisable recovery cause arp-inspection`.
  - **Verify**: `show errdisable recovery`, `show ip arp inspection`.
- **Validation Overwritten**:
  - **Issue**: Only `validate src-mac` active, not `dst-mac` or `ip`.
  - **Cause**: Multiple `ip arp inspection validate` commands overwrite.
  - **Fix**: `ip arp inspection validate src-mac dst-mac ip` (single command).
  - **Verify**: `show running-config | include validate`.
- **Syslog Not Seen**:
  - **Issue**: No `%SW_DAI-4-DHCP_SNOOPING_DENY`.
  - **Cause**: Packet Tracer limitation or logging disabled.
  - **Fix**: Ensure `logging console` (real devices).
  - **Verify**: Check Syslog in real devices.

**Analogy**: Troubleshooting is like fixing a club’s ID checker: ensure guest list (binding table), add special guests (ARP ACLs), reset locked doors (rate-limit), and check all ID fields (validations).

**Lifelong Retention Tip**: Simulate SRV1’s ARP drop, PC2’s flood in Packet Tracer. Checklist: “No ARP? Check binding, trust, ACLs. Err-disabled? Rate-limit, recover.” Practice 3 times.

## Quiz Answers and Explanations

1. **What is true after `ip arp inspection vlan 1` on SW1?**
   - **A**: All interfaces in VLAN 1 untrusted, **B**: All interfaces trusted, **C**: DAI disabled, **D**: Rate-limiting disabled.
   - **Answer**: A (All ports untrusted by default, topic 5.7).
   - **Explanation**: Must configure `ip arp inspection trust` manually.

2. **What is true after `ip arp inspection validate dst-mac`, then `ip`, then `src-mac`?**
   - **A**: All validations enabled, **B**: Only IP enabled, **C**: Only dst-mac enabled, **D**: Only src-mac enabled.
   - **Answer**: D (Last command overwrites, topic 5.7).
   - **Explanation**: Use single command: `ip arp inspection validate src-mac dst-mac ip`.

3. **What is true about DAI rate-limiting? (Select two)**
   - **A**: Disabled by default, **B**: Enabled on untrusted ports, **C**: Enabled on trusted ports, **D**: 15 pps default.
   - **Answer**: B, D (Enabled on untrusted ports, 15 pps default, topic 5.7).
   - **Explanation**: Unlike DHCP snooping (disabled), DAI enables rate-limiting; supports burst interval.

4. **What does DAI check sender IP/MAC against? (Select two)**
   - **A**: MAC address table, **B**: DHCP snooping binding table, **C**: Routing table, **D**: ARP ACLs.
   - **Answer**: B, D (Binding table for DHCP clients, ARP ACLs for static IPs, topic 5.7).
   - **Explanation**: Validates against binding table or ACLs, not MAC/routing tables.

5. **Which commands limit ARP to 15 pps? (Select two)**
   - **A**: `ip arp inspection limit rate 15`, **B**: `ip arp inspection limit rate 30 burst-interval 2`, **C**: `ip arp inspection limit rate 45 burst-interval 3`, **D**: `ip arp inspection limit rate 15 burst-interval 2`.
   - **Answer**: A, C (15 pps, 45/3 = 15 pps, topic 5.7).
   - **Explanation**: A enforces 15 pps per second; C averages 15 pps over 3 seconds.

**Bonus Question (Boson ExSim)**:
- **Question**: What is the default DAI rate-limit on untrusted ports?
  - **A**: Disabled, **B**: 10 pps, **C**: 15 pps, **D**: 20 pps.
  - **Answer**: C (15 pps, topic 5.7).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “DAI rate-limit? 15 pps untrusted. Validations? Single command.”). Simulate drops in Packet Tracer. Review 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - ARP Types: “Request, Reply, GARP” (RRG).
   - Config Steps: “VLAN, Trust, Rate-limit, Validate, ACL” (VTRVA).
   - Attacks: “Poisoning” (P).
2. **Visualize Topology**:
   - Draw PC1 → SW1 (G0/1 untrusted) → SW2 (G0/0 trusted) → R1, PC2/SRV1 on G0/2–3. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DAI? VLAN 1, trust G0/0. Rate-limit? 15 pps untrusted. ARP ACL? Static IPs.”
4. **Practice in Packet Tracer**:
   - Configure DAI, test PC2’s GARP, SRV1’s ARP with ACL. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: DHCP Snooping (binding table) vs. DAI (ARP checks).
   - Quiz: “GARP dropped? No binding match. Rate-limit? 15 pps default.”
6. **Troubleshooting Practice**:
   - Simulate SRV1’s ARP drop, rate-limit violation. Checklist: “No ARP? Check binding, ACL, trust. Err-disabled? Recover.”
7. **Teach Someone**:
   - Explain DAI and ARP poisoning to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`ip arp inspection`), validations, ACLs. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DAI**:
   - Filters ARP messages on untrusted ports (G0/1–3), trusts uplinks (G0/0).
   - Checks sender MAC/IP against DHCP snooping binding table or ARP ACLs.
2. **Attacks Mitigated**:
   - **ARP Poisoning**: Discards spoofed GARP (e.g., PC2’s B.B.B, 192.168.1.1), prevents MITM (topic 5.1).
3. **Configuration**:
   - Enable: `ip arp inspection vlan 1`, trust G0/0.
   - Rate-limit: `ip arp inspection limit rate 25 burst-interval 2` (untrusted).
   - Validations: `ip arp inspection validate src-mac dst-mac ip` (single command).
   - ARP ACLs: `arp access-list` for static IPs (SRV1).
4. **Verification**:
   - `show ip arp inspection`, `show ip arp inspection interfaces`, `show arp access-list`.
5. **CCNA Focus**:
   - Topic 5.7: Configure/verify DAI (`ip arp inspection`, `trust`, `validate`, `filter`).
   - Topic 5.1: Mitigates MITM (Confidentiality).
   - Topic 2.4: Secures ARP process.
6. **Dependencies**: Relies on DHCP snooping; ARP ACLs for static IPs.

These notes provide a comprehensive foundation for DAI (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/validations, and apply analogies/retention tips to master concepts for life!