# First Hop Redundancy Protocol (FHRP) Lab Notes for CCNA Beginners

## Lab Overview

This lab demonstrates the configuration and verification of HSRP (Hot Standby Router Protocol) version 2 on two routers (R1 and R2) to provide a redundant default gateway for the 10.0.1.0/24 subnet (VLAN 1). It includes testing connectivity, troubleshooting a version mismatch, and verifying failover and preemption, reinforcing the concepts from the FHRP lecture for CCNA exam topic 3.5. The lab uses a simple topology with PCs, switches, and routers, focusing on practical hands-on skills to understand HSRP’s role in providing default gateway redundancy.

**Lab Objectives**:
- **Step 1**: Verify connectivity from PC1 and PC2 to an external server (8.8.8.8) and check their default gateway.
- **Step 2**: Configure HSRP version 2 on R1 and R2, making R1 the Active router (higher priority) and enabling preemption.
- **Step 3**: Update PC1 and PC2 to use the HSRP virtual IP (VIP) as their default gateway and verify connectivity.
- **Step 4**: Simulate R1 failure, verify R2 takes over as Active, and test connectivity.
- **Step 5**: Restore R1, verify preemption (R1 reclaims Active role), and confirm connectivity.
- **Note**: The lab uses Packet Tracer, with R3’s loopback (8.8.8.8) simulating an external server. No VLAN configuration is needed (all devices in VLAN 1).

**Network Topology**:
- **Devices**:
  - **PC1**: 10.0.1.1/24, default gateway initially 10.0.1.253 (R1), later 10.0.1.254 (VIP).
  - **PC2**: 10.0.1.2/24, default gateway initially 10.0.1.253 (R1), later 10.0.1.254 (VIP).
  - **R1**: G0/0 (10.0.1.253/24), connected to SW1.
  - **R2**: G0/0 (10.0.1.252/24), connected to SW1.
  - **R3**: Loopback 0 (8.8.8.8/32, simulating external server).
  - **SW1–SW4**: Layer 2 switches, all in VLAN 1 (no VLAN configuration needed).
- **Key Notes**:
  - Subnet: 10.0.1.0/24 (VLAN 1).
  - HSRP VIP: 10.0.1.254, group 1, version 2.
  - R1: Priority 200 (Active), preemption enabled.
  - R2: Priority 50 (Standby), preemption enabled (optional).
  - Misconfiguration: Initial HSRP version mismatch (R1: v2, R2: v1).

**Analogy**: HSRP is like two receptionists sharing a company phone number (VIP 10.0.1.254). R1 (senior, Active) answers calls (packets), but if R1 is unavailable, R2 (junior, Standby) takes over. Preemption ensures R1 reclaims the role when back. The lab is like setting up this phone system, testing it, and ensuring failover works.

**Lifelong Retention Tip**: Draw the topology with PC1/PC2 (10.0.1.1–2), R1 (10.0.1.253, Active), R2 (10.0.1.252, Standby), and VIP 10.0.1.254. Label HSRP group 1, v2, and show ARP request/reply with virtual MAC (0000.0c9f.f001). Chant: “HSRP v2, VIP, priority, preempt, failover, check ARP.” In Packet Tracer, replicate the lab, simulate R1 failure, and verify with `show standby`. Create flashcards for commands (`standby 1 ip 10.0.1.254`, `show standby`) and HSRP details (v2 MAC: 0000.0c9f.fXXX). Practice 3 times.

## Lab Steps and Configurations

### Step 1: Verify Connectivity and Default Gateway
**Objective**: Confirm PC1 and PC2 can ping 8.8.8.8 and check their default gateway (initially R1’s G0/0: 10.0.1.253).

**On PC1**:
```bash
ping 8.8.8.8
ipconfig
```
- **Output**:
  - `ping 8.8.8.8`: Successful (replies from R3’s loopback).
  - `ipconfig`: Default gateway 10.0.1.253 (R1’s G0/0).
- **Explanation**: PC1 sends packets to 8.8.8.8 via R1 (10.0.1.253), which routes to R3’s loopback.

