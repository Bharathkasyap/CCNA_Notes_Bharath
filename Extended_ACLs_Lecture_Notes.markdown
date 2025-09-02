# Extended ACLs Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture (Day 35) covers extended Access Control Lists (ACLs) for the CCNA, aligning with exam topic 5.1 ("Configure and verify access control lists"). Extended ACLs filter IPv4 traffic based on protocol, source/destination IP addresses, and port numbers, offering more precision than standard ACLs (Day 34). The lecture builds on the Standard ACLs lecture/lab, using the same network topology, and introduces advanced configuration methods, including numbered ACLs in named ACL config mode, editing ACLs, and resequencing. It assumes familiarity with IPv6 concepts from prior labs (Part 1: manual addressing, Part 2: EUI-64, static routes, Part 3: SLAAC, advanced static routes).

**Lecture Objectives**:
- **Understand Extended ACLs**: Learn how extended ACLs differ from standard ACLs by matching protocol, source/destination IPs, and ports.
- **Configure Extended ACLs**: Master commands for numbered (`access-list 100-199`) and named (`ip access-list extended`) ACLs.
- **Edit ACLs**: Delete entries, insert new ones with sequence numbers, and resequence ACLs.
- **Apply ACLs**: Apply extended ACLs close to the source (inbound), unlike standard ACLs (close to destination, outbound).
- **Verify ACLs**: Use `show access-lists`, `show ip interface`, and `show running-config` to confirm configurations.
- **Prepare for Exam**: Master topic 5.1, focusing on IPv4 extended ACLs (IPv6 ACLs not required for CCNA).

**Network Topology** (Same as Day 34 Standard ACLs):
- **Devices**:
  - **R1**:
    - G0/1: Connected to `192.168.1.0/24` (PC1: `192.168.1.1`, PC2: `192.168.1.2`).
    - G0/2: Connected to `192.168.2.0/24` (PC3: `192.168.2.1`, PC4: `192.168.2.2`).
    - G0/0: Point-to-point to R2 (`10.0.0.0/30`, not detailed in transcript).
  - **R2**:
    - G0/0: Point-to-point to R1.
    - G0/1: Connected to `10.0.1.0/24` (SRV1: `10.0.1.100`).
    - G0/2: Connected to `10.0.2.0/24` (SRV2: `10.0.2.100`).
- **Note**: PCs connect via switches (implied, not shown).
- **Requirements (Examples)**:
  1. Deny `192.168.1.0/24` from using HTTPS to SRV1 (`10.0.1.100`).
  2. Deny `192.168.2.0/24` from accessing `10.0.2.0/24`.
  3. Deny `192.168.1.0/24` and `192.168.2.0/24` from pinging `10.0.1.0/24` or `10.0.2.0/24`.

**Analogy**: Extended ACLs are like a high-tech security guard at a building entrance (interface), checking not just IDs (source IPs, like standard ACLs) but also tickets (destination IPs), travel modes (protocols like TCP/UDP), and specific seats (port numbers). The guard checks rules in order (sequential processing), allows/denies entry, and denies unlisted guests (implicit deny). Unlike standard ACLs (placed near the destination), extended ACLs are placed near the source to stop unwanted traffic early.

**Lifelong Retention Tip**: Visualize R1 blocking PC1’s HTTPS to SRV1 with an extended ACL on G0/1 inbound. Practice in Packet Tracer: configure `ip access-list extended NO_HTTPS`, add `deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443`, apply `ip access-group NO_HTTPS in`, verify with `show access-lists`. Create flashcards for commands (`access-list 100`, `ip access-list extended`) and rules (close-to-source, implicit deny). Practice 3 times daily for a week.

## Extended ACLs Overview

**Purpose**:
- **Definition**: Extended ACLs filter IPv4 traffic based on protocol (e.g., TCP, UDP, ICMP), source/destination IP addresses, and port numbers (e.g., 80 for HTTP, 443 for HTTPS), offering precise access control for security.
- **Comparison to Standard ACLs (Day 34)**:
  - Standard ACLs: Match source IP only (e.g., `permit 192.168.1.1`).
  - Extended ACLs: Match protocol, source/destination IPs, ports (e.g., `permit tcp host 192.168.1.1 host 10.0.1.100 eq 443`).
