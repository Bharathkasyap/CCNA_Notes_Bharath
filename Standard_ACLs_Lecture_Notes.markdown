# Standard ACLs Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture introduces Standard Access Control Lists (ACLs) for the CCNA, focusing on exam topic 5.1 ("Configure and verify access control lists"). Standard ACLs filter IPv4 traffic based on source IP addresses, used to control network access for security purposes. The lecture is part of a two-part series (Day 34: Standard ACLs, Day 35: Extended ACLs), covering ACL purpose, logic, types, configuration (standard numbered and named), and application. It assumes familiarity with IPv6 concepts from prior labs/lectures (Part 1: manual addressing, Part 2: EUI-64, static routes, Part 3: SLAAC, advanced static routes) and uses a sample network topology to demonstrate ACL functionality.

**Lecture Objectives**:
- **Understand ACL Purpose**: Learn how ACLs control network access by filtering packets.
- **Grasp ACL Logic**: Understand sequential processing, implicit deny, and interface application (inbound/outbound).
- **Identify ACL Types**: Differentiate standard (source IP only) vs. extended ACLs, and numbered vs. named.
- **Configure Standard ACLs**: Master commands for standard numbered (`access-list`) and named (`ip access-list standard`) ACLs.
- **Apply ACLs**: Apply ACLs to interfaces, following the rule-of-thumb to place standard ACLs close to the destination.
- **Verify ACLs**: Use `show access-lists`, `show ip access-lists`, and `show running-config` to verify configurations.
- **Prepare for Exam**: Master topic 5.1, focusing on IPv4 ACLs (IPv6 ACLs not required for CCNA).

**Network Topology**:
- **Devices**:
  - **R1**:
    - G0/1: Connected to `192.168.1.0/24` (PC1: `192.168.1.1`, PC2: `192.168.1.2`).
    - G0/2: Connected to `192.168.2.0/24` (PC3: `192.168.2.1`, PC4: `192.168.2.2`).
    - G0/0: Point-to-point to R2 (`10.0.0.0/30`, not detailed in transcript).
  - **R2**:
    - G0/0: Point-to-point to R1.
    - G0/1: Connected to `10.0.1.0/24` (SRV1: `10.0.1.100`).
    - G0/2: Connected to `10.0.2.0/24` (SRV2: `10.0.2.100`).
- **Note**: PCs are implied to connect via switches, but switches are omitted in the diagram (common in network diagrams).
- **Requirements (Examples)**:
  - Allow `192.168.1.0/24` (PC1, PC2) to access `10.0.1.0/24` (SRV1), deny `192.168.2.0/24` (PC3, PC4).
  - Allow PC1 (`192.168.1.1`) to access `192.168.2.0/24`, deny other `192.168.1.0/24` hosts.
  - Deny `192.168.1.0/24` from `10.0.2.0/24`, deny PC3 (`192.168.2.1`) from `10.0.1.0/24`, allow PC1 to `10.0.1.0/24` and other `192.168.2.0/24` hosts.

**Analogy**: Standard ACLs are like a security guard at a building entrance (interface), checking IDs (source IPs) against a guest list (ACL entries). The guard checks names in order (sequential processing), allows or denies entry, and denies unlisted guests (implicit deny). Placing the guard near the destination (e.g., server room) ensures only specific traffic is filtered.

**Lifelong Retention Tip**: Visualize R1 blocking PC3’s ping to SRV1 using an ACL on R2’s G0/1 outbound. Practice in Packet Tracer: configure `access-list 1 permit 192.168.1.0 0.0.0.255`, apply `ip access-group 1 out`, and verify with `show access-lists`. Create flashcards for commands (`access-list`, `ip access-group`) and rules (implicit deny, close-to-destination). Practice 3 times daily for a week.

## ACL Purpose and Logic

