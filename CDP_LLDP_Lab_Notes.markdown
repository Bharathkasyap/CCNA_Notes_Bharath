# CDP and LLDP Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on using Cisco Discovery Protocol (CDP) to map a network topology and transitioning to Link Layer Discovery Protocol (LLDP) by disabling CDP and enabling LLDP, aligning with CCNA exam topic 2.3 ("Configure and verify Layer 2 discovery protocols CDP and LLDP"). It builds on the CDP/LLDP lecture, Standard ACLs lecture/lab (Day 34), Extended ACLs lecture/lab (Day 35), and IPv6 labs (Parts 1–3). Performed in Packet Tracer, the lab involves a network with three routers (R1, R2, R3), three switches (SW1, SW2, SW3), and three PCs (PC1, PC2, PC3), initially without IP or interface labels. The goal is to label the topology using CDP, disable CDP on PC-connected interfaces, and switch to LLDP to prepare for multi-vendor devices.

**Lab Objectives**:
- **Map Network with CDP**: Use `show cdp neighbors` and other commands to label interfaces and IP addresses.
- **Disable CDP**: Turn off CDP on PC-connected interfaces (`no cdp enable`) and globally (`no cdp run`).
- **Enable LLDP**: Activate LLDP globally (`lldp run`) and on router/switch interfaces (`lldp transmit`, `lldp receive`).
- **Verify Configurations**: Confirm topology with `show cdp neighbors`, `show lldp neighbors`, and check disabled CDP interfaces.
- **Prepare for Exam**: Master topic 2.3, focusing on CDP/LLDP commands, neighbor tables, and multi-vendor considerations.

**Network Topology**:
- **Devices and Connections**:
  - **R1**:
    - G0/0: Connected to R3 (G0/1), `10.0.13.0/30` (R1: `.1`, R3: `.2`).
    - G0/1: Connected to R2 (G0/0), `10.0.12.0/30` (R1: `.1`, R2: `.2`).
    - G0/2: Connected to SW1 (G0/1), `192.168.1.0/24` (R1: `.254`, PC1: `.1`).
  - **R2**:
    - G0/0: Connected to R1 (G0/1), `10.0.12.0/30`.
    - G0/1: Connected to SW2 (G0/2), `192.168.2.0/24` (R2: `.254`, PC2: `.1`).
    - G0/2: Connected to R3 (G0/2), `10.0.23.0/30` (R2: `.1`, R3: `.2`).
  - **R3**:
    - G0/0: Connected to SW3 (G0/1), `192.168.3.0/24` (R3: `.254`, PC3: `.1`).
    - G0/1: Connected to R1 (G0/0), `10.0.13.0/30`.
    - G0/2: Connected to R2 (G0/2), `10.0.23.0/30`.
  - **SW1**: G0/1 to R1 (G0/2), F0/10 to PC1.
  - **SW2**: G0/2 to R2 (G0/1), F0/1 to PC2.
  - **SW3**: G0/1 to R3 (G0/0), F0/24 to PC3.
- **Notes**:
  - PCs connect to switches via FastEthernet; routers/switches use GigabitEthernet.
  - EIGRP pre-configured for connectivity (similar to OSPF in Day 34), allowing pings across networks.
  - Switches are Layer 2 (no IP on interfaces, unlike multilayer switches in CDP/LLDP lecture).
- **Lab Steps**:
  1. **Label Interfaces**: Use `show cdp neighbors` for router/switch connections and `show interfaces status` for PC connections.
  2. **Disable CDP on PC Interfaces**: Use `no cdp enable` on SW1 (F0/10), SW2 (F0/1), SW3 (F0/24).
  3. **Disable CDP Globally**: Use `no cdp run` on all routers and switches.
  4. **Enable LLDP**: Use `lldp run` globally and `lldp transmit`, `lldp receive` on router/switch interfaces.

