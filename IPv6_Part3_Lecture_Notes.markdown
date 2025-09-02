# IPv6 Part 3 Lecture Notes for CCNA Beginners

## Introduction

This lecture concludes the IPv6 series for the CCNA, focusing on exam topic 3.3 (IPv4/IPv6 static routing), with connections to 1.9 (IPv6 address types, e.g., solicited-node multicast) and 1.8 (IPv6 configuration/verification, e.g., SLAAC). It builds on the IPv6 Part 1 lecture/lab (address basics, dual-stack) and Part 2 lecture/lab (address types, EUI-64, static routes), addressing IPv6 address representation, the IPv6 header, Neighbor Discovery Protocol (NDP), Stateless Address Auto-configuration (SLAAC), and IPv6 static routing. The goal is to solidify IPv6 confidence, as many CCNA students find it challenging due to limited exposure compared to IPv4.

**Lecture Objectives**:
- **Understand IPv6 Address Representation**: Learn RFC 5952 rules for writing IPv6 addresses.
- **Explore IPv6 Header**: Compare its fixed 40-byte structure to IPv4’s variable header.
- **Learn NDP**: Understand how NDP replaces ARP and supports router discovery.
- **Master SLAAC**: Configure and verify Stateless Address Auto-configuration.
- **Configure IPv6 Static Routes**: Apply IPv4 static routing concepts to IPv6, including network, host, default, and floating routes.
- **Prepare for Exam**: Master topics 3.3, 1.9 (solicited-node multicast), and 1.8 (SLAAC, verification).

**Network Topology (Example)**:
- **Devices**: R1, R2, R3, PC1, PC2.
- **Subnets**:
  - R1 G0/1: `2001:db8::/64` (PC1: `2001:db8::2/64`).
  - R2 G0/1: `2001:db8:0:1::/64` (PC2: `2001:db8:0:1::2/64`).
  - R3 LAN: `2001:db8:0:3::/64`.
  - R1–R2 link (G0/0): `2001:db8:0:12::/64` (R1: `2001:db8:0:12::1/64`, R2: `::2/64`).
  - Link-local addresses: `FE80::/10` (EUI-64-based, e.g., `FE80::34:56FF:FE78:90AA`).
- **Purpose**: Demonstrate NDP, SLAAC, and static routes (e.g., R1 to R3’s LAN via R2).

**Analogy**: IPv6 is like a modern postal system replacing IPv4’s older system. NDP is the postmaster, directing mail (packets) using solicited-node multicast (group mailboxes) instead of ARP’s broadcast shouts. SLAAC automates address assignment like self-registering mailboxes, and static routes are predefined delivery paths. RFC 5952 standardizes address labels, and the IPv6 header is a streamlined envelope.

**Lifelong Retention Tip**: Visualize R1 pinging PC2 via R2, using NDP to resolve MACs, SLAAC to assign addresses, and static routes to navigate subnets. Practice in Packet Tracer: configure `ipv6 address autoconfig`, static routes, and verify with `show ipv6 neighbor`. Create flashcards for NDP messages (`NS = 135`), header fields (`hop limit = TTL`), and route types (`recursive = next-hop only`). Practice 3 times daily for a week.

## IPv6 Address Representation (RFC 5952)

**Overview**:
- **RFC 5952**: Standardizes IPv6 address text representation (Request for Comments, Internet Society/IETF).
- **Rules**:
  1. **Remove Leading Zeros**: Each quartet must omit leading zeros (e.g., `2001:0db8:0000:0001::` → `2001:db8:0:1::`).
  2. **Double Colon for Longest Zero Run**: Use `::` to shorten the longest string of all-zero quartets (e.g., `2001:db8:0:0:0:0:0:1` → `2001:db8::1`). If equal-length runs, shorten the leftmost.
  3. **Single Zero Quartet**: Don’t use `::` for one all-zero quartet (e.g., `2001:db8:0:0:1:0:0:1` → `2001:db8:0:0:1:0:0:1`, not `2001:db8::1:0:0:1`).
  4. **Lower-Case Hex**: Use `a–f` (not `A–F`) for hex digits (e.g., `2001:DB8::1` → `2001:db8::1`).
