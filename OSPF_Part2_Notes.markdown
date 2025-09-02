# OSPF Part 2 Notes for CCNA Beginners

## Introduction

This lecture builds on OSPF Part 1, diving deeper into critical OSPF topics for the CCNA exam (Topic 3.4: Configure and verify single-area OSPFv2). It focuses on OSPF’s metric (cost), the neighbor adjacency process, and additional configuration options, providing a detailed understanding of how OSPF operates in a single-area environment. The lecture assumes familiarity with basic OSPF operations, single-area configurations, and router ID concepts from Part 1.

**Lecture Objectives**:
- Understand OSPF’s **cost** metric, how it’s calculated, and methods to modify it.
- Learn the **OSPF neighbor adjacency process**, including states and message types.
- Explore additional OSPF configurations, such as enabling OSPF directly on interfaces and managing passive interfaces.
- Prepare for CCNA exam questions on cost, neighbor states, and configurations.

**Analogy**: OSPF is like a city’s GPS network where control centers (routers) share a detailed map (LSDB) via traffic reports (LSAs). Part 2 explains how centers calculate the fastest routes (cost), handshake to become partners (neighbor states), and fine-tune their settings (configurations).

**Lifelong Retention Tip**: Visualize OSPF as a “city GPS” with routers as traffic centers. Draw a 4-router network (R1–R4) and label costs (e.g., 100 for FastEthernet) and neighbor states (Down to Full). Repeat the phrase “Cost for routes, states for neighbors, configs for control” daily for a week. Use flashcards for key terms (cost, Hello, DBD, LSR, LSU, LSAck).

## OSPF Metric: Cost

### Overview
- **Metric Name**: **Cost**, based on interface bandwidth (speed).
- **Purpose**: Determines the best path to a destination; lower cost = better path.
- **Calculation**: Cost = Reference Bandwidth ÷ Interface Bandwidth (in Mbps).
  - **Reference Bandwidth**: Default is 100 Mbps (10^8 bits per second).
  - **Interface Bandwidth**: Speed of the interface (e.g., 10 Mbps for Ethernet, 100 Mbps for FastEthernet).
- **Key Rule**: Costs < 1 are rounded up to 1 (e.g., GigabitEthernet’s 0.1 becomes 1).
- **Total Cost**: Sum of costs of all outgoing (exit) interfaces along the path to the destination.

### Default Costs
- **Ethernet (10 Mbps)**: 100 ÷ 10 = 10.
- **FastEthernet (100 Mbps)**: 100 ÷ 100 = 1.
- **GigabitEthernet (1000 Mbps)**: 100 ÷ 1000 = 0.1 → 1 (rounded up).
- **10GigEthernet (10,000 Mbps)**: 100 ÷ 10,000 = 0.01 → 1 (rounded up).
- **Loopback Interfaces**: Fixed cost of 1 (virtual, no physical bandwidth).

**Issue with Defaults**:
- FastEthernet, GigabitEthernet, and 10GigEthernet all have a cost of 1, making them appear equally preferable despite different speeds.
- **Solution**: Adjust the reference bandwidth to differentiate faster interfaces.

### Modifying Cost
1. **Change Reference Bandwidth**:
   - **Command** (in OSPF configuration mode):
     ```bash
     router ospf 1
      auto-cost reference-bandwidth <value>
     ```
     - `<value>`: Bandwidth in Mbps (e.g., 100,000 for 100 Gbps).
   - **Example** (reference bandwidth = 100,000 Mbps):
     - FastEthernet (100 Mbps): 100,000 ÷ 100 = 1000.
     - GigabitEthernet (1000 Mbps): 100,000 ÷ 1000 = 100.
     - 10GigEthernet (10,000 Mbps): 100,000 ÷ 10,000 = 10.
   - **Best Practice**: Set reference bandwidth higher than the fastest link (e.g., 100,000 Mbps for future-proofing).
   - **Requirement**: Configure the same reference bandwidth on all OSPF routers for consistent costs.
   - **Message**: “Please ensure reference bandwidth is consistent across all routers.”