**Analogy**: CDP is like a Cisco-only neighborhood directory, helping you map houses (devices) and streets (interfaces) by sharing business cards (frames) with details like addresses (IPs). LLDP is a universal directory for mixed neighborhoods (multi-vendor devices). Disabling CDP on PC interfaces is like not sharing your address with non-residents (PCs), and switching to LLDP prepares for new neighbors from other vendors (e.g., Juniper).

**Lifelong Retention Tip**: Visualize R1–SW1 link (G0/2–G0/1) with CDP showing “SW1, switch, G0/1.” In Packet Tracer, run `show cdp neighbors` on R1, label G0/0–R3, disable CDP on SW1’s F0/10, enable LLDP, and verify with `show lldp neighbors`. Create flashcards for commands (`no cdp enable`, `lldp run`), MACs (`0100.0CCC.CCCC` for CDP, `0180.C200.000E` for LLDP), and timers (CDP: 60s/180s, LLDP: 30s/120s). Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Label Interfaces Using CDP
**Objective**: Use CDP and other commands to map interfaces and IP addresses across the network.

**Actions**:
- **PC1**:
  ```plaintext
  PC1> ipconfig
  IP Address......................: 192.168.1.1
  Subnet Mask.....................: 255.255.255.0
  Default Gateway.................: 192.168.1.254
  ```
  - **Labels**: Network `192.168.1.0/24`, PC1: `192.168.1.1`, R1: `192.168.1.254`.
  - **Note**: PCs don’t run CDP; `ipconfig` provides IP and default gateway (router interface).
- **SW1**:
  ```plaintext
  SW1> enable
  SW1# show cdp neighbors
  Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
  R1           Gig 0/1         175       R            2900       Gig 0/2
  SW1# show interfaces status
  Port      Name    Status       Vlan  Duplex  Speed Type
  Fa0/10            connected    1     a-full  a-100 10/100BaseTX
  Gig0/1            connected    1     a-full  a-1000 10/100/1000BaseTX
  ```
  - **Labels**: SW1 G0/1 to R1 G0/2, F0/10 to PC1 (only Fa0/10 connected besides G0/1).
  - **Note**: `show interfaces status` identifies PC connection since PCs don’t support CDP.
- **R1**:
  ```plaintext
  R1# show cdp neighbors
  Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
  SW1          Gig 0/2         165       S            2960       Gig 0/1
  R2           Gig 0/1         170       R            2900       Gig 0/0
  R3           Gig 0/0         160       R            2900       Gig 0/1
  R1# show interfaces g0/0
  Internet address is 10.0.13.1/30
  R1# show interfaces g0/1
  Internet address is 10.0.12.1/30
  R1# show cdp entry R3
  Device ID: R3
  IP address: 10.0.13.2
  ```
  - **Labels**: R1 G0/0 to R3 G0/1 (`10.0.13.0/30`, R1: `.1`, R3: `.2`), R1 G0/1 to R2 G0/0 (`10.0.12.0/30`, R1: `.1`, R2: `.2`), R1 G0/2 to SW1 G0/1.
  - **Note**: `show interfaces` provides local IPs and prefix length; `show cdp entry` confirms neighbor IPs; /30 subnets have only two host addresses (e.g., `.1`, `.2`).
- **PC2**:
  ```plaintext
  PC2> ipconfig
  IP Address......................: 192.168.2.1
  Subnet Mask.....................: 255.255.255.0
  Default Gateway.................: 192.168.2.254
  ```
  - **Labels**: Network `192.168.2.0/24`, PC2: `192.168.2.1`, R2: `192.168.2.254`.
- **SW2**:
  ```plaintext
  SW2# show cdp neighbors
  Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
  R2           Gig 0/2         175       R            2900       Gig 0/1
  SW2# show interfaces status
  Port      Name    Status       Vlan  Duplex  Speed Type
  Fa0/1             connected    1     a-full  a-100 10/100BaseTX
  Gig0/2            connected    1     a-full  a-1000 10/100/1000BaseTX
  ```
  - **Labels**: SW2 G0/2 to R2 G0/1, F0/1 to PC2.
