# NAT Part 1 Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **Network Address Translation (NAT)**, a critical topic for the **CCNA exam topic 4.1** ("Configure and verify inside source NAT using static and pools"). It introduces private IPv4 addresses, the purpose of NAT, and static source NAT, including its configuration on Cisco routers. NAT translates source and/or destination IP addresses, enabling private IP hosts to access the Internet. The lecture emphasizes private IP ranges (RFC 1918), static NAT’s one-to-one mappings, and key terms (inside local, inside global, outside local, outside global). Part 2 (Day 45) will cover additional NAT types, such as Port Address Translation (PAT). A bonus question from Boson Software’s ExSim reinforces key concepts.

**Lecture Objectives**:
- **Understand Private IPv4 Addresses**: Learn RFC 1918 ranges and their role in internal networks.
- **Introduce NAT**: Explain NAT’s purpose in translating IP addresses for Internet access.
- **Explore Static NAT**: Describe one-to-one mappings and configuration for source NAT.
- **Master Key Terms**: Differentiate inside local, inside global, outside local, and outside global addresses.
- **Prepare for Exam**: Understand static NAT configuration and verification (topic 4.1).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      R1        |-------|    Server      |
| 192.168.0.167  |       | G0/1: Inside   |       |   8.8.8.8      |
|                |       | G0/0: 203.0.113.1 |    |                |
+----------------+       +----------------+       +----------------+
         |                       |
         |                       |