**Purpose**:
- **Definition**: Access Control Lists (ACLs) control network access by filtering packets based on criteria (e.g., source IP for standard ACLs). Primarily used for security to permit or deny traffic.
- **Security Use**: Control which devices access specific network segments (e.g., allow PC1 to SRV1, deny PC3).
- **Other Uses**: Beyond security (e.g., QoS, NAT), but CCNA focuses on security (topic 5.1).
- **Operation**: Routers forward packets per the routing table unless an ACL denies them. ACLs override routing decisions.

**Logic**:
- **Access Control Entries (ACEs)**: ACLs consist of ordered ACEs (e.g., permit `192.168.1.0/24`, deny `192.168.2.0/24`).
- **Sequential Processing**: Routers process ACEs top-to-bottom. The first matching entry determines the action (permit/deny), and remaining entries are ignored.
- **Implicit Deny**: All ACLs end with an invisible “deny any” entry, blocking unmatched traffic.
- **Interface Application**:
  - **Inbound**: Filter packets entering an interface (e.g., PC3’s ping entering R1 G0/2).
  - **Outbound**: Filter packets exiting an interface (e.g., PC3’s ping exiting R2 G0/1 to SRV1).
  - **Limit**: One ACL per interface per direction (max two: one in, one out). Applying a new ACL in the same direction replaces the previous one.
- **Rule-of-Thumb**: Apply standard ACLs as close to the destination as possible to avoid over-restricting traffic (e.g., apply on R2 G0/1 outbound to control access to `10.0.1.0/24`).

**Example** (Deny `192.168.2.0/24` from `10.0.1.0/24`, allow `192.168.1.0/24`):
- **ACL 1**:
  - ACE 1: `permit 192.168.1.0 0.0.0.255`
  - ACE 2: `deny 192.168.2.0 0.0.0.255`
  - ACE 3: `permit any` (to counter implicit deny).
- **Applied Outbound on R2 G0/1**:
  - PC1 (`192.168.1.1`) pings SRV1: Matches ACE 1, permitted.
  - PC3 (`192.168.2.1`) pings SRV1: Matches ACE 2, denied.
  - Other traffic (e.g., SRV2 to PC1): Matches ACE 3, permitted.
- **Incorrect Application**:
  - **Inbound on R1 G0/2**: Denies `192.168.2.0/24` from all external networks (e.g., `10.0.2.0/24`), too restrictive.
  - **Outbound on R1 G0/2**: Doesn’t check incoming pings from PC3, only SRV1’s replies (wrong source).

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1 Lecture/Lab**: No ACLs, focused on IPv6 addressing (`2001:db8::/64`) and `ipv6 unicast-routing`.
- **IPv6 Part 2 Lecture/Lab**: Static routes controlled traffic paths, not access. ACLs filter traffic, not routes.
- **IPv6 Part 3 Lecture/Lab**: Static routes (e.g., `ipv6 route 2001:db8:0:3::/64 g0/1`) resemble ACLs in controlling traffic flow, but ACLs filter based on source IP, not destination.
- **Difference**: IPv6 ACLs (not required for CCNA) use similar logic but apply to IPv6 addresses (e.g., `permit ipv6 2001:db8:0:1::/64 any`).

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Routes direct traffic (e.g., `ip route 10.0.1.0 255.255.255.0 g0/0`); ACLs filter it (e.g., `deny 192.168.2.0 0.0.0.255`).
- **EIGRP/OSPF**: Use wildcard masks (e.g., `network 192.168.1.0 0.0.0.255`), like ACLs. Dynamic routing doesn’t filter traffic.
- **NDP (IPv6)**: No filtering, resolves MACs (NS/NA) or configures addresses (SLAAC). ACLs control access.

**Analogy**: ACLs are like a nightclub bouncer checking IDs (source IPs) against a guest list (ACEs). The bouncer checks names in order, lets in approved guests (permit), turns away banned ones (deny), and denies everyone else (implicit deny). Placing the bouncer at the VIP room (destination interface) ensures only specific guests are filtered.

**Lifelong Retention Tip**: Draw R2 with ACL 1 on G0/1 outbound, permitting PC1, denying PC3. Flashcard: “Implicit deny? Denies all unmatched traffic.” Practice in Packet Tracer: configure `access-list 1 deny 192.168.2.0 0.0.0.255`, apply `ip access-group 1 out`, test pings. Repeat 3 times.