- **Note**: Cisco routers may use upper-case hex (e.g., `FE80::34:56FF:FE78:90AB`), technically incorrect but common. Not critical for CCNA.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Used mixed case, some leading zeros (e.g., `2001:DB8:0000:1::1/64`). Part 3 standardizes to lower-case, no leading zeros.
- **Part 1 Lab**: Configured `2001:db8:0:1::2/64`, no focus on representation rules.
- **Part 2 Lecture/Lab**: Used upper-case in examples (e.g., `2001:DB8::/64`), noted link-local `FE80::`. Part 3 enforces RFC 5952.

**Comparison to Other CCNA Topics**:
- **IPv4 Addressing**: No equivalent standardization (e.g., `192.168.001.001` valid but rare).
- **OSPFv3**: Uses IPv6 addresses, applies RFC 5952 in modern implementations.
- **STP**: No addressing, irrelevant.

**Analogy**: RFC 5952 is like a postal formatting guide, ensuring all addresses (e.g., `2001:db8::1`) are written consistently for clarity.

**Lifelong Retention Tip**: Write `2001:0db8:0000:0000:0000:0000:0000:0001`, shorten to `2001:db8::1` (lower-case, no leading zeros). Flashcard: “RFC 5952? Lower-case, remove leading zeros, `::` for longest zero run.” Practice 3 times.

## IPv6 Header

**Overview**:
- **Structure**: Fixed 40-byte header (vs. IPv4’s 20–60 bytes), simpler for router processing.
- **Fields** (8):
  1. **Version** (4 bits): `0110` (IPv6).
  2. **Traffic Class** (8 bits): QoS priority (e.g., VoIP), covered later in CCNA.
  3. **Flow Label** (20 bits): Identifies traffic flows (e.g., client-server file transfer).
  4. **Payload Length** (16 bits): Size of Layer 4 segment (e.g., TCP/UDP) in bytes, excludes 40-byte header.
  5. **Next Header** (8 bits): Type of next header (e.g., TCP=6, UDP=17), like IPv4’s Protocol field.
  6. **Hop Limit** (8 bits): Decremented by each router; packet discarded at 0 (like IPv4 TTL).
  7. **Source Address** (128 bits): Sender’s IPv6 address.
  8. **Destination Address** (128 bits): Receiver’s IPv6 address.
- **Note**: No header length field (fixed 40 bytes). Unlikely to be tested directly but foundational.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Not covered, focused on address basics.
- **Part 1 Lab**: No header discussion, focused on configuration.
- **Part 2 Lecture/Lab**: Not covered, focused on address types, EUI-64.
- **Similarity**: All assume IPv6 packet processing, not detailed until Part 3.

**Comparison to Other CCNA Topics**:
- **IPv4 Header**: Variable length (20–60 bytes), more fields (e.g., Header Length, Checksum). IPv6 simpler, no checksum.
- **TCP/UDP**: Next Header field specifies TCP (6) or UDP (17), like IPv4 Protocol field.
- **OSPFv3**: Uses IPv6 header for packet transport, hop limit for routing.

**Analogy**: The IPv6 header is a streamlined envelope (40 bytes) vs. IPv4’s bulky, variable-sized package. Fields like hop limit (TTL) and addresses guide delivery.

**Lifelong Retention Tip**: Draw IPv6 header (40 bytes, 8 fields), label Version (`0110`), Hop Limit (TTL). Flashcard: “IPv6 header size? 40 bytes, fixed.” Practice 3 times.

## Neighbor Discovery Protocol (NDP)

