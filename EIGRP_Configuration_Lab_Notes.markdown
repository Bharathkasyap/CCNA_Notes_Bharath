# EIGRP Configuration Lab Notes for CCNA Beginners

## Lab Overview

This lab introduces **EIGRP** (Enhanced Interior Gateway Routing Protocol) configuration and key concepts like metrics, path selection, and **unequal-cost load-balancing**, which are relevant for the CCNA exam despite EIGRP not being a primary focus. The lab combines hands-on configuration with lecture-style explanations to deepen understanding of EIGRP’s functionality, particularly how it differs from RIP and OSPF. The network uses EIGRP Autonomous System (AS) 100, and the lab emphasizes loopback interfaces, passive interfaces, and EIGRP’s unique features.

**Lab Objectives**:
- Configure hostnames, IP addresses, and enable router interfaces.
- Configure loopback interfaces on each router.
- Enable EIGRP on all routers with AS 100, using appropriate `network` commands and passive interfaces.
- Disable auto-summarization to support VLSM/CIDR.
- Understand and verify EIGRP metrics, feasible distance, reported distance, successor, feasible successor, and unequal-cost load-balancing.
- Verify EIGRP neighbors and routes using show commands.

**Network Topology**:
- **Routers**: R1, R2, R3, R4 (EIGRP AS 100).
- **Links**:
  - R1–R2: GigabitEthernet (10.0.12.0/30).
  - R2–R4, R3–R4, R1–R3: FastEthernet (10.0.24.0/30, 10.0.34.0/30, 10.0.13.0/30).
  - R4’s LAN: 192.168.4.0/24.
  - Loopback interfaces: R1 (1.1.1.1/32), R2 (2.2.2.2/32), R3 (3.3.3.3/32), R4 (4.4.4.4/32).
- **Key Notes**:
  - GigabitEthernet (R1–R2) is faster than FastEthernet links, impacting EIGRP metrics.
  - All routers must use AS 100 to form EIGRP adjacencies.

**Analogy**: EIGRP is like a smart delivery company (AS 100) where trucks (routers) share delivery routes (routes) using a radio channel (multicast 224.0.0.10). The company picks the fastest routes based on road speed (bandwidth) and traffic lights (delay), and can split deliveries across multiple roads, even if they’re not equally fast (unequal-cost load-balancing). Loopback interfaces are like virtual mailboxes for testing, and passive interfaces are like muting a truck’s radio to save battery.

## Lab Steps and Configurations

### Step 1: Configure Hostnames, IP Addresses, and Enable Interfaces
**Objective**: Set up basic connectivity by configuring hostnames, IP addresses, and enabling interfaces.

**Details**:
- Pre-configured in the video for time, but in Packet Tracer, you configure:
  - **R1**: G0/0 (10.0.12.1/30 to R2), G1/0 (10.0.13.1/30 to R3).
  - **R2**: G0/0 (10.0.12.2/30 to R1), G1/0 (10.0.24.2/30 to R4).
  - **R3**: G0/0 (10.0.13.2/30 to R1), G1/0 (10.0.34.2/30 to R4).
  - **R4**: G0/0 (10.0.24.1/30 to R2), G1/0 (10.0.34.1/30 to R3), G2/0 (192.168.4.1/24 to LAN).
- Command (example for R1’s G0/0):
  ```bash
  enable
  conf t
  hostname R1
  interface GigabitEthernet0/0
   ip address 10.0.12.1 255.255.255.252
   no shutdown
  exit
  ```
- **Verification**:
  ```bash
  show ip interface brief
  ```
  - Ensures interfaces are up/up with correct IPs.

**Analogy**: Setting up IP addresses is like assigning street addresses to delivery depots (routers). Enabling interfaces is like opening the depot doors for trucks to move packages (packets).

