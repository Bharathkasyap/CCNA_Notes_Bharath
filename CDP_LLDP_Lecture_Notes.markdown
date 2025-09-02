# CDP and LLDP Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers Cisco Discovery Protocol (CDP) and Link Layer Discovery Protocol (LLDP), addressing CCNA exam topic 2.3 ("Configure and verify Layer 2 discovery protocols CDP and LLDP"). These Layer 2 discovery protocols allow directly connected devices to share and learn information like hostname, device type, and IP address, operating at the Data Link layer without IP packets. The lecture explains their purpose, configuration, and verification, using a network of two routers (R1, R2) and two multilayer switches (SW1, SW2). It builds on prior topics like ACLs (Days 34–35), IPv6 (Parts 1–3), and others (e.g., OSPF, VLANs), contrasting their Layer 3 focus with CDP/LLDP’s Layer 2 operation.

**Lecture Objectives**:
- **Understand Layer 2 Discovery Protocols**: Learn how CDP and LLDP share device information (e.g., hostname, IP, platform) at Layer 2.
- **Configure CDP/LLDP**: Enable/disable globally and per interface, adjust timers (e.g., `cdp timer`, `lldp holdtime`).
- **Verify CDP/LLDP**: Use `show cdp neighbors`, `show lldp neighbors`, and related commands to check neighbors and timers.
- **Compare CDP vs. LLDP**: Understand CDP (Cisco proprietary, enabled by default) vs. LLDP (industry standard, disabled by default).
- **Prepare for Exam**: Master topic 2.3, including key MAC addresses, timers, and commands.

**Network Topology**:
- **Devices**:
  - **R1**: Connected to SW1 (G0/0) and R2 (G0/1).
  - **R2**: Connected to R1 (G0/0) and SW2 (G0/1).
  - **SW1**: Multilayer switch, connected to R1 (G0/0).
  - **SW2**: Multilayer switch, connected to R2 (G0/1).
- **Note**: Multilayer switches (SW1, SW2) have routing capabilities but no IP addresses assigned to interfaces in this lab (Layer 2 focus). Full connectivity assumed (likely via OSPF, as in Day 34).

**Analogy**: CDP and LLDP are like neighbors exchanging business cards (frames) over a direct phone line (Layer 2 link), sharing details like name (hostname), job (device type), and contact info (IP, port ID). CDP is a Cisco-only club, while LLDP is a universal standard. Cards are sent to a group address (multicast MAC) but only read by the immediate neighbor, not forwarded.

**Lifelong Retention Tip**: Visualize R1 sending CDP frames to SW1 via G0/0, listing “R1, router, 10.0.0.1.” Practice in Packet Tracer: enable CDP, run `show cdp neighbors`, note SW1 on G0/0. Create flashcards for MACs (`0100.0CCC.CCCC` for CDP, `0180.C200.000E` for LLDP), timers (CDP: 60s/180s, LLDP: 30s/120s), and commands (`cdp run`, `lldp transmit`). Practice 3 times daily for a week.

## Layer 2 Discovery Protocols Overview

**Purpose**:
- **Definition**: Protocols that enable directly connected devices to exchange and learn information (e.g., hostname, device type, IP address, port ID) at Layer 2 (Data Link layer).
- **Operation**: Send periodic frames to a multicast MAC address, processed and discarded by neighbors (not forwarded).
- **Information Shared**: Hostname, device type (router, switch), IP address (if assigned), platform, software version, native VLAN, duplex.
- **Layer 2 Nature**: Use Ethernet frames without IP packets, identified by destination MAC addresses.
- **Security Consideration**: Sharing device details can be a risk; often disabled in secure networks.

**Comparison to Prior Content**:
- **Standard ACLs (Day 34)**: Filter traffic by source IP (Layer 3); CDP/LLDP share device info at Layer 2, no filtering.
- **Extended ACLs (Day 35)**: Filter by protocol/port (Layer 3/4); CDP/LLDP operate at Layer 2, sharing IP but not using it for transport.
- **IPv6 Parts 1–3**: Focus on Layer 3 addressing (`2001:db8::/64`) and routing; CDP/LLDP are Layer 2, IP-independent.
- **Spanning Tree (Earlier Lessons)**: Uses multicast MAC (`0180.C200.0000` for STP); CDP/LLDP use different MACs for discovery.

**Comparison to Other CCNA Topics**:
- **OSPF**: Layer 3 protocol using IP packets; CDP/LLDP use Layer 2 frames.
- **VLANs/VTP**: Layer 2 protocols; CDP shares VTP info (Cisco proprietary), LLDP does not.
- **Day 10 (Ethernet)**: CDP/LLDP use Ethernet II (ARPA) encapsulation, no IP headers.