**On PC2**:
```bash
ping 8.8.8.8
ipconfig
tracert 8.8.8.8
```
- **Output**:
  - `ping 8.8.8.8`: Successful.
  - `ipconfig`: Default gateway 10.0.1.253 (R1).
  - `tracert 8.8.8.8`: First hop 10.0.1.253 (R1), then 8.8.8.8 (R3).
- **Explanation**:
  - `tracert` (traceroute on PCs) shows the path: PC2 → R1 (10.0.1.253) → R3 (8.8.8.8).
  - Note: Cisco IOS uses `traceroute`, PCs use `tracert`.

**Comparison to FHRP Lecture**:
- **Lecture**: Introduced FHRP purpose (redundant gateway) and ARP process (PCs resolve VIP to virtual MAC).
- **Lab**: Verifies initial setup (no HSRP yet, PCs use R1’s physical IP), setting the stage for HSRP configuration.
- **Similarity**: Both emphasize the default gateway’s role in routing traffic to external networks (e.g., 8.8.8.8).

**Comparison to Other Protocols**:
- **OSPF**: Uses `traceroute` to verify routes (e.g., OSPF Part 3 lab pings 8.8.8.8 via default route).
- **STP**: Ensures no broadcast storms for ARP requests in this lab’s switched network.
- **RIP/EIGRP**: No gateway redundancy; focus on routing table updates, not gateway verification.

**Analogy**: Checking the default gateway is like confirming the company’s main receptionist (R1, 10.0.1.253) is answering calls (packets) from PCs to the external server (8.8.8.8).

**Lifelong Retention Tip**: In Packet Tracer, ping 8.8.8.8 from PC1/PC2, run `ipconfig` to confirm gateway (10.0.1.253), and `tracert 8.8.8.8` to see R1 as the first hop. Memorize: “PC gateway = router IP before HSRP.” Create flashcard: “PC traceroute command? tracert.” Practice 3 times.

### Step 2: Configure HSRP Version 2 on R1 and R2
**Objective**: Configure HSRP v2 on R1 and R2 (G0/0, 10.0.1.0/24), with R1 as Active (priority 200), R2 as Standby (priority 50), and preemption enabled. Troubleshoot a version mismatch.

**Configuration on R1**:
```bash
enable
conf t
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 10.0.1.254
 standby 1 priority 200
 standby 1 preempt
exit
end
write
```
- **Explanation**:
  - `standby version 2`: Sets HSRP version 2 (group range 0–4095, virtual MAC 0000.0c9f.fXXX).
  - `standby 1 ip 10.0.1.254`: Sets VIP 10.0.1.254 for group 1 (matches VLAN 1, not mandatory).
  - `standby 1 priority 200`: Sets high priority (default 100) to make R1 Active.
  - `standby 1 preempt`: Allows R1 to reclaim Active role if it recovers.
  - `end` and `write`: Saves configuration to prevent loss on restart.

**Initial Configuration on R2 (Version Mismatch)**:
```bash
enable
conf t
interface GigabitEthernet0/0
 standby 1 priority 50
 standby 1 ip 10.0.1.254
exit
```
- **Issue**:
  - R2 uses HSRP v1 (default, not specified), while R1 uses v2.
  - After ~30 seconds (Packet Tracer fast-forward), R2 logs “duplicate address” errors, as both R1 and R2 claim Active status (no negotiation due to version mismatch).
  - **Cause**: HSRP v1 (224.0.0.2, MAC 0000.0c07.acXX) and v2 (224.0.0.102, MAC 0000.0c9f.fXXX) are incompatible.

**Fix on R2**:
```bash
conf t
interface GigabitEthernet0/0
 standby version 2
 standby 1 preempt
exit
end
write
```
- **Explanation**:
  - `standby version 2`: Matches R1’s HSRP v2.
  - `standby 1 preempt`: Optional (not needed for Standby, added for consistency).
  - `standby 1 priority 50`: Already set (lower than R1’s 200, ensures Standby).
  - `standby 1 ip 10.0.1.254`: Already set (same VIP as R1).
  - After ~30 seconds, R2 becomes Standby, and errors stop.