**Lifelong Retention Tip**: Visualize a map with labeled buildings (routers) and roads (links). Write down the IP addresses (e.g., 10.0.12.1) on sticky notes and place them on a physical or mental map to associate interfaces with networks. Repeat the configuration in Packet Tracer to build muscle memory.

### Step 2: Configure Loopback Interfaces
**Objective**: Create a loopback interface on each router to simulate virtual networks and understand their role in EIGRP.

**What is a Loopback Interface?**:
- A virtual interface inside the router, not tied to physical hardware.
- Always up unless manually disabled (`shutdown`).
- Commonly uses /32 mask (single host address).
- Uses: Simulate networks, provide stable router IDs, or test connectivity in labs.

**Configuration**:
- **R1**:
  ```bash
  enable
  conf t
  interface Loopback0
   ip address 1.1.1.1 255.255.255.255
  exit
  ```
- **R2**:
  ```bash
  interface Loopback0
   ip address 2.2.2.2 255.255.255.255
  ```
- **R3**:
  ```bash
  interface Loopback0
   ip address 3.3.3.3 255.255.255.255
  ```
- **R4**:
  ```bash
  interface Loopback0
   ip address 4.4.4.4 255.255.255.255
  ```
- **Verification**:
  ```bash
  show ip interface brief
  ```
  - Shows Loopback0 as up/up with the configured IP (e.g., 1.1.1.1/32 on R1).

**Analogy**: A loopback interface is like a virtual postbox in a depot (router) used for internal testing. It’s always open (up/up) and has a unique address (e.g., 1.1.1.1) for sending test packages (packets).

**Lifelong Retention Tip**: Imagine loopbacks as “personal phone numbers” for routers (e.g., 1.1.1.1 for R1). Memorize them as “R1 = 1.1.1.1, R2 = 2.2.2.2” using a rhyme or mnemonic: “R1’s one-one-one, R2’s two-two-two, R3’s three-three-three, R4’s four-four-four.” Practice creating loopbacks in Packet Tracer and check with `show ip interface brief` to reinforce.

### Step 3: Configure EIGRP on All Routers
**Objective**: Enable EIGRP AS 100, configure `network` commands, disable auto-summarization, and set passive interfaces.

**Configuration on R4**:
```bash
enable
conf t
router eigrp 100
 network 0.0.0.0 255.255.255.255
 no auto-summary
 passive-interface GigabitEthernet0/0
 passive-interface Loopback0
exit
```
- **Explanation**:
  - `router eigrp 100`: Enters EIGRP configuration mode with AS 100 (must match on all routers).
  - `network 0.0.0.0 255.255.255.255`: Activates EIGRP on all interfaces (wildcard mask /0 matches any IP). Not recommended in production but simplifies lab setup.
  - `no auto-summary`: Prevents classful summarization (e.g., 192.168.4.0/24 stays /24, not /16).
  - `passive-interface GigabitEthernet0/0`: Stops EIGRP advertisements on G0/0 (LAN, no neighbors).
  - `passive-interface Loopback0`: Stops EIGRP messages on Loopback0 (virtual, no neighbors), saving resources.
- **Verification**:
  ```bash
  show ip protocols
  ```
  - Confirms AS 100, auto-summary disabled, networks (0.0.0.0/0), passive interfaces (G0/0, Loopback0).

**Configuration on R3**:
```bash
enable
conf t
router eigrp 100
 network 10.0.13.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 3.3.3.3 0.0.0.0
 no auto-summary
 passive-interface Loopback0
exit
```
- **Explanation**:
  - `network 10.0.13.0 0.0.0.3`: Activates EIGRP on G0/0 (/30 = 255.255.255.252, wildcard 0.0.0.3).
  - `network 10.0.34.0 0.0.0.3`: Activates EIGRP on G1/0.
  - `network 3.3.3.3 0.0.0.0`: Activates EIGRP on Loopback0 (/32 = 0.0.0.0).
  - `no auto-summary` and `passive-interface Loopback0`: Same as R4.
