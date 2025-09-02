# IPv6 Part 2 Lab Notes for CCNA Beginners

## Lab Overview

This lab builds on the IPv6 Part 1 lecture/lab (address basics, dual-stack) and Part 2 lecture (address types, EUI-64), focusing on configuring IPv6 addresses using modified EUI-64, enabling IPv6 with link-local addresses, and setting up IPv6 static routes. It aligns with CCNA exam topics 1.9 (address types, EUI-64) and 1.8 (configuration/verification). The lab uses two routers (R1, R2) and two PCs (PC1, PC2) to demonstrate EUI-64 address generation, link-local address auto-configuration, static routing, and connectivity testing via pings. The use of context-sensitive help for static routes introduces practical Cisco IOS skills.

**Lab Objectives**:
- **Configure EUI-64 Addresses**: Use `ipv6 address <prefix>/64 eui-64` on router interfaces.
- **Enable IPv6 with Link-Local**: Use `ipv6 enable` to auto-configure link-local addresses.
- **Configure IPv6 Static Routes**: Set up routes using link-local next-hop addresses.
- **Verify Configurations**: Confirm addresses and routes with `show ipv6 interface brief` and pings.
- **Prepare for Exam**: Master EUI-64 (1.9.f), address types (1.9.a–e), and configuration/verification (1.8).

**Network Topology**:
- **Devices**:
  - **R1**:
    - G0/1: `2001:db8::/64` with EUI-64 (e.g., `2001:db8::02<MAC-based-ID>/64`), link-local `FE80::02<MAC-based-ID>`.
    - G0/0: Link-local only (`FE80::/10`) via `ipv6 enable`.
  - **R2**:
    - G0/1: `2001:db8:0:1::/64` with EUI-64 (e.g., `2001:db8:0:1:02<MAC-based-ID>/64`), link-local `FE80::02<MAC-based-ID>`.
    - G0/0: Link-local only (`FE80::/10`) via `ipv6 enable`.
  - **PC1**: `2001:db8::2/64`, default gateway R1’s G0/1 (`2001:db8::02<MAC-based-ID>/64`).
  - **PC2**: `2001:db8:0:1::2/64`, default gateway R2’s G0/1 (`2001:db8:0:1:02<MAC-based-ID>/64`).
- **Address Block**: `2001:db8::/32` (example range), subnets `2001:db8::/64` and `2001:db8:0:1::/64`.
- **Connectivity**: R1’s G0/0 connects to R2’s G0/0, static routes enable PC1 to ping PC2.

**Analogy**: The lab is like setting up a new postal system (IPv6) in two towns (subnets). R1 and R2 use their IDs (MAC addresses) to generate house numbers (EUI-64), activate local mailboxes (link-local), and establish delivery routes (static routes) to connect PC1 and PC2.

**Lifelong Retention Tip**: Visualize R1 and R2 with G0/1 addresses (`2001:db8::/64`, EUI-64) and G0/0 link-local (`FE80::`). Draw PC1 pinging PC2 via static routes. Practice in Packet Tracer: configure EUI-64, `ipv6 enable`, static routes, and verify pings. Create flashcards for commands (`ipv6 address eui-64`, `ipv6 route`) and address types (`FE80:: = link-local`). Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure EUI-64 Addresses on R1 and R2 G0/1
**Objective**: Configure IPv6 addresses on R1 and R2’s G0/1 using EUI-64, calculate interface IDs from MAC addresses.

**Process**:
- **EUI-64 Steps** (from Part 2 lecture):
  1. Split 48-bit MAC (12 hex digits) in half.
  2. Insert `FFFE` in middle (64 bits).
  3. Invert 7th bit (3rd bit of 2nd hex digit).
