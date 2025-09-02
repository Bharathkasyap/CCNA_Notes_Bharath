# NAT Part 2 Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture concludes the **Network Address Translation (NAT)** topic for **CCNA exam topic 4.1** ("Configure and verify inside source NAT using static and pools"). Building on Day 44’s static NAT, it covers dynamic NAT and dynamic Port Address Translation (PAT, or NAT overload). Dynamic NAT automatically maps private to public IPs using an access control list (ACL) and a pool, while PAT extends this by translating port numbers, allowing multiple devices to share one public IP. The lecture emphasizes configuration, verification, and NAT pool exhaustion, preparing students for CCNA NAT questions. A bonus question from Boson Software’s ExSim reinforces concepts.

**Lecture Objectives**:
- **Review Static NAT**: Highlight two-way communication (inside-to-outside and outside-to-inside).
- **Understand Dynamic NAT**: Learn automatic one-to-one mappings using ACLs and pools.
- **Explore Dynamic PAT**: Understand many-to-one mappings with port translation.
- **Configure and Verify**: Master commands for dynamic NAT and PAT, including `show` commands.
- **Prepare for Exam**: Address NAT pool exhaustion and ACL usage for NAT (topic 4.1).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      R1        |-------|    Server      |
| 192.168.0.167  |       | G0/1: Inside   |       |   8.8.8.8      |
|                |       | G0/0: 203.0.113.1 |    |   (DNS)        |
+----------------+       +----------------+       +----------------+
         |                       |
         |                       |