- **Verification**:
  ```bash
  show ip protocols
  ```
  - Shows networks, passive interface, and AS 100.

**Configuration on R2**:
```bash
enable
conf t
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.24.0 0.0.0.3
 network 2.2.2.2 0.0.0.0
 no auto-summary
 passive-interface Loopback0
exit
```
- **Explanation**: Similar to R3, using precise wildcard masks (/30, /32).

**Configuration on R1**:
```bash
enable
conf t
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 network 1.1.1.1 0.0.0.0
 no auto-summary
 passive-interface Loopback0
exit
```
- **Verification**:
  ```bash
  show ip protocols
  show ip eigrp neighbors
  show ip route eigrp
  ```
  - `show ip protocols`: Confirms AS 100, networks, passive interface, neighbors (R2, R3).
  - `show ip eigrp neighbors`: Shows R2 (10.0.12.2) and R3 (10.0.13.2) as neighbors.
  - `show ip route eigrp`: Shows EIGRP routes (marked `D`), e.g., 192.168.4.0/24, loopbacks (2.2.2.2/32, 3.3.3.3/32, 4.4.4.4/32), 10.0.24.0/30, 10.0.34.0/30.

**Analogy**: Configuring EIGRP is like setting up a delivery company’s radio network. You assign a channel (AS 100), choose which trucks (interfaces) use the radio (`network`), mute unnecessary chatter on virtual or LAN trucks (`passive-interface`), and ensure addresses aren’t oversimplified (`no auto-summary`). Neighbors are like trucks on the same channel sharing delivery routes instantly.

**Lifelong Retention Tip**: Create a mental “EIGRP radio station” with AS 100 as the channel. Chant: “Network for interfaces, passive for silence, no auto-summary for precision.” Practice in Packet Tracer, typing commands while saying their purpose aloud (e.g., “network 10.0.12.0 0.0.0.3 activates EIGRP on G0/0”). Use flashcards for wildcard masks (e.g., /30 = 0.0.0.3, /32 = 0.0.0.0).

### Step 4: Understand EIGRP Metrics and Path Selection
**Objective**: Learn how EIGRP calculates metrics, selects paths (successor, feasible successor), and performs unequal-cost load-balancing.

**EIGRP Metric**:
- **Formula**: Complex, involving bandwidth, delay, and K-values (K1=1, K2=0, K3=1, K4=0, K5=0 by default).
  - Simplified: **Metric = Bandwidth (slowest link) + Delay (sum of all links)**.
  - Bandwidth: Based on the slowest link in the path (e.g., FastEthernet 100 Mbps vs. GigabitEthernet 1000 Mbps).
  - Delay: Sum of default delay values (based on interface bandwidth, not measured via pings).
- **Example (R1 to 192.168.4.0/24)**:
  - Via R2: GigabitEthernet (R1–R2) + FastEthernet (R2–R4). Lower metric due to faster R1–R2 link.
  - Via R3: FastEthernet (R1–R3) + FastEthernet (R3–R4). Higher metric due to slower links.

**Key Terms**:
- **Feasible Distance (FD)**: This router’s total metric to the destination.
  - Example: R1’s FD to 192.168.4.0/24 via R2 = 28,672.
- **Reported Distance (RD)** (or Advertised Distance): Neighbor’s metric to the destination.
  - Example: R2’s RD to 192.168.4.0/24 = 28,416.
- **Successor**: Route with the lowest FD (best path).
  - Example: R1 to 192.168.4.0/24 via R2 (FD 28,672).
- **Feasible Successor**: Alternate route with RD < successor’s FD (loop-prevention).
  - Example: R3’s route (FD 30,976, RD 28,416) is a feasible successor because RD (28,416) < successor’s FD (28,672).
- **Feasibility Condition**: Ensures no loops by requiring RD < successor’s FD.