**Analogy**: Layer 2 discovery protocols are like a neighborhood watch meeting, where houses (devices) share details (hostname, type) via flyers (frames) sent only to next-door neighbors (direct links), not mailed further.

**Lifelong Retention Tip**: Draw R1–SW1 link with CDP frame showing “R1, router, G0/0.” Flashcard: “Layer 2 discovery shares? Hostname, device type, IP, port ID.” Practice `show cdp neighbors` in Packet Tracer 3 times.

## Cisco Discovery Protocol (CDP)

**Overview**:
- **Definition**: Cisco proprietary protocol for discovering directly connected Cisco devices (routers, switches, firewalls, IP phones).
- **Default State**: Enabled globally (`cdp run`) and on all interfaces (`cdp enable`).
- **Operation**:
  - Sends frames every 60 seconds (default) to multicast MAC `0100.0CCC.CCCC`.
  - Frames are processed and discarded by neighbors (not forwarded).
  - Holdtime: 180 seconds (default); neighbors removed if no frame received.
  - Uses CDP version 2 (default; version 1 obsolete, supports fewer features like no VLAN mismatch detection).

**Information Shared**:
- Hostname (e.g., R1).
- Device type (e.g., router, switch).
- IP address (if assigned, e.g., R2’s `10.0.0.2`).
- Platform (e.g., Cisco 2900 router, Catalyst 2960 switch).
- Port ID (e.g., G0/0).
- Software version (IOS version).
- VTP domain, native VLAN, duplex (useful for mismatch detection).

**Verification Commands**:
```plaintext
R1# show cdp
Global CDP information:
    Sending CDP packets every 60 seconds
    Sending a holdtime value of 180 seconds
    Sending CDP version 2 advertisements
R1# show cdp traffic
CDP counters:
    Total packets output: 105, Input: 112
    Version 2 advertisements: 105 (output), 112 (input)
R1# show cdp interface
GigabitEthernet0/0 is up, line protocol is up
    Encapsulation ARPA
    Sending CDP packets every 60 seconds
    Holdtime is 180 seconds
R1# show cdp neighbors
Capability Codes: R - Router, S - Switch, I - IGMP, B - Source Route Bridge
Device ID    Local Intrfce   Holdtme   Capability   Platform   Port ID
SW1          Gig 0/0         175       R S I        2960       Gig 0/0
R2           Gig 0/1         165       R B          2900       Gig 0/0
R1# show cdp neighbors detail
Device ID: SW1
  Platform: Catalyst 2960
  Interface: GigabitEthernet0/0, Port ID (outgoing port): GigabitEthernet0/0
  Holdtime: 175 sec
  Version: Cisco IOS Software, C2960 Software
  VTP Management Domain: ''
  Native VLAN: 1
  Duplex: Full
Device ID: R2
  IP Address: 10.0.0.2
R1# show cdp entry R2
[Same output as detail, but for R2 only]
```

**Configuration Commands**:
```plaintext
R1(config)# cdp run                 ! Enable CDP globally (default)
R1(config)# no cdp run              ! Disable CDP globally
R1(config)# interface g0/0
R1(config-if)# cdp enable           ! Enable CDP on interface (default)
R1(config-if)# no cdp enable        ! Disable CDP on interface
R1(config)# cdp timer 30            ! Set message interval to 30 seconds
R1(config)# cdp holdtime 90         ! Set holdtime to 90 seconds
R1(config)# cdp advertise-v2        ! Enable version 2 (default)
R1(config)# no cdp advertise-v2     ! Revert to version 1
```

**Wireshark Capture**:
- **Destination MAC**: `0100.0CCC.CCCC` (shared with VTP, DTP, PagP, UDLD).
- **Fields**: Device ID (R1), version (2), TTL (180s), capabilities (Router, Source Route Bridge), port ID (G0/0).
- **No IP**: Ethernet II frame, no IP packet.

**Comparison to Prior Content**:
- **Standard ACLs (Day 34)**: Filter traffic by source IP; CDP shares IP but operates at Layer 2.
- **Extended ACLs (Day 35)**: Block protocols (e.g., `deny cdp` not shown, but possible); CDP sends device info.
- **IPv6 Parts 1–3**: Use IP packets; CDP uses Layer 2 frames, no IP required.
- **Spanning Tree**: STP uses `0180.C200.0000`; CDP uses `0100.0CCC.CCCC`.