- **R2**:
  ```plaintext
  R2# show cdp neighbors
  Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
  R1           Gig 0/0         165       R            2900       Gig 0/1
  SW2          Gig 0/1         170       S            2960       Gig 0/2
  R3           Gig 0/2         160       R            2900       Gig 0/2
  R2# show interfaces g0/2
  Internet address is 10.0.23.1/30
  ```
  - **Labels**: R2 G0/2 to R3 G0/2 (`10.0.23.0/30`, R2: `.1`, R3: `.2`), R2 G0/1 to SW2 G0/2, R2 G0/0 to R1 G0/1.
- **PC3**:
  ```plaintext
  PC3> ipconfig
  IP Address......................: 192.168.3.1
  Subnet Mask.....................: 255.255.255.0
  Default Gateway.................: 192.168.3.254
  ```
  - **Labels**: Network `192.168.3.0/24`, PC3: `192.168.3.1`, R3: `192.168.3.254`.
- **SW3**:
  ```plaintext
  SW3# show cdp neighbors
  Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
  R3           Gig 0/1         175       R            2900       Gig 0/0
  SW3# show interfaces status
  Port      Name    Status       Vlan  Duplex  Speed Type
  Fa0/24            connected    1     a-full  a-100 10/100BaseTX
  Gig0/1            connected    1     a-full  a-1000 10/100/1000BaseTX
  ```
  - **Labels**: SW3 G0/1 to R3 G0/0, F0/24 to PC3.

**Observations**:
- **CDP Usage**: `show cdp neighbors` maps router/switch connections (e.g., R1 G0/0 to R3 G0/1) by showing Device ID, Local Interface, and Port ID.
- **PC Connections**: PCs don’t support CDP; `show interfaces status` identifies connected FastEthernet ports (e.g., SW1 F0/10 to PC1, only connected port besides G0/1).
- **IP Addressing**:
  - PCs: `ipconfig` provides IP and default gateway (e.g., PC1: `192.168.1.1`, R1: `192.168.1.254`).
  - Routers: `show interfaces` gives local IPs with prefix length (e.g., R1 G0/0: `10.0.13.1/30`); `show cdp entry` confirms neighbor IPs (e.g., R3: `10.0.13.2`); /30 subnets have two hosts (`.1`, `.2`).
- **EIGRP**: Pre-configured for connectivity (similar to OSPF in Day 34), enabling pings across networks (e.g., PC1 to PC2).

**Comparison to Prior Content**:
- **CDP/LLDP Lecture**: Used `show cdp neighbors` for R1–SW1, R1–R2; this lab maps a larger topology (3 routers, 3 switches, 3 PCs) and includes PCs.
- **Standard ACLs (Day 34)**: Used `show ip interface brief` for IPs; here, `show interfaces` includes prefix length (e.g., `/30`), and CDP adds neighbor IPs.
- **Extended ACLs (Day 35)**: Blocked DNS/HTTP; CDP shares IPs (e.g., R3: `10.0.13.2`) for topology mapping, not filtering.
- **IPv6 Parts 1–3**: Used `show ipv6 interface` for addresses (e.g., `2001:db8::/64`); here, CDP shares IPv4 addresses at Layer 2.
- **Spanning Tree**: Used `show spanning-tree` to verify VLANs; here, `show interfaces status` confirms connected ports (VLAN 1).

**Comparison to Other CCNA Topics**:
- **EIGRP**: Pre-configured here (like OSPF in Day 34) for connectivity; CDP/LLDP map topology, not routes.
- **VLANs/VTP**: Switches in VLAN 1 (seen in `show interfaces status`); CDP could share VTP info, but not used here.
- **Ethernet (Day 10)**: CDP uses Ethernet II (ARPA) encapsulation, seen in `show cdp interface`.

