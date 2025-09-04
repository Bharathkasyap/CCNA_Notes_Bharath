# Port Security Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **port security**, a Cisco switch feature for controlling source MAC addresses and their quantity on switch ports, as outlined in **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **Security Fundamentals** lecture (topic 5.1) and the **Kali Linux Demo Lab** (DHCP exhaustion attack), focusing on preventing unauthorized access and attacks like DHCP starvation. The lecture explains port security’s purpose, configuration, violation modes, secure MAC address aging, sticky MACs, and recovery methods, preparing you for configuration and verification tasks in the CCNA exam.

**Lecture Objectives**:
- **Understand Port Security**: Define port security and its role in controlling MAC addresses.
- **Explain Benefits**: Prevent unauthorized devices and mitigate attacks (e.g., DHCP exhaustion).
- **Configure Port Security**: Enable, set violation modes, configure MACs (static, dynamic, sticky), and adjust aging.
- **Verify and Troubleshoot**: Use `show` commands and recover from violations (manual, ErrDisable recovery).
- **Prepare for CCNA**: Master topic 5.7 (port security configuration) and connect to topics 5.1 (security concepts) and 2.2 (DHCP).
- **Reinforce Prior Content**: Link to DHCP (Day 13), DTP (Day 22), and Kali Linux Demo Lab (Day 48).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      R1        |
| MAC: A.A.A     | G0/1  | G0/0           | G0/0  | 192.168.1.1    |
| (Authorized)   |       |                |       | (Router)       |
+----------------+       +----------------+       +----------------+
                       |
                       | G0/2
                       |
                       +----------------+
                       |      PC2       |
                       | MAC: B.B.B     |
                       | (Unauthorized) |
                       +----------------+
