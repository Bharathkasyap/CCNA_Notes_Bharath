# Dynamic ARP Inspection (DAI) Lab Notes for CCNA Beginners

## Lab Overview

This lab configures **Dynamic ARP Inspection (DAI)** on Cisco switches SW1 and SW2 in **Packet Tracer**, reinforcing **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **DAI Lecture**, **DHCP Snooping Lecture**, **Port Security Lecture**, **Security Fundamentals Lecture** (topic 5.1), **ARP Lecture** (Day 18), and **DHCP Lecture** (Day 13, topic 2.2). The lab sets up R1 as a DHCP server, enables DHCP snooping and DAI on SW1 and SW2 for VLAN 1, configures trusted ports and validation checks, and tests DHCP and ARP functionality for PC1–3.

**Lab Objectives**:
- **Configure R1 as DHCP Server**: Set up a DHCP pool for 192.168.1.0/24, excluding 192.168.1.1–9.
- **Configure DHCP Snooping**: Enable on SW1/SW2 for VLAN 1, trust uplink ports, disable Option 82.
- **Configure DAI**: Enable on SW1/SW2 for VLAN 1, trust uplink ports, enable all validation checks.
- **Test Functionality**: Verify PC1–3 get IPs via DHCP and can ping R1 (192.168.1.1).
- **Verify Configurations**: Use `show` commands to confirm DHCP snooping and DAI settings.
- **Prepare for CCNA**: Master topic 5.7 (DAI/DHCP snooping configuration/verification) and connect to topics 5.1 (security concepts) and 2.4 (ARP).
- **Reinforce Prior Content**: Link to DAI Lecture, DHCP Snooping, ARP (Day 18), DHCP (Day 13).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW2       |-------|      R1        |
| (DHCP Client)  | G0/0  | G0/1           | G0/0  | 192.168.1.1    |
+----------------+       |                |       | (DHCP Server)  |
|      PC2       |-------|                |       +----------------+
| (DHCP Client)  | G0/2  |                |
+----------------+       |                |
|      PC3       |-------|                |
| (DHCP Client)  | G0/3  |                |
+----------------+       |                |
                        | G0/4           |
                        |                |
                        +----------------+
                        |      SW1       |
                        +----------------+