## ACL Types

**Overview**:
- **Standard ACLs**: Filter based on source IP only. Simple, used for basic access control.
  - **Numbered**: Identified by numbers (1–99, 1300–1999).
  - **Named**: Identified by names (e.g., `BLOCK_BOB`).
- **Extended ACLs**: Filter based on source/destination IP, port numbers, etc. (Covered in Day 35).
- **CCNA Focus**: IPv4 standard and extended ACLs (IPv6 ACLs not required).

**Number Ranges**:
- **Standard ACLs**: 1–99, 1300–1999 (originally 1–99, expanded for more ACLs).
- **Extended ACLs**: 100–199, 2000–2699 (Day 35).
- **Other Types** (not CCNA-required): Different ranges (e.g., IPX, AppleTalk).
- **Note**: Numbers identify ACL type; can’t use 100 for standard ACLs.

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1–3**: No ACLs, but static routes (`ipv6 route`) control traffic paths, similar to ACLs’ control of access. IPv6 ACLs (not covered) use `ipv6 access-list`.
- **Part 3 Lecture**: SLAAC and static routes manage connectivity; ACLs manage security.

**Comparison to Other CCNA Topics**:
- **EIGRP/OSPF**: Use network statements with wildcard masks, like ACLs (e.g., `0.0.0.255` for `/24`).
- **VLANs**: VLAN ACLs (VACLs) filter within VLANs, not covered in CCNA.
- **NAT**: Uses ACLs to identify traffic for translation (later topic).

**Analogy**: Standard ACLs are like a basic guest list checking only names (source IPs); extended ACLs check IDs and tickets (ports). Numbered ACLs use ID numbers (1–99); named ACLs use names (e.g., `VIP_LIST`).

**Lifelong Retention Tip**: Flashcard: “Standard ACL numbers? 1–99, 1300–1999.” List standard vs. extended ACLs in a table, compare to IPv6 static routes. Practice 3 times.

## Configuring Standard Numbered ACLs

**Commands**:
```plaintext
R1(config)# access-list <1-99,1300-1999> {permit | deny} <source-ip> <wildcard-mask>
R1(config)# access-list <1-99,1300-1999> remark <description>
R1(config-if)# ip access-group <1-99,1300-1999> {in | out}
```

**Options**:
- **Single Host**:
  - `access-list 1 deny 1.1.1.1 0.0.0.0` (explicit `/32`).
  - `access-list 1 deny 1.1.1.1` (implicit `/32`).
  - `access-list 1 deny host 1.1.1.1` (legacy, same as above).
- **Subnet**: `access-list 1 permit 192.168.1.0 0.0.0.255` (matches `/24`).
- **All Traffic**: `access-list 1 permit any` or `access-list 1 permit 0.0.0.0 255.255.255.255`.
- **Remark**: `access-list 1 remark #Block unauthorized hosts#` (aids readability, not functional).

**Example** (Deny `192.168.2.0/24`, allow others):
```plaintext
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)# access-list 1 deny 192.168.2.0 0.0.0.255
R1(config)# access-list 1 permit any
R1(config)# interface g0/1
R1(config-if)# ip access-group 1 out
```

**Observations**:
- **Wildcard Masks**: Inverse of subnet masks (e.g., `0.0.0.255` for `/24`, `0.0.0.0` for `/32`).
- **Auto-Conversion**: Router converts `1.1.1.1 0.0.0.0` to `1.1.1.1`, `0.0.0.0 255.255.255.255` to `any`.
- **Sequence Numbers**: Automatically assigned (10, 20, 30, etc.) in order of entry.
- **Order Matters**: Deny `192.168.1.0/24` before `permit 192.168.1.1` blocks PC1 (first match wins).

