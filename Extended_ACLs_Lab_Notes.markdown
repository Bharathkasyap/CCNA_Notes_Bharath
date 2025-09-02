# Extended ACLs Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring extended Access Control Lists (ACLs) to enforce network policies, aligning with CCNA exam topic 5.1 ("Configure and verify access control lists"). It builds on the Extended ACLs lecture (Day 35) and Standard ACLs lecture/lab (Day 34), using extended ACLs to filter IPv4 traffic based on protocol, source/destination IP addresses, and port numbers. The lab is performed in Packet Tracer, configuring two numbered extended ACLs on R1 to meet three requirements, with verification via DNS lookups, web access, and pings. It assumes familiarity with IPv6 concepts from prior labs (Part 1: manual addressing, Part 2: EUI-64, static routes, Part 3: SLAAC, advanced static routes) and introduces DNS briefly (covered later in the course).

**Lab Objectives**:
- **Configure Extended ACLs**: Create numbered extended ACLs (`access-list 100`, `101`) to deny specific traffic (DNS, HTTP/HTTPS, all IP) from subnets.
- **Apply ACLs**: Apply ACLs inbound on R1 interfaces, close to the source, per extended ACL guidelines.
- **Verify Configurations**: Use `show access-lists` and test connectivity (DNS lookups, web access, pings) to confirm ACL functionality.
- **Understand DNS Basics**: Observe DNS resolution (e.g., `PING PC2` resolves to `172.16.1.2` via SRV1) to test ACLs.
- **Prepare for Exam**: Master topic 5.1 (extended ACLs) and reinforce related concepts (e.g., DNS, port numbers from Day 30).

**Network Topology** (Similar to Day 34 Standard ACLs):
- **Devices**:
  - **R1**:
    - G0/0: Connected to `172.16.1.0/24` (PC1: `172.16.1.1`, PC2: `172.16.1.2`).
    - G0/1: Connected to `172.16.2.0/24` (PC3: `172.16.2.1`, PC4: `172.16.2.2`).
    - Unspecified interface to R2 (likely serial, e.g., `203.0.113.0/30` from Day 34).
  - **R2**:
    - G0/0: Connected to `192.168.1.0/24` (SRV1: `192.168.1.100`, DNS server).
    - G0/1: Connected to `192.168.2.0/24` (SRV2: `192.168.2.100`, web server).
    - Unspecified interface to R1.
- **Note**: PCs/SRV connect via switches (implied, not shown). Full connectivity assumed (likely via OSPF, as in Day 34).
- **DNS Setup**: SRV1 (`192.168.1.100`) is the DNS server, resolving names (e.g., `PC2` to `172.16.1.2`, `cisco.com` to SRV2’s IP).
- **Requirements**:
  1. Deny `172.16.2.0/24` (PC3, PC4) from communicating with PC1 (`172.16.1.1`).
  2. Deny `172.16.1.0/24` (PC1, PC2) from accessing DNS (port 53, TCP/UDP) on SRV1 (`192.168.1.100`).
  3. Deny `172.16.2.0/24` (PC3, PC4) from accessing HTTP (port 80) and HTTPS (port 443) on SRV2 (`192.168.2.100`).

**Analogy**: Extended ACLs are like a high-tech security checkpoint at an airport (R1 interfaces), checking travelers’ origins (source IPs), destinations (destination IPs), travel modes (protocols like TCP/UDP), and ticket numbers (ports like 53, 80, 443). The checkpoint is placed at the departure gate (source interface, inbound) to stop unwanted travelers early, unlike standard ACLs at the destination gate. DNS is like a travel agent (SRV1) translating names (e.g., `PC2`) to addresses (e.g., `172.16.1.2`).

**Lifelong Retention Tip**: Visualize R1 blocking PC3’s ping to PC1 with ACL 101 on G0/1 inbound. Practice in Packet Tracer: configure `ip access-list extended 100`, add `deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53`, apply `ip access-group 100 in`, test DNS lookup. Create flashcards for commands (`ip access-list extended`, `eq 53`), port numbers (DNS=53, HTTP=80, HTTPS=443), and rules (close-to-source). Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Understand DNS and Test Connectivity
**Objective**: Verify DNS functionality before applying ACLs to understand its role in the lab.

