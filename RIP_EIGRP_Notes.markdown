# RIP and EIGRP Notes for CCNA Beginners

## Introduction

This lesson covers **RIP** (Routing Information Protocol) and **EIGRP** (Enhanced Interior Gateway Routing Protocol), two dynamic routing protocols that are not explicitly listed in the CCNA exam topics but may appear in questions. The focus is on providing a basic understanding of their functionality, configuration, and comparison to **OSPF** (the primary CCNA protocol). These protocols are simpler to configure than OSPF, making them a good introduction to dynamic routing concepts.

**Why Study RIP and EIGRP?**:
- Potential exam questions may test basic knowledge or comparison with OSPF.
- Concepts like the `network` command, passive interfaces, and default route advertisement apply to OSPF, aiding future learning.
- Both are **Interior Gateway Protocols (IGPs)** and use **distance vector** algorithms, contrasting with OSPF’s link-state approach.

**Analogy**: Dynamic routing protocols are like different navigation apps (e.g., Google Maps for OSPF, simpler apps for RIP and EIGRP). RIP is a basic app with limited features, EIGRP is more advanced but less universal, and OSPF is the industry-standard app used by most networks.

## RIP (Routing Information Protocol)

### Overview
- **Type**: Distance vector IGP, industry-standard (not Cisco-proprietary).
- **Metric**: **Hop count** (each router is one hop, regardless of link speed).
  - Example: A 10 Mbps link and a 10 Gbps link both count as one hop.
  - Maximum hop count: 15 (routes with >15 hops are unreachable).
- **Operation**: Uses **routing by rumor**, where routers share their routing tables with neighbors without a full network view.
- **Use Case**: Rarely used in modern networks due to limitations but suitable for small networks or labs due to simplicity.
- **Versions**:
  - **RIPv1**: Obsolete, supports only classful addressing (e.g., /8, /16, /24).
  - **RIPv2**: Supports **VLSM** (Variable Length Subnet Masking) and **CIDR** (Classless Inter-Domain Routing).
  - **RIPng**: For IPv6, not covered in CCNA.
- **Messages**:
  - **Request**: Asks neighbors for their routing table.
  - **Response**: Sends the local routing table to neighbors (every 30 seconds by default).
  - **RIPv1**: Broadcasts to 255.255.255.255 (all devices on the segment).
  - **RIPv2**: Multicasts to 224.0.0.9 (only RIP-enabled devices).

**RIPv1 Limitations**:
- Classful addressing: Converts subnets to classful networks (e.g., 10.1.1.0/24 → 10.0.0.0/8, 172.16.192.0/18 → 172.16.0.0/16, 192.168.1.4/30 → 192.168.1.0/24).
- No subnet mask in advertisements, assuming classful masks (/8, /16, /24).
- Incompatible with modern networks using VLSM/CIDR.

**RIPv2 Improvements**:
- Supports VLSM/CIDR by including subnet masks in advertisements.
- Uses multicast (224.0.0.9) instead of broadcast, reducing network traffic.

**Analogy**: RIP is like passing notes in a classroom (network). Each student (router) tells their neighbor about destinations they know (routing table), but they don’t see the whole school (network topology). RIPv1 uses a megaphone (broadcast), while RIPv2 whispers to specific friends (multicast).

### Basic RIP Configuration (on R1)
**Topology**:
- R1 interfaces: G0/0 (10.0.12.1/30 to R2), G1/0 (10.0.13.1/30 to R3), G2/0 (172.16.1.14/28 to LAN), G3/0 (203.0.113.1/30 to Internet).
- Goal: Enable RIP on R1 to advertise its networks and learn routes from R2, R3, and R4.

