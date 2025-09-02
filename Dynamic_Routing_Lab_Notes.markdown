# Dynamic Routing Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring **floating static routes** to provide backup paths in a network using **OSPF** as the primary dynamic routing protocol. Floating static routes are static routes with a higher **administrative distance (AD)** than dynamic routes, ensuring they are only used when the dynamic route is unavailable (e.g., due to a link failure). The lab demonstrates how to configure and verify these routes to ensure connectivity between two LANs when the primary link fails.

**Lab Objectives**:
- Verify the initial routing tables on routers R1 and R2.
- Configure floating static routes on R1 and R2 to use ISP A as a backup path for LAN connectivity.
- Simulate a link failure to test the floating static routes.
- Verify the backup path using ping and traceroute commands.
- Understand the role of OSPF and administrative distance in route selection.

**Network Topology**:
- **Enterprise A**:
  - **R1**: Connected to LAN 10.0.1.0/24 (PC1) and R2 via fiber optic (10.0.0.0/30). Also connects to ISP A (SPR1, 203.0.113.0/30) and ISP B (ISPBR1, 203.0.113.8/30).
  - **R2**: Connected to LAN 10.0.2.0/24 (SRV1) and R1. Also connects to ISP A (SPR2, 203.0.113.4/30) and ISP B (ISPBR2, 203.0.113.12/30).
- **ISPs**:
  - **ISP A**: SPR1 and SPR2, connected via 192.168.1.0/30.
  - **ISP B**: ISPBR1 and ISPBR2, with loopback interfaces (e.g., 1.1.1.1 on ISPBR1 simulating a remote server).
- **Routing Protocol**: OSPF is used within Enterprise A as the Interior Gateway Protocol (IGP).

**Analogy**: Think of the network as a city with two neighborhoods (LANs) connected by a main road (R1–R2 fiber link). OSPF is like a GPS that prefers this road. Floating static routes are like handwritten backup directions to go through a side street (ISP A) if the main road is closed, only used when the GPS route fails.

## Lab Steps and Configurations

### Step 1: Verify Initial Routing Tables

**Objective**: Check the routing tables on R1 and R2 to understand the current routes, including OSPF-learned routes and static default routes.

**On R1**:
```bash
enable
show ip route
```
- **Output**:
  - **Connected and Local Routes**: For interfaces (e.g., 10.0.0.1/30, 10.0.1.254/24, 203.0.113.9/30).
  - **Static Default Route**: `S* 0.0.0.0/0 [1/0] via 203.0.113.9` (to ISPBR1).
  - **OSPF Route**: `O 10.0.2.0/24 [110/2] via 10.0.0.2` (R2’s LAN, learned via OSPF).
- **Explanation**:
  - **Connected Routes** (AD 0): Automatically added for directly connected interfaces.
  - **Static Default Route** (AD 1): Points to ISP B (203.0.113.9) for Internet traffic.
  - **OSPF Route** (AD 110): Learned from R2 for 10.0.2.0/24 (SRV1’s LAN).
  - Traffic from PC1 to SRV1 (10.0.2.1) uses the OSPF route via R2.
  - Traffic to 1.1.1.1 (remote server) uses the default route to ISPBR1.

**On R2**:
```bash
enable
show ip route
```
- **Output**:
  - **Connected and Local Routes**: For interfaces (e.g., 10.0.0.2/30, 10.0.2.254/24, 203.0.113.13/30).
  - **Static Default Route**: `S* 0.0.0.0/0 [1/0] via 203.0.113.13` (to ISPBR2).
  - **OSPF Route**: `O 10.0.1.0/24 [110/2] via 10.0.0.1` (R1’s LAN, learned via OSPF).
- **Explanation**:
  - Similar to R1, with an OSPF route to R1’s LAN (10.0.1.0/24) via the direct link (10.0.0.1).

**Test Connectivity from PC1**:
- **Ping SRV1 (10.0.2.1)**:
  ```bash
  ping 10.0.2.1
  ```
  - Path (in Packet Tracer simulation mode): PC1 → R1 → R2 → SRV1 → R2 → R1 → PC1.
  - Uses OSPF route (10.0.2.0/24 via R2).
