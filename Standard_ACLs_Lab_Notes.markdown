# Standard ACLs Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring standard Access Control Lists (ACLs) to filter IPv4 traffic based on source IP addresses, aligning with CCNA exam topic 5.1 ("Configure and verify access control lists"). It also includes basic OSPF configuration to establish network connectivity, supporting exam topic 3.4 ("Configure and verify single area OSPFv2"). The lab uses Packet Tracer to configure OSPF on R1 and R2, followed by standard numbered ACLs on R1 and named ACLs on R2 to meet specific access requirements. It builds on the Standard ACLs lecture (Day 34) and assumes familiarity with IPv6 concepts from prior labs (Part 1: manual addressing, Part 2: EUI-64, static routes, Part 3: SLAAC, advanced static routes).

**Lab Objectives**:
- **Configure OSPF**: Enable single-area OSPFv2 on R1 and R2 for full network connectivity.
- **Configure Standard ACLs**: Set up numbered ACLs on R1 and named ACLs on R2 to restrict traffic per requirements.
- **Apply ACLs**: Apply ACLs outbound on appropriate interfaces, following the "close to destination" rule.
- **Verify Configurations**: Use `show` commands to confirm OSPF and ACL functionality.
- **Test Connectivity**: Verify ACLs with pings (`ping`, `ping -t`) to ensure correct traffic filtering.
- **Prepare for Exam**: Master topic 5.1 (ACLs) and reinforce 3.4 (OSPFv2), with ties to 1.8/1.9 (IPv6 comparisons).

**Network Topology**:
- **Devices**:
  - **R1**:
    - G0/0: Connected to `172.16.1.0/24` (PC1: `172.16.1.1`, PC2: `172.16.1.2`).
    - G0/1: Connected to `172.16.2.0/24` (PC3: `172.16.2.1`, PC4: `172.16.2.2`).
    - S0/0/0: Point-to-point to R2 (`203.0.113.0/30`, R1: `203.0.113.1`).
  - **R2**:
    - S0/0/0: Point-to-point to R1 (`203.0.113.2`).
    - G0/0: Connected to `192.168.1.0/24` (SRV1: `192.168.1.100`).
    - G0/1: Connected to `192.168.2.0/24` (SRV2: `192.168.2.100`).
- **Note**: PCs/SRV connect via switches (implied, not shown in diagram).
- **OSPF**:
  - Single-area (Area 0), enabled on all interfaces.
  - Routes: R1 learns `192.168.1.0/24`, `192.168.2.0/24`; R2 learns `172.16.1.0/24`, `172.16.2.0/24`.
- **ACL Requirements**:
  1. Allow only PC1 (`172.16.1.1`) and PC3 (`172.16.2.1`) to access `192.168.1.0/24` (SRV1).
  2. Deny `172.16.2.0/24` (PC3, PC4) from accessing `192.168.2.0/24` (SRV2).
  3. Deny `172.16.1.0/24` (PC1, PC2) from accessing `172.16.2.0/24` (PC3, PC4).
  4. Deny `172.16.2.0/24` (PC3, PC4) from accessing `172.16.1.0/24` (PC1, PC2).

**Analogy**: The lab is like setting up a postal network (OSPF) to deliver mail between cities (subnets), then adding security guards (ACLs) at city gates (interfaces) to check sender addresses (source IPs). Numbered ACLs are numbered guest lists, named ACLs are labeled lists, and guards block or allow mail based on rules, placed near the destination city.

**Lifelong Retention Tip**: Visualize R2 blocking PC2’s ping to SRV1 with a named ACL on G0/0 outbound. Practice in Packet Tracer: configure OSPF, add `access-list 1 deny 172.16.1.0 0.0.0.255`, apply `ip access-group 1 out`, test pings. Create flashcards for commands (`access-list`, `ip access-list standard`, `network <ip> <mask> area 0`). Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure OSPF on R1 and R2
**Objective**: Enable single-area OSPFv2 to ensure full connectivity between all subnets.