**Actions (on PC1)**:
- **Config**: Set SRV1 (`192.168.1.100`) as DNS server in PC1’s config tab.
- **Test DNS**:
  ```plaintext
  PC1> ping PC2
  Pinging 172.16.1.2 with 32 bytes of data:
  Reply from 172.16.1.2: bytes=32 time<1ms TTL=128
  ```
  - **Observation**: PC1 queries SRV1 to resolve `PC2` to `172.16.1.2`, then pings successfully.
- **Test Web Access (on PC3)**:
  - Open web browser, enter `cisco.com`, resolves to SRV2 (`192.168.2.100`), displays sample webpage.
- **Note**: DNS uses UDP port 53 (primary) and TCP port 53 (e.g., for zone transfers). HTTP (80) and HTTPS (443) use TCP.

**DNS Overview**:
- **Purpose**: Translates names (e.g., `PC2`, `cisco.com`) to IP addresses (e.g., `172.16.1.2`, `192.168.2.100`).
- **Process**: PC sends DNS query to SRV1 (port 53), receives IP, then communicates (e.g., ping, HTTP).
- **Lab Context**: Blocking DNS (Requirement 2) prevents name resolution, affecting pings/web access by name.

**Comparison to Prior Content**:
- **Standard ACLs Lab (Day 34)**: Used OSPF for connectivity, tested with pings (`ping 192.168.1.100`). No DNS involved.
- **Extended ACLs Lecture (Day 35)**: Introduced DNS as a protocol (TCP/UDP port 53) for ACL matching.
- **IPv6 Part 1–3**: No DNS; used IPv6 addresses (`2001:db8::/64`) or static routes, not name resolution.
- **Day 30 (Ports)**: Covered DNS (53), HTTP (80), HTTPS (443), applied here.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Routes (`ip route`) enable connectivity, like OSPF in Day 34; DNS enables name-based access.
- **OSPF**: Likely used here (from Day 34) for full connectivity before ACLs.
- **NAT**: May use extended ACLs to identify traffic (later topic); DNS resolves NAT’d addresses.

**Analogy**: DNS is like a phonebook (SRV1) translating names (PC2) to numbers (172.16.1.2). ACLs block calls to the phonebook (DNS port 53) or specific numbers (PC1, SRV2).

**Lifelong Retention Tip**: In Packet Tracer, ping `PC2` from PC1, note DNS resolution to `172.16.1.2`. Flashcard: “DNS port? 53 (UDP/TCP).” Practice 3 times.

### Step 2: Configure and Apply ACL for Requirement 2
**Objective**: Deny `172.16.1.0/24` (PC1, PC2) from accessing DNS (port 53, TCP/UDP) on SRV1 (`192.168.1.100`), allow other traffic.

**Commands (on R1)**:
```plaintext
R1> enable
R1# configure terminal
R1(config)# ip access-list extended 100
R1(config-ext-nacl)# deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
R1(config-ext-nacl)# deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/0
R1(config-if)# ip access-group 100 in
```

**Observations**:
- **ACL 100**:
  - Denies UDP and TCP port 53 (DNS) from `172.16.1.0/24` to SRV1 (`192.168.1.100`).
  - `permit ip any any` allows all other traffic (e.g., pings, HTTP).
- **Application**: Inbound on G0/0 (source interface for `172.16.1.0/24`), following extended ACL rule (close to source).
- **Why TCP and UDP?**: DNS primarily uses UDP/53; TCP/53 used for large queries (e.g., zone transfers). Blocking both ensures complete DNS denial.

**Verification (on PC1)**:
```plaintext
PC1> ping SRV2
Ping request could not find host SRV2. Please check the name and try again.
PC1> ping 192.168.2.100
Reply from 192.168.2.100: bytes=32 time<1ms TTL=128
```
- **Result**: DNS lookup fails (`PING SRV2` can’t resolve to `192.168.2.100`); direct IP ping succeeds (not blocked by ACL).
- **Note**: First ping may fail due to ARP delay in Packet Tracer.

