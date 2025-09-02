# OSPF Part 2 Configuration Lab Notes for CCNA Beginners

## Lab Overview

This lab builds on the OSPF Part 1 lab, applying new configurations from the OSPF Part 2 lecture in the same network topology. It focuses on enabling OSPF directly on interfaces, configuring passive interfaces (including using `passive-interface default`), adjusting the OSPF reference bandwidth, and advertising a default route. The lab also explores OSPF neighbor states and Hello packet details in Packet Tracer’s simulation mode, reinforcing CCNA exam topic 3.4 (Configure and verify single-area OSPFv2).

**Lab Objectives**:
- Configure hostnames and IP addresses (pre-configured, steps 1 and 2).
- Enable OSPF directly on interfaces (not using `network` commands).
- Configure passive interfaces, including using `passive-interface default` on R4.
- Set the OSPF reference bandwidth to achieve a FastEthernet cost of 100.
- Advertise a default route from R1 to the OSPF domain.
- Verify OSPF configurations, neighbor states, routing tables, and Hello packet fields.
- Understand Packet Tracer limitations (e.g., missing `show ip ospf interface brief`, E2 route behavior).

**Network Topology**:
- **Routers**: R1, R2, R3, R4 (all in OSPF area 0).
- **Links**:
  - R1–R2: GigabitEthernet (10.0.12.0/30, R1: 10.0.12.1, R2: 10.0.12.2).
  - R1–R3: FastEthernet (10.0.13.0/30, R1: 10.0.13.1, R3: 10.0.13.2).
  - R2–R4: FastEthernet (10.0.24.0/30, R4: 10.0.24.1, R2: 10.0.24.2).
  - R3–R4: FastEthernet (10.0.34.0/30, R4: 10.0.34.1, R3: 10.0.34.2).
  - R1–ISP: GigabitEthernet (203.0.113.0/30, R1: 203.0.113.1, ISPR1: 203.0.113.2).
  - R4–LAN: GigabitEthernet (192.168.4.0/24, R4: 192.168.4.1).
- **Loopback Interfaces**:
  - R1: 1.1.1.1/32.
  - R2: 2.2.2.2/32.
  - R3: 3.3.3.3/32.
  - R4: 4.4.4.4/32.
- **Key Notes**:
  - GigabitEthernet (1000 Mbps) is faster than FastEthernet (100 Mbps), affecting OSPF cost.
  - All routers in area 0 (single-area OSPF).
  - R1 connects to an ISP, enabling default route advertisement.
  - Packet Tracer limitations: Missing `show ip ospf interface brief` and incorrect E2 route behavior (CCNP topic).

**Analogy**: OSPF is like a city’s GPS network where control centers (routers) share a map (LSDB) via traffic reports (LSAs). This lab fine-tunes the GPS by installing it directly at intersections (interfaces), muting unnecessary signals (passive interfaces), recalibrating route times (reference bandwidth), and adding a highway exit sign (default route).

**Lifelong Retention Tip**: Visualize the network as a city with R1–R4 as control centers. Draw the topology, label IPs, loopbacks, and costs (e.g., FastEthernet = 100). Chant: “OSPF on interfaces, passive by default, reference for cost, default for exit.” Practice in Packet Tracer to build muscle memory. Create flashcards for commands (`ip ospf 1 area 0`, `auto-cost reference-bandwidth 10000`).

## Lab Steps and Configurations

### Step 1 & 2: Configure Hostnames and IP Addresses
**Objective**: Set up basic connectivity by configuring hostnames, IP addresses, and enabling interfaces.

**Details**:
- Pre-configured in the video to save time, but in Packet Tracer, you configure:
  - **R1**:
    - G0/0: 10.0.12.1/30 (to R2).
    - F1/0: 10.0.13.1/30 (to R3).
    - G3/0: 203.0.113.1/30 (to ISPR1).
    - Loopback0: 1.1.1.1/32.
  - **R2**:
    - G0/0: 10.0.12.2/30 (to R1).
    - F1/0: 10.0.24.2/30 (to R4).
    - Loopback0: 2.2.2.2/32.
  - **R3**:
    - F1/0: 10.0.13.2/30 (to R1).
    - F2/0: 10.0.34.2/30 (to R4).
    - Loopback0: 3.3.3.3/32.
  - **R4**:
    - F1/0: 10.0.24.1/30 (to R2).
    - F2/0: 10.0.34.1/30 (to R3).
    - G0/0: 192.168.4.1/24 (LAN).
    - Loopback0: 4.4.4.4/32.
