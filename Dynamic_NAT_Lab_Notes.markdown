# Dynamic NAT Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring **dynamic Network Address Translation (NAT)** and **Port Address Translation (PAT)** on a Cisco router (R1) to enable devices with private IP addresses to access the Internet, aligning with **CCNA exam topic 4.1** ("Configure and verify inside source NAT using static and pools"). Using Packet Tracer, it configures dynamic NAT with a pool of two public IPs (100.0.0.1–2) for three PCs (172.16.0.1–3), demonstrating NAT pool exhaustion, then transitions to PAT using R1’s interface IP (203.0.113.1). The lab verifies functionality through pings to 8.8.8.8 and google.com, using `show ip nat translations` and `clear ip nat translation *`. It reinforces concepts from the NAT Part 2 lecture.

**Lab Objectives**:
- **Configure Dynamic NAT**: Set up inside/outside interfaces, ACL, pool, and dynamic NAT.
- **Test Dynamic NAT**: Verify connectivity and NAT pool exhaustion (only two IPs available).
- **Configure PAT**: Transition to PAT using R1’s interface IP with `overload`.
- **Verify NAT/PAT**: Confirm translations with `show ip nat translations`.
- **Prepare for Exam**: Master dynamic NAT/PAT configuration, verification, and pool exhaustion (topic 4.1).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|                |-------|    Server      |
| 172.16.0.1     |       |      R1        |       |   8.8.8.8      |
|                |       | G0/1: Inside   |       |   (DNS)        |
+----------------+       | G0/0: 203.0.113.1 |   +----------------+
         |              +----------------+
         |                       |
+----------------+       +----------------+
|      PC2       |       |      PC3       |
| 172.16.0.2     |       | 172.16.0.3     |
+----------------+       +----------------+
```
- **Devices**:
  - **PC1**: 172.16.0.1, NAT to 100.0.0.1 (dynamic) or 203.0.113.1 (PAT), DNS server 8.8.8.8.
  - **PC2**: 172.16.0.2, NAT to 100.0.0.2 (dynamic) or 203.0.113.1 (PAT), DNS server 8.8.8.8.
  - **PC3**: 172.16.0.3, NAT fails (pool exhaustion) or 203.0.113.1 (PAT), DNS server 8.8.8.8.
  - **R1**: Router performing dynamic NAT/PAT (G0/1 inside, G0/0 outside).
  - **Server**: DNS server at 8.8.8.8 (resolves google.com to 142.250.190.78).
- **Note**: R1 uses ACL to identify 172.16.0.0/24 for translation to pool (100.0.0.1–2) or G0/0 IP (203.0.113.1). Connectivity to 8.8.8.8 via OSPF/static routes assumed. PCs use 8.8.8.8 as DNS server.
- **Requirements**:
  1. Configure dynamic NAT with ACL and pool (100.0.0.1–2).
  2. Test pings to 8.8.8.8/google.com; observe pool exhaustion (PC3 fails).
  3. Reconfigure for PAT using G0/0 IP (203.0.113.1).
  4. Test pings from all PCs (all succeed).
  5. Verify with `show ip nat translations`, clear dynamic translations.

**Analogy**: Dynamic NAT is like a post office (R1) assigning temporary PO boxes (100.0.0.1–2) to PCs from a limited stock, causing delays (pool exhaustion) for PC3. PAT is like using one shared PO box (203.0.113.1) with unique slot numbers (ports) for all PCs, ensuring everyone gets service.

**Lifelong Retention Tip**: Visualize PC1–3 pinging google.com via R1, with NAT (two IPs) failing for PC3, then PAT (one IP, ports) succeeding. In Packet Tracer, configure `ip nat inside source list 1 interface g0/0 overload`, ping google.com, verify with `show ip nat translations`. Create flashcards: “Dynamic NAT? ACL+pool, one-to-one. PAT? `overload`, many-to-one.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure Dynamic NAT on R1
**Objective**: Set up dynamic NAT to translate 172.16.0.0/24 to a pool of two IPs (100.0.0.1–2).

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
2. Configure ACL and NAT pool:
```plaintext
R1(config)# access-list 1 permit 172.16.0.0 0.0.0.255
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1
R1(config)# exit
```

**Observations**:
- **Interfaces**: G0/0 (Internet, 203.0.113.1) is `outside`; G0/1 (PCs) is `inside`.
- **ACL**: Permits 172.16.0.0/24 for translation; deny = no translation (not dropped).
- **Pool**: Limited to two IPs (100.0.0.1–2); prone to exhaustion.
- **NAT Statement**: Maps ACL 1 to POOL1 for dynamic one-to-one translations.
- **Exam Note**: Know `access-list`, `ip nat pool`, `ip nat inside source list` for dynamic NAT (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 2 Lecture**: Lab mirrors dynamic NAT config (ACL+pool, one-to-one).
- **Static NAT Lab**: Dynamic NAT automates mappings vs. static’s manual mappings.
- **Syslog Lab**: NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACLs identify NAT traffic, not block.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Dynamic NAT preserves ports (e.g., UDP 53 for DNS).
- **IP Addressing (Day 10)**: Maps private (172.16.0.0/12) to public IPs (100.0.0.0/24).
- **Extended ACLs (Day 35)**: NAT ACL uses wildcard mask (0.0.0.255).

**Analogy**: Configuring dynamic NAT is like setting up a post office (R1) to assign two temporary PO boxes (100.0.0.1–2) to approved senders (ACL 1) for Internet mail.

**Lifelong Retention Tip**: In Packet Tracer, configure dynamic NAT, verify with `show running-config`. Flashcard: “Dynamic NAT command? `ip nat inside source list 1 pool POOL1`. ACL role? Permit = translate.” Practice 3 times.

### Step 2: Test Dynamic NAT and Observe Pool Exhaustion
**Objective**: Verify connectivity for PC1/PC2, observe NAT pool exhaustion for PC3.

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
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Ping statistics: Sent = 4, Received = 0, Lost = 4 (100% loss)
```
4. On R1, check translations:
```plaintext
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
udp 100.0.0.1:1024    172.16.0.1:1024    8.8.8.8:53         8.8.8.8:53
udp 100.0.0.2:1025    172.16.0.2:1025    8.8.8.8:53         8.8.8.8:53
icmp 100.0.0.1:1      172.16.0.1:1       8.8.8.8:1          8.8.8.8:1
icmp 100.0.0.1:2      172.16.0.1:2       142.250.190.78:2   142.250.190.78:2
icmp 100.0.0.2:3      172.16.0.2:3       142.250.190.78:3   142.250.190.78:3
```