**Overview**:
- **Definition**: Replaces IPv4’s ARP, supports router discovery and Duplicate Address Detection (DAD). Uses ICMPv6.
- **Functions**:
  1. **MAC Address Resolution** (replaces ARP):
     - **Neighbor Solicitation (NS)**: ICMPv6 Type 135, asks for MAC (like ARP Request).
     - **Neighbor Advertisement (NA)**: ICMPv6 Type 136, provides MAC (like ARP Reply).
     - **Solicited-Node Multicast**: Address `ff02::1:ffXX:XXXX` (last 6 hex digits of unicast address, e.g., `2001:db8::78:9abc` → `ff02::1:ff78:9abc`).
     - **Example**:
       - R1 pings R2 (`2001:db8::78:9abc`).
       - R1 sends NS to `ff02::1:ff78:9abc` (source: R1’s IP/MAC, destination: multicast IP/MAC).
       - R2 replies with NA (source: R2’s IP/MAC, destination: R1’s IP/MAC).
     - **Efficiency**: Multicast vs. ARP’s broadcast.
     - **Verification**: `show ipv6 neighbor` shows neighbor table (IPv6 address, MAC, interface).
  2. **Router Discovery**:
     - **Router Solicitation (RS)**: ICMPv6 Type 133, sent to `ff02::2` (all routers) when interface enables.
     - **Router Advertisement (RA)**: ICMPv6 Type 134, sent to `ff02::1` (all nodes), provides prefix, default gateway.
     - **Example**: R2’s G0/0 enables, sends RS; R1 replies with RA, announcing `2001:db8::/64`.
  3. **Duplicate Address Detection (DAD)**:
     - Device sends NS to its own solicited-node multicast address (e.g., `ff02::1:ff78:9abc`).
     - No NA reply = address unique; NA reply = duplicate, router logs error.
- **Note**: NDP operates on all IPv6 devices, not just routers. Critical for CCNA but not explicitly listed in exam topics.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Not covered, focused on basics.
- **Part 1 Lab**: No NDP, but link-local addresses used (NDP-related).
- **Part 2 Lecture**: Mentioned NDP for link-local (OSPFv3, static routes), not detailed. Noted `ff02::1` (all nodes), `ff02::2` (all routers).
- **Part 2 Lab**: Used link-local addresses for static routes, implying NDP.
- **Similarity**: Link-local (`FE80::`) and multicast (`ff02::`) addresses tied to NDP.

**Comparison to Other CCNA Topics**:
- **IPv4 ARP**: Broadcast-based, less efficient than NDP’s multicast. No router discovery or DAD.
- **OSPFv3**: Uses NDP (link-local, `ff02::5`) for neighbor adjacencies.
- **ICMP**: NDP uses ICMPv6 (Types 133–136), unlike IPv4 ICMP (ping).

**Analogy**: NDP is the IPv6 postmaster, using group mailboxes (`ff02::1:ffXX:XXXX`) to find addresses (NS/NA), announce routers (RS/RA), and check for duplicate mailboxes (DAD).

**Lifelong Retention Tip**: In Packet Tracer, ping R2’s IPv6, check `show ipv6 neighbor` for MAC. Draw NS (`ff02::1:ffXX:XXXX`) and RA (`ff02::1`) flow. Flashcards: “NS type? 135. RA address? `ff02::1`.” Practice 3 times.

## Stateless Address Auto-configuration (SLAAC)

**Overview**:
- **Definition**: Hosts auto-configure IPv6 addresses using NDP’s RS/RA messages to learn the prefix (e.g., `2001:db8::/64`), then generate interface ID via EUI-64 or random.
- **Command**: `ipv6 address autoconfig` (Cisco IOS, interface mode).
- **Process**:
  - Host sends RS (`ff02::2`) on interface enable.
  - Router replies with RA (`ff02::1`), providing prefix (e.g., `2001:db8::/64`).
  - Host uses prefix + EUI-64 (or random ID) to form address (e.g., `2001:db8::34:56ff:fe78:90ab/64`).
- **Example**:
  - R1 G0/0: `2001:db8::1/64`.
  - R2 G0/0: `ipv6 address autoconfig`, learns `2001:db8::/64`, generates `2001:db8::34:56ff:fe78:90ad/64`.
  - Link-local (`FE80::`) always auto-configured.
