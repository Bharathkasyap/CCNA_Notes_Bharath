# First Hop Redundancy Protocols (FHRP) Lecture Notes for CCNA Beginners

## Introduction

This lecture covers First Hop Redundancy Protocols (FHRPs) for the CCNA exam (topic 3.5). FHRPs are not a single protocol but a category of protocols designed to provide default gateway redundancy. The lecture introduces the purpose of FHRPs, details three protocols (HSRP, VRRP, GLBP), and provides basic HSRP configuration for hands-on understanding, even though configuration is not required for the exam. The focus is on understanding concepts and comparing protocols, with practical examples to reinforce learning.

**Lecture Objectives**:
- Understand the **purpose** of FHRPs: providing a redundant default gateway.
- Learn the characteristics of **HSRP** (Hot Standby Router Protocol), **VRRP** (Virtual Router Redundancy Protocol), and **GLBP** (Gateway Load Balancing Protocol).
- Compare HSRP, VRRP, and GLBP (roles, multicast addresses, virtual MAC addresses).
- Explore basic **HSRP configuration** (group, virtual IP, priority, preemption).
- Prepare for CCNA exam questions on FHRP purpose and protocol differences.

**Analogy**: FHRPs are like having two receptionists at a company’s front desk. If the main receptionist (active router) is unavailable, the backup (standby router) takes over, ensuring visitors (packets) are always directed correctly using a shared phone number (virtual IP) and ID (virtual MAC).

**Lifelong Retention Tip**: Visualize a network with two routers (R1, R2) sharing a virtual IP (172.16.0.254) for PCs. Draw R1 as Active, R2 as Standby, with arrows showing Hello messages (multicast). Memorize: “FHRP = redundant gateway, HSRP/VRRP single active, GLBP load balances.” Create flashcards for multicast IPs (224.0.0.2, 224.0.0.18, 224.0.0.102) and virtual MACs (e.g., HSRP v1: 0000.0c07.acXX). Practice reciting 3 times daily for a week.

## Purpose of FHRPs

### Overview
- **Definition**: FHRPs protect the default gateway by allowing multiple routers to share a virtual IP (VIP) and virtual MAC address, ensuring hosts can reach external networks even if one router fails.
- **Problem Addressed**:
  - Without FHRP: PCs use a single router’s IP (e.g., R1: 172.16.0.254) as the default gateway. If R1 fails, PCs can’t send traffic outside the subnet (e.g., to 8.8.8.8).
  - With FHRP: PCs use a VIP (e.g., 172.16.0.254) shared by R1 (Active) and R2 (Standby). If R1 fails, R2 takes over, maintaining connectivity.
- **How It Works**:
  - Routers negotiate roles (Active/Standby) via multicast **Hello messages**.
  - PCs send traffic to the VIP, resolved to a virtual MAC via ARP.
  - Active router handles traffic; if it fails, the Standby router becomes Active, sending **gratuitous ARP** to update switch MAC tables.
- **Key Mechanism**:
  - **Virtual IP (VIP)**: Shared IP (e.g., 172.16.0.254) configured as the default gateway on hosts.
  - **Virtual MAC**: Shared MAC address (protocol-specific) used in ARP replies.
  - **Gratuitous ARP**: Broadcast ARP replies sent by the new Active router to update switch MAC tables without changing PC ARP tables.
- **Non-Preemptive by Default**:
  - If R1 (Active) fails and R2 becomes Active, R1 becomes Standby upon recovery unless **preemption** is enabled.
  - Similar to OSPF DR/BDR (non-preemptive by default).

**Example**:
- **Topology**: PCs (172.16.0.1–172.16.0.10/24) on VLAN1, connected to SW1–SW4, with R1 (172.16.0.253) and R2 (172.16.0.252) sharing VIP 172.16.0.254.
- **Scenario**:
  - PC1 sends an ARP request for 172.16.0.254.
  - R1 (Active) replies with virtual MAC (e.g., 0000.0c07.ac01 for HSRP v1, group 1).
  - PC1 sends packets to 8.8.8.8 with source IP 172.16.0.1, destination IP 8.8.8.8, source MAC (PC1), destination MAC (virtual MAC).
  - If R1 fails, R2 sends gratuitous ARP, updating switches to forward virtual MAC traffic to R2.
  - PC1’s ARP table (172.16.0.254 → virtual MAC) remains unchanged.