**Configuration**:
```bash
enable
conf t
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 network 172.16.0.0
 passive-interface GigabitEthernet2/0
 default-information originate
exit
```
- **Explanation**:
  - `router rip`: Enters RIP configuration mode (`config-router`).
  - `version 2`: Enables RIPv2 for VLSM/CIDR support.
  - `no auto-summary`: Disables automatic classful summarization (e.g., prevents 172.16.1.0/28 from being advertised as 172.16.0.0/16).
  - `network 10.0.0.0`: Activates RIP on interfaces in the 10.0.0.0/8 range (G0/0, G1/0). Advertises 10.0.12.0/30 and 10.0.13.0/30.
  - `network 172.16.0.0`: Activates RIP on G2/0 (172.16.1.14/28). Advertises 172.16.1.0/28.
  - `passive-interface GigabitEthernet2/0`: Stops RIP advertisements on G2/0 (no RIP neighbors), but still advertises 172.16.1.0/28 to R2/R3.
  - `default-information originate`: Advertises R1’s default route (0.0.0.0/0 via 203.0.113.2) to RIP neighbors.

**Network Command Details**:
- Classful: Converts to classful networks (e.g., `network 10.0.12.0` → 10.0.0.0/8).
- Function: Identifies interfaces to activate RIP, not the networks to advertise.
- Example: `network 10.0.0.0` matches G0/0 (10.0.12.1) and G1/0 (10.0.13.1), enabling RIP and advertising their /30 prefixes.

**Verification**:
```bash
show ip protocols
```
- **Output**:
  - Protocol: RIP, version 2.
  - Auto-summarization: Disabled.
  - Maximum paths: 4 (ECMP load-balancing).
  - Networks: 10.0.0.0, 172.16.0.0.
  - Passive interface: G2/0.
  - Neighbors: R2 (10.0.12.2), R3 (10.0.13.2).
  - AD: 120.
```bash
show ip route
```
- **Output**: RIP routes marked with `R`, e.g., `R 192.168.4.0/24 [120/2] via 10.0.12.2` (2 hops via R2).

**Additional Configuration**:
- Change maximum paths:
  ```bash
  router rip
   maximum-paths 8
  ```
  - Allows up to 8 equal-cost paths for ECMP load-balancing.
- Change AD:
  ```bash
  router rip
   distance 85
  ```
  - Sets RIP’s AD to 85 (e.g., to prefer RIP over EIGRP’s AD 90).

**Analogy**: Configuring RIP is like setting up a group chat (RIP process) where you choose which phones (interfaces) join (`network`) and stop one phone from sending messages (`passive-interface`). You also share a shortcut to the Internet (`default-information originate`).

## EIGRP (Enhanced Interior Gateway Routing Protocol)

### Overview
- **Type**: Advanced/hybrid distance vector IGP, Cisco-proprietary (partially open but primarily Cisco-only).
- **Metric**: Based on **bandwidth** (slowest link) and **delay** (sum of all links), configurable for other factors (e.g., load, reliability).
- **Operation**: Distance vector with improvements, faster convergence than RIP, supports large networks (no hop limit).
- **Messages**: Multicasts to 224.0.0.10 (vs. RIPv2’s 224.0.0.9).
- **Unique Feature**: **Unequal-cost load-balancing**, allowing traffic distribution across paths with different metrics, proportional to bandwidth (covered in the lab video).
- **Use Case**: Preferred over RIP in Cisco environments but less common than OSPF due to vendor specificity.

**Analogy**: EIGRP is like an upgraded navigation app compared to RIP’s basic version. It considers road speed (bandwidth) and travel time (delay), can handle bigger cities (networks), and even splits traffic across roads of different speeds (unequal-cost load-balancing).

### Basic EIGRP Configuration (on R1)
**Topology**: Same as RIP (R1: G0/0 10.0.12.1/30, G1/0 10.0.13.1/30, G2/0 172.16.1.14/28, G3/0 203.0.113.1/30).

