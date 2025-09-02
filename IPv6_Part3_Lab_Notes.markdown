# IPv6 Part 3 Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring IPv6 static routes and Stateless Address Auto-configuration (SLAAC) to enable connectivity between two PCs (PC1 and PC2) via a primary path (R1–R3) and a backup path (R1–R2–R3). It builds on the IPv6 Part 1 lecture/lab (address basics, dual-stack), Part 2 lecture/lab (EUI-64, link-local, static routes), and Part 3 lecture (NDP, SLAAC, static routing), aligning with CCNA exam topics 3.3 (static routing), 1.8 (SLAAC, verification), and 1.9 (link-local, global unicast). The lab uses Packet Tracer to demonstrate enabling IPv6 routing, SLAAC on PCs, static routes (recursive, fully specified, floating), and connectivity testing via pings and traceroutes.

**Lab Objectives**:
- **Enable IPv6 Routing**: Use `ipv6 unicast-routing` to allow routing and NDP router advertisements (RAs).
- **Configure SLAAC on PCs**: Use Packet Tracer’s Config tab to auto-configure PC1 and PC2 addresses and default gateways.
- **Configure IPv6 Static Routes**: Set up primary (R1–R3) and backup (R1–R2–R3) routes, including floating static routes.
- **Verify Connectivity**: Confirm PC1 can ping PC2 via primary and backup paths using `ping` and `tracert`.
- **Prepare for Exam**: Master static routing (3.3), SLAAC configuration (1.8), and address types (1.9: link-local, global unicast).

**Network Topology**:
- **Devices**:
  - **R1**:
    - G0/1: `2001:db8:0:1::1/64` (preconfigured, global unicast), link-local `FE80::34:56ff:fe78:90ab`.
    - G0/0: `2001:db8:0:13::1/64` (to R3), link-local `FE80::34:56ff:fe78:90aa`.
    - S0/0/0: Link-local only (`FE80::34:56ff:fe78:90a9`, via `ipv6 enable`).
  - **R2**:
    - S0/0/0: Link-local only (`FE80::34:56ff:fe78:90ac`, to R1).
    - S0/0/1: Link-local only (`FE80::34:56ff:fe78:90ad`, to R3).
  - **R3**:
    - G0/1: `2001:db8:0:3::1/64` (preconfigured, global unicast), link-local `FE80::34:56ff:fe78:90ae`.
    - G0/0: `2001:db8:0:13::2/64` (to R1), link-local `FE80::34:56ff:fe78:90af`.
    - S0/0/0: Link-local only (`FE80::34:56ff:fe78:90b0`, to R2).
  - **PC1**: `2001:db8:0:1::/64` (SLAAC, EUI-64, e.g., `2001:db8:0:1:34:56ff:fe78:1234`), default gateway `FE80::34:56ff:fe78:90ab` (R1 G0/1 link-local).
  - **PC2**: `2001:db8:0:3::/64` (SLAAC, EUI-64, e.g., `2001:db8:0:3:34:56ff:fe78:5678`), default gateway `FE80::34:56ff:fe78:90ae` (R3 G0/1 link-local).
- **Address Block**: `2001:db8::/32` (example range), subnets `2001:db8:0:1::/64`, `2001:db8:0:3::/64`, `2001:db8:0:13::/64`.
- **Routing**:
  - **Primary Path**: PC1 → R1 → R3 → PC2.
  - **Backup Path**: PC1 → R1 → R2 → R3 → PC2 (floating static routes, AD=5).
- **Connectivity**: PC1 pings PC2 (`2001:db8:0:3:34:56ff:fe78:5678`), verified via `ping` and `tracert`.

**Analogy**: The lab is like setting up a postal network with two delivery routes: a fast highway (R1–R3) and a slower backup road (R1–R2–R3). PCs get their addresses automatically (SLAAC) like self-registering mailboxes, routers act as post offices (`ipv6 unicast-routing`), and static routes are predefined delivery paths, with backups for emergencies.