**Verification**:
```plaintext
R1# show access-lists
Standard IP access list 1
    10 permit 192.168.1.0, wildcard bits 0.0.0.255
    20 deny 192.168.2.0, wildcard bits 0.0.0.255
    30 permit any
R1# show ip access-lists
[Same output as above]
R1# show running-config | include access-list
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 deny 192.168.2.0 0.0.0.255
access-list 1 permit any
access-list 1 remark #Block unauthorized hosts#
```

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1–3**: No ACLs, but `ipv6 route` uses similar precision (e.g., `/64` vs. `/128`). IPv6 ACLs use `ipv6 access-list`, not CCNA-required.
- **Part 3 Lab**: Static routes (`ipv6 route 2001:db8:0:3::/64 g0/1`) control paths; ACLs control access by source.

**Comparison to Other CCNA Topics**:
- **EIGRP/OSPF**: Wildcard masks in `network` statements (e.g., `0.0.0.255`) match ACL syntax.
- **IPv4 Static Routing**: Routes use subnet masks; ACLs use wildcard masks.
- **VLANs**: No filtering, but VLANs segment traffic, complementing ACLs.

**Analogy**: Configuring numbered ACLs is like writing a numbered guest list (1–99) with rules (permit Bob, deny others). Applying it to an exit (outbound) checks outgoing guests.

**Lifelong Retention Tip**: In Packet Tracer, configure `access-list 1 deny 192.168.2.0 0.0.0.255`, apply `ip access-group 1 out`, verify with `show access-lists`. Flashcard: “Standard ACL command? `access-list <1-99> {permit|deny} <ip> <mask>`.” Practice 3 times.

## Configuring Standard Named ACLs

**Commands**:
```plaintext
R2(config)# ip access-list standard <name>
R2(config-std-nacl)# [<sequence>] {permit | deny} <source-ip> <wildcard-mask>
R2(config-std-nacl)# [<sequence>] remark <description>
R2(config)# interface <interface>
R2(config-if)# ip access-group <name> {in | out}
```

**Example** (Deny `192.168.1.0/24` from `10.0.2.0/24`, allow others):
```plaintext
R2(config)# ip access-list standard TO_10.0.2.0
R2(config-std-nacl)# 10 deny 192.168.1.0 0.0.0.255
R2(config-std-nacl)# 20 permit any
R2(config-std-nacl)# remark #Block 192.168.1.0/24 from SRV2#
R2(config-std-nacl)# exit
R2(config)# interface g0/2
R2(config-if)# ip access-group TO_10.0.2.0 out
```

**Observations**:
- **Named ACLs**: Use descriptive names (e.g., `TO_10.0.2.0`) instead of numbers.
- **Config Mode**: `ip access-list standard <name>` enters standard named ACL mode.
- **Sequence Numbers**: Optional; default increments by 10 (10, 20, 30). Manual numbers (e.g., 5, 10) allow precise ordering.
- **Auto-Reordering**: Routers may reorder `/32` entries (e.g., `deny 192.168.2.1`) for efficiency, but effect remains unchanged. Packet Tracer doesn’t reorder.
- **Application**: Same as numbered ACLs (`ip access-group <name> out`).

**Verification**:
```plaintext
R2# show access-lists
Standard IP access list TO_10.0.2.0
    10 deny   192.168.1.0, wildcard bits 0.0.0.255
    20 permit any
R2# show running-config | section access-list
ip access-list standard TO_10.0.2.0
    deny   192.168.1.0 0.0.0.255
    permit any
    remark #Block 192.168.1.0/24 from SRV2#
```

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 1–3**: No ACLs, but named IPv6 ACLs (not CCNA-required) use similar syntax (`ipv6 access-list <name>`).
- **Part 3 Lecture**: SLAAC uses `ipv6 address autoconfig` for auto-configuration; named ACLs use descriptive names for clarity, like `TO_10.0.2.0`.

**Comparison to Other CCNA Topics**:
- **EIGRP/OSPF**: Named EIGRP (`router eigrp AS_NAME`) uses names like named ACLs.
- **VLANs**: Named VLANs (e.g., `vlan 10 name SALES`) resemble named ACLs in clarity.
- **NAT**: Uses named ACLs for traffic identification (later topic).