**Verification on R2**:
```bash
show standby
```
- **Output**:
  - Group 1, HSRP v2, state: Standby.
  - VIP: 10.0.1.254, virtual MAC: 0000.0c9f.f001 (v2, group 1).
  - Priority: 50, preemption enabled.
  - Active: 10.0.1.253 (R1), Standby: local (R2).
  - Timers: Default (Hello 3s, Dead 10s).

**Comparison to FHRP Lecture**:
- **Lecture**: Covered HSRP configuration (`standby version`, `standby ip`, `priority`, `preempt`) and version mismatch issues (v1 vs. v2).
- **Lab**: Applies these commands, demonstrates version mismatch troubleshooting, and verifies with `show standby`.
- **Similarity**: Both emphasize Active/Standby roles, priority, preemption, and virtual MAC (0000.0c9f.f001 for v2, group 1).

**Comparison to Other Protocols**:
- **OSPF**: Similar interface configuration (`ip ospf 1 area 0`) and troubleshooting (e.g., mismatched timers/areas in OSPF Part 3 lab).
- **STP**: Complements HSRP by preventing ARP broadcast storms in the switched network.
- **RIP/EIGRP**: No gateway redundancy; focus on routing, unlike HSRP’s gateway focus.

**Analogy**: Configuring HSRP is like assigning two receptionists the same phone number (VIP 10.0.1.254). R1 (senior, priority 200) answers, R2 (junior, priority 50) waits. A version mismatch is like using different phone systems (v1 vs. v2), causing both to answer chaotically until aligned (v2).

**Lifelong Retention Tip**: In Packet Tracer, configure HSRP v2 on R1/R2 (VIP 10.0.1.254, group 1, R1 priority 200, preempt), then set R2 to v1 to simulate mismatch (check `show standby` for errors). Fix with `standby version 2`. Memorize: “HSRP v2: priority high = Active, preempt for recovery.” Create flashcard: “HSRP v2 MAC? 0000.0c9f.fXXX.” Practice 3 times.

### Step 3: Update PC1 and PC2 Default Gateway
**Objective**: Change PC1 and PC2’s default gateway to the HSRP VIP (10.0.1.254) and verify connectivity.

**On PC1**:
- **GUI**: Config tab → Default Gateway: 10.0.1.254
- **CLI Equivalent**:
  ```bash
  ip 10.0.1.1 255.255.255.0 10.0.1.254
  ```

**On PC2**:
- **GUI**: Config tab → Default Gateway: 10.0.1.254
- **CLI Equivalent**:
  ```bash
  ip 10.0.1.2 255.255.255.0 10.0.1.254
  ```

**Verification on PC2**:
```bash
ping 8.8.8.8
arp -a
tracert 8.8.8.8
```
- **Output**:
  - `ping 8.8.8.8`: Successful (via R1, Active router).
  - `arp -a`: Shows 10.0.1.254 → 0000.0c9f.f001 (HSRP v2, group 1 virtual MAC).
  - `tracert 8.8.8.8`: First hop 10.0.1.253 (R1’s physical IP), then 8.8.8.8 (R3).
- **Explanation**:
  - PCs now use VIP 10.0.1.254, resolved to virtual MAC 0000.0c9f.f001 via ARP.
  - Traceroute shows R1’s physical IP (10.0.1.253), not VIP, as routers reply with interface IP.

**Comparison to FHRP Lecture**:
- **Lecture**: Explained PCs using VIP and resolving it to virtual MAC via ARP.
- **Lab**: Configures VIP (10.0.1.254) on PCs and verifies ARP table (virtual MAC).
- **Similarity**: Both highlight ARP process (VIP → virtual MAC) and traceroute behavior (physical IP).

**Comparison to Other Protocols**:
- **OSPF**: No PC configuration; OSPF Part 3 lab focused on router routes (e.g., 8.8.8.8 via default route).
- **STP**: Ensures ARP requests/replies (e.g., for 10.0.1.254) don’t cause loops.
- **RIP/EIGRP**: No PC gateway changes; focus on router protocols.