- **Command (example for R1’s G0/0)**:
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

**Analogy**: Assigning IP addresses is like giving each control center a unique street address. Enabling interfaces is like opening the center’s gates for traffic (packets).

**Lifelong Retention Tip**: Draw the topology with IPs (e.g., 10.0.12.1 for R1’s G0/0) and memorize using a rhyme: “R1 to R2, 10.0.12, R1 to R3, 10.0.13.” Configure in Packet Tracer and verify with `show ip interface brief`. Repeat 3 times to reinforce.

### Step 3: Enable OSPF Directly on Interfaces
**Objective**: Activate OSPF on specific interfaces using `ip ospf` instead of `network` commands, and configure passive interfaces.

**Configuration on R1**:
```bash
enable
conf t
interface range GigabitEthernet0/0, FastEthernet1/0, Loopback0
 ip ospf 1 area 0
router ospf 1
 passive-interface Loopback0
exit
```
- **Explanation**:
  - `interface range GigabitEthernet0/0, FastEthernet1/0, Loopback0`: Selects multiple interfaces (comma-separated for different types).
  - `ip ospf 1 area 0`: Enables OSPF process ID 1 in area 0 on G0/0 (to R2), F1/0 (to R3), and Loopback0.
  - **Why Not G3/0?**: The 203.0.113.0/30 link to ISPR1 is not advertised; only the default route is needed.
  - `passive-interface Loopback0`: Stops Hello packets on Loopback0 (no neighbors, saves resources).
- **Verification**:
  ```bash
  show ip protocols
  ```
  - Expected: Lists OSPF process ID 1, area 0, passive interface Loopback0, and interfaces G0/0, F1/0, Loopback0 under “Routing on Interfaces Configured Explicitly” (not shown in Packet Tracer due to limitations).
  - **Packet Tracer Note**: Shows “Routing for Networks” instead, a known limitation.

**Configuration on R2**:
```bash
enable
conf t
interface range GigabitEthernet0/0, FastEthernet1/0, Loopback0
 ip ospf 1 area 0
router ospf 1
 passive-interface Loopback0
exit
```
- **Explanation**:
  - Enables OSPF on G0/0 (to R1), F1/0 (to R4), and Loopback0.
  - Sets Loopback0 as passive.

**Configuration on R3**:
```bash
enable
conf t
interface range FastEthernet1/0, FastEthernet2/0, Loopback0
 ip ospf 1 area 0
router ospf 1
 passive-interface Loopback0
exit
```
- **Explanation**:
  - Enables OSPF on F1/0 (to R1), F2/0 (to R4), and Loopback0.
  - Sets Loopback0 as passive.

**Configuration on R4**:
```bash
enable
conf t
interface range GigabitEthernet0/0, FastEthernet1/0, FastEthernet2/0, Loopback0
 ip ospf 1 area 0
router ospf 1
 passive-interface default
 no passive-interface FastEthernet1/0
 no passive-interface FastEthernet2/0
exit
```
- **Explanation**:
  - Enables OSPF on G0/0 (LAN), F1/0 (to R2), F2/0 (to R3), and Loopback0.
  - `passive-interface default`: Sets all interfaces as passive (no Hellos), causing neighbor adjacencies to go Down.
  - `no passive-interface FastEthernet1/0` and `no passive-interface FastEthernet2/0`: Re-enables Hellos on F1/0 and F2/0 (to form neighbors with R2 and R3).
  - G0/0 and Loopback0 remain passive, advertising 192.168.4.0/24 and 4.4.4.4/32 without Hellos.
- **Verification**:
  ```bash
  show ip protocols
  ```
  - Confirms passive interfaces (G0/0, Loopback0) and active interfaces (F1/0, F2/0).
  ```bash
  show ip ospf neighbor
  ```
  - Shows R2 (2.2.2.2 via 10.0.24.2) and R3 (3.3.3.3 via 10.0.34.2) in Full state.
  - Dead timer counts down from 40 to ~30, resets on Hello receipt (every 10 seconds).