**Analogy**: Mapping with CDP is like using a Cisco-only address book to label streets (interfaces) and houses (IPs) in a neighborhood. For PCs, you check mailbox labels (`ipconfig`, `show interfaces status`) since they don’t share cards.

**Lifelong Retention Tip**: In Packet Tracer, run `show cdp neighbors` on R1, label G0/0–R3 (G0/1), `10.0.13.0/30`. Flashcard: “Map topology? `show cdp neighbors` for routers/switches, `show interfaces status` for PCs.” Practice 3 times.

### Step 2: Disable CDP on PC-Connected Interfaces
**Objective**: Disable CDP on switch interfaces connected to PCs (SW1 F0/10, SW2 F0/1, SW3 F0/24) to prevent sharing device info with non-CDP devices.

**Actions**:
- **SW1**:
  ```plaintext
  SW1# configure terminal
  SW1(config)# interface f0/10
  SW1(config-if)# do show cdp interface f0/10
  FastEthernet0/10 is up, line protocol is up
    Sending CDP packets every 60 seconds
    Holdtime is 180 seconds
  SW1(config-if)# no cdp enable
  SW1(config-if)# do show cdp interface f0/10
  [No output, confirming CDP disabled]
  ```
- **SW2**:
  ```plaintext
  SW2(config)# interface f0/1
  SW2(config-if)# no cdp enable
  ```
- **SW3**:
  ```plaintext
  SW3(config)# interface f0/24
  SW3(config-if)# no cdp enable
  ```

**Observations**:
- **Command**: `no cdp enable` disables CDP on specific interfaces (default: enabled with `cdp enable`).
- **Verification**: `show cdp interface f0/10` shows CDP status; no output after `no cdp enable` confirms disablement.
- **Purpose**: PCs don’t support CDP, so disabling it on PC interfaces (F0/10, F0/1, F0/24) prevents unnecessary frame transmission and enhances security.
- **Distinction**: `no cdp run` (global) disables CDP entirely; `no cdp enable` (interface) disables it per interface.

**Comparison to Prior Content**:
- **CDP/LLDP Lecture**: Explained `no cdp enable` for interfaces; here, applied to PC-connected ports to limit CDP scope.
- **Standard ACLs (Day 34)**: Disabled traffic with `deny` statements; here, `no cdp enable` stops CDP frames, a security measure.
- **Extended ACLs (Day 35)**: Blocked specific protocols (e.g., DNS); here, CDP is disabled to prevent info sharing with PCs.
- **IPv6 Parts 1–3**: No equivalent; focused on Layer 3 addressing, not Layer 2 discovery.

**Comparison to Other CCNA Topics**:
- **Security (Later Topic)**: Disabling CDP on PC interfaces aligns with reducing attack surfaces (like disabling CDP globally in secure networks).
- **VLANs**: PC interfaces in VLAN 1; disabling CDP doesn’t affect VLAN operation.
- **EIGRP**: Connectivity unaffected by CDP disablement, as EIGRP uses IP packets.

**Analogy**: Disabling CDP on PC interfaces is like removing non-residents (PCs) from the Cisco neighborhood directory, keeping device info (hostname, platform) private to routers/switches.

**Lifelong Retention Tip**: In Packet Tracer, disable CDP on SW1 F0/10, verify with `show cdp interface f0/10` (no output). Flashcard: “Disable CDP on interface? `no cdp enable`.” Practice 3 times.

### Step 3: Disable CDP Globally
**Objective**: Disable CDP on all devices (R1, R2, R3, SW1, SW2, SW3) using `no cdp run` to stop CDP frame transmission network-wide.

**Actions**:
- **SW3**:
  ```plaintext
  SW3(config)# no cdp run
  ```
- **R3**:
  ```plaintext
  R3> enable
  R3# configure terminal
  R3(config)# no cdp run
  ```