```
- **Devices**:
  - **PC1–3**: DHCP clients on SW2 (G0/0, G0/2–3, untrusted), request IPs from R1.
  - **SW2**: Cisco switch, DHCP snooping and DAI enabled, G0/0, G0/2–3 untrusted (to PC1–3), G0/1 trusted (to R1), G0/4 untrusted (to SW1).
  - **SW1**: Cisco switch, DHCP snooping and DAI enabled, G0/4 untrusted (to SW2), G0/1–2 trusted (optional, per lab choice).
  - **R1**: DHCP server and default gateway, IP 192.168.1.1, pool 192.168.1.0/24 (excludes 192.168.1.1–9).
- **Note**:
  - **Untrusted Ports**: SW2’s G0/0, G0/2–3 (PC1–3), G0/4 (SW1); SW1’s G0/4 (SW2), inspected for ARP/DHCP messages.
  - **Trusted Ports**: SW2’s G0/1 (to R1), SW1’s G0/1–2 (optional), G0/4 trusted for DHCP/DAI.
  - **VLAN**: Single VLAN 1 (192.168.1.0/24).
  - **Dependencies**: DAI relies on DHCP snooping binding table.
  - **Packet Tracer Limitation**: Rate-limiting enabled on trusted ports (non-standard), no ARP ACLs or errdisable recovery supported.
- **Requirements**:
  1. Configure R1’s DHCP pool for 192.168.1.0/24.
  2. Enable DHCP snooping on SW1/SW2 for VLAN 1, trust SW2’s G0/1, SW1’s G0/2, disable Option 82.
  3. Enable DAI on SW1/SW2 for VLAN 1, trust SW2’s G0/1, SW1’s G0/1–2, enable all validations (src-mac, dst-mac, ip).
  4. Test PC1–3’s DHCP (`ipconfig /renew`) and ping to R1 (192.168.1.1).
  5. Verify with `show ip arp inspection`, `show ip dhcp snooping`.

**Analogy**: DAI and DHCP snooping are like a club’s security team (SW1/SW2) checking IDs (ARP/DHCP messages) at untrusted doors (G0/0, G0/2–4), trusting VIP doors (G0/1–2), verifying against a guest list (binding table), and ensuring no extra stamps (Option 82) or fake IDs (spoofed ARP).

**Lifelong Retention Tip**: Visualize PC1’s ARP request passing DAI checks and PC1 getting an IP after disabling Option 82. Create flashcards: “DAI? VLAN 1, trust G0/1. DHCP snooping? Trust G0/1, no Option 82.” Practice in Packet Tracer: configure, test pings, verify. Review 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure R1 as DHCP Server
**Objective**: Set up R1 as a DHCP server for 192.168.1.0/24, excluding 192.168.1.1–9.

**Steps (on R1)**:
1. Enter global configuration mode:
```plaintext
R1> enable
R1# configure terminal
```
2. Exclude addresses:
```plaintext
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.9
```
3. Configure DHCP pool:
```plaintext
R1(config)# ip dhcp pool POOL1
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
```

**Observations**:
- **DHCP Pool**: Assigns IPs from 192.168.1.10–254, default gateway 192.168.1.1.
- **Excluded Addresses**: 192.168.1.1 (R1), 192.168.1.2–9 (reserved).
- **Exam Note**: Topic 2.2 (DHCP) requires configuring `ip dhcp pool`, `network`, `default-router`.

**Comparison to Prior Content**:
- **DHCP Lecture (Day 13)**: Configures D-O-R-A (DISCOVER, OFFER, REQUEST, ACK).
- **DHCP Snooping Lecture**: Provides binding table for DAI.
- **Security Fundamentals Lecture**: Secures DHCP process (topic 5.1).
- **DAI Lecture**: DHCP enables ARP table population.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Uses 192.168.1.0/24 for DHCP.
- **Day 15 (TCP/UDP)**: DHCP uses UDP 67/68.
- **Day 18 (ARP)**: DHCP IPs used in ARP tables.

**Analogy**: R1’s DHCP pool is a ticket booth issuing valid tickets (IPs) to PC1–3, reserving some (192.168.1.1–9) for staff.

**Lifelong Retention Tip**: Configure DHCP pool in Packet Tracer, verify with `show ip dhcp pool`. Flashcard: “DHCP pool? `network`, `default-router`. Excluded? `ip dhcp excluded-address`.” Practice 3 times.

### Step 2: Configure DHCP Snooping on SW1
**Objective**: Enable DHCP snooping on SW1 for VLAN 1, trust G0/2 (to SW2/R1), disable Option 82, leave G0/1 untrusted.

**Steps (on SW1)**:
1. Enter global configuration mode:
```plaintext
SW1> enable
SW1# configure terminal
```
2. Enable DHCP snooping:
```plaintext
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
```
3. Disable Option 82:
```plaintext
SW1(config)# no ip dhcp snooping information option
```
4. Configure trusted port:
```plaintext
SW1(config)# interface g0/2
SW1(config-if)# ip dhcp snooping trust
```

**Observations**:
- **Global Enable**: `ip dhcp snooping` activates feature.
- **VLAN 1**: Enabled with `ip dhcp snooping vlan 1`.
- **Option 82**: Disabled to prevent SW1 adding relay info, which SW2 drops (G0/4 untrusted).
- **Trusted Port**: G0/2 (to SW2/R1) allows server messages (OFFER, ACK).
- **Untrusted G0/1**: More secure, inspects DHCP messages from SW2, but uses more CPU.
- **Exam Note**: Topic 5.7 requires `ip dhcp snooping`, `vlan`, `trust`, `no information option`.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Configures global, VLAN, trust, Option 82 (topic 5.7).
- **DAI Lecture**: DHCP snooping provides binding table for DAI.
- **Port Security Lecture**: Complements MAC limiting.
- **Security Fundamentals Lecture**: Mitigates DHCP attacks (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Protects DHCP (UDP 67/68).
- **Day 18 (ARP)**: Supports DAI’s ARP validation.

**Analogy**: SW1’s G0/2 is a trusted VIP door to R1, disabling Option 82 removes extra ticket stamps, while G0/1 checks tickets for safety.

**Lifelong Retention Tip**: Configure DHCP snooping in Packet Tracer, verify with `show ip dhcp snooping`. Flashcard: “DHCP snooping? Global, VLAN 1, trust G0/2, no Option 82.” Practice 3 times.

### Step 3: Configure DHCP Snooping on SW2
**Objective**: Enable DHCP snooping on SW2 for VLAN 1, trust G0/1 (to R1), disable Option 82.

**Steps (on SW2)**:
1. Enter global configuration mode:
```plaintext
SW2> enable
SW2# configure terminal
```
2. Enable DHCP snooping:
```plaintext
SW2(config)# ip dhcp snooping
SW2(config)# ip dhcp snooping vlan 1
```
3. Disable Option 82:
```plaintext
SW2(config)# no ip dhcp snooping information option
```
4. Configure trusted port:
```plaintext
SW2(config)# interface g0/1
SW2(config-if)# ip dhcp snooping trust
```

**Observations**:
- **Global/VLAN**: Enables snooping for VLAN 1.
- **Option 82**: Disabled to prevent SW2 adding relay info, which R1 drops (non-relay).
- **Trusted Port**: G0/1 (to R1) allows OFFER/ACK without inspection.
- **Untrusted G0/4**: Inspects SW1’s DHCP messages, aligns with SW1’s G0/1 untrusted choice.
- **Exam Note**: Topic 5.7 tests DHCP snooping on multiple switches.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Mirrors SW1 config (topic 5.7).
- **DAI Lecture**: Provides binding table for DAI.
- **Port Security Lecture**: Complements untrusted port filtering.
- **Security Fundamentals Lecture**: Secures DHCP (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Protects DHCP traffic.
- **Day 18 (ARP)**: Enables DAI validation.

**Analogy**: SW2’s G0/1 is a trusted door to R1’s ticket booth, disabling Option 82 ensures clean tickets for PC1–3.

**Lifelong Retention Tip**: Configure SW2 in Packet Tracer, verify trusted G0/1. Flashcard: “SW2? Trust G0/1, no Option 82. VLAN 1? Enable snooping.” Practice 3 times.

### Step 4: Configure DAI on SW2
**Objective**: Enable DAI on SW2 for VLAN 1, trust G0/1, enable all validation checks (src-mac, dst-mac, ip).

**Steps (on SW2)**:
1. Enable DAI:
```plaintext
SW2(config)# ip arp inspection vlan 1
```
2. Enable validations:
```plaintext
SW2(config)# ip arp inspection validate src-mac dst-mac ip
```
3. Configure trusted port:
```plaintext
SW2(config)# interface g0/1
SW2(config-if)# ip arp inspection trust
SW2(config-if)# exit
```
4. Verify:
```plaintext
SW2# show running-config
SW2# show ip arp inspection interfaces
```

**Observations**:
- **VLAN 1**: `ip arp inspection vlan 1` enables DAI, no global command (unlike DHCP snooping).
- **Validations**: Single command enables src-mac (Ethernet vs. ARP sender MAC), dst-mac (Ethernet vs. ARP target MAC, replies), ip (no invalid IPs like 0.0.0.0).
- **Trusted Port**: G0/1 (to R1) allows ARP messages without inspection.
- **Verification**:
  - `show running-config`: Confirms `ip arp inspection vlan 1`, `validate src-mac dst-mac ip`, G0/1 trust.
  - `show ip arp inspection interfaces`: G0/1 trusted, others untrusted, rate-limit 15 pps (Packet Tracer anomaly: trusted ports rate-limited, non-standard).
- **Packet Tracer Issue**: Rate-limiting enabled on G0/1 (trusted), contradicts Cisco documentation (disabled on trusted ports).
- **Exam Note**: Topic 5.7 requires `ip arp inspection`, `trust`, `validate`.

**Comparison to Prior Content**:
- **DAI Lecture**: Configures VLAN, trust, validations (topic 5.7).
- **DHCP Snooping Lecture**: Provides binding table for DAI.
- **ARP Lecture (Day 18)**: Validates ARP request/reply.
- **Security Fundamentals Lecture**: Mitigates ARP poisoning (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 ARP mappings.
- **Day 15 (TCP/UDP)**: Enables ARP for IP traffic.
- **Day 2 (Ethernet)**: Checks Ethernet frame MACs.

**Analogy**: SW2’s DAI is a gatekeeper checking ARP IDs (MAC/IP) at untrusted doors (G0/0, G0/2–4), trusting G0/1, and verifying extra ID fields (validations).

**Lifelong Retention Tip**: Configure DAI in Packet Tracer, verify with `show ip arp inspection interfaces`. Flashcard: “DAI? VLAN 1, trust G0/1, validate all. Rate-limit? 15 pps untrusted.” Practice 3 times.

### Step 5: Configure DAI on SW1
**Objective**: Enable DAI on SW1 for VLAN 1, trust G0/1–2, enable all validation checks.

**Steps (on SW1)**:
1. Enable DAI:
```plaintext
SW1(config)# ip arp inspection vlan 1
```
2. Enable validations:
```plaintext
SW1(config)# ip arp inspection validate src-mac dst-mac ip
```
3. Configure trusted ports:
```plaintext
SW1(config)# interface range g0/1 - 2
SW1(config-if-range)# ip arp inspection trust
SW1(config-if-range)# exit
```
4. Verify:
```plaintext
SW1# show running-config
SW1# show ip arp inspection interfaces
```

**Observations**:
- **VLAN 1**: Enables DAI for VLAN 1.
- **Validations**: Enables src-mac, dst-mac, ip checks in one command.
- **Trusted Ports**: G0/1–2 trusted (G0/2 to SW2/R1, G0/1 for flexibility), G0/4 untrusted (to SW2).
- **Verification**:
  - `show running-config`: Confirms DAI, DHCP snooping, G0/2 trusted for both, G0/1 trusted for DAI.
  - `show ip arp inspection interfaces`: G0/1–2 trusted, rate-limit 15 pps (Packet Tracer anomaly).
- **Design Choice**: G0/1 trusted for DAI, untrusted for DHCP (more secure, higher CPU).
- **Exam Note**: Topic 5.7 tests DAI configuration on multiple switches.

**Comparison to Prior Content**:
- **DAI Lecture**: Mirrors SW2 config (topic 5.7).
- **DHCP Snooping Lecture**: Binding table supports DAI.
- **ARP Lecture (Day 18)**: Validates ARP messages.
- **Security Fundamentals Lecture**: Prevents ARP poisoning (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures ARP mappings.
- **Day 15 (TCP/UDP)**: Supports ARP for IP.
- **Day 2 (Ethernet)**: Validates Ethernet MACs.

**Analogy**: SW1’s DAI checks ARP IDs at untrusted G0/4, trusts G0/1–2, and verifies extra fields for valid tickets.

**Lifelong Retention Tip**: Configure SW1 in Packet Tracer, verify trusted G0/1–2. Flashcard: “SW1? Trust G0/1–2, validate all. G0/4? Untrusted.” Practice 3 times.

### Step 6: Test DHCP and ARP Functionality
**Objective**: Verify PC1–3 get IPs via DHCP and can ping R1 (192.168.1.1).

**Steps**:
1. On PC1–3, set to DHCP:
   - Packet Tracer GUI: Desktop > IP Configuration > DHCP (changes FastEthernet0 to DHCP).
2. On PC1 CLI, check IP:
```plaintext
C:\> ipconfig
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
```
3. Ping R1:
```plaintext
C:\> ping 192.168.1.1
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
```
4. Verify DHCP bindings on SW2:
```plaintext
SW2# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               192.168.1.10     86400       dhcp-snooping  1     GigabitEthernet0/0
B.B.B               192.168.1.11     86400       dhcp-snooping  1     GigabitEthernet0/2
C.C.C               192.168.1.12     86400       dhcp-snooping  1     GigabitEthernet0/3
```
5. Verify DAI on SW2:
```plaintext
SW2# show ip arp inspection interfaces
Interface        Trust State  Rate Limit (pps)  Burst Interval
---------------  -----------  ----------------  --------------
Gi0/0            Untrusted             15               1
Gi0/1            Trusted               15               1
Gi0/2            Untrusted             15               1
Gi0/3            Untrusted             15               1
Gi0/4            Untrusted             15               1
```

**Observations**:
- **DHCP Success**: PC1–3 get IPs (192.168.1.10–12) via DHCP, confirming Option 82 disabled.
- **Ping Success**: PC1 pings R1, indicating valid ARP (A.A.A, 192.168.1.10 checked against binding table).
- **Binding Table**: Logs PC1–3’s MACs/IPs, supports DAI validation.
- **DAI Issue**: Packet Tracer shows G0/1 (trusted) rate-limited at 15 pps, non-standard (should be disabled per Cisco).
- **Exam Note**: Topic 5.7 tests DHCP/DAI functionality, ping verifies ARP.

**Comparison to Prior Content**:
- **DAI Lecture**: Validates ARP for ping (topic 5.7).
- **DHCP Snooping Lecture**: Binding table enables DAI.
- **ARP Lecture (Day 18)**: Ping requires ARP request/reply.
- **Security Fundamentals Lecture**: Ensures valid ARP/DHCP (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Assigns 192.168.1.10–12.
- **Day 15 (TCP/UDP)**: Enables DHCP (UDP 67/68), ping (ICMP).
- **Day 2 (Ethernet)**: Validates ARP Ethernet frames.

**Analogy**: PC1–3’s tickets (DHCP IPs, ARP requests) pass security checks (DAI, DHCP snooping), allowing entry (ping) to R1’s VIP area.

**Lifelong Retention Tip**: Test DHCP and ping in Packet Tracer, verify bindings. Flashcard: “Ping works? DHCP IP, DAI ARP pass. Binding table? Logs MAC/IP.” Practice 3 times.

## Common Issues and Troubleshooting

- **DHCP Fails**:
  - **Issue**: PC1–3 fail to get IPs.
  - **Cause**: Option 82 enabled, dropped by SW1 (G0/4 untrusted) or R1 (non-relay).
  - **Fix**: `no ip dhcp snooping information option` on SW1/SW2.
  - **Verify**: `show ip dhcp snooping`, `show ip dhcp snooping binding`.
- **Ping Fails**:
  - **Issue**: PC1 cannot ping R1 (192.168.1.1).
  - **Cause**: DAI drops ARP (no binding table entry, G0/1–3 trusted, or validations misconfigured).
  - **Fix**: Ensure `ip dhcp snooping vlan 1`, G0/0–3 untrusted, `ip arp inspection validate src-mac dst-mac ip`.
  - **Verify**: `show ip arp inspection`, `show ip dhcp snooping binding`.
- **Rate-Limit on Trusted Ports**:
  - **Issue**: G0/1 (SW2) rate-limited at 15 pps.
  - **Cause**: Packet Tracer bug (Cisco: trusted ports no rate-limit).
  - **Fix**: Not configurable in Packet Tracer; note for CCNA (disabled on trusted ports).
  - **Verify**: `show ip arp inspection interfaces`.
- **Validation Overwritten**:
  - **Issue**: Only `validate src-mac` active.
  - **Cause**: Multiple `ip arp inspection validate` commands.
  - **Fix**: `ip arp inspection validate src-mac dst-mac ip` (single command).
  - **Verify**: `show running-config | include validate`.
- **Syslog Missing**:
  - **Issue**: No `%SW_DAI-4-DHCP_SNOOPING_DENY`.
  - **Cause**: Packet Tracer limitation.
  - **Fix**: Use real devices with `logging console`.
  - **Verify**: Check Syslog in real devices.

**Analogy**: Troubleshooting is like fixing a club’s security: ensure clean tickets (no Option 82), valid guest list (binding table), correct VIP doors (trust G0/1), and proper ID checks (validations).

**Lifelong Retention Tip**: Simulate ping failure in Packet Tracer, fix with correct trust/validations. Checklist: “No ping? Check binding, trust, validations. No IP? Option 82.” Practice 3 times.

## Additional CCNA-Relevant Information

**Why This Matters for CCNA**:
- **Exam Topic 5.7**: Configures/verifies DAI (`ip arp inspection`, `trust`, `validate`) and DHCP snooping.
- **Exam Topic 5.1**: Mitigates ARP poisoning (Confidentiality).
- **Exam Topic 2.4**: Secures ARP process for ping.
- **Exam Topic 2.2**: Secures DHCP for binding table.
- **Packet Tracer Limitations**: No ARP ACLs, errdisable recovery, or rate-limit disabling on trusted ports; sufficient for topic 5.7 basics.

**Key CCNA Concepts Reinforced**:
- **DHCP Snooping**:
  - Enables binding table (MAC, IP, interface, VLAN) for DAI.
  - Trusts G0/1 (SW2), G0/2 (SW1), disables Option 82.
- **DAI**:
  - Filters ARP on untrusted ports (G0/0, G0/2–4), trusts G0/1–2.
  - Validates sender MAC/IP against binding table, optional checks (src-mac, dst-mac, ip).
- **DHCP Server**: Assigns 192.168.1.10–12, supports ARP table population.
- **Ping**: Verifies DHCP (IP) and DAI (ARP) functionality.

**Additional Configurations (Not in Lab, Packet Tracer Unsupported)**:
- Rate-Limiting:
```plaintext
SW2(config)# interface g0/0
SW2(config-if)# ip arp inspection limit rate 25 burst-interval 2
```
- ErrDisable Recovery:
```plaintext
SW2(config)# errdisable recovery cause arp-inspection
```
- ARP ACL (for static IPs):
```plaintext
SW2(config)# arp access-list ARP-ACL-1
SW2(config-arp-acl)# permit ip host 192.168.1.100 mac host C.C.C
SW2(config)# ip arp inspection filter ARP-ACL-1 vlan 1
```

**Practice in Packet Tracer**:
1. Replicate lab: Configure R1’s DHCP, SW1/SW2’s DHCP snooping/DAI, trust G0/1–2, validations.
2. Test PC1–3’s DHCP (`ipconfig /renew`) and ping (192.168.1.1).
3. Verify with `show ip dhcp snooping binding`, `show ip arp inspection interfaces`.
4. Experiment: Misconfigure trust (G0/0 trusted), observe ping failure.

**Additional Resources**:
- **Cisco Documentation**: Review DAI/DHCP snooping (topic 5.7) in CCNA study guide.
- **Boson ExSim**: Practice DAI questions (trust, validations, binding table).
- **Cloudflare Articles**: Search “Dynamic ARP Inspection Cisco” for real-world context.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Config Steps: “DHCP: GVTN (Global, VLAN, Trust, No Option 82). DAI: VTV (VLAN, Trust, Validate).”
   - ARP Types: “Request, Reply, GARP” (RRG).
2. **Visualize Topology**:
   - Draw PC1–3 → SW2 (G0/0,2–3 untrusted, G0/1 trusted) → R1, SW1 on G0/4. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DAI? VLAN 1, trust G0/1, validate all. DHCP? Trust G0/1, no Option 82. Binding? MAC/IP.”
4. **Practice in Packet Tracer**:
   - Configure DHCP/DAI, test ping, verify bindings. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: DHCP Snooping (binding table) vs. DAI (ARP checks).
   - Quiz: “Ping fails? Check DAI trust, validations. No IP? Option 82.”
6. **Troubleshooting Practice**:
   - Simulate ping failure (wrong trust), fix validations. Checklist: “No ping? Binding, trust, validations.”
7. **Teach Someone**:
   - Explain DAI, DHCP snooping, and ping success to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`ip arp inspection`, `validate`), binding table. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DHCP Server (R1)**:
   - Pool: 192.168.1.10–254, gateway 192.168.1.1, excludes 192.168.1.1–9.
2. **DHCP Snooping (SW1/SW2)**:
   - Enabled: Global, VLAN 1, trust G0/1 (SW2), G0/2 (SW1), no Option 82.
   - Binding table: Logs PC1–3 (e.g., A.A.A, 192.168.1.10, G0/0).
3. **DAI (SW1/SW2)**:
   - Enabled: VLAN 1, trust G0/1 (SW2), G0/1–2 (SW1).
   - Validations: src-mac, dst-mac, ip (single command).
4. **Testing**:
   - PC1–3 get IPs (192.168.1.10–12), ping R1 (ARP validated).
5. **Verification**:
   - `show ip dhcp snooping binding`: Confirms PC1–3’s leases.
   - `show ip arp inspection interfaces`: Confirms G0/1–2 trusted, 15 pps rate-limit (Packet Tracer bug).
6. **CCNA Focus**:
   - Topic 5.7: Configure/verify DAI (`ip arp inspection`, `trust`, `validate`), DHCP snooping.
   - Topic 5.1: Mitigates ARP poisoning (Confidentiality).
   - Topic 2.4: Secures ARP for ping.
   - Topic 2.2: Secures DHCP for bindings.
7. **Packet Tracer**: Limited to basic DAI (no ARP ACLs, errdisable), sufficient for topic 5.7.

These notes provide a comprehensive foundation for the DAI Lab (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/validations, and apply analogies/retention tips to master concepts for life!