**Analogy**: Named ACLs are like a labeled guest list (e.g., `VIP_LIST`) with custom order numbers, easier to read than numbered lists. The bouncer (router) checks the list at the exit (interface).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip access-list standard BLOCK_BOB`, add `deny 192.168.2.1`, apply `ip access-group BLOCK_BOB out`. Flashcard: “Named ACL command? `ip access-list standard <name>`.” Practice 3 times.

## Example Configurations

**Requirement 1** (R1: Allow PC1 to `192.168.2.0/24`, deny other `192.168.1.0/24` hosts):
```plaintext
R1(config)# access-list 1 permit 192.168.1.1
R1(config)# access-list 1 deny 192.168.1.0 0.0.0.255
R1(config)# access-list 1 permit any
R1(config)# interface g0/2
R1(config-if)# ip access-group 1 out
```
- **Why Outbound on G0/2**: Close to destination (`192.168.2.0/24`). Inbound on G0/1 would block `192.168.1.0/24` from all external networks.

**Requirement 2** (R2: Deny `192.168.1.0/24` from `10.0.2.0/24`, deny PC3 from `10.0.1.0/24`, allow PC1 and other `192.168.2.0/24` to `10.0.1.0/24`):
```plaintext
R2(config)# ip access-list standard TO_10.0.2.0
R2(config-std-nacl)# deny 192.168.1.0 0.0.0.255
R2(config-std-nacl)# permit any
R2(config-std-nacl)# exit
R2(config)# interface g0/2
R2(config-if)# ip access-group TO_10.0.2.0 out
R2(config)# ip access-list standard TO_10.0.1.0
R2(config-std-nacl)# 30 deny 192.168.2.1
R2(config-std-nacl)# 10 permit 192.168.2.0 0.0.0.255
R2(config-std-nacl)# 20 permit 192.168.1.1
R2(config-std-nacl)# 40 deny 192.168.1.0 0.0.0.255
R2(config-std-nacl)# 50 permit any
R2(config-std-nacl)# exit
R2(config)# interface g0/1
R2(config-if)# ip access-group TO_10.0.1.0 out
```
- **Note**: Router may reorder `/32` entries (e.g., `deny 192.168.2.1`) for efficiency, but effect is unchanged.

**Verification**:
```plaintext
R2# show ip access-lists
Standard IP access list TO_10.0.2.0
    10 deny   192.168.1.0, wildcard bits 0.0.0.255
    20 permit any
Standard IP access list TO_10.0.1.0
    30 deny   192.168.2.1
    10 permit 192.168.2.0, wildcard bits 0.0.0.255
    20 permit 192.168.1.1
    40 deny   192.168.1.0, wildcard bits 0.0.0.255
    50 permit any
R2# show running-config | section access-list
ip access-list standard TO_10.0.2.0
    deny   192.168.1.0 0.0.0.255
    permit any
ip access-list standard TO_10.0.1.0
    deny   192.168.2.1
    permit 192.168.2.0 0.0.0.255
    permit 192.168.1.1
    deny   192.168.1.0 0.0.0.255
    permit any