**Lifelong Retention Tip**: Visualize PC1 pinging PC2 via R1–R3, switching to R1–R2–R3 when the cable is unplugged. Practice in Packet Tracer: configure `ipv6 unicast-routing`, SLAAC, static routes, and verify with `ping`, `tracert`, `show ipv6 route`. Create flashcards for commands (`ipv6 address autoconfig`, `ipv6 route`) and route types (fully specified, floating). Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Enable IPv6 Routing on R1, R2, R3
**Objective**: Enable IPv6 routing to allow packet forwarding and NDP router advertisements for SLAAC.

**Commands** (R1, R2, R3):
```plaintext
R1> enable
R1# configure terminal
R1(config)# ipv6 unicast-routing
```
```plaintext
R2> enable
R2# configure terminal
R2(config)# ipv6 unicast-routing
```
```plaintext
R3> enable
R3# configure terminal
R3(config)# ipv6 unicast-routing
```

- **Observations**:
  - `ipv6 unicast-routing`: Enables IPv6 routing (disabled by default, unlike IPv4). Required for routing between subnets and sending RA messages (ICMPv6 Type 134) for SLAAC.
  - Without this, routers drop IPv6 traffic and PCs cannot use SLAAC.
- **Note**: Preconfigured router addresses (`2001:db8:0:1::1/64`, etc.) assumed from lab setup.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Introduced `ipv6 unicast-routing` for inter-subnet pings.
- **Part 1 Lab**: Used `ipv6 unicast-routing` for PC1–PC2 pings, no routing config.
- **Part 2 Lecture**: Noted `ipv6 unicast-routing` for static routes.
- **Part 2 Lab**: Enabled `ipv6 unicast-routing` on R1, R2 for EUI-64 and static routes.
- **Part 3 Lecture**: Emphasized `ipv6 unicast-routing` as critical for routing and NDP.

**Comparison to Other CCNA Topics**:
- **IPv4 Routing**: Enabled by default, unlike IPv6.
- **OSPFv3**: Requires `ipv6 unicast-routing` for dynamic routing.
- **NDP**: RA messages (enabled by `ipv6 unicast-routing`) support SLAAC, unlike ARP.

**Analogy**: `ipv6 unicast-routing` is like activating a post office to forward mail (route packets) and announce zip codes (RA for SLAAC).

**Lifelong Retention Tip**: In Packet Tracer, enable `ipv6 unicast-routing` on all routers, verify RA with `debug ipv6 nd`. Flashcard: “Enable IPv6 routing? `ipv6 unicast-routing`.” Practice 3 times.

### Step 2: Configure SLAAC on PC1 and PC2
**Objective**: Use Packet Tracer’s Config tab to enable SLAAC for PC1 and PC2, auto-configuring addresses and default gateways.

**Configuration** (Packet Tracer GUI):
- **PC1**:
  - Config Tab → Gateway: Select “Automatic” to set default gateway as R1’s G0/1 link-local (`FE80::34:56ff:fe78:90ab`).
  - FastEthernet0: Enable SLAAC, auto-configures `2001:db8:0:1:34:56ff:fe78:1234/64` (prefix from R1’s RA, EUI-64 interface ID).
- **PC2**:
  - Config Tab → Gateway: Select “Automatic” to set default gateway as R3’s G0/1 link-local (`FE80::34:56ff:fe78:90ae`).
  - FastEthernet0: Enable SLAAC, auto-configures `2001:db8:0:3:34:56ff:fe78:5678/64` (prefix from R3’s RA, EUI-64).
- **CLI Verification (PC2)**:
  ```plaintext
  PC2> ipconfig
  FastEthernet0: 2001:db8:0:3:34:56ff:fe78:5678
  ```
  - Copy PC2’s address for ping testing.