**Comparison to Other Protocols**:
- **OSPF**: DR/BDR election is non-preemptive, similar to FHRPs. OSPF uses multicast (224.0.0.5/6) for Hellos, like FHRPs (224.0.0.2/102/18).
- **Spanning Tree Protocol (STP)**: Ensures loop-free Layer 2 topology (prevents broadcast storms for ARP requests), complementing FHRP’s Layer 3 redundancy.
- **RIP/EIGRP**: No default gateway redundancy; focus on routing table updates, not gateway failover.

**Analogy**: The VIP is like a shared office number for two receptionists. If the main one (Active) is out, the backup (Standby) answers calls, ensuring visitors (packets) reach their destination.

**Lifelong Retention Tip**: Draw a 2-router network with PCs using VIP 172.16.0.254. Label R1 (Active), R2 (Standby), and show ARP request/reply with virtual MAC. Memorize: “FHRP = VIP + virtual MAC, Active/Standby, gratuitous ARP on failover.” In Packet Tracer, simulate R1 failure and observe traffic redirection. Create flashcard: “FHRP purpose? Redundant gateway.” Practice 3 times.

## FHRP Protocols

### HSRP (Hot Standby Router Protocol)
- **Overview**:
  - Cisco proprietary; runs only on Cisco devices.
  - Elects one **Active** router and one **Standby** router per group.
  - Supports multiple groups for different subnets/VLANs (e.g., R1 Active for VLAN1, R2 Active for VLAN2).
- **Versions**:
  - **Version 1**: Default, supports groups 0–255, no IPv6 support.
  - **Version 2**: Supports groups 0–4095, adds IPv6 support.
- **Multicast Address**:
  - v1: 224.0.0.2
  - v2: 224.0.0.102
- **Virtual MAC Address**:
  - v1: 0000.0c07.acXX (XX = group number in hex, e.g., group 1 = 0000.0c07.ac01).
  - v2: 0000.0c9f.fXXX (XXX = group number in hex, e.g., group 1 = 0000.0c9f.f001).
- **Load Balancing**:
  - Not within a single subnet (all hosts use Active router).
  - Across subnets/VLANs by configuring different Active routers per group (e.g., R1 Active for VLAN1, R2 Active for VLAN2).
- **Example**:
  - VLAN1 (172.16.1.0/24): R1 Active, R2 Standby, VIP 172.16.1.252, group 1.
  - VLAN2 (172.16.2.0/24): R2 Active, R1 Standby, VIP 172.16.2.252, group 2.

### VRRP (Virtual Router Redundancy Protocol)
- **Overview**:
  - Open standard; runs on any device supporting VRRP (e.g., Cisco, Juniper).
  - Elects one **Master** router and one or more **Backup** routers per group.
  - Nearly identical to HSRP in functionality.
- **Multicast Address**: 224.0.0.18
- **Virtual MAC Address**: 0000.5e00.01XX (XX = group number in hex, e.g., group 10 = 0000.5e00.010a, group 200 = 0000.5e00.01c8).
- **Load Balancing**: Same as HSRP (across subnets/VLANs, not within a subnet).
- **Example**:
  - VLAN1: R1 Master, R2 Backup, VIP 172.16.1.252, group 10 (MAC: 0000.5e00.010a).

### GLBP (Gateway Load Balancing Protocol)
- **Overview**:
  - Cisco proprietary.
  - Elects one **Active Virtual Gateway (AVG)** and up to four **Active Virtual Forwarders (AVFs)** per group.
  - Supports **load balancing within a single subnet** (e.g., PC1 uses R1, PC2 uses R2 as default gateway).
- **Multicast Address**: 224.0.0.102 (same as HSRP v2).
- **Virtual MAC Address**: 0007.b400.XXYY (XX = group number, YY = AVF number, e.g., group 1, AVF 1 = 0007.b400.0101).
- **Load Balancing**:
  - AVG assigns virtual MACs to AVFs, distributing traffic among hosts in the same subnet.
  - Example: PC1 uses R1’s virtual MAC, PC2 uses R2’s virtual MAC in 172.16.0.0/24.