**Commands (R1)**:
```plaintext
R1> enable
R1# configure terminal
R1(config)# router ospf 1
R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
R1(config-router)# network 203.0.113.0 0.0.0.3 area 0
R1(config-router)# end
R1# show ip ospf interface
R1# show ip route
```

**Commands (R2)**:
```plaintext
R2> enable
R2# configure terminal
R2(config)# router ospf 1
R2(config-router)# network 192.168.0.0 0.0.255.255 area 0
R2(config-router)# network 203.0.113.0 0.0.0.3 area 0
R2(config-router)# end
R2# show ip ospf interface
R2# show ip ospf neighbor
R2# show ip route
```

**Observations**:
- **OSPF Configuration**:
  - `router ospf 1`: Enables OSPF process 1.
  - `network <ip> <wildcard-mask> area 0`: Activates OSPF on interfaces matching the network (e.g., `172.16.0.0 0.0.255.255` covers `172.16.1.0/24`, `172.16.2.0/24`).
  - Serial link (`203.0.113.0/30`) uses point-to-point OSPF network type (no DR/BDR, shown as `-` in `show ip ospf neighbor`).
- **Verification**:
  - `show ip ospf interface`: Confirms OSPF on G0/0, G0/1, S0/0/0 for both routers.
  - `show ip ospf neighbor`: R1 and R2 are in FULL state (point-to-point).
  - `show ip route`: R1 learns `192.168.1.0/24`, `192.168.2.0/24`; R2 learns `172.16.1.0/24`, `172.16.2.0/24` via OSPF (O routes).
- **Outcome**: Full connectivity (e.g., PC1 can ping SRV1, SRV2, PC3) before ACLs.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1 Lecture/Lab**: Used manual IPv6 addresses, no dynamic routing. OSPF here ensures connectivity, like `ipv6 unicast-routing` in IPv6 labs.
- **IPv6 Part 2 Lecture/Lab**: Static routes (`ipv6 route`) provided connectivity, similar to OSPF’s dynamic routes here.
- **IPv6 Part 3 Lecture/Lab**: Static routes with SLAAC; OSPF here is dynamic, but both enable inter-subnet communication.
- **Difference**: IPv6 labs used static routes or no routing; this lab uses OSPFv2 for dynamic routing.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Static routes (`ip route`) are manual, unlike OSPF’s dynamic routes.
- **EIGRP**: Similar to OSPF, uses wildcard masks (`0.0.0.255`) and dynamic routing.
- **VLANs**: No routing, but VLANs segment traffic, complementing OSPF’s connectivity.

**Analogy**: OSPF is like a postal service automatically mapping delivery routes (routes) between cities (subnets), ensuring mail (packets) reaches destinations before guards (ACLs) check senders.

**Lifelong Retention Tip**: In Packet Tracer, configure OSPF on R1 (`network 172.16.0.0 0.0.255.255 area 0`), verify with `show ip route`. Flashcard: “OSPF command? `router ospf 1`, `network <ip> <mask> area 0`.” Practice 3 times.

### Step 2: Configure and Apply Standard ACLs
**Objective**: Configure numbered ACLs on R1 and named ACLs on R2 to meet the four requirements, applying them outbound close to the destination.

**Requirements and Configurations**:
1. **Allow PC1 (`172.16.1.1`) and PC3 (`172.16.2.1`) to `192.168.1.0/24` (SRV1), deny others**:
   - **Location**: R2, outbound on G0/0 (close to `192.168.1.0/24`).
   - **Commands**:
     ```plaintext
     R2(config)# ip access-list standard TO_192.168.1.0
     R2(config-std-nacl)# permit 172.16.1.1
     R2(config-std-nacl)# permit 172.16.2.1
     R2(config-std-nacl)# deny any
     R2(config-std-nacl)# exit
     R2(config)# interface g0/0
     R2(config-if)# ip access-group TO_192.168.1.0 out
     ```
   - **Notes**:
     - Named ACL for clarity (`TO_192.168.1.0`).
     - Explicit `deny any` (optional, mirrors implicit deny).
     - Outbound on G0/0 filters traffic to SRV1.