**Comparison to Other CCNA Topics**:
- **VTP**: CDP shares VTP domain info (Cisco proprietary); LLDP does not.
- **OSPF**: Uses IP multicast (`224.0.0.5`); CDP uses Layer 2 multicast.
- **Ethernet (Day 10)**: CDP uses Ethernet II (ARPA) encapsulation.

**Analogy**: CDP is like a Cisco-only newsletter sent every 60 seconds to neighbors’ mailboxes (`0100.0CCC.CCCC`), sharing device details. Neighbors read and discard it, updating their address book (neighbor table).

**Lifelong Retention Tip**: In Packet Tracer, run `show cdp neighbors`, note SW1’s port ID (G0/0). Flashcard: “CDP MAC? `0100.0CCC.CCCC`. Timers? 60s/180s.” Practice 3 times.

## Link Layer Discovery Protocol (LLDP)

**Overview**:
- **Definition**: Industry-standard protocol (IEEE 802.1AB) for discovering devices from multiple vendors (e.g., Cisco, Juniper, Palo Alto).
- **Default State**: Disabled globally and on interfaces on Cisco devices.
- **Operation**:
  - Sends frames every 30 seconds (default) to multicast MAC `0180.C200.000E`.
  - Frames processed and discarded by neighbors (not forwarded).
  - Holdtime: 120 seconds (default); neighbors removed if no frame received.
  - Reinitialization delay: 2 seconds (delays LLDP start after enabling, prevents flapping).

**Information Shared**:
- Hostname (e.g., SW1).
- Device type (e.g., Bridge for switch, Router).
- IP address (if assigned, e.g., R2’s `10.0.0.2`).
- Platform, software version.
- System capabilities (supported: Bridge, Router) vs. enabled capabilities (active, e.g., Router if `ip routing` enabled).
- Port ID (e.g., G0/0).
- **Note**: No VTP info (unlike CDP, as VTP is Cisco proprietary).

**Configuration Commands**:
```plaintext
R1(config)# lldp run               ! Enable LLDP globally
R1(config)# no lldp run            ! Disable LLDP globally
R1(config)# interface g0/0
R1(config-if)# lldp transmit       ! Enable LLDP sending
R1(config-if)# lldp receive        ! Enable LLDP receiving
R1(config-if)# no lldp transmit    ! Disable LLDP sending
R1(config-if)# no lldp receive     ! Disable LLDP receiving
R1(config)# lldp timer 15          ! Set message interval to 15 seconds
R1(config)# lldp holdtime 60       ! Set holdtime to 60 seconds
R1(config)# lldp reinit 5          ! Set reinitialization delay to 5 seconds
```

**Verification Commands**:
```plaintext
R1# show lldp
Global LLDP information:
    LLDP enabled
    LLDP timer: 30 seconds
    LLDP holdtime: 120 seconds
    LLDP reinitialization delay: 2 seconds
R1# show lldp traffic
LLDP counters:
    Total frames out: 4, in: 3
R1# show lldp interface
GigabitEthernet0/0:
    Tx: Enabled
    Rx: Enabled
    Tx state: IDLE
    Rx state: WAIT FOR FRAME
R1# show lldp neighbors
Capability Codes: B - Bridge, R - Router
Device ID    Local Intrfce   Holdtme   Capability   Port ID
SW1          Gig 0/0         120                  Gig 0/0
R2           Gig 0/1         120       R          Gig 0/0
R1# show lldp neighbors detail
Device ID: SW1
  System Capabilities: B, R
  Enabled Capabilities: Not advertised
  Platform: Catalyst 2960
  Interface: GigabitEthernet0/0, Port ID: GigabitEthernet0/0
  Time remaining: 115 seconds
  Version: Cisco IOS Software, C2960 Software
Device ID: R2
  System Capabilities: R
  Enabled Capabilities: R
  IP Address: 10.0.0.2
R1# show lldp entry SW1
[Same output as detail, but for SW1 only]
```

**Wireshark Capture**:
- **Destination MAC**: `0180.C200.000E`.
- **Fields**: System name (SW1), TTL (120s), capabilities (Bridge, Router; enabled: Router), port ID (G0/0).
- **No IP**: Ethernet II frame, no IP packet.

**Comparison to CDP**:
- **Vendor**: CDP (Cisco only); LLDP (multi-vendor).
- **Default**: CDP enabled; LLDP disabled.
- **Timers**: CDP (60s/180s); LLDP (30s/120s, plus 2s reinit).
- **Interface Config**: CDP (`cdp enable` for Tx/Rx); LLDP (`lldp transmit`, `lldp receive`).
- **Info**: CDP shares VTP, native VLAN, duplex; LLDP shares system/enabled capabilities, no VTP.
- **MAC**: CDP (`0100.0CCC.CCCC`); LLDP (`0180.C200.000E`).