- **R1 Example**:
  - Command: `show interfaces g0/1`.
  - MAC: `0034.5678.90AB` (example).
  - Split: `0034.56` | `78.90AB`.
  - Insert `FFFE`: `0034.56FF.FE78.90AB`.
  - Invert 7th bit: `0` (in `00`) = `0000` → `0000` (no change).
  - Interface ID: `0034.56FF.FE78.90AB`.
  - Address: `2001:db8::0034:56FF:FE78:90AB/64`.
- **R2 Example**:
  - MAC: `0034.5678.90AC` (example, differs in last digit).
  - Split: `0034.56` | `78.90AC`.
  - Insert `FFFE`: `0034.56FF.FE78.90AC`.
  - Invert 7th bit: `0` → `0` (no change).
  - Interface ID: `0034.56FF.FE78.90AC`.
  - Address: `2001:db8:0:1:0034:56FF:FE78:90AC/64`.

**Commands (R1)**:
```plaintext
R1> enable
R1# configure terminal
R1(config)# ipv6 unicast-routing
R1(config)# interface g0/1
R1(config-if)# ipv6 address 2001:db8::/64 eui-64
R1(config-if)# end
R1# show ipv6 interface brief
GigabitEthernet0/1 [up/up]
    FE80::34:56FF:FE78:90AB
    2001:DB8::34:56FF:FE78:90AB/64
```

**Commands (R2)**:
```plaintext
R2> enable
R2# configure terminal
R2(config)# ipv6 unicast-routing
R2(config)# interface g0/1
R2(config-if)# ipv6 address 2001:db8:0:1::/64 eui-64
R2(config-if)# end
R2# show ipv6 interface brief
GigabitEthernet0/1 [up/up]
    FE80::34:56FF:FE78:90AC
    2001:DB8:0:1:34:56FF:FE78:90AC/64
```

- **Observations**:
  - `ipv6 unicast-routing` enables IPv6 routing (required for later pings).
  - EUI-64 generates interface ID from MAC, combined with prefix (`2001:db8::/64` or `2001:db8:0:1::/64`).
  - Link-local addresses (`FE80::/10`) auto-configured with same EUI-64 interface ID.
- **Note**: MAC addresses differ slightly (e.g., `90AB` vs. `90AC`), affecting interface IDs.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Used manual addresses (`2001:db8:0:1::1/64`). Part 2 lab uses EUI-64 for automation.
- **Part 1 Lab**: Configured static addresses on PCs, similar to PC1/PC2 here.
- **Part 2 Lecture**: Detailed EUI-64 process (split, FFFE, invert 7th bit), applied here.
- **Similarity**: All use `2001:db8::/32` range, /64 prefixes, link-local auto-configuration.

**Comparison to Other CCNA Topics**:
- **IPv4 Configuration**: No EUI-64; uses manual or DHCP addresses (e.g., `192.168.1.1/24`).
- **MAC Addresses**: EUI-64 extends 48-bit MAC to 64-bit interface ID.
- **OSPFv3**: Uses EUI-64 for link-local addresses, not global unicast like this lab.
- **STP**: No addressing, irrelevant.

**Analogy**: EUI-64 is like stamping a house number (interface ID) from a resident’s ID (MAC), adding a standard code (`FFFE`), and flipping a switch (7th bit) for R1/R2’s mailboxes.

**Lifelong Retention Tip**: In Packet Tracer, configure `ipv6 address 2001:db8::/64 eui-64` on R1’s G0/1, check MAC with `show interfaces g0/1`, verify address with `show ipv6 interface brief`. Calculate EUI-64 for `0034.5678.90AB`. Flashcard: “EUI-64 steps? Split, FFFE, invert 7th.” Practice 3 times.

### Step 2: Configure IPv6 Addresses and Default Gateways on PC1 and PC2
**Objective**: Assign IPv6 addresses and default gateways to PC1 and PC2.

**Configuration** (Packet Tracer GUI):
- **PC1**:
  - Default Gateway: R1’s G0/1 (`2001:db8::34:56FF:FE78:90AB/64`, EUI-64 address).
  - IPv6 Address: `2001:db8::2/64` (FastEthernet0).