**Observations**:
- SLAAC uses NDP RA messages (from `ipv6 unicast-routing`) to learn prefix (e.g., `2001:db8:0:1::/64`).
- PCs generate interface IDs via EUI-64 (MAC-based, e.g., `34:56ff:fe78:1234`).
- Default gateways are link-local (`FE80::`), learned via RA, unlike IPv4’s global addresses.
- Packet Tracer auto-enables SLAAC in Config tab; equivalent IOS command is `ipv6 address autoconfig`.

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Not covered, focused on manual addresses.
- **Part 1 Lab**: Configured PCs manually (`2001:db8::2/64`), no SLAAC.
- **Part 2 Lecture**: Explained EUI-64, not SLAAC; both use `/64` prefixes.
- **Part 2 Lab**: PCs used manual addresses, not SLAAC.
- **Part 3 Lecture**: Introduced SLAAC (`ipv6 address autoconfig`), applied here.

**Comparison to Other CCNA Topics**:
- **IPv4 DHCP**: Similar auto-configuration, but DHCP uses a server, SLAAC uses router RA.
- **OSPFv3**: May use SLAAC addresses for interfaces, like PCs here.
- **NDP**: SLAAC relies on RS/RA (Types 133/134), unlike ARP.

**Analogy**: SLAAC is like PCs requesting a zip code (prefix) from the post office (router) via RA, then stamping their own house number (EUI-64) to form an address.

**Lifelong Retention Tip**: In Packet Tracer, enable SLAAC on PC1, verify address (`2001:db8:0:1::/64`) and gateway (`FE80::`) with `ipconfig`. Flashcard: “SLAAC on PC? Config tab, Automatic.” Practice 3 times.

### Step 3: Configure IPv6 Static Routes
**Objective**: Configure static routes on R1, R2, R3 for PC1–PC2 connectivity via primary (R1–R3) and backup (R1–R2–R3) paths, using recursive and fully specified routes, including floating static routes.

**Commands (R1)**:
```plaintext
R1# configure terminal
R1(config)# ipv6 route 2001:db8:0:3::/64 g0/1 2001:db8:0:13::2  ! Primary, fully specified, network
R1(config)# ipv6 route 2001:db8:0:3::/64 s0/0/0 FE80::34:56ff:fe78:90ac 5  ! Backup, fully specified, floating
R1(config)# end
R1# show ipv6 route
S   2001:db8:0:3::/64 [1/0] via 2001:db8:0:13::2, GigabitEthernet0/1
R1# show running-config | include ipv6 route
ipv6 route 2001:db8:0:3::/64 GigabitEthernet0/1 2001:db8:0:13::2
ipv6 route 2001:db8:0:3::/64 Serial0/0/0 FE80::34:56ff:fe78:90ac 5
```

**Commands (R2)**:
```plaintext
R2# configure terminal
R2(config)# ipv6 route 2001:db8:0:1::/64 s0/0/0 FE80::34:56ff:fe78:90a9  ! To R1’s LAN, fully specified
R2(config)# ipv6 route 2001:db8:0:3::/64 s0/0/1 FE80::34:56ff:fe78:90b0  ! To R3’s LAN, fully specified
R2(config)# end
R2# show ipv6 route
S   2001:db8:0:1::/64 [1/0] via FE80::34:56ff:fe78:90a9, Serial0/0/0
S   2001:db8:0:3::/64 [1/0] via FE80::34:56ff:fe78:90b0, Serial0/0/1
```

**Commands (R3)**:
```plaintext
R3# configure terminal
R3(config)# ipv6 route 2001:db8:0:1::/64 g0/1 2001:db8:0:13::1  ! Primary, fully specified, network
R3(config)# ipv6 route 2001:db8:0:1::/64 s0/0/0 FE80::34:56ff:fe78:90ad 5  ! Backup, fully specified, floating
R3(config)# end
```