```

**Comparison to Prior IPv6 Content**:
- **IPv6 Part 3 Lab**: Configured static routes for connectivity; ACLs configure access restrictions. Both use precise addressing (IPv6: `/64`, `/128`; ACLs: `/24`, `/32`).
- **Part 3 Lecture**: NDP and SLAAC manage addressing; ACLs manage security, but both require enabling features (`ipv6 unicast-routing` vs. `ip access-group`).

**Comparison to Other CCNA Topics**:
- **IPv4 Routing**: Routes forward traffic; ACLs filter it. Both use IP addressing.
- **Security Fundamentals**: ACLs align with firewalls, controlling access (topic 5.1).
- **VLANs**: Segment traffic; ACLs filter between segments.

**Analogy**: Configuring ACLs is like setting up a guest list for two rooms (G0/1, G0/2). Each room gets its own list (ACL), checked at the door (interface) to allow or deny guests (packets).

**Lifelong Retention Tip**: In Packet Tracer, configure both ACLs on R2, test pings (PC1 to SRV1, PC3 to SRV1), verify with `show ip access-lists`. Flashcard: “Apply ACL? `ip access-group <name> out`.” Practice 3 times.

## Quiz Answers and Explanations

1. **Which ACL permits only PC1 and PC4 to access `10.0.1.0/24` outbound on R2 G0/1?**
   - **Answer**: ACL 1 (`permit 192.168.1.1`, `permit 192.168.2.2`, implicit deny).
   - **Explanation**: Entry 10 permits PC1 (`192.168.1.1`), entry 20 permits PC4 (`192.168.2.2`). Implicit deny blocks others (e.g., PC2, PC3).

2. **Which interface and direction for ACL to deny `192.168.1.0/24` from `10.0.2.0/24`?**
   - **Answer**: R2 G0/2, outbound.
   - **Explanation**: Apply close to destination (`10.0.2.0/24`). Outbound on G0/2 filters traffic to SRV2, avoiding over-restriction.

3. **Effect of multiple ACLs on R2 G0/1 inbound?**
   - **Commands**: `access-list 1 deny 192.168.0.0 0.0.255.255`, `access-list 10 deny 10.0.0.0 0.0.0.255`, applied inbound on G0/1.
   - **Answer**: B (Deny `10.0.0.0/24`).
   - **Explanation**: Last ACL applied (ACL 10) replaces ACL 1. Only `deny 10.0.0.0 0.0.0.255` is active.

4. **Which PCs can ping SRV2 with ACL inbound on R1 G0/0?**
   - **ACL**: `permit 192.168.1.1`, `deny 192.168.1.0 0.0.0.255`, `permit any`.
   - **Answer**: D (All PCs).
   - **Explanation**: Inbound on G0/0 checks SRV2’s replies (`10.0.2.100`), not PC sources. `permit any` allows all replies.

5. **What happens if a packet doesn’t match any ACL entries?**
   - **Answer**: C (Packet dropped).
   - **Explanation**: Implicit deny at the end of every ACL drops unmatched packets.

**Bonus Question (Boson ExSim)**: Likely tests ACL application, sequence, or implicit deny (e.g., “Which ACL permits PC1, denies PC2?”).

**Analogy**: The quiz is like testing the bouncer’s guest list: identifying who gets in (permit), where to stand (interface), and what happens to unlisted guests (implicit deny).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Implicit deny? Drops packet”). Practice ACL scenarios in Packet Tracer (e.g., deny PC3, permit PC1), verify with `show access-lists`. Repeat quiz 3 times.

## Common Issues and Troubleshooting

- **Traffic Not Blocked**:
  - **Issue**: PC3 can ping SRV1 despite ACL denying `192.168.2.0/24`.
  - **Cause**: ACL applied to wrong interface/direction (e.g., R1 G0/2 outbound checks SRV1’s replies, not PC3’s pings).
  - **Fix**: Apply ACL outbound on R2 G0/1, verify with `show ip access-lists`, test ping.
- **All Traffic Blocked**:
  - **Issue**: No PCs can ping SRV1.
  - **Cause**: Missing `permit any` (implicit deny blocks all unmatched traffic).
  - **Fix**: Add `access-list 1 permit any`, reapply `ip access-group 1 out`.
- **Wrong ACL Applied**:
  - **Issue**: ACL 10 applied instead of ACL 1.
  - **Cause**: Multiple ACLs applied to same interface/direction; last one overrides.
  - **Fix**: Verify with `show running-config | include access-group`, reapply correct ACL.
- **Incorrect Order**:
  - **Issue**: PC1 blocked despite `permit 192.168.1.1`.
  - **Cause**: `deny 192.168.1.0 0.0.0.255` precedes `permit 192.168.1.1`.
  - **Fix**: Reconfigure ACL with correct order or use named ACL with sequence numbers.

**Analogy**: Troubleshooting ACLs is like fixing a bouncer’s mistakes: checking the guest list (ACL entries), ensuring they’re at the right door (interface), and adding “allow others” to avoid turning away everyone (implicit deny).

**Lifelong Retention Tip**: Simulate failed pings in Packet Tracer, check `show ip access-lists`, adjust ACLs. Checklist: “No ping? Check interface, direction, order, permit any.” Practice 3 times.

## Practice Commands

- **Standard Numbered ACL**:
  ```plaintext
  R1(config)# access-list 1 permit 192.168.1.1
  R1(config)# access-list 1 deny 192.168.1.0 0.0.0.255
  R1(config)# access-list 1 permit any
  R1(config)# interface g0/2
  R1(config-if)# ip access-group 1 out
  ```
- **Standard Named ACL**:
  ```plaintext
  R2(config)# ip access-list standard TO_10.0.1.0
  R2(config-std-nacl)# deny 192.168.2.1
  R2(config-std-nacl)# permit 192.168.2.0 0.0.0.255
  R2(config-std-nacl)# permit 192.168.1.1
  R2(config-std-nacl)# deny 192.168.1.0 0.0.0.255
  R2(config-std-nacl)# permit any
  R2(config-std-nacl)# exit
  R2(config)# interface g0/1
  R2(config-if)# ip access-group TO_10.0.1.0 out
  ```
- **Verify**:
  ```plaintext
  show access-lists
  show ip access-lists
  show running-config | include access-list
  show running-config | section access-list
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - ACL Logic: “Top-to-bottom, first match wins, implicit deny ends.”
   - Standard ACL: “Source IP only, 1–99 or 1300–1999.”
   - Application: “Close to destination, outbound for standard ACLs.”