2. **Deny `172.16.2.0/24` (PC3, PC4) from `192.168.2.0/24` (SRV2), allow others**:
   - **Location**: R2, outbound on G0/1 (close to `192.168.2.0/24`).
   - **Commands**:
     ```plaintext
     R2(config)# ip access-list standard TO_192.168.2.0
     R2(config-std-nacl)# deny 172.16.2.0 0.0.0.255
     R2(config-std-nacl)# permit any
     R2(config-std-nacl)# exit
     R2(config)# interface g0/1
     R2(config-if)# ip access-group TO_192.168.2.0 out
     ```
   - **Notes**: Denies PC3, PC4; permits PC1, PC2 to SRV2.

3. **Deny `172.16.1.0/24` (PC1, PC2) from `172.16.2.0/24` (PC3, PC4)**:
   - **Location**: R1, outbound on G0/1 (close to `172.16.2.0/24`).
   - **Commands**:
     ```plaintext
     R1(config)# access-list 1 deny 172.16.1.0 0.0.0.255
     R1(config)# access-list 1 permit any
     R1(config)# interface g0/1
     R1(config-if)# ip access-group 1 out
     ```

4. **Deny `172.16.2.0/24` (PC3, PC4) from `172.16.1.0/24` (PC1, PC2)**:
   - **Location**: R1, outbound on G0/0 (close to `172.16.1.0/24`).
   - **Commands**:
     ```plaintext
     R1(config)# access-list 2 deny 172.16.2.0 0.0.0.255
     R1(config)# access-list 2 permit any
     R1(config)# interface g0/0
     R1(config-if)# ip access-group 2 out
     ```

**Observations**:
- **Numbered vs. Named ACLs**:
  - Numbered (R1): `access-list <1-99>`, simple, uses numbers (1, 2).
  - Named (R2): `ip access-list standard <name>`, descriptive (e.g., `TO_192.168.1.0`), allows sequence numbers.
- **Wildcard Masks**: `0.0.0.255` for `/24`, `0.0.0.0` or `host` for `/32`.
- **Close to Destination**: ACLs on R2 G0/0, G0/1 for `192.168.x.0/24`; R1 G0/0, G0/1 for `172.16.x.0/24`.
- **Implicit Deny**: `permit any` counters implicit deny, ensuring non-targeted traffic passes.
- **Application**: `ip access-group <number|name> out` applies ACLs outbound.

**Verification**:
```plaintext
R2# show access-lists
Standard IP access list TO_192.168.1.0
    10 permit 172.16.1.1 (X matches)
    20 permit 172.16.2.1 (Y matches)
    30 deny any (Z matches)
Standard IP access list TO_192.168.2.0
    10 deny 172.16.2.0, wildcard bits 0.0.0.255 (W matches)
    20 permit any
R1# show access-lists
Standard IP access list 1
    10 deny 172.16.1.0, wildcard bits 0.0.0.255 (A matches)
    20 permit any
Standard IP access list 2
    10 deny 172.16.2.0, wildcard bits 0.0.0.255 (B matches)
    20 permit any
```

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1 Lecture/Lab**: No ACLs or dynamic routing; used manual IPv6 addresses (`2001:db8::/64`) and `ipv6 unicast-routing`.
- **IPv6 Part 2 Lecture/Lab**: Static routes (`ipv6 route`) controlled paths, like OSPF here; no filtering.
- **IPv6 Part 3 Lecture/Lab**: Static routes and SLAAC; ACLs here filter traffic, unlike routes directing paths.
- **Difference**: IPv6 labs used static routes; this lab uses OSPF for connectivity and ACLs for filtering.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Manual routes vs. OSPF’s dynamic routes; ACLs filter, not route.
- **EIGRP**: Similar to OSPF, uses wildcard masks (`0.0.0.255`), but no filtering.
- **Standard ACL Lecture (Day 34)**: Covered ACL logic (sequential, implicit deny), numbered/named ACLs, close-to-destination rule, applied here.