**Comparison to Prior Content**:
- **Standard ACLs Lab (Day 34)**: Denied source IPs (e.g., `172.16.2.0/24`) on R2 outbound; here, extended ACLs deny specific protocols/ports on R1 inbound.
- **Extended ACLs Lecture (Day 35)**: Configured `deny tcp ... eq 443` for HTTPS; here, `deny udp ... eq 53` for DNS.
- **IPv6 Part 3**: Used `ping 2001:db8:0:3::1` to test connectivity; no DNS or ACLs.
- **Day 30**: DNS port 53 identified, applied here.

**Comparison to Other CCNA Topics**:
- **Day 10 (IPv4 Header)**: Protocol field (UDP=17, TCP=6) matches ACL protocol options.
- **OSPF (Day 34)**: Enabled connectivity; ACLs here filter specific services (DNS).
- **DNS (Later Topic)**: Lab introduces DNS resolution, critical for name-based access.

**Analogy**: ACL 100 is like a guard at the airport gate (G0/0 in) blocking travelers from one city (`172.16.1.0/24`) asking the travel agent (SRV1, DNS) for directions (port 53), but allowing other trips (e.g., direct flights to SRV2).

**Lifelong Retention Tip**: In Packet Tracer, configure ACL 100, test `PING SRV2` from PC1 (should fail), ping `192.168.2.100` (should succeed). Flashcard: “DNS ACL? `deny udp ... eq 53`, `deny tcp ... eq 53`.” Practice 3 times.

### Step 3: Configure and Apply ACL for Requirements 1 and 3
**Objective**: Deny `172.16.2.0/24` (PC3, PC4) from communicating with PC1 (`172.16.1.1`) and accessing HTTP/HTTPS on SRV2 (`192.168.2.100`), allow other traffic.

**Commands (on R1)**:
```plaintext
R1(config)# ip access-list extended 101
R1(config-ext-nacl)# deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/1
R1(config-if)# ip access-group 101 in
```

**Observations**:
- **ACL 101**:
  - **Requirement 1**: `deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1` blocks all traffic from `172.16.2.0/24` to PC1.
  - **Requirement 3**: `deny tcp ... eq 80` and `eq 443` block HTTP/HTTPS from `172.16.2.0/24` to SRV2.
  - `permit ip any any` allows other traffic (e.g., pings to PC2, DNS to SRV1).
- **Application**: Inbound on G0/1 (source interface for `172.16.2.0/24`), close to source.
- **Efficiency**: Combines both requirements in one ACL, as both source from `172.16.2.0/24`.

**Verification (on PC3)**:
- **Web Access**:
  - Before ACL: Open browser, enter `cisco.com`, resolves to `192.168.2.100` (via SRV1 DNS), displays webpage.
  - After ACL: Reopen browser, enter `cisco.com`, times out (HTTP/HTTPS blocked to SRV2).
- **Ping Tests**:
  ```plaintext
  PC3> ping PC1
  Ping request could not find host PC1. Please check the name and try again.
  PC3> ping 172.16.1.1
  Destination host unreachable
  PC3> ping PC2
  Pinging 172.16.1.2 with 32 bytes of data:
  Reply from 172.16.1.2: bytes=32 time<1ms TTL=128
  ```
  - **Result**: Ping to PC1 fails (blocked by `deny ip ... host 172.16.1.1`); ping to PC2 succeeds (DNS allowed, no ACL restriction).

**Comparison to Prior Content**:
- **Standard ACLs Lab (Day 34)**: Denied `172.16.2.0/24` to `192.168.2.0/24` on R2 G0/1 out; here, extended ACL denies specific protocols (HTTP/HTTPS) on R1 G0/1 in.
- **Extended ACLs Lecture (Day 35)**: Configured `deny tcp ... eq 443` to SRV1; here, similar for SRV2, plus `deny ip` for PC1.
- **IPv6 Part 3**: Tested connectivity with `ping 2001:db8:0:3::1`; no protocol-specific filtering.
- **Day 30**: HTTP (80), HTTPS (443) ports used here for ACL matching.

**Comparison to Other CCNA Topics**:
- **Day 10 (IPv4 Header)**: Protocol field (IP=0, TCP=6) matches `deny ip`, `deny tcp`.
- **OSPF (Day 34)**: Enabled connectivity; extended ACLs filter specific services (DNS, HTTP).
- **DNS (Later Topic)**: Lab tests DNS resolution, blocked by ACL 100.

