# DHCP Snooping Lab Notes for CCNA Beginners

## Lab Overview

This lab configures **DHCP snooping** on Cisco switches SW1 and SW2 in **Packet Tracer**, reinforcing **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **DHCP Snooping Lecture**, **Port Security Lecture**, **Security Fundamentals Lecture** (topic 5.1), **DHCP Lecture** (Day 13, topic 2.2), and **Kali Linux Demo Lab** (DHCP exhaustion attack). The lab sets up R1 as a DHCP server, enables DHCP snooping on SW1 and SW2 for VLAN 1, configures trusted ports, and addresses Option 82 issues to ensure PC1 obtains an IP address.

**Lab Objectives**:
- **Configure R1 as DHCP Server**: Set up a DHCP pool for 192.168.1.0/24, excluding 192.168.1.1–9.
- **Configure DHCP Snooping on SW1/SW2**: Enable globally, for VLAN 1, trust uplink ports, disable Option 82.
- **Test DHCP Functionality**: Verify PC1 gets an IP via DHCP, troubleshoot Option 82 issues.
- **Verify Configurations**: Use `show` commands to confirm DHCP snooping settings and bindings.
- **Prepare for CCNA**: Master topic 5.7 (DHCP snooping configuration/verification) and connect to topics 5.1 (security concepts) and 2.2 (DHCP).
- **Reinforce Prior Content**: Link to DHCP (Day 13), Port Security, Kali Linux Demo Lab (Day 48), DHCP Snooping Lecture.

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW2       |-------|      R1        |
| (DHCP Client)  | G0/0  | G0/1           | G0/0  | 192.168.1.1    |
|                |       |                |       | (DHCP Server)  |
+----------------+       +----------------+       +----------------+
                        |
                        | G0/2
                        |
                        +----------------+
                        |      SW1       |
                        |                |
                        +----------------+