- **PC2**:
  - Default Gateway: R2’s G0/1 (`2001:db8:0:1:34:56FF:FE78:90AC/64`, EUI-64 address).
  - IPv6 Address: `2001:db8:0:1::2/64`.
- **Note**: Configured via Config tab, /64 prefix aligns with subnets. PCs have link-local addresses based on their MACs.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Didn’t cover PC config, focused on routers.
- **Part 1 Lab**: Configured PCs similarly (`2001:db8:0:1::2/64`, gateway `::1`).
- **Part 2 Lecture**: Noted manual or EUI-64 interface IDs; PCs here use manual addresses.
- **Similarity**: PCs use `2001:db8::/32` range, /64 prefixes, link-local auto-configured.

**Comparison to Other CCNA Topics**:
- **IPv4 PC Config**: Similar (e.g., `192.168.1.2/24`, gateway `192.168.1.1`), but IPv6 uses hex, slash notation.
- **DHCPv6**: Not used here; manual config like Part 1 lab.
- **OSPFv3**: PCs use router’s IPv6 address as gateway, like OSPFv3’s global unicast.
- **STP**: No PC addressing, irrelevant.

**Analogy**: Configuring PCs is like assigning house numbers (`::2/64`) and a post office (gateway, EUI-64 address) in the IPv6 postal system.

**Lifelong Retention Tip**: In Packet Tracer, configure PC1 with `2001:db8::2/64`, gateway `2001:db8::34:56FF:FE78:90AB`. Repeat for PC2. Flashcard: “PC1 address? `2001:db8::2/64`.” Practice 3 times.

### Step 3: Enable IPv6 on R1 and R2 G0/0 (Link-Local Only)
**Objective**: Use `ipv6 enable` to auto-configure link-local addresses on R1 and R2’s G0/0.

