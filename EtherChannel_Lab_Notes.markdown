# EtherChannel Lab Notes for CCNA Beginners

## Lab Overview

This lab configures EtherChannel in a network with redundant links between switches to increase bandwidth and ensure redundancy. The goal is to bundle multiple physical interfaces into a single logical interface (Port Channel) using different protocols (LACP, PAgP, and static) for Layer 2 and Layer 3 connections. The lab addresses the issue of Spanning Tree Protocol (STP) blocking redundant links and demonstrates how EtherChannel allows all links to forward traffic simultaneously.

**Lab Objectives**:
- Configure **Layer 2 EtherChannels** between access switches (ASW1, ASW2) and distribution switches (DSW1, DSW2) using LACP and PAgP.
- Configure a **Layer 3 EtherChannel** between distribution switches (DSW1, DSW2) using static mode.
- Enable IP routing and configure static routes for PC-to-server connectivity.
- Modify and verify the EtherChannel load-balancing method.
- Verify configurations using show commands.

**Network Topology**:
- **Access Switches (ASW1, ASW2)**: Connect end devices (PCs, servers) and link to distribution switches.
- **Distribution Switches (DSW1, DSW2)**: Aggregate traffic from access switches and connect to each other.
- **Connections**:
  - ASW1 to DSW1: Two links (LACP, Layer 2).
  - ASW2 to DSW2: Two links (PAgP, Layer 2).
  - DSW1 to DSW2: Two links (static, Layer 3).
  - PCs on ASW1 (VLAN1, subnet 172.16.1.0/24) connect to SRV1 on DSW2 (subnet 172.16.2.0/24).

**Analogy**: Think of the network as a city with roads (links) between neighborhoods (switches). STP is like a traffic cop closing redundant roads to prevent chaos (loops). EtherChannel combines these roads into a single superhighway, allowing more traffic (bandwidth) and keeping all roads open.

## Lab Steps and Configurations

### Step 1: Configure Layer 2 EtherChannel (ASW1 to DSW1 using LACP)

**Objective**: Bundle two links (GigabitEthernet0/1–0/2 on ASW1 to GigabitEthernet1/0/3–1/0/4 on DSW1) into a Layer 2 EtherChannel using LACP, configured as a trunk.

**Why LACP?**: LACP (Link Aggregation Control Protocol) is an IEEE standard (802.3ad) that dynamically negotiates EtherChannel formation, ensuring compatibility across vendors.

**Pre-Configuration Check**:
- On ASW1, check STP status to confirm the issue:
  ```bash
  show spanning-tree
  ```
  - **Output**: GigabitEthernet0/1 is the root port (forwarding), but GigabitEthernet0/2 is an alternate port (blocked by STP), wasting bandwidth.

**Configuration on ASW1**:
```bash
enable
conf t
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode active
exit
interface Port-channel1
 switchport mode trunk
exit
```
- **Explanation**:
  - `interface range`: Configures both interfaces simultaneously to ensure identical settings.
  - `channel-group 1 mode active`: Assigns interfaces to channel group 1 using LACP active mode (actively negotiates EtherChannel).
  - `interface Port-channel1`: Configures the virtual Port Channel interface as a trunk to carry VLAN traffic.
  - Trunk settings (e.g., `switchport mode trunk`) applied to Port-channel1 are automatically applied to member interfaces.

**Verification on ASW1**:
- Check running configuration:
  ```bash
  do show run
  ```
  - Confirms Port-channel1 creation and trunk settings on GigabitEthernet0/1–0/2.
- Check EtherChannel status:
  ```bash
  do show etherchannel summary
  ```
  - **Output**: Port-channel1 shows `SD` (Layer 2, Down), and member ports show `I` (stand-alone) because DSW1’s side isn’t configured yet.

**Configuration on DSW1**:
```bash
enable
conf t
interface range GigabitEthernet1/0/3 - 4
 channel-group 1 mode active
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
```
- **Explanation**:
  - Uses `active` mode for LACP (could use `passive` since ASW1 is active).
  - `switchport trunk encapsulation dot1q`: Specifies 802.1Q trunking (required on this switch model, unlike ASW1).
  - Configures Port-channel1 as a trunk.

