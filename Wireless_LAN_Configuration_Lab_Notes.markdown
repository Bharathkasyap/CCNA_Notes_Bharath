# Wireless LAN Configuration Lab Notes for CCNA Beginners

## Lab Overview

This lab focuses on configuring a wireless LAN (WLAN) using a Cisco Wireless LAN Controller (WLC) in Packet Tracer, covering CCNA exam topics 2.7 (AP and WLC management access connections), 2.8 (Configure WLAN components for client connectivity), 2.9 (Configure WLAN using WPA2 PSK via GUI), and 5.10 (Configure WLANs using WPA2 PSK via GUI). Building on the Wireless Fundamentals, Wireless Architectures, Wireless Security, and Wireless LAN Configuration Lecture, it provides hands-on practice for setting up a split-MAC WLAN with lightweight Access Points (APs), a WLC, and a switch, emphasizing GUI-based configuration. The lab configures dynamic interfaces and WLANs (“Internal” and “Guest”) with WPA2 PSK, and connects a client device (smartphone). Designed for beginners, the notes include comparisons to prior CCNA topics, troubleshooting tips for Packet Tracer quirks, practical examples, and real-world applications to prepare you for the CCNA exam and a career as a top-tier networking engineer.

**Lab Objectives**:
- **Understand Topology**: Review switch, WLC, APs, and client roles in a split-MAC WLAN.
- **Configure Dynamic Interfaces**: Set up “Internal” and “Guest” interfaces to map WLANs to VLANs.
- **Configure WLANs**: Create “Internal” and “Guest” WLANs with WPA2 PSK using the WLC GUI.
- **Connect Clients**: Associate a smartphone to the “Internal” SSID and verify connectivity.
- **Navigate WLC GUI**: Explore tabs (Monitor, Controller, WLANs) for configuration and troubleshooting.
- **Prepare for Exam**: Master topics 2.7, 2.8, 2.9, and 5.10, focusing on GUI-based WPA2 PSK setup.
- **Build Expertise**: Gain skills in WLC configuration, client troubleshooting, and WLAN deployment.

**Lab Context**:
- **Focus**: WLAN configuration in a split-MAC architecture using IEEE 802.11 (Wi-Fi) with Cisco 3504 WLC in Packet Tracer.
- **Topology**: One switch (VLANs 10, 100, 200), one WLC, two lightweight APs (AP1, AP2), one PC (GUI access), one smartphone (client).
- **Preconfiguration**: Switch configured with VLANs, SVIs, DHCP, and NTP; WLC initial setup completed (CLI wizard).
- **Tools**: Packet Tracer (Cisco 3504 WLC, APs, Catalyst switch), Cisco GUI for real-world reference.

**Analogy**: This lab is like setting up a town’s secure Wi-Fi network. The switch is the town’s infrastructure (roads/VLANs), the WLC is the control center (mayor’s office), APs are broadcasters (town criers), and WPA2 PSK is a password for residents (clients). The GUI is a control panel for easy management.

**Lifelong Retention Tip**: Visualize a smartphone joining the “Internal” SSID via AP1, tunneled to the WLC over CAPWAP. In Packet Tracer, configure a WLC with WPA2 PSK, verify with `show wlan summary`. Create flashcards for WLC tabs (Monitor, Controller, WLANs), interfaces (management, dynamic), and WPA2 settings. Practice 3 times daily for a week. For expertise, explore Cisco Catalyst 9800 WLC GUI demo online and test WPA2 PSK on your home router.

## Lab Topology