**Analogy**: Updating the default gateway is like telling employees (PCs) to call the shared phone number (VIP 10.0.1.254) instead of R1’s direct line (10.0.1.253). The ARP table confirms they’re dialing the shared ID (virtual MAC).

**Lifelong Retention Tip**: In Packet Tracer, set PC1/PC2’s gateway to 10.0.1.254, verify with `arp -a` (shows 0000.0c9f.f001), and `tracert 8.8.8.8` (shows R1’s 10.0.1.253). Memorize: “VIP in ARP, physical IP in traceroute.” Create flashcard: “HSRP PC command? arp -a.” Practice 3 times.

### Step 4: Simulate R1 Failure and Verify Failover
**Objective**: Turn off R1, verify R2 becomes Active, and confirm PC1 connectivity.

**On R1**:
- **GUI**: Physical tab → Flip power switch to OFF (simulates failure).
- **CLI Equivalent**:
  ```bash
  enable
  conf t
  interface GigabitEthernet0/0
   shutdown
  ```

**Verification on PC1**:
```bash
ping 8.8.8.8
tracert 8.8.8.8
```
- **Output**:
  - `ping 8.8.8.8`: Successful (via R2, now Active).
  - `tracert 8.8.8.8`: First hop 10.0.1.252 (R2’s physical IP), then 8.8.8.8.
- **Explanation**:
  - R1’s failure triggers R2 to become Active (~30 seconds after missing Hellos).
  - R2 sends gratuitous ARP (MAC 0000.0c9f.f001), updating switch MAC tables to forward VIP traffic to R2.
  - PC1’s ARP table (10.0.1.254 → 0000.0c9f.f001) remains unchanged, ensuring seamless failover.

**Verification on R2** (optional):
```bash
show standby
```
- **Output**: State: Active, Active router: local (10.0.1.252), Standby: none.

**Comparison to FHRP Lecture**:
- **Lecture**: Described failover with gratuitous ARP to update switch MAC tables.
- **Lab**: Simulates R1 failure, verifies R2’s Active role, and checks connectivity.
- **Similarity**: Both emphasize seamless failover using virtual MAC and gratuitous ARP.

**Comparison to Other Protocols**:
- **OSPF**: DR/BDR failover (non-preemptive, like HSRP default) vs. HSRP’s preemptive failover.
- **STP**: Ensures switches update MAC tables correctly during HSRP failover.
- **RIP/EIGRP**: No failover mechanism for gateways.

**Analogy**: R1’s failure is like the senior receptionist leaving; R2 (junior) takes over, broadcasting the shared ID (gratuitous ARP) to redirect calls (packets) without employees (PCs) changing their contact list (ARP table).

**Lifelong Retention Tip**: In Packet Tracer, shut down R1’s G0/0, wait 30 seconds, verify R2’s Active status (`show standby`), and ping 8.8.8.8 from PC1 (`tracert` shows 10.0.1.252). Check switch MAC table (`show mac address-table`). Memorize: “Failover = R2 Active, gratuitous ARP.” Create flashcard: “Failover trigger? Missing Hellos.” Practice 3 times.

### Step 5: Restore R1 and Verify Preemption
**Objective**: Turn R1 back on, verify it reclaims Active role (due to preemption), and confirm connectivity.

**On R1**:
- **GUI**: Physical tab → Flip power switch to ON.
- **CLI Equivalent**:
  ```bash
  enable
  conf t
  interface GigabitEthernet0/0
   no shutdown
  ```

**Verification on PC1**:
```bash
ping 8.8.8.8
tracert 8.8.8.8
```
- **Output**:
  - `ping 8.8.8.8`: Successful (via R1, now Active).
  - `tracert 8.8.8.8`: First hop 10.0.1.253 (R1’s physical IP), then 8.8.8.8.