**Verification on DSW1**:
- Check EtherChannel status:
  ```bash
  do show etherchannel summary
  ```
  - **Output**: Port-channel1 shows `SU` (Layer 2, in use), and member ports show `P` (bundled), indicating a working EtherChannel.
- Check trunk status:
  ```bash
  do show interfaces trunk
  ```
  - **Output**: Port-channel1 is listed as a trunk, not individual interfaces.
- Check STP on ASW1:
  ```bash
  do show spanning-tree
  ```
  - **Output**: Port-channel1 (Po1) is the root port, replacing GigabitEthernet0/1–0/2, confirming STP treats the EtherChannel as one link.

**Analogy**: Configuring LACP is like two people (switches) agreeing to team up by shaking hands (active mode). Both enthusiastically agree (active + active), forming a strong team (EtherChannel) that carries more packages (traffic) as a single unit (Port Channel).

### Step 2: Configure Layer 2 EtherChannel (ASW2 to DSW2 using PAgP)

**Objective**: Bundle two links (GigabitEthernet0/1–0/2 on ASW2 to GigabitEthernet1/0/3–1/0/4 on DSW2) into a Layer 2 EtherChannel using PAgP, configured as a trunk.

**Why PAgP?**: PAgP (Port Aggregation Protocol) is Cisco-proprietary, similar to LACP but limited to Cisco devices.

**Configuration on ASW2**:
```bash
enable
conf t
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode desirable
exit
interface Port-channel1
 switchport mode trunk
exit
```
- **Explanation**:
  - `channel-group 1 mode desirable`: Uses PAgP desirable mode (actively negotiates EtherChannel).
  - Configures Port-channel1 as a trunk.

**Configuration on DSW2**:
```bash
enable
conf t
interface range GigabitEthernet1/0/3 - 4
 channel-group 1 mode desirable
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
```
- **Explanation**:
  - Uses `desirable` mode for PAgP (could use `auto` since ASW2 is desirable).
  - Specifies 802.1Q trunking for this switch model.

**Verification on DSW2**:
- Check EtherChannel status:
  ```bash
  do show etherchannel summary
  ```
  - **Output**: Port-channel1 shows `SU` (Layer 2, in use), and member ports show `P` (bundled).

**Analogy**: PAgP is like a Cisco-only club where members (switches) eagerly invite each other (desirable mode) to form a team (EtherChannel). The team works as one unit (Port Channel) to deliver more packages (traffic).

### Step 3: Configure Layer 3 EtherChannel (DSW1 to DSW2 using Static Mode)

**Objective**: Bundle two links (GigabitEthernet1/0/1–1/0/2 on DSW1 and DSW2) into a Layer 3 EtherChannel using static mode, with IP addresses for routing.

**Why Static Mode?**: Static mode (`on`) manually forms the EtherChannel without negotiation, used here for a Layer 3 connection to avoid STP.

**Configuration on DSW2**:
```bash
enable
conf t
interface range GigabitEthernet1/0/1 - 2
 no switchport
 channel-group 2 mode on
exit
interface Port-channel2
 ip address 10.0.0.2 255.255.255.252
exit
```
- **Explanation**:
  - `no switchport`: Makes interfaces routed (Layer 3).
  - `channel-group 2 mode on`: Uses static mode (no negotiation); channel-group 2 avoids conflict with channel-group 1.
  - Assigns IP address to Port-channel2 (not member interfaces).

**Configuration on DSW1**:
```bash
enable
conf t
interface range GigabitEthernet1/0/1 - 2
 no switchport
 channel-group 2 mode on
exit
interface Port-channel2
 ip address 10.0.0.1 255.255.255.252
exit
```
- **Explanation**: Same as DSW2, with a different IP address.

**Verification on DSW1**:
- Check EtherChannel status:
  ```bash
  do show etherchannel summary
  ```
  - **Output**: Port-channel2 shows `RU` (Layer 3, in use), and member ports show `P` (bundled).
- Test connectivity:
  ```bash
  do ping 10.0.0.2
  ```
  - **Output**: Successful ping confirms the Layer 3 EtherChannel is operational.