**Verification**:
```bash
show ip eigrp topology
```
- **Output (for 192.168.4.0/24)**:
  - Via R2: FD = 28,672, RD = 28,416 (successor, in routing table).
  - Via R3: FD = 30,976, RD = 28,416 (feasible successor, not in routing table yet).

**Analogy**: EIGRP metrics are like calculating delivery time. The slowest road (bandwidth) sets the pace, and traffic lights (delay) add up. The successor is the fastest delivery route, and the feasible successor is a backup route that’s guaranteed not to loop back (like avoiding a U-turn to the starting depot).

**Lifelong Retention Tip**: Picture a delivery truck choosing the fastest highway (successor) but keeping a slightly slower backroad (feasible successor) ready. Memorize: “FD is my cost, RD is neighbor’s cost, successor’s the best, feasible successor’s the rest.” Create a chart in Packet Tracer: list FD/RD for routes (e.g., 192.168.4.0/24) and quiz yourself.

### Step 5: Configure Unequal-Cost Load-Balancing
**Objective**: Enable EIGRP to load-balance traffic across paths with different metrics.

**Default Behavior**:
- EIGRP performs **Equal-Cost MultiPath (ECMP)** load-balancing (up to 4 paths) if FDs are equal.
- Variance = 1 (default): Only equal-cost paths are used.

**Configuration on R1**:
```bash
enable
conf t
router eigrp 100
 variance 2
exit
```
- **Explanation**:
  - `variance 2`: Allows load-balancing over feasible successors with FD up to 2x the successor’s FD.
  - Example: Successor’s FD = 28,672 (via R2). 2x28,672 = 57,344. R3’s FD (30,976) < 57,344, so it’s included.
  - Traffic is distributed proportionally (more via R2’s faster path).
- **Verification**:
  ```bash
  show ip route eigrp
  ```
  - **Output**: Shows 192.168.4.0/24 via R2 (FD 28,672) and R3 (FD 30,976), both in the routing table.
  ```bash
  show ip eigrp topology
  ```
  - Confirms both routes, with R3 now used for load-balancing.

**Key Notes**:
- Only **feasible successors** qualify for load-balancing (RD < successor’s FD).
- Variance multiplies the successor’s FD to set a threshold (e.g., 57,344).
- Non-feasible routes are never used to prevent loops.

**Analogy**: Unequal-cost load-balancing is like a delivery company splitting packages across a fast highway (R2) and a slower backroad (R3), sending more packages on the faster route. The variance is like saying, “Use any road up to twice as slow as the fastest one, but only if it’s loop-free.”

**Lifelong Retention Tip**: Imagine a pizza delivery splitting orders: “Fast highway gets 70%, backroad gets 30%.” Memorize: “Variance multiplies FD, picks feasible successors for load-balancing.” Simulate in Packet Tracer: enable `variance 2`, check `show ip route eigrp`, and trace traffic to confirm load-balancing.

## Key Takeaways

1. **Loopback Interfaces**:
   - Virtual, always up, use /32 (e.g., 1.1.1.1/32).
   - Configured with `interface Loopback0`, `ip address`.
   - Useful for router IDs or simulating networks.
2. **EIGRP Configuration**:
   - Use `router eigrp <AS>` (e.g., 100, must match).
   - `network <network> <wildcard>` activates EIGRP on matching interfaces.
   - `no auto-summary` ensures VLSM/CIDR support.
   - `passive-interface` reduces unnecessary traffic.
3. **EIGRP Metrics**:
   - Bandwidth (slowest link) + delay (sum of links).
   - FD: Router’s total metric.
   - RD: Neighbor’s metric.
4. **Path Selection**:
   - Successor: Lowest FD (best path).
   - Feasible Successor: RD < successor’s FD (loop-free backup).
5. **Unequal-Cost Load-Balancing**:
   - Enabled with `variance <number>` (e.g., 2).
   - Includes feasible successors with FD < (successor’s FD × variance).
   - Proportional traffic distribution based on metrics.