- **Use**: Security (e.g., deny HTTPS to SRV1), QoS, NAT (CCNA focuses on security, topic 5.1).
- **Operation**: Routers process ACLs sequentially, applying the first matching entry (permit/deny), with an implicit `deny ip any any` at the end.

**Logic** (Same as Standard ACLs):
- **Access Control Entries (ACEs)**: Ordered rules (e.g., `deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443`).
- **Sequential Processing**: Top-to-bottom, first match determines action.
- **Implicit Deny**: Unmatched packets are dropped (`deny ip any any`).
- **Interface Application**:
  - Inbound: Filters packets entering an interface (e.g., PC1’s HTTPS entering R1 G0/1).
  - Outbound: Filters packets exiting an interface (e.g., PC1’s HTTPS exiting R1 G0/0).
  - Limit: One ACL per interface per direction.
- **Rule-of-Thumb**: Apply extended ACLs close to the source (inbound) to drop unwanted traffic early, minimizing router resource use. Standard ACLs are applied close to the destination (outbound) due to their less specific matching.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1 Lecture/Lab**: No ACLs, focused on IPv6 addressing (`2001:db8::/64`) and `ipv6 unicast-routing`.
- **IPv6 Part 2 Lecture/Lab**: Static routes (`ipv6 route`) controlled paths, not access. Extended ACLs filter specific protocols/ports, unlike routes.
- **IPv6 Part 3 Lecture/Lab**: SLAAC and static routes managed connectivity; extended ACLs manage security with precise filtering (e.g., TCP port 80).
- **Difference**: IPv6 ACLs (not CCNA-required) use `ipv6 access-list`, similar logic but different syntax.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Routes direct traffic (e.g., `ip route 10.0.1.0 255.255.255.0 g0/0`); extended ACLs filter it (e.g., `deny tcp any host 10.0.1.100 eq 80`).
- **OSPF/EIGRP**: Use wildcard masks (e.g., `network 192.168.1.0 0.0.0.255`), like ACLs. Dynamic routing doesn’t filter traffic.
- **Standard ACLs (Day 34)**: Simpler, source IP only, applied close to destination. Extended ACLs are more precise, applied close to source.

**Analogy**: Extended ACLs are like a firewall checking passports (source IPs), destinations (destination IPs), travel types (TCP/UDP), and seat numbers (ports). Standard ACLs only check passports and are placed near the destination gate.

**Lifelong Retention Tip**: Draw R1 with an ACL on G0/1 inbound denying HTTPS from PC1. Flashcard: “Extended ACLs match? Protocol, source/destination IP, ports.” Practice in Packet Tracer: deny ICMP from PC3, verify with `show access-lists`. Repeat 3 times.

## Configuring Numbered ACLs in Named ACL Config Mode

**Purpose**: Configure numbered ACLs (standard or extended) using named ACL config mode for easier editing (delete/insert entries, specify sequence numbers).

**Commands**:
```plaintext
R1(config)# ip access-list {standard | extended} <1-99,100-199,1300-1999,2000-2699>
R1(config-std-nacl)# [<sequence>] {permit | deny} <parameters>
R1(config-std-nacl)# no <sequence>
R1(config-if)# ip access-group <number> {in | out}
```

**Example (Standard ACL, Deny `192.168.1.1`, Permit Others)**:
```plaintext
R1(config)# ip access-list standard 1
R1(config-std-nacl)# deny 192.168.1.1
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit
R1(config)# interface g0/1
R1(config-if)# ip access-group 1 out
```

**Observations**:
- **Syntax**: `ip access-list standard 1` enters named ACL config mode for numbered ACL 1.
- **Running Config**: Displays as traditional numbered ACL (`access-list 1 deny 192.168.1.1`).
- **Advantages**:
  - Delete entries: `no <sequence>` (e.g., `no 10` removes entry 10).
  - Insert entries: Specify sequence (e.g., `20 deny 192.168.2.0 0.0.0.255`).
- **Traditional Method**: `access-list 1 deny 192.168.1.1` (global config mode) doesn’t allow deleting individual entries; `no access-list 1` deletes entire ACL.