+----------------+       +----------------+
|      PC2       |       |                |
| 192.168.0.168  |       |                |
+----------------+       +----------------+
```
- **Devices**:
  - **PC1**: Inside host (192.168.0.167), NAT to 100.0.0.1.
  - **PC2**: Inside host (192.168.0.168), NAT to 100.0.0.2.
  - **R1**: Router performing static NAT (G0/1 inside, G0/0 outside).
  - **Server**: External host (8.8.8.8, e.g., DNS server).
- **Note**: R1 translates private IPs (192.168.0.167/168) to public IPs (100.0.0.1/2) for Internet access. Connectivity via OSPF/static routes assumed.
- **Requirements**:
  1. Configure R1’s interfaces as inside (G0/1) and outside (G0/0).
  2. Set up static NAT mappings (`ip nat inside source static`).
  3. Verify translations with `show ip nat translations` and `show ip nat statistics`.
  4. Test connectivity (e.g., PC1 pings 8.8.8.8).

**Analogy**: NAT is like a mail forwarding service. PC1’s private address (192.168.0.167) is like a home address, translated by R1 (post office) to a public PO box (100.0.0.1) for Internet delivery, then reversed for replies. Static NAT assigns each PC its own dedicated PO box.

**Lifelong Retention Tip**: Visualize PC1 (192.168.0.167) pinging 8.8.8.8 via R1, with NAT changing the source to 100.0.0.1. In Packet Tracer, configure `ip nat inside source static`, ping 8.8.8.8, verify with `show ip nat translations`. Create flashcards: “Private IP ranges? 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. Static NAT? One-to-one.” Practice 3 times daily for a week.

## Private IPv4 Addresses

**Purpose**: Enable non-unique IP addressing in internal networks, conserving public IPv4 addresses.

**Key Points**:
- **RFC 1918**: Defines private IPv4 ranges (not routable on the Internet):
  - **10.0.0.0/8**: 10.0.0.0–10.255.255.255 (1 Class A range).
  - **172.16.0.0/12**: 172.16.0.0–172.31.255.255 (16 Class B ranges).
  - **192.168.0.0/16**: 192.168.0.0–192.168.255.255 (256 Class C ranges).
- **Usage**: Freely used in internal networks; no global uniqueness required.
- **Internet Access**: Private IPs dropped by ISPs; require NAT for Internet communication.
- **Example**: PC1 (192.168.0.167) and default gateway (192.168.0.1) use private IPs, common in home networks.
- **IPv4 Scarcity**: Private IPs, CIDR, and NAT extend IPv4’s lifespan (IPv6 is long-term solution).

**Observations**:
- **Classless**: Ranges not tied to old Class A/B/C; use CIDR (e.g., /24 subnets within 192.168.0.0/16).
- **Duplicate IPs**: Multiple networks can use same private IPs (e.g., 192.168.0.167 in different homes).
- **Exam Note**: Memorize RFC 1918 ranges and their non-routability (topic 4.1).

**Comparison to Prior Content**:
- **IPv6 Lectures**: IPv6 solves address scarcity; private IPv4 and NAT are interim solutions.
- **Syslog Lab**: Private IPs used in lab networks (e.g., 192.168.0.0/24).
- **SSH Lecture**: Devices with private IPs (e.g., SW2: 192.168.2.253) use NAT for SSH access.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: NAT interacts with ports (e.g., UDP 53 for DNS in examples).
- **IP Addressing (Day 10)**: Private IPs contrast with public IPs requiring uniqueness.
- **Extended ACLs (Day 35)**: ACLs can block private IP traffic (e.g., `deny ip 192.168.0.0 0.0.255.255 any`).

**Analogy**: Private IPs are like internal office extensions (e.g., 192.168.0.167), usable within a company but not for external calls (Internet) without NAT’s forwarding service.

**Lifelong Retention Tip**: Memorize RFC 1918 ranges (10/8, 172.16/12, 192.168/16). Flashcard: “Private IP ranges? 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. Internet routable? No.” Practice 3 times.

## NAT Introduction

**Purpose**: Translate IP addresses to enable private IP hosts to access the Internet and resolve duplicate address issues.

**Key Points**:
- **Function**: Modifies source and/or destination IP addresses in packets.
- **Primary Use**: Allows private IP devices (e.g., 192.168.0.167) to communicate over the Internet using public IPs (e.g., 203.0.113.1).
- **Benefits**:
  - Enables Internet access for private IPs.
  - Allows multiple devices to share a single public IP (covered in Day 45).
- **Source NAT**: Translates source IP (e.g., private to public); focus for CCNA.
- **Example**:
  - PC1 (192.168.0.167) pings 8.8.8.8.
  - R1 translates source to 203.0.113.1 (G0/0 interface).
  - Server replies to 203.0.113.1; R1 reverses to 192.168.0.167.

**Demonstration**:
- PC1 sends packet: Source 192.168.0.167, Destination 8.8.8.8.
- R1 performs NAT: Source → 203.0.113.1.
- Server replies: Source 8.8.8.8, Destination 203.0.113.1.
- R1 reverses NAT: Destination → 192.168.0.167, forwards to PC1.

**Observations**:
- **Source NAT**: Changes source IP; reverse translation for replies.
- **Public IPs**: Must be unique, owned by organization (unlike private IPs).
- **Exam Note**: Understand NAT’s role in enabling private IP Internet access (topic 4.1).

**Comparison to Prior Content**:
- **IPv6 Lectures**: NAT unnecessary in IPv6 due to abundant addresses.
- **Syslog Lab**: NAT configs may trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: NAT enables private IP devices to SSH externally.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: NAT preserves port numbers in static NAT (e.g., UDP 53).
- **IP Addressing (Day 10)**: NAT bridges private and public IP realms.
- **Extended ACLs (Day 35)**: ACLs may need to permit NAT’d traffic (e.g., `permit ip host 100.0.0.1 any`).

**Analogy**: NAT is like a translator at a border, converting PC1’s private language (192.168.0.167) to a global one (203.0.113.1) for Internet communication, then back for replies.

**Lifelong Retention Tip**: In Packet Tracer, simulate PC1 pinging 8.8.8.8 with NAT on R1. Flashcard: “NAT purpose? Translate private to public IPs. Source NAT? Changes source IP.” Practice 3 times.

## Static NAT

**Purpose**: Configure one-to-one mappings of private (inside local) to public (inside global) IP addresses.

**Key Terms**:
- **Inside Local**: Actual IP of inside host (e.g., PC1: 192.168.0.167).
- **Inside Global**: Translated IP of inside host (e.g., 100.0.0.1).
- **Outside Local**: Outside host’s IP from inside perspective (e.g., 8.8.8.8).
- **Outside Global**: Outside host’s IP from outside perspective (e.g., 8.8.8.8, same unless destination NAT used).

**Demonstration**:
- PC1 (192.168.0.167) pings 8.8.8.8; R1 translates to 100.0.0.1.
- PC2 (192.168.0.168) pings 8.8.8.8; R1 translates to 100.0.0.2.
- Server replies to 100.0.0.1/2; R1 reverses to 192.168.0.167/168.

**Configuration (on R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# ip nat inside source static 192.168.0.167 100.0.0.1
R1(config)# ip nat inside source static 192.168.0.168 100.0.0.2
```