- **Ping Remote Server (1.1.1.1)**:
  ```bash
  ping 1.1.1.1
  ```
  - Path: PC1 → R1 → ISPBR1 → R1 → PC1.
  - Uses default route to ISPBR1.
  - Note: 1.1.1.1 is a **loopback interface** on ISPBR1, a virtual interface used to simulate a remote destination (explained later in the course).

**Analogy**: Checking the routing table is like looking at a driver’s GPS (router) to see which roads (routes) it knows. The OSPF route is the main road to the other LAN, and the default route is a catch-all for unknown destinations (Internet).

### Step 2: Configure Floating Static Routes

**Objective**: Configure floating static routes on R1 and R2 to use ISP A (SPR1, SPR2) as a backup path for LAN-to-LAN connectivity if the direct R1–R2 link fails.

**Why Floating Static Routes?**:
- Static routes have a default AD of 1, making them preferred over OSPF (AD 110).
- A floating static route has a higher AD (e.g., 111), making it less preferred than OSPF but available as a backup if the OSPF route is removed (e.g., due to a link failure).

**Configuration on R1**:
```bash
enable
conf t
ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
exit
```
- **Explanation**:
  - `ip route 10.0.2.0 255.255.255.0 203.0.113.1`: Configures a static route to R2’s LAN (10.0.2.0/24) via SPR1 (ISP A, 203.0.113.1).
  - `111`: Sets AD to 111, higher than OSPF’s 110, making it a backup to the OSPF route.
- **Verification**:
  ```bash
  do show ip route
  ```
  - **Output**: The static route is not in the routing table because the OSPF route (AD 110) is preferred.

**Configuration on R2**:
```bash
enable
conf t
ip route 10.0.1.0 255.255.255.0 203.0.113.5 111
exit
```
- **Explanation**:
  - Configures a static route to R1’s LAN (10.0.1.0/24) via SPR2 (ISP A, 203.0.113.5).
  - AD 111 ensures it’s a backup to the OSPF route.
- **Verification**:
  ```bash
  do show ip route
  ```
  - **Output**: The static route is not in the routing table (OSPF route preferred).

**Analogy**: A floating static route is like a backup map in your glove compartment. You use the GPS (OSPF) by default, but if the GPS fails (link down), you pull out the backup map (static route) to navigate via a side road (ISP A).

### Step 3: Simulate Link Failure and Verify Floating Static Routes

**Objective**: Shut down the direct link between R1 and R2 to test if the floating static routes activate.

**Simulate Failure on R2**:
```bash
enable
conf t
interface GigabitEthernet0/2/0
 shutdown
exit
```
- **Explanation**: Disables the R1–R2 link (10.0.0.0/30), making the OSPF route to 10.0.1.0/24 (on R2) and 10.0.2.0/24 (on R1) invalid.

**Verify on R2**:
```bash
do show ip route
```
- **Output**: The OSPF route to 10.0.1.0/24 is gone, and the floating static route appears: `S 10.0.1.0/24 [111/0] via 203.0.113.5`.

**Verify on R1**:
```bash
do show ip route
```
- **Output**: The OSPF route to 10.0.2.0/24 is gone, and the floating static route appears: `S 10.0.2.0/24 [111/0] via 203.0.113.1`.

**Test Connectivity from PC1**:
- **Ping SRV1 (10.0.2.1)**:
  ```bash
  ping 10.0.2.1
  ```
  - Initial ping in real-time mode to allow ARP resolution.
  - In Packet Tracer simulation mode, path: PC1 → R1 → SPR1 → SPR2 → R2 → SRV1 → R2 → SPR2 → SPR1 → R1 → PC1.
  - **Result**: Pings succeed, confirming the floating static route works via ISP A.
- **Traceroute to SRV1**:
  ```bash
  traceroute 10.0.2.1
  ```
  - **Output**:
    1. 10.0.1.254 (R1’s G0/1)
    2. 203.0.113.1 (SPR1’s G0/0/0)
    3. 192.168.1.2 (SPR2’s G0/1/0)
    4. 203.0.113.6 (R2’s G0/0/0)
    5. 10.0.2.1 (SRV1)
  - **Explanation**: Traceroute shows each Layer 3 hop, confirming the backup path via ISP A.