**Analogy**: OSPF is a postal map for mail delivery; ACLs are guards at city gates (interfaces) checking sender addresses (source IPs). Numbered ACLs are numbered lists (1, 2); named ACLs are labeled (e.g., `TO_SRV1`).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip access-list standard TO_192.168.1.0`, add `permit 172.16.1.1`, apply on G0/0 out. Flashcard: “Named ACL? `ip access-list standard <name>`.” Practice 3 times.

### Step 3: Test and Verify ACLs
**Objective**: Confirm ACLs enforce requirements using pings (`ping`, `ping -t`) and verify matches with `show access-lists`.

**Tests**:
- **Requirement 1 (PC1, PC3 to SRV1)**:
  ```plaintext
  PC1> ping 192.168.1.100
  Reply from 192.168.1.100: bytes=32 time<1ms TTL=128
  PC2> ping 192.168.1.100
  Destination host unreachable
  ```
  - **Result**: PC1 succeeds (matches `permit 172.16.1.1`), PC2 fails (implicit deny or `deny any`).
- **Requirement 2 (PC3, PC4 to SRV2)**:
  ```plaintext
  PC1> ping 192.168.2.100
  Reply from 192.168.2.100: bytes=32 time<1ms TTL=128
  PC3> ping 192.168.2.100
  Destination host unreachable
  ```
  - **Result**: PC1 succeeds (`permit any`), PC3 fails (`deny 172.16.2.0 0.0.0.255`).
- **Requirement 3/4 (PC1–PC3, PC3–PC1)**:
  ```plaintext
  PC1> ping -t 172.16.2.1
  Destination host unreachable
  PC3> ping -t 172.16.1.1
  Destination host unreachable
  ```
  - **Result**: Pings fail (`deny 172.16.1.0 0.0.0.255` on R1 G0/1, `deny 172.16.2.0 0.0.0.255` on G0/0).

**Verification**:
- `show access-lists`: Shows match counts (e.g., PC2’s failed pings increment `deny any`).
- `ping -t`: Continuous ping (stop with Ctrl+C), useful for real-time ACL match monitoring.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 3 Lab**: Used `ping` and `tracert` to verify IPv6 static routes; here, `ping` verifies ACL filtering.
- **Difference**: IPv6 lab tested connectivity (R1–R3 path); this lab tests access restrictions.

**Comparison to Other CCNA Topics**:
- **IPv4 Routing**: Pings verify routes (OSPF); ACLs block specific pings.
- **Standard ACL Lecture**: Emphasized `show access-lists` for verification, match counts shown here.

**Analogy**: Testing ACLs is like checking if the guard (ACL) lets the right mail (packets) through. Continuous pings (`ping -t`) are like sending test letters to see which get blocked.

**Lifelong Retention Tip**: In Packet Tracer, ping from PC2 to SRV1 (should fail), check `show access-lists` for matches. Flashcard: “Verify ACL? `show access-lists` for matches.” Practice 3 times.

## Common Issues and Troubleshooting

- **Traffic Not Blocked**:
  - **Issue**: PC2 can ping SRV1.
  - **Cause**: ACL not applied (`ip access-group TO_192.168.1.0 out`) or wrong interface (e.g., G0/1).
  - **Fix**: Verify with `show running-config | include access-group`, apply on R2 G0/0 out.
- **All Traffic Blocked**:
  - **Issue**: PC1 can’t ping SRV1.
  - **Cause**: Missing `permit 172.16.1.1` or `permit any` (implicit deny blocks all).
  - **Fix**: Add `permit 172.16.1.1`, reapply ACL, check `show access-lists`.
- **Wrong ACL Applied**:
  - **Issue**: ACL 2 applied on G0/1 instead of ACL 1.
  - **Cause**: Configuration error.
  - **Fix**: Verify with `show running-config | section access-list`, reapply correct ACL.
- **OSPF Failure**:
  - **Issue**: No connectivity before ACLs.
  - **Cause**: Incorrect `network` statement or OSPF not enabled.
  - **Fix**: Check `show ip ospf interface`, ensure `network 172.16.0.0 0.0.255.255 area 0`.

**Analogy**: Troubleshooting is like fixing a guard’s guest list: check if the list (ACL) is at the right gate (interface), has the right names (permit/deny), and allows others (permit any).

**Lifelong Retention Tip**: Simulate failed ping (PC2 to SRV1), check `show access-lists`, adjust ACL. Checklist: “No ping? Check interface, direction, permit any, OSPF routes.” Practice 3 times.

## Practice Commands

- **OSPF**:
  ```plaintext
  R1(config)# router ospf 1
  R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
  R1(config-router)# network 203.0.113.0 0.0.0.3 area 0
  ```
- **Numbered ACL (R1)**:
  ```plaintext
  R1(config)# access-list 1 deny 172.16.1.0 0.0.0.255
  R1(config)# access-list 1 permit any
  R1(config)# interface g0/1
  R1(config-if)# ip access-group 1 out
  ```
- **Named ACL (R2)**:
  ```plaintext
  R2(config)# ip access-list standard TO_192.168.1.0
  R2(config-std-nacl)# permit 172.16.1.1
  R2(config-std-nacl)# permit 172.16.2.1
  R2(config-std-nacl)# deny any
  R2(config-std-nacl)# exit
  R2(config)# interface g0/0
  R2(config-if)# ip access-group TO_192.168.1.0 out
  ```
- **Verify**:
  ```plaintext
  show ip ospf interface
  show ip ospf neighbor
  show ip route
  show access-lists
  show running-config | section access-list
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - ACL: “Permit/deny source IP, close to destination, implicit deny.”
   - OSPF: “Network <ip> <mask> area 0, FULL state for neighbors.”
