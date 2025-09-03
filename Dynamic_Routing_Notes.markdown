# Dynamic Routing Notes for CCNA Beginners

## Introduction to Dynamic Routing

Dynamic routing involves configuring a routing protocol on routers, allowing them to automatically discover and maintain routes to destination networks. Unlike static routing, where routes are manually configured using the `ip route` command, dynamic routing protocols enable routers to adapt to network changes, such as new networks or link failures, without manual intervention.

**Key Benefits**:
- **Automation**: Routers share route information, eliminating the need to manually configure routes for every destination.
- **Adaptability**: Automatically updates routes when a new network is added or a path becomes unavailable.
- **Redundancy**: Supports multiple paths to a destination, switching to a backup path if the primary fails.

**Analogy**: Think of static routing as giving a driver a fixed set of directions to a destination, which never change. Dynamic routing is like using a GPS that updates routes in real-time based on traffic conditions, road closures, or new roads, guiding the driver to the best path.

**CCNA Exam Relevance**:
- Dynamic routing is part of the **IP Connectivity** section (3.0), accounting for 25% of the CCNA exam.
- The exam emphasizes **OSPF** (Open Shortest Path First, topic 3.4), but you need a basic understanding of other protocols (RIP, EIGRP, BGP) for comparison.

## Static vs. Dynamic Routing

- **Static Routing** (covered in Day 11):
  - Manually configured using `ip route <destination> <mask> <next-hop>`.
  - Example: `ip route 192.168.4.0 255.255.255.0 10.0.12.2`.
  - **Pros**: Simple for small networks, predictable.
  - **Cons**: Doesn’t adapt to changes (e.g., link failures), impractical for large networks.
  - **Example Issue**: If a link fails, the router continues forwarding packets to a dead-end until manually reconfigured.
- **Dynamic Routing**:
  - Routers use protocols to share route information and select the best paths.
  - Adapts to network changes (e.g., removes invalid routes, uses backup paths).
  - **Example**: If a link fails, routers automatically update their routing tables to use an alternate path.

**Analogy**: Static routing is like a paper map—fixed and unchanging. Dynamic routing is like a live navigation app that reroutes you if a road is blocked.

## Dynamic Routing Protocols Overview

Dynamic routing protocols allow routers to advertise their connected networks and learn routes from neighbors, forming **adjacencies** (neighbor relationships) to exchange routing information. The protocols determine the best route to each destination based on a **metric** and prioritize routes using **administrative distance (AD)**.

**Network Topology Example**:
- Four routers: R1, R2, R3, R4.
- R4 has a LAN (192.168.4.0/24).
- R1 connects to R2 (10.0.12.0/30) and R3 (10.0.13.0/30).
- R2 connects to R4 (10.0.24.0/30).
- R3 connects to R4 (FastEthernet, slower link).

**How Dynamic Routing Works**:
1. R4 advertises 192.168.4.0/24 to R2.
2. R2 adds the route and advertises it to R1, along with 10.0.24.0/30.
3. R1 adds these routes to its routing table and advertises them to R3.
4. If R4’s interface to 192.168.4.0/24 fails, routers automatically remove the route.
5. If a backup path exists (e.g., via R3), routers switch to it.

**Analogy**: Dynamic routing is like a group of friends (routers) sharing directions to a party (destination network). Each friend tells others how to get there, and if a road (link) is closed, they quickly share an alternate route.

## Types of Dynamic Routing Protocols

Dynamic routing protocols are categorized by their scope and algorithm type.

### By Scope
1. **Interior Gateway Protocols (IGPs)**:
   - Used within a single **autonomous system (AS)**, a network under one organization’s control (e.g., a company).
   - Examples: RIP, EIGRP, OSPF, IS-IS.
2. **Exterior Gateway Protocols (EGPs)**:
   - Used between different autonomous systems (e.g., between companies or ISPs).
   - Example: BGP (Border Gateway Protocol), the only EGP used today.

**Analogy**: IGPs are like city buses sharing routes within a city (AS). EGPs are like international flights connecting different cities (ASs).

### By Algorithm Type
1. **Distance Vector Protocols**:
   - Protocols: **RIP** (Routing Information Protocol), **EIGRP** (Enhanced Interior Gateway Routing Protocol).
   - Operation: Share their routing table (or parts of it) with directly connected neighbors, including destination networks and metrics. Known as **routing by rumor** because routers rely on neighbors’ information without a full network view.
   - Characteristics:
     - Simpler, less resource-intensive (less CPU/memory).
     - Slower to react to network changes.
     - Example: R4 tells R2, “Reach 192.168.4.0/24 via me, metric 1.” R2 tells R1, “Reach it via me, metric 2.”
   - **Analogy**: Distance vector is like gossip—each person (router) passes along what they heard (routes) from their friend (neighbor), but they don’t know the full story (network topology).