- **SW2**:
  ```plaintext
  SW2(config)# no cdp run
  ```
- **R2**:
  ```plaintext
  R2(config)# no cdp run
  ```
- **SW1**:
  ```plaintext
  SW1(config)# no cdp run
  ```
- **R1**:
  ```plaintext
  R1(config)# no cdp run
  ```

**Observations**:
- **Command**: `no cdp run` disables CDP globally, stopping all CDP frame transmission (default: `cdp run`).
- **Impact**: No CDP neighbors appear in `show cdp neighbors` after disablement; prepares network for LLDP.
- **Security**: Common in secure networks to prevent device info leakage (e.g., hostname, IP).

**Comparison to Prior Content**:
- **CDP/LLDP Lecture**: Noted `no cdp run` disables CDP globally; here, applied to all devices for LLDP transition.
- **Standard ACLs (Day 34)**: `deny` statements block traffic; `no cdp run` blocks discovery frames, both enhance security.
- **Extended ACLs (Day 35)**: Blocked specific services (e.g., HTTP); `no cdp run` stops all CDP info sharing.
- **IPv6 Parts 1–3**: No Layer 2 discovery; focused on IPv6 addressing/routing.

**Comparison to Other CCNA Topics**:
- **Security**: Disabling CDP globally aligns with hardening practices (like disabling unused services).
- **EIGRP**: Connectivity remains (EIGRP uses IP, not CDP).
- **VTP**: CDP could share VTP info; disabling CDP stops this, but VTP unaffected.

**Analogy**: Disabling CDP globally is like shutting down the Cisco neighborhood newsletter, stopping all address sharing to prepare for a universal directory (LLDP).

**Lifelong Retention Tip**: In Packet Tracer, run `no cdp run` on R1, verify `show cdp neighbors` (no output). Flashcard: “Disable CDP globally? `no cdp run`.” Practice 3 times.

### Step 4: Enable LLDP
**Objective**: Enable LLDP globally (`lldp run`) and on router/switch interfaces (`lldp transmit`, `lldp receive`) to replace CDP, supporting multi-vendor devices.

**Actions**:
- **SW3**:
  ```plaintext
  SW3(config)# lldp run
  SW3(config)# interface g0/1
  SW3(config-if)# lldp transmit
  SW3(config-if)# lldp receive
  ```
- **R3**:
  ```plaintext
  R3(config)# lldp run
  R3(config)# interface range g0/0 - 2
  R3(config-if-range)# lldp transmit
  R3(config-if-range)# lldp receive
  ```
- **SW2**:
  ```plaintext
  SW2(config)# lldp run
  SW2(config)# interface g0/2
  SW2(config-if)# lldp transmit
  SW2(config-if)# lldp receive
  ```
- **R2**:
  ```plaintext
  R2(config)# lldp run
  R2(config)# interface range g0/0 - 2
  R2(config-if-range)# lldp transmit
  R2(config-if-range)# lldp receive
  ```
- **SW1**:
  ```plaintext
  SW1(config)# lldp run
  SW1(config)# interface g0/1
  SW1(config-if)# lldp transmit
  SW1(config-if)# lldp receive
  ```
- **R1**:
  ```plaintext
  R1(config)# lldp run
  R1(config)# interface range g0/0 - 2
  R1(config-if-range)# lldp transmit
  R1(config-if-range)# lldp receive
  ```

**Observations**:
- **Commands**:
  - `lldp run`: Enables LLDP globally (default: disabled on Cisco devices).
  - `lldp transmit`, `lldp receive`: Enable LLDP sending/receiving on interfaces (unlike CDP’s single `cdp enable`).
  - `interface range`: Efficiently configures multiple interfaces (e.g., R3 G0/0–2).