2. **Manually Set Interface Cost**:
   - **Command** (in interface configuration mode):
     ```bash
     interface GigabitEthernet0/0
      ip ospf cost <value>
     ```
     - `<value>`: Desired cost (e.g., 10,000).
     - Overrides auto-calculated cost.
   - **Example**:
     ```bash
     interface GigabitEthernet0/0
      ip ospf cost 10000
     ```
     - Sets G0/0 cost to 10,000, regardless of bandwidth.
3. **Change Interface Bandwidth** (not recommended):
   - **Command** (in interface configuration mode):
     ```bash
     interface GigabitEthernet0/0
      bandwidth <value>
     ```
     - `<value>`: Bandwidth in **kbps** (e.g., 100,000 for 100 Mbps).
   - **Note**: Changes the logical bandwidth for calculations (e.g., OSPF cost, EIGRP metric), not physical speed.
   - **Speed Command** (changes physical speed):
     ```bash
     interface GigabitEthernet0/0
      speed <value>
     ```
     - `<value>`: Physical speed (e.g., 100 for 100 Mbps).
   - **Why Not Recommended?**: Bandwidth affects other protocols (e.g., EIGRP, QoS), so use `ip ospf cost` for OSPF-specific changes.

### Example (4-Router Network, Area 0)
- **Topology**: R1–R2 (GigabitEthernet, 10.0.12.0/30), R1–R3 (FastEthernet, 10.0.13.0/30), R2–R4 (FastEthernet, 10.0.24.0/30), R3–R4 (FastEthernet, 10.0.34.0/30), R4–LAN (GigabitEthernet, 192.168.4.0/24).
- **Default Reference Bandwidth (100 Mbps)**:
  - R1 to 192.168.4.0/24 via R2: G0/0 (1) + R2’s G1/0 (1) + R4’s G1/0 (1) = **3**.
  - R1 to 192.168.4.0/24 via R3: G1/0 (1) + R3’s F2/0 (1) + R4’s G1/0 (1) = **3**.
  - **Result**: Equal-Cost MultiPath (ECMP) with two routes in R1’s routing table (`show ip route`).
- **New Reference Bandwidth (100,000 Mbps)**:
  - FastEthernet (100 Mbps): 100,000 ÷ 100 = 1000.
  - GigabitEthernet (1000 Mbps): 100,000 ÷ 1000 = 100.
  - R1 to 192.168.4.0/24 via R2: G0/0 (100) + R2’s G1/0 (100) + R4’s G1/0 (100) = **300**.
  - R1 to 192.168.4.0/24 via R3: G1/0 (1000) + R3’s F2/0 (1000) + R4’s G1/0 (100) = **2100**.
  - **Result**: R1 prefers the path via R2 (cost 300 < 2100), single route in routing table.
- **Loopback Example**: R1 to R2’s Loopback0 (2.2.2.2/32):
  - Path: R1’s G0/0 (100) + R2’s Loopback0 (1) = **101**.

**Verification**:
- **Show Interface Cost**:
  ```bash
  show ip ospf interface FastEthernet2/0
  ```
  - Displays cost (e.g., 1 for default, 1000 after `auto-cost reference-bandwidth 100000`).
- **Show Brief Overview**:
  ```bash
  show ip ospf interface brief
  ```
  - Lists OSPF-enabled interfaces with costs (e.g., F2/0: 1000, G0/0: 100).
- **Show Routing Table**:
  ```bash
  show ip route ospf
  ```
  - Confirms costs (e.g., `O 192.168.4.0/24 [110/300] via 10.0.12.2`).

**Comparison to RIP/EIGRP**:
- **RIP**: Metric is hop count (max 15), ignores bandwidth.
- **EIGRP**: Metric uses bandwidth (slowest link) and delay, supports unequal-cost load-balancing.
- **OSPF**: Cost based on bandwidth, supports only ECMP (up to 4 paths by default).
- **Example**: In RIP, R1 to 192.168.4.0/24 has 2 hops via R2 or R3; in EIGRP, prefers R2 (higher bandwidth); in OSPF, prefers R2 after adjusting reference bandwidth.

**Analogy**: Cost is like travel time on city roads. Faster roads (GigabitEthernet) have lower costs, but default settings make all highways (FastEthernet+) seem equal (cost 1). Adjusting the reference bandwidth is like recalibrating the GPS to prioritize expressways.