**Verification**:
- **Show IP NAT Translations**:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         192.168.0.167      ---                ---
--- 100.0.0.2         192.168.0.168      ---                ---
udp 100.0.0.1:5000    192.168.0.167:5000 8.8.8.8:53         8.8.8.8:53
udp 100.0.0.2:5001    192.168.0.168:5001 8.8.8.8:53         8.8.8.8:53
```
- **Show IP NAT Statistics**:
```plaintext
R1# show ip nat statistics
Total active translations: 2 (2 static, 0 dynamic; 0 extended)
Peak translations: 4
Outside interfaces: GigabitEthernet0/0
Inside interfaces: GigabitEthernet0/1
```
- **Clear Dynamic Translations**:
```plaintext
R1# clear ip nat translation *
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         192.168.0.167      ---                ---
--- 100.0.0.2         192.168.0.168      ---                ---
```

**Observations**:
- **Static NAT**: One-to-one mapping (e.g., 192.168.0.167 → 100.0.0.1); permanent in NAT table.
- **Dynamic Entries**: Added when NAT is used (e.g., UDP 53 for DNS); cleared with `clear ip nat translation *`.
- **Port Numbers**: Unchanged in static NAT; relevant for PAT (Day 45).
- **Limitations**: Doesn’t conserve public IPs (one per device); less efficient than PAT.
- **Exam Note**: Know `ip nat inside`, `ip nat outside`, `ip nat inside source static` (topic 4.1).

**Comparison to Prior Content**:
- **IPv6 Lectures**: Static NAT unnecessary in IPv6; used for IPv4 private IPs.
- **Syslog Lab**: NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: NAT enables private IP devices to SSH externally.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Static NAT preserves ports (e.g., UDP 53 in example).
- **IP Addressing (Day 10)**: Static NAT maps private to public IPs.
- **Extended ACLs (Day 35)**: ACLs must permit NAT’d traffic (e.g., `permit ip host 100.0.0.1 any`).

**Analogy**: Static NAT is like assigning each PC a unique PO box (100.0.0.1) at R1’s post office, ensuring one-to-one delivery to the Internet, unlike PAT’s shared box (Day 45).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip nat inside source static`, ping 8.8.8.8, check `show ip nat translations`. Flashcard: “Static NAT? One-to-one. Command? `ip nat inside source static`.” Practice 3 times.

## Common Issues and Troubleshooting

- **NAT Not Working**:
  - **Issue**: PC1 (192.168.0.167) can’t ping 8.8.8.8.
  - **Cause**: Missing `ip nat inside/outside` or incorrect mapping.
  - **Fix**: Verify `ip nat inside` (G0/1), `ip nat outside` (G0/0), `ip nat inside source static`.
- **Duplicate Public IP**:
  - **Issue**: Second mapping (`ip nat inside source static 10.0.0.2 20.0.0.1`) rejected.
  - **Cause**: Static NAT is one-to-one; 20.0.0.1 already mapped.
  - **Fix**: Use unique public IP (e.g., 20.0.0.2).
- **Dynamic Entries Persist**:
  - **Issue**: `show ip nat translations` shows old dynamic entries.
  - **Cause**: Active connections (e.g., UDP 53).
  - **Fix**: Run `clear ip nat translation *`; verify with `show ip nat translations`.
- **No Internet Access**:
  - **Issue**: PC1 can’t reach 8.8.8.8.
  - **Cause**: Routing issue or ACL blocking NAT’d traffic.
  - **Fix**: Check routes (`show ip route`), ACLs (`show access-lists`).

**Analogy**: Troubleshooting NAT is like ensuring a mail forwarding service (R1) uses correct addresses (inside local/global) and has clear delivery routes (no ACL blocks).

**Lifelong Retention Tip**: Simulate NAT failure (remove `ip nat inside`), retry, verify with `show ip nat translations`. Checklist: “No NAT? Check interfaces, mappings, routes, ACLs.” Practice 3 times.

## Practice Commands