+----------------+       +----------------+
|      PC2       |       |                |
| 192.168.0.168  |       |                |
+----------------+       +----------------+
```
- **Devices**:
  - **PC1**: Inside host (192.168.0.167), NAT to pool (100.0.0.1–10) or G0/0 (203.0.113.1).
  - **PC2**: Inside host (192.168.0.168), NAT to pool or G0/0.
  - **R1**: Router performing dynamic NAT/PAT (G0/1 inside, G0/0 outside).
  - **Server**: External DNS server (8.8.8.8).
- **Note**: R1 uses ACL to identify 192.168.0.0/24 for translation to pool (100.0.0.1–10) or G0/0 IP (203.0.113.1). Connectivity via OSPF/static routes assumed.
- **Requirements**:
  1. Review static NAT’s two-way functionality.
  2. Configure dynamic NAT with ACL and pool.
  3. Configure PAT with pool or interface, using `overload`.
  4. Verify with `show ip nat translations`, `show ip nat statistics`.
  5. Understand NAT pool exhaustion.

**Analogy**: Dynamic NAT is like a post office (R1) automatically assigning temporary PO boxes (100.0.0.1–10) to PCs for Internet mail. PAT is like sharing one PO box (203.0.113.1) with unique slot numbers (ports) for each PC’s mail, maximizing efficiency.

**Lifelong Retention Tip**: Visualize PC1 (192.168.0.167) and PC2 (192.168.0.168) pinging 8.8.8.8 via R1, with PAT using one IP (203.0.113.1) and unique ports. In Packet Tracer, configure `ip nat inside source list 1 interface g0/0 overload`, ping 8.8.8.8, verify with `show ip nat translations`. Create flashcards: “Dynamic NAT? One-to-one, ACL+pool. PAT? Many-to-one, ports.” Practice 3 times daily for a week.

## Static NAT Review

**Purpose**: Reinforce static NAT’s two-way functionality, allowing external hosts to initiate communication.

**Key Points**:
- **Static NAT**: One-to-one mapping (e.g., 192.168.0.167 → 100.0.0.1).
- **Two-Way Communication**:
  - Inside-to-outside: PC1 (192.168.0.167) pings 8.8.8.8; R1 translates to 100.0.0.1.
  - Outside-to-inside: Server (8.8.8.8) sends to 100.0.0.1; R1 translates to 192.168.0.167.
- **Permanent Mappings**: Static entries persist in NAT table.

**Demonstration**:
- PC1 sends: Source 192.168.0.167, Destination 8.8.8.8 → R1 translates to 100.0.0.1.
- Server sends to 100.0.0.1: R1 translates to 192.168.0.167, forwards to PC1.
- External host sends to 100.0.0.1: R1 translates to 192.168.0.167, enabling access.

**Observations**:
- **Use Case**: Allows external access to internal servers (e.g., web server at 192.168.0.167 via 100.0.0.1).
- **Limitation**: Requires one public IP per device; not IP-efficient.
- **Exam Note**: Static NAT supports bidirectional communication (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 1**: Introduced static NAT; Part 2 adds outside-to-inside functionality.
- **Syslog Lab**: Static NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: Static NAT enables external SSH to internal devices.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Static NAT preserves ports (e.g., UDP 53 for DNS).
- **IP Addressing (Day 10)**: Maps private (192.168.0.0/16) to public IPs.
- **Extended ACLs (Day 35)**: ACLs permit inbound traffic to 100.0.0.1.

**Analogy**: Static NAT is like a dedicated PO box (100.0.0.1) for PC1, allowing both outgoing mail (to 8.8.8.8) and incoming mail from external hosts.

**Lifelong Retention Tip**: In Packet Tracer, configure static NAT, test external ping to 100.0.0.1. Flashcard: “Static NAT? One-to-one, two-way. Command? `ip nat inside source static`.” Practice 3 times.

## Dynamic NAT

**Purpose**: Automatically map private (inside local) to public (inside global) IPs using an ACL and pool.

**Key Points**:
- **Mechanism**: Router dynamically assigns inside global IPs from a pool to permitted traffic.
- **ACL Usage**: Identifies traffic to translate (permit = translate, deny = no translation, not dropped).
- **NAT Pool**: Defines available public IPs (e.g., 100.0.0.1–10).
- **One-to-One**: Each inside local IP gets one inside global IP.
- **Timeouts**: Dynamic entries timeout (default 24 hours) or can be cleared manually.
- **NAT Pool Exhaustion**: If pool is full (e.g., 10 IPs used), new packets are dropped.

**Demonstration**:
- ACL 1: `permit ip 192.168.0.0 0.0.0.255`.
- Pool: 100.0.0.1–100.0.0.10.
- PC1 (192.168.0.167) pings 8.8.8.8: R1 assigns 100.0.0.1.
- PC2 (192.168.0.168) pings: R1 assigns 100.0.0.2.
- If 10 IPs used, PCx (192.168.0.98) packet dropped until an IP frees up (timeout or `clear ip nat translation *`).

**Configuration (on R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.10 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1
```