**Configuration**:
```bash
enable
conf t
router eigrp 1
 no auto-summary
 network 10.0.0.0
 network 172.16.1.0 0.0.0.15
 passive-interface GigabitEthernet2/0
 eigrp router-id 1.1.1.1
exit
```
- **Explanation**:
  - `router eigrp 1`: Enters EIGRP configuration mode with AS number 1 (must match on all routers).
  - `no auto-summary`: Disables classful summarization (may be disabled by default).
  - `network 10.0.0.0`: Activates EIGRP on interfaces in 10.0.0.0/8 (G0/0, G1/0), advertising 10.0.12.0/30 and 10.0.13.0/30.
  - `network 172.16.1.0 0.0.0.15`: Activates EIGRP on G2/0 (172.16.1.14/28) using a wildcard mask (0.0.0.15 = /28).
  - `passive-interface GigabitEthernet2/0`: Stops EIGRP advertisements on G2/0, but advertises 172.16.1.0/28.
  - `eigrp router-id 1.1.1.1`: Manually sets the router ID (overrides default).

**Wildcard Masks**:
- **Definition**: Inverted subnet mask (1s become 0s, 0s become 1s).
- **Examples**:
  - /24 (255.255.255.0) → 0.0.0.255
  - /16 (255.255.0.0) → 0.0.255.255
  - /8 (255.0.0.0) → 0.255.255.255
  - /28 (255.255.255.240) → 0.0.0.15
  - /25 (255.255.255.128) → 0.0.0.127
  - /14 (255.252.0.0) → 0.3.255.255
  - /19 (255.255.224.0) → 0.0.31.255
  - /21 (255.255.248.0) → 0.0.7.255
- **Shortcut**: Subtract each octet from 255 (e.g., 255.255.255.240 → 255-255=0, 255-255=0, 255-255=0, 255-240=15 → 0.0.0.15).
- **Function**: In `network` command, 0s mean bits must match, 1s mean bits don’t matter.
  - Example: `network 172.16.1.0 0.0.0.15` matches 172.16.1.14/28 (first 28 bits match).
  - Example: `network 172.16.1.0 0.0.0.7` doesn’t match (29th bit differs).
  - Example: `network 168.0.0.0 7.255.255.255` matches 172.16.1.14 (first 5 bits match).

**Router ID**:
- **Priority**:
  1. Manually configured (`eigrp router-id <id>`).
  2. Highest loopback interface IP.
  3. Highest physical interface IP (e.g., 172.16.1.14 for R1).
- **Format**: 32-bit number in dotted-decimal (e.g., 1.1.1.1), not necessarily an IP address.

**Verification**:
```bash
show ip protocols
```
- **Output**:
  - Protocol: EIGRP, AS 1.
  - Metric: K1 (bandwidth), K3 (delay) enabled.
  - Router ID: 1.1.1.1 (manually set).
  - Auto-summarization: Disabled.
  - Maximum paths: 4.
  - Networks: 10.0.0.0, 172.16.1.0/28.
  - Passive interface: G2/0.
  - Neighbors: R2 (10.0.12.2), R3 (10.0.13.2).
  - AD: 90 (internal), 170 (external).
```bash
show ip route
```
- **Output**: EIGRP routes marked with `D`, e.g., `D 192.168.4.0/24 [90/3072] via 10.0.12.2` (higher metrics than RIP/OSPF).

**Analogy**: EIGRP configuration is like setting up a smarter group chat than RIP. You specify the chat group (AS), choose which phones join (`network` with wildcard masks), set a nickname (router ID), and mute unnecessary chatter (`passive-interface`).

## Key Comparisons

| **Feature**               | **RIP**                              | **EIGRP**                            |
|---------------------------|--------------------------------------|--------------------------------------|
| **Type**                  | Distance vector IGP                 | Advanced distance vector IGP         |
| **Metric**                | Hop count (max 15)                  | Bandwidth (slowest link) + delay     |
| **Messages**              | RIPv1: Broadcast (255.255.255.255)<br>RIPv2: Multicast (224.0.0.9) | Multicast (224.0.0.10)              |
| **VLSM/CIDR Support**     | RIPv1: No<br>RIPv2: Yes             | Yes                                  |
| **Network Command**       | Classful (e.g., 10.0.0.0/8)         | Classful or with wildcard mask       |
| **AD**                    | 120                                 | 90 (internal), 170 (external)        |
| **Load Balancing**        | ECMP (up to 4 paths)                | ECMP (up to 4 paths) + unequal-cost  |
| **Use Case**              | Small networks, labs                | Cisco networks, larger scale         |