**Analogy**: ACL 101 is like a guard at the gate (G0/1 in) blocking travelers from one city (`172.16.2.0/24`) to a VIP (PC1, all protocols) and specific flights (HTTP/80, HTTPS/443 to SRV2), but allowing other trips (e.g., to PC2).

**Lifelong Retention Tip**: In Packet Tracer, configure ACL 101, test `cisco.com` from PC3 (should fail), ping PC2 (should succeed). Flashcard: “HTTP/HTTPS ACL? `deny tcp ... eq 80`, `eq 443`.” Practice 3 times.

### Step 4: Verify ACLs
**Objective**: Confirm ACLs enforce requirements using `show access-lists` and connectivity tests.

**Commands (on R1)**:
```plaintext
R1# show access-lists
Extended IP access list 100
    10 deny udp 172.16.1.0, wildcard bits 0.0.0.255 host 192.168.1.100 eq domain (X matches)
    20 deny tcp 172.16.1.0, wildcard bits 0.0.0.255 host 192.168.1.100 eq domain (Y matches)
    30 permit ip any any
Extended IP access list 101
    10 deny ip 172.16.2.0, wildcard bits 0.0.0.255 host 172.16.1.1 (Z matches)
    20 deny tcp 172.16.2.0, wildcard bits 0.0.0.255 host 192.168.2.100 eq www (W matches)
    30 deny tcp 172.16.2.0, wildcard bits 0.0.0.255 host 192.168.2.100 eq 443 (V matches)
    40 permit ip any any
```

**Observations**:
- **Match Counts**: `X`, `Y`, `Z`, `W`, `V` indicate packets blocked (e.g., DNS queries, HTTP requests, pings to PC1).
- **Port Display**: `eq domain` (port 53), `eq www` (port 80) shown in IOS output, but configured as `eq 53`, `eq 80`.
- **Verification Commands**:
  - `show access-lists`: Shows ACL entries and matches.
  - `show ip interface g0/0`: Confirms `access-group 100 in`.
  - `show running-config | section access-list`: Displays full ACL config.

**Comparison to Prior Content**:
- **Standard ACLs Lab (Day 34)**: Used `show access-lists` for standard ACLs (e.g., `deny 172.16.2.0 0.0.0.255`); here, extended ACLs show protocol/port matches.
- **Extended ACLs Lecture (Day 35)**: Used `show ip interface` to verify ACL application; same here.
- **IPv6 Part 3**: Used `ping`, `tracert` to verify connectivity; here, pings and web access test ACLs.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DNS (53), HTTP (80), HTTPS (443) verified in ACL matches.
- **OSPF (Day 34)**: Connectivity assumed; ACLs filter specific traffic.
- **DNS**: Lab tests DNS resolution failures due to ACL 100.

**Analogy**: Verifying ACLs is like checking a guard’s logbook (`show access-lists`) to see which travelers were stopped (matches) and testing routes (pings, web access) to confirm only allowed paths work.

**Lifelong Retention Tip**: In Packet Tracer, run `show access-lists` after pinging PC1 from PC3 (should increment `deny ip` matches). Flashcard: “Verify ACL? `show access-lists` for matches.” Practice 3 times.

## Common Issues and Troubleshooting

- **DNS Still Accessible**:
  - **Issue**: PC1 resolves `SRV2` to `192.168.2.100`.
  - **Cause**: ACL 100 not applied, wrong interface (e.g., G0/1), or missing `deny tcp ... eq 53`.
  - **Fix**: Verify with `show ip interface g0/0`, ensure `ip access-group 100 in`, add TCP/UDP entries.
- **HTTP/HTTPS Not Blocked**:
  - **Issue**: PC3 accesses `cisco.com` (SRV2).
  - **Cause**: Wrong ports (e.g., `eq 8080`), wrong interface (G0/0), or outbound instead of inbound.
  - **Fix**: Check `show access-lists`, correct to `eq 80`, `eq 443`, apply on G0/1 in.
- **All Traffic Blocked**:
  - **Issue**: PC3 can’t ping PC2.
  - **Cause**: Missing `permit ip any any` in ACL 101.
  - **Fix**: Add `permit ip any any`, reapply ACL, verify with `show running-config | section access-list`.