6. **Verification Commands**:
   - `show ip protocols`: AS, networks, passive interfaces, neighbors.
   - `show ip eigrp neighbors`: Lists neighbors.
   - `show ip route eigrp`: EIGRP routes (marked `D`).
   - `show ip eigrp topology`: FD, RD, successor, feasible successor.

## Common Issues and Troubleshooting

- **EIGRP Neighbors Not Forming**:
  - **Issue**: AS mismatch.
  - **Fix**: Verify AS 100 on all routers (`show ip protocols`).
- **Routes Missing**:
  - **Issue**: Auto-summarization enabled, converting to classful networks.
  - **Fix**: Ensure `no auto-summary` is configured.
- **High CPU Usage**:
  - **Issue**: EIGRP sending messages on non-neighbor interfaces.
  - **Fix**: Use `passive-interface` for LAN/loopback interfaces.
- **No Load-Balancing**:
  - **Issue**: Variance too low or route not a feasible successor.
  - **Fix**: Check `show ip eigrp topology` for RD vs. FD; increase `variance`.

**Analogy**: Troubleshooting is like fixing a delivery company’s radio system. Ensure all trucks are on the same channel (AS), routes aren’t oversimplified (no auto-summary), and unnecessary chatter is muted (passive interfaces).

## Practice Commands

- **Configure Loopback**:
  ```bash
  interface Loopback0
   ip address 1.1.1.1 255.255.255.255
  ```
- **Configure EIGRP**:
  ```bash
  router eigrp 100
   network 10.0.12.0 0.0.0.3
   no auto-summary
   passive-interface Loopback0
  ```
- **Enable Unequal-Cost Load-Balancing**:
  ```bash
  router eigrp 100
   variance 2
  ```
- **Verify**:
  ```bash
  show ip interface brief
  show ip protocols
  show ip eigrp neighbors
  show ip route eigrp
  show ip eigrp topology
  ```

## Study Tips for Lifelong Retention

1. **Use Mnemonics for Commands**:
   - Chant: “Router EIGRP 100, network with wildcard, no auto-summary, passive for silence.” Associate with a tune (e.g., “Twinkle Twinkle Little Star”) to make it stick.
2. **Visualize the Topology**:
   - Draw the network (R1–R4, Gigabit vs. FastEthernet) on paper or a whiteboard. Label IPs and loopbacks. Revisit weekly to reinforce.
3. **Memorize Key Values**:
   - AD: EIGRP 90 (internal), 170 (external).
   - Multicast: 224.0.0.10.
   - Use flashcards: “EIGRP AD? 90/170. Multicast? 224.0.0.10.”
4. **Practice in Packet Tracer**:
   - Replicate the lab: configure IPs, loopbacks, EIGRP, and variance. Check routes with `show ip route eigrp`. Repeat 3–5 times to build muscle memory.
5. **Analogy for Metrics and Load-Balancing**:
   - Picture a delivery race: successor is the fastest driver, feasible successor is a reliable backup driver who won’t circle back (loop-free). Variance is like allowing slower drivers if they’re not too slow. Tell this story to a friend to internalize it.
6. **Wildcard Masks**:
   - Practice converting: /30 = 0.0.0.3, /32 = 0.0.0.0, /24 = 0.0.0.255. Write 5 examples daily for a week.
   - Mnemonic: “Subtract from 255, wildcard you’ll address.”
7. **Teach Someone**:
   - Explain EIGRP metrics (bandwidth + delay), successor, and feasible successor to a peer or record yourself. Teaching reinforces memory.
8. **Use Spaced Repetition**:
   - Review these notes after 1 day, 1 week, 1 month using a tool like Anki. Quiz yourself on terms (FD, RD, successor, feasible successor).

These notes provide a complete foundation for the EIGRP Configuration Lab, preparing you for CCNA exam questions and building a base for OSPF. Practice the commands, use the analogies, and apply the retention tips to master EIGRP concepts for life!