**Example (Delete/Insert Entry)**:
```plaintext
R1# show access-lists
Standard IP access list 1
    10 permit 192.168.1.0, wildcard bits 0.0.0.255
    20 deny 192.168.2.0, wildcard bits 0.0.0.255
    30 deny 192.168.3.0, wildcard bits 0.0.0.255
    40 permit any
R1(config)# ip access-list standard 1
R1(config-std-nacl)# no 30
R1(config-std-nacl)# 25 deny 192.168.2.0 0.0.0.255
R1(config-std-nacl)# exit
R1# show access-lists
Standard IP access list 1
    10 permit 192.168.1.0, wildcard bits 0.0.0.255
    20 deny 192.168.2.0, wildcard bits 0.0.0.255
    25 deny 192.168.2.0, wildcard bits 0.0.0.255
    40 permit any
```

**Comparison to Standard ACLs (Day 34)**:
- **Standard ACL Lecture/Lab**: Configured numbered ACLs in global config mode (`access-list 1`), named ACLs in config mode (`ip access-list standard`). This method extends to numbered ACLs.
- **Lab Topology**: Used `172.16.x.0/24` vs. `192.168.x.0/24` here, but same logic (sequential, implicit deny).

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1–3**: No ACL editing; static routes (`ipv6 route`) used fixed paths, not editable like ACLs.
- **Part 3 Lab**: Static routes were replaced entirely if changed, similar to `no access-list 1` deleting entire ACL.

**Comparison to Other CCNA Topics**:
- **OSPF/EIGRP**: Configurations are fixed; no dynamic editing like ACLs.
- **VLANs**: VLAN configs (`vlan 10`) are static, unlike ACLs’ flexible editing.

**Analogy**: Configuring numbered ACLs in named mode is like editing a numbered guest list (ACL 1) in a notebook (config mode), where you can erase (delete) or insert lines (entries) with specific numbers (sequences).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip access-list standard 1`, add `deny 192.168.1.1`, delete with `no 10`, insert `15 permit 192.168.1.2`. Flashcard: “Delete ACL entry? `no <sequence>` in named mode.” Practice 3 times.

## Resequencing ACLs

**Purpose**: Reassign sequence numbers to ACL entries to allow inserting new entries when sequence numbers are too close (e.g., 1, 2, 3).

**Command**:
```plaintext
R1(config)# ip access-list resequence <number|name> <start> <increment>
```

**Example**:
```plaintext
R1# show access-lists
Standard IP access list 1
    1 deny 192.168.1.1
    2 deny 192.168.3.1
    3 deny 192.168.2.1
    4 deny 192.168.4.1
    5 permit any
R1(config)# ip access-list resequence 1 10 10
R1# show access-lists
Standard IP access list 1
    10 deny 192.168.1.1
    20 deny 192.168.3.1
    30 deny 192.168.2.1
    40 deny 192.168.4.1
    50 permit any