**Comparison to OSPF Part 1 Lab**:
- **Part 1**: Used `network` commands with wildcard masks (e.g., `network 0.0.0.0 255.255.255.255 area 0` on R4) and individual `passive-interface` commands.
- **Part 2**: Uses `ip ospf <process-id> area 0` directly on interfaces and `passive-interface default` on R4.
- **Similarity**: Both enable OSPF in area 0, set Loopback0 as passive, and avoid OSPF on R1’s G3/0.
- **Difference**: Interface-based activation is more precise; `passive-interface default` is faster for multiple passive interfaces.

**Comparison to EIGRP**:
- **EIGRP**: Requires `network` commands in `router eigrp <AS>` mode; no interface-based activation.
- **OSPF**: Offers both `network` and `ip ospf` methods, with `ip ospf` being more granular.
- **Similarity**: Both support `passive-interface` and `passive-interface default`.

**Analogy**: Enabling OSPF on interfaces is like installing GPS directly at each intersection (interface) instead of centrally mapping roads (`network`). Setting `passive-interface default` is like muting all radios and turning on only those needed for neighbor communication.

**Lifelong Retention Tip**: In Packet Tracer, configure OSPF with `ip ospf 1 area 0` on R1–R3 and `passive-interface default` on R4. Verify with `show ip protocols` and `show ip ospf neighbor`. Memorize: “ip ospf for direct, passive default for silence.” Create flashcards for commands (`ip ospf 1 area 0`, `passive-interface default`). Practice 3 times to reinforce.

### Step 4: Configure Reference Bandwidth
**Objective**: Set the OSPF reference bandwidth so FastEthernet interfaces have a cost of 100.

**Configuration on All Routers**:
```bash
enable
conf t
router ospf 1
 auto-cost reference-bandwidth 10000
exit
```
- **Explanation**:
  - **Formula**: Cost = Reference Bandwidth ÷ Interface Bandwidth (in Mbps).
  - FastEthernet: 100 Mbps.
  - Desired cost: 100.
  - Calculation: Reference Bandwidth ÷ 100 = 100 → Reference Bandwidth = 10,000 Mbps.
  - `auto-cost reference-bandwidth 10000`: Sets reference bandwidth to 10,000 Mbps (entered in Mbps).
  - **Costs**:
    - FastEthernet (100 Mbps): 10,000 ÷ 100 = 100.
    - GigabitEthernet (1000 Mbps): 10,000 ÷ 1000 = 10.
    - 10GigEthernet (10,000 Mbps): 10,000 ÷ 10,000 = 1.
    - Loopback: Fixed cost of 1.
  - **Best Practice**: Configure the same reference bandwidth on all routers for consistent costs.
- **Verification**:
  ```bash
  show ip ospf interface FastEthernet1/0
  ```
  - Confirms FastEthernet cost = 100.
  ```bash
  show ip ospf interface GigabitEthernet0/0
  ```
  - Confirms GigabitEthernet cost = 10.
  - **Packet Tracer Note**: `show ip ospf interface brief` (shows all interface costs) is unavailable; use `show ip ospf interface <interface>`.

**Comparison to OSPF Part 1 Lab**:
- **Part 1**: Used default reference bandwidth (100 Mbps), causing FastEthernet and GigabitEthernet to have cost 1, leading to unexpected ECMP on R4.
- **Part 2**: Sets reference bandwidth to 10,000 Mbps, differentiating FastEthernet (100) and GigabitEthernet (10).
- **Similarity**: Both adjust OSPF behavior to influence routing decisions.

**Comparison to EIGRP**:
- **EIGRP**: Uses bandwidth and delay for metric, adjustable via `bandwidth` or `delay` commands.
- **OSPF**: Uses cost (based on bandwidth), adjustable via `auto-cost reference-bandwidth`, `ip ospf cost`, or `bandwidth` (not recommended).
- **Similarity**: Both rely on interface bandwidth for metric calculations.

**Analogy**: Adjusting the reference bandwidth is like recalibrating a GPS to measure travel times accurately, making expressways (GigabitEthernet) faster (cost 10) than regular roads (FastEthernet, cost 100).