**Verification**:
- **Show IP NAT Translations** (after PC1/PC2 ping/DNS to 8.8.8.8):
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         192.168.0.167      ---                ---
--- 100.0.0.2         192.168.0.168      ---                ---
udp 100.0.0.1:5000    192.168.0.167:5000 8.8.8.8:53         8.8.8.8:53
udp 100.0.0.2:5001    192.168.0.168:5001 8.8.8.8:53         8.8.8.8:53
icmp 100.0.0.1:1      192.168.0.167:1    8.8.8.8:1          8.8.8.8:1
icmp 100.0.0.2:2      192.168.0.168:2    8.8.8.8:2          8.8.8.8:2
```
- **Show IP NAT Statistics**:
```plaintext
R1# show ip nat statistics
Total active translations: 6 (2 dynamic; 4 extended)
Peak translations: 6
Outside interfaces: GigabitEthernet0/0
Inside interfaces: GigabitEthernet0/1
Dynamic mappings: Inside -> Outside: ACL 1 pool POOL1
```

**Observations**:
- **ACL Role**: Permits 192.168.0.0/24 for translation; deny doesn’t drop packets.
- **Pool Exhaustion**: Limited to 10 IPs; excess traffic dropped.
- **Timeouts**: Dynamic mappings last 24 hours; extended entries (UDP/ICMP) ~1 minute.
- **Clearing**: `clear ip nat translation *` removes all dynamic entries.
- **Exam Note**: Know ACL, pool, and `ip nat inside source list` for dynamic NAT (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 1**: Static NAT is manual; dynamic NAT automates mappings.
- **Syslog Lab**: Dynamic NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACLs identify traffic for NAT, not block.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Dynamic NAT preserves ports; extended entries show UDP/ICMP.
- **IP Addressing (Day 10)**: Maps private (192.168.0.0/16) to public IPs.
- **Extended ACLs (Day 35)**: NAT ACL uses wildcard masks (0.0.0.255).

**Analogy**: Dynamic NAT is like a post office assigning temporary PO boxes (100.0.0.1–10) to PCs based on an approved sender list (ACL 1), recycling boxes when unused.

**Lifelong Retention Tip**: In Packet Tracer, configure dynamic NAT, ping 8.8.8.8, verify with `show ip nat translations`. Flashcard: “Dynamic NAT? ACL+pool, one-to-one. Pool exhaustion? Drop packets.” Practice 3 times.

## Dynamic PAT (NAT Overload)

**Purpose**: Allow multiple private IPs to share one public IP by translating both IP and port numbers.

**Key Points**:
- **Mechanism**: Translates inside local IP to one inside global IP, using unique ports to track flows.
- **Port Translation**: Changes source port if needed (e.g., both PCs use 54321 → 54321, 54322).
- **Efficiency**: Conserves public IPs; one IP supports ~65,000 flows (16-bit ports).
- **Common Use**: Home routers use PAT for multiple devices (e.g., PC, phone sharing one IP).

**Demonstration**:
- PC1: 192.168.0.167:54321 → 100.0.0.1:54321 (DNS to 8.8.8.8).
- PC2: 192.168.0.168:54321 → 100.0.0.1:54322 (same port; R1 changes to 54322).
- Replies to 100.0.0.1:54321 → PC1; 100.0.0.1:54322 → PC2.
- If ports differ (e.g., 65205, 59641), no translation needed.

**Configuration (Pool-Based PAT)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)# ip nat pool POOL1 100.0.0.0 100.0.0.3 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1 overload
```

**Configuration (Interface-Based PAT)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)# ip nat inside source list 1 interface g0/0 overload
```

**Verification** (Interface-Based PAT):
- **Show IP NAT Translations**:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
udp 203.0.113.1:65205 192.168.0.167:65205 8.8.8.8:53         8.8.8.8:53
udp 203.0.113.1:59641 192.168.0.168:59641 8.8.8.8:53         8.8.8.8:53
```
- **Show IP NAT Statistics**:
```plaintext
R1# show ip nat statistics
Total active translations: 2 (0 dynamic; 2 extended)
Peak translations: 2
Outside interfaces: GigabitEthernet0/0
Inside interfaces: GigabitEthernet0/1
Dynamic mappings: Inside -> Outside: ACL 1 interface GigabitEthernet0/0
```