```
- **Devices**:
  - **PC1**: Authorized device, MAC A.A.A, connected to SW1’s G0/1.
  - **SW1**: Cisco switch, port security on G0/1 (allows A.A.A), G0/0 to R1, G0/2 to PC2.
  - **R1**: Router, IP 192.168.1.1, destination for PC1/PC2 pings.
  - **PC2**: Unauthorized device, MAC B.B.B, attempts access via G0/1 or G0/2.
- **Note**: Port security on G0/1 allows only A.A.A (default: 1 MAC). PC2’s B.B.B triggers violation (e.g., err-disabled). Topology also applies to VoIP scenarios (phone + PC).
- **Requirements**:
  1. Configure port security on SW1’s G0/1 to allow A.A.A, max 1 MAC, shutdown mode.
  2. Test PC2’s violation, verify with `show port-security`.
  3. Recover interface (manual or ErrDisable).
  4. Configure for VoIP (max 2 MACs), sticky MACs, and aging.
  5. Mitigate DHCP exhaustion (limit MACs on G0/2).

**Analogy**: Port security is like a bouncer at a club (SW1’s G0/1): checks IDs (MAC A.A.A), allows only approved guests (PC1), and blocks or reports fakes (PC2’s B.B.B). Violations may lock the door (shutdown), ignore fakes (protect), or log them (restrict).

**Lifelong Retention Tip**: Visualize SW1 as a gatekeeper rejecting PC2’s B.B.B. Create flashcards: “Port security? MAC control. Shutdown? Err-disabled. Sticky? Auto-config MACs.” Practice in Packet Tracer: configure port security, test violations. Review 3 times daily for a week.

## Port Security Fundamentals

**Purpose**: Define port security and its role in network security (topic 5.7).

**Key Points**:
- **Definition**: Cisco switch feature to control source MAC addresses and their quantity per port.
- **Configuration Scope**: Per-interface (e.g., G0/1), not global.
- **Default Settings**:
  - Max 1 MAC address.
  - First MAC dynamically learned (e.g., A.A.A from PC1).
  - Violation mode: shutdown (err-disabled state).
- **Operation**:
  - SW1 checks incoming frame’s source MAC on G0/1.
  - Allowed (A.A.A): Forwards frame.
  - Unauthorized (B.B.B): Triggers violation (e.g., err-disabled).
- **Exam Note**: Topic 5.7 requires configuration and verification, not just theory.

**Demonstration**:
- **Topology**: PC1 (A.A.A) → SW1 (G0/1) → R1, PC2 (B.B.B) attempts access.
- **Scenario**:
  - G0/1 allows A.A.A (default max 1). PC1 pings R1 (192.168.1.1), works.
  - PC2 replaces PC1 on G0/1, pings R1. SW1 detects B.B.B, err-disables G0/1.
  - PC1 reconnects, fails (G0/1 err-disabled).
- **Commands**:
```plaintext
SW1# configure terminal
SW1(config)# interface g0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
```
- **Verification**:
```plaintext
SW1# show port-security interface g0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 1
Total MAC Addresses         : 0
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : 0000.0000.0000:0
Security Violation Count    : 0
```
- Post-PC1 ping:
```plaintext
Total MAC Addresses         : 1
Last Source Address:Vlan    : A.A.A:1
```
- Post-PC2 violation:
```plaintext
Port Status                : Secure-shutdown
Total MAC Addresses         : 0
Last Source Address:Vlan    : B.B.B:1
Security Violation Count    : 1
```

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: Port security mitigates spoofing/DoS (e.g., DHCP exhaustion, topic 5.1).
- **Kali Linux Demo Lab**: Limits MACs to prevent DHCP starvation (spoofed MACs).
- **Day 22 (DTP)**: Port security requires static `switchport mode access` (not dynamic auto/desirable).
- **Syslog Lab**: Violations trigger Syslog (e.g., `%PORT_SECURITY-2-PSECURE_VIOLATION`).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Mitigates ARP spoofing by limiting MACs.
- **Day 13 (DHCP)**: Prevents DHCP exhaustion (Kali Lab).
- **Day 10 (IP Addressing)**: Secures access to 192.168.1.0/24 network.

**Analogy**: G0/1 is a locked door allowing only A.A.A’s key (MAC). B.B.B’s fake key triggers an alarm (violation).

**Lifelong Retention Tip**: Simulate in Packet Tracer: enable port security, test PC2 violation. Flashcard: “Port security? MAC limit, shutdown default. Err-disabled? Violation.” Practice 3 times.

## Why Use Port Security?

**Purpose**: Explain security benefits, including preventing unauthorized access and mitigating attacks (topics 5.1, 5.7).

**Key Points**:
- **Prevent Unauthorized Access**:
  - Restricts devices to authorized MACs (e.g., A.A.A on G0/1).
  - Example: PC2 (B.B.B) cannot access network via G0/1.
- **Mitigate Attacks**:
  - **DHCP Exhaustion** (Kali Lab): Limits MACs to prevent spoofed DHCP Discover messages.
  - **MAC Flooding**: Prevents switch MAC table overflow (flooding all packets).
- **Limitations**:
  - MAC spoofing is easy (e.g., Attacker fakes A.A.A).
  - Solution: Combine with DHCP snooping, dynamic ARP inspection (topic 5.7, next lectures).
- **Exam Note**: Port security protects Availability (DHCP exhaustion) and Confidentiality (unauthorized access).

**Demonstration**:
- **Topology**: PC1 (A.A.A) → SW1 (G0/1), PC2 (B.B.B) attempts G0/1.
- **Scenario**:
  - G0/1 allows 1 MAC (A.A.A). PC2 connects, triggers violation.
  - DHCP exhaustion scenario: Attacker on G0/2 sends spoofed MACs; max 1 MAC blocks attack.
- **Verification**:
  - Post-PC2: `show port-security interface g0/1` shows secure-shutdown, violation count 1.
  - DHCP attack: Max 1 MAC on G0/2 limits spoofed MACs.

**Comparison to Prior Content**:
- **Kali Linux Demo Lab**: Port security on G0/2 stops Attacker’s spoofed MACs.
- **Security Fundamentals Lecture**: Mitigates DoS (Availability), spoofing (Confidentiality).
- **Day 13 (DHCP)**: Protects R1’s DHCP pool.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Complements dynamic ARP inspection.
- **Day 15 (TCP/UDP)**: Secures DHCP (UDP 67/68).
- **Day 10 (IP Addressing)**: Protects 192.168.1.0/24 access.

**Analogy**: Port security is a guest list (MACs) limiting club entry, stopping fake IDs (spoofed MACs) and overbooking (DHCP exhaustion).

**Lifelong Retention Tip**: Simulate DHCP exhaustion in Packet Tracer, apply port security (max 1 MAC). Flashcard: “Port security? Blocks unauthorized, DHCP exhaustion. Max MACs? Default 1.” Practice 3 times.

## Port Security Configuration

**Purpose**: Configure and verify port security, including violation modes, MAC addresses, and aging (topic 5.7).

**Key Commands**:
```plaintext
SW1# configure terminal
SW1(config)# interface g0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 2
SW1(config-if)# switchport port-security mac-address A.A.A
SW1(config-if)# switchport port-security violation {shutdown | restrict | protect}
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security aging time 30
SW1(config-if)# switchport port-security aging type {absolute | inactivity}
SW1(config-if)# switchport port-security aging static
SW1(config)# errdisable recovery cause psecure-violation
SW1(config)# errdisable recovery interval 180
```

**Key Points**:
- **Enable Port Security**:
  - Requires static `switchport mode access` or `trunk` (not dynamic auto/desirable, Day 22).
  - `switchport port-security`: Enables with defaults (max 1 MAC, shutdown mode).
- **Maximum MACs**:
  - Default: 1 MAC (e.g., A.A.A).
  - Configure: `switchport port-security maximum 2` (e.g., for VoIP phone + PC).
- **MAC Address Configuration**:
  - **Dynamic**: First MAC(s) learned (e.g., A.A.A).
  - **Static**: `switchport port-security mac-address A.A.A`.
  - **Sticky**: `switchport port-security mac-address sticky` (auto-adds learned MACs to running-config).
- **Violation Modes**:
  - **Shutdown** (default): Err-disables port, sends Syslog/SNMP, violation count = 1.
  - **Restrict**: Discards unauthorized traffic, port stays up, increments violation count, sends Syslog/SNMP.
  - **Protect**: Discards unauthorized traffic, port stays up, no Syslog/SNMP, no violation count.
- **Aging**:
  - Default: No aging (0 mins, permanent).
  - `switchport port-security aging time 30`: Sets aging (e.g., 30 mins).
  - Types: `absolute` (timer runs regardless of traffic), `inactivity` (resets with traffic).
  - `switchport port-security aging static`: Ages static MACs.
- **Recovery**:
  - **Manual**: `shutdown`, `no shutdown` on interface (after disconnecting unauthorized device).
  - **ErrDisable Recovery**: `errdisable recovery cause psecure-violation`, `errdisable recovery interval 180` (default 300s).
- **Verification Commands**:
```plaintext
SW1# show port-security
SW1# show port-security interface g0/1
SW1# show mac address-table secure
SW1# show errdisable recovery
SW1# show interfaces g0/1 switchport
SW1# show interfaces status
```

**Demonstration**:
- **Topology**: PC1 (A.A.A) → SW1 (G0/1), PC2 (B.B.B) attempts access.
- **Scenario 1 (Shutdown)**:
  - Configure: `switchport port-security`, max 1, shutdown mode.
  - PC1 pings R1: Works, MAC A.A.A learned.
  - PC2 pings R1: G0/1 err-disabled, Syslog sent, violation count = 1.
  - Recover: Disconnect PC2, `shutdown`, `no shutdown`.
- **Scenario 2 (Restrict)**:
  - Configure: `switchport port-security violation restrict`, `mac-address A.A.A`.
  - PC2 pings: Traffic dropped, port stays up, violation count increments, Syslog sent.
  - PC1 pings: Works (A.A.A authorized).
- **Scenario 3 (Protect)**:
  - Configure: `switchport port-security violation protect`, `mac-address A.A.A`.
  - PC2 pings: Traffic dropped, port stays up, no Syslog, violation count 0.
- **Scenario 4 (VoIP)**:
  - Topology: Phone1 (C.C.C) → SW1 (G0/1), PC1 (A.A.A) → Phone1.
  - Configure: `switchport port-security maximum 2`, sticky MACs.
  - Phone1/PC1: Both MACs learned, pings work.
  - PC2 (B.B.B): Triggers violation.
- **Verification**:
```plaintext
SW1# show port-security interface g0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses       : 1
Total MAC Addresses         : 1
Configured MAC Addresses    : 0
Sticky MAC Addresses        : 0
Last Source Address:Vlan    : A.A.A:1
Security Violation Count    : 0
```
- Post-PC2 (shutdown):
```plaintext
Port Status                : Secure-shutdown
Last Source Address:Vlan    : B.B.B:1
Security Violation Count    : 1
```
- Sticky MACs:
```plaintext
SW1# show running-config
interface GigabitEthernet0/1
 switchport port-security mac-address sticky A.A.A