- **LLDP Behavior**: Sends frames every 30 seconds to `0180.C200.000E`, holdtime 120 seconds, reinitialization delay 2 seconds (not configurable in Packet Tracer).
- **Multi-Vendor**: LLDP prepares network for non-Cisco devices (e.g., Juniper), unlike Cisco-only CDP.
- **Verification**: Packet Tracer simulation mode shows LLDP frames (no CDP frames), alongside EIGRP and STP frames.

**Comparison to Prior Content**:
- **CDP/LLDP Lecture**: Explained LLDP’s `lldp transmit`, `lldp receive`; here, applied to replace CDP, with note that `lldp run` may auto-enable Tx/Rx on some devices (not in this lab).
- **Standard ACLs (Day 34)**: Configured `access-group` on interfaces; here, `lldp transmit/receive` enables LLDP per interface.
- **Extended ACLs (Day 35)**: Used `interface` commands for ACLs; similar interface-specific LLDP config.
- **IPv6 Parts 1–3**: Configured interfaces with IPv6 addresses; here, LLDP shares IPv4 addresses at Layer 2.

**Comparison to Other CCNA Topics**:
- **EIGRP**: LLDP doesn’t affect EIGRP connectivity (Layer 3); seen in simulation mode.
- **VTP**: CDP shares VTP info; LLDP does not, but both map topology.
- **Security**: Enabling LLDP maintains discovery but supports multi-vendor environments.

**Analogy**: Enabling LLDP is like launching a universal neighborhood directory, sending new business cards (frames) to all neighbors (routers, switches), welcoming future non-Cisco residents.

**Lifelong Retention Tip**: In Packet Tracer, enable LLDP on R1, verify `show lldp neighbors` shows SW1 on G0/2. Flashcard: “Enable LLDP? `lldp run`, `lldp transmit`, `lldp receive`.” Practice 3 times.

### Step 5: Verify Configurations
**Objective**: Confirm CDP is disabled and LLDP is enabled, verifying topology with LLDP.

**Actions**:
- **Check CDP Disabled**:
  ```plaintext
  R1# show cdp
  CDP is not enabled
  R1# show cdp neighbors
  [No output]
  SW1# show cdp interface f0/10
  [No output, confirming CDP disabled on PC interface]
  ```
- **Check LLDP Enabled**:
  ```plaintext
  R1# show lldp neighbors
  Capability Codes: B - Bridge, R - Router
  Device ID    Local Intrfce   Holdtme   Capability   Port ID
  SW1          Gig 0/2         120       B            Gig 0/1
  R2           Gig 0/1         120       R            Gig 0/0
  R3           Gig 0/0         120       R            Gig 0/1
  R1# show lldp interface
  GigabitEthernet0/0:
    Tx: Enabled
    Rx: Enabled
    Tx state: IDLE
    Rx state: WAIT FOR FRAME
  [Similar for G0/1, G0/2]
  R1# show lldp
  Global LLDP information:
    LLDP enabled
    LLDP timer: 30 seconds
    LLDP holdtime: 120 seconds
    LLDP reinitialization delay: 2 seconds
  ```
- **Simulation Mode**: In Packet Tracer, observe LLDP frames (destination MAC `0180.C200.000E`), no CDP frames (`0100.0CCC.CCCC`), alongside EIGRP and STP frames.

**Observations**:
- **CDP Verification**: `show cdp` confirms global disablement; `show cdp interface f0/10` on SW1 shows no output (CDP off).
- **LLDP Verification**: `show lldp neighbors` mirrors CDP’s topology (e.g., R1 G0/2–SW1 G0/1); `show lldp interface` confirms Tx/Rx enabled; `show lldp` shows default timers (30s/120s/2s).
- **Simulation**: Confirms LLDP frames sent/received, no CDP frames, validating transition.

**Comparison to Prior Content**:
- **CDP/LLDP Lecture**: Used `show cdp neighbors`, `show lldp neighbors` to verify; here, confirms topology after LLDP enablement.
- **Standard ACLs (Day 34)**: Used `show access-lists` for matches; here, `show lldp neighbors` verifies topology.
- **Extended ACLs (Day 35)**: Verified with `show access-lists`, pings; here, `show lldp neighbors`, simulation mode.
- **IPv6 Parts 1–3**: Verified with `ping 2001:db8::1`; here, LLDP verifies Layer 2 connections.