**Comparison to Prior Content**:
- **Standard ACLs (Day 34)**: Block traffic (e.g., `172.16.2.0/24`); CDP/LLDP share device info, no filtering.
- **Extended ACLs (Day 35)**: Block DNS/HTTP; CDP/LLDP could be blocked with `deny cdp` or `deny lldp` (not shown).
- **IPv6 Parts 1–3**: Layer 3 focus; CDP/LLDP are Layer 2, sharing IP but not using it.
- **Spanning Tree**: LLDP uses `Bridge` term, like STP; CDP uses `Switch`.

**Comparison to Other CCNA Topics**:
- **VTP**: CDP shares VTP info; LLDP does not (non-Cisco).
- **OSPF**: Layer 3; CDP/LLDP are Layer 2, no routing.
- **Ethernet**: Both use Ethernet II (ARPA), seen in `show cdp interface`.

**Analogy**: LLDP is like an international newsletter sent every 30 seconds to neighbors’ mailboxes (`0180.C200.000E`), sharing device details. Unlike CDP’s Cisco-only club, LLDP welcomes all vendors.

**Lifelong Retention Tip**: In Packet Tracer, enable LLDP (`lldp run`, `lldp transmit`, `lldp receive`), run `show lldp neighbors`. Flashcard: “LLDP MAC? `0180.C200.000E`. Timers? 30s/120s/2s.” Practice 3 times.

## Quiz Answers and Explanations

1. **Which commands show configured CDP timers? (Select two)**
   - **Answer**: A (`show cdp`), C (`show cdp interface`).
   - **Explanation**: Both display message timer (60s) and holdtime (180s). `show cdp neighbors` shows current holdtime countdown, not configured value. `show cdp traffic` shows packet counts, not timers.

2. **Which commands represent the default CDP state? (Select two)**
   - **Answer**: C (`cdp enable`), D (`cdp timer 60`).
   - **Explanation**: `cdp enable` is default on interfaces; `cdp timer 60` is default message interval. `no cdp run` disables CDP (not default). `cdp holdtime 120` is incorrect (default is 180s).

3. **What is in the ‘system capabilities’ field for SW1 (multilayer switch) in `show lldp entry SW1`?**
   - **Answer**: B (System Capabilities: B, R).
   - **Explanation**: B (Bridge, i.e., switch), R (Router) for multilayer switch. LLDP uses B, not S (CDP’s switch code).

4. **Which statements about LLDP are true? (Select two)**
   - **Answer**: B (Separate Tx/Rx configuration), F (Learn OS version).
   - **Explanation**:
     - B: LLDP requires `lldp transmit` and `lldp receive` per interface.
     - F: `show lldp neighbors detail` shows IOS version.
     - A: LLDP is industry standard, not Cisco proprietary.
     - C: Default timer is 30s, not 60s.
     - D: LLDP doesn’t share OSPF settings.
     - E: LLDP doesn’t share VTP (Cisco proprietary).

5. **Which interface on R2 is SW2 connected to?**
   - **Answer**: G0/1.
   - **Explanation**: `show cdp neighbors` lists SW2 under local interface G0/1 on R2.

**Bonus Question (Boson ExSim)**: Likely tests CDP/LLDP configuration (e.g., enable LLDP), verification (`show cdp neighbors`), or differences (e.g., CDP vs. LLDP timers).

**Analogy**: The quiz is like testing a neighbor’s address book: ensuring it lists correct contacts (neighbors), timers (CDP/LLDP intervals), and device types (B, R).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “CDP timers? `show cdp`, `show cdp interface`”). Practice `show cdp neighbors` and `show lldp neighbors` in Packet Tracer, verify SW1 on G0/0. Repeat quiz 3 times.

## Common Issues and Troubleshooting

- **No CDP Neighbors**:
  - **Issue**: `show cdp neighbors` shows no output.
  - **Cause**: CDP disabled globally (`no cdp run`) or on interface (`no cdp enable`).
  - **Fix**: Enable with `cdp run`, `cdp enable` on G0/0, verify with `show cdp`.
- **No LLDP Neighbors**:
  - **Issue**: `show lldp neighbors` shows no output.
  - **Cause**: LLDP disabled (default), or only `lldp transmit` enabled (no `lldp receive`).
  - **Fix**: Enable `lldp run`, `lldp transmit`, `lldp receive` on G0/0, verify with `show lldp interface`.