- **Example**:
  - VLAN1 (172.16.0.0/24): R1 as AVG, R1/R2 as AVFs, VIP 172.16.0.254, group 1 (MACs: 0007.b400.0101 for R1, 0007.b400.0102 for R2).

**Comparison Table**:
| **Protocol** | **Proprietary?** | **Roles** | **Multicast Address** | **Virtual MAC Address** | **Load Balancing** |
|--------------|------------------|-----------|-----------------------|-------------------------|--------------------|
| HSRP v1      | Cisco            | Active, Standby | 224.0.0.2           | 0000.0c07.acXX         | Across subnets     |
| HSRP v2      | Cisco            | Active, Standby | 224.0.0.102         | 0000.0c9f.fXXX         | Across subnets     |
| VRRP         | Open standard    | Master, Backup  | 224.0.0.18          | 0000.5e00.01XX         | Across subnets     |
| GLBP         | Cisco            | AVG, AVF        | 224.0.0.102         | 0007.b400.XXYY         | Within subnet      |

**Comparison to Other Protocols**:
- **OSPF**: DR/BDR use multicast (224.0.0.5/6) and non-preemptive roles, similar to FHRP Active/Standby. OSPF focuses on routing, not gateway redundancy.
- **STP**: Prevents Layer 2 loops (e.g., ARP broadcast storms), complementing FHRP’s Layer 3 redundancy.
- **RIP/EIGRP**: No gateway redundancy; focus on routing updates.

**Analogy**: HSRP/VRRP are like two receptionists sharing one phone number (VIP), with one answering (Active/Master) and the other waiting (Standby/Backup). GLBP is like multiple receptionists each handling different calls (AVFs) within the same office (subnet).

**Lifelong Retention Tip**: Memorize the comparison table using flashcards: “HSRP v1 MAC? 0000.0c07.acXX. GLBP load balancing? Within subnet.” Draw a network with R1/R2 using HSRP (Active/Standby), VRRP (Master/Backup), and GLBP (AVG/AVF). In Packet Tracer, configure HSRP and observe MAC tables (`show mac address-table`). Practice 3 times.

## Basic HSRP Configuration

### Overview
- **Purpose**: Configure HSRP on R1 and R2 to provide a redundant default gateway (VIP 172.16.0.254) for the 172.16.0.0/24 subnet (VLAN1).
- **Key Commands**:
  - `standby version 2`: Sets HSRP version 2 (groups 0–4095, IPv6 support).
  - `standby <group> ip <VIP>`: Sets virtual IP (e.g., 172.16.0.254).
  - `standby <group> priority <value>`: Sets priority (0–255, default 100; higher = Active).
  - `standby <group> preempt`: Enables preemption (router takes Active role if priority/IP higher).
- **Notes**:
  - Configured on the interface connected to the subnet (e.g., G0/0).
  - Group number must match between routers; HSRP v1/v2 must match (not compatible).
  - Priority determines Active router; if tied, highest interface IP wins (e.g., R1: 172.16.0.253 > R2: 172.16.0.252).

**Configuration on R1**:
```bash
conf t
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 172.16.0.254
 standby 1 priority 200
 standby 1 preempt
exit
```
- **Explanation**:
  - `standby version 2`: Uses HSRP v2 (group range 0–4095).
  - `standby 1 ip 172.16.0.254`: Sets VIP for group 1.
  - `standby 1 priority 200`: Sets high priority to make R1 Active (default 100).
  - `standby 1 preempt`: Allows R1 to reclaim Active role if it recovers.

**Configuration on R2**:
```bash
conf t
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 172.16.0.254
 standby 1 priority 100
 standby 1 preempt
exit
```
- **Explanation**:
  - `standby version 2`: Matches R1’s version.
  - `standby 1 ip 172.16.0.254`: Same VIP and group as R1.
  - `standby 1 priority 100`: Default priority (lower than R1’s 200, so R2 is Standby).
  - `standby 1 preempt`: Optional (not needed for Standby, but configured for consistency).

**Verification**:
```bash
show standby
```
- **R1 Output**:
  - Group 1, HSRP v2, state: Active.
  - VIP: 172.16.0.254, virtual MAC: 0000.0c9f.f001 (v2, group 1).
  - Priority: 200, preemption enabled.
  - Active: local (R1), Standby: 172.16.0.252 (R2).
  - Timers: Default (Hello 3s, Dead 10s).