- **Note**: SLAAC is standard for IPv6 hosts (PCs, routers), not Cisco-specific. No prefix specified in command (unlike `ipv6 address <prefix>/64 eui-64`).

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Not covered, focused on manual config.
- **Part 1 Lab**: Used manual addresses (`2001:db8:0:1::2/64`), no SLAAC.
- **Part 2 Lecture**: Covered EUI-64 (`ipv6 address <prefix>/64 eui-64`), requires manual prefix. SLAAC auto-learns prefix.
- **Part 2 Lab**: Used EUI-64, not SLAAC. Both generate link-local addresses.
- **Similarity**: SLAAC and EUI-64 use `/64` prefixes, `2001:db8::/32` range.

**Comparison to Other CCNA Topics**:
- **IPv4 DHCP**: Similar auto-configuration, but DHCP requires a server, SLAAC uses router RA.
- **OSPFv3**: May use SLAAC addresses for routing interfaces.
- **STP**: No addressing, irrelevant.

**Analogy**: SLAAC is like a new resident (host) asking the postmaster (router) for the town’s zip code (prefix) via RS/RA, then creating a house number (EUI-64) automatically.

**Lifelong Retention Tip**: In Packet Tracer, configure `ipv6 address autoconfig` on R2, verify with `show ipv6 interface brief`. Flashcard: “SLAAC command? `ipv6 address autoconfig`.” Practice 3 times.

## IPv6 Static Routing

**Overview**:
- **Definition**: Configures fixed paths for IPv6 traffic, similar to IPv4. Requires `ipv6 unicast-routing` (disabled by default, unlike IPv4).
- **Command**: `ipv6 route <destination-prefix> {<exit-interface> | <next-hop> | <exit-interface> <next-hop>} [ad <value>]`.
- **Route Types** (same as IPv4):
  1. **Directly Attached**: Specifies exit interface only (e.g., `ipv6 route 2001:db8:0:3::/64 g0/0`). **Note**: Doesn’t work on Ethernet interfaces in IPv6 (works in IPv4).
  2. **Recursive**: Specifies next-hop only (e.g., `ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2`). Requires recursive lookup to find exit interface.
  3. **Fully Specified**: Specifies both (e.g., `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2` or `FE80::34:56ff:fe78:90ad`). Required for link-local next-hop.
- **Route Categories**:
  - **Network Route**: To a subnet (e.g., `2001:db8:0:3::/64`).
  - **Host Route**: To a single host, `/128` (e.g., `2001:db8:0:1::2/128`).
  - **Default Route**: To all destinations, `::/0` (e.g., `ipv6 route ::/0 2001:db8:0:12::1`).
  - **Floating Static**: Backup route with higher administrative distance (AD) (e.g., `ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2 120`, AD > OSPF’s 110).
- **Routing Table**:
  - Command: `show ipv6 route`.
  - Includes:
    - **Connected Network Routes**: `/64` for configured interfaces (e.g., `2001:db8::/64`).
    - **Local Host Routes**: `/128` for each interface address (e.g., `2001:db8::1/128`).
    - **Multicast Route**: `ff00::/8` to `Null0` (discards multicast, auto-configured).
    - **No Link-Local Routes**: `FE80::` addresses not in table.
- **Link-Local Next-Hop**:
  - Requires interface specification (e.g., `ipv6 route 2001:db8:0:3::/64 g0/0 FE80::34:56ff:fe78:90ad`).
  - Fully specified route, as router can’t resolve link-local without interface.

**Example Configuration**:
```plaintext
R1# configure terminal
R1(config)# ipv6 unicast-routing
R1(config)# ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2  ! Recursive, network
R2(config)# ipv6 route 2001:db8::2/128 2001:db8:0:12::1    ! Recursive, host (PC1)
R2(config)# ipv6 route 2001:db8:0:1::2/128 2001:db8:0:1::1 ! Recursive, host (PC2)
R3(config)# ipv6 route ::/0 2001:db8:0:12::1              ! Recursive, default
R1(config)# ipv6 route 2001:db8:0:3::/64 g0/0 FE80::34:56ff:fe78:90ad  ! Fully specified, link-local
```

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Introduced `ipv6 unicast-routing`, no routing details.
- **Part 1 Lab**: Used `ipv6 unicast-routing` for pings, no routes.
- **Part 2 Lecture**: Noted link-local in static routes (OSPFv3 context).
- **Part 2 Lab**: Configured fully specified routes with link-local next-hops (`ipv6 route 2001:db8:0:1::/64 g0/0 FE80::...`).
- **Similarity**: All require `ipv6 unicast-routing`, use `2001:db8::/32`, `/64` prefixes.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Same types (directly attached, recursive, fully specified), but IPv6 disallows directly attached on Ethernet.
- **OSPFv3**: Dynamic routing, uses link-local like static routes here.
- **FHRP (HSRP)**: Virtual IPs for redundancy, unlike link-local next-hops.