- **Details**:
  - **R1 Routes**:
    - Primary: To `2001:db8:0:3::/64` (PC2’s subnet) via G0/1, next-hop `2001:db8:0:13::2` (R3 G0/0, AD=1).
    - Backup: To `2001:db8:0:3::/64` via S0/0/0, next-hop `FE80::34:56ff:fe78:90ac` (R2 S0/0/0, AD=5, floating).
  - **R2 Routes**:
    - To `2001:db8:0:1::/64` (PC1’s subnet) via S0/0/0, next-hop `FE80::34:56ff:fe78:90a9` (R1 S0/0/0).
    - To `2001:db8:0:3::/64` (PC2’s subnet) via S0/0/1, next-hop `FE80::34:56ff:fe78:90b0` (R3 S0/0/0).
  - **R3 Routes**:
    - Primary: To `2001:db8:0:1::/64` (PC1’s subnet) via G0/1, next-hop `2001:db8:0:13::1` (R1 G0/0, AD=1).
    - Backup: To `2001:db8:0:1::/64` via S0/0/0, next-hop `FE80::34:56ff:fe78:90ad` (R2 S0/0/1, AD=5, floating).
  - **Notes**:
    - **Fully Specified Routes**: Used for Ethernet (R1 G0/1, R3 G0/1) and link-local next-hops (serial interfaces), as directly attached routes don’t work on Ethernet in IPv6.
    - **Floating Static Routes**: AD=5 ensures backup routes (R1–R2, R3–R2) activate only if primary fails (AD=1).
    - **Link-Local Next-Hops**: Require interface specification (e.g., `s0/0/0 FE80::...`), verified via `show ipv6 interface brief`.
    - **Verification**: `show ipv6 route` shows active routes (AD=1); `show running-config | include ipv6 route` shows all routes, including floating (AD=5).

**Comparison to Prior IPv6 Content**:
- **Part 1 Lecture**: Used `ipv6 unicast-routing`, no routing details.
- **Part 1 Lab**: Configured `ipv6 unicast-routing`, manual PC addresses, no routes.
- **Part 2 Lecture**: Noted link-local in static routes, introduced EUI-64.
- **Part 2 Lab**: Configured fully specified routes with link-local next-hops, no SLAAC or floating routes.
- **Part 3 Lecture**: Covered SLAAC (`ipv6 address autoconfig`), static route types (recursive, fully specified, floating), and NDP (RA for SLAAC).

**Comparison to Other CCNA Topics**:
- **IPv4 Static Routing**: Similar types (recursive, fully specified), but IPv6 disallows directly attached on Ethernet and uses link-local next-hops.
- **OSPFv3**: Dynamic routing, uses link-local like static routes here.
- **NDP**: RA messages enable SLAAC, unlike ARP.
- **FHRP (HSRP)**: Uses virtual IPs, not link-local like IPv6 routes.

**Analogy**: Static routes are like predefined postal paths: primary (R1–R3 highway) and backup (R1–R2–R3 backroad, AD=5). SLAAC is PCs self-registering addresses via postmaster announcements (RA).

**Lifelong Retention Tip**: In Packet Tracer, configure `ipv6 route 2001:db8:0:3::/64 g0/1 2001:db8:0:13::2` and floating route with AD=5, verify with `show ipv6 route`. Flashcard: “Floating route? Higher AD (e.g., 5).” Practice 3 times.

### Step 4: Verify Connectivity with Ping and Traceroute
**Objective**: Confirm PC1 can ping PC2 via primary (R1–R3) and backup (R1–R2–R3) paths, using `ping` and `tracert`.

**Commands (PC1)**:
```plaintext
PC1> ping 2001:db8:0:3:34:56ff:fe78:5678
Reply from 2001:db8:0:3:34:56ff:fe78:5678: time<1ms
PC1> tracert 2001:db8:0:3:34:56ff:fe78:5678
1  FE80::34:56ff:fe78:90ab  [R1 G0/1]
2  2001:db8:0:13::2         [R3 G0/0]
3  2001:db8:0:3:34:56ff:fe78:5678  [PC2]
```

**Test Backup Path**:
- **