**Comparison to Other CCNA Topics**:
- **EIGRP**: Seen in simulation mode; LLDP doesn’t affect Layer 3 connectivity.
- **Spanning Tree**: STP frames in simulation mode; LLDP uses `Bridge` (B) capability for switches.
- **Security**: Disabling CDP, enabling LLDP balances discovery with multi-vendor support.

**Analogy**: Verifying CDP/LLDP is like checking a neighborhood directory: ensure the old Cisco newsletter (CDP) is stopped (`show cdp`) and the new universal one (LLDP) lists all neighbors (`show lldp neighbors`).

**Lifelong Retention Tip**: In Packet Tracer, run `show lldp neighbors` on R1, confirm SW1 on G0/2. Flashcard: “Verify LLDP? `show lldp neighbors`, `show lldp interface`.” Practice 3 times.

## Common Issues and Troubleshooting

- **No CDP Neighbors**:
  - **Issue**: `show cdp neighbors` shows no output before Step 3.
  - **Cause**: CDP disabled on interface (`no cdp enable`) or neighbor interface, or link down.
  - **Fix**: Verify `cdp enable` on both interfaces (e.g., R1 G0/2, SW1 G0/1), check `show interfaces g0/2` for up/up.
- **PC Interface Not Identified**:
  - **Issue**: Can’t find SW1 F0/10 to PC1.
  - **Cause**: Relying on CDP (PCs don’t support it).
  - **Fix**: Use `show interfaces status`, look for connected FastEthernet ports (e.g., F0/10).
- **Wrong IP Labels**:
  - **Issue**: R3 G0/1 labeled `10.0.13.1` (should be `.2`).
  - **Cause**: Misread `show cdp entry` or /30 subnet logic.
  - **Fix**: Confirm with `show cdp entry R3` (IP: `10.0.13.2`), note /30 has two hosts (`.1`, `.2`).
- **No LLDP Neighbors**:
  - **Issue**: `show lldp neighbors` shows no output.
  - **Cause**: Missing `lldp run`, `lldp transmit`, or `lldp receive` on local/neighbor interfaces.
  - **Fix**: Enable `lldp run`, `lldp transmit`, `lldp receive` on both sides (e.g., R1 G0/2, SW1 G0/1), verify with `show lldp interface`.
- **CDP Still Active**:
  - **Issue**: CDP frames in simulation mode after Step 3.
  - **Cause**: Missed `no cdp run` on a device.
  - **Fix**: Run `no cdp run` on all devices, verify `show cdp` (not enabled).

**Analogy**: Troubleshooting is like fixing a neighborhood directory: ensure the old newsletter (CDP) is off, the new one (LLDP) is sent to all neighbors, and all addresses (IPs, interfaces) are correctly listed.

**Lifelong Retention Tip**: Simulate no LLDP neighbors in Packet Tracer, check `show lldp interface`, enable `lldp transmit/receive`. Checklist: “No neighbors? Check `lldp run`, Tx/Rx, link status.” Practice 3 times.

## Practice Commands

- **Step 1: Map Topology**:
  ```plaintext
  PC1> ipconfig
  SW1# show cdp neighbors
  SW1# show interfaces status
  R1# show cdp neighbors
  R1# show interfaces g0/0
  R1# show interfaces g0/1
  R1# show cdp entry R3
  [Repeat for PC2, SW2, R2, PC3, SW3]
  ```
- **Step 2: Disable CDP on PC Interfaces**:
  ```plaintext
  SW1(config)# interface f0/10
  SW1(config-if)# no cdp enable
  SW1(config-if)# do show cdp interface f0/10
  SW2(config)# interface f0/1
  SW2(config-if)# no cdp enable
  SW3(config)# interface f0/24
  SW3(config-if)# no cdp enable
  ```