```
- **Devices**:
  - **PC1**: DHCP client on SW2 (G0/0, untrusted), requests IP from R1.
  - **SW2**: Cisco switch, DHCP snooping enabled, G0/0 untrusted (to PC1), G0/1 trusted (to R1), G0/2 to SW1.
  - **SW1**: Cisco switch, DHCP snooping enabled, G0/2 untrusted (to SW2).
  - **R1**: DHCP server, IP 192.168.1.1, pool 192.168.1.0/24 (excludes 192.168.1.1–9).
- **Note**:
  - **Untrusted Ports**: SW2’s G0/0 (PC1), G0/2 (SW1); SW1’s G0/2 (SW2), inspected for DHCP messages.
  - **Trusted Ports**: SW2’s G0/1 (to R1), no inspection.
  - **VLAN**: Single VLAN 1 (192.168.1.0/24).
  - **Option 82 Issue**: Switches add Option 82 by default, dropped by untrusted ports or R1 (non-relay).
- **Requirements**:
  1. Configure R1’s DHCP pool for 192.168.1.0/24.
  2. Enable DHCP snooping on SW1/SW2 for VLAN 1, trust SW2’s G0/1.
  3. Disable Option 82 to allow PC1’s DHCP requests.
  4. Test PC1’s IP assignment (`ipconfig /renew`).
  5. Verify with `show ip dhcp snooping` and `show ip dhcp snooping binding`.

**Analogy**: DHCP snooping is like a club’s ticket checker (SW1/SW2) ensuring only valid tickets (DHCP messages) from trusted doors (G0/1) pass, while checking client IDs (MACs) at untrusted doors (G0/0, G0/2) and removing extra stamps (Option 82) that cause issues.

**Lifelong Retention Tip**: Visualize SW2 dropping PC1’s DISCOVER due to Option 82, then succeeding after disabling it. Create flashcards: “DHCP snooping? Global, VLAN, trust uplinks. Option 82? Disable for non-relay.” Practice in Packet Tracer: configure, test, troubleshoot. Review 3 times daily for a week.

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
- **DHCP Lecture (Day 13)**: Configures D-O-R-A (DISCOVER, OFFER, REQUEST, ACK), pool setup.
- **Security Fundamentals Lecture**: Prepares for DHCP snooping to secure DHCP (topic 5.1).
- **Kali Linux Demo Lab**: DHCP pool is target for exhaustion attacks.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Uses 192.168.1.0/24 for DHCP assignments.
- **Day 15 (TCP/UDP)**: DHCP uses UDP 67/68.
- **Day 18 (ARP)**: DHCP IPs used in ARP tables.

**Analogy**: R1’s DHCP pool is like a ticket booth issuing valid tickets (IPs) to clients, reserving some (192.168.1.1–9) for staff.

**Lifelong Retention Tip**: Configure DHCP pool in Packet Tracer, verify with `show ip dhcp pool`. Flashcard: “DHCP pool? `network`, `default-router`. Excluded? `ip dhcp excluded-address`.” Practice 3 times.

### Step 2: Configure DHCP Snooping on SW1
**Objective**: Enable DHCP snooping on SW1 for VLAN 1, trust G0/2 (to R1 via SW2), disable Option 82.

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
- **VLAN 1**: Only VLAN in use, enabled with `ip dhcp snooping vlan 1`.
- **Option 82**: Disabled to prevent SW1 adding relay info, which SW2 drops (untrusted G0/2).
- **Trusted Port**: G0/2 (to SW2/R1) allows server messages (OFFER, ACK) without inspection.
- **Exam Note**: Topic 5.7 requires `ip dhcp snooping`, `vlan`, `trust`, and handling Option 82.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Configures global, VLAN, trust, Option 82 (topic 5.7).
- **Port Security Lecture**: Complements MAC limiting for DHCP security.
- **Security Fundamentals Lecture**: Mitigates DHCP attacks (topic 5.1).
- **DHCP Lecture (Day 13)**: Protects D-O-R-A process.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Protects DHCP (UDP 67/68).
- **Day 18 (ARP)**: Prepares for dynamic ARP inspection.

**Analogy**: SW1’s G0/2 is a trusted VIP door to R1, while disabling Option 82 removes extra ticket stamps that cause rejections.

**Lifelong Retention Tip**: Configure DHCP snooping in Packet Tracer, verify with `show ip dhcp snooping`. Flashcard: “DHCP snooping? Global, VLAN 1, trust G0/2. Option 82? Disable.” Practice 3 times.

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
- **Global/VLAN**: Same as SW1, enables snooping for VLAN 1.
- **Option 82**: Disabled to prevent SW2 adding relay info, which R1 drops (non-relay).
- **Trusted Port**: G0/1 (to R1) allows OFFER/ACK without inspection.
- **Exam Note**: Topic 5.7 tests identical configuration on multiple switches.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Mirrors SW1 config (global, VLAN, trust, Option 82).
- **Port Security Lecture**: Complements untrusted port filtering.
- **Security Fundamentals Lecture**: Secures DHCP process (topic 5.1).
- **Kali Linux Demo Lab**: Prevents spoofed DHCP messages.

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Protects DHCP traffic.
- **Day 18 (ARP)**: Supports ARP security.

**Analogy**: SW2’s G0/1 is a trusted door to R1’s ticket booth, while disabling Option 82 ensures clean tickets for PC1.

**Lifelong Retention Tip**: Configure SW2 in Packet Tracer, verify trusted G0/1. Flashcard: “SW2? Trust G0/1, no Option 82. VLAN 1? Enable snooping.” Practice 3 times.

### Step 4: Test DHCP Functionality (Initial Failure)
**Objective**: Test PC1’s DHCP request, observe Option 82 issue.

**Steps (on PC1)**:
1. Request IP:
```plaintext
C:\> ipconfig /renew
```
2. **Result**: Fails (no IP assigned).
3. Switch to Packet Tracer simulation mode:
   - PC1 sends DISCOVER.
   - SW2 (G0/0, untrusted) adds Option 82.
   - SW1 (G0/2, untrusted) drops DISCOVER (Option 82 on untrusted port).
4. Check SW1 Syslog:
```plaintext
%DHCP_SNOOPING-5-DHCP_SNOOPING_UNTRUSTED_PORT: DHCP_SNOOPING drop message with option82 value on untrusted port
```

**Observations**:
- **Failure Cause**: SW2 adds Option 82 (relay info) to PC1’s DISCOVER, SW1 drops it (G0/2 untrusted).
- **Syslog**: Confirms drop due to Option 82.
- **Exam Note**: Topic 5.7 tests Option 82 troubleshooting.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Explains Option 82 drops on untrusted ports.
- **DHCP Lecture (Day 13)**: DISCOVER part of D-O-R-A.
- **Security Fundamentals Lecture**: Mitigates invalid DHCP messages (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: No IP assigned to PC1.
- **Day 15 (TCP/UDP)**: Affects DHCP (UDP 67).
- **Day 18 (ARP)**: Prevents IP for ARP table.

**Analogy**: PC1’s ticket (DISCOVER) gets an extra stamp (Option 82) from SW2, rejected by SW1’s guard (G0/2).

**Lifelong Retention Tip**: Simulate Option 82 drop in Packet Tracer, check Syslog. Flashcard: “Option 82? Dropped on untrusted. Fix? `no ip dhcp snooping information option`.” Practice 3 times.

### Step 5: Fix Option 82 and Retest
**Objective**: Disable Option 82 on SW1/SW2, retest PC1’s DHCP request.

**Steps**:
1. **Already Done in Steps 2–3**:
   - SW1: `no ip dhcp snooping information option`.
   - SW2: `no ip dhcp snooping information option`.
2. On PC1, retry DHCP:
```plaintext
C:\> ipconfig /renew
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
```
3. Verify on SW2:
```plaintext
SW2# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               192.168.1.10     86400       dhcp-snooping  1     GigabitEthernet0/0
```
4. Verify DHCP snooping:
```plaintext
SW2# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs: 1
DHCP snooping is operational on following VLANs: 1
DHCP snooping is configured on the following Interfaces:
GigabitEthernet0/1 (trust)
Insertion of option 82 is disabled
```

**Observations**:
- **Success**: PC1 gets IP 192.168.1.10, gateway 192.168.1.1.
- **Option 82**: Disabled, no drops by SW1 or R1.
- **Binding Table**: Logs PC1’s MAC A.A.A, IP 192.168.1.10, G0/0, VLAN 1.
- **Trusted Port**: G0/1 allows R1’s OFFER/ACK.
- **Exam Note**: Topic 5.7 requires fixing Option 82, verifying bindings.

**Comparison to Prior Content**:
- **DHCP Snooping Lecture**: Disabling Option 82 resolves untrusted port drops.
- **DHCP Lecture (Day 13)**: Completes D-O-R-A for PC1.
- **Port Security Lecture**: Complements MAC checks.
- **Security Fundamentals Lecture**: Ensures valid DHCP (topic 5.1).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Assigns 192.168.1.10.
- **Day 15 (TCP/UDP)**: Enables DHCP (UDP 67/68).
- **Day 18 (ARP)**: IP enables ARP table entry.

**Analogy**: Disabling Option 82 removes the extra stamp, letting PC1’s ticket (DISCOVER) pass to R1’s booth for a valid IP.

**Lifelong Retention Tip**: Simulate successful DHCP after disabling Option 82. Flashcard: “PC1 IP? Disable Option 82, check bindings. Trusted? G0/1.” Practice 3 times.

## Common Issues and Troubleshooting

- **DHCP Fails (Option 82)**:
  - **Issue**: PC1 fails to get IP (`ipconfig /renew`).
  - **Cause**: SW2 adds Option 82, SW1 drops (G0/2 untrusted).
  - **Fix**: `no ip dhcp snooping information option` on SW1/SW2.
  - **Verify**: `show ip dhcp snooping`, `show ip dhcp snooping binding`.
- **No Binding Table Entries**:
  - **Issue**: Empty `show ip dhcp snooping binding`.
  - **Cause**: DHCP snooping not enabled for VLAN 1 or G0/1 not trusted.
  - **Fix**: `ip dhcp snooping vlan 1`, `ip dhcp snooping trust` on G0/1.
  - **Verify**: `show ip dhcp snooping binding`.
- **Server Messages Forwarded**:
  - **Issue**: Spurious DHCP server messages pass.
  - **Cause**: G0/0 or G0/2 trusted (misconfigured).
  - **Fix**: Remove `ip dhcp snooping trust` from non-uplink ports.
  - **Verify**: `show ip dhcp snooping`.
- **Syslog Not Seen**:
  - **Issue**: No `%DHCP_SNOOPING-5-DHCP_SNOOPING_UNTRUSTED_PORT`.
  - **Cause**: Packet Tracer limitation or logging disabled.
  - **Fix**: Ensure `logging console` (real devices).
  - **Verify**: Check Syslog in real devices.

**Analogy**: Troubleshooting is like fixing a ticket checker: remove extra stamps (Option 82), ensure VIP doors (G0/1 trusted), and check ticket logs (bindings).

**Lifelong Retention Tip**: Simulate Option 82 failure in Packet Tracer, fix with `no ip dhcp snooping information option`. Checklist: “No IP? Check Option 82, trust, VLAN.” Practice 3 times.

## Additional CCNA-Relevant Information

**Why This Matters for CCNA**:
- **Exam Topic 5.7**: Configures/verifies DHCP snooping (`ip dhcp snooping`, `vlan`, `trust`, `no information option`).
- **Exam Topic 5.1**: Mitigates DHCP starvation/poisoning (Confidentiality, Availability).
- **Exam Topic 2.2**: Secures DHCP D-O-R-A process.
- **Packet Tracer Limitations**: Limited Syslog, no rate-limiting/errdisable recovery tested, but sufficient for topic 5.7.

**Key CCNA Concepts Reinforced**:
- **DHCP Snooping**:
  - Filters untrusted ports (G0/0, G0/2), trusts uplinks (G0/1).
  - Discards server messages (OFFER, ACK, NAK) on untrusted ports.
  - Inspects client messages (DISCOVER, REQUEST: MAC vs. CHADDR).
- **Option 82**: Disabled for non-relay networks to prevent drops.
- **Binding Table**: Logs valid DHCP leases (MAC, IP, interface, VLAN).
- **DHCP Server**: Configured with `ip dhcp pool`, `excluded-address`.

**Additional Configurations (Not in Lab)**:
- Rate-Limiting:
```plaintext
SW2(config)# interface g0/0
SW2(config-if)# ip dhcp snooping limit rate 10
```
- ErrDisable Recovery:
```plaintext
SW2(config)# errdisable recovery cause dhcp-rate-limit
```
- Static Binding (Rare):
```plaintext
SW2(config)# ip source binding A.A.A vlan 1 192.168.1.10 interface g0/0
```

**Practice in Packet Tracer**:
1. Replicate lab: Configure R1’s DHCP pool, SW1/SW2 snooping, trust G0/1, disable Option 82.
2. Test PC1’s `ipconfig /renew` before/after Option 82 fix.
3. Verify bindings with `show ip dhcp snooping binding`.
4. Experiment: Add rate-limiting, simulate DISCOVER flood (if supported).

**Additional Resources**:
- **Cisco Documentation**: Review DHCP snooping (topic 5.7) in CCNA study guide.
- **Boson ExSim**: Practice questions on Option 82, trusted ports, bindings.
- **Cloudflare Articles**: Search “DHCP snooping Cisco” for real-world context.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Config Steps: “Global, VLAN, Trust, No Option 82” (GVTN).
   - Messages: “Server: OAN (OFFER, ACK, NAK). Client: DRRD (DISCOVER, REQUEST, RELEASE, DECLINE).”
2. **Visualize Topology**:
   - Draw PC1 → SW2 (G0/0 untrusted, G0/1 trusted) → R1, SW1 on G0/2. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DHCP snooping? Global, VLAN, trust G0/1. Option 82? Disable, else dropped. Binding? MAC, IP, interface.”
4. **Practice in Packet Tracer**:
   - Configure snooping, test Option 82 failure/success. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Option 82 enabled (drops) vs. disabled (works).
   - Quiz: “No IP? Option 82 drop. Trusted? G0/1. Binding? Logs valid leases.”
6. **Troubleshooting Practice**:
   - Simulate Option 82 drop, fix with `no ip dhcp snooping information option`. Checklist: “No IP? Check Option 82, trust.”
7. **Teach Someone**:
   - Explain DHCP snooping setup and Option 82 to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`ip dhcp snooping`), Option 82, bindings. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DHCP Server (R1)**:
   - Pool: 192.168.1.10–254, gateway 192.168.1.1, excludes 192.168.1.1–9.
2. **DHCP Snooping (SW1/SW2)**:
   - Enabled: Global, VLAN 1, trust G0/1 (SW2), G0/2 (SW1).
   - Option 82: Disabled to prevent drops by SW1 (untrusted) or R1 (non-relay).
3. **Testing**:
   - PC1 fails `ipconfig /renew` with Option 82 enabled (dropped by SW1).
   - Succeeds after disabling Option 82 (IP 192.168.1.10).
4. **Verification**:
   - `show ip dhcp snooping`: Confirms VLAN 1, G0/1 trusted, Option 82 disabled.
   - `show ip dhcp snooping binding`: Logs PC1’s lease (MAC A.A.A, 192.168.1.10, G0/0).
5. **CCNA Focus**:
   - Topic 5.7: Configure/verify DHCP snooping (`ip dhcp snooping`, `trust`, `no information option`).
   - Topic 5.1: Mitigates DHCP attacks (Confidentiality, Availability).
   - Topic 2.2: Secures DHCP D-O-R-A.
6. **Packet Tracer**: Limited Syslog, no rate-limiting tested, but sufficient for topic 5.7.

These notes provide a comprehensive foundation for the DHCP Snooping Lab (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/Option 82, and apply analogies/retention tips to master concepts for life!