**Lifelong Retention Tip**: In Packet Tracer, set `auto-cost reference-bandwidth 10000` and verify costs (FastEthernet = 100, GigabitEthernet = 10) with `show ip ospf interface`. Memorize: “10,000 ÷ 100 = 100 for FastEthernet.” Practice calculating costs for 5 interfaces daily (e.g., 10,000 ÷ 1000 = 10 for GigabitEthernet). Create flashcard: “FastEthernet cost with 10,000 Mbps? 100.”

### Step 5: Advertise Default Route on R1
**Objective**: Configure R1 to advertise a default route (0.0.0.0/0) to the OSPF domain, making R1 an ASBR.

**Configuration**:
```bash
enable
conf t
router ospf 1
 default-information originate
exit
ip route 0.0.0.0 0.0.0.0 203.0.113.2
exit
```
- **Explanation**:
  - `ip route 0.0.0.0 0.0.0.0 203.0.113.2`: Creates a static default route to ISPR1 (next hop 203.0.113.2).
  - `default-information originate`: Advertises the default route as a Type-5 AS External LSA (E2 type by default), making R1 an **Autonomous System Boundary Router (ASBR)**.
- **Verification on R4**:
  ```bash
  show ip route
  ```
  - Expected: Default route (`O*E2 0.0.0.0/0 [110/1]`) via R2 (10.0.24.2) and R3 (10.0.34.2).
  - **Packet Tracer Issue**: Only one route (via R2) is shown with cost 1, ignoring internal OSPF costs due to E2 metric type (CCNP topic).
  - **Correct Behavior (e.g., GNS3, real equipment)**: Both routes (via R2 and R3) should appear with cost 1, as E2 routes use a fixed cost (default 1) and ignore internal OSPF costs (e.g., FastEthernet vs. GigabitEthernet).
  - **Test**: Shut down R4’s F1/0 to drop R2 adjacency:
    ```bash
    interface FastEthernet1/0
     shutdown
    ```
    - Verify: Route via R3 (10.0.34.1) appears with cost 1.
    - Confirms E2 routes have equal cost (1), enabling ECMP.
  - **Note**: E2 metric is a CCNP topic; for CCNA, focus on default route configuration and verification.

**Comparison to OSPF Part 1 Lab**:
- **Identical**: Both configure `ip route 0.0.0.0 0.0.0.0 203.0.113.2` and `default-information originate`.
- **Difference**: Part 1 used `network` commands; Part 2 uses `ip ospf`. Part 2’s reference bandwidth change (10,000 Mbps) should prefer R2’s path, but E2 routes cause equal-cost routes (Packet Tracer bug).

**Comparison to EIGRP**:
- **EIGRP**: Uses `default-information originate` to advertise default route, but metric includes bandwidth/delay, potentially preferring R2’s path.
- **OSPF**: E2 routes use fixed cost (1), ignoring internal costs, leading to ECMP via R2 and R3.
- **Similarity**: Both require a static default route and `default-information originate`.

**Analogy**: Advertising a default route is like posting a sign at R1: “To anywhere outside the city, take this highway (203.0.113.2).” E2 routes make both roads to R1 (via R2 or R3) equally appealing (cost 1), like two identical highway exits.

**Lifelong Retention Tip**: In Packet Tracer, configure the default route and verify with `show ip route` on R4. Test shutting down F1/0 to confirm ECMP. Memorize: “Default route needs ip route and default-information originate.” Create flashcard: “E2 route cost? 1.” Practice 3 times to reinforce.

### Step 6: Examine OSPF Hello Packet in Simulation Mode
**Objective**: Analyze an OSPF Hello packet in Packet Tracer’s simulation mode to understand its fields.

**Steps**:
- Enter Packet Tracer’s simulation mode.
- Capture an OSPF Hello packet (e.g., from R4’s F1/0 to R2).
- Check **PDU Details**:
  - **Layer 3 Destination**: 224.0.0.5 (multicast for all OSPF routers).
  - **Ethernet Frame**: Encapsulates IP packet.
  - **IP Packet**:
    - Protocol field: 0x59 (hex) = 89 (decimal), indicating OSPF.
  - **OSPF Hello Packet**:
    - Version: 2 (OSPFv2 for IPv4).
    - Type: 1 (Hello message).
    - Router ID: Sender’s ID (e.g., 4.4.4.4 for R4).
    - Area ID: 0.0.0.0 (area 0 in dotted decimal).
    - Network Mask: Interface’s subnet mask (e.g., 255.255.255.252 for F1/0).
    - Hello Timer: 10 seconds.
    - Dead Timer: 40 seconds.
    - DR/BDR Addresses: Designated Router and Backup Designated Router (covered in Day 28).
    - Neighbor Address: Neighbor’s router ID (e.g., 2.2.2.2 for R2).

