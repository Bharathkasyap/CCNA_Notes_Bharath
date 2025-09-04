# DHCP Snooping Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **DHCP snooping**, a Cisco switch security feature to protect against DHCP-based attacks, as outlined in **CCNA exam topic 5.7** ("Configure and verify Layer 2 security features: DHCP snooping, dynamic ARP inspection, and port security"). It builds on the **Port Security Lecture**, **Security Fundamentals Lecture** (topic 5.1), **DHCP Lecture** (Day 13, topic 2.2), and **Kali Linux Demo Lab** (DHCP exhaustion attack). The lecture explains DHCP snooping’s purpose, operation, protected attacks (DHCP starvation, DHCP poisoning), configuration, and troubleshooting, preparing you for configuration and verification tasks in the CCNA exam.

**Lecture Objectives**:
- **Understand DHCP Snooping**: Define DHCP snooping and its role in filtering DHCP messages.
- **Explain Benefits**: Prevent DHCP starvation and poisoning (Confidentiality, Availability, topic 5.1).
- **Configure DHCP Snooping**: Enable globally, per VLAN, set trusted ports, handle Option 82, and configure rate-limiting.
- **Verify and Troubleshoot**: Use `show` commands and recover from rate-limit violations.
- **Prepare for CCNA**: Master topic 5.7 (DHCP snooping configuration) and connect to topics 5.1 (security concepts) and 2.2 (DHCP).
- **Reinforce Prior Content**: Link to DHCP (Day 13), Port Security, Kali Linux Demo Lab (Day 48).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      SW2       |-------|      R1        |
| 172.16.1.10    | G0/1  | G0/0           | G0/1  | G0/0           |       | (DHCP Server)  |
| (Client)       |       |                |       |                |       | 172.16.1.1     |
+----------------+       +----------------+       +----------------+       +----------------+
|      PC2       | G0/2  |
| (Client)       |       |
+----------------+       |
|    Attacker    | G0/3  |
| 172.16.1.2     |       |
| (Spurious DHCP)|       |
+----------------+       |
```
- **Devices**:
  - **PC1, PC2**: DHCP clients on SW1 (G0/1–2), receive IPs (e.g., 172.16.1.10).
  - **Attacker**: Spurious DHCP server on SW1 (G0/3), MAC E.E.E, IP 172.16.1.2, attempts poisoning.
  - **SW1**: Cisco switch, DHCP snooping enabled, G0/1–2 untrusted (downlink), G0/0 trusted (uplink to SW2).
  - **SW2**: Cisco switch, DHCP snooping enabled, G0/1 untrusted (downlink to SW1), G0/0 trusted (uplink to R1).
  - **R1**: DHCP server or relay agent, IP 172.16.1.1.
- **Note**:
  - **Downlink Ports**: SW1’s G0/1–2 (to PC1–2), SW2’s G0/1 (to SW1), untrusted by default.
  - **Uplink Ports**: SW1’s G0/0 (to SW2), SW2’s G0/0 (to R1), configured as trusted.
  - **VLAN**: Single VLAN 1 (172.16.1.0/24).
- **Requirements**:
  1. Configure DHCP snooping on SW1/SW2 for VLAN 1, trust G0/0, untrust G0/1–3.
  2. Disable Option 82 to ensure DHCP functionality.
  3. Configure rate-limiting on SW1’s G0/1–3 to prevent DHCP starvation.
  4. Test legitimate DHCP (PC1–2) and attacker’s messages (G0/3).
  5. Verify binding table and recover from rate-limit violations.

**Analogy**: DHCP snooping is like a club’s security guard (SW1/SW2) checking IDs (DHCP messages) at the door (ports). Trusted doors (G0/0) let messages pass; untrusted doors (G0/1–3) check for fakes (spoofed MACs, server messages) and block floods (rate-limiting).

**Lifelong Retention Tip**: Visualize SW1 dropping Attacker’s DHCP messages on G0/3. Create flashcards: “DHCP snooping? Filters untrusted ports. Trusted? Uplinks. Rate-limit? Err-disabled.” Practice in Packet Tracer: configure, test violations, verify bindings. Review 3 times daily for a week.

## DHCP Snooping Fundamentals

**Purpose**: Define DHCP snooping and its role in filtering DHCP messages (topic 5.7).

**Key Points**:
- **Definition**: Cisco switch feature that filters DHCP messages on untrusted ports to prevent attacks.
- **Scope**: Filters only DHCP messages (e.g., DISCOVER, OFFER), not other traffic (e.g., ICMP).
- **Port Trust**:
  - **Untrusted (default)**: Downlink ports (e.g., SW1’s G0/1–2, SW2’s G0/1) to clients, inspected.
  - **Trusted**: Uplink ports (e.g., SW1’s G0/0, SW2’s G0/0) to DHCP server/relay, no inspection.
- **Operation**:
  - **Trusted Ports**: Forward DHCP messages without checks.
  - **Untrusted Ports**:
    - **Server Messages** (OFFER, ACK, NAK): Discarded.
    - **Client Messages** (DISCOVER, REQUEST, RELEASE, DECLINE): Inspected (MAC, binding table).
- **Binding Table**: Logs client MAC, IP, lease time, VLAN, interface after successful DHCP lease.
- **Exam Note**: Topic 5.7 requires configuring/verifying DHCP snooping, understanding trusted/untrusted ports.

**Demonstration**:
- **Topology**: PC1 → SW1 (G0/1, untrusted) → SW2 (G0/1, untrusted; G0/0, trusted) → R1.
- **Scenario**:
  - PC1 sends DISCOVER (MAC A.A.A). SW1 (G0/1) inspects, forwards to SW2 (G0/1), which forwards to R1 (G0/0, trusted).
  - R1’s OFFER/ACK returns via trusted G0/0 (SW2), G0/0 (SW1), no inspection.
  - Attacker (G0/3, untrusted) sends OFFER. SW1 discards (server message on untrusted port).
- **Verification**:
```plaintext
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               172.16.1.10      86400       dhcp-snooping  1     GigabitEthernet0/1
B.B.B               172.16.1.11      86400       dhcp-snooping  1     GigabitEthernet0/2
C.C.C               172.16.1.12      86400       dhcp-snooping  1     GigabitEthernet0/2
```

**Comparison to Prior Content**:
- **Port Security Lecture**: Complements port security (limits MACs, topic 5.7).
- **Security Fundamentals Lecture**: Mitigates DHCP-based attacks (Confidentiality, Availability, topic 5.1).
- **Kali Linux Demo Lab**: Prevents DHCP starvation (spoofed MACs).
- **DHCP Lecture (Day 13)**: Builds on D-O-R-A (DISCOVER, OFFER, REQUEST, ACK).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Prepares for dynamic ARP inspection (topic 5.7).
- **Day 10 (IP Addressing)**: Secures 172.16.1.0/24 DHCP assignments.
- **Day 15 (TCP/UDP)**: Protects DHCP (UDP 67/68).

**Analogy**: DHCP snooping is a gatekeeper checking client IDs (MACs) at untrusted doors (G0/1–3), trusting server messages only from uplink doors (G0/0).

**Lifelong Retention Tip**: Simulate PC1’s DISCOVER and Attacker’s OFFER in Packet Tracer, verify bindings. Flashcard: “DHCP snooping? Untrusted filters server messages. Binding table? MAC, IP, interface.” Practice 3 times.

## DHCP-Based Attacks and Mitigation

**Purpose**: Explain DHCP starvation and poisoning, and how DHCP snooping mitigates them (topics 5.1, 5.7).

**Key Points**:
- **DHCP Starvation (Exhaustion)**:
  - **Attack**: Attacker (G0/3) sends DISCOVER messages with spoofed MACs (e.g., E.E.E), exhausting R1’s DHCP pool (172.16.1.0/24).
  - **Impact**: Denies IPs to PC1–2 (DoS, Availability, topic 5.1).
  - **Mitigation**: DHCP snooping rate-limiting (e.g., 1 message/sec) err-disables G0/3 on flood.
  - **Prior Content**: Kali Linux Demo Lab (Day 48) showed starvation attack.
- **DHCP Poisoning**:
  - **Attack**: Attacker (G0/3, 172.16.1.2) runs spurious DHCP server, offers IPs with itself as default gateway (172.16.1.2), not R1 (172.16.1.1).
  - **Impact**: Man-in-the-middle (MITM), intercepts PC1’s traffic (Confidentiality, topic 5.1).
  - **Process**:
    - PC1 broadcasts DISCOVER.
    - Attacker’s OFFER (faster, local) arrives before R1’s (remote via relay).
    - PC1 accepts Attacker’s OFFER, sends DECLINE to R1, uses 172.16.1.2 as gateway.
    - PC1’s external traffic goes to Attacker, who examines/modifies before forwarding to R1.
  - **Mitigation**: DHCP snooping discards Attacker’s OFFER/ACK/NAK on untrusted G0/3.
- **Exam Note**: Topic 5.7 tests mitigation of DHCP attacks; topic 5.1 covers Confidentiality/Availability.

**Demonstration**:
- **Topology**: Attacker (G0/3) vs. R1 (DHCP server).
- **Starvation**:
  - Attacker floods DISCOVERs (spoofed MACs). SW1’s G0/3 rate-limit (1/sec) err-disables port.
  - Verify: `show ip dhcp snooping binding` (no Attacker entries).
- **Poisoning**:
  - Attacker sends OFFER (172.16.1.2 as gateway). SW1 discards (server message, untrusted).
  - PC1 gets R1’s OFFER, uses 172.16.1.1 as gateway.
  - Verify: `show ip dhcp snooping binding` (PC1: 172.16.1.10, gateway 172.16.1.1).

**Comparison to Prior Content**:
- **Kali Linux Demo Lab**: DHCP snooping prevents starvation (spoofed MACs).
- **Port Security Lecture**: Complements port security (limits MACs, topic 5.7).
- **Security Fundamentals Lecture**: Mitigates DoS (starvation) and MITM (poisoning, topic 5.1).
- **DHCP Lecture (Day 13)**: Protects D-O-R-A, introduces DECLINE.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Similar to ARP poisoning (MITM), mitigated by dynamic ARP inspection.
- **Day 10 (IP Addressing)**: Protects 172.16.1.0/24 assignments.
- **Day 15 (TCP/UDP)**: Secures DHCP (UDP 67/68).

**Analogy**: DHCP snooping stops fake vendors (Attacker) from selling bad tickets (OFFERs) at untrusted gates (G0/3) and limits ticket requests (rate-limiting) to prevent a rush (starvation).

**Lifelong Retention Tip**: Simulate starvation (flood DISCOVERs) and poisoning (spurious OFFER) in Packet Tracer. Flashcard: “Starvation? Flood DISCOVERs, rate-limit stops. Poisoning? Fake OFFER, untrusted discards.” Practice 3 times.

## DHCP Message Types and Snooping Operation

**Purpose**: Review DHCP message types and explain how DHCP snooping filters them (topic 5.7).

**Key Points**:
- **DHCP Message Types**:
  - **Server Messages** (discarded on untrusted ports):
    - **OFFER**: Offers IP to client (D-O-R-A).
    - **ACK**: Confirms lease (D-O-R-A).
    - **NAK**: Declines client’s REQUEST.
  - **Client Messages** (inspected on untrusted ports):
    - **DISCOVER**: Requests IP (D-O-R-A).
    - **REQUEST**: Accepts OFFER (D-O-R-A).
    - **RELEASE**: Returns IP to server.
    - **DECLINE**: Rejects OFFER (e.g., Attacker’s).
- **Snooping Operation**:
  - **Trusted Ports**: Forward all DHCP messages (e.g., SW2’s G0/0 to R1).
  - **Untrusted Ports**:
    - **Server Messages** (OFFER, ACK, NAK): Discarded (e.g., Attacker’s OFFER on G0/3).
    - **DISCOVER/REQUEST**:
      - Check: Frame’s source MAC (e.g., A.A.A) vs. DHCP Client Hardware Address (CHADDR).
      - Match: Forward to R1.
      - Mismatch: Discard (prevents CHADDR spoofing).
    - **RELEASE/DECLINE**:
      - Check: Packet’s source IP and interface vs. DHCP snooping binding table.
      - Match: Forward to R1.
      - Mismatch: Discard (prevents fake RELEASE/DECLINE).
- **Binding Table**:
  - Created on successful lease (e.g., PC1: A.A.A, 172.16.1.10, G0/1, VLAN 1).
  - Used to validate RELEASE/DECLINE.
- **Exam Note**: Topic 5.7 requires knowing message types and filtering logic.

**Demonstration**:
- **Topology**: PC1 (G0/1) sends DISCOVER, Attacker (G0/3) sends OFFER.
- **Operation**:
  - PC1’s DISCOVER: MAC A.A.A = CHADDR, forwarded.
  - Attacker’s OFFER: Discarded (server message, untrusted G0/3).
  - PC1’s RELEASE: IP 172.16.1.10, G0/1 matches binding table, forwarded.
  - Attacker’s RELEASE (spoofed 172.16.1.10): Discarded (no binding table match).
- **Verification**:
```plaintext
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               172.16.1.10      86400       dhcp-snooping  1     GigabitEthernet0/1
```

**Comparison to Prior Content**:
- **DHCP Lecture (Day 13)**: D-O-R-A (DISCOVER, OFFER, REQUEST, ACK), RELEASE introduced.
- **Port Security Lecture**: Complements MAC limiting with CHADDR checks.
- **Kali Linux Demo Lab**: Prevents spoofed DISCOVERs.
- **Security Fundamentals Lecture**: Protects Confidentiality (poisoning), Availability (starvation).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Similar MAC checks for dynamic ARP inspection.
- **Day 10 (IP Addressing)**: Ensures valid 172.16.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Secures DHCP (UDP 67/68).

**Analogy**: DHCP snooping checks client IDs (DISCOVER/REQUEST) and tickets (RELEASE/DECLINE) at untrusted gates, rejecting fake server responses (OFFER/ACK/NAK).

**Lifelong Retention Tip**: Memorize message types (OFFER/ACK/NAK: server, DISCOVER/REQUEST/RELEASE/DECLINE: client). Simulate in Packet Tracer, verify bindings. Flashcard: “Server messages? Discarded untrusted. Client? Check MAC, binding.” Practice 3 times.

## DHCP Snooping Configuration

**Purpose**: Configure and verify DHCP snooping, including trusted ports, Option 82, and rate-limiting (topic 5.7).

**Key Commands**:
```plaintext
SW1# configure terminal
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface g0/0
SW1(config-if)# ip dhcp snooping trust
SW1(config)# interface range g0/1 - 3
SW1(config-if-range)# ip dhcp snooping limit rate 1
SW1(config)# errdisable recovery cause dhcp-rate-limit
```

**Key Points**:
- **Enable DHCP Snooping**:
  - Globally: `ip dhcp snooping`.
  - Per VLAN: `ip dhcp snooping vlan 1` (VLAN 1 in topology).
- **Trusted Ports**:
  - Default: All ports untrusted.
  - Configure: `ip dhcp snooping trust` on uplinks (SW1’s G0/0, SW2’s G0/0).
- **Option 82**:
  - **Purpose**: DHCP relay agent adds interface/VLAN info (topic 2.2).
  - **Issue**: Switches add Option 82 by default, dropped on untrusted ports or by R1 (non-relay).
  - **Fix**: `no ip dhcp snooping information option` on SW1/SW2.
- **Rate-Limiting**:
  - Configure: `ip dhcp snooping limit rate 1` (1 message/sec, low for demo).
  - Exceed: Port err-disabled, Syslog `%DHCP_SNOOPING-5-DHCP_SNOOPING_RATE_LIMIT_EXCEEDED`.
  - Recover: `shutdown`, `no shutdown` or `errdisable recovery cause dhcp-rate-limit`.
- **Verification Commands**:
```plaintext
SW1# show ip dhcp snooping
SW1# show ip dhcp snooping binding
SW1# show errdisable recovery
```

**Demonstration**:
- **Topology**: SW1 (G0/0 trusted, G0/1–3 untrusted, rate-limit 1), SW2 (G0/0 trusted, G0/1 untrusted).
- **Scenario 1 (Legitimate DHCP)**:
  - Configure: `ip dhcp snooping`, `vlan 1`, `no ip dhcp snooping information option`, trust G0/0.
  - PC1 sends DISCOVER, gets IP 172.16.1.10 from R1.
  - Verify:
```plaintext
SW1# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
A.A.A               172.16.1.10      86400       dhcp-snooping  1     GigabitEthernet0/1
```
- **Scenario 2 (Option 82 Issue)**:
  - Without `no ip dhcp snooping information option`:
    - SW1 adds Option 82 to PC1’s DISCOVER, SW2 drops (untrusted G0/1).
    - Syslog: `%DHCP_SNOOPING-5-DHCP_SNOOPING_UNTRUSTED_PORT`.
    - SW2 adds Option 82, R1 drops (non-relay).
    - Syslog: `%DHCP_SNOOPING-5-DHCP_SNOOPING_INVALID_RELAY`.
  - Fix: Add `no ip dhcp snooping information option` on SW1/SW2.
- **Scenario 3 (Rate-Limit Violation)**:
  - Attacker floods DISCOVERs on G0/3 (>1/sec).
  - G0/3 err-disabled, Syslog: `%DHCP_SNOOPING-5-DHCP_SNOOPING_RATE_LIMIT_EXCEEDED`.
  - Recover: `shutdown`, `no shutdown` or `errdisable recovery`.
  - Verify:
```plaintext
SW1# show errdisable recovery
ErrDisable Reason            Timer Status
-----------------            --------------
dhcp-rate-limit              Enabled
Timer interval: 300 seconds
Interfaces that will be enabled at the next timeout:
Interface       Errdisable reason       Time left(sec)
---------       ---------------------   --------------
Gi0/3           dhcp-rate-limit         280
```

**Comparison to Prior Content**:
- **Port Security Lecture**: Rate-limiting complements MAC limiting (topic 5.7).
- **Kali Linux Demo Lab**: Rate-limiting stops DHCP starvation.
- **Security Fundamentals Lecture**: Mitigates DoS (starvation), MITM (poisoning, topic 5.1).
- **DHCP Lecture (Day 13)**: Protects D-O-R-A, Option 82 introduced.

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Prepares for dynamic ARP inspection.
- **Day 10 (IP Addressing)**: Secures 172.16.1.0/24 leases.
- **Day 15 (TCP/UDP)**: Protects DHCP (UDP 67/68).

**Analogy**: Configuring DHCP snooping is like setting a club’s security system: activate globally (VLAN 1), trust VIP doors (G0/0), block fake tickets (server messages), and limit entry speed (rate-limiting).

**Lifelong Retention Tip**: Configure DHCP snooping in Packet Tracer, test Option 82 issues, rate-limit violations. Flashcard: “DHCP snooping? Global, VLAN, trust uplinks. Option 82? Disable for non-relay. Rate-limit? Err-disabled.” Practice 3 times.

## Common Issues and Troubleshooting

- **DHCP Fails (Option 82)**:
  - **Issue**: PC1–2 don’t get IPs.
  - **Cause**: SW1/SW2 add Option 82, dropped by SW2 (untrusted) or R1 (non-relay).
  - **Fix**: `no ip dhcp snooping information option` on SW1/SW2.
  - **Verify**: `show ip dhcp snooping`, `show ip dhcp snooping binding`.
- **Port Err-Disabled (Rate-Limit)**:
  - **Issue**: G0/3 err-disabled after Attacker’s DISCOVER flood.
  - **Cause**: Exceeds `ip dhcp snooping limit rate 1`.
  - **Fix**: `shutdown`, `no shutdown` or `errdisable recovery cause dhcp-rate-limit`.
  - **Verify**: `show errdisable recovery`, `show ip dhcp snooping`.
- **Server Messages Forwarded**:
  - **Issue**: Attacker’s OFFER on G0/3 forwarded.
  - **Cause**: G0/3 trusted (misconfigured).
  - **Fix**: Remove `ip dhcp snooping trust` from G0/3.
  - **Verify**: `show ip dhcp snooping`.
- **Binding Table Empty**:
  - **Issue**: No entries in `show ip dhcp snooping binding`.
  - **Cause**: DHCP snooping not enabled for VLAN 1 or trusted ports misconfigured.
  - **Fix**: `ip dhcp snooping vlan 1`, trust G0/0.
  - **Verify**: `show ip dhcp snooping binding`.

**Analogy**: Troubleshooting is like fixing a club’s security system: disable fake ticket scanners (Option 82), reset locked doors (rate-limit), and ensure VIP doors (trusted) are correct.

**Lifelong Retention Tip**: Simulate Option 82 drops and rate-limit violations in Packet Tracer. Checklist: “No IPs? Check Option 82, trust. Err-disabled? Rate-limit, recover.” Practice 3 times.

## Quiz Answers and Explanations

1. **Which DHCP message types are discarded on untrusted interfaces? (Select three)**
   - **A**: DISCOVER, **B**: REQUEST, **C**: NAK, **D**: OFFER, **E**: RELEASE, **F**: DECLINE, **G**: ACK.
   - **Answer**: C, D, G (NAK, OFFER, ACK are server messages, discarded on untrusted ports).
   - **Explanation**: Client messages (DISCOVER, REQUEST, RELEASE, DECLINE) are inspected, not discarded.

2. **What is NOT stored in the DHCP snooping binding database?**
   - **A**: MAC address, **B**: IP address, **C**: Interface, **D**: Default gateway.
   - **Answer**: D (Default gateway not stored).
   - **Explanation**: Binding table includes MAC, IP, lease time, VLAN, interface.

3. **What are functions of DHCP snooping? (Select two)**
   - **A**: Limiting DHCP message rate, **B**: Filtering all DHCP messages, **C**: Filtering messages on untrusted ports, **D**: Filtering trusted ports.
   - **Answer**: A, C (Rate-limiting, filtering untrusted ports).
   - **Explanation**: Trusted ports are not filtered; only untrusted ports are.

4. **What does DHCP snooping check for DISCOVER on untrusted interface? (Select two)**
   - **A**: Source MAC address, **B**: Client hardware address, **C**: Source IP address, **D**: Default gateway.
   - **Answer**: A, B (Frame’s source MAC vs. CHADDR).
   - **Explanation**: Mismatch discards DISCOVER/REQUEST; IP checks for RELEASE/DECLINE.

5. **What happens if DHCP messages exceed rate-limit on G0/1?**
   - **A**: Messages forwarded, **B**: Interface disabled, **C**: Messages queued, **D**: Syslog not sent.
   - **Answer**: B (Interface err-disabled).
   - **Explanation**: Recover with `shutdown`, `no shutdown` or `errdisable recovery`.

**Bonus Question (Boson ExSim)**:
- **Question**: What is the default port state for DHCP snooping?
  - **A**: Trusted, **B**: Untrusted, **C**: Disabled, **D**: Rate-limited.
  - **Answer**: B (All ports untrusted by default, topic 5.7).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Server messages? OFFER, ACK, NAK, discarded. Binding table? No gateway.”). Simulate violations in Packet Tracer. Review 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Message Types: “Server: OAN (OFFER, ACK, NAK). Client: DRRD (DISCOVER, REQUEST, RELEASE, DECLINE).”
   - Config Steps: “Global, VLAN, Trust, No Option 82, Rate-limit” (GVTNR).
   - Attacks: “Starvation, Poisoning” (SP).
2. **Visualize Topology**:
   - Draw PC1 → SW1 (G0/1 untrusted) → SW2 (G0/0 trusted) → R1, Attacker on G0/3. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DHCP snooping? Filters untrusted. Trusted? Uplinks. Rate-limit? Err-disabled. Option 82? Disable for non-relay.”
4. **Practice in Packet Tracer**:
   - Configure DHCP snooping, test Attacker’s OFFER, rate-limit violations. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Starvation (rate-limit) vs. Poisoning (discard server messages).
   - Quiz: “OFFER/ACK/NAK? Discarded untrusted. DISCOVER? Check MAC, CHADDR.”
6. **Troubleshooting Practice**:
   - Simulate Option 82 drops, rate-limit err-disable. Checklist: “No IPs? Check Option 82, trust. Err-disabled? Recover.”
7. **Teach Someone**:
   - Explain DHCP snooping and attacks to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands (`ip dhcp snooping`), message types, attacks. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DHCP Snooping**:
   - Filters DHCP messages on untrusted ports (e.g., G0/1–3), trusts uplinks (G0/0).
   - Discards server messages (OFFER, ACK, NAK) on untrusted ports.
   - Inspects client messages (DISCOVER/REQUEST: MAC vs. CHADDR; RELEASE/DECLINE: binding table).
2. **Attacks Mitigated**:
   - **Starvation**: Rate-limiting prevents DISCOVER floods (Availability, topic 5.1).
   - **Poisoning**: Discards spurious OFFERs (Confidentiality, topic 5.1).
3. **Configuration**:
   - Enable: `ip dhcp snooping`, `vlan 1`, `trust`, `no ip dhcp snooping information option`.
   - Rate-limit: `ip dhcp snooping limit rate 1`, recover with `errdisable recovery`.
4. **Verification**:
   - `show ip dhcp snooping`, `show ip dhcp snooping binding`, `show errdisable recovery`.
5. **CCNA Focus**:
   - Topic 5.7: Configure/verify DHCP snooping (trusted ports, Option 82, rate-limiting).
   - Topic 5.1: Mitigates DoS (starvation), MITM (poisoning).
   - Topic 2.2: Protects DHCP D-O-R-A process.
6. **Binding Table**: Logs MAC, IP, lease, VLAN, interface for valid leases.

These notes provide a comprehensive foundation for DHCP snooping (CCNA topic 5.7). Practice configurations in Packet Tracer, use flashcards for commands/messages, and apply analogies/retention tips to master concepts for life!