- **Ping to PC1 Succeeds**:
  - **Issue**: PC3 pings PC1.
  - **Cause**: ACL 101 misconfigured (e.g., wrong destination IP).
  - **Fix**: Ensure `deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1`, check `show access-lists`.

**Analogy**: Troubleshooting is like fixing a checkpoint’s rulebook: ensure the guard blocks the right flights (DNS, HTTP), stands at the correct gate (G0/0, G0/1 in), and allows other travelers (`permit ip any any`).

**Lifelong Retention Tip**: Simulate failed DNS lookup in Packet Tracer, check `show access-lists`, adjust ACL 100. Checklist: “No block? Check protocol, port, interface, direction, permit any.” Practice 3 times.

## Practice Commands

- **ACL 100 (Deny DNS)**:
  ```plaintext
  R1(config)# ip access-list extended 100
  R1(config-ext-nacl)# deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
  R1(config-ext-nacl)# deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
  R1(config-ext-nacl)# permit ip any any
  R1(config-ext-nacl)# exit
  R1(config)# interface g0/0
  R1(config-if)# ip access-group 100 in
  ```
- **ACL 101 (Deny PC1, HTTP/HTTPS)**:
  ```plaintext
  R1(config)# ip access-list extended 101
  R1(config-ext-nacl)# deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
  R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
  R1(config-ext-nacl)# deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
  R1(config-ext-nacl)# permit ip any any
  R1(config-ext-nacl)# exit
  R1(config)# interface g0/1
  R1(config-if)# ip access-group 101 in
  ```
- **Verify**:
  ```plaintext
  show access-lists
  show ip interface g0/0
  show running-config | section access-list
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Extended ACL: “Protocol, source/destination IP, ports, close to source, inbound.”
   - Ports: “DNS=53 (UDP/TCP), HTTP=80, HTTPS=443 (TCP).”
   - Application: “Extended ACLs: inbound, source interface.”
2. **Visualize Topology**:
   - Draw R1 with ACL 100 on G0/0 in (DNS block), ACL 101 on G0/1 in (PC1, HTTP/HTTPS block). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Extended ACL range? 100–199, 2000–2699. DNS port? 53. Apply where? Inbound, source.”
4. **Practice in Packet Tracer**:
   - Configure ACL 100, test `PING SRV2` from PC1 (fails), ping `192.168.2.100` (succeeds). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Standard ACLs (source IP, destination) vs. Extended ACLs (protocol/ports, source).
   - Quiz: “Block DNS? `deny udp ... eq 53`, `deny tcp ... eq 53`.”
6. **Troubleshooting Practice**:
   - Simulate failed HTTP access, check `show access-lists`, adjust ACL 101. Checklist: “No block? Check protocol, port, direction, permit any.”
7. **Teach Someone**:
   - Explain extended ACL syntax, DNS blocking, and close-to-source rule.
8. **Spaced Repetition**:
   - Use Anki for ACL commands, port numbers, and verification. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Extended ACL Configuration**:
   - Numbered: `ip access-list extended 100`, deny TCP/UDP (DNS=53), HTTP (80), HTTPS (443), or all IP.
   - Apply inbound, close to source (e.g., G0/0 for `172.16.1.0/24`).
2. **DNS Basics**:
   - Resolves names to IPs (e.g., `PC2` to `172.16.1.2`) via SRV1 (port 53, UDP/TCP).
   - Blocking DNS prevents name-based access (e.g., `PING SRV2`).
3. **Verification**:
   - `show access-lists` shows match counts (e.g., DNS queries blocked).
   - Test with DNS lookups (`PING PC1`), web access (`cisco.com`), pings.
4. **Application**:
   - ACL 100 on G0/0 in: Deny DNS from `172.16.1.0/24`.
   - ACL 101 on G0/1 in: Deny all to PC1, HTTP/HTTPS to SRV2 from `172.16.2.0/24`.
5. **CCNA Focus**:
   - Topic 5.1: Configure/verify IPv4 extended ACLs, know protocol (IP, TCP, UDP, ICMP) and ports (53, 80, 443).

These notes provide a complete foundation for the Extended ACLs lab, preparing you for CCNA exam topic 5.1. Practice configurations, troubleshoot access, and use analogies/retention tips to master extended ACLs for life!