```

**Comparison to Prior Content**:
- **Kali Linux Demo Lab**: Port security on G0/2 limits MACs, stops DHCP exhaustion.
- **Security Fundamentals Lecture**: Mitigates spoofing/DoS (topic 5.1).
- **Day 22 (DTP)**: Requires `switchport mode access`.
- **Syslog Lab**: Violations trigger `%PORT_SECURITY-2-PSECURE_VIOLATION`.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Complements dynamic ARP inspection.
- **Day 13 (DHCP)**: Protects DHCP pool.
- **Day 10 (IP Addressing)**: Secures 192.168.1.0/24 access.

**Analogy**: Configuring port security is like setting a guest list (MACs), choosing a reaction to fakes (shutdown, restrict, protect), and setting an expiration (aging).

**Lifelong Retention Tip**: Configure port security in Packet Tracer, test all violation modes. Flashcard: “Shutdown? Err-disabled, Syslog. Restrict? Drops, counts violations. Protect? Drops silently.” Practice 3 times.

## Common Issues and Troubleshooting

- **Port Security Not Enabled**:
  - **Issue**: `switchport port-security` rejected.
  - **Cause**: G0/1 in dynamic auto/desirable (DTP).
  - **Fix**: `switchport mode access`, then `switchport port-security`.
  - **Verify**: `show interfaces g0/1 switchport`.
- **Interface Err-Disabled**:
  - **Issue**: G0/1 err-disabled after PC2 connects.
  - **Cause**: Unauthorized MAC (B.B.B) in shutdown mode.
  - **Fix**: Disconnect PC2, `shutdown`, `no shutdown` or `errdisable recovery cause psecure-violation`.
  - **Verify**: `show port-security interface g0/1`, `show errdisable recovery`.
- **Sticky MACs Lost**:
  - **Issue**: Sticky MACs gone after reload.
  - **Cause**: Running-config not saved.
  - **Fix**: `write memory` after sticky learning.
  - **Verify**: `show running-config`.
- **VoIP Failure**:
  - **Issue**: Phone1 + PC1 fail on G0/1.
  - **Cause**: Max 1 MAC (default).
  - **Fix**: `switchport port-security maximum 2`.
  - **Verify**: `show port-security interface g0/1`.
- **DHCP Exhaustion Not Prevented**:
  - **Issue**: Attacker on G0/2 exhausts DHCP pool.
  - **Cause**: No port security on G0/2.
  - **Fix**: `switchport port-security maximum 1` on G0/2.
  - **Verify**: `show port-security`.

**Analogy**: Troubleshooting is like fixing a club’s entry system: ensure the door accepts IDs (static access), reset locks (err-disable recovery), and save the guest list (sticky MACs).

**Lifelong Retention Tip**: Simulate violations in Packet Tracer, recover with `no shutdown`. Checklist: “Err-disabled? Check MAC, mode, recovery.” Practice 3 times.

## Quiz Answers and Explanations

1. **How many secure MAC addresses were dynamically learned?**
   - Output: `Total MAC Addresses: 4`, `Configured MAC Addresses: 1`, `Sticky MAC Addresses: 3`.
   - **A**: 0, **B**: 1, **C**: 3, **D**: 4.
   - **Answer**: C (3 sticky MACs are dynamically learned, 1 configured is static).
   - **Explanation**: Sticky MACs are dynamically learned but appear as static in MAC table.

2. **What occurs in restrict mode? (Select two)**
   - **A**: Interface err-disabled, **B**: Unauthorized traffic discarded, **C**: Syslog not sent, **D**: SNMP Get sent, **E**: Violation counter incremented.
   - **Answer**: B, E (Restrict discards unauthorized traffic, increments violation count, sends Syslog/SNMP).
   - **Explanation**: SNMP Get is from manager to agent, not sent by switch.

3. **What does SW1 do with unauthorized frame on G0/1 (protect mode)?**
   - **A**: Drop traffic, **B**: Err-disable port, **C**: Send Syslog, **D**: Increment violation count.
   - **Answer**: A (Protect silently drops unauthorized traffic).
   - **Explanation**: No Syslog, no violation count, port stays up.

4. **What re-enables an err-disabled interface? (Select two)**
   - **A**: `shutdown`, `no shutdown`, **B**: `errdisable recovery cause psecure-violation`, **C**: Unplug device, **D**: Clear MAC table.
   - **Answer**: A, B (Manual shutdown/no shutdown or ErrDisable recovery).
   - **Explanation**: Unplugging alone doesn’t re-enable.

5. **What happens when `switchport port-security` is issued on G0/1 (static access)?**
   - **A**: Command accepted, **B**: Command rejected, **C**: Port err-disabled, **D**: MAC table cleared.
   - **Answer**: A (Static access allows port security).
   - **Explanation**: Dynamic auto/desirable rejects command.

**Bonus Question (Boson ExSim)**:
- **Question**: What is the default port security violation mode?
  - **A**: Protect, **B**: Restrict, **C**: Shutdown, **D**: Dynamic.
  - **Answer**: C (Shutdown err-disables port, topic 5.7).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Restrict? Drops, counts, Syslog. Protect? Drops silently. Shutdown? Err-disabled.”). Simulate violations in Packet Tracer. Review 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Violation Modes: “Shutdown, Restrict, Protect” (SRP).
   - Recovery: “Manual, ErrDisable” (ME).
   - MAC Types: “Static, Dynamic, Sticky” (SDS).
2. **Visualize Topology**:
   - Draw PC1 (A.A.A) → SW1 (G0/1), PC2 (B.B.B) blocked. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Port security? MAC control. Shutdown? Err-disabled. Sticky? Running-config. Aging? Absolute/inactivity.”
4. **Practice in Packet Tracer**:
   - Configure `switchport port-security`, test shutdown/restrict/protect, recover. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Shutdown (err-disabled) vs. Restrict (drops, counts) vs. Protect (drops silently).
   - Quiz: “Default mode? Shutdown. Sticky MACs? Dynamic but static in config.”
6. **Troubleshooting Practice**:
   - Simulate PC2 violation, recover with `no shutdown`. Checklist: “Err-disabled? Check MAC, mode, recovery.”
7. **Teach Someone**:
   - Explain port security and violation modes to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`switchport port-security`), modes, recovery. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Port Security**:
   - Controls MAC addresses (static/dynamic/sticky) and quantity per port.
   - Default: 1 MAC, shutdown mode, dynamic learning.
2. **Violation Modes**:
   - **Shutdown**: Err-disables port, Syslog/SNMP, violation count = 1.
   - **Restrict**: Drops unauthorized traffic, port up, counts violations, Syslog/SNMP.
   - **Protect**: Drops unauthorized traffic, port up, no Syslog, no count.
3. **Configuration**:
   - Requires `switchport mode access` or `trunk`.
   - Commands: `switchport port-security`, `maximum`, `mac-address`, `sticky`, `violation`, `aging`.
4. **Recovery**:
   - Manual: `shutdown`, `no shutdown`.
   - ErrDisable: `errdisable recovery cause psecure-violation`, interval 180s.
5. **Benefits**:
   - Prevents unauthorized access (Confidentiality).
   - Mitigates DHCP exhaustion, MAC flooding (Availability, topic 5.1).
6. **CCNA Focus**:
   - Topic 5.7: Configure/verify port security.
   - Topic 5.1: Mitigates spoofing/DoS.
   - Topic 2.2: Complements DHCP security.

These notes provide a comprehensive foundation for port security (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/modes, and apply analogies/retention tips to master concepts for life!