**Analogy**: A Layer 3 EtherChannel is like a high-speed internet link between two data centers (DSW1, DSW2). Static mode is like manually connecting the cables without negotiation, and assigning IPs makes it a direct router-to-router link, bypassing traffic cops (STP).

### Step 4: Configure Static Routes for PC-to-Server Connectivity

**Objective**: Enable IP routing on DSW1 and DSW2 and configure static routes so PCs (172.16.1.0/24 on ASW1) can reach SRV1 (172.16.2.1 on DSW2).

**Pre-Configuration Check**:
- On DSW1, check routing table:
  ```bash
  do show ip route
  ```
  - **Output**: No routing table because IP routing is disabled.

**Configuration on DSW1**:
```bash
enable
conf t
ip routing
ip route 172.16.2.0 255.255.255.0 10.0.0.2
exit
```
- **Explanation**:
  - `ip routing`: Enables Layer 3 routing on the multilayer switch.
  - `ip route`: Adds a static route to SRV1’s subnet (172.16.2.0/24) via DSW2’s Port-channel2 IP (10.0.0.2).

**Verification on DSW1**:
- Check routing table:
  ```bash
  do show ip route
  ```
  - **Output**: Shows routes for VLAN1 SVI, Port-channel2, and the static route to 172.16.2.0/24.

**Configuration on DSW2**:
```bash
enable
conf t
ip routing
ip route 172.16.1.0 255.255.255.0 10.0.0.1
exit
```
- **Explanation**: Adds a static route to the PCs’ subnet (172.16.1.0/24) via DSW1’s Port-channel2 IP (10.0.0.1).

**Verification on DSW2**:
- Check routing table:
  ```bash
  do show ip route
  ```
  - **Output**: Shows routes for SRV1’s subnet and the static route to 172.16.1.0/24.

**Test Connectivity**:
- From PC1 (on ASW1), ping SRV1 (172.16.2.1):
  ```bash
  ping 172.16.2.1
  ```
  - **Output**: Initial timeouts may occur due to ARP, but subsequent pings succeed, confirming routing via the Layer 3 EtherChannel.

**Analogy**: Static routes are like giving a delivery driver (switch) a map to reach a destination (SRV1) via a specific road (Port-channel2). Enabling `ip routing` is like turning on the GPS to make the driver a router.

### Step 5: Check and Modify EtherChannel Load Balancing

**Objective**: Check the default load-balancing method on ASW1, ASW2, DSW1, and DSW2, then change it to source and destination IP addresses.

**Check Default Load Balancing**:
- On ASW1:
  ```bash
  do show etherchannel load-balance
  ```
  - **Output**: Default is `src-mac` (source MAC address).
- On DSW1:
  ```bash
  do show etherchannel load-balance
  ```
  - **Output**: Default is `src-mac`.

**Configure Load Balancing on ASW1**:
```bash
enable
conf t
port-channel load-balance src-dst-ip
exit
```
- **Verification**:
  ```bash
  do show etherchannel load-balance
  ```
  - **Output**: Confirms `src-dst-ip` (source and destination IP addresses).

**Configure Load Balancing on ASW2**:
```bash
enable
conf t
port-channel load-balance src-dst-ip
exit
do show etherchannel load-balance
```
- **Output**: Confirms `src-dst-ip`.

**Configure Load Balancing on DSW1**:
```bash
enable
conf t
port-channel load-balance src-dst-ip
exit
do show etherchannel load-balance
```
- **Output**: Confirms `src-dst-ip`.

**Configure Load Balancing on DSW2**:
```bash
enable
conf t
port-channel load-balance src-dst-ip
exit
do show etherchannel load-balance
```
- **Output**: Confirms `src-dst-ip`.

**Explanation**:
- Default load balancing (`src-mac`) sends all frames from the same source MAC address over the same physical link.
- Changing to `src-dst-ip` uses both source and destination IP addresses, providing more granular distribution, especially for diverse traffic (e.g., multiple PCs to servers).

**Analogy**: Load balancing is like a mail sorter (switch) deciding which conveyor belt (physical link) to send packages (frames) on. Using `src-mac` is like sorting by sender’s name, while `src-dst-ip` sorts by both sender and recipient, spreading packages more evenly across belts.