```

**Observations**:
- **Syntax**: `resequence 1 10 10` sets first entry to 10, increments by 10 (20, 30, 40, 50).
- **Use**: Fixes tight sequence numbers (e.g., 1, 2, 3) to allow insertions (e.g., `15 deny 192.168.5.0 0.0.0.255`).
- **Scope**: Works for numbered/named, standard/extended ACLs, from global config mode.
- **Order**: Preserves entry order, only changes sequence numbers.
- **Note**: IOS may reorder `/32` entries (e.g., `192.168.1.1`) for efficiency, but effect is unchanged.

**Comparison to Standard ACLs (Day 34)**:
- **Lecture/Lab**: Didn’t cover resequencing; focused on default sequence numbers (10, 20, 30).
- **Lab**: Used numbered ACLs (`access-list 1`) without editing; resequencing enhances flexibility.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 3**: Static routes were fixed; no resequencing equivalent. ACL resequencing is unique to ACL editing.
- **Part 1–2**: No dynamic configuration changes, unlike ACL resequencing.

**Comparison to Other CCNA Topics**:
- **OSPF/EIGRP**: No equivalent to resequencing; routing tables are dynamically updated.
- **VLANs**: VLAN IDs are fixed, not resequenced.

**Analogy**: Resequencing is like renumbering a guest list (1, 2, 3 to 10, 20, 30) to leave gaps for new names (entries), making it easier to add guests without rewriting the list.

**Lifelong Retention Tip**: In Packet Tracer, configure ACL 1 with tight sequences (1, 2, 3), resequence with `ip access-list resequence 1 10 10`, verify with `show access-lists`. Flashcard: “Resequence? `ip access-list resequence <name> <start> <increment>`.” Practice 3 times.

## Configuring Extended ACLs

**Numbered ACL Ranges**:
- **Standard ACLs**: 1–99, 1300–1999 (Day 34).
- **Extended ACLs**: 100–199, 2000–2699.
- **Memorize**: Essential for CCNA exam (topic 5.1).

**Commands (Numbered, Global Config Mode)**:
```plaintext
R1(config)# access-list <100-199,2000-2699> {permit | deny} <protocol> <source-ip> <source-mask> <dest-ip> <dest-mask> [eq <port>] [options]
R1(config-if)# ip access-group <100-199,2000-2699> {in | out}
```

**Commands (Named or Numbered in Named Config Mode)**:
```plaintext
R1(config)# ip access-list extended <name|100-199,2000-2699>
R1(config-ext-nacl)# [<sequence>] {permit | deny} <protocol> <source-ip> <source-mask> [eq <port>] <dest-ip> <dest-mask> [eq <port>] [options]
R1(config-if)# ip access-group <name|number> {in | out}
```

**Parameters**:
- **Protocol**: IP (all packets), TCP, UDP, ICMP, OSPF (89), EIGRP (88), or protocol number (e.g., 6 for TCP, 17 for UDP).
- **Source/Destination IP**: `any`, `host <ip>`, or `<ip> <wildcard-mask>` (e.g., `192.168.1.0 0.0.0.255` for `/24`).
- **Port Options** (for TCP/UDP):
  - `eq <port>`: Equal to port (e.g., `eq 80` for HTTP).
  - `gt <port>`: Greater than port (e.g., `gt 9999`).
  - `lt <port>`: Less than port.
  - `neq <port>`: Not equal to port.
  - `range <start> <end>`: Range of ports (e.g., `range 20000 30000`).
- **Other Options** (not CCNA-required): `ack`, `fin`, `syn` (TCP flags), `ttl`, `dscp`.

**Example (Named, Deny HTTPS from `192.168.1.0/24` to SRV1)**:
```plaintext
R1(config)# ip access-list extended NO_HTTPS
R1(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/1
R1(config-if)# ip access-group NO_HTTPS in
```

**Observations**:
- **Protocol Matching**: `ip` matches all IP packets; `tcp`, `udp`, `icmp` match specific protocols.
- **Port Matching**: Optional for TCP/UDP (e.g., `eq 443` for HTTPS); omitted for all ports.
- **Source/Destination**: Requires `host` or wildcard mask for `/32` (e.g., `host 10.0.1.100` or `10.0.1.100 0.0.0.0`).
- **Application**: Inbound on source interface (e.g., G0/1 for `192.168.1.0/24`) to drop traffic early.
- **Match All Parameters**: Packet must match protocol, source IP, destination IP, and ports (if specified).

**Practice Entries**:
1. **Permit all traffic**:
   ```plaintext
   permit ip any any
   ```
2. **Deny UDP from `10.0.0.0/16` to `192.168.1.1/32`**:
   ```plaintext
   deny udp 10.0.0.0 0.0.255.255 host 192.168.1.1
   ```
3. **Deny ICMP (ping) from `172.16.1.1/32` to `192.168.0.0/24`**:
   ```plaintext
   deny icmp host 172.16.1.1 192.168.0.0 0.0.0.255
   ```
4. **Permit HTTPS from `10.0.0.0/16` to `2.2.2.2/32`**:
   ```plaintext
   permit tcp 10.0.0.0 0.0.255.255 host 2.2.2.2 eq 443
   ```
5. **Deny UDP source ports 20000–30000 to `3.3.3.3/32`**:
   ```plaintext
   deny udp any range 20000 30000 host 3.3.3.3
   ```
6. **Permit TCP source ports >9999 from `172.16.1.0/24` to `4.4.4.4/32`, except port 23**:
   ```plaintext
   permit tcp 172.16.1.0 0.0.0.255 gt 9999 host 4.4.4.4 neq 23
   ```

**Comparison to Standard ACLs (Day 34)**:
- **Standard ACLs**: Source IP only, applied close to destination (e.g., R2 G0/1 out).
- **Extended ACLs**: Protocol, source/destination IPs, ports, applied close to source (e.g., R1 G0/1 in).
- **Lab**: Used standard ACLs for simple filtering (e.g., `deny 172.16.2.0 0.0.0.255`); extended ACLs here add protocol/port specificity.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1–3**: No ACLs; focused on addressing and static routes. Extended ACLs resemble IPv6 ACLs (not CCNA-required) in matching protocols/ports.
- **Part 3 Lab**: Static routes (`ipv6 route 2001:db8:0:3::/64 g0/1`) controlled paths; extended ACLs control access with precision.

**Comparison to Other CCNA Topics**:
- **Day 10 (IPv4 Header)**: Protocol field (e.g., 6 for TCP, 17 for UDP) matches extended ACL protocol options.
- **Day 30 (Ports)**: TCP/UDP port numbers (e.g., 80 for HTTP, 443 for HTTPS) used in extended ACLs.
- **OSPF/EIGRP**: Protocol numbers (89 for OSPF, 88 for EIGRP) can be matched in extended ACLs.

**Analogy**: Extended ACLs are like a detailed security checkpoint checking travel documents (protocol), departure city (source IP), destination city (destination IP), and flight number (port). Apply near the departure gate (source interface) to stop invalid travelers early.

**Lifelong Retention Tip**: In Packet Tracer, configure `ip access-list extended NO_PING`, add `deny icmp 192.168.1.0 0.0.0.255 any`, apply on G0/1 in. Flashcard: “Extended ACL protocol? IP, TCP, UDP, ICMP, OSPF (89).” Practice 3 times.

## Example Configurations

**Requirement 1 (Deny HTTPS from `192.168.1.0/24` to SRV1)**:
```plaintext
R1(config)# ip access-list extended NO_HTTPS
R1(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/1
R1(config-if)# ip access-group NO_HTTPS in
```

**Requirement 2 (Deny `192.168.2.0/24` from `10.0.2.0/24`)**:
```plaintext
R1(config)# ip access-list extended NO_10.0.2.0
R1(config-ext-nacl)# deny ip 192.168.2.0 0.0.0.255 10.0.2.0 0.0.0.255
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/2
R1(config-if)# ip access-group NO_10.0.2.0 in
```

**Requirement 3 (Deny `192.168.1.0/24` and `192.168.2.0/24` from pinging `10.0.1.0/24` or `10.0.2.0/24`)**:
```plaintext
R1(config)# ip access-list extended NO_PING
R1(config-ext-nacl)# deny icmp 192.168.1.0 0.0.0.255 10.0.1.0 0.0.0.255
R1(config-ext-nacl)# deny icmp 192.168.1.0 0.0.0.255 10.0.2.0 0.0.0.255
R1(config-ext-nacl)# deny icmp 192.168.2.0 0.0.0.255 10.0.1.0 0.0.0.255
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
R1(config)# interface g0/0
R1(config-if)# ip access-group NO_PING out
```

**Observations**:
- **Efficiency Note**: Requirement 3 omits `192.168.2.0/24` to `10.0.2.0/24` (covered by Requirement 2), reducing redundancy.
- **Application**: Inbound on G0/1, G0/2 for Requirements 1–2 (source-based); outbound on G0/0 for Requirement 3 (covers both sources to both destinations).
- **Verification**:
  ```plaintext
  R1# show access-lists
  Extended IP access list NO_HTTPS
      10 deny tcp 192.168.1.0, wildcard bits 0.0.0.255 host 10.0.1.100 eq 443
      20 permit ip any any
  Extended IP access list NO_10.0.2.0
      10 deny ip 192.168.2.0, wildcard bits 0.0.0.255 10.0.2.0, wildcard bits 0.0.0.255
      20 permit ip any any
  Extended IP access list NO_PING
      10 deny icmp 192.168.1.0, wildcard bits 0.0.0.255 10.0.1.0, wildcard bits 0.0.0.255
      20 deny icmp 192.168.1.0, wildcard bits 0.0.0.255 10.0.2.0, wildcard bits 0.0.0.255
      30 deny icmp 192.168.2.0, wildcard bits 0.0.0.255 10.0.1.0, wildcard bits 0.0.0.255
      40 permit ip any any
  R1# show ip interface g0/1
  GigabitEthernet0/1 is up, line protocol is up
    Inbound access list is NO_HTTPS
  ```

**Comparison to Standard ACLs (Day 34)**:
- **Standard ACL Lab**: Configured `deny 172.16.2.0 0.0.0.255` on R2 G0/1 out; extended ACLs here use `deny tcp` or `deny icmp` on R1 inbound.
- **Lecture**: Standard ACLs lacked protocol/port matching; extended ACLs add specificity (e.g., `eq 443`).

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 3**: Static routes (`ipv6 route`) directed traffic; extended ACLs filter specific traffic (e.g., ICMP).
- **Part 1–2**: No filtering, only addressing/routing; extended ACLs add protocol/port granularity.

**Comparison to Other CCNA Topics**:
- **Day 10 (IPv4 Header)**: Protocol field (e.g., ICMP=1, TCP=6) used in extended ACLs.
- **Day 30 (Ports)**: Port numbers (e.g., 80, 443) critical for extended ACLs.
- **NAT**: Uses extended ACLs to identify traffic for translation (later topic).

**Analogy**: Configuring extended ACLs is like setting up a detailed checkpoint list for specific flights (TCP/UDP), departure cities (source IPs), destination cities (destination IPs), and seat numbers (ports), applied at the departure gate (source interface).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip access-list extended NO_HTTPS`, test HTTPS from PC1 (should fail), verify with `show ip interface g0/1`. Flashcard: “Extended ACL apply? Close to source, inbound.” Practice 3 times.

## Quiz Answers and Explanations

1. **Which ACL permits only PC1 to access TFTP on SRV1, applied outbound on R1 G0/0?**
   - **Answer**: ACL 103 (`permit udp host 192.168.1.1 host 10.0.1.100 eq tftp`, `deny udp any host 10.0.1.100 eq tftp`, `permit ip any any`).
   - **Explanation**: Permits PC1 (`192.168.1.1`) to SRV1 (`10.0.1.100`) on UDP port 69 (TFTP), denies others, allows remaining traffic. ACL 102 incorrectly uses source port.

2. **Effect of `no access-list 1 deny 10.0.2.0 0.0.0.255` on ACL 1?**
   - **Answer**: B (ACL 1 deleted).
   - **Explanation**: In global config mode, `no access-list 1` deletes the entire ACL, not just the entry. Use named config mode (`ip access-list standard 1`, `no 10`) to delete individual entries.

3. **Which command resequenced ACL 199 from (1, 2, 3, 4, 5) to (5, 15, 25, 35, 45)?**
   - **Answer**: C (`ip access-list resequence 199 5 10`).
   - **Explanation**: Sets first entry to 5, increments by 10 (15, 25, 35, 45).

4. **Which ACL prevents R1 from forwarding OSPF packets out G0/2?**
   - **Answer**: ACL 112 (`deny 89 any any`, `permit ip any any`).
   - **Explanation**: Denies protocol 89 (OSPF) outbound on G0/2, allows other traffic. Protocol 88 (EIGRP) is incorrect.

5. **Fix ACL 150 to deny HTTP/HTTPS from `192.168.1.0/24` to `10.0.2.0/24`, allow others (select two)**:
   - **Answer**: C (apply inbound on G0/1), E (use TCP, not UDP).
   - **Explanation**:
     - C: Inbound on G0/1 (source interface) is correct for extended ACLs, not outbound.
     - E: HTTP (80), HTTPS (443) use TCP, not UDP.

**Bonus Question (Boson ExSim)**: Likely tests extended ACL configuration, application (close to source), or protocol/port matching (e.g., “Deny TCP 80 from PC1 to SRV2”).

**Analogy**: The quiz is like testing a security guard’s checklist: ensuring correct rules (entries), application (interface/direction), and protocol/port accuracy.

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Extended ACL ranges? 100–199, 2000–2699”). Practice scenarios in Packet Tracer (e.g., deny ICMP from PC1), verify with `show access-lists`. Repeat quiz 3 times.

## Common Issues and Troubleshooting

- **Traffic Not Blocked**:
  - **Issue**: PC1 can access HTTPS on SRV1.
  - **Cause**: Wrong interface (e.g., G0/0 out) or direction (outbound vs. inbound).
  - **Fix**: Apply `ip access-group NO_HTTPS in` on G0/1, verify with `show ip interface g0/1`.
- **All Traffic Blocked**:
  - **Issue**: No traffic reaches SRV1.
  - **Cause**: Missing `permit ip any any` (implicit deny blocks all).
  - **Fix**: Add `permit ip any any`, reapply ACL, check `show access-lists`.
- **Wrong Protocol/Port**:
  - **Issue**: HTTPS not blocked.
  - **Cause**: Used UDP or wrong port (e.g., `eq 80` instead of `eq 443`).
  - **Fix**: Correct to `deny tcp ... eq 443`, verify with `show running-config | section access-list`.
- **Cannot Delete Entry**:
  - **Issue**: `no access-list 100 deny ...` deletes entire ACL.
  - **Cause**: Used global config mode.
  - **Fix**: Use `ip access-list extended 100`, `no <sequence>`, check `show access-lists`.

**Analogy**: Troubleshooting extended ACLs is like fixing a high-tech checkpoint: ensure the guard checks the right documents (protocol/ports), stands at the departure gate (source interface), and allows valid travelers (permit any).

**Lifelong Retention Tip**: Simulate failed HTTPS access in Packet Tracer, check `show ip interface`, adjust ACL. Checklist: “No block? Check protocol, ports, interface, direction, permit any.” Practice 3 times.

## Practice Commands

- **Numbered ACL in Named Mode**:
  ```plaintext
  R1(config)# ip access-list extended 100
  R1(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 80
  R1(config-ext-nacl)# permit ip any any
  R1(config-ext-nacl)# no 10
  R1(config-ext-nacl)# 15 deny tcp 192.168.1.0 0.0.0.255 host 10.0.1.100 eq 443
  R1(config-ext-nacl)# exit
  R1(config)# ip access-list resequence 100 10 10
  R1(config)# interface g0/1
  R1(config-if)# ip access-group 100 in
  ```
- **Named ACL**:
  ```plaintext
  R1(config)# ip access-list extended NO_PING
  R1(config-ext-nacl)# deny icmp 192.168.1.0 0.0.0.255 any
  R1(config-ext-nacl)# permit ip any any
  R1(config-ext-nacl)# exit
  R1(config)# interface g0/1
  R1(config-if)# ip access-group NO_PING in
  ```
- **Verify**:
  ```plaintext
  show access-lists
  show ip interface g0/1
  show running-config | section access-list
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Extended ACL: “Protocol, source/destination IP, ports, close to source.”
   - Ranges: “Standard: 1–99, 1300–1999; Extended: 100–199, 2000–2699.”
   - Protocols: “ICMP=1, TCP=6, UDP=17, EIGRP=88, OSPF=89.”
2. **Visualize Topology**:
   - Draw R1 with ACLs on G0/1 in (NO_HTTPS), G0/2 in (NO_10.0.2.0), G0/0 out (NO_PING). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Extended ACL command? `ip access-list extended <name>`. Port for HTTPS? 443. Apply where? Inbound, source.”
4. **Practice in Packet Tracer**:
   - Configure `deny tcp ... eq 80`, apply on G0/1 in, test HTTP access, verify with `show access-lists`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Standard ACLs (source IP, destination) vs. Extended ACLs (protocol/ports, source).
   - Quiz: “Protocol for ping? ICMP.”
6. **Troubleshooting Practice**:
   - Simulate failed HTTPS access, check `show ip interface`, adjust protocol/port. Checklist: “No block? Check protocol, port, direction, permit any.”
7. **Teach Someone**:
   - Explain extended ACL syntax, close-to-source rule, and editing advantages.
8. **Spaced Repetition**:
   - Use Anki for ACL commands, protocol numbers, and quiz answers. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Extended ACL Purpose**:
   - Filter IPv4 traffic by protocol, source/destination IPs, ports for precise security (topic 5.1).
2. **Configuration**:
   - Numbered: `access-list 100-199`, or named mode: `ip access-list extended <100-199|name>`.
   - Match: Protocol (IP, TCP, UDP, ICMP), IPs (`host`, `any`, wildcard), ports (`eq 443`).
3. **Editing**:
   - Named config mode: Delete (`no <sequence>`), insert (`15 deny ...`), resequence (`ip access-list resequence`).
4. **Application**:
   - Close to source, inbound (e.g., R1 G0/1 in for `192.168.1.0/24`).
   - Command: `ip access-group <number|name> in`.
5. **Verification**:
   - `show access-lists`, `show ip interface`, `show running-config | section access-list`.
6. **CCNA Focus**:
   - Topic 5.1: Configure/verify IPv4 extended ACLs, know protocol numbers (ICMP=1, TCP=6, UDP=17, OSPF=89).

These notes provide a complete foundation for the Extended ACLs lecture, preparing you for CCNA exam topic 5.1. Practice configurations, troubleshoot access, and use analogies/retention tips to master extended ACLs for life!