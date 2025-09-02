# DHCP Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring the **Dynamic Host Configuration Protocol (DHCP)** to automatically assign network configuration parameters (IP address, subnet mask, default gateway, DNS server) to clients, aligning with **CCNA exam topics 4.3** ("Explain the role of DHCP and DNS within the network") and **4.6** ("Configure and verify DHCP client and relay"). It builds on the DHCP lecture, which covered the DORA process (Discover, Offer, Request, Ack), UDP ports (67 server, 68 client), and Cisco IOS configurations. In this lab, we configure R2 as a DHCP server for three subnets, R1 as a DHCP client on G0/0 and relay agent for PC1, and PC1/PC2 as DHCP clients in Packet Tracer. The lab assumes connectivity (e.g., OSPF or static routes from Day 34) and demonstrates verification using Windows and Cisco IOS commands.

**Lab Objectives**:
- **Configure DHCP Server**: Set up R2 with three DHCP pools for 192.168.1.0/24, 192.168.2.0/24, and 203.0.113.0/30, excluding reserved addresses.
- **Configure DHCP Client**: Set R1’s G0/0 as a DHCP client to obtain an IP from R2.
- **Configure DHCP Relay**: Set R1 as a relay agent to forward PC1’s DHCP requests to R2.
- **Verify Configurations**: Confirm IP assignments on PC1/PC2 (`ipconfig /all`) and R1/R2 (`show ip dhcp binding`, `show ip interface`).
- **Prepare for Exam**: Master DHCP server, client, and relay configurations for topics 4.3 and 4.6.

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      R2        |-------|      R1        |-------|      SW1       |
| G0/0: 203.0.113.1/30 | G0/0: DHCP      |       |                |
| (DHCP Server)  |       | G0/1: 192.168.1.1/24 |  |                |
|                |       | G0/2: 192.168.2.1/24 |  |                |
+----------------+       +----------------+       +----------------+
                                         |       |                |
                                         |       |                |
                                         +-------| PC1: 192.168.1.x |
                                         |       | (DHCP client)  |
                                         +-------| PC2: 192.168.2.x |
                                         |       | (DHCP client)  |
                                         +-------+----------------+
