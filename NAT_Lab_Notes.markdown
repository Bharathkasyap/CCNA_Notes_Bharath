# NAT Lab Notes for CCNA Beginners

## Lab Overview

This lab demonstrates configuring **static Network Address Translation (NAT)** on a Cisco router (R1) to enable devices with private IP addresses to access the Internet, aligning with **CCNA exam topic 4.1** ("Configure and verify inside source NAT using static and pools"). Using Packet Tracer, it configures static NAT mappings for three PCs (PC1, PC2, PC3) to translate their private IPs (172.16.0.1–3) to public IPs (100.0.0.1–3), allowing connectivity to a DNS server (8.8.8.8) and google.com. The lab verifies NAT functionality through pings, `show ip nat translations`, and `show ip nat statistics`, and demonstrates clearing dynamic translations. It reinforces concepts from the NAT Part 1 lecture.

**Lab Objectives**:
- **Test Connectivity Without NAT**: Verify PCs with private IPs cannot ping 8.8.8.8.
- **Configure Static NAT**: Set up inside/outside interfaces and one-to-one NAT mappings on R1.
- **Verify NAT**: Confirm translations and statistics using `show` commands.
- **Test Connectivity With NAT**: Ping 8.8.8.8 and google.com from PCs.
- **Clear Dynamic Translations**: Use `clear ip nat translation *` and verify static entries remain.
- **Prepare for Exam**: Master static NAT configuration and verification (topic 4.1).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|                |-------|    Server      |
| 172.16.0.1     |       |      R1        |       |   8.8.8.8      |
|                |       | G0/1: Inside   |       |   (DNS)        |
+----------------+       | G0/0: Outside  |       +----------------+
         |              +----------------+
         |                       |
+----------------+       +----------------+
|      PC2       |       |      PC3       |
| 172.16.0.2     |       | 172.16.0.3     |
+----------------+       +----------------+
```
- **Devices**:
  - **PC1**: 172.16.0.1, NAT to 100.0.0.1, DNS server 8.8.8.8.
  - **PC2**: 172.16.0.2, NAT to 100.0.0.2, DNS server 8.8.8.8.
  - **PC3**: 172.16.0.3, NAT to 100.0.0.3, DNS server 8.8.8.8.
  - **R1**: Router performing static NAT (G0/1 inside, G0/0 outside).
  - **Server**: DNS server at 8.8.8.8 (resolves google.com).
- **Note**: R1 translates private IPs (172.16.0.1–3) to public IPs (100.0.0.1–3). Connectivity to 8.8.8.8 via OSPF/static routes assumed. PCs use 8.8.8.8 as DNS server.
- **Requirements**:
  1. Ping 8.8.8.8 from PC1 (fails without NAT).
  2. Configure static NAT on R1 (`ip nat inside source static`).
  3. Verify with `show ip nat translations`, `show ip nat statistics`.
  4. Ping 8.8.8.8 and google.com from PC1, PC2, PC3 (succeeds with NAT).
  5. Clear dynamic translations (`clear ip nat translation *`).

**Analogy**: Static NAT is like a post office (R1) assigning each PC a unique public PO box (100.0.0.1–3) for sending mail (packets) to the Internet (8.8.8.8), ensuring replies return to the correct private address (172.16.0.1–3).

**Lifelong Retention Tip**: Visualize PC1 (172.16.0.1) pinging 8.8.8.8 via R1, with NAT to 100.0.0.1. In Packet Tracer, configure `ip nat inside source static`, ping google.com, verify with `show ip nat translations`. Create flashcards: “Static NAT command? `ip nat inside source static`. Private IP range? 172.16.0.0/12.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Test Connectivity Without NAT
**Objective**: Verify PCs with private IPs cannot ping 8.8.8.8 due to ISP dropping private IP traffic.

**Steps (on PC1)**:
1. Ping DNS server:
```plaintext
PC1> ping 8.8.8.8
Pinging 8.8.8.8 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Ping statistics: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Observations**:
- **Result**: All pings fail; ISP drops private IP (172.16.0.1) traffic.
- **Reason**: Private IPs (RFC 1918, 172.16.0.0/12) are not routable on the Internet.
- **Exam Note**: Understand private IPs require NAT for Internet access (topic 4.1).