**Lifelong Retention Tip**: Memorize: “Cost = Reference ÷ Bandwidth, minimum 1.” Draw a network with costs (e.g., FastEthernet = 1000, GigabitEthernet = 100) after setting `auto-cost reference-bandwidth 100000`. Practice in Packet Tracer: change reference bandwidth, set manual cost (`ip ospf cost 10000`), and verify with `show ip ospf interface brief`. Use mnemonic: “Auto-cost, ip ospf cost, bandwidth last.” Create flashcards for costs (e.g., “FastEthernet cost with 100,000 Mbps? 1000”).

## OSPF Neighbor Adjacency Process

### Overview
- **Purpose**: Routers must become OSPF neighbors to exchange LSAs, build identical LSDBs, and calculate routes.
- **Key Task**: Ensure routers meet conditions to form neighbors; troubleshooting neighbor issues is critical for OSPF.
- **Process**: Routers exchange **Hello packets**, progress through neighbor states, and synchronize LSDBs.

### Hello Packets
- **Sent**: When OSPF is activated on an interface (`network` or `ip ospf`).
- **Multicast Address**: 224.0.0.5 (all OSPF routers).
- **Protocol**: IP header with protocol number 89 (vs. RIP: 224.0.0.9, EIGRP: 224.0.0.10).
- **Interval**: Default **Hello timer** = 10 seconds (Ethernet); **Dead timer** = 40 seconds.
  - **Hello Timer**: Frequency of sending Hello packets.
  - **Dead Timer**: Time to wait for a Hello before declaring a neighbor down (resets on Hello receipt).
- **Contents**: Router ID, neighbor router ID, area ID, timers, etc.

### Neighbor States
1. **Down**:
   - No Hellos sent or received.
   - Example: R1 activates OSPF on G0/0, sends Hello to 224.0.0.5 with its router ID (1.1.1.1) and neighbor ID (0.0.0.0, unknown).
2. **Init**:
   - Router receives a Hello but its own router ID is not in it.
   - Example: R2 receives R1’s Hello, adds R1 to its neighbor table in Init state (R2’s ID 2.2.2.2 not in R1’s Hello).
3. **2-Way**:
   - Routers exchange Hellos containing each other’s router IDs, confirming compatibility.
   - Example: R2 sends Hello with R1’s ID (1.1.1.1); R1 receives it, moves R2 to 2-Way state, and sends Hello with R2’s ID.
   - **Significance**: All neighbor conditions met (e.g., same area, timers). Routers are now OSPF neighbors.
   - **Note**: On multi-access networks (e.g., Ethernet), a Designated Router (DR) and Backup Designated Router (BDR) may be elected (covered in Day 28).
4. **Exstart**:
   - Routers decide Master/Slave roles for LSDB exchange.
   - **Master**: Higher router ID (e.g., R2: 2.2.2.2 > R1: 1.1.1.1).
   - **Slave**: Lower router ID.
   - **Mechanism**: Exchange Database Description (DBD) packets to negotiate roles.
5. **Exchange**:
   - Routers exchange DBD packets listing LSAs in their LSDBs (not full LSAs, just summaries).
   - Example: R2 (Master) sends DBD with LSA list; R1 compares to its LSDB to identify missing LSAs.
6. **Loading**:
   - Routers request missing LSAs using **Link State Request (LSR)** packets.
   - Neighbor responds with **Link State Update (LSU)** packets containing full LSAs.
   - Routers acknowledge receipt with **Link State Acknowledgment (LSAck)** packets.
   - Example: R1 sends LSR for missing LSAs; R2 sends LSUs; R1 sends LSAcks.
7. **Full**:
   - LSDBs are synchronized; routers have full adjacency.
   - Continue sending Hellos (every 10 seconds) to maintain adjacency.
   - Dead timer (40 seconds) resets on each Hello; if it expires, neighbor is removed.

### OSPF Message Types
| **Type** | **Name**            | **Purpose**                                      |
|----------|---------------------|--------------------------------------------------|
| 1        | Hello              | Discover neighbors, maintain adjacency.           |
| 2        | Database Description (DBD) | List LSAs in LSDB for exchange.                |
| 3        | Link State Request (LSR)  | Request specific LSAs.                         |
| 4        | Link State Update (LSU)   | Send full LSAs.                                |
| 5        | Link State Acknowledgment (LSAck) | Acknowledge LSAs.                      |

