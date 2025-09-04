# GRE Tunnels Lab Notes for CCNA Beginners

## Lab Overview

This lab configures a **Generic Routing Encapsulation (GRE)** tunnel between two routers (R1 and R2) over a service provider network, aligning with **CCNA exam topic 5.5 (site-to-site VPNs)**. The GRE tunnel creates a virtual point-to-point connection, allowing R1 and R2 to behave as if directly connected, with packets encapsulated in GRE headers. Although GRE is not explicitly required for the CCNA exam, it illustrates tunneling concepts, supports multicast (e.g., OSPF), and complements the **WAN Architecture Lecture (topic 1.2.d)** and **VPNs (topic 5.5)**. The lab configures the tunnel, enables OSPF for route sharing, and verifies connectivity between PC1 and PC2 across the tunnel.

**Lab Objectives**:
- **Understand GRE Tunnels**: Create a virtual point-to-point link over a service provider network.
- **Configure GRE**: Set up tunnel interfaces on R1 and R2 with source, destination, and IP addresses.
- **Enable OSPF**: Allow R1 and R2 to share LAN routes via the tunnel, supporting multicast.
- **Verify Connectivity**: Ensure PC1 (R1’s LAN) can ping PC2 (R2’s LAN) through the tunnel.
- **Prepare for CCNA**: Reinforce topic 5.5 (site-to-site VPNs, GRE over IPsec) and 1.2.d (WAN tunneling).
- **Reinforce Prior Content**: Link to WAN Architecture, OSPF (1.3.d), and IP Addressing (2.2).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------| Router R1      |-------| Service Provider|-------| Router R2      |
| (10.0.1.100)   | G0/0  | G0/0/0:100.0.0.2|       |                |       | G0/0/0:200.0.0.2|
|                |       | Tun0:192.168.1.1|       |                |       | Tun0:192.168.1.2|
+----------------+       +----------------+       +----------------+       +----------------+
                                                        |                | G0/0  |      PC2       |
                                                        |                |-------| (10.0.2.100)   |
                                                        |                |       +----------------+