- **R2 Output**:
  - Group 1, state: Standby.
  - VIP: 172.16.0.254, virtual MAC: 0000.0c9f.f001.
  - Priority: 100, preemption enabled.
  - Active: 172.16.0.253 (R1), Standby: local (R2).

**Scenario**:
- R1 (Active, 172.16.0.253, priority 200) handles traffic for VIP 172.16.0.254.
- If R1 fails, R2 (Standby, 172.16.0.252, priority 100) becomes Active, sends gratuitous ARP (MAC: 0000.0c9f.f001), updating switch MAC tables.
- If R1 recovers, preemption enables it to reclaim Active role (higher priority).

**Comparison to Other Protocols**:
- **OSPF**: Configured on interfaces (`ip ospf 1 area 0`) like HSRP (`standby 1 ip`). OSPF uses priority for DR/BDR, similar to HSRP’s Active/Standby election.
- **STP**: No interface configuration; focuses on Layer 2 redundancy, not gateway.
- **RIP/EIGRP**: No gateway redundancy configuration; focus on routing.

**Analogy**: Configuring HSRP is like assigning two receptionists the same phone number (VIP). The senior one (higher priority) answers calls (Active), but the junior (Standby) takes over if needed, with a rule (preemption) to let the senior reclaim the role.

**Lifelong Retention Tip**: In Packet Tracer, configure HSRP on R1/R2 (VIP 172.16.0.254, group 1, v2), set R1’s priority to 200, enable preemption, and verify with `show standby`. Shut down R1’s G0/0, observe R2 becoming Active, and check switch MAC tables (`show mac address-table`). Memorize: “HSRP: standby ip, priority, preempt.” Create flashcard: “HSRP v2 MAC? 0000.0c9f.fXXX.” Practice 3 times.

## Quiz Answers and Explanations

1. **HSRP Version 1 Virtual MAC Address**:
   - **Answer**: D, 0000.0c07.acab
   - **Explanation**: HSRP v1 MAC format is 0000.0c07.acXX (XX = group number in hex). AB (hex) = 171 (decimal), so group 171. Options A/C are HSRP v2, B is invalid.
   - **Tip**: Memorize HSRP v1 MAC: 0000.0c07.acXX.

2. **VRRP Virtual MAC Address**:
   - **Answer**: A, 0000.5e00.010a
   - **Explanation**: VRRP MAC format is 0000.5e00.01XX (XX = group number). 0a (hex) = 10 (decimal), so group 10. B is GLBP, C is HSRP v2, D is invalid.
   - **Tip**: Memorize VRRP MAC: 0000.5e00.01XX.

3. **Valid VRRP Router Roles** (Select Two):
   - **Answers**: B (Backup), D (Master)
   - **Explanation**: VRRP uses Master and Backup roles. A/E are HSRP (Active/Standby), C/F are GLBP (AVG/AVF).
   - **Tip**: Memorize VRRP roles: Master = Active, Backup = Standby.

4. **Messages Sent by HSRP Standby Becoming Active**:
   - **Answer**: B, Gratuitous ARP
   - **Explanation**: When Standby becomes Active, it sends gratuitous ARP (broadcast ARP replies) to update switch MAC tables. A (Hello messages) is ongoing, D (ARP reply) is less specific, C (ARP request) is incorrect.
   - **Tip**: Memorize: “Failover = gratuitous ARP to update switches.”

5. **Accurate Description of HSRP**:
   - **Answer**: C, Provides a redundant default gateway address for hosts in a subnet
   - **Explanation**: HSRP’s primary role is gateway redundancy (VIP). A is incorrect (all hosts use same VIP), B is GLBP’s feature, D is incorrect (routers share one VIP/MAC).
   - **Tip**: Memorize: “HSRP = redundant gateway, not load balancing.”

**Bonus Question (Boson ExSim)**: Not detailed, but likely tests FHRP roles, multicast addresses, or MAC formats.

**Analogy**: The quiz is like a receptionist test: know the shared phone number (VIP/MAC), who answers (Active/Master/AVG), and how backups take over (gratuitous ARP). Practice makes you the network’s top receptionist.

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “HSRP v1 MAC? 0000.0c07.acXX”). In Packet Tracer, configure HSRP, check virtual MAC in ARP tables (`show arp` on PC), and simulate failover. Repeat quiz questions 3 times.