**Comparison to RIP/EIGRP**:
- **RIP**: Neighbors exchange full routing tables via updates (224.0.0.9), no states, simpler but less efficient.
- **EIGRP**: Neighbors use Hello (224.0.0.10), form adjacencies via states (Init, Reply, Update), but no Master/Slave or DR/BDR.
- **OSPF**: Structured states (Down to Full), uses five message types, requires identical LSDBs, supports DR/BDR on multi-access networks.

**Analogy**: Neighbor formation is like two traffic centers (routers) shaking hands. Hellos are introductions (“Hi, I’m R1”), states are handshake steps (greeting, exchanging contact lists, syncing maps), and the Full state is a signed partnership with ongoing check-ins (Hellos every 10 seconds).

**Lifelong Retention Tip**: Memorize states with a story: “Down: strangers, Init: one-way greeting, 2-Way: mutual hello, Exstart: pick leader, Exchange: swap lists, Loading: get details, Full: partners.” Draw a timeline of R1–R2 states (Down → Full) and label messages (Hello, DBD, LSR, LSU, LSAck). In Packet Tracer, enable OSPF and watch neighbor states with `show ip ospf neighbor`. Use mnemonic: “Hello, DBD, LSR, LSU, LSAck = 1, 2, 3, 4, 5.” Create flashcards for states and timers (Hello: 10, Dead: 40).

## Additional OSPF Configurations

### Enabling OSPF Directly on Interfaces
- **Command** (in interface configuration mode):
  ```bash
  interface GigabitEthernet0/0
   ip ospf 1 area 0
  ```
  - **1**: Process ID (locally significant).
  - **area 0**: OSPF area for the interface.
- **Purpose**: Activates OSPF on the interface without using `network` in OSPF configuration mode.
- **Example**:
  ```bash
  enable
  conf t
  interface GigabitEthernet0/0
   ip ospf 1 area 0
  interface GigabitEthernet1/0
   ip ospf 1 area 0
  interface Loopback0
   ip ospf 1 area 0
  exit
  ```
- **Verification**:
  ```bash
  show ip protocols
  ```
  - Shows “Routing on Interfaces Configured Explicitly” instead of “Routing for Networks.”
- **Comparison to RIP/EIGRP**: RIP and EIGRP require `network` commands in router configuration mode; OSPF’s interface-based method is unique.

### Configuring Passive Interfaces by Default
- **Commands** (in OSPF configuration mode):
  ```bash
  router ospf 1
   passive-interface default
   no passive-interface GigabitEthernet0/0
   no passive-interface GigabitEthernet1/0
  ```
- **Purpose**: Sets all interfaces as passive (no Hello packets) by default, then enables Hellos on specific interfaces (e.g., those with neighbors).
- **Use Case**: Faster for networks with many passive interfaces (e.g., LANs, loopbacks) vs. setting `passive-interface` individually.
- **Effect**: Same as individual `passive-interface` commands; prevents unnecessary Hellos while advertising networks.
- **Comparison to RIP/EIGRP**: Both support `passive-interface` and `passive-interface default`, but OSPF’s interface-based activation adds flexibility.

**Analogy**: Enabling OSPF on interfaces is like installing GPS directly at each intersection (interface) instead of centrally (network command). Setting passive interfaces by default is like muting all radios (interfaces) and turning on only those needed for neighbor communication.

**Lifelong Retention Tip**: Practice both methods in Packet Tracer: use `network` on R1, `ip ospf` on R2, and compare `show ip protocols`. For passive interfaces, try `passive-interface default` and enable specific interfaces; verify with `show ip ospf interface`. Memorize: “ip ospf for direct, passive default for silence.” Use flashcards for commands (`ip ospf 1 area 0`, `passive-interface default`).

## Verification Commands

- **Show IP OSPF Neighbor**:
  ```bash
  show ip ospf neighbor
  ```
  - **Output**: Lists neighbors (e.g., R2: 2.2.2.2, R3: 3.3.3.3), state (Full), Dead timer (counts down from 40, resets on Hello).
  - **Example**: R1 shows R2 and R3 in Full state, with R2/R3 as DRs (explained in Day 28).