2. **Visualize Topology**:
   - Draw R1–R2, show ACL on R2 G0/1 outbound denying PC3, permitting PC1. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Standard ACL range? 1–99, 1300–1999. Implicit deny? Drops all unmatched. Apply command? `ip access-group`.”
4. **Practice in Packet Tracer**:
   - Configure ACL to deny PC3, permit PC1 to SRV1, apply on R2 G0/1 out, test pings, verify with `show ip access-lists`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: IPv6 Part 3 (SLAAC, static routes) vs. ACLs (filtering). Quiz: “Implicit deny effect? Drops packet.”
6. **Troubleshooting Practice**:
   - Simulate failed ping (PC3 to SRV1), check ACL order, interface, `permit any`. Checklist: “No ping? Check direction, order, implicit deny.”
7. **Teach Someone**:
   - Explain ACL logic (top-to-bottom, implicit deny), numbered vs. named, and why apply close to destination. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki for ACL commands, logic, and quiz answers. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **ACL Purpose**:
   - Filter IPv4 traffic for security (permit/deny based on source IP for standard ACLs).
2. **ACL Logic**:
   - Sequential processing (top-to-bottom), first match wins, implicit deny blocks unmatched traffic.
   - Apply one ACL per interface per direction (inbound/outbound).
3. **Standard ACL Types**:
   - Numbered (1–99, 1300–1999): `access-list <number> {permit|deny} <ip> <mask>`.
   - Named: `ip access-list standard <name>`, with optional sequence numbers.
4. **Configuration**:
   - Use wildcard masks (e.g., `0.0.0.255` for `/24`).
   - Options: `host`, `any`, or explicit masks.
   - Remarks improve readability.
5. **Application**:
   - Apply close to destination (e.g., R2 G0/1 out for `10.0.1.0/24`).
   - Command: `ip access-group <number|name> {in|out}`.
6. **Verification**:
   - `show access-lists`, `show ip access-lists`, `show running-config | section access-list`.
7. **CCNA Focus**:
   - Topic 5.1: Configure/verify IPv4 standard ACLs (IPv6 not required).

These notes provide a complete foundation for the Standard ACLs lecture, preparing you for CCNA exam topic 5.1. Practice configurations, troubleshoot pings, and use analogies/retention tips to master ACLs for life!