**Commands (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface g0/0
R1(config-if)# ipv6 enable
R1(config-if)# end
R1# show ipv6 interface brief
GigabitEthernet0/0 [up/up]
    FE80::34:56FF:FE78:90AA
```

**Commands (R2)**:
```plaintext
R2# configure terminal
R2(config)# interface g0/0
R2(config-if)# ipv6 enable
R2(config-if)# end
R2# show ipv6 interface brief
GigabitEthernet0/0 [up/up]
    FE80::34:56FF:FE78:90AD
```

- **Observations**:
  - `ipv6 enable`: Enables IPv6, auto-configures link-local address (`FE80::/10`) using EUI-64.
  - Example: R1’s G0/0 MAC `0034.5678.90AA` → `FE80::34:56FF:FE78:90AA`.
  - Different interface IDs from G0/1 due to unique MACs.
- **Note**: Link-local addresses used for static route next-hops (Step 4).

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Noted link-local auto-configuration in `show ipv6 interface brief`.
- **Part 1 Lab**: Observed link-local (`FE80::1`) with `ipv6 address`, not `ipv6 enable`.
- **Part 2 Lecture**: Explained link-local (`FE80::/10`, subnet-only, EUI-64), used in OSPFv3/NDP.
- **Similarity**: All show link-local auto-configuration with EUI-64.

**Comparison to Other CCNA Topics**:
- **IPv4 Link-Local**: Exists (169.254.0.0/16), but not auto-enabled like IPv6.
- **OSPFv3**: Uses link-local for neighbor adjacencies, like G0/0 here.
- **ARP/NDP**: Link-local used in NDP, not relevant here but noted in Part 2 lecture.

**Analogy**: `ipv6 enable` is like activating a local mailbox (link-local) for a router’s branch (G0/0), using its ID (MAC) to generate the address.

**Lifelong Retention Tip**: In Packet Tracer, run `ipv6 enable` on G0/0, verify `FE80::` with `show ipv6 interface brief`. Flashcard: “Link-local command? `ipv6 enable`.” Practice 3 times.

### Step 4: Configure IPv6 Static Routes and Test Connectivity
**Objective**: Configure static routes on R1 and R2 to enable PC1 to ping PC2, using link-local next-hop addresses.

**Commands (R1)**:
```plaintext
R1# configure terminal
R1(config)# ipv6 route 2001:db8:0:1::/64 g0/0 FE80::34:56FF:FE78:90AD
R1(config)# end
```

**Commands (R2)**:
```plaintext
R2# configure terminal
R2(config)# ipv6 route 2001:db8::/64 g0/0 FE80::34:56FF:FE78:90AA
R2(config)# end
```

**Ping Test (PC1)**:
```plaintext
PC1> ping 2001:db8:0:1::2
Reply from 2001:db8:0:1::2: time<1ms
```

- **Details**:
  - **Static Route Syntax**: `ipv6 route <destination-prefix> <interface> <next-hop>`.
  - **R1 Route**: To `2001:db8:0:1::/64` (PC2’s subnet) via G0/0, next-hop R2’s G0/0 link-local (`FE80::34:56FF:FE78:90AD`).
  - **R2 Route**: To `2001:db8::/64` (PC1’s subnet) via G0/0, next-hop R1’s G0/0 link-local (`FE80::34:56FF:FE78:90AA`).
  - **Error Handling**: Link-local next-hop requires interface specification (e.g., `g0/0 FE80::...`), shown via context-sensitive help (`?`).
  - **Ping Success**: PC1 (`2001:db8::2/64`) pings PC2 (`2001:db8:0:1::2/64`) after routes configured.
- **Note**: Static routes preview Day 33; lab uses link-local next-hops, a key IPv6 feature.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: No routing; focused on `ipv6 unicast-routing`, manual addresses.
- **Part 1 Lab**: Required `ipv6 unicast-routing` for inter-subnet pings, no routes configured.
- **Part 2 Lecture**: Noted link-local use in static routes and OSPFv3, demonstrated here.
- **Similarity**: All emphasize `ipv6 unicast-routing`, link-local addresses.

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routes**: Similar (`ip route 192.168.1.0 255.255.255.0 fa0/0 192.168.2.1`), but IPv6 uses link-local next-hops.
- **OSPFv3**: Uses link-local for adjacencies, like static route next-hops here.
- **FHRP (HSRP)**: Uses virtual IPs, not link-local like IPv6 routes.

**Analogy**: Static routes are like setting up delivery paths between towns (subnets), using local mailbox addresses (link-local) to guide mail from PC1 to PC2.

**Lifelong Retention Tip**: In Packet Tracer, configure `ipv6 route 2001:db8:0:1::/64 g0/0 FE80::...`, ping PC1 to PC2. Flashcard: “Static route syntax? `ipv6 route <prefix> <interface> <next-hop>`.” Practice 3 times.

## Common Issues and Troubleshooting

- **Ping Failure**:
  - **Issue**: PC1 can’t ping PC2 (`2001:db8:0:1::2`).
  - **Cause**: Missing `ipv6 unicast-routing` or static routes.
  - **Fix**: Enable `ipv6 unicast-routing`, configure routes, verify with `show ipv6 route`.
- **Incorrect EUI-64 Address**:
  - **Issue**: Wrong address on G0/1 (e.g., not matching MAC).
  - **Fix**: Recalculate EUI-64, reconfigure `ipv6 address <prefix>/64 eui-64`, verify with `show ipv6 interface brief`.
- **Link-Local Not Configured**:
  - **Issue**: No `FE80::` on G0/0.
  - **Cause**: `ipv6 enable` not issued.
  - **Fix**: Run `ipv6 enable`, verify with `show ipv6 interface brief`.
- **Static Route Error**:
  - **Issue**: “Interface has to be specified for a link-local nexthop.”
  - **Fix**: Use `ipv6 route <prefix> <interface> <next-hop>`, e.g., `g0/0 FE80::...`.
  - **Verify**: `show ipv6 route`.

**Analogy**: Troubleshooting is like fixing a postal route: ensure the hub is active (`ipv6 unicast-routing`), mailboxes are set (EUI-64, link-local), and delivery paths are correct (static routes).

**Lifelong Retention Tip**: Simulate a failed ping, check `ipv6 unicast-routing`, routes, and addresses. Create checklist: “No ping? Check routing, EUI-64, link-local, routes.” Practice 3 times.

## Practice Commands

- **Enable IPv6 Routing**:
  ```plaintext
  ipv6 unicast-routing
  ```
- **Configure EUI-64 Address**:
  ```plaintext
  interface g0/1
  ipv6 address 2001:db8::/64 eui-64
  ```
- **Enable Link-Local Address**:
  ```plaintext
  interface g0/0
  ipv6 enable
  ```
- **Configure Static Route**:
  ```plaintext
  ipv6 route 2001:db8:0:1::/64 g0/0 FE80::34:56FF:FE78:90AD
  ```
- **Verify**:
  ```plaintext
  show ipv6 interface brief
  show ipv6 route
  ping 2001:db8:0:1::2
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - EUI-64: “Split, FFFE, flip 7th.”
   - Link-Local: “`ipv6 enable`, FE80::.”
   - Static Route: “`ipv6 route`, interface, link-local.”
2. **Visualize Topology**:
   - Draw R1/R2 with G0/1 (EUI-64), G0/0 (link-local), routes to PC1/PC2. Show ping path. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “EUI-64 address? Prefix + MAC-based ID. Link-local? FE80::/10. Static route? `ipv6 route <prefix> <interface> <next-hop>`.”
   - Addresses: `2001:db8::2/64` (PC1), `FE80::34:56FF:FE78:90AB` (R1 G0/1).
4. **Practice in Packet Tracer**:
   - Configure EUI-64, `ipv6 enable`, static routes. Ping PC1 to PC2. Verify with `show ipv6 interface brief`, `show ipv6 route`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Part 1 (manual addresses, no routes) vs. Part 2 (EUI-64, link-local, routes). Quiz: “Link-local command? `ipv6 enable`.”
6. **Troubleshooting Practice**:
   - Simulate missing route or wrong EUI-64, fix, verify. Checklist: “No ping? Check unicast-routing, addresses, routes.”
7. **Teach Someone**:
   - Explain EUI-64, link-local, static routes to a peer. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki for EUI-64 steps, link-local range (`FE80::/10`), route syntax. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **EUI-64 Configuration**:
   - `ipv6 address <prefix>/64 eui-64` generates address from MAC (split, FFFE, invert 7th bit).
   - Example: MAC `0034.5678.90AB` → `2001:db8::34:56FF:FE78:90AB/64`.
2. **Link-Local Addresses**:
   - `ipv6 enable` auto-configures `FE80::/10` address using EUI-64.
   - Used for static route next-hops, OSPFv3, NDP.
3. **IPv6 Static Routes**:
   - Syntax: `ipv6 route <prefix> <interface> <next-hop>` (link-local requires interface).
   - Enables inter-subnet connectivity (e.g., PC1 to PC2).
4. **Verification and Testing**:
   - `show ipv6 interface brief`: Shows EUI-64 and link-local addresses.
   - `show ipv6 route`: Verifies routes.
   - Ping: Confirms connectivity.
5. **CCNA Focus**:
   - Exam topic 1.9: EUI-64 (1.9.f), link-local (1.9.c), global unicast (1.9.a).
   - Exam topic 1.8: Configure/verify EUI-64, link-local, static routes.
   - Practice: Calculate EUI-64, configure routes, troubleshoot pings.

These notes provide a complete foundation for the IPv6 Part 2 lab, preparing you for CCNA exam topics 1.8 and 1.9. Practice EUI-64 calculations, link-local configuration, static routes, and troubleshooting in Packet Tracer, using the analogies and retention tips to master IPv6 for life!