**Comparison to OSPF Part 1 Lab**:
- **Part 1**: Focused on basic configurations and routing table verification.
- **Part 2**: Adds packet analysis to understand OSPF’s operation (Hello messages).

**Comparison to EIGRP**:
- **EIGRP**: Uses 224.0.0.10 for Hellos, protocol 88, simpler neighbor discovery.
- **OSPF**: Uses 224.0.0.5, protocol 89, structured states (Down to Full), and five message types (Hello, DBD, LSR, LSU, LSAck).
- **Similarity**: Both use multicast Hellos to discover neighbors.

**Analogy**: A Hello packet is like a traffic center’s introduction letter sent to all nearby centers (224.0.0.5), saying, “I’m R4, in area 0, let’s talk every 10 seconds or I’ll assume you’re offline in 40.”

**Lifelong Retention Tip**: In Packet Tracer’s simulation mode, capture a Hello packet and note fields (224.0.0.5, protocol 89, Type 1, timers 10/40). Memorize: “Hello to 224.0.0.5, protocol 89, Type 1.” Create flashcard: “OSPF Hello multicast? 224.0.0.5.” Practice capturing packets 3 times to internalize.

## Key Takeaways

1. **OSPF Interface Configuration**:
   - Use `ip ospf <process-id> area <area>` to enable OSPF directly on interfaces (e.g., `ip ospf 1 area 0`).
   - More precise than `network` commands; avoids enabling OSPF on unintended interfaces (e.g., R1’s G3/0).
2. **Passive Interfaces**:
   - `passive-interface Loopback0`: Stops Hellos on Loopback0 (all routers).
   - `passive-interface default` with `no passive-interface <interface>`: Efficient for multiple passive interfaces (R4’s G0/0, Loopback0).
3. **Reference Bandwidth**:
   - Set with `auto-cost reference-bandwidth 10000` (Mbps) for FastEthernet cost = 100 (10,000 ÷ 100).
   - GigabitEthernet cost = 10 (10,000 ÷ 1000).
   - Configure identically on all routers for consistency.
4. **Default Route**:
   - Configure with `ip route 0.0.0.0 0.0.0.0 203.0.113.2` and `default-information originate`.
   - Appears as `O*E2 0.0.0.0/0 [110/1]` on R4, ideally via both R2 and R3 (ECMP, cost 1 due to E2 metric).
   - **Packet Tracer Bug**: Shows only one route (via R2); correct behavior includes both (test by shutting down F1/0).
5. **Hello Packet**:
   - Multicast to 224.0.0.5, protocol 89, Type 1, includes Router ID, Area ID (0.0.0.0), timers (10/40).
6. **Packet Tracer Limitations**:
   - `show ip ospf interface brief` unavailable; use `show ip ospf interface <interface>`.
   - E2 routes incorrectly show one path (cost 1) instead of ECMP; correct in GNS3/real equipment.
7. **Verification Commands**:
   - `show ip protocols`: OSPF settings, passive interfaces, explicit interfaces.
   - `show ip ospf neighbor`: Neighbor states (Full), Dead timer (40, resets every 10 seconds).
   - `show ip ospf interface <interface>`: Interface cost, timers.
   - `show ip route`: Default route (`O*E2`), costs.

## Common Issues and Troubleshooting

- **Neighbors Not Forming**:
  - **Issue**: Mismatched area IDs, timers, or passive interfaces.
  - **Fix**: Verify area 0 (`show ip ospf interface`), timers (10/40), and ensure non-passive interfaces (e.g., R4’s F1/0, F2/0) with `show ip protocols`.
- **Incorrect Costs**:
  - **Issue**: Different reference bandwidths across routers.
  - **Fix**: Confirm `auto-cost reference-bandwidth 10000` on all routers (`show ip ospf interface`).