2. **Visualize Topology**:
   - Draw R1–R2, show ACLs on R2 G0/0 (SRV1), G0/1 (SRV2), R1 G0/0, G0/1. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Standard ACL range? 1–99, 1300–1999. Named ACL command? `ip access-list standard <name>`. OSPF verify? `show ip ospf neighbor`.”
4. **Practice in Packet Tracer**:
   - Configure OSPF, ACLs, test pings (PC1 to SRV1, PC2 to SRV1), verify with `show access-lists`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: IPv6 Part 3 (static routes, SLAAC) vs. ACLs (filtering) vs. OSPF (dynamic routing).
6. **Troubleshooting Practice**:
   - Simulate failed ping (PC3 to SRV2), check `show access-lists`, adjust ACL. Checklist: “No ping? Check OSPF, ACL order, interface, permit any.”
7. **Teach Someone**:
   - Explain ACL logic (sequential, implicit deny), OSPF setup, and why apply close to destination.
8. **Spaced Repetition**:
   - Use Anki for ACL/OSPF commands, lab requirements, and verification. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **OSPF Configuration**:
   - Enable with `router ospf 1`, `network <ip> <mask> area 0` for full connectivity.
   - Verify with `show ip ospf interface`, `show ip ospf neighbor`, `show ip route`.
2. **Standard ACLs**:
   - Filter source IP only (e.g., `deny 172.16.2.0 0.0.0.255`).
   - Numbered: `access-list <1-99>`, named: `ip access-list standard <name>`.
   - Apply outbound close to destination (`ip access-group <name> out`).
3. **Verification**:
   - `show access-lists` shows match counts; `show running-config | section access-list` shows full config.
4. **Testing**:
   - Use `ping`, `ping -t` to confirm ACLs (e.g., PC1 to SRV1 succeeds, PC2 fails).
5. **CCNA Focus**:
   - Topic 5.1: Configure/verify IPv4 standard ACLs.
   - Topic 3.4: Configure/verify OSPFv2 for connectivity.

These notes provide a complete foundation for the Standard ACLs lab, preparing you for CCNA exam topics 5.1 and 3.4. Practice OSPF and ACL configurations, test pings, and use analogies/retention tips to master the material for life!