**Analogy**: RIP is a basic bicycle (simple but limited), while EIGRP is a motorcycle (faster, more features, but Cisco-specific). OSPF is a car (industry-standard, widely used).

## Key Takeaways

1. **RIP**:
   - Simple, distance vector protocol using hop count.
   - RIPv2 is preferred (VLSM/CIDR, multicast).
   - Configure with `version 2`, `no auto-summary`, `network`, `passive-interface`, and `default-information originate`.
   - Limited by 15-hop maximum, rarely used in production.
2. **EIGRP**:
   - Advanced distance vector, uses bandwidth/delay for metric.
   - Supports unequal-cost load-balancing, Cisco-proprietary.
   - Uses wildcard masks in `network` command, similar to OSPF.
   - Configure with AS number, `no auto-summary`, `router-id`.
3. **Verification**:
   - `show ip protocols`: Displays protocol details, neighbors, AD.
   - `show ip route`: Shows routes with AD/metric (RIP: `R`, EIGRP: `D`).
4. **CCNA Relevance**:
   - Know RIP and EIGRP basics for comparison with OSPF.
   - Memorize ADs (RIP: 120, EIGRP: 90/170) and multicast addresses.
   - Understand wildcard masks and passive interfaces (applies to OSPF).

## Quiz for Practice

1. **R1 and R2 use RIP. How does R1 advertise its default route to R2?**
   - A. `default-information originate` (config-router)
   - B. `network 203.0.113.0` (config-router)
   - C. `ip route 0.0.0.0 0.0.0.0 10.0.12.1` (global config on R2)
   - D. `default-information originate` (config-router on R2)
   - **Answer**: A. Advertises R1’s default route via RIP.

2. **R1’s G1/0 (172.20.20.17) and G2/0 (172.26.20.12). Which EIGRP network command activates both?**
   - A. `network 128.0.0.0 127.255.255.255`
   - B. (Other options not shown)
   - **Answer**: A. Matches first bit (both IPs start with 1 in binary).

3. **EIGRP router ID priority order?**
   - A. Highest loopback, highest physical, manual
   - B. Highest physical, highest loopback, manual
   - C. Manual, highest physical, highest loopback
   - D. Manual, highest loopback, highest physical
   - **Answer**: D. Manual configuration takes priority, then loopback, then physical.

**Analogy for Quiz**: The quiz is like a pop quiz on which navigation app (protocol) to use and how to set it up. Know the app’s settings (commands) and how it picks routes (metrics, AD).

## Study Tips for CCNA

- **Focus on Comparisons**: Understand RIP (simple, hop count) vs. EIGRP (advanced, bandwidth/delay) vs. OSPF (link-state, covered next).
- **Memorize Key Values**: AD (RIP: 120, EIGRP: 90/170), multicast addresses (RIPv2: 224.0.0.9, EIGRP: 224.0.0.10).
- **Practice Wildcard Masks**: Convert subnet masks to wildcard masks (e.g., 255.255.255.240 → 0.0.0.15).
- **Lab Practice**: Use Packet Tracer to configure RIP and EIGRP, focusing on `network`, `passive-interface`, and `show ip protocols`.
- **Use Analogies**: Think of RIP as gossip, EIGRP as a smarter gossip with preferences, and OSPF as a shared map.

These notes provide a solid foundation for understanding RIP and EIGRP, preparing you for potential CCNA exam questions and easing the transition to OSPF. Practice the configurations and review the analogies to solidify your understanding!