**Analogy**: IPv6 static routes are like predefined postal routes, with recursive routes checking multiple maps (lookups) and fully specified routes including both street (interface) and house (next-hop).

**Lifelong Retention Tip**: In Packet Tracer, configure `ipv6 route 2001:db8:0:3::/64 g0/0 FE80::...`, verify with `show ipv6 route`. Flashcards: “Recursive route? Next-hop only. Link-local route? Fully specified.” Practice 3 times.

## Quiz Answers and Explanations

1. **Message for MAC Address (R2 to R1)**:
   - **Answer**: B (NA, Neighbor Advertisement).
   - **Explanation**: R1 sends NS (Type 135) to learn R2’s MAC; R2 replies with NA (Type 136). RS/RA are for router discovery.

2. **DAD Message**:
   - **Answer**: D (NS, Neighbor Solicitation).
   - **Explanation**: DAD sends NS to own solicited-node multicast address (`ff02::1:ffXX:XXXX`). No NA = unique address.

3. **RA Destination Address**:
   - **Answer**: C (`ff02::1`, all-nodes).
   - **Explanation**: RA sent to `ff02::1` (all hosts), not `ff02::2` (all routers) or `ff01::` (interface-local).

4. **Static Route Type** (`ipv6 route 2001:db8:0:3::/64 g0/0 2001:db8:0:12::2`):
   - **Answers**: A (Fully Specified), B (Network).
   - **Explanation**: Specifies interface and next-hop (fully specified), targets subnet `/64` (network), not `/128` (host).

5. **Recursive Host Route**:
   - **Answer**: C (`ipv6 route 2001:db8:0:1::2/128 2001:db8:0:12::1`).
   - **Explanation**: Recursive (next-hop only), host route (`/128`). A = directly attached, network; B = fully specified, host; D = recursive, network.

**Bonus Question (Boson ExSim)**: Likely tests NDP (NS/NA, RS/RA), SLAAC, or static route types.

**Analogy**: The quiz is like sorting postal tasks: identifying delivery requests (NA), checking for duplicate mailboxes (NS), or routing mail to group addresses (`ff02::1`).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “RA address? `ff02::1`”). Practice NDP scenarios (ping, `show ipv6 neighbor`) and static route configs in Packet Tracer. Repeat quiz 3 times.

## Common Issues and Troubleshooting

- **Ping Failure**:
  - **Issue**: R1 can’t ping PC2 (`2001:db8:0:1::2`).
  - **Cause**: Missing `ipv6 unicast-routing` or static route.
  - **Fix**: Enable `ipv6 unicast-routing`, configure `ipv6 route 2001:db8:0:1::/64 2001:db8:0:12::2`, verify with `show ipv6 route`.
- **No MAC Resolution**:
  - **Issue**: No entry in `show ipv6 neighbor`.
  - **Cause**: NDP failure, interface down, or wrong address.
  - **Fix**: Check interface (`show ipv6 interface brief`), ping destination, verify NS/NA with debug (`debug ipv6 nd`).
- **SLAAC Failure**:
  - **Issue**: No address after `ipv6 address autoconfig`.
  - **Cause**: No RA from router (missing `ipv6 unicast-routing` or prefix).
  - **Fix**: Ensure router has `ipv6 unicast-routing`, configured prefix (e.g., `2001:db8::/64`).
- **Static Route Error**:
  - **Issue**: “Interface has to be specified for a link-local nexthop.”
  - **Fix**: Use fully specified route (e.g., `ipv6 route 2001:db8:0:3::/64 g0/0 FE80::...`).