```
- **Devices**:
  - **PC1**: End host in R1’s LAN (10.0.1.100/24, gateway 10.0.1.1).
  - **R1**: Router with LAN (G0/0, 10.0.1.1/24), WAN (G0/0/0, 100.0.0.2/24), Tunnel0 (192.168.1.1/30).
  - **Service Provider**: Connects R1 and R2, forwards GRE-encapsulated packets.
  - **R2**: Router with LAN (G0/0, 10.0.2.1/24), WAN (G0/0/0, 200.0.0.2/24), Tunnel0 (192.168.1.2/30).
  - **PC2**: End host in R2’s LAN (10.0.2.100/24, gateway 10.0.2.1).
- **Note**:
  - **GRE Tunnel**: Virtual interface (Tunnel0), IP 192.168.1.0/30, source/destination on WAN interfaces.
  - **Encapsulation**: Original packet (e.g., PC1 to PC2) encapsulated with GRE header and outer IP header (100.0.0.2 to 200.0.0.2).
  - **OSPF**: Enabled on Tunnel0 and LAN interfaces to share routes (10.0.1.0/24, 10.0.2.0/24).
  - **Security**: GRE is not secure (no encryption); typically paired with IPsec in production.
- **Requirements**:
  1. Configure GRE tunnel: R1 (source G0/0/0, destination 200.0.0.2, IP 192.168.1.1), R2 (source G0/0/0, destination 100.0.0.2, IP 192.168.1.2).
  2. Add default routes for tunnel destinations (R1 to 200.0.0.2, R2 to 100.0.0.2).
  3. Enable OSPF on Tunnel0 and LAN interfaces, set passive interfaces for LANs.
  4. Verify tunnel status (`show ip interface brief`), routes (`show ip route`), and ping from PC1 to PC2.
  5. Connect to WAN Architecture (GRE over IPsec) and OSPF concepts.

**Analogy**: A GRE tunnel is like a private mail tube between two offices (R1, R2) through a public post office (service provider). Packets are sealed in an outer envelope (GRE header) to travel as if directly connected, but without locks (no encryption).

**Lifelong Retention Tip**: Visualize PC1’s packet traveling through the GRE tunnel from R1 to R2. Create flashcards: “GRE? Tunnel, no encryption. Source? WAN interface. OSPF? Multicast over tunnel.” Practice in Packet Tracer: configure, ping, verify. Review 3 times daily for a week.

## GRE Tunnels Fundamentals

**Purpose**: Explain GRE tunnels as a site-to-site VPN technology (topic 5.5).

**Key Points**:
- **Definition**: GRE creates a virtual point-to-point link over a service provider network, encapsulating packets with GRE and outer IP headers.
- **Characteristics**:
  - Supports multicast (e.g., OSPF), unlike plain IPsec.
  - No encryption, not secure alone; often paired with IPsec (topic 5.5).
  - Uses virtual tunnel interfaces (e.g., Tunnel0).
- **Operation**:
  - Source: Local router’s WAN interface (e.g., R1 G0/0/0).
  - Destination: Remote router’s WAN IP (e.g., R2’s 200.0.0.2).
  - Tunnel IP: Virtual interface subnet (e.g., 192.168.1.0/30).
- **Use Case**: Enables OSPF between R1 and R2, sharing LAN routes (10.0.1.0/24, 10.0.2.0/24).
- **Exam Note**: Topic 5.5 tests GRE over IPsec for multicast; lab focuses on GRE basics for tunneling.

**Comparison to Prior Content**:
- **WAN Architecture Lecture (1.2.d, 5.5)**: GRE supports site-to-site VPNs, complements IPsec.
- **OSPF Lecture (1.3.d)**: GRE enables OSPF over tunnels (multicast).
- **IP Addressing Lecture (2.2)**: Tunnel uses /30 subnet, WAN/LAN use /24.
- **Security Fundamentals Lecture (5.1)**: GRE lacks encryption (no Confidentiality).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: Tunnel (192.168.1.0/30), WAN (100.0.0.0/24, 200.0.0.0/24), LAN (10.0.1.0/24, 10.0.2.0/24).
- **Day 15 (TCP/UDP)**: Supports ping (ICMP) over tunnel.
- **Day 18 (ARP)**: Resolves tunnel IPs (192.168.1.1 to 192.168.1.2).

**Analogy**: GRE is a clear plastic tube through a public network—packets are visible unless encrypted with IPsec.

**Lifelong Retention Tip**: Simulate GRE tunnel in Packet Tracer, ping across. Flashcard: “GRE? Virtual tunnel, multicast OK, no encryption.” Practice 3 times.

## Lab Steps and Analysis

### Step 1: Configure GRE Tunnel on R1
**Objective**: Create Tunnel0 on R1 with source G0/0/0, destination 200.0.0.2, IP 192.168.1.1.

**Steps (on R1)**:
1. Enter global configuration mode:
```plaintext
R1> enable
R1# configure terminal
```
2. Create tunnel interface:
```plaintext
R1(config)# interface tunnel 0
```
3. Set tunnel source and destination:
```plaintext
R1(config-if)# tunnel source g0/0/0
R1(config-if)# tunnel destination 200.0.0.2
```
4. Assign tunnel IP address:
```plaintext
R1(config-if)# ip address 192.168.1.1 255.255.255.252
```
5. Verify tunnel status:
```plaintext
R1(config-if)# do show ip interface brief
```

**Observations**:
- **Tunnel0**: Up/down state (down because R2 not configured, no route to 200.0.0.2).
- **Configuration**:
  - Source: G0/0/0 (100.0.0.2).
  - Destination: R2’s WAN (200.0.0.2).
  - IP: 192.168.1.1/30 (Tunnel0).
- **Exam Note**: Topic 5.5 tests GRE concepts (tunnel interface, source/destination).

**Comparison to Prior Content**:
- **WAN Architecture Lecture (5.5)**: GRE creates virtual link for site-to-site VPN.
- **IP Addressing Lecture (2.2)**: Tunnel uses /30 subnet for point-to-point.
- **OSPF Lecture (1.3.d)**: Prepares for OSPF over tunnel.

**Analogy**: R1’s Tunnel0 is a mail tube entrance, waiting for R2’s end to connect.

**Lifelong Retention Tip**: Configure Tunnel0 in Packet Tracer, check `show ip interface brief`. Flashcard: “GRE R1? Source G0/0/0, dest 200.0.0.2, IP 192.168.1.1.” Practice 3 times.

### Step 2: Configure GRE Tunnel on R2
**Objective**: Create Tunnel0 on R2 with source G0/0/0, destination 100.0.0.2, IP 192.168.1.2.

**Steps (on R2)**:
1. Enter global configuration mode:
```plaintext
R2> enable
R2# configure terminal
```
2. Create tunnel interface:
```plaintext
R2(config)# interface tunnel 0
```
3. Set tunnel source and destination:
```plaintext
R2(config-if)# tunnel source g0/0/0
R2(config-if)# tunnel destination 100.0.0.2
```
4. Assign tunnel IP address:
```plaintext
R2(config-if)# ip address 192.168.1.2 255.255.255.252
```
5. Verify tunnel status:
```plaintext
R2(config-if)# do show ip interface brief
```

**Observations**:
- **Tunnel0**: Up/down state (down because no route to 100.0.0.2).
- **Configuration**:
  - Source: G0/0/0 (200.0.0.2).
  - Destination: R1’s WAN (100.0.0.2).
  - IP: 192.168.1.2/30 (Tunnel0).
- **Issue**: Tunnel down due to missing route to 100.0.0.2.
- **Exam Note**: Topic 5.5 tests tunnel configuration and troubleshooting.

**Comparison to Prior Content**:
- **WAN Architecture Lecture (5.5)**: Symmetric GRE config for site-to-site.
- **IP Addressing Lecture (2.2)**: /30 for tunnel, /24 for WAN.
- **Routing Lecture (1.3)**: Need route to tunnel destination.

**Analogy**: R2’s Tunnel0 is the other end of the mail tube, incomplete without a route to R1.

**Lifelong Retention Tip**: Configure Tunnel0 on R2, check `show ip interface brief`. Flashcard: “GRE R2? Source G0/0/0, dest 100.0.0.2, IP 192.168.1.2.” Practice 3 times.

### Step 3: Add Routes and Verify Tunnel
**Objective**: Add default routes on R1 and R2 to reach tunnel destinations, verify tunnel up.

**Steps**:
1. On R2, add default route to 100.0.0.2:
```plaintext
R2(config)# exit
R2(config)# ip route 0.0.0.0 0.0.0.0 200.0.0.1
R2(config)# do show ip route
R2(config)# do show ip interface brief
```
2. On R1, add default route to 200.0.0.2:
```plaintext
R1(config)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 100.0.0.1
R1(config)# do show ip interface brief
```
3. Test tunnel connectivity from R1:
```plaintext
R1(config)# do ping 192.168.1.2
```

**Observations**:
- **R2 Routes**:
  - Before: No route to 100.0.0.2, Tunnel0 down.
  - After: Default route via 200.0.0.1, Tunnel0 up, connected route 192.168.1.0/30.
- **R1 Routes**:
  - Before: No route to 200.0.0.2, Tunnel0 down.
  - After: Default route via 100.0.0.1, Tunnel0 up, ping to 192.168.1.2 succeeds.
- **Ping**: Initial failures due to ARP (Packet Tracer), then succeeds.
- **Encapsulation** (Simulation Mode):
  - Packet: ICMP (ping 192.168.1.1 to 192.168.1.2).
  - Inner IP: Source 192.168.1.1, destination 192.168.1.2.
  - GRE Header: Encapsulates inner packet.
  - Outer IP: Source 100.0.0.2, destination 200.0.0.2.
- **Exam Note**: Topic 5.5 tests GRE troubleshooting (routes, tunnel status).

**Comparison to Prior Content**:
- **WAN Architecture Lecture (5.5)**: GRE requires routes to destination IPs.
- **Routing Lecture (1.3)**: Default route enables tunnel connectivity.
- **IP Addressing Lecture (2.2)**: Tunnel IPs (192.168.1.0/30) form virtual link.

**Analogy**: Default routes are like postal directions to ensure the mail tube connects R1 to R2.

**Lifelong Retention Tip**: Add routes in Packet Tracer, ping 192.168.1.2. Flashcard: “Tunnel down? No route to dest. Fix? Default route.” Practice 3 times.

### Step 4: Configure OSPF for Route Sharing
**Objective**: Enable OSPF on R1 and R2 to share LAN routes (10.0.1.0/24, 10.0.2.0/24) via tunnel.

**Steps**:
1. On R1, configure OSPF:
```plaintext
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.1 0.0.0.0 area 0
R1(config-router)# network 10.0.1.1 0.0.0.0 area 0
R1(config-router)# passive-interface g0/0
```
2. On R2, configure OSPF:
```plaintext
R2(config)# router ospf 1
R2(config-router)# network 192.168.1.2 0.0.0.0 area 0
R2(config-router)# network 10.0.2.1 0.0.0.0 area 0
R2(config-router)# passive-interface g0/0
```
3. Verify OSPF routes:
```plaintext
R1(config-router)# do show ip route
R2(config-router)# do show ip route
```

**Observations**:
- **OSPF**:
  - R1: Advertises 10.0.1.0/24 (LAN), 192.168.1.1 (tunnel), area 0.
  - R2: Advertises 10.0.2.0/24 (LAN), 192.168.1.2 (tunnel), area 0.
  - Passive G0/0: Prevents OSPF neighbor formation on LAN interfaces.
- **Routes**:
  - R1: Learns 10.0.2.0/24 via Tunnel0 (192.168.1.2).
  - R2: Learns 10.0.1.0/24 via Tunnel0 (192.168.1.1).
- **Exam Note**: Topic 1.3.d (OSPF) tests route sharing; topic 5.5 tests GRE’s multicast support.

**Comparison to Prior Content**:
- **WAN Architecture Lecture (5.5)**: GRE enables OSPF over tunnels (multicast).
- **OSPF Lecture (1.3.d)**: `network` commands, passive interfaces.
- **IP Addressing Lecture (2.2)**: LAN routes shared via OSPF.

**Analogy**: OSPF over GRE is like exchanging office addresses through the mail tube, enabling direct deliveries.

**Lifelong Retention Tip**: Configure OSPF in Packet Tracer, check `show ip route`. Flashcard: “OSPF GRE? Shares LAN routes, passive LAN.” Practice 3 times.

### Step 5: Test Connectivity from PC1 to PC2
**Objective**: Verify PC1 (10.0.1.100) can ping PC2 (10.0.2.100) via GRE tunnel.

**Steps (on PC1)**:
1. Check IP configuration:
```plaintext
C:\> ipconfig
IP Address: 10.0.1.100
Subnet Mask: 255.255.255.0
Default Gateway: 10.0.1.1
```
2. Ping PC2:
```plaintext
C:\> ping 10.0.2.100
```

**Observations**:
- **Before OSPF**: Ping fails (no route to 10.0.2.0/24).
- **After OSPF**: Ping succeeds (R1 learns 10.0.2.0/24, R2 learns 10.0.1.0/24).
- **Encapsulation**: PC1’s packet (10.0.1.100 to 10.0.2.100) encapsulated with GRE header, outer IP (100.0.0.2 to 200.0.0.2).
- **Packet Tracer**: Initial ping failures due to ARP, then succeeds.
- **Exam Note**: Topic 5.5 tests GRE connectivity; topic 1.3.d tests OSPF routes.

**Comparison to Prior Content**:
- **WAN Architecture Lecture (5.5)**: GRE enables site-to-site connectivity.
- **OSPF Lecture (1.3.d)**: Routes enable PC1-PC2 communication.
- **IP Addressing Lecture (2.2)**: LAN IPs for ping test.

**Analogy**: PC1’s ping is a letter sent through the GRE tube, delivered after OSPF provides the address.

**Lifelong Retention Tip**: Test ping in Packet Tracer, verify routes. Flashcard: “PC1 to PC2? OSPF shares routes, GRE tunnels.” Practice 3 times.

## Common Issues and Troubleshooting

- **Tunnel Down**:
  - **Issue**: Tunnel0 up/down on R1/R2.
  - **Cause**: No route to tunnel destination (100.0.0.2/200.0.0.2).
  - **Fix**: Add `ip route 0.0.0.0 0.0.0.0 <next-hop>` (R1: 100.0.0.1, R2: 200.0.0.1).
  - **Verify**: `show ip interface brief`, `show ip route`.
- **Ping Fails (Tunnel IPs)**:
  - **Issue**: R1 cannot ping 192.168.1.2.
  - **Cause**: Tunnel down or ARP incomplete.
  - **Fix**: Ensure tunnel up, wait for ARP (Packet Tracer delay).
  - **Verify**: `ping 192.168.1.2`, `show arp`.
- **Ping Fails (PC1 to PC2)**:
  - **Issue**: PC1 cannot ping 10.0.2.100.
  - **Cause**: No OSPF routes, incorrect network statements, or non-passive LAN interface.
  - **Fix**: Check `router ospf 1`, `network` commands, `passive-interface g0/0`.
  - **Verify**: `show ip route`, `show ip ospf neighbor`.
- **OSPF Neighbor Failure**:
  - **Issue**: R1-R2 not OSPF neighbors.
  - **Cause**: Tunnel down, incorrect `network` statements, or MTU mismatch.
  - **Fix**: Verify tunnel, `network 192.168.1.x 0.0.0.0 area 0`.
  - **Verify**: `show ip ospf neighbor`.

**Analogy**: Troubleshooting is like fixing a mail tube: ensure directions (routes), proper addressing (OSPF), and tube connectivity (tunnel up).

**Lifelong Retention Tip**: Simulate ping failure in Packet Tracer, fix routes/OSPF. Checklist: “No ping? Check tunnel, routes, OSPF neighbor.” Practice 3 times.

## Additional CCNA-Relevant Information

**Why This Matters for CCNA**:
- **Exam Topic 5.5**: GRE over IPsec supports multicast for site-to-site VPNs.
- **Exam Topic 1.3.d**: OSPF over GRE shares routes, leveraging multicast.
- **Exam Topic 1.2.d**: GRE as a WAN tunneling technology.
- **Packet Tracer**: Supports GRE, OSPF configs for lab practice.

**Key CCNA Concepts Reinforced**:
- **GRE**:
  - Virtual tunnel, no encryption, supports multicast.
  - Config: `interface tunnel`, `tunnel source`, `tunnel destination`, `ip address`.
- **OSPF**:
  - Shares LAN routes over tunnel (10.0.1.0/24, 10.0.2.0/24).
  - Passive interfaces prevent unnecessary neighbor formation.
- **Tunneling**:
  - Encapsulates packets (inner IP, GRE, outer IP).
  - Requires routes to destination IPs.
- **Connectivity**: Ping from PC1 to PC2 tests GRE/OSPF functionality.

**Additional Configurations (Not in Lab)**:
- MTU Adjustment (if needed):
```plaintext
R1(config-if)# interface tunnel 0
R1(config-if)# ip mtu 1400
```
- GRE Keepalives:
```plaintext
R1(config-if)# keepalive 10 3
```

**Practice in Packet Tracer**:
1. Replicate lab: Configure GRE, OSPF on R1/R2.
2. Test ping from PC1 to PC2, verify routes (`show ip route`).
3. Break tunnel (remove route), troubleshoot, fix.
4. Verify encapsulation in simulation mode (`ping 192.168.1.2`).

**Additional Resources**:
- **Cisco Documentation**: Review GRE, OSPF in CCNA study guide.
- **Boson ExSim**: Practice GRE/VPN questions (tunneling, multicast).
- **Cloudflare Articles**: Search “GRE tunnels” for real-world context.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - GRE: “Tunnel, Source, Destination, IP” (TSDI).
   - OSPF: “Network, Area, Passive” (NAP).
2. **Visualize Topology**:
   - Draw PC1 → R1 → Tunnel → R2 → PC2, label IPs (10.0.1.100, 192.168.1.1, 200.0.0.2). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “GRE? Tunnel0, no encryption. Tunnel IPs? 192.168.1.0/30. OSPF? Shares LAN routes.”
4. **Practice in Packet Tracer**:
   - Configure GRE/OSPF, test ping, verify routes. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: GRE (multicast, no encryption) vs. IPsec (unicast, secure).
   - Quiz: “GRE tunnel? Source WAN, dest remote WAN. Ping fails? No route.”
6. **Troubleshooting Practice**:
   - Simulate tunnel down, fix with routes. Checklist: “Tunnel down? Check route, interface status.”
7. **Teach Someone**:
   - Explain GRE tunnel and OSPF to a peer, use mail tube analogy.
8. **Spaced Repetition**:
   - Use Anki for commands (`tunnel source`, `network`), IPs. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **GRE Tunnels**:
   - Virtual point-to-point link, encapsulates packets with GRE and outer IP headers.
   - Supports multicast (OSPF), no encryption (use with IPsec in production).
2. **Configuration**:
   - R1: `interface tunnel 0`, source G0/0/0, destination 200.0.0.2, IP 192.168.1.1/30.
   - R2: Source G0/0/0, destination 100.0.0.2, IP 192.168.1.2/30.
   - Default routes to reach tunnel destinations.
3. **OSPF**:
   - Shares LAN routes (10.0.1.0/24, 10.0.2.0/24) via Tunnel0.
   - Passive LAN interfaces (G0/0) to prevent neighbor formation.
4. **Verification**:
   - `show ip interface brief`: Tunnel up.
   - `show ip route`: OSPF routes for remote LANs.
   - `ping 10.0.2.100`: PC1 to PC2 succeeds.
5. **CCNA Focus**:
   - Topic 5.5: GRE over IPsec for site-to-site VPNs, multicast support.
   - Topic 1.3.d: OSPF over GRE for route sharing.
   - Topic 1.2.d: GRE as WAN tunneling technology.
6. **Benefits**:
   - Simplifies connectivity between distant LANs.
   - Enables dynamic routing (OSPF) over tunnels.

These notes provide a comprehensive foundation for GRE Tunnels (CCNA topics 5.5, 1.2.d, 1.3.d). Practice configurations in Packet Tracer, use flashcards for commands/IPs, and apply analogies/retention tips to master concepts for life!