2. **Link State Protocols**:
   - Protocols: **OSPF** (Open Shortest Path First), **IS-IS** (Intermediate System to Intermediate System).
   - Operation: Each router builds a **connectivity map** of the network by advertising its interfaces and connected networks. All routers share these advertisements, creating an identical map, then independently calculate the best routes.
   - Characteristics:
     - More resource-intensive (higher CPU/memory).
     - Faster to react to network changes.
     - Example: Each router shares its connections, forming a complete network map, like a puzzle.
   - **Analogy**: Link state is like everyone in a city sharing a detailed map (topology) with landmarks (routers) and roads (links), allowing each person to plan the best route independently.
3. **Path Vector Protocol**:
   - Protocol: **BGP** (Border Gateway Protocol).
   - Operation: Maintains a table of paths (AS paths) to destinations, used for inter-AS routing.
   - CCNA Relevance: Limited; know it’s the only EGP and used between ASs.

**Key Memory Points**:
- **Distance Vector**: RIP, EIGRP.
- **Link State**: OSPF, IS-IS.
- **Path Vector**: BGP.
- **CCNA Focus**: OSPF (exam topic 3.4), with basic understanding of RIP, EIGRP, and BGP for comparison.

## Routing Metrics

A **metric** is a value used by a routing protocol to determine the best route to a destination. Lower metrics indicate better routes, similar to STP’s root cost.

**Metrics by Protocol**:
1. **RIP**: **Hop count**.
   - Each router in the path is one hop.
   - Example: R1 to 192.168.4.0/24 via R2 = 2 hops (R1→R2→R4).
   - Drawback: Treats all links equally (e.g., 10 Mbps and 10 Gbps links are both 1 hop).
   - **Analogy**: Hop count is like counting how many friends you need to ask to get a message to someone, ignoring how fast each friend can deliver it.
2. **EIGRP**: **Bandwidth and delay** (by default).
   - Uses the slowest link’s bandwidth and the sum of delays across all links.
   - Configurable to include other factors (e.g., load, reliability).
   - **Analogy**: EIGRP is like choosing a delivery route based on the slowest road (bandwidth) and total travel time (delay).
3. **OSPF**: **Cost**.
   - Calculated based on link bandwidth (higher bandwidth = lower cost).
   - Total cost is the sum of costs across the path.
   - Example: GigabitEthernet links have lower cost than FastEthernet.
   - **Analogy**: OSPF is like choosing a highway based on speed limits (bandwidth), preferring faster roads.
4. **IS-IS**: **Cost** (default = 10 per link).
   - Without configuration, acts like hop count (all links equal).
   - CCNA Relevance: Minimal; know it’s a link state protocol.
   - **Analogy**: IS-IS is like a basic map where every road has the same cost unless customized.

**Example**:
- R1 learns two routes to 192.168.4.0/24:
  - Via R2 (GigabitEthernet links, lower cost).
  - Via R3 (FastEthernet link, higher cost).
- Using RIP: Both routes have 2 hops, so both are added (equal cost).
- Using OSPF: The route via R2 has a lower cost (higher bandwidth), so only it is added.

**Equal Cost MultiPath (ECMP)**:
- If two routes to the same destination (same network and prefix) have the same metric and are learned via the same protocol, both are added to the routing table, and traffic is load-balanced.
- Example: R1’s OSPF routes to 192.168.4.0/24 via R2 and R3 (both metric 3) are both added, enabling ECMP.
- Static routes can also use ECMP if configured with the same destination and next hops.

**Analogy**: ECMP is like a delivery company using two equally fast routes to the same destination, splitting packages between them to speed up delivery.

## Administrative Distance (AD)

**Definition**: Administrative distance is a value that determines the preference of a route’s source (e.g., routing protocol, static route) when multiple sources advertise routes to the same destination. Lower AD is preferred, indicating a more trustworthy source.