## Key Takeaways

1. **EtherChannel Benefits**:
   - Combines multiple links into one logical interface (Port Channel), increasing bandwidth and providing redundancy.
   - Prevents STP from blocking redundant links, allowing all to forward traffic.
2. **Layer 2 vs. Layer 3**:
   - Layer 2 EtherChannel (trunk): Used between access and distribution switches for VLAN traffic (ASW1–DSW1, ASW2–DSW2).
   - Layer 3 EtherChannel (routed): Used between distribution switches for IP routing, bypassing STP (DSW1–DSW2).
3. **Protocols**:
   - **LACP**: Industry-standard, used active/active for ASW1–DSW1.
   - **PAgP**: Cisco-proprietary, used desirable/desirable for ASW2–DSW2.
   - **Static**: Manual configuration, used for DSW1–DSW2 Layer 3 link.
4. **Verification Flags**:
   - `SU`: Layer 2, in use (ideal for Layer 2 EtherChannel).
   - `RU`: Layer 3, in use (ideal for Layer 3 EtherChannel).
   - `P`: Physical interface bundled in Port Channel.
   - `I`: Stand-alone (issue, e.g., other side not configured).
   - `SD`: Layer 2, down (issue, e.g., other side not configured).
5. **Load Balancing**: Changed from `src-mac` to `src-dst-ip` for better traffic distribution.
6. **Routing**: Enabled `ip routing` and added static routes for PC-to-server connectivity via the Layer 3 EtherChannel.

## Common Issues and Troubleshooting

- **Stand-Alone Ports (I Flag)**: Occurs if the other switch isn’t configured. Fix by ensuring both sides have compatible modes (e.g., active/passive for LACP).
- **Down EtherChannel (SD Flag)**: Indicates a configuration mismatch or unconfigured peer. Verify both switches’ configurations.
- **Trunk Encapsulation**: Some switches (e.g., DSW1, DSW2) require `switchport trunk encapsulation dot1q` before `switchport mode trunk`.
- **Ping Failures**: Initial timeouts may occur due to ARP resolution. Ensure `ip routing` is enabled and static routes are correct.

**Analogy**: Troubleshooting is like checking why a team (EtherChannel) isn’t working. If members (ports) are marked “I” (stand-alone), it’s like team members not talking to the other side. Ensure both sides agree to work together (compatible modes) and follow the same rules (trunk or routed settings).

## Practice Commands

- **Configure EtherChannel**:
  ```bash
  interface range <interface-list>
   channel-group <number> mode <active | passive | desirable | auto | on>
  ```
- **Configure Trunk (Layer 2)**:
  ```bash
  interface Port-channel<number>
   switchport trunk encapsulation dot1q
   switchport mode trunk
  ```
- **Configure Routed Port (Layer 3)**:
  ```bash
  interface range <interface-list>
   no switchport
   channel-group <number> mode <mode>
  interface Port-channel<number>
   ip address <ip> <mask>
  ```
- **Enable IP Routing**:
  ```bash
  ip routing
  ip route <destination> <mask> <next-hop>
  ```
- **Verify**:
  ```bash
  show etherchannel summary
  show etherchannel load-balance
  show interfaces trunk
  show spanning-tree
  show ip route
  ping <destination>
  ```

## Study Tips for CCNA

- **Lab Practice**: Replicate this lab in Packet Tracer or GNS3 to practice configurations and verify outputs.
- **Memorize Flags**: Understand `SU`, `RU`, `P`, `I`, and `SD` for `show etherchannel summary`.
- **Understand Protocols**: Know the differences between LACP (active/passive), PAgP (desirable/auto), and static (on).
- **Focus on Verification**: Practice `show` commands to interpret EtherChannel status and troubleshoot issues.
- **Use Analogies**: Think of EtherChannel as a superhighway (bandwidth) or a team (Port Channel) to make configurations intuitive.

These notes cover all aspects of the EtherChannel lab, from setup to verification, ensuring you have a complete resource for CCNA preparation. Practice the commands and review the analogies to solidify your understanding!