**Analogy**: Shutting down the link is like closing the main road (R1–R2). The backup map (floating static route) kicks in, guiding traffic through the side street (ISP A). Traceroute is like a travel log, listing every stop (hop) along the way.

### Step 4: Additional Notes on Loopback Interfaces

**What is a Loopback Interface?**:
- A virtual interface on a router, not tied to physical hardware (like an SVI on a switch).
- Used in the lab to simulate a remote server (1.1.1.1 on ISPBR1’s Loopback0).
- **Command to Verify**:
  ```bash
  show ip interface brief
  ```
  - **Output on ISPBR1**: Shows Loopback0 (1.1.1.1) and Loopback1 (unused).
- **Uses**:
  - Simulate remote destinations in labs.
  - Provide stable IP addresses for routing protocols or management (covered later).

**Analogy**: A loopback interface is like a virtual mailbox on a router, used to represent a destination (e.g., a server) without needing a physical device.

## Key Takeaways

1. **Floating Static Routes**:
   - Configured like regular static routes but with a higher AD (e.g., 111 vs. OSPF’s 110).
   - Act as backups, only installed when dynamic routes (e.g., OSPF) are unavailable.
   - Example: `ip route 10.0.2.0 255.255.255.0 203.0.113.1 111`.
2. **OSPF in the Lab**:
   - Primary IGP for Enterprise A, advertising LAN routes (10.0.1.0/24, 10.0.2.0/24).
   - AD 110, preferred over floating static routes until the link fails.
3. **Administrative Distance (AD)**:
   - Determines route preference across different sources.
   - Connected (0), Static (1), OSPF (110), Floating Static (e.g., 111).
4. **Verification Tools**:
   - `show ip route`: Displays routing table, AD, and next hop.
   - `ping`: Tests connectivity.
   - `traceroute`: Shows the Layer 3 path, useful for troubleshooting.
5. **Loopback Interfaces**: Virtual interfaces used to simulate remote destinations in labs.

## Common Issues and Troubleshooting

- **Floating Static Route Not in Table**:
  - **Issue**: The route doesn’t appear if the OSPF route (lower AD) is active.
  - **Fix**: Expected behavior; verify with `show ip route` after disabling the primary link.
- **Ping Failures**:
  - **Issue**: Initial timeouts due to ARP resolution.
  - **Fix**: Ping in real-time mode first, then retry in simulation mode.
- **Incorrect Path**:
  - **Issue**: Traffic not using the backup path after link failure.
  - **Fix**: Verify floating static route AD is higher than OSPF’s (110) but lower than unusable routes (255).
- **Traceroute Unexpected Path**:
  - **Issue**: Traffic takes the wrong path.
  - **Fix**: Use `traceroute` to confirm hops; check routing tables and static route configurations.

**Analogy**: Troubleshooting is like checking why a delivery truck (traffic) isn’t using the backup route. Ensure the backup map (floating static route) is ready and the GPS (OSPF) is disabled when testing.

## Practice Commands

- **View Routing Table**:
  ```bash
  show ip route
  ```
- **Configure Floating Static Route**:
  ```bash
  ip route <destination> <mask> <next-hop> <AD>
  ```
  - Example: `ip route 10.0.2.0 255.255.255.0 203.0.113.1 111`
- **Simulate Link Failure**:
  ```bash
  interface <interface>
   shutdown
  ```
- **Test Connectivity**:
  ```bash
  ping <destination>
  traceroute <destination>
  ```
- **View Loopback Interfaces**:
  ```bash
  show ip interface brief
  ```

## Study Tips for CCNA

- **Lab Practice**: Replicate this lab in Packet Tracer to practice floating static routes and traceroute.
- **Memorize AD Values**: Use flashcards for key ADs: Connected (0), Static (1), OSPF (110).
- **Understand OSPF Role**: Recognize OSPF as the primary IGP and its interaction with static routes.
- **Master Traceroute**: Practice `traceroute` to understand paths and troubleshoot routing issues.
- **Use Analogies**: Relate floating static routes to backup maps and OSPF to GPS for intuitive understanding.

These notes cover all aspects of the Dynamic Routing Lab, focusing on floating static routes and their interaction with OSPF. Practice the commands, review the analogies, and use the lab to solidify your CCNA preparation!