- **Show IP OSPF Interface**:
  ```bash
  show ip ospf interface GigabitEthernet0/0
  ```
  - **Output**: Shows timers (Hello: 10, Dead: 40), neighbor count (e.g., 1 for R2), DR/BDR status.
  - **Example**: G0/0 shows Hello due in 7 seconds, neighbor 2.2.2.2 (DR).
- **Show IP OSPF Interface Brief**:
  ```bash
  show ip ospf interface brief
  ```
  - **Output**: Lists OSPF-enabled interfaces, costs, and states (e.g., G0/0 cost 100, F2/0 cost 1000).
- **Show IP Route OSPF**:
  ```bash
  show ip route ospf
  ```
  - **Output**: OSPF routes with costs (e.g., `O 192.168.4.0/24 [110/300] via 10.0.12.2`).

**Comparison to RIP/EIGRP**:
- **RIP**: `show ip rip database`, `show ip route rip` (no neighbor states).
- **EIGRP**: `show ip eigrp neighbors`, `show ip eigrp topology` (similar neighbor concept, no LSDB).
- **OSPF**: Unique for `show ip ospf database` (LSDB) and detailed neighbor states.

**Analogy**: Verification is like checking a control center’s GPS logs: neighbor list (`show ip ospf neighbor`) shows partners, interface details (`show ip ospf interface`) show settings, and routing table (`show ip route ospf`) shows calculated routes.

**Lifelong Retention Tip**: In Packet Tracer, run `show ip ospf neighbor` and watch Dead timer count down; send Hellos to reset it. Memorize: “Neighbor for partners, interface for settings, route for paths.” Create a checklist: “Full state? Timers? Costs?” Practice 3 times to internalize.

## Key Takeaways

1. **OSPF Cost**:
   - Calculated as Reference Bandwidth ÷ Interface Bandwidth (minimum 1).
   - Default reference bandwidth (100 Mbps) makes FastEthernet+ equal (cost 1).
   - Modify with:
     - `auto-cost reference-bandwidth <Mbps>` (all routers, e.g., 100,000).
     - `ip ospf cost <value>` (per interface, e.g., 10,000).
     - `bandwidth <kbps>` (not recommended, affects other protocols).
   - Total cost = sum of outgoing interface costs (e.g., R1 to 192.168.4.0/24 = 300 via R2).
2. **Neighbor Adjacency Process**:
   - States: Down, Init, 2-Way, Exstart, Exchange, Loading, Full.
   - Messages: Hello (1), DBD (2), LSR (3), LSU (4), LSAck (5).
   - Key timers: Hello (10 seconds), Dead (40 seconds) on Ethernet.
   - Master/Slave decided in Exstart (higher router ID = Master).
   - Full state = synchronized LSDBs, maintained by Hellos.
3. **Additional Configurations**:
   - Enable OSPF on interfaces: `ip ospf <process-id> area <area>`.
   - Passive interfaces by default: `passive-interface default`, `no passive-interface <interface>`.
4. **Verification**:
   - `show ip ospf neighbor`: Full state, Dead timer, DR/BDR.
   - `show ip ospf interface`: Timers, neighbor count.
   - `show ip ospf interface brief`: Interface costs.
   - `show ip route ospf`: Route costs and paths.
5. **CCNA Focus**:
   - Master cost calculation and modification.
   - Memorize neighbor states and message types.
   - Understand alternative configuration methods.

## Quiz Answers and Explanations

1. **Order OSPF neighbor states**:
   - **Answer**: Down, Init, 2-Way, Exstart, Exchange, Loading, Full.
   - **Explanation**: Down (no Hellos), Init (receive Hello), 2-Way (mutual Hellos), Exstart (Master/Slave), Exchange (DBD swap), Loading (LSR/LSU/LSAck), Full (synchronized LSDBs).
   - **Tip**: Memorize with a story: “Strangers (Down) greet (Init), handshake (2-Way), pick leader (Exstart), swap lists (Exchange), get details (Loading), partners (Full).”
2. **OSPF default cost**:
   - **C**: FastEthernet, GigabitEthernet, and 10GigEthernet have the same cost (1).
   - **Explanation**: Default reference bandwidth (100 Mbps) gives FastEthernet (100 ÷ 100 = 1), GigabitEthernet (100 ÷ 1000 = 0.1 → 1), 10GigEthernet (0.01 → 1). Ethernet (10 Mbps) is 10.
   - **Tip**: Memorize: “100 Mbps reference, FastEthernet+ = 1.”