- **Configure Static NAT (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# ip nat inside source static 192.168.0.167 100.0.0.1
R1(config)# ip nat inside source static 192.168.0.168 100.0.0.2
```
- **Verify NAT**:
```plaintext
R1# show ip nat translations
R1# show ip nat statistics
R1# clear ip nat translation *
```

## Quiz Answers and Explanations

1. **Which command configures static NAT for 192.168.10.10 to 203.0.113.10?**  
   - **A**: `ip nat outside source static 192.168.10.10 203.0.113.10`  
   - **B**: `ip nat inside source static 203.0.113.10 192.168.10.10`  
   - **C**: `ip nat inside 192.168.10.10 203.0.113.10`  
   - **D**: `ip nat inside source static 192.168.10.10 203.0.113.10`  
   - **Answer**: D (`ip nat inside source static` maps inside local to inside global).  

2. **What happens if you configure `ip nat inside source static 10.0.0.1 20.0.0.1`, then `ip nat inside source static 10.0.0.2 20.0.0.1`?**  
   - **A**: Both translated to 20.0.0.1  
   - **B**: Only 10.0.0.1 translated to 20.0.0.1  
   - **C**: Neither translated  
   - **D**: Error on first command  
   - **Answer**: B (Static NAT is one-to-one; second command rejected).  

3. **How many active translations remain after `clear ip nat translation *`? (3 static, 2 dynamic)**  
   - **A**: 0  
   - **B**: 3  
   - **C**: 2  
   - **D**: 5  
   - **Answer**: B (`clear ip nat translation *` removes dynamic entries; static remain).  

4. **Which are private IPv4 addresses? (Select all)**  
   - **A**: 10.254.255.0  
   - **B**: 172.15.255.1  
   - **C**: 192.167.255.1  
   - **D**: 203.0.113.1  
   - **E**: 172.20.2.3  
   - **F**: 10.11.12.13  
   - **Answer**: A, E, F (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16).  

5. **Identify addresses in PC1 (172.20.0.101) pinging 8.8.8.8 via R1 (NAT to 200.0.0.1):**  
   - **Outside global**: 8.8.8.8  
   - **Outside local**: 8.8.8.8  
   - **Inside local**: 172.20.0.101  
   - **Inside global**: 200.0.0.1  
   - **Answer**: Correct (outside local/global same without destination NAT; inside local/global reflect PC1’s IPs).

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: Which command configures static NAT?
    - **A**: `ip nat inside source static 192.168.1.1 100.0.0.1`  
    - **B**: `ip nat outside source static 192.168.1.1 100.0.0.1`  
    - **C**: `ip nat inside 192.168.1.1 100.0.0.1`  
    - **D**: `ip nat source static 192.168.1.1 100.0.0.1`  
    - **Answer**: A (`ip nat inside source static` is correct syntax).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Static NAT command? `ip nat inside source static`. Private IPs? 10/8, 172.16/12, 192.168/16”). Practice 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Private IPs: “10, 172.16, 192.168” (10/8, 172.16/12, 192.168/16).
   - NAT Terms: “Inside Local, Inside Global, Outside Local, Outside Global” (ILIGOLOG).
2. **Visualize Topology**:
   - Draw PC1 → R1 → 8.8.8.8 with NAT (192.168.0.167 → 100.0.0.1). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Private IPs? 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. Static NAT? One-to-one.”
4. **Practice in Packet Tracer**:
   - Configure `ip nat inside source static`, ping 8.8.8.8, verify (`show ip nat translations`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Static NAT (one-to-one) vs. PAT (many-to-one, Day 45).
   - Quiz: “Static NAT command? `ip nat inside source static`. Inside local? Private IP.”
6. **Troubleshooting Practice**:
   - Remove `ip nat inside`, retry ping, fix, verify. Checklist: “No NAT? Check interfaces, mappings, routes.”
7. **Teach Someone**:
   - Explain static NAT and terms (inside local/global) to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, IP ranges, NAT terms. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Private IPv4 Addresses**:
   - RFC 1918: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16; not Internet-routable.
2. **NAT Purpose**:
   - Translates private to public IPs for Internet access.
3. **Static NAT**:
   - One-to-one mapping (e.g., 192.168.0.167 → 100.0.0.1); permanent entries.
4. **Key Terms**:
   - Inside local (private IP), inside global (public IP), outside local/global (same without destination NAT).
5. **Commands**:
   - `ip nat inside`, `ip nat outside`, `ip nat inside source static`, `show ip nat translations`.
6. **CCNA Focus**:
   - Topic 4.1: Master static NAT configuration and verification.

These notes provide a complete foundation for the NAT Part 1 lecture, preparing you for CCNA exam topic 4.1. Practice static NAT in Packet Tracer, verify with `show ip nat translations`, and use analogies/retention tips to master NAT for life!