**Observations**:
- **Many-to-One**: Multiple PCs share one IP (203.0.113.1) with unique ports.
- **Port Translation**: Only if source ports conflict; else preserved (e.g., 65205, 59641).
- **No One-to-One Mappings**: Unlike dynamic NAT, no static pool entries in table.
- **Interface-Based**: Common for home routers; uses router’s IP (203.0.113.1).
- **Exam Note**: Know `overload` keyword, interface-based PAT (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 1**: Static NAT is one-to-one; PAT is many-to-one with ports.
- **Syslog Lab**: PAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACLs identify PAT traffic (permit = translate).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: PAT relies on TCP/UDP ports (e.g., 65205, 59641).
- **IP Addressing (Day 10)**: PAT conserves public IPs.
- **Extended ACLs (Day 35)**: ACLs use wildcard masks for PAT.

**Analogy**: PAT is like a post office using one PO box (203.0.113.1) for all PCs, assigning unique slot numbers (ports) to track each PC’s mail.

**Lifelong Retention Tip**: In Packet Tracer, configure `ip nat inside source list 1 interface g0/0 overload`, ping 8.8.8.8 from PC1/PC2, verify ports in `show ip nat translations`. Flashcard: “PAT? Many-to-one, ports. Keyword? `overload`.” Practice 3 times.

## Common Issues and Troubleshooting

- **NAT Pool Exhaustion**:
  - **Issue**: Dynamic NAT drops packets from 192.168.0.98.
  - **Cause**: Pool (100.0.0.1–10) full.
  - **Fix**: Clear translations (`clear ip nat translation *`) or use PAT.
- **PAT Fails**:
  - **Issue**: PC1 can’t ping 8.8.8.8.
  - **Cause**: Missing `overload`, incorrect ACL, or interface misconfiguration.
  - **Fix**: Verify `ip nat inside source list 1 interface g0/0 overload`, ACL (`show access-lists`).
- **Incorrect ACL**:
  - **Issue**: Packets not translated.
  - **Cause**: ACL denies traffic (e.g., `deny ip 192.168.0.0 0.0.0.255`).
  - **Fix**: Use `permit ip 192.168.0.0 0.0.0.255`, check `show access-lists`.
- **Pool Misconfiguration**:
  - **Issue**: `ip nat pool` rejected.
  - **Cause**: Start/end IPs not in same subnet (e.g., 203.0.113.0–255).
  - **Fix**: Use valid range (e.g., 100.0.0.0–3, netmask 255.255.255.0).
- **No Internet Access**:
  - **Issue**: PCs can’t reach 8.8.8.8.
  - **Cause**: Routing or ACL blocking NAT’d traffic.
  - **Fix**: Check routes (`show ip route`), ACLs (`show access-lists`).

**Analogy**: Troubleshooting PAT is like ensuring a shared PO box (203.0.113.1) assigns correct slot numbers (ports) and has clear delivery routes (no ACL blocks).

**Lifelong Retention Tip**: Simulate PAT failure (wrong ACL), retry with `permit ip 192.168.0.0 0.0.0.255`, verify. Checklist: “No PAT? Check ACL, pool, overload, routes.” Practice 3 times.

## Practice Commands

- **Dynamic NAT (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.10 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1
```
- **PAT (Pool-Based)**:
```plaintext
R1(config)# ip nat pool POOL1 100.0.0.0 100.0.0.3 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1 overload
```
- **PAT (Interface-Based)**:
```plaintext
R1(config)# ip nat inside source list 1 interface g0/0 overload
```
- **Verify NAT/PAT**:
```plaintext
R1# show ip nat translations
R1# show ip nat statistics
R1# clear ip nat translation *
```

## Quiz Answers and Explanations

1. **Which NAT type best preserves public IPv4 addresses?**  
   - **A**: Static NAT  
   - **B**: Dynamic NAT  
   - **C**: Destination NAT  
   - **D**: NAT Overload (PAT)  
   - **Answer**: D (PAT allows many devices to share one IP via port translation).  

2. **Which dynamic NAT config translates 172.16.1.0/24 to 203.0.113.0/25?**  
   - **A**: `access-list 1 permit 172.16.1.0 0.0.0.255`, `ip nat pool POOL1 203.0.113.0 203.0.113.255 netmask 255.255.255.128`  
   - **B**: `access-list 1 permit 172.16.1.0 0.0.0.255`, `ip nat pool POOL1 203.0.113.0 203.0.113.127 netmask 255.255.255.128`  
   - **C**: `access-list 1 permit 172.16.1.0 255.255.255.0`, `ip nat pool POOL1 203.0.113.0 203.0.113.127 netmask 255.255.255.128`  
   - **D**: Incorrect interfaces  
   - **Answer**: B (A: pool range exceeds /25; C: ACL uses netmask, not wildcard).  

3. **What does R1 do if dynamic NAT pool (10 IPs) is exhausted?**  
   - **A**: Translates using router’s IP  
   - **B**: Discards the packet  
   - **C**: Uses PAT  
   - **D**: Queues the packet  
   - **Answer**: B (No available IPs = packet dropped).  

4. **Which config translates 10.0.1.0/27 to G0/1’s IP?**  
   - **A**: `access-list 1 permit 10.0.1.0 0.0.0.31`, `ip nat inside source list 1 interface g0/1 overload`  
   - **B**: Incorrect wildcard mask  
   - **C**: Incorrect interface  
   - **D**: Missing `overload`  
   - **Answer**: A (Correct /27 wildcard, interface, `overload`).  

5. **What happens to 192.168.1.0/24 with `access-list 1 deny ip 192.168.1.0 0.0.0.255`?**  
   - **A**: Packets dropped  
   - **B**: Packets translated  
   - **C**: Packets not translated  
   - **D**: ACL rejected  
   - **Answer**: C (Deny in NAT ACL means no translation, not dropped).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: Which command configures PAT?
    - **A**: `ip nat inside source list 1 interface g0/0 overload`  
    - **B**: `ip nat inside source list 1 pool POOL1`  
    - **C**: `ip nat inside source static 192.168.1.1 100.0.0.1`  
    - **D**: `ip nat pool POOL1 100.0.0.1 100.0.0.1`  
    - **Answer**: A (`overload` enables PAT).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “PAT command? `ip nat inside source list 1 interface g0/0 overload`. Pool exhaustion? Drop packets”). Practice 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - NAT Types: “Static, Dynamic, PAT” (SDP).
   - PAT: “Overload = Many-to-one, Ports” (OMP).
2. **Visualize Topology**:
   - Draw PC1/PC2 → R1 → 8.8.8.8 with PAT (203.0.113.1, unique ports). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Dynamic NAT? ACL+pool. PAT? `overload`, ports. Pool exhaustion? Drop.”
4. **Practice in Packet Tracer**:
   - Configure PAT, ping 8.8.8.8, verify (`show ip nat translations`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Static NAT (one-to-one, permanent) vs. Dynamic NAT (one-to-one, temporary) vs. PAT (many-to-one, ports).
   - Quiz: “PAT command? `ip nat inside source list 1 interface g0/0 overload`. ACL deny? No translation.”
6. **Troubleshooting Practice**:
   - Use wrong ACL, retry with `permit ip 192.168.0.0 0.0.0.255`, verify. Checklist: “No PAT? Check ACL, overload, interfaces.”
7. **Teach Someone**:
   - Explain PAT and port translation to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, NAT types, ACL usage. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Static NAT**:
   - One-to-one, permanent, two-way (inside-to-outside, outside-to-inside).
2. **Dynamic NAT**:
   - One-to-one, temporary, uses ACL and pool; prone to pool exhaustion.
3. **Dynamic PAT**:
   - Many-to-one, uses ports (~65,000 flows), `overload` keyword, interface-based common.
4. **Configuration**:
   - `ip nat inside`, `ip nat outside`, `access-list`, `ip nat pool`, `ip nat inside source list`.
5. **Verification**:
   - `show ip nat translations` (dynamic/extended entries), `show ip nat statistics`.
6. **CCNA Focus**:
   - Topic 4.1: Master dynamic NAT/PAT config, ACL usage, pool exhaustion.

These notes provide a complete foundation for NAT Part 2, preparing you for CCNA exam topic 4.1. Practice dynamic NAT/PAT in Packet Tracer, verify with `show ip nat translations`, and use analogies/retention tips to master NAT for life!