**AD Values (Cisco Devices)**:
- **Connected Routes**: 0 (most preferred, directly connected networks).
- **Static Routes**: 1.
- **External BGP (eBGP)**: 20.
- **EIGRP**: 90.
- **IGRP**: 100 (older, replaced by EIGRP).
- **OSPF**: 110.
- **IS-IS**: 115.
- **RIP**: 120.
- **EIGRP External**: 170 (routes from outside the EIGRP AS).
- **Internal BGP (iBGP)**: 200.
- **Unknown/Untrusted**: 255 (not installed in routing table).

**Usage**:
- AD is used to compare routes from different protocols (e.g., OSPF vs. EIGRP).
- Metrics compare routes within the same protocol; AD compares across protocols.
- Example: R1 learns 192.168.4.0/24 from OSPF (AD 110, metric 30) and EIGRP (AD 90, metric 33280). The EIGRP route is chosen due to lower AD, despite its higher metric.

**Floating Static Routes**:
- A static route with a manually set AD higher than a dynamic protocol’s AD (e.g., `ip route 192.168.4.0 255.255.255.0 10.0.12.2 100`).
- Used as a backup; only installed if the dynamic route is unavailable.
- Example: A static route with AD 100 is ignored if an OSPF route (AD 110) exists, but activates if the OSPF route is removed.

**Analogy**: AD is like a trust ranking for news sources (protocols). A trusted source (lower AD, e.g., EIGRP) is believed over a less trusted one (higher AD, e.g., RIP), even if the less trusted source has a simpler story (lower metric).

**Route Table Example**:
- Command: `show ip route`
- Output format: `[AD/metric]`
  - Example: `O 192.168.4.0/24 [110/3] via 10.0.12.2` (OSPF, AD 110, metric 3).
  - Example: `S 192.168.4.0/24 [1/0] via 10.0.12.2` (static, AD 1, metric 0).
- Connected routes (AD 0) don’t display AD in the output.

## Key Commands

- **View Routing Table**:
  ```bash
  show ip route
  ```
  - Displays routes, AD, metric, and next hop.
- **Configure Static Route**:
  ```bash
  ip route <destination> <mask> <next-hop> [distance]
  ```
  - Example: `ip route 192.168.4.0 255.255.255.0 10.0.12.2 100` (floating static route).
- **Configure Host Route**:
  ```bash
  ip route <host> 255.255.255.255 <next-hop>
  ```
  - Example: `ip route 192.168.4.1 255.255.255.255 10.0.12.2` (route to a single host).

## Quiz for Practice

1. **R1 learns four routes to 192.168.1.0/24 via RIP, EIGRP, OSPF, and IS-IS. Which route(s) will be added to the routing table?**
   - A. RIP only
   - B. EIGRP only
   - C. OSPF only
   - D. IS-IS only
   - E. RIP and EIGRP
   - F. OSPF and IS-IS
   - G. All routes
   - **Answer**: B. EIGRP (AD 90) has the lowest AD, so only its route is added.

2. **Which type of routing protocol is known as ‘routing by rumor’?**
   - A. Link state
   - B. Path vector
   - C. Distance vector
   - D. Interior gateway
   - **Answer**: C. Distance vector (RIP, EIGRP) shares routes without a full network view.

3. **R1 learns two RIP routes to 172.16.0.0/16, both 5 hops, via 10.0.0.1 and 10.1.0.1. Which route(s) will be added?**
   - A. Both routes
   - B. Only via 10.0.0.1
   - C. Only via 10.1.0.1
   - D. Neither (RIP’s AD too high)
   - **Answer**: A. Both routes (same protocol, same metric, ECMP load-balancing).

**Analogy for Quiz**: The quiz is like a test to see if you understand which friend (protocol) is most trusted (AD) and how they choose paths (metrics). ECMP is like picking two equally good paths to a party.

## Study Tips for CCNA

- **Focus on OSPF**: It’s the primary dynamic routing protocol for the exam (3.4).
- **Memorize AD Values**: Use flashcards to learn ADs (e.g., EIGRP 90, OSPF 110, RIP 120).
- **Understand Metrics**: Know RIP (hop count), EIGRP (bandwidth/delay), OSPF (cost).
- **Practice Commands**: Use Packet Tracer to practice `show ip route` and static route configurations.
- **Use Analogies**: Relate dynamic routing to GPS or gossip to make concepts intuitive.
- **Lab Practice**: Follow upcoming lab videos (Days 25–27) for RIP, EIGRP, and OSPF configurations.

These notes provide a complete foundation for understanding dynamic routing for the CCNA exam. Review the analogies, practice the commands, and use the quiz to test your knowledge. Future videos on RIP, EIGRP, and OSPF will reinforce these concepts.