**Analogy**: Troubleshooting is like fixing postal delays: ensure the hub is active (`ipv6 unicast-routing`), mailboxes are reachable (NDP), and routes are clear (static routes).

**Lifelong Retention Tip**: Simulate failed ping, check `ipv6 unicast-routing`, neighbor table, routes. Checklist: “No ping? Check routing, NDP, routes.” Practice 3 times.

## Practice Commands

- **Enable IPv6 Routing**:
  ```plaintext
  ipv6 unicast-routing
  ```
- **SLAAC Configuration**:
  ```plaintext
  interface g0/0
  ipv6 address autoconfig
  ```
- **Static Routes**:
  ```plaintext
  ipv6 route 2001:db8:0:3::/64 2001:db8:0:12::2           ! Recursive, network
  ipv6 route 2001:db8:0:1::2/128 2001:db8:0:12::1         ! Recursive, host
  ipv6 route ::/0 2001:db8:0:12::1                        ! Recursive, default
  ipv6 route 2001:db8:0:3::/64 g0/0 FE80::34:56ff:fe78:90ad  ! Fully specified, link-local
  ```
- **Verify**:
  ```plaintext
  show ipv6 interface brief
  show ipv6 neighbor
  show ipv6 route
  ping 2001:db8:0:1::2
  debug ipv6 nd
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - NDP: “NS = 135, NA = 136, RS = 133, RA = 134.”
   - SLAAC: “`autoconfig`, learns prefix via RA.”
   - Static Routes: “Recursive = next-hop, fully specified = interface + next-hop.”
2. **Visualize Topology**:
   - Draw R1–R3, show NDP (NS to `ff02::1:ffXX:XXXX`, RA to `ff02::1`), SLAAC on R2, static routes to PC2. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Solicited-node? `ff02::1:ffXX:XXXX`. RA address? `ff02::1`. Recursive route? Next-hop only.”
   - Header: “40 bytes, hop limit = TTL.”
4. **Practice in Packet Tracer**:
   - Configure `ipv6 address autoconfig`, static routes (recursive, fully specified), ping PC2, verify with `show ipv6 neighbor`, `show ipv6 route`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Part 1 (manual config), Part 2 (EUI-64, link-local routes), Part 3 (NDP, SLAAC, static routes). Quiz: “NS type? 135.”
6. **Troubleshooting Practice**:
   - Simulate missing route or NDP failure, fix, verify. Checklist: “No ping? Check unicast-routing, NDP, routes.”
7. **Teach Someone**:
   - Explain NDP (NS/NA for MAC, RS/RA for routers), SLAAC, static routes. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki for NDP messages, SLAAC command, route types. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **IPv6 Address Representation**:
   - RFC 5952: Lower-case hex, remove leading zeros, `::` for longest zero run.
2. **IPv6 Header**:
   - Fixed 40 bytes, 8 fields (Version, Hop Limit, etc.), simpler than IPv4.
3. **NDP**:
   - Replaces ARP (NS Type 135, NA Type 136, solicited-node `ff02::1:ffXX:XXXX`).
   - Router discovery (RS Type 133, RA Type 134, `ff02::1`).
   - DAD (NS to own address).
4. **SLAAC**:
   - `ipv6 address autoconfig`, learns prefix via RA, uses EUI-64 or random ID.
5. **IPv6 Static Routing**:
   - Types: Directly attached (not Ethernet), recursive, fully specified.
   - Categories: Network (`/64`), host (`/128`), default (`::/0`), floating (higher AD).
   - Link-local next-hop requires interface (fully specified).
6. **CCNA Focus**:
   - Exam topic 3.3: Configure/verify static routes (network, host, default, floating).
   - Exam topic 1.9: Solicited-node multicast (`ff02::1:ffXX:XXXX`).
   - Exam topic 1.8: SLAAC, verification (`show ipv6 neighbor`, `show ipv6 route`).

These notes provide a complete foundation for the IPv6 Part 3 lecture, preparing you for CCNA exam topics 3.3, 1.9, and 1.8. Practice NDP, SLAAC, and static routing in Packet Tracer, using the analogies and retention tips to master IPv6 for life!