**Overview**:
- **Components**:
  - **Switch**: Provides VLANs, SVIs, DHCP, NTP, and connectivity (G1/0/1 to WLC, G1/0/2–3 to APs, G1/0/4 to PC).
  - **WLC**: Cisco 3504, manages APs, maps WLANs to VLANs, uses CAPWAP (UDP 5246/5247).
  - **APs**: Two lightweight APs (AP1, AP2) in local mode, broadcasting “Internal” (VLAN 100) and “Guest” (VLAN 200) SSIDs.
  - **PC1**: Accesses WLC GUI (HTTPS://172.16.1.10) on VLAN 10.
  - **Smartphone**: Client connecting to “Internal” SSID with WPA2 PSK.
- **Connections**:
  - **Switch to APs**: Access ports (G1/0/2–3, VLAN 10), PoE-enabled.
  - **Switch to WLC**: Trunk port (G1/0/1, VLANs 10, 100, 200, native VLAN 10).
  - **Switch to PC**: Access port (G1/0/4, VLAN 10).
  - **WLC to APs**: CAPWAP tunnels for control (UDP 5246) and data (UDP 5247).
- **IP Addressing**:
  - **VLAN 10 (Management)**: 172.16.1.0/24 (switch SVI: 172.16.1.1, WLC: 172.16.1.10, APs/PC via DHCP).
  - **VLAN 100 (Internal)**: 10.0.0.0/24 (switch SVI: 10.0.0.1, WLC: 10.0.0.10, clients via DHCP).
  - **VLAN 200 (Guest)**: 10.1.0.0/24 (switch SVI: 10.1.0.1, WLC: 10.1.0.10, clients via DHCP).
- **Traffic Flow (Split-MAC)**:
  - Client to gateway: Smartphone on “Internal” SSID sends traffic via AP1’s CAPWAP tunnel to WLC, mapped to VLAN 100, forwarded to switch SVI (10.0.0.1).
  - Client to client (e.g., Internal to Guest): Traffic tunnels to WLC, routes via switch (VLAN 100 to 200), returns via CAPWAP.
- **Additional Details**:
  - **Native VLAN**: VLAN 10 untagged on WLC and switch G1/0/1 for management traffic.
  - **DHCP Option 43**: Provides WLC IP (172.16.1.10) to APs; optional in same VLAN due to CAPWAP discovery broadcasts.
  - **Packet Tracer Limitation**: Smartphone may get VLAN 10 IP (172.16.1.x) instead of VLAN 100 (10.0.0.x); real hardware assigns correctly.
  - **Real-World Tip**: Use Cisco Catalyst 9200 switches and 9800 WLC for robust deployments; verify CAPWAP with `show capwap status`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced SSIDs, BSS; here, configures “Internal”/“Guest” SSIDs.
- **Wireless Architectures**: Detailed split-MAC, CAPWAP; here, implements with access ports and WLC trunk.
- **Wireless Security**: Introduced WPA2 PSK; here, configures via WLC GUI.
- **Wireless LAN Configuration Lecture**: Configured similar topology; here, uses Packet Tracer with simplified GUI.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover APs/WLC on VLAN 10; here, APs use CAPWAP.
- **Standard ACLs (Day 34)**: Filter client traffic post-WLAN setup; here, VLAN 10 for management.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment traffic.
- **IPv6 Parts 1–3**: Supports IPv6 DHCP; here, uses IPv4 but extensible.

**Comparison to Other CCNA Topics**:
- **VLANs**: Switch VLANs (10, 100, 200) align with WLAN-to-VLAN mapping.
- **DHCP**: Switch as DHCP server, like DHCP labs for wired clients.
- **Security (AAA)**: WLC GUI login (admin/Cisco123) aligns with AAA security.

**Analogy**: The topology is a town square: switch is the road network (VLANs), WLC is the mayor’s office, APs are criers broadcasting SSIDs, and clients are residents joining secure channels (CAPWAP, WPA2).

**Lifelong Retention Tip**: Draw topology with switch (VLANs 10, 100, 200), WLC (trunk), APs (access), and smartphone (Internal SSID). Flashcard: “Lab topology? Switch (VLANs, DHCP), WLC (CAPWAP), APs (access), client (WPA2).” Practice in Packet Tracer 3 times.

## Switch Configuration (Preconfigured)

**Overview**:
- **Purpose**: Provide VLANs, IP addressing, DHCP, and connectivity for APs, WLC, and clients.
- **Configuration** (review from transcript):
  ```plaintext
  Switch# show running-config
  ip dhcp excluded-address 172.16.1.1 172.16.1.10
  ip dhcp excluded-address 10.0.0.1 10.0.0.10
  ip dhcp excluded-address 10.1.0.1 10.1.0.10
  ip dhcp pool VLAN10
   network 172.16.1.0 255.255.255.0
   default-router 172.16.1.1
   option 43 ip 172.16.1.10
  ip dhcp pool VLAN100
   network 10.0.0.0 255.255.255.0
   default-router 10.0.0.1
  ip dhcp pool VLAN200
   network 10.1.0.0 255.255.255.0
   default-router 10.1.0.1
  vlan 10
   name Management
  vlan 100
   name Internal
  vlan 200
   name Guest
  interface GigabitEthernet1/0/1
   switchport mode trunk
   switchport trunk native vlan 10
   switchport trunk allowed vlan 10,100,200
  interface GigabitEthernet1/0/2
   switchport mode access
   switchport access vlan 10
   spanning-tree portfast
  interface GigabitEthernet1/0/3
   switchport mode access
   switchport access vlan 10
   spanning-tree portfast
  interface GigabitEthernet1/0/4
   switchport mode access
   switchport access vlan 10
   spanning-tree portfast
  interface vlan 10
   ip address 172.16.1.1 255.255.255.0
  interface vlan 100
   ip address 10.0.0.1 255.255.255.0
  interface vlan 200
   ip address 10.1.0.1 255.255.255.0
  ```
- **Key Points**:
  - **VLANs**: 10 (Management), 100 (Internal), 200 (Guest) for segmentation.
  - **Interfaces**:
    - G1/0/1: Trunk to WLC, native VLAN 10 (untagged management), allowed VLANs 10, 100, 200.
    - G1/0/2–4: Access ports in VLAN 10 for APs (G1/0/2–3) and PC (G1/0/4), PortFast enabled.
  - **SVIs**: Default gateways (172.16.1.1, 10.0.0.1, 10.1.0.1).
  - **DHCP**: Pools for VLAN 10 (APs/PC, Option 43), 100 (Internal clients), 200 (Guest clients); excluded addresses for static IPs.
  - **Native VLAN**: VLAN 10 untagged, must match WLC configuration.
- **Additional Details**:
  - **Option 43**: Optional in same VLAN; APs discover WLC via CAPWAP broadcasts.
  - **Packet Tracer Note**: Auto-added MAC-ADDRESS commands are cosmetic, not user-configured.
  - **Real-World Tip**: Use Cisco Catalyst 9200 for VLANs and DHCP; verify with `show vlan brief`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: VLANs map to SSIDs; here, VLANs 100/200 for Internal/Guest.
- **Wireless Architectures**: Lightweight APs use access ports; here, G1/0/2–3 in VLAN 10.
- **Wireless Security**: Supports WPA2 PSK; here, VLANs segment secured traffic.
- **Wireless LAN Configuration Lecture**: Similar switch config (LAG omitted); here, uses native VLAN 10.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP identify APs on G1/0/2–3; here, APs get DHCP IPs.
- **Standard ACLs (Day 34)**: Protect VLAN 10; here, sets up management VLAN.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment traffic.
- **IPv6 Parts 1–3**: Supports IPv6 DHCP; here, uses IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: Configures VLANs/SVIs, like VLAN labs.
- **DHCP**: Switch as DHCP server, like DHCP labs.
- **Security**: VLAN 10 isolates management, like switch security.

**Analogy**: Switch config is like laying town roads: VLANs are lanes, SVIs are intersections, DHCP assigns addresses, and native VLAN 10 is an unmarked main road for management.

**Lifelong Retention Tip**: In Packet Tracer, verify switch config with `show running-config`. Flashcard: “Switch config? VLANs (10, 100, 200), SVIs, DHCP (Option 43), trunk (native VLAN 10).” Practice 3 times.

## WLC Initial Setup (Preconfigured)

**Overview**:
- **Purpose**: Enable GUI access and AP management via CLI wizard (preconfigured in lab).
- **Settings** (from lecture, applied in Packet Tracer):
  - System name: `WLC1`.
  - Admin username/password: `admin`/`Cisco123`.
  - Management interface: IP `172.16.1.10`, Mask `255.255.255.0`, Gateway `172.16.1.1`, VLAN 10, DHCP server `172.16.1.1`.
  - Virtual interface: `1.1.1.1` (for DHCP relay).
  - Country code: Matches AP regulatory domain (e.g., US for -A).
  - 802.11b/a/g enabled, auto-RF for channel/power.
- **Additional Details**:
  - **Packet Tracer Limitation**: No console access; initial setup via GUI (preconfigured here).
  - **GUI Access**: HTTPS://172.16.1.10 from PC1 (VLAN 10, G1/0/4).
  - **Real-World Tip**: Use Cisco 9800 WLC; verify setup with `show running-config`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced WLC role; here, preconfigured for GUI access.
- **Wireless Architectures**: CAPWAP setup; here, management IP for CAPWAP.
- **Wireless Security**: Supports WPA2; here, skips RADIUS.
- **Wireless LAN Configuration Lecture**: Identical wizard setup; here, preconfigured.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover WLC; here, management IP 172.16.1.10.
- **Standard ACLs (Day 34)**: Protect management IP; here, VLAN 10 access.
- **Extended ACLs (Day 35)**: Restrict protocols; here, HTTPS for GUI.
- **IPv6 Parts 1–3**: Supports IPv6 management; here, IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: Management on VLAN 10, like switch SVIs.
- **Security (AAA)**: Admin login aligns with AAA security.
- **DHCP**: WLC uses switch DHCP, like DHCP labs.

**Analogy**: WLC setup is like appointing a mayor: assign name (WLC1), office address (172.16.1.10), and rules (WPA2 later) via a form (wizard).

**Lifelong Retention Tip**: In Packet Tracer, verify WLC IP with `show interface summary`. Flashcard: “WLC setup? Name, management IP, virtual IP, country.” Practice 3 times.

## WLC GUI Navigation (2.7)

**Overview**:
- **Purpose**: Access and explore WLC GUI for configuration and monitoring.
- **Steps**:
  - On PC1, Desktop > Web Browser.
  - Enter `https://172.16.1.10`, login with `admin`/`Cisco123`.
  - **Monitor Tab**:
    - Dashboard: Shows port status (Port 1 green), AP status (AP1, AP2 joined), client count (0 initially).
    - Statistics > AP Join: View AP join status, troubleshooting details (e.g., last unsuccessful attempt).
  - **Controller Tab**: Manage interfaces (dynamic interfaces configured here).
  - **WLANs Tab**: Create/edit WLANs (Internal, Guest).
- **Additional Details**:
  - **GUI Features**: Limited in Packet Tracer but mirrors Cisco 3504/9800 GUI for CCNA.
  - **Troubleshooting**: AP Join stats help diagnose CAPWAP failures (e.g., wrong country code).
  - **Real-World Tip**: Use Cisco DNA Center for centralized GUI management; verify with `show ap summary`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced WLC GUI; here, navigates for configuration.
- **Wireless Architectures**: CAPWAP monitoring; here, checks AP join status.
- **Wireless Security**: GUI for WPA2; here, accesses security settings.
- **Wireless LAN Configuration Lecture**: Similar GUI navigation; here, Packet Tracer-specific.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP for device discovery; here, GUI for AP status.
- **Standard ACLs (Day 34)**: Protect GUI access; here, HTTPS to 172.16.1.10.
- **Extended ACLs (Day 35)**: Restrict protocols; here, HTTPS only.
- **IPv6 Parts 1–3**: GUI supports IPv6; here, IPv4.

**Comparison to Other CCNA Topics**:
- **Security (AAA)**: GUI login aligns with AAA.
- **VLANs**: GUI access via VLAN 10, like management VLANs.

**Analogy**: WLC GUI is the mayor’s control panel: Monitor (overview), Controller (settings), WLANs (Wi-Fi setup).

**Lifelong Retention Tip**: In Packet Tracer, explore Monitor tab, check AP Join stats. Flashcard: “WLC GUI tabs? Monitor (status), Controller (interfaces), WLANs (SSIDs).” Practice 3 times.

## Dynamic Interface Configuration (2.8)

**Overview**:
- **Purpose**: Create logical interfaces to map WLANs to VLANs for client traffic.
- **GUI Configuration**:
  - **Internal Interface**:
    - Navigate: Controller > Interfaces > New.
    - Name: `Internal`, VLAN ID: `100`.
    - Port: `1`, IP: `10.0.0.10`, Mask: `255.255.255.0`, Gateway: `10.0.0.1`, DHCP: `10.0.0.1`.
    - Apply, confirm warning, save.
  - **Guest Interface**:
    - Navigate: Controller > Interfaces > New.
    - Name: `Guest`, VLAN ID: `200`.
    - Port: `1`, IP: `10.1.0.10`, Mask: `255.255.255.0`, Gateway: `10.1.0.1`, DHCP: `10.1.0.1`.
    - Apply, save.
- **Additional Details**:
  - **Role**: Maps WLANs to VLANs (Internal to VLAN 100, Guest to VLAN 200).
  - **Packet Tracer Limitation**: Limited interface options but sufficient for CCNA.
  - **Real-World Tip**: Use Cisco 9800 WLC; verify with `show interface summary`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced distribution system; here, dynamic interfaces map to VLANs.
- **Wireless Architectures**: Detailed WLC interfaces; here, configures dynamic interfaces.
- **Wireless Security**: Interfaces support WPA2; here, maps WLANs.
- **Wireless LAN Configuration Lecture**: Identical interface setup; here, Packet Tracer-specific.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP for WLC ports; here, logical interfaces.
- **Standard ACLs (Day 34)**: Protect interfaces; here, VLAN 10 management.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment traffic.
- **IPv6 Parts 1–3**: Interfaces support IPv6; here, IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: Dynamic interfaces like SVIs for VLAN mapping.
- **DHCP**: Interfaces use switch DHCP, like DHCP labs.

**Analogy**: Dynamic interfaces are like bridges connecting Wi-Fi zones (WLANs) to town roads (VLANs).

**Lifelong Retention Tip**: In Packet Tracer, create Internal/Guest interfaces. Flashcard: “Dynamic interfaces? Map WLANs to VLANs (Internal: VLAN 100, Guest: VLAN 200).” Practice 3 times.

## WLAN Configuration with WPA2 PSK (2.9, 5.10)

**Overview**:
- **Purpose**: Configure “Internal” and “Guest” WLANs with WPA2 PSK for client connectivity.
- **GUI Configuration**:
  - **Internal WLAN**:
    - Navigate: WLANs > Create New > Go.
    - Type: WLAN, Profile Name: `Internal`, SSID: `Internal`, ID: `1`.
    - **General Tab**: Enable WLAN, Interface: `Internal` (VLAN 100).
    - **Security Tab**:
      - Layer 2: `WPA+WPA2`, enable WPA2 policy, AES encryption.
      - Authentication: Select `PSK`, Pass Phrase: `Cisco123`.
    - **QoS Tab**: Silver (best effort, default; Packet Tracer limits changes).
    - **Advanced Tab**: Set max clients to `100`.
    - Apply, save.
  - **Guest WLAN**:
    - Navigate: WLANs > Create New > Go.
    - Type: WLAN, Profile Name: `Guest`, SSID: `Guest`, ID: `2`.
    - **General Tab**: Enable WLAN, Interface: `Guest` (VLAN 200).
    - **Security Tab**:
      - Layer 2: `WPA+WPA2`, enable WPA2 policy, AES encryption.
      - Authentication: Select `PSK`, Pass Phrase: `Cisco123`.
    - **QoS Tab**: Silver (default).
    - **Advanced Tab**: Leave defaults.
    - Apply, save.
- **Additional Details**:
  - **WPA2 PSK**: Uses CCMP (AES encryption, CBC-MAC for MIC), four-way handshake for key generation.
  - **Packet Tracer Limitation**: QoS tab uneditable; max clients adjustable but limited functionality.
  - **Real-World Tip**: Use WPA3 for modern deployments; WPA2 for CCNA and legacy support.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Configured SSIDs; here, maps to VLANs with WPA2.
- **Wireless Architectures**: Lightweight APs/WLC; here, configures WLANs via GUI.
- **Wireless Security**: Detailed WPA2 PSK; here, implements in GUI.
- **Wireless LAN Configuration Lecture**: Identical WLAN setup; here, Packet Tracer-specific.
- **CDP/LLDP Lecture/Lab**: APs discovered; here, WLANs enable clients.
- **Standard ACLs (Day 34)**: Filter post-WLAN traffic; here, WPA2 secures.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment.
- **IPv6 Parts 1–3**: Supports IPv6; here, IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: WLANs map to VLANs, like switch VLANs.
- **Security (AAA)**: WPA2 PSK like password security; no RADIUS.
- **QoS**: Silver aligns with QoS topics.

**Analogy**: WLAN config is like setting up secure meeting halls: name (SSID), password (WPA2 PSK), and road (VLAN) assignment via the mayor’s panel (GUI).

**Lifelong Retention Tip**: In Packet Tracer, configure WPA2 PSK for “Internal” SSID. Flashcard: “WPA2 PSK? GUI, WPA+WPA2, PSK, Cisco123.” Practice 3 times.

## Client Connection (2.8)

**Overview**:
- **Purpose**: Connect a smartphone to “Internal” SSID, verify DHCP and connectivity.
- **Configuration**:
  - Add smartphone in Packet Tracer (End Devices > Smartphone).
  - Navigate: Config > Wireless0.
  - SSID: `Internal`.
  - Authentication: `WPA2-PSK`, Pass Phrase: `Cisco123`.
  - IP Configuration: DHCP.
- **Verification**:
  - Smartphone associates with AP, gets IP (should be 10.0.0.x, VLAN 100).
  - Packet Tracer bug: May assign 172.16.1.x (VLAN 10); real hardware assigns correctly.
- **Additional Details**:
  - **CAPWAP**: Tunnels client traffic to WLC, maps to VLAN 100.
  - **DHCP**: Switch assigns IP from VLAN100 pool (10.0.0.0/24).
  - **Real-World Tip**: Use Cisco 9100 APs; verify clients with `show wlan clients`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Client association; here, configures smartphone.
- **Wireless Architectures**: CAPWAP for client traffic; here, tunnels to WLC.
- **Wireless Security**: WPA2 secures client; here, applies PSK.
- **Wireless LAN Configuration Lecture**: Similar client setup; here, Packet Tracer-specific.
- **CDP/LLDP Lecture/Lab**: APs support clients; here, verifies association.
- **Standard ACLs (Day 34)**: Filter client traffic; here, WPA2 secures.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment.
- **IPv6 Parts 1–3**: Supports IPv6 clients; here, IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: Client on VLAN 100, like wired clients.
- **DHCP**: Smartphone uses DHCP, like DHCP labs.
- **Security**: WPA2 PSK like password security.

**Analogy**: Client connection is like a resident joining a secure meeting hall (SSID) with a password (WPA2 PSK) and getting an address (DHCP).

**Lifelong Retention Tip**: In Packet Tracer, connect smartphone to “Internal” SSID. Flashcard: “Client config? SSID Internal, WPA2-PSK, Cisco123, DHCP.” Practice 3 times.

## Common Issues and Troubleshooting

- **APs Not Joining WLC**:
  - **Issue**: AP1/AP2 fail to form CAPWAP tunnel.
  - **Cause**: Wrong country code, no IP, firewall blocking UDP 5246/5247, or missing Option 43.
  - **Fix**: Check country (`show country`), AP IP (`show capwap ip config`), firewall, Option 43 (`show ip dhcp pool`).
- **Client Fails to Connect**:
  - **Issue**: Smartphone can’t join “Internal” SSID.
  - **Cause**: Wrong PSK, WLAN disabled, or incorrect interface mapping.
  - **Fix**: Verify PSK (`show wlan security`), WLAN status (`show wlan summary`), interface (`show wlan`).
- **Incorrect Client IP (Packet Tracer Bug)**:
  - **Issue**: Smartphone gets 172.16.1.x (VLAN 10) instead of 10.0.0.x (VLAN 100).
  - **Cause**: Packet Tracer limitation in WLAN-to-VLAN mapping.
  - **Fix**: Note bug; real hardware assigns correctly. Verify DHCP pool (`show ip dhcp pool`).
- **GUI Access Denied**:
  - **Issue**: Can’t access WLC GUI.
  - **Cause**: Wrong IP, HTTPS disabled, or PC not in VLAN 10.
  - **Fix**: Verify IP (`172.16.1.10`), HTTPS (`show management`), PC VLAN (`show vlan brief`).
- **WLAN Not Broadcasting**:
  - **Issue**: “Internal” SSID not visible.
  - **Cause**: WLAN disabled or radio off.
  - **Fix**: Check WLAN status (`show wlan summary`), radio (`show ap config general`).

**Analogy**: Troubleshooting is like fixing a town’s Wi-Fi: ensure criers (APs) connect to the mayor (WLC, CAPWAP), residents (clients) have correct addresses (DHCP), and the control panel (GUI) is accessible.

**Lifelong Retention Tip**: In Packet Tracer, simulate client IP bug, check `show wlan summary`. Checklist: “No client? Check PSK, WLAN status, VLAN. No AP? Check country, IP, UDP 5246/5247.” Practice 3 times.

## Practice Commands (CLI Verification)

- **Switch**:
  ```plaintext
  Switch# show running-config
  Switch# show vlan brief
  Switch# show ip interface brief
  Switch# show ip dhcp pool
  Switch# show ip dhcp binding
  ```
- **WLC**:
  ```plaintext
  WLC# show interface summary
  WLC# show wlan summary
  WLC# show ap summary
  WLC# show capwap status
  WLC# show wlan clients
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Topology: “Switch (VLANs, DHCP), WLC (GUI, CAPWAP), APs (access), client (WPA2).”
   - Interfaces: “Management (CAPWAP), Dynamic (VLANs).”
   - WPA2: “WPA+WPA2, PSK, AES, Cisco123.”
2. **Visualize Topology**:
   - Draw switch (VLANs 10, 100, 200), WLC (trunk), APs (access), smartphone (Internal SSID). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “WLC GUI tabs? Monitor, Controller, WLANs. WPA2 PSK? GUI, PSK, Cisco123.”
4. **Practice in Packet Tracer**:
   - Configure WLANs, connect smartphone, verify `show wlan clients`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Switch (VLANs, DHCP), WLC (WPA2 PSK), APs (Local mode).
   - Quiz: “WLAN-to-VLAN? Dynamic interface.”
6. **Troubleshooting Practice**:
   - Simulate client IP bug, check `show ip dhcp binding`. Checklist: “No IP? Check DHCP, VLAN mapping.”
7. **Teach Someone**:
   - Explain WPA2 PSK setup, dynamic interfaces, and client connection.
8. **Spaced Repetition**:
   - Use Anki for GUI tabs, interfaces, WPA2 settings. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Topology**:
   - Switch (VLANs 10, 100, 200, native VLAN 10), WLC (172.16.1.10), APs (access, CAPWAP), client (smartphone).
2. **Switch Configuration**:
   - VLANs, SVIs, DHCP (Option 43), trunk (G1/0/1, native VLAN 10), access ports (G1/0/2–4).
3. **WLC Setup**:
   - Preconfigured: management IP (172.16.1.10), virtual IP, admin/Cisco123.
4. **Dynamic Interfaces**:
   - Internal (VLAN 100, 10.0.0.10), Guest (VLAN 200, 10.1.0.10).
5. **WLAN Configuration**:
   - Internal/Guest WLANs, WPA2 PSK (Cisco123), mapped to VLANs 100/200.
6. **Client Connection**:
   - Smartphone joins Internal SSID, WPA2 PSK, DHCP (Packet Tracer bug: may get VLAN 10 IP).
7. **CCNA Focus**:
   - Topics 2.7 (GUI access), 2.8 (WLAN components), 2.9/5.10 (WPA2 PSK via GUI).

These notes provide a complete foundation for the Wireless LAN Configuration Lab, preparing you for CCNA exam topics 2.7, 2.8, 2.9, and 5.10. Practice configuring WPA2 PSK, troubleshooting client IPs, and exploring WLC GUI in Packet Tracer to master WLAN setup for life!