- **Default Route Missing**:
  - **Issue**: Missing `ip route` or `default-information originate`.
  - **Fix**: Verify static route (`show ip route`) and OSPF configuration (`show ip protocols`).
- **ECMP Not Working (Packet Tracer)**:
  - **Issue**: Only one default route shown on R4.
  - **Fix**: Recognize Packet Tracer bug; test by shutting down F1/0 to confirm R3 path. For CCNA, focus on configuration, not E2 behavior.
- **Hello Packets Not Seen**:
  - **Issue**: Interface passive or OSPF not enabled.
  - **Fix**: Check `show ip ospf interface` for OSPF status and passive settings.

**Analogy**: Troubleshooting is like fixing a city’s GPS system. Ensure centers use the same map (area 0), have correct clocks (timers), send highway signs (default route), and mute silent roads (passive interfaces).

**Lifelong Retention Tip**: In Packet Tracer, simulate errors (e.g., set R4’s F1/0 to area 1, omit `no passive-interface F1/0`) and diagnose with `show ip ospf neighbor` and `show ip route`. Create a troubleshooting checklist: “Area match? Timers? Passive set? Costs?” Practice 3 times to master.

## Practice Commands

- **Enable OSPF on Interfaces**:
  ```bash
  interface range GigabitEthernet0/0, FastEthernet1/0, Loopback0
   ip ospf 1 area 0
  ```
- **Configure Passive Interfaces**:
  ```bash
  router ospf 1
   passive-interface Loopback0
  ```
  ```bash
  router ospf 1
   passive-interface default
   no passive-interface FastEthernet1/0
  ```
- **Set Reference Bandwidth**:
  ```bash
  router ospf 1
   auto-cost reference-bandwidth 10000
  ```
- **Advertise Default Route**:
  ```bash
  ip route 0.0.0.0 0.0.0.0 203.0.113.2
  router ospf 1
   default-information originate
  ```
- **Verify**:
  ```bash
  show ip protocols
  show ip ospf neighbor
  show ip ospf interface FastEthernet1/0
  show ip route
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics for Commands**:
   - Chant: “ip ospf for direct, passive default for silence, auto-cost for 100, default for exit.” Associate with a tune (e.g., “Twinkle Twinkle”) to make it stick.
2. **Visualize the Topology**:
   - Draw the network (R1–R4, area 0) with IPs, loopbacks, and costs (FastEthernet = 100, GigabitEthernet = 10). Label R1 as ASBR. Revisit weekly.
3. **Memorize Key Values**:
   - OSPF multicast: 224.0.0.5.
   - Protocol: 89.
   - Hello/Dead timers: 10/40 (Ethernet).
   - FastEthernet cost: 10,000 ÷ 100 = 100.
   - Use flashcards: “OSPF multicast? 224.0.0.5. Protocol? 89.”
4. **Practice in Packet Tracer**:
   - Replicate the lab: configure `ip ospf`, `passive-interface default`, reference bandwidth, default route. Verify with `show ip route` and capture Hello packets. Repeat 3–5 times.
   - Test ECMP by shutting down R4’s F1/0 and checking R3’s route.
5. **Compare OSPF Part 1 and 2**:
   - Create a table: Part 1 (`network`, individual passive), Part 2 (`ip ospf`, `passive-interface default`). Quiz yourself: “Part 2 OSPF activation? ip ospf.”
6. **Packet Analysis**:
   - In simulation mode, capture 5 Hello packets and note fields (224.0.0.5, Type 1, timers). Memorize: “Hello is Type 1, protocol 89.”
7. **Troubleshooting Practice**:
   - Break adjacencies (e.g., set different area, make F1/0 passive) and fix using verification commands. Create a checklist: “Neighbors Full? Costs correct? Routes present?”
8. **Teach Someone**:
   - Explain `ip ospf`, reference bandwidth, and E2 routes to a peer or record yourself. Teaching reinforces memory.
9. **Spaced Repetition**:
   - Use Anki to review commands, packet fields, and costs after 1 day, 1 week, 1 month.

These notes provide a complete foundation for the OSPF Part 2 Configuration Lab, preparing you for CCNA exam topic 3.4 and future OSPF labs (e.g., Day 28 on DR/BDR). Practice the configurations, use the analogies, and apply the retention tips to master OSPF for life!