- **Explanation**:
  - R1 comes online, detects higher priority (200 vs. R2’s 50), and preempts R2 to become Active (~30 seconds).
  - R1 sends gratuitous ARP (MAC 0000.0c9f.f001), updating switch MAC tables.
  - PC1’s ARP table remains unchanged (10.0.1.254 → 0000.0c9f.f001).

**Verification on R1** (optional):
```bash
show standby
```
- **Output**: State: Active, Active router: local (10.0.1.253), Standby: 10.0.1.252 (R2).

**Comparison to FHRP Lecture**:
- **Lecture**: Explained preemption (Active router reclaims role if priority higher).
- **Lab**: Demonstrates R1 preempting R2 due to priority 200 and `standby 1 preempt`.
- **Similarity**: Both highlight preemption’s role in restoring the preferred Active router.

**Comparison to Other Protocols**:
- **OSPF**: DR/BDR non-preemptive by default, unlike HSRP with `preempt`.
- **STP**: No preemption; root bridge election based on priority/bridge ID.
- **RIP/EIGRP**: No equivalent preemption mechanism.

**Analogy**: R1’s return is like the senior receptionist coming back and reclaiming the phone (VIP) because of seniority (higher priority) and a rule (preemption), ensuring calls (packets) go to her again.

**Lifelong Retention Tip**: In Packet Tracer, re-enable R1’s G0/0, wait 30 seconds, verify R1’s Active status (`show standby`), and ping 8.8.8.8 from PC1 (`tracert` shows 10.0.1.253). Memorize: “Preemption = higher priority wins.” Create flashcard: “Preemption command? standby 1 preempt.” Practice 3 times.

## Key Takeaways

1. **HSRP Configuration**:
   - Commands: `standby version 2`, `standby <group> ip <VIP>`, `standby <group> priority <value>`, `standby <group> preempt`.
   - R1: Priority 200, preempt, Active; R2: Priority 50, Standby.
   - VIP: 10.0.1.254, group 1, virtual MAC: 0000.0c9f.f001 (HSRP v2).
2. **Troubleshooting**:
   - **Version Mismatch**: R1 (v2) vs. R2 (v1) causes both to claim Active, duplicate IP errors.
     - Fix: Set R2 to `standby version 2`.
     - Verify: `show standby` (R2: Standby, Active: 10.0.1.253).
3. **Verification Commands**:
   - **PC**: `ipconfig` (gateway), `arp -a` (VIP → virtual MAC), `tracert 8.8.8.8` (physical IP).
   - **Router**: `show standby` (group, state, VIP, MAC, priority, Active/Standby).
   - **Switch**: `show mac address-table` (virtual MAC port after failover).
4. **Failover and Preemption**:
   - R1 failure → R2 Active, sends gratuitous ARP to update switch MAC tables.
   - R1 recovery → Preempts R2 (priority 200 > 50), sends gratuitous ARP.
   - PC ARP table unchanged (10.0.1.254 → 0000.0c9f.f001).
5. **Key Concepts**:
   - HSRP v2: Multicast 224.0.0.102, MAC 0000.0c9f.fXXX, group 0–4095.
   - Priority (default 100) or highest IP (10.0.1.253 > 10.0.1.252) determines Active.
   - Traceroute shows physical IP (10.0.1.253/252), not VIP (10.0.1.254).
   - Save config (`write`) to persist after restart.

## Common Issues and Troubleshooting

- **No Failover (R2 Doesn’t Become Active)**:
  - **Issue**: HSRP version mismatch (R1: v2, R2: v1).
  - **Fix**: `standby version 2` on R2.
  - **Verify**: `show standby` (R2: Standby, Active: 10.0.1.253).
- **Duplicate IP Errors**:
  - **Issue**: Both routers Active due to version mismatch or mismatched group/VIP.
  - **Fix**: Ensure same version, group, and VIP (`standby 1 ip 10.0.1.254`).
  - **Verify**: `show standby`, check console logs.