- **Wrong Timers**:
  - **Issue**: `show cdp` shows 30s timer, not 60s.
  - **Cause**: Misconfigured with `cdp timer 30`.
  - **Fix**: Reset with `cdp timer 60`, check `show cdp`.
- **No IP Address in Output**:
  - **Issue**: `show cdp neighbors detail` shows no IP for SW1.
  - **Cause**: SW1 (switch) has no IP assigned (normal for Layer 2 interfaces).
  - **Fix**: Expected behavior; assign IP to VLAN interface if needed (e.g., `interface vlan 1`, `ip address 192.168.1.2 255.255.255.0`).

**Analogy**: Troubleshooting CDP/LLDP is like fixing a neighborhood newsletter: ensure it’s sent (enabled), reaches neighbors (Tx/Rx), and lists correct details (hostname, port ID).

**Lifelong Retention Tip**: Simulate no CDP neighbors in Packet Tracer, check `show cdp`, enable `cdp run`. Checklist: “No neighbors? Check global/interface enable, Tx/Rx for LLDP.” Practice 3 times.

## Practice Commands

- **CDP Configuration**:
  ```plaintext
  R1(config)# cdp run
  R1(config)# interface g0/0
  R1(config-if)# cdp enable
  R1(config)# cdp timer 30
  R1(config)# cdp holdtime 90
  ```
- **LLDP Configuration**:
  ```plaintext
  R1(config)# lldp run
  R1(config)# interface g0/0
  R1(config-if)# lldp transmit
  R1(config-if)# lldp receive
  R1(config)# lldp timer 15
  R1(config)# lldp holdtime 60
  R1(config)# lldp reinit 5
  ```
- **Verify**:
  ```plaintext
  show cdp
  show cdp traffic
  show cdp interface
  show cdp neighbors
  show cdp neighbors detail
  show cdp entry R2
  show lldp
  show lldp traffic
  show lldp interface
  show lldp neighbors
  show lldp neighbors detail
  show lldp entry SW1
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - CDP: “Cisco, enabled, 60s/180s, `0100.0CCC.CCCC`.”
   - LLDP: “Industry standard, disabled, 30s/120s/2s, `0180.C200.000E`.”
   - Capabilities: “CDP: R=Router, S=Switch; LLDP: R=Router, B=Bridge.”
2. **Visualize Topology**:
   - Draw R1–SW1 (G0/0–G0/0), R1–R2 (G0/1–G0/0), label CDP/LLDP frames. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “CDP default? Enabled, 60s/180s. LLDP default? Disabled, 30s/120s. CDP MAC? `0100.0CCC.CCCC`.”
4. **Practice in Packet Tracer**:
   - Enable CDP/LLDP, run `show cdp neighbors`, `show lldp neighbors`, verify SW1 on G0/0. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: CDP (Cisco, enabled, VTP) vs. LLDP (multi-vendor, disabled, no VTP).
   - Quiz: “CDP timers? `show cdp`, `show cdp interface`.”
6. **Troubleshooting Practice**:
   - Simulate no LLDP neighbors, check `show lldp interface`, enable `lldp receive`. Checklist: “No neighbors? Check enable, Tx/Rx, timers.”
7. **Teach Someone**:
   - Explain CDP vs. LLDP, multicast MACs, and neighbor table output.
8. **Spaced Repetition**:
   - Use Anki for commands, MACs, and timers. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **CDP Overview**:
   - Cisco proprietary, enabled by default, uses `0100.0CCC.CCCC`, 60s/180s timers.
   - Shares hostname, device type (R, S), IP, platform, VTP, VLAN, duplex.
2. **LLDP Overview**:
   - Industry standard (IEEE 802.1AB), disabled by default, uses `0180.C200.000E`, 30s/120s/2s timers.
   - Shares hostname, device type (R, B), IP, platform, no VTP.
3. **Configuration**:
   - CDP: `cdp run`, `cdp enable`, `cdp timer`, `cdp holdtime`.
   - LLDP: `lldp run`, `lldp transmit`, `lldp receive`, `lldp timer`, `lldp holdtime`, `lldp reinit`.
4. **Verification**:
   - `show cdp/lldp`, `show cdp/lldp traffic`, `show cdp/lldp interface`, `show cdp/lldp neighbors`, `show cdp/lldp neighbors detail`, `show cdp/lldp entry`.
5. **CCNA Focus**:
   - Topic 2.3: Configure/verify CDP/LLDP, know MACs, timers, and capability codes (R, S, B).

These notes provide a complete foundation for the CDP and LLDP lecture, preparing you for CCNA exam topic 2.3. Practice configurations, troubleshoot neighbor issues, and use analogies/retention tips to master CDP and LLDP for life!