**Comparison to Prior Content**:
- **NAT Lecture**: Confirms private IPs (172.16.0.0/12) need NAT to reach 8.8.8.8.
- **IPv6 Lectures**: IPv6 avoids NAT; private IPv4 relies on it.
- **Syslog Lab**: Ping failures may trigger Syslog on R1 (e.g., `%ICMP-3-NO_ROUTE`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Pings use ICMP; NAT preserves ports in static NAT.
- **IP Addressing (Day 10)**: Private IPs (172.16.0.0/12) vs. public IPs (100.0.0.0/24).
- **Extended ACLs (Day 35)**: ISP’s ACLs drop private IPs (e.g., `deny ip 172.16.0.0 0.15.255.255 any`).

**Analogy**: Without NAT, PC1’s private address (172.16.0.1) is like an internal office extension, unusable for external calls (Internet) and blocked by the ISP’s gatekeeper.

**Lifelong Retention Tip**: In Packet Tracer, ping 8.8.8.8 from PC1, observe timeout. Flashcard: “Private IPs on Internet? Dropped. Solution? NAT.” Practice 3 times.

### Step 2: Configure Static NAT on R1
**Objective**: Configure inside/outside interfaces and static NAT mappings for PC1, PC2, PC3.

**Steps (on R1)**:
1. Configure interfaces:
```plaintext
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
```
2. Configure static NAT mappings:
```plaintext
R1(config)# ip nat inside source static 172.16.0.1 100.0.0.1
R1(config)# ip nat inside source static 172.16.0.2 100.0.0.2
R1(config)# ip nat inside source static 172.16.0.3 100.0.0.3
R1(config)# exit
```

**Observations**:
- **Interfaces**: G0/0 (to Internet) is `outside`; G0/1 (to PCs) is `inside`.
- **Mappings**: One-to-one (e.g., 172.16.0.1 → 100.0.0.1); public IPs must be owned.
- **Static NAT**: Permanent entries in NAT table; no timeout.
- **Exam Note**: Know `ip nat inside`, `ip nat outside`, `ip nat inside source static` (topic 4.1).

**Comparison to Prior Content**:
- **NAT Lecture**: Lab mirrors lecture’s static NAT config (one-to-one mappings).
- **Syslog Lab**: NAT config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: NAT enables private IP devices to SSH externally via public IPs.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Static NAT preserves ports (e.g., ICMP for pings).
- **IP Addressing (Day 10)**: Maps private (172.16.0.0/12) to public IPs (100.0.0.0/24).
- **Extended ACLs (Day 35)**: ACLs must permit NAT’d traffic (e.g., `permit ip host 100.0.0.1 any`).

**Analogy**: Configuring NAT on R1 is like setting up a post office to assign each PC (172.16.0.1–3) a unique public PO box (100.0.0.1–3) for Internet mail delivery.

**Lifelong Retention Tip**: In Packet Tracer, configure `ip nat inside source static`, verify with `show running-config`. Flashcard: “Static NAT command? `ip nat inside source static`. Inside interface? `ip nat inside`.” Practice 3 times.

### Step 3: Verify NAT Configuration
**Objective**: Confirm static NAT mappings and statistics before testing connectivity.

**Steps (on R1)**:
1. Check NAT translations:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         172.16.0.1         ---                ---
--- 100.0.0.2         172.16.0.2         ---                ---
--- 100.0.0.3         172.16.0.3         ---                ---
```
2. Check NAT statistics:
```plaintext
R1# show ip nat statistics
Total active translations: 3 (3 static, 0 dynamic; 0 extended)
Peak translations: 3
Outside interfaces: GigabitEthernet0/0
Inside interfaces: GigabitEthernet0/1
```

**Observations**:
- **Translations**: Three static entries (172.16.0.1–3 → 100.0.0.1–3); no dynamic entries yet.
- **Statistics**: Confirms 3 static translations, G0/0 (outside), G0/1 (inside).
- **Exam Note**: Know `show ip nat translations`, `show ip nat statistics` for verification (topic 4.1).

**Comparison to Prior Content**:
- **NAT Lecture**: Lab verifies lecture’s static NAT table (permanent entries).
- **Syslog Lab**: `show` commands may trigger Syslog if logging enabled.
- **SSH Lecture**: NAT verification ensures SSH access to public IPs.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: No ports in static entries until traffic (e.g., UDP 53).
- **IP Addressing (Day 10)**: Verifies private-to-public mappings.
- **Extended ACLs (Day 35)**: Ensure ACLs allow NAT traffic (`permit ip 100.0.0.0 0.0.0.255 any`).

**Analogy**: Verifying NAT is like checking the post office’s forwarding list to ensure each PC’s private address is correctly mapped to a public PO box.

**Lifelong Retention Tip**: Run `show ip nat translations`, note static entries. Flashcard: “Verify NAT? `show ip nat translations`. Static entries? Permanent.” Practice 3 times.

### Step 4: Test Connectivity With NAT
**Objective**: Verify PCs can ping 8.8.8.8 and google.com using static NAT; check translations.

**Steps**:
1. On PC1:
```plaintext
PC1> ping 8.8.8.8
Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=10ms TTL=255
Reply from 8.8.8.8: bytes=32 time=8ms TTL=255
Reply from 8.8.8.8: bytes=32 time=9ms TTL=255
Reply from 8.8.8.8: bytes=32 time=11ms TTL=255
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
PC1> ping google.com
Pinging google.com [142.250.190.78] with 32 bytes of data:
Reply from 142.250.190.78: bytes=32 time=12ms TTL=254
Reply from 142.250.190.78: bytes=32 time=10ms TTL=254
Reply from 142.250.190.78: bytes=32 time=11ms TTL=254
Reply from 142.250.190.78: bytes=32 time=9ms TTL=254
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
2. On PC2:
```plaintext
PC2> ping google.com
Pinging google.com [142.250.190.78] with 32 bytes of data:
Reply from 142.250.190.78: bytes=32 time=13ms TTL=254
Reply from 142.250.190.78: bytes=32 time=11ms TTL=254
Reply from 142.250.190.78: bytes=32 time=10ms TTL=254
Reply from 142.250.190.78: bytes=32 time=12ms TTL=254
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
3. On PC3:
```plaintext
PC3> ping google.com
Pinging google.com [142.250.190.78] with 32 bytes of data:
Reply from 142.250.190.78: bytes=32 time=11ms TTL=254
Reply from 142.250.190.78: bytes=32 time=9ms TTL=254
Reply from 142.250.190.78: bytes=32 time=12ms TTL=254
Reply from 142.250.190.78: bytes=32 time=10ms TTL=254
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
4. On R1, check translations:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         172.16.0.1         ---                ---
--- 100.0.0.2         172.16.0.2         ---                ---
--- 100.0.0.3         172.16.0.3         ---                ---
udp 100.0.0.1:1024    172.16.0.1:1024    8.8.8.8:53         8.8.8.8:53
udp 100.0.0.2:1025    172.16.0.2:1025    8.8.8.8:53         8.8.8.8:53
udp 100.0.0.3:1026    172.16.0.3:1026    8.8.8.8:53         8.8.8.8:53
icmp 100.0.0.1:1      172.16.0.1:1       8.8.8.8:1          8.8.8.8:1
icmp 100.0.0.1:2      172.16.0.1:2       142.250.190.78:2   142.250.190.78:2
icmp 100.0.0.2:3      172.16.0.2:3       142.250.190.78:3   142.250.190.78:3
icmp 100.0.0.3:4      172.16.0.3:4       142.250.190.78:4   142.250.190.78:4
```

**Observations**:
- **Pings**: Succeed post-NAT (100.0.0.1–3); DNS resolves google.com to 142.250.190.78.
- **Translations**: Static entries (3) plus dynamic entries for DNS (UDP 53) and pings (ICMP).
- **Outside Local/Global**: Same (8.8.8.8, 142.250.190.78); no destination NAT.
- **Exam Note**: Verify NAT enables Internet access; know dynamic entries (topic 4.1).

**Comparison to Prior Content**:
- **NAT Lecture**: Lab confirms static NAT (172.16.0.1 → 100.0.0.1) and dynamic entries (DNS, ICMP).
- **Syslog Lab**: Pings, NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: NAT enables private IPs to access external services (e.g., DNS).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DNS uses UDP 53; pings use ICMP; static NAT preserves ports.
- **IP Addressing (Day 10)**: Private (172.16.0.0/12) to public (100.0.0.0/24) mappings.
- **Extended ACLs (Day 35)**: ACLs must permit NAT’d traffic (`permit ip 100.0.0.0 0.0.0.255 any`).

**Analogy**: Testing NAT is like sending mail from PCs through R1’s post office, confirming delivery to 8.8.8.8 and google.com using public PO boxes (100.0.0.1–3).

**Lifelong Retention Tip**: In Packet Tracer, ping google.com from PC1–3, check `show ip nat translations` for UDP/ICMP entries. Flashcard: “NAT pings work? Private to public mapping. DNS port? UDP 53.” Practice 3 times.

### Step 5: Clear Dynamic Translations
**Objective**: Clear dynamic NAT entries and verify static entries remain.

**Steps (on R1)**:
1. Clear translations:
```plaintext
R1# clear ip nat translation *
```
2. Verify translations:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 100.0.0.1         172.16.0.1         ---                ---
--- 100.0.0.2         172.16.0.2         ---                ---
--- 100.0.0.3         172.16.0.3         ---                ---
```

**Observations**:
- **Result**: Dynamic entries (UDP, ICMP) removed; static entries remain.
- **Timeout**: Dynamic entries would timeout naturally; `clear ip nat translation *` forces removal.
- **Exam Note**: Know `clear ip nat translation *` affects only dynamic entries (topic 4.1).

**Comparison to Prior Content**:
- **NAT Lecture**: Lab confirms static entries are permanent; dynamic entries cleared.
- **Syslog Lab**: Clearing translations may trigger Syslog (e.g., `%NAT-6-DELETE`).
- **SSH Lecture**: NAT ensures SSH access persists via static mappings.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Clearing removes UDP/ICMP entries; static NAT port-independent.
- **IP Addressing (Day 10)**: Static mappings (172.16.0.1–3 → 100.0.0.1–3) persist.
- **Extended ACLs (Day 35)**: Ensure ACLs don’t block cleared traffic.

**Analogy**: Clearing dynamic translations is like removing temporary forwarding notes from R1’s post office log, keeping only permanent PC-to-PO-box mappings.

**Lifelong Retention Tip**: Run `clear ip nat translation *`, verify static entries with `show ip nat translations`. Flashcard: “Clear NAT? `clear ip nat translation *`. Static entries? Stay.” Practice 3 times.

## Common Issues and Troubleshooting

- **Ping Fails Post-NAT**:
  - **Issue**: PC1 can’t ping 8.8.8.8 after NAT.
  - **Cause**: Missing `ip nat inside/outside` or incorrect mapping.
  - **Fix**: Verify `ip nat inside` (G0/1), `ip nat outside` (G0/0), `ip nat inside source static 172.16.0.1 100.0.0.1`.
- **DNS Resolution Fails**:
  - **Issue**: PC1 can’t ping google.com.
  - **Cause**: Incorrect DNS server (not 8.8.8.8) or NAT issue.
  - **Fix**: Check PC1’s DNS (8.8.8.8), verify NAT translations (`show ip nat translations`).
- **Duplicate Public IP**:
  - **Issue**: Second mapping (`ip nat inside source static 172.16.0.2 100.0.0.1`) rejected.
  - **Cause**: Static NAT is one-to-one; 100.0.0.1 used.
  - **Fix**: Use unique public IP (100.0.0.2).
- **Dynamic Entries Persist**:
  - **Issue**: Dynamic entries remain after traffic stops.
  - **Cause**: Active connections (e.g., UDP 53).
  - **Fix**: Run `clear ip nat translation *`; verify with `show ip nat translations`.
- **No Internet Access**:
  - **Issue**: PCs can’t reach 8.8.8.8.
  - **Cause**: Routing issue or ACL blocking NAT’d traffic.
  - **Fix**: Check routes (`show ip route`), ACLs (`show access-lists`).

**Analogy**: Troubleshooting NAT is like ensuring R1’s post office correctly forwards PC mail (172.16.0.1–3) to public addresses (100.0.0.1–3) with clear delivery routes.

**Lifelong Retention Tip**: Simulate NAT failure (remove `ip nat inside`), retry ping, fix, verify. Checklist: “No ping? Check interfaces, mappings, DNS, routes, ACLs.” Practice 3 times.

## Practice Commands

- **Configure Static NAT (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# ip nat inside source static 172.16.0.1 100.0.0.1
R1(config)# ip nat inside source static 172.16.0.2 100.0.0.2
R1(config)# ip nat inside source static 172.16.0.3 100.0.0.3
R1(config)# exit
```
- **Verify NAT**:
```plaintext
R1# show ip nat translations
R1# show ip nat statistics
R1# clear ip nat translation *
```
- **Test Connectivity (PC1–3)**:
```plaintext
PC1> ping 8.8.8.8
PC1> ping google.com
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Private IPs: “10, 172.16, 192.168” (10/8, 172.16/12, 192.168/16).
   - NAT Terms: “Inside Local, Inside Global, Outside Local, Outside Global” (ILIGOLOG).
2. **Visualize Topology**:
   - Draw PC1–3 → R1 → 8.8.8.8 with NAT (172.16.0.1–3 → 100.0.0.1–3). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Static NAT? One-to-one. Command? `ip nat inside source static`. DNS port? UDP 53.”
4. **Practice in Packet Tracer**:
   - Configure NAT, ping google.com, verify (`show ip nat translations`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Static NAT (one-to-one) vs. PAT (many-to-one, Day 45).
   - Quiz: “Static NAT command? `ip nat inside source static`. Clear dynamic? `clear ip nat translation *`.”
6. **Troubleshooting Practice**:
   - Remove `ip nat inside`, retry ping, fix, verify. Checklist: “No NAT? Check interfaces, mappings, DNS.”
7. **Teach Someone**:
   - Explain static NAT and pings to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, IP ranges, NAT terms. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Static NAT**:
   - One-to-one mapping (e.g., 172.16.0.1 → 100.0.0.1); enables private IP Internet access.
2. **Configuration**:
   - `ip nat inside`, `ip nat outside`, `ip nat inside source static`.
3. **Verification**:
   - `show ip nat translations` (static/dynamic entries), `show ip nat statistics`.
4. **Testing**:
   - Pings to 8.8.8.8, google.com confirm NAT functionality (DNS: UDP 53, ICMP).
5. **Clearing**:
   - `clear ip nat translation *` removes dynamic entries; static remain.
6. **CCNA Focus**:
   - Topic 4.1: Master static NAT config, verification, and private IP ranges.

These notes provide a complete foundation for the NAT lab, reinforcing CCNA exam topic 4.1. Practice static NAT in Packet Tracer, verify with `show ip nat translations`, and use analogies/retention tips to master NAT for life!