```
- **Devices**:
  - **R2**: DHCP server (G0/0: 203.0.113.1/30), serves three pools (192.168.1.0/24, 192.168.2.0/24, 203.0.113.0/30).
  - **R1**: G0/0 (DHCP client, e.g., 203.0.113.2/30), G0/1 (192.168.1.1/24, relay agent), G0/2 (192.168.2.1/24).
  - **SW1**: Connects R1 to PC1 (192.168.1.x, DHCP) and PC2 (192.168.2.x, DHCP).
  - **PC1/PC2**: Windows PCs, obtain IPs via DHCP (e.g., 192.168.1.11, 192.168.2.11).
- **Note**: Connectivity between R1 and R2 (203.0.113.0/30) via static/OSPF routes (assumed from Day 34). PC1 uses R1 as relay; PC2 directly accesses R2’s pool.
- **Requirements**:
  1. Configure R2 with DHCP pools (POOL1: 192.168.1.0/24, POOL2: 192.168.2.0/24, POOL3: 203.0.113.0/30), exclude 192.168.1.1–10, 192.168.2.1–10, 203.0.113.1.
  2. Configure R1’s G0/0 as DHCP client (`ip address dhcp`).
  3. Configure R1’s G0/1 as relay agent (`ip helper-address 203.0.113.1`).
  4. Verify PC1/PC2 IPs (`ipconfig /all`) and R2 bindings (`show ip dhcp binding`).

**Analogy**: DHCP is like a hotel concierge (R2) assigning room numbers (IPs) to guests (PC1, PC2, R1). R1 as a relay agent is a messenger forwarding PC1’s request to the concierge (R2), and R1 as a client is a guest asking for a room number.

**Lifelong Retention Tip**: Visualize PC1 broadcasting a DHCP Discover, R1 relaying to R2, and R2 assigning 192.168.1.11. In Packet Tracer, configure `ip dhcp pool` on R2, `ip helper-address` on R1, and verify with `show ip dhcp binding`. Create flashcards: “DHCP server? `ip dhcp pool`. Relay? `ip helper-address`. Client? `ip address dhcp`.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure DHCP Server on R2
**Objective**: Configure R2 as a DHCP server with three pools (POOL1, POOL2, POOL3), excluding reserved addresses.

**Commands (on R2)**:
```plaintext
R2> enable
R2# configure terminal
R2(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
R2(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10
R2(config)# ip dhcp excluded-address 203.0.113.1
R2(config)# ip dhcp pool POOL1
R2(dhcp-config)# network 192.168.1.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.1.1
R2(dhcp-config)# exit
R2(config)# ip dhcp pool POOL2
R2(dhcp-config)# network 192.168.2.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.2.1
R2(dhcp-config)# exit
R2(config)# ip dhcp pool POOL3
R2(dhcp-config)# network 203.0.113.0 255.255.255.252
```

**Verification (on R2)**:
```plaintext
R2# show running-config | section dhcp
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp excluded-address 203.0.113.1
ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name jeremysitlab.com
ip dhcp pool POOL2
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 domain-name jeremysitlab.com
ip dhcp pool POOL3
 network 203.0.113.0 255.255.255.252
```

**Observations**:
- **Excluded Addresses**: Reserve 192.168.1.1–10, 192.168.2.1–10 (e.g., for routers/servers), and 203.0.113.1 (R2’s G0/0).
- **POOL1 (192.168.1.0/24)**: Assigns IPs (e.g., 192.168.1.11), DNS (8.8.8.8), domain (jeremysitlab.com), gateway (192.168.1.1).
- **POOL2 (192.168.2.0/24)**: Assigns IPs (e.g., 192.168.2.11), same DNS/domain, gateway (192.168.2.1).
- **POOL3 (203.0.113.0/30)**: Assigns IPs (e.g., 203.0.113.2) for R1’s G0/0; no DNS/gateway (point-to-point link).
- **show running-config | section dhcp**: Filters DHCP config, showing pools and excluded addresses.
- **Note**: No lease time specified (Packet Tracer defaults apply); lecture used 5h30m.

**Comparison to Prior Content**:
- **DHCP Lecture**: Configured one pool with lease time; lab uses three pools, no lease specified.
- **DNS Lab**: DHCP assigns DNS server (8.8.8.8); DNS lab used static DNS (1.1.1.1).
- **Extended ACLs Lab (Day 35)**: Could block DHCP (e.g., `deny udp any host 203.0.113.1 eq 67`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DHCP uses UDP 67 (server), 68 (client).
- **OSPF (Day 34)**: Routes needed for R1 to reach R2 (203.0.113.1).
- **Security (Section 5.0)**: DHCP snooping (later topic) prevents rogue servers.

**Analogy**: R2 as a DHCP server is a concierge assigning room numbers (IPs) to guests in three hotels (subnets). Excluded addresses are VIP suites reserved for staff (routers).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip dhcp pool POOL1`, verify with `show running-config | section dhcp`. Flashcard: “DHCP pool? `ip dhcp pool <name>`, `network <subnet>`. Exclude? `ip dhcp excluded-address`.” Practice 3 times.

### Step 2: Configure R1 as DHCP Client
**Objective**: Configure R1’s G0/0 interface to obtain an IP address from R2’s POOL3.

**Commands (on R1)**:
```plaintext
R1> enable
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ip address dhcp
R1(config-if)# no shutdown
```

**Verification (on R1)**:
```plaintext
R1# show ip interface g0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 203.0.113.2/30 (address determined by DHCP)
```

**Observations**:
- **ip address dhcp**: R1 sends DORA messages (Discover, Request via UDP 68; Offer, Ack via UDP 67) to R2.
- **IP Assignment**: R1 gets 203.0.113.2/30 from POOL3 (only available address after excluding 203.0.113.1).
- **no shutdown**: Ensures interface is active to initiate DORA.
- **Note**: Rare for routers (static IPs preferred); used here for demonstration.

**Comparison to Prior Content**:
- **DHCP Lecture**: Configured R2 as client; lab uses R1’s G0/0, same command.
- **DNS Lab**: R1 was DNS client (`ip name-server`); here, DHCP client (`ip address dhcp`).
- **IP Addressing (Days 5–6)**: DHCP assigns IPs like static configs in earlier labs.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DHCP client uses UDP 68, server 67.
- **OSPF (Day 34)**: Routes ensure R1 reaches R2 for DORA.
- **Security (Section 5.0)**: DHCP client vulnerable to rogue servers (snooping later).

**Analogy**: R1 as a DHCP client is a guest (G0/0) asking the concierge (R2) for a room number (203.0.113.2) in a small hotel (203.0.113.0/30).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip address dhcp` on R1’s G0/0, verify with `show ip interface g0/0`. Flashcard: “DHCP client? `ip address dhcp`. Port? UDP 68.” Practice 3 times.

### Step 3: Configure R1 as DHCP Relay Agent
**Objective**: Configure R1’s G0/1 to forward PC1’s DHCP requests to R2 (203.0.113.1).

**Commands (on R1)**:
```plaintext
R1(config)# interface g0/1
R1(config-if)# ip helper-address 203.0.113.1
```

**Verification (on R1)**:
```plaintext
R1# show ip interface g0/1
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 192.168.1.1/24
  Helper-address is 203.0.113.1
```

**Observations**:
- **ip helper-address**: Forwards PC1’s broadcast Discover/Request (UDP 68) as unicast to R2 (203.0.113.1, UDP 67).
- **Interface**: Applied on G0/1 (connected to PC1), not G0/0 (R2 link).
- **Routing**: R1 must have route to 203.0.113.1 (assumed via OSPF/static).
- **Process**: PC1 broadcasts Discover; R1 relays to R2; R2 offers 192.168.1.11 (POOL1); DORA completes via relay.

**Comparison to Prior Content**:
- **DHCP Lecture**: Relay agent forwards to remote server (SRV1); lab uses R2 as server.
- **DNS Lab**: No relay; PCs sent DNS queries directly or via R1 (routing).
- **Extended ACLs (Day 35)**: Could block relay traffic (e.g., `deny udp any host 203.0.113.1 eq 67`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Relay forwards UDP 68 to 67.
- **OSPF (Day 34)**: Routes needed for relay to reach R2.
- **Security (Section 5.0)**: Relay vulnerable to misconfigured server IPs.

**Analogy**: R1 as a relay agent is a messenger carrying PC1’s room request (Discover) to the concierge (R2) in another building, ensuring PC1 gets a room number (192.168.1.11).

**Lifelong Retention Tip**: In Packet Tracer, configure `ip helper-address 203.0.113.1` on R1’s G0/1, verify with `show ip interface g0/1`. Flashcard: “Relay command? `ip helper-address <server-ip>`. Port? UDP 67.” Practice 3 times.

### Step 4: Verify DHCP on PC1 and PC2
**Objective**: Confirm PC1 and PC2 receive IPs and configurations from R2’s pools via DHCP.

**Commands (on PC2)**:
```plaintext
C:\> ipconfig /renew
C:\> ipconfig /all
```

**Verification (on PC2)**:
```plaintext
C:\> ipconfig /all
   DHCP Enabled. . . . . . . . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 192.168.2.11
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.2.1
   DHCP Server . . . . . . . . . . . : 192.168.2.1
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   Domain Name . . . . . . . . . . . : jeremysitlab.com
```

**Commands (on PC1)**:
```plaintext
C:\> ipconfig /renew
C:\> ipconfig /all
```

**Verification (on PC1)**:
```plaintext
C:\> ipconfig /all
   DHCP Enabled. . . . . . . . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 192.168.1.11
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
   DHCP Server . . . . . . . . . . . : 203.0.113.1
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   Domain Name . . . . . . . . . . . : jeremysitlab.com
```

**Verification (on R2)**:
```plaintext
R2# show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address      Client-ID/              Lease expiration        Type
                Hardware address
192.168.1.11    [PC1 MAC]               [Timestamp]             Automatic
192.168.2.11    [PC2 MAC]               [Timestamp]             Automatic
203.0.113.2     [R1 G0/0 MAC]           [Timestamp]             Automatic
```

**Observations**:
- **PC2**: Gets 192.168.2.11 from POOL2 (first available after 192.168.2.1–10), gateway (192.168.2.1), DNS (8.8.8.8), domain (jeremysitlab.com).
- **PC1**: Gets 192.168.1.11 from POOL1 via R1’s relay, gateway (192.168.1.1), DNS (8.8.8.8), domain; DHCP server is R2 (203.0.113.1).
- **R1 G0/0**: Gets 203.0.113.2 from POOL3.
- **ipconfig /renew**: Triggers DORA; may require multiple attempts due to Packet Tracer’s slow ARP.
- **show ip dhcp binding**: Shows assigned IPs, client MACs, lease times.

**Comparison to Prior Content**:
- **DHCP Lecture**: Configured one pool; lab uses three, with relay and client roles.
- **DNS Lab**: DHCP assigns DNS server (8.8.8.8); DNS lab used static DNS (1.1.1.1).
- **Extended ACLs Lab (Day 35)**: Could block DHCP traffic (e.g., `deny udp any any eq 67 68`).

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: DHCP uses UDP 67/68, critical for relay.
- **OSPF (Day 34)**: Routes ensure R1 relays to R2.
- **Security (Section 5.0)**: DHCP snooping prevents rogue servers.

**Analogy**: PC1/PC2 are guests receiving room numbers (192.168.1.11, 192.168.2.11) from the concierge (R2), with PC1’s request relayed by a messenger (R1). R1’s G0/0 is a guest getting a room (203.0.113.2).

**Lifelong Retention Tip**: In Packet Tracer, run `ipconfig /renew` on PC1, verify with `ipconfig /all` and R2’s `show ip dhcp binding`. Flashcard: “Verify DHCP? `ipconfig /all`, `show ip dhcp binding`.” Practice 3 times.

## Common Issues and Troubleshooting

- **PC1 Fails to Get IP**:
  - **Issue**: `ipconfig /renew` on PC1 fails.
  - **Cause**: Missing `ip helper-address` on R1’s G0/1 or no route to R2 (203.0.113.1).
  - **Fix**: Verify `ip helper-address 203.0.113.1` (`show ip interface g0/1`), check route (`show ip route`).
- **PC2 Gets Wrong IP**:
  - **Issue**: PC2 gets IP outside 192.168.2.0/24.
  - **Cause**: Incorrect `network` in POOL2 or overlapping pools.
  - **Fix**: Verify `network 192.168.2.0 255.255.255.0` in POOL2 (`show running-config | section dhcp`).
- **R1 G0/0 No IP**:
  - **Issue**: `ip address dhcp` fails on R1’s G0/0.
  - **Cause**: POOL3 misconfigured or R2 unreachable.
  - **Fix**: Verify POOL3 (`network 203.0.113.0 255.255.255.252`), ping 203.0.113.1.
- **Slow IP Assignment**:
  - **Issue**: `ipconfig /renew` requires multiple attempts.
  - **Cause**: Packet Tracer’s slow ARP process.
  - **Fix**: Retry `ipconfig /renew` or fast-forward simulation mode.

**Analogy**: Troubleshooting DHCP is like ensuring a hotel guest gets a room key: check if the concierge (R2) has rooms (`ip dhcp pool`), the messenger (R1) is forwarding requests (`ip helper-address`), and the phone line (UDP 67/68) is open.

**Lifelong Retention Tip**: In Packet Tracer, simulate failed `ipconfig /renew` on PC1 (remove `ip helper-address`), check `show ip interface g0/1`, reconfigure. Checklist: “No IP? Check pool, helper, routes, UDP 67/68.” Practice 3 times.

## Practice Commands

- **DHCP Server (R2)**:
```plaintext
R2(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
R2(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10
R2(config)# ip dhcp excluded-address 203.0.113.1
R2(config)# ip dhcp pool POOL1
R2(dhcp-config)# network 192.168.1.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.1.1
R2(config)# ip dhcp pool POOL2
R2(dhcp-config)# network 192.168.2.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.2.1
R2(config)# ip dhcp pool POOL3
R2(dhcp-config)# network 203.0.113.0 255.255.255.252
```
- **DHCP Client (R1)**:
```plaintext
R1(config)# interface g0/0
R1(config-if)# ip address dhcp
R1(config-if)# no shutdown
```
- **DHCP Relay (R1)**:
```plaintext
R1(config)# interface g0/1
R1(config-if)# ip helper-address 203.0.113.1
```
- **Verify (R2)**:
```plaintext
R2# show running-config | section dhcp
R2# show ip dhcp binding
```
- **Verify (R1)**:
```plaintext
R1# show ip interface g0/0
R1# show ip interface g0/1
```
- **Verify (PC1/PC2)**:
```plaintext
C:\> ipconfig /renew
C:\> ipconfig /all
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - DORA: “Discover, Offer, Request, Ack.”
   - Commands: “Server: `ip dhcp pool`, Relay: `ip helper-address`, Client: `ip address dhcp`.”
2. **Visualize Topology**:
   - Draw PC1 → R1 (relay) → R2 (server) for 192.168.1.11; PC2 → R2 for 192.168.2.11. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DHCP ports? 67 server, 68 client. Relay? `ip helper-address`. Verify bindings? `show ip dhcp binding`.”
4. **Practice in Packet Tracer**:
   - Configure `ip dhcp pool` on R2, `ip helper-address` on R1, `ip address dhcp` on R1’s G0/0. Verify with `show ip dhcp binding`, `ipconfig /all`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: DHCP (UDP 67/68, assigns IPs) vs. DNS (UDP 53, resolves names).
   - Quiz: “Relay command? `ip helper-address <server-ip>`.”
6. **Troubleshooting Practice**:
   - Simulate failed `ipconfig /renew` (remove `ip helper-address`), check `show ip interface`, reconfigure. Checklist: “No IP? Check pool, helper, routes, UDP 67/68.”
7. **Teach Someone**:
   - Explain DORA, relay, and server roles to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, ports, DORA. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DHCP Purpose**:
   - Automates IP, subnet, gateway, DNS assignment for clients (PC1, PC2, R1’s G0/0).
2. **Configuration**:
   - Server (R2): `ip dhcp pool`, `network`, `dns-server`, `default-router`, exclude addresses.
   - Relay (R1): `ip helper-address 203.0.113.1` on G0/1.
   - Client (R1): `ip address dhcp` on G0/0.
3. **Verification**:
   - PC1/PC2: `ipconfig /all` (192.168.1.11, 192.168.2.11, DNS 8.8.8.8).
   - R2: `show ip dhcp binding` (bindings for 192.168.1.11, 192.168.2.11, 203.0.113.2).
   - R1: `show ip interface g0/0` (203.0.113.2), `g0/1` (helper-address).
4. **Packet Tracer**:
   - Slow ARP may delay `ipconfig /renew`; retry or fast-forward.
5. **CCNA Focus**:
   - Topics 4.3 (DHCP role), 4.6 (configure/verify server, relay, client).
6. **Relay Role**:
   - Essential when DHCP server (R2) is not in client’s subnet (PC1).

These notes provide a complete foundation for the DHCP lab, preparing you for CCNA exam topics 4.3 and 4.6. Practice configurations in Packet Tracer, verify with `show ip dhcp binding` and `ipconfig /all`, and use analogies/retention tips to master DHCP for life!