- **Step 3: Disable CDP Globally**:
  ```plaintext
  SW3(config)# no cdp run
  R3(config)# no cdp run
  SW2(config)# no cdp run
  R2(config)# no cdp run
  SW1(config)# no cdp run
  R1(config)# no cdp run
  ```
- **Step 4: Enable LLDP**:
  ```plaintext
  SW3(config)# lldp run
  SW3(config)# interface g0/1
  SW3(config-if)# lldp transmit
  SW3(config-if)# lldp receive
  R3(config)# lldp run
  R3(config)# interface range g0/0 - 2
  R3(config-if-range)# lldp transmit
  R3(config-if-range)# lldp receive
  [Repeat for SW2, R2, SW1, R1]
  ```
- **Step 5: Verify**:
  ```plaintext
  R1# show cdp
  R1# show cdp neighbors
  SW1# show cdp interface f0/10
  R1# show lldp
  R1# show lldp neighbors
  R1# show lldp interface
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - CDP: “Cisco-only, enabled, `0100.0CCC.CCCC`, 60s/180s.”
   - LLDP: “Multi-vendor, disabled, `0180.C200.000E`, 30s/120s/2s.”
   - Commands: “CDP off? `no cdp run`, `no cdp enable`. LLDP on? `lldp run`, `transmit`, `receive`.”
2. **Visualize Topology**:
   - Draw R1–SW1 (G0/2–G0/1), R1–R2 (G0/1–G0/0), label IPs (`10.0.12.1/30`, `192.168.1.254`). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “CDP MAC? `0100.0CCC.CCCC`. LLDP MAC? `0180.C200.000E`. Disable CDP? `no cdp enable`.”
4. **Practice in Packet Tracer**:
   - Map R1–R3 (G0/0–G0/1), disable CDP on SW1 F0/10, enable LLDP, verify `show lldp neighbors`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: CDP (Cisco, enabled, VTP) vs. LLDP (multi-vendor, disabled, no VTP).
   - Quiz: “Map topology? `show cdp neighbors`, `show interfaces status`.”
6. **Troubleshooting Practice**:
   - Simulate no LLDP neighbors, check `show lldp interface`, enable `lldp receive`. Checklist: “No neighbors? Check enable, Tx/Rx, link.”
7. **Teach Someone**:
   - Explain CDP topology mapping, disabling CDP, and enabling LLDP for multi-vendor support.
8. **Spaced Repetition**:
   - Use Anki for commands, MACs, and timers. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **CDP Topology Mapping**:
   - Use `show cdp neighbors` for router/switch links (e.g., R1 G0/2–SW1 G0/1), `show interfaces status` for PC ports (e.g., SW1 F0/10).
   - IPs from `ipconfig` (PCs), `show interfaces` (local), `show cdp entry` (neighbors).
2. **Disable CDP**:
   - `no cdp enable` on PC interfaces (SW1 F0/10, SW2 F0/1, SW3 F0/24) for security.
   - `no cdp run` globally to stop all CDP frames.
3. **Enable LLDP**:
   - `lldp run` globally, `lldp transmit`, `lldp receive` on router/switch interfaces for multi-vendor support.
4. **Verification**:
   - `show cdp` (disabled), `show lldp neighbors` (topology), `show lldp interface` (Tx/Rx), simulation mode (LLDP frames).
5. **CCNA Focus**:
   - Topic 2.3: Configure/verify CDP/LLDP, know MACs (`0100.0CCC.CCCC`, `0180.C200.000E`), timers, interface commands.

These notes provide a complete foundation for the CDP and LLDP lab, preparing you for CCNA exam topic 2.3. Practice mapping, disabling CDP, enabling LLDP, and troubleshooting to master Layer 2 discovery protocols for life!