- **R1 Doesn’t Preempt**:
  - **Issue**: Missing `standby 1 preempt` or lower priority on R1.
  - **Fix**: `standby 1 preempt`, ensure priority > R2 (e.g., 200).
  - **Verify**: `show standby` after R1 restart (R1: Active).
- **PCs Can’t Ping 8.8.8.8**:
  - **Issue**: Incorrect VIP or gateway misconfigured on PCs.
  - **Fix**: Set PC gateway to 10.0.1.254 (`ip 10.0.1.x 255.255.255.0 10.0.1.254`).
  - **Verify**: `ipconfig`, `arp -a`, `ping 8.8.8.8`.

**Analogy**: Troubleshooting HSRP is like fixing a phone system where two receptionists (R1, R2) mistakenly use different networks (v1 vs. v2). Align them (v2), ensure the senior (R1, priority 200) answers, and confirm employees (PCs) dial the right number (VIP).

**Lifelong Retention Tip**: In Packet Tracer, simulate issues (e.g., set R2 to v1, omit `preempt` on R1), troubleshoot using `show standby`, and verify failover/preemption. Create a checklist: “Failover? Check version, group, VIP, priority.” Practice 3 times.

## Practice Commands

- **HSRP Configuration (R1)**:
  ```bash
  interface GigabitEthernet0/0
   standby version 2
   standby 1 ip 10.0.1.254
   standby 1 priority 200
   standby 1 preempt
  end
  write
  ```
- **HSRP Configuration (R2)**:
  ```bash
  interface GigabitEthernet0/0
   standby version 2
   standby 1 ip 10.0.1.254
   standby 1 priority 50
   standby 1 preempt
  end
  write
  ```
- **PC Configuration**:
  ```bash
  ip 10.0.1.x 255.255.255.0 10.0.1.254
  ```
- **Verification**:
  - **PC**: `ipconfig`, `arp -a`, `ping 8.8.8.8`, `tracert 8.8.8.8`
  - **Router**: `show standby`
  - **Switch**: `show mac address-table`
- **Troubleshooting**:
  ```bash
  show standby
  interface GigabitEthernet0/0
   standby version 2
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics for Commands**:
   - Chant: “HSRP v2, VIP, priority, preempt, save config, check standby.” Associate with a tune (e.g., “Twinkle Twinkle”) for recall.
2. **Visualize the Topology**:
   - Draw PC1/PC2, R1 (10.0.1.253, Active), R2 (10.0.1.252, Standby), VIP 10.0.1.254, and switches. Show ARP (10.0.1.254 → 0000.0c9f.f001) and traceroute (R1’s IP). Pin to study space.
3. **Memorize Key Values**:
   - HSRP v2: Multicast 224.0.0.102, MAC 0000.0c9f.fXXX, group 1 = f001.
   - Priority: Default 100, R1: 200 (Active), R2: 50 (Standby).
   - Flashcards: “HSRP v2 MAC? 0000.0c9f.fXXX. Preemption command? standby 1 preempt.”
4. **Practice in Packet Tracer**:
   - Configure HSRP v2, simulate R1 failure (`shutdown`), verify R2 Active (`show standby`), and check PC connectivity (`ping`, `tracert`). Restore R1, verify preemption. Repeat 3–5 times.
5. **Compare to Lecture**:
   - Create a table: Lecture (theory, HSRP/VRRP/GLBP, MACs) vs. Lab (HSRP v2 config, failover, preemption). Quiz yourself: “Lab focus? HSRP v2, troubleshoot version mismatch.”
6. **Troubleshooting Practice**:
   - Break HSRP (e.g., set R2 to v1, mismatch VIP), fix using `show standby`. Create checklist: “No Active/Standby? Check version, group, VIP.”
7. **Teach Someone**:
   - Explain HSRP setup, failover, and preemption to a peer or record yourself. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki to review commands, HSRP details, and troubleshooting steps after 1 day, 1 week, 1 month.

These notes provide a complete foundation for the FHRP lab, preparing you for CCNA exam topic 3.5 and reinforcing HSRP concepts from the lecture. Practice the configurations, use the analogies, and apply the retention tips to master HSRP for life!