3. **State for Master/Slave decision**:
   - **A**: Exstart.
   - **Explanation**: Master (higher router ID) and Slave are decided via DBD packets in Exstart, before DBD exchange in Exchange state. 2-Way elects DR/BDR (some networks); Loading involves LSR/LSU/LSAck.
   - **Tip**: Associate Exstart with “start exchange, pick Master.”
4. **Command for FastEthernet cost of 100**:
   - **C**: `auto-cost reference-bandwidth 10000`.
   - **Explanation**: FastEthernet (100 Mbps): 10,000 ÷ 100 = 100. Other values (100, 1000, 100,000) yield incorrect costs (1, 10, 1000).
   - **Tip**: Practice formula: “Reference ÷ 100 = FastEthernet cost.” Test in Packet Tracer.
5. **Default Hello/Dead timers on Ethernet**:
   - **B**: Hello 10 seconds, Dead 40 seconds.
   - **Explanation**: Ethernet defaults are 10/40; other connections (e.g., serial) may use 30/120 (covered in Day 28).
   - **Tip**: Memorize: “Ethernet: 10 Hello, 40 Dead.” Use flashcard for timers.

**Bonus Question (Boson ExSim)**: Not detailed, but typically tests cost, neighbor states, or configurations.

**Analogy**: The quiz is like a city planner’s exam: know route calculations (cost), handshake steps (states), and GPS settings (configurations). Practice makes you the master planner.

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “Neighbor states? Down, Init, 2-Way, Exstart, Exchange, Loading, Full”). In Packet Tracer, simulate neighbor formation (enable/disable OSPF) and check states with `show ip ospf neighbor`. Repeat quiz questions 3 times to reinforce.

## Study Tips for Lifelong Retention

1. **Mnemonics for Key Concepts**:
   - Cost: “Reference ÷ Bandwidth, minimum 1.”
   - Neighbor states: “Down, Init, 2-Way, Exstart, Exchange, Loading, Full = strangers to partners.”
   - Messages: “Hello (1), DBD (2), LSR (3), LSU (4), LSAck (5).”
   - Timers: “Ethernet: 10 Hello, 40 Dead.”
2. **Visualize the Network**:
   - Draw a 4-router topology (R1–R4, area 0). Label costs (e.g., FastEthernet = 1000, GigabitEthernet = 100) and neighbor states (e.g., R1–R2: Full). Pin to your study space.
3. **Practice in Packet Tracer**:
   - Configure `auto-cost reference-bandwidth 100000`, `ip ospf cost 10000`, and check `show ip ospf interface brief`.
   - Enable OSPF with `ip ospf 1 area 0` vs. `network`, use `passive-interface default`, and verify with `show ip protocols`.
   - Simulate neighbor formation: enable OSPF on R1–R2, check states with `show ip ospf neighbor`.
   - Repeat 3–5 times to build muscle memory.
4. **Cost Calculations**:
   - Practice: FastEthernet (100 Mbps) with 10,000 Mbps reference = 100; GigabitEthernet (1000 Mbps) = 10. Write 10 examples daily for a week.
   - Mnemonic: “Reference high, cost low for fast links.”
5. **Compare Protocols**:
   - Create a table: OSPF (cost, states, LSDB), RIP (hops, no states), EIGRP (bandwidth/delay, simpler states). Quiz yourself: “OSPF metric? Cost. RIP? Hops.”
6. **Teach and Explain**:
   - Explain cost calculation and neighbor states to a peer or record yourself. Teaching reinforces memory.
7. **Troubleshooting Practice**:
   - In Packet Tracer, break neighbor formation (e.g., different areas, mismatched timers) and diagnose with `show ip ospf neighbor` and `show ip ospf interface`.
8. **Spaced Repetition**:
   - Use Anki to review cost formula, neighbor states, message types, and commands after 1 day, 1 week, 1 month.

These notes provide a solid foundation for OSPF Part 2, preparing you for CCNA exam topic 3.4 and future OSPF lectures (e.g., Day 28 on DR/BDR and network types). Practice the configurations, use the analogies, and apply the retention tips to master OSPF for life!