**Observations**:
- **PC1/PC2**: Pings succeed (100.0.0.1, 100.0.0.2); DNS resolves google.com to 142.250.190.78.
- **PC3**: Pings fail due to NAT pool exhaustion (only two IPs available).
- **Translations**: Dynamic entries for DNS (UDP 53) and pings (ICMP); no one-to-one mappings in Packet Tracer (unlike lecture).
- **Packet Tracer Note**: Differs from Cisco IOS; lacks simple inside local-to-global mappings.
- **Exam Note**: Understand pool exhaustion (packets dropped) and dynamic entries (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 2 Lecture**: Lab confirms dynamic NAT (ACL+pool) and pool exhaustion.
- **Static NAT Lab**: Dynamic NAT automates vs. static’s manual mappings.
- **Syslog Lab**: Pings, NAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACL identifies NAT traffic (permit = translate).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DNS uses UDP 53; pings use ICMP; dynamic NAT preserves ports.
- **IP Addressing (Day 10)**: Maps private (172.16.0.0/12) to public (100.0.0.0/24).
- **Extended ACLs (Day 35)**: NAT ACL uses wildcard mask (0.0.0.255).

**Analogy**: Testing dynamic NAT is like sending mail from PCs through R1’s post office, with only two PO boxes (100.0.0.1–2) available, leaving PC3 waiting (dropped packets).

**Lifelong Retention Tip**: In Packet Tracer, ping google.com from PC1–3, note PC3’s failure. Flashcard: “Dynamic NAT issue? Pool exhaustion = dropped packets. ACL? Permit 172.16.0.0 0.0.0.255.” Practice 3 times.

### Step 3: Configure PAT Using R1’s Interface IP
**Objective**: Replace dynamic NAT with PAT, using G0/0’s IP (203.0.113.1) for all PCs.

**Steps (on R1)**:
1. Clear translations:
```plaintext
R1# clear ip nat translation *
```
2. Verify running-config (optional):
```plaintext
R1# show running-config | include nat
 ip nat inside source list 1 pool POOL1
```
3. Configure PAT:
```plaintext
R1(config)# ip nat inside source list 1 interface g0/0 overload
```
4. Verify running-config (optional):
```plaintext
R1# show running-config | include nat
 ip nat inside source list 1 interface g0/0 overload
```

**Observations**:
- **Clearing**: Removes dynamic entries; static pool remains unless deleted.
- **PAT Config**: Replaces pool-based NAT; `overload` enables many-to-one translation.
- **Interface-Based**: Uses G0/0’s IP (203.0.113.1); no pool needed.
- **ACL Reuse**: Same ACL (172.16.0.0/24) applies.
- **Exam Note**: Know `ip nat inside source list 1 interface g0/0 overload` for PAT (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 2 Lecture**: Lab mirrors interface-based PAT (203.0.113.1, ports).
- **Static NAT Lab**: PAT is many-to-one vs. static’s one-to-one.
- **Syslog Lab**: PAT config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACL reused for PAT (permit = translate).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: PAT relies on unique ports (e.g., UDP 53).
- **IP Addressing (Day 10)**: PAT conserves public IPs (one IP for all).
- **Extended ACLs (Day 35)**: ACL permits PAT traffic (0.0.0.255 wildcard).

**Analogy**: Configuring PAT is like switching R1’s post office to one shared PO box (203.0.113.1) with unique slot numbers (ports) for all PCs’ mail.

**Lifelong Retention Tip**: In Packet Tracer, configure PAT, verify with `show running-config`. Flashcard: “PAT command? `ip nat inside source list 1 interface g0/0 overload`. Pool needed? No.” Practice 3 times.

### Step 4: Test PAT and Verify Translations
**Objective**: Verify all PCs can ping google.com using PAT; check translations.

**Steps**:
1. On PC1:
```plaintext
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
udp 203.0.113.1:1024  172.16.0.1:1024    8.8.8.8:53         8.8.8.8:53
udp 203.0.113.1:1025  172.16.0.2:1025    8.8.8.8:53         8.8.8.8:53
udp 203.0.113.1:1026  172.16.0.3:1026    8.8.8.8:53         8.8.8.8:53
icmp 203.0.113.1:1    172.16.0.1:1       142.250.190.78:1   142.250.190.78:1
icmp 203.0.113.1:2    172.16.0.2:2       142.250.190.78:2   142.250.190.78:2
icmp 203.0.113.1:3    172.16.0.3:3       142.250.190.78:3   142.250.190.78:3
```

**Observations**:
- **Pings**: All PCs succeed (203.0.113.1, unique ports); no pool exhaustion.
- **Translations**: All PCs share 203.0.113.1; DNS (UDP 53), pings (ICMP) use unique ports (1024–1026).
- **Outside Local/Global**: Same (8.8.8.8, 142.250.190.78); no destination NAT.
- **Packet Tracer Note**: No one-to-one mappings; only extended entries (UDP/ICMP).
- **Exam Note**: Verify PAT uses one IP with ports; know `show ip nat translations` (topic 4.1).

**Comparison to Prior Content**:
- **NAT Part 2 Lecture**: Lab confirms interface-based PAT (203.0.113.1, ports).
- **Static NAT Lab**: PAT is many-to-one vs. static’s one-to-one.
- **Syslog Lab**: Pings, PAT configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **ACL Lecture (Day 35)**: ACL reused for PAT (permit = translate).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: PAT uses unique ports (1024–1026) for tracking.
- **IP Addressing (Day 10)**: PAT conserves public IPs (one IP for all).
- **Extended ACLs (Day 35)**: ACL permits PAT traffic (0.0.0.255 wildcard).

**Analogy**: Testing PAT is like sending mail from all PCs through one shared PO box (203.0.113.1), with R1 tracking each PC’s mail via unique slot numbers (ports).

**Lifelong Retention Tip**: In Packet Tracer, ping google.com from PC1–3, check `show ip nat translations` for shared IP (203.0.113.1). Flashcard: “PAT? One IP, unique ports. Command? `ip nat inside source list 1 interface g0/0 overload`.” Practice 3 times.

## Common Issues and Troubleshooting

- **NAT Pool Exhaustion**:
  - **Issue**: PC3 can’t ping google.com.
  - **Cause**: Pool (100.0.0.1–2) full; only PC1/PC2 get IPs.
  - **Fix**: Clear translations (`clear ip nat translation *`) or use PAT.
- **PAT Fails**:
  - **Issue**: No PCs can ping google.com.
  - **Cause**: Missing `overload`, incorrect ACL, or interface misconfiguration.
  - **Fix**: Verify `ip nat inside source list 1 interface g0/0 overload`, ACL (`show access-lists`).
- **Incorrect ACL**:
  - **Issue**: Packets not translated.
  - **Cause**: ACL denies traffic (e.g., `deny ip 172.16.0.0 0.0.0.255`).
  - **Fix**: Use `permit ip 172.16.0.0 0.0.0.255`, check `show access-lists`.
- **Pool Misconfiguration**:
  - **Issue**: `ip nat pool` rejected.
  - **Cause**: Start/end IPs not in same subnet.
  - **Fix**: Use valid range (e.g., 100.0.0.1–2, netmask 255.255.255.0).
- **No Internet Access**:
  - **Issue**: PCs can’t reach 8.8.8.8.
  - **Cause**: Routing, ARP delay, or ACL blocking NAT’d traffic.
  - **Fix**: Check routes (`show ip route`), wait for ARP, check ACLs (`show access-lists`).

**Analogy**: Troubleshooting PAT is like ensuring R1’s shared PO box (203.0.113.1) assigns correct slot numbers (ports) and has clear delivery routes (no ACL blocks).

**Lifelong Retention Tip**: Simulate PAT failure (omit `overload`), retry, fix, verify. Checklist: “No PAT? Check ACL, overload, interfaces, routes, ARP.” Practice 3 times.

## Practice Commands

- **Dynamic NAT (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
R1(config)# interface g0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# access-list 1 permit 172.16.0.0 0.0.0.255
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
R1(config)# ip nat inside source list 1 pool POOL1
```
- **PAT (Interface-Based)**:
```plaintext
R1# clear ip nat translation *
R1(config)# ip nat inside source list 1 interface g0/0 overload
```
- **Verify NAT/PAT**:
```plaintext
R1# show ip nat translations
R1# show running-config | include nat
R1# clear ip nat translation *
```
- **Test Connectivity (PC1–3)**:
```plaintext
PC1> ping 8.8.8.8
PC1> ping google.com
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - NAT Types: “Static, Dynamic, PAT” (SDP).
   - PAT: “Overload = Many-to-one, Ports” (OMP).
2. **Visualize Topology**:
   - Draw PC1–3 → R1 → 8.8.8.8 with PAT (203.0.113.1, unique ports). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Dynamic NAT? ACL+pool, one-to-one. PAT? `overload`, one IP, ports. Pool exhaustion? Drop.”
4. **Practice in Packet Tracer**:
   - Configure dynamic NAT, observe PC3 failure, switch to PAT, verify. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Dynamic NAT (one-to-one, pool) vs. PAT (many-to-one, ports).
   - Quiz: “PAT command? `ip nat inside source list 1 interface g0/0 overload`. Pool exhaustion? Drop.”
6. **Troubleshooting Practice**:
   - Use wrong ACL, retry with `permit ip 172.16.0.0 0.0.0.255`, verify. Checklist: “No NAT? Check ACL, pool, overload.”
7. **Teach Someone**:
   - Explain dynamic NAT vs. PAT to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, NAT types, ACL usage. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Dynamic NAT**:
   - One-to-one, temporary, uses ACL and pool (100.0.0.1–2); prone to exhaustion.
2. **PAT**:
   - Many-to-one, uses one IP (203.0.113.1) with unique ports; no exhaustion.
3. **Configuration**:
   - Dynamic NAT: `ip nat inside source list 1 pool POOL1`.
   - PAT: `ip nat inside source list 1 interface g0/0 overload`.
4. **Verification**:
   - `show ip nat translations` (dynamic/extended entries), `show running-config | include nat`.
5. **Pool Exhaustion**:
   - Dynamic NAT drops packets when pool is full; PAT avoids this.
6. **CCNA Focus**:
   - Topic 4.1: Master dynamic NAT/PAT config, ACL, pool exhaustion.

These notes provide a complete foundation for the Dynamic NAT lab, reinforcing CCNA exam topic 4.1. Practice dynamic NAT/PAT in Packet Tracer, verify with `show ip nat translations`, and use analogies/retention tips to master NAT for life!