## Key Takeaways

1. **FHRP Purpose**:
   - Provides redundant default gateway using a virtual IP (VIP) and virtual MAC.
   - Active/Standby (HSRP/VRRP) or AVG/AVF (GLBP) roles ensure failover.
   - Gratuitous ARP updates switch MAC tables on failover.
   - Non-preemptive by default; preemption configurable.
2. **Protocol Characteristics**:
   - **HSRP**: Cisco proprietary, Active/Standby, v1 (224.0.0.2, 0000.0c07.acXX), v2 (224.0.0.102, 0000.0c9f.fXXX).
   - **VRRP**: Open standard, Master/Backup, 224.0.0.18, 0000.5e00.01XX.
   - **GLBP**: Cisco proprietary, AVG/AVF, 224.0.0.102, 0007.b400.XXYY, load balances within subnet.
3. **HSRP Configuration**:
   - Commands: `standby version 2`, `standby <group> ip <VIP>`, `standby <group> priority <value>`, `standby <group> preempt`.
   - Priority (default 100) or highest IP determines Active router.
   - Verify: `show standby` (group, state, VIP, MAC, priority).
4. **Verification**:
   - `show standby`: Shows Active/Standby roles, VIP, virtual MAC, priority, preemption.
   - `show arp` (on PC): Confirms VIP maps to virtual MAC.
   - `show mac address-table` (on switch): Verifies virtual MAC port after failover.
5. **CCNA Focus**:
   - Understand FHRP purpose (redundant gateway).
   - Memorize protocol roles, multicast IPs, and virtual MAC formats.
   - Recognize HSRP/VRRP load balance across subnets, GLBP within subnet.

## Study Tips for Lifelong Retention

1. **Mnemonics for Protocols**:
   - HSRP: “Hot Standby, Cisco only, Active/Standby, 224.0.0.2/102.”
   - VRRP: “Virtual Router, open standard, Master/Backup, 224.0.0.18.”
   - GLBP: “Gateway Load Balancing, Cisco, AVG/AVF, within subnet.”
2. **Visualize the Topology**:
   - Draw R1/R2 with VIP 172.16.0.254, PCs using it as default gateway. Label HSRP (Active/Standby), VRRP (Master/Backup), GLBP (AVG/AVF). Show ARP request/reply and gratuitous ARP on failover. Pin to study space.
3. **Memorize Key Values**:
   - Multicast: HSRP v1 (224.0.0.2), v2/GLBP (224.0.0.102), VRRP (224.0.0.18).
   - MACs: HSRP v1 (0000.0c07.acXX), v2 (0000.0c9f.fXXX), VRRP (0000.5e00.01XX), GLBP (0007.b400.XXYY).
   - Flashcards: “HSRP v2 multicast? 224.0.0.102. GLBP MAC? 0007.b400.XXYY.”
4. **Practice in Packet Tracer**:
   - Configure HSRP on R1/R2 (VIP 172.16.0.254, group 1, v2, priority 200 on R1, preempt). Verify with `show standby`, `show arp` (PC), `show mac address-table` (switch).
   - Shut down R1’s interface, observe R2 becoming Active, and check MAC table updates. Repeat 3–5 times.
5. **Compare Protocols**:
   - Create a table: HSRP (Cisco, Active/Standby), VRRP (open, Master/Backup), GLBP (Cisco, AVG/AVF, load balance). Quiz yourself: “GLBP unique feature? Load balance within subnet.”
6. **Troubleshooting Practice**:
   - Simulate issues (e.g., mismatched HSRP versions, incorrect VIP) and fix using `show standby`. Create checklist: “No failover? Check version, group, VIP, priority.”
7. **Teach Someone**:
   - Explain FHRP purpose, HSRP/VRRP/GLBP differences, and HSRP configuration to a peer or record yourself. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki to review multicast IPs, MAC formats, and roles after 1 day, 1 week, 1 month.

These notes provide a solid foundation for understanding FHRPs for CCNA exam topic 3.5. Practice the configurations, use the analogies, and apply the retention tips to master FHRPs for life!