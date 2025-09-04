# Wireless LAN Configuration Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture focuses on configuring a wireless LAN (WLAN) using a Cisco Wireless LAN Controller (WLC) and Access Points (APs), covering CCNA exam topics 2.7 (AP and WLC management access connections), 2.8 (Configure WLAN components for client connectivity), 2.9 (Configure WLAN using WPA2 PSK via GUI), and 5.10 (Configure WLANs using WPA2 PSK via GUI). Building on the Wireless Fundamentals, Wireless Architectures, and Wireless Security lectures, it provides hands-on guidance for setting up a split-MAC WLAN with lightweight APs, a WLC, and a switch. The notes emphasize GUI-based configuration (per exam requirements), switch setup, WLC interfaces, and WPA2 PSK security. Designed for beginners, the content includes comparisons to prior CCNA topics (e.g., Standard ACLs Day 34, Extended ACLs Day 35, CDP/LLDP lecture/lab, IPv6 Parts 1–3), practical configurations, troubleshooting tips, and real-world applications to prepare you for the CCNA exam and a career as a top-tier networking engineer.

**Lecture Objectives**:
- **Understand Network Topology**: Learn the roles of switches, WLCs, APs, and clients in a split-MAC WLAN.
- **Configure Switch**: Set up VLANs, SVIs, DHCP, and Link Aggregation Groups (LAGs) for WLAN connectivity.
- **Set Up WLC**: Perform initial CLI wizard setup and access the GUI for management.
- **Configure WLC Interfaces**: Create management, virtual, and dynamic interfaces to map WLANs to VLANs.
- **Configure WLANs**: Set up Internal and Guest WLANs with WPA2 PSK using the WLC GUI.
- **Explore WLC Features**: Understand QoS, ACLs, and AP modes for advanced management.
- **Prepare for Exam**: Master topics 2.7, 2.8, 2.9, and 5.10, focusing on GUI-based WPA2 PSK configuration.
- **Build Networking Expertise**: Gain practical skills in WLC configuration, client connectivity, and troubleshooting.

**Network Context**:
- **Focus**: WLAN configuration using IEEE 802.11 (Wi-Fi) in a split-MAC architecture with lightweight APs and a WLC.
- **Topology**: One switch (VLANs 10, 100, 200), one WLC, two lightweight APs, clients (e.g., laptops, phones).
- **Preconfiguration**: Assumes Layer 2/3 connectivity (e.g., VLANs, EIGRP/OSPF), PoE-enabled switch ports, and DHCP/NTP services.
- **Tools**: Cisco Catalyst 9800 WLC, Cisco APs (e.g., 9100 series), Cisco switch (e.g., Catalyst 9200), Packet Tracer for simulation.

**Analogy**: Configuring a WLAN is like setting up a town’s communication network. The switch is the town’s roads (VLANs), the WLC is the mayor’s office (central management), APs are town criers broadcasting signals (SSIDs), and WPA2 PSK is a secure password for trusted residents (clients). The GUI is a user-friendly control panel for the mayor.

**Lifelong Retention Tip**: Visualize a laptop joining “Internal” SSID (VLAN 100) via a lightweight AP, tunneled to a WLC over CAPWAP. In Packet Tracer, configure a WLC with WPA2 PSK, verify client connectivity with `show wlan summary`. Create flashcards for WLC interfaces (management, virtual, dynamic), QoS levels (Platinum, Gold, Silver, Bronze), and LAG setup. Practice 3 times daily for a week. For expertise, explore Cisco Catalyst 9800 WLC GUI demo online and test WPA2 PSK on your home Wi-Fi.

## Network Topology

**Overview**:
- **Components**:
  - **Switch**: Central connectivity hub, provides PoE, VLANs, DHCP, and NTP services.
  - **WLC**: Manages lightweight APs, maps WLANs to VLANs, handles CAPWAP tunnels.
  - **APs**: Two lightweight APs in local mode, broadcasting “Internal” (VLAN 100) and “Guest” (VLAN 200) SSIDs.
  - **Clients**: Devices (e.g., laptops, phones) connecting to SSIDs.
- **Connections**:
  - **Switch to APs**: Access ports (VLAN 10, management), PoE-enabled for power and data.
  - **Switch to WLC**: Trunk port with LAG (static EtherChannel, VLANs 10, 100, 200).
  - **WLC to APs**: CAPWAP tunnels (control: UDP 5246, data: UDP 5247) for management and client traffic.
- **IP Addressing**:
  - **VLAN 10 (Management)**: 192.168.1.0/24 (switch SVI: 192.168.1.1, WLC: 192.168.1.100, APs via DHCP).
  - **VLAN 100 (Internal)**: 10.0.0.0/24 (switch SVI: 10.0.0.1, WLC: 10.0.0.100, clients via DHCP).
  - **VLAN 200 (Guest)**: 10.1.0.0/24 (switch SVI: 10.1.0.1, WLC: 10.1.0.100, clients via DHCP).
- **Traffic Flow (Split-MAC)**:
  - Client to default gateway: Traffic tunnels via CAPWAP to WLC, maps to VLAN (e.g., VLAN 100), forwards to switch SVI.
  - Client to client (different SSIDs): Traffic tunnels to WLC, routes via switch (e.g., VLAN 100 to VLAN 200), returns via CAPWAP.
- **Additional Details**:
  - **PoE**: 802.3af/at provides power to APs (e.g., 15.4W for 802.3af), eliminating power cables.
  - **LAG**: Static EtherChannel (no PAgP/LACP) for WLC-switch redundancy and throughput.
  - **DHCP Option 43**: Informs APs of WLC IP (192.168.1.100); optional in same VLAN due to CAPWAP discovery broadcasts.
  - **Real-World Tip**: Use Cisco Catalyst 9200 switches for PoE and LAG; verify AP power with `show power inline`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced SSIDs, BSS; here, configures SSIDs (“Internal,” “Guest”) on WLC.
- **Wireless Architectures**: Detailed split-MAC (lightweight APs, CAPWAP); here, implements with access ports and WLC trunk.
- **Wireless Security**: Introduced WPA2 PSK; here, configures it via WLC GUI.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover APs/WLC on VLAN 10; here, APs use CAPWAP for WLC discovery.
- **Standard ACLs (Day 34)**: Filter client traffic post-WLAN setup; here, CPU ACLs protect WLC management.
- **Extended ACLs (Day 35)**: Restrict protocols; here, WLANs map to VLANs for traffic segmentation.
- **IPv6 Parts 1–3**: Clients get IPv6 via DHCP post-WLAN setup; here, focuses on IPv4 but supports IPv6.

**Comparison to Other CCNA Topics**:
- **VLANs**: Switch VLANs (10, 100, 200) align with WLAN-to-VLAN mapping (Internal to VLAN 100).
- **EtherChannel**: LAG is static EtherChannel, like prior labs (mode on).
- **DHCP**: Switch acts as DHCP server, like DHCP labs for wired clients.
- **Security (AAA)**: CPU ACLs align with AAA access control for management.

**Analogy**: The topology is a town square: switch is the road network (VLANs), WLC is the mayor’s office coordinating criers (APs), and clients are residents joining via secure channels (CAPWAP, WPA2).

**Lifelong Retention Tip**: Draw topology with switch (VLAN 10, 100, 200), WLC (LAG), APs (access ports), and CAPWAP tunnels. Flashcard: “WLAN topology? Switch (VLANs, PoE), WLC (LAG, CAPWAP), APs (access, DHCP).” Practice in Packet Tracer 3 times.

## Switch Configuration

**Overview**:
- **Purpose**: Provide VLANs, IP addressing, DHCP, NTP, and connectivity for APs and WLC.
- **Configuration**:
  ```plaintext
  Switch# configure terminal
  Switch(config)# vlan 10
  Switch(config-vlan)# name Management
  Switch(config-vlan)# exit
  Switch(config)# vlan 100
  Switch(config-vlan)# name Internal
  Switch(config-vlan)# exit
  Switch(config)# vlan 200
  Switch(config-vlan)# name Guest
  Switch(config-vlan)# exit
  Switch(config)# interface range FastEthernet0/6 - 8
  Switch(config-if-range)# switchport mode access
  Switch(config-if-range)# switchport access vlan 10
  Switch(config-if-range)# spanning-tree portfast
  Switch(config-if-range)# exit
  Switch(config)# interface range FastEthernet0/1 - 2
  Switch(config-if-range)# channel-group 1 mode on
  Switch(config-if-range)# exit
  Switch(config)# interface Port-channel1
  Switch(config-if)# switchport mode trunk
  Switch(config-if)# switchport trunk allowed vlan 10,100,200
  Switch(config-if)# exit
  Switch(config)# interface vlan 10
  Switch(config-if)# ip address 192.168.1.1 255.255.255.0
  Switch(config-if)# exit
  Switch(config)# interface vlan 100
  Switch(config-if)# ip address 10.0.0.1 255.255.255.0
  Switch(config-if)# exit
  Switch(config)# interface vlan 200
  Switch(config-if)# ip address 10.1.0.1 255.255.255.0
  Switch(config-if)# exit
  Switch(config)# ip dhcp pool VLAN10
  Switch(config-dhcp)# network 192.168.1.0 255.255.255.0
  Switch(config-dhcp)# default-router 192.168.1.1
  Switch(config-dhcp)# option 43 ip 192.168.1.100
  Switch(config-dhcp)# exit
  Switch(config)# ip dhcp pool VLAN100
  Switch(config-dhcp)# network 10.0.0.0 255.255.255.0
  Switch(config-dhcp)# default-router 10.0.0.1
  Switch(config-dhcp)# exit
  Switch(config)# ip dhcp pool VLAN200
  Switch(config-dhcp)# network 10.1.0.0 255.255.255.0
  Switch(config-dhcp)# default-router 10.1.0.1
  Switch(config-dhcp)# exit
  Switch(config)# ntp master
  ```
- **Key Points**:
  - **VLANs**: 10 (Management), 100 (Internal), 200 (Guest) for segmentation.
  - **Access Ports**: Fa0/6 (PC for GUI access), Fa0/7–8 (APs), all in VLAN 10.
  - **LAG**: Fa0/1–2 form static EtherChannel (mode on) to WLC, trunking VLANs 10, 100, 200.
  - **SVIs**: Default gateways for each VLAN (e.g., 192.168.1.1 for VLAN 10).
  - **DHCP**: Pools for VLANs 10 (APs, Option 43 for WLC discovery), 100 (Internal clients), 200 (Guest clients).
  - **NTP**: Switch as NTP server for WLC time synchronization.
- **Additional Details**:
  - **PoE**: Enabled on Fa0/6–8 (verify with `show power inline`).
  - **Option 43**: Optional in same VLAN (APs use CAPWAP discovery broadcasts); critical for cross-subnet WLC discovery.
  - **Real-World Tip**: Use Cisco Catalyst 9200 for PoE and LAG; verify VLANs with `show vlan brief`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: VLANs map to SSIDs; here, configures VLANs 100/200 for Internal/Guest SSIDs.
- **Wireless Architectures**: Lightweight APs use access ports; here, Fa0/7–8 in VLAN 10 for CAPWAP.
- **Wireless Security**: WPA2 PSK configured later; switch VLANs segment secured traffic.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP identify APs on Fa0/7–8; here, APs get IPs via DHCP.
- **Standard ACLs (Day 34)**: Protect VLAN 10 (management) with ACLs; here, sets up VLAN 10.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment traffic before ACLs.
- **IPv6 Parts 1–3**: Supports IPv6 DHCP; here, uses IPv4 but extensible to IPv6.

**Comparison to Other CCNA Topics**:
- **VLANs**: Configures VLANs/SVIs, like prior VLAN labs.
- **EtherChannel**: LAG is static EtherChannel, like mode on in labs.
- **DHCP**: Switch as DHCP server, like DHCP labs for wired clients.
- **NTP**: Switch as NTP server, aligns with NTP labs (later topics).

**Analogy**: Switch configuration is like building town roads: VLANs are lanes, SVIs are intersections, DHCP assigns addresses to residents (APs, clients), and LAG is a wide highway to the mayor’s office (WLC).

**Lifelong Retention Tip**: In Packet Tracer, configure switch with VLANs 10/100/200, LAG, and DHCP Option 43. Flashcard: “Switch config? VLANs, SVIs, access ports (APs), LAG (WLC), DHCP, NTP.” Practice 3 times.

## WLC Initial Setup

**Overview**:
- **Purpose**: Configure WLC via CLI wizard to enable GUI access and AP management.
- **Configuration (via Console)**:
  - **Wizard Steps**:
    1. Terminate autoinstall: Press Enter (skip TFTP config download).
    2. System name: `WLC1`.
    3. Admin username/password: `admin`/`Cisco123`.
    4. Enable LAG: `yes` (for Fa0/1–2 on switch).
    5. Management interface:
       - IP: `192.168.1.100`, Mask: `255.255.255.0`.
       - Gateway: `192.168.1.1` (switch SVI).
       - VLAN: `10`.
       - DHCP server: `192.168.1.1` (switch).
    6. Virtual interface: `1.1.1.1` (for client DHCP relay).
    7. Multicast IP: `239.0.0.1` (Class D, for AP traffic).
    8. Mobility/RF group: `default` (for multiple WLCs, optional).
    9. WLAN: SSID `Internal`, disable DHCP bridging, allow static IPs, no RADIUS.
    10. Country code: `FR` (matches AP regulatory domain, e.g., -E for Europe).
    11. Enable 802.11b/a/g, auto-RF (channel/power).
    12. NTP server: `192.168.1.1` (switch).
    13. Save and reset.
- **Additional Details**:
  - **Regulatory Domain**: Must match AP (e.g., -E for Europe, -A for US/Canada) to allow AP-WLC joining.
  - **GUI Access**: Post-setup, access WLC at `https://192.168.1.100` from PC on VLAN 10 (Fa0/6).
  - **Certificate Warning**: WLC’s self-signed certificate may trigger browser warning; proceed safely in lab.
  - **Real-World Tip**: Use Cisco Catalyst 9800 WLC; verify setup with `show running-config`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced WLC role; here, configures WLC for AP management.
- **Wireless Architectures**: Detailed CAPWAP and split-MAC; here, sets up management interface for CAPWAP.
- **Wireless Security**: WPA2 PSK configured later; here, skips RADIUS in wizard.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover WLC on VLAN 10; here, assigns management IP.
- **Standard ACLs (Day 34)**: Protect management IP; here, sets up VLAN 10 for management.
- **Extended ACLs (Day 35)**: Restrict protocols; here, management access via HTTPS.
- **IPv6 Parts 1–3**: Supports IPv6 management; here, uses IPv4 but extensible.

**Comparison to Other CCNA Topics**:
- **VLANs**: Management interface on VLAN 10, like switch SVIs.
- **Security (AAA)**: Admin username/password aligns with AAA login security.
- **NTP**: WLC uses switch as NTP server, like NTP labs.

**Analogy**: WLC setup is like appointing a mayor: assign a name (WLC1), office address (192.168.1.100), and rules (no RADIUS, WPA2 later) via a hiring form (wizard).

**Lifelong Retention Tip**: Simulate WLC wizard in Packet Tracer, set IP `192.168.1.100`. Flashcard: “WLC wizard? Name, LAG, management IP, virtual IP, SSID, country, NTP.” Practice 3 times.

## WLC Interface Configuration

**Overview**:
- **Purpose**: Configure logical interfaces (management, virtual, dynamic) to support CAPWAP, client traffic, and VLAN mapping.
- **WLC Ports vs. Interfaces**:
  - **Ports (Physical)**:
    - **Service Port**: Out-of-band management, access port (single VLAN).
    - **Distribution System Ports**: Data traffic, trunk ports, support LAG (e.g., Fa0/1–2).
    - **Console Port**: RJ45/USB for CLI access.
    - **Redundancy Port**: Connects WLCs for high availability (HA).
  - **Interfaces (Logical)**:
    - **Management Interface**: CAPWAP, SSH, HTTPS, RADIUS, NTP (e.g., 192.168.1.100, VLAN 10).
    - **Virtual Interface**: DHCP relay, web authentication (e.g., 1.1.1.1).
    - **Dynamic Interfaces**: Map WLANs to VLANs (e.g., Internal to VLAN 100, Guest to VLAN 200).
    - **Service Port Interface**: Binds to service port for out-of-band management (not used here).
    - **Redundancy Management Interface**: Manages standby WLC in HA pair.
- **GUI Configuration**:
  - **Management Interface**: Preconfigured in wizard (192.168.1.100, VLAN 10).
  - **Virtual Interface**: Preconfigured (1.1.1.1).
  - **Dynamic Interfaces**:
    - **Internal**:
      - Navigate: Controller > Interfaces > New.
      - Name: `Internal`, VLAN: `100`.
      - IP: `10.0.0.100`, Mask: `255.255.255.0`, Gateway: `10.0.0.1`, DHCP: `192.168.1.1`.
    - **Guest**:
      - Name: `Guest`, VLAN: `200`.
      - IP: `10.1.0.100`, Mask: `255.255.255.0`, Gateway: `10.1.0.1`, DHCP: `192.168.1.1`.
- **Additional Details**:
  - **LAG**: Distribution ports (Fa0/1–2) form static LAG, trunking VLANs 10, 100, 200.
  - **HA**: Redundancy port/interface unused in single WLC setup.
  - **Real-World Tip**: Use Cisco 9800 WLC GUI; verify interfaces with `show interface summary`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced distribution system; here, distribution ports/interfaces handle data.
- **Wireless Architectures**: Detailed WLC interfaces; here, configures management/dynamic interfaces.
- **Wireless Security**: Interfaces support WPA2; here, maps WLANs to VLANs.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover WLC ports; here, configures logical interfaces.
- **Standard ACLs (Day 34)**: Protect management interface; here, sets up 192.168.1.100.
- **Extended ACLs (Day 35)**: Restrict management traffic; here, CPU ACLs applied later.
- **IPv6 Parts 1–3**: Dynamic interfaces support IPv6; here, uses IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: Dynamic interfaces map to VLANs, like SVIs.
- **EtherChannel**: LAG aligns with static EtherChannel.
- **Security**: Management interface uses HTTPS, like secure access in AAA.

**Analogy**: WLC interfaces are like the mayor’s office departments: management (CAPWAP, SSH), virtual (client relay), dynamic (WLAN-to-VLAN mapping). Ports are physical doors (distribution, service).

**Lifelong Retention Tip**: In Packet Tracer, create dynamic interfaces for VLANs 100/200. Flashcard: “WLC interfaces? Management (CAPWAP), Virtual (DHCP), Dynamic (VLANs).” Practice 3 times.

## WLAN Configuration with WPA2 PSK (2.9, 5.10)

**Overview**:
- **Purpose**: Configure “Internal” and “Guest” WLANs with WPA2 PSK via WLC GUI for client connectivity.
- **GUI Configuration**:
  - **Internal WLAN** (preconfigured in wizard, needs edits):
    - Navigate: WLANs > WLAN ID 1.
    - **General Tab**:
      - Profile Name: `Internal`, SSID: `Internal`, Status: Enabled.
      - Interface: Change from `management` to `Internal` (VLAN 100).
    - **Security Tab**:
      - Layer 2: Select `WPA+WPA2`.
      - Authentication Key Management: Deselect `802.1X`, select `PSK`.
      - PSK Format: `ASCII`, Password: `Jeremy1234` (minimum 8 characters).
    - **QoS Tab**: Set to `Silver` (best effort, default).
    - **Advanced Tab**: Leave defaults (e.g., no max clients, no FlexConnect).
    - Apply changes.
  - **Guest WLAN**:
    - Navigate: WLANs > Create New > Go.
    - **General Tab**:
      - Profile Name: `Guest`, SSID: `Guest`, ID: `2`.
      - Status: Enable.
      - Interface: Select `Guest` (VLAN 200).
    - **Security Tab**:
      - Layer 2: Select `WPA+WPA2`.
      - Authentication Key Management: Select `PSK`.
      - PSK Format: `ASCII`, Password: `Guest1234`.
    - **QoS Tab**: Set to `Silver`.
    - **Advanced Tab**: Leave defaults.
    - Apply changes.
- **Additional Details**:
  - **WPA2 PSK**: Uses CCMP (AES encryption, CBC-MAC for MIC), four-way handshake for key generation.
  - **QoS Levels**: Platinum (voice), Gold (video), Silver (best effort), Bronze (background).
  - **Layer 3 Options**: Web authentication/passthrough unused; common in public Wi-Fi (e.g., cafes).
  - **Real-World Tip**: Use WPA3 for new deployments; WPA2 for CCNA and legacy compatibility.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Configured SSIDs; here, maps SSIDs to VLANs with WPA2.
- **Wireless Architectures**: Lightweight APs/WLC; here, configures WLANs via GUI.
- **Wireless Security**: Detailed WPA2 PSK; here, implements via GUI.
- **CDP/LLDP Lecture/Lab**: APs discovered on VLAN 10; here, WLANs enable client connectivity.
- **Standard ACLs (Day 34)**: Filter client traffic post-WLAN setup; here, secures with WPA2.
- **Extended ACLs (Day 35)**: Restrict protocols; here, VLANs segment traffic.
- **IPv6 Parts 1–3**: Clients get IPv6 post-WLAN setup; here, uses IPv4.

**Comparison to Other CCNA Topics**:
- **VLANs**: WLANs map to VLANs, like switch VLAN configs.
- **Security (AAA)**: WPA2 PSK aligns with password-based security; no RADIUS here.
- **QoS**: QoS levels (Silver, Gold) align with later QoS topics.

**Analogy**: Configuring WLANs is like setting up town meeting halls: assign names (SSIDs), secure entry with passwords (WPA2 PSK), and direct attendees to roads (VLANs).

**Lifelong Retention Tip**: In Packet Tracer, configure WPA2 PSK for “Internal” SSID. Flashcard: “WPA2 PSK config? GUI, WPA+WPA2, PSK, ASCII password (8+ chars).” Practice 3 times.

## Additional WLC Features (2.7, 2.8)

**Overview**:
- **AP Management**:
  - Navigate: Wireless > Access Points.
  - Verify AP status, IP, mode (e.g., Local), model, MAC.
  - Change mode (e.g., FlexConnect, Monitor) if needed.
- **Management Access (2.7)**:
  - Navigate: Management > HTTP/HTTPS, SSH.
  - Settings: Enable HTTPS, SSH; disable Telnet (insecure).
  - Management via Wireless: Disabled by default; enable for wireless client access (not recommended).
- **CPU ACLs**:
  - Navigate: Security > Access Control Lists > New.
  - Create `MANAGEMENT_ACL` (IPv4).
  - Add rules (e.g., permit 192.168.1.0/24 to 192.168.1.100, HTTPS).
  - Apply to CPU: Security > CPU Access Control Lists > Enable, select `MANAGEMENT_ACL`.
- **Additional Details**:
  - **AP Modes**: Local (default), FlexConnect (local switching), Monitor (rogue detection), etc.
  - **Security**: CPU ACLs protect WLC management (SSH, HTTPS, SNMP), not client traffic.
  - **Real-World Tip**: Use Cisco DNA Center for centralized WLC management; verify ACLs with `show access-lists`.

**Comparison to Prior Content**:
- **Wireless Fundamentals**: Introduced AP modes; here, verifies/configures via GUI.
- **Wireless Architectures**: Detailed split-MAC modes; here, manages APs in Local mode.
- **Wireless Security**: CPU ACLs align with WPA2 access control.
- **CDP/LLDP Lecture/Lab**: CDP/LLDP discover APs/WLC; here, manages via GUI.
- **Standard ACLs (Day 34)**: CPU ACLs are standard ACLs for management traffic.
- **Extended ACLs (Day 35)**: CPU ACLs can restrict protocols (e.g., HTTPS only).
- **IPv6 Parts 1–3**: CPU ACLs support IPv6; here, uses IPv4.

**Comparison to Other CCNA Topics**:
- **Security (AAA)**: CPU ACLs align with AAA access control.
- **VLANs**: Management access on VLAN 10, like switch management.
- **QoS**: QoS settings (Silver) align with later QoS topics.

**Analogy**: WLC features are like the mayor’s toolkit: manage criers (APs), secure office access (CPU ACLs), and prioritize messages (QoS).

**Lifelong Retention Tip**: In Packet Tracer, create CPU ACL to permit HTTPS to WLC. Flashcard: “WLC features? AP modes, HTTPS/SSH, CPU ACLs, QoS (Silver).” Practice 3 times.

## Common Issues and Troubleshooting

- **APs Not Joining WLC**:
  - **Issue**: APs fail to form CAPWAP tunnel.
  - **Cause**: Wrong regulatory domain, no IP, firewall blocking UDP 5246/5247, or missing Option 43.
  - **Fix**: Verify country code (`show country`), AP IP (`show capwap ip config`), firewall rules, Option 43.
- **Client Fails to Connect**:
  - **Issue**: Laptop can’t join “Internal” SSID.
  - **Cause**: Wrong PSK, WLAN disabled, or incorrect interface mapping.
  - **Fix**: Check PSK (`show wlan security`), WLAN status (`show wlan summary`), interface (`show wlan`).
- **No IP for Clients**:
  - **Issue**: Clients don’t get DHCP IPs.
  - **Cause**: Missing DHCP pool, wrong VLAN mapping, or WLC virtual interface misconfigured.
  - **Fix**: Verify DHCP pools (`show ip dhcp pool`), WLAN-to-VLAN mapping, virtual IP (`show interface virtual`).
- **GUI Access Denied**:
  - **Issue**: Can’t access WLC GUI.
  - **Cause**: Wrong IP, HTTPS disabled, or CPU ACL blocking.
  - **Fix**: Verify IP (`192.168.1.100`), HTTPS status (`show management`), ACL (`show access-lists`).
- **Certificate Warning**:
  - **Issue**: Browser warns “connection not private.”
  - **Cause**: Self-signed WLC certificate.
  - **Fix**: Proceed in lab; use trusted certificate in production.

**Analogy**: Troubleshooting is like fixing a town’s communication: ensure criers (APs) connect to the mayor (WLC, CAPWAP), residents (clients) have addresses (DHCP), and the mayor’s office is accessible (GUI, HTTPS).

**Lifelong Retention Tip**: Simulate AP join failure in Packet Tracer, check `show capwap status`. Checklist: “No AP join? Check country, IP, UDP 5246/5247. No client? Check PSK, VLAN, DHCP.” Practice 3 times.

## Practice Commands (CLI Verification)

- **Switch**:
  ```plaintext
  Switch# show vlan brief
  Switch# show ip interface brief
  Switch# show etherchannel summary
  Switch# show ip dhcp pool
  Switch# show power inline
  ```
- **WLC**:
  ```plaintext
  WLC# show interface summary
  WLC# show wlan summary
  WLC# show ap summary
  WLC# show capwap status
  WLC# show access-lists
  WLC# show management
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Topology: “Switch (VLANs, PoE), WLC (LAG, CAPWAP), APs (access, DHCP).”
   - Interfaces: “Management (CAPWAP), Virtual (DHCP), Dynamic (VLANs).”
   - QoS: “Platinum (voice), Gold (video), Silver (default), Bronze (background).”
2. **Visualize Topology**:
   - Draw switch (VLANs 10, 100, 200), WLC (LAG), APs (CAPWAP), clients (SSIDs). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “WLC ports? Service, Distribution, Console, Redundancy. WPA2 PSK? GUI, ASCII, 8+ chars.”
4. **Practice in Packet Tracer**:
   - Configure switch (LAG, VLANs), WLC (WPA2 PSK), verify `show wlan summary`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Switch (access ports, LAG), WLC (GUI, WPA2), APs (Local mode).
   - Quiz: “WLAN-to-VLAN mapping? Dynamic interface.”
6. **Troubleshooting Practice**:
   - Simulate no CAPWAP, check `show capwap status`, configure Option 43. Checklist: “No GUI? Check IP, HTTPS, ACL.”
7. **Teach Someone**:
   - Explain WPA2 PSK setup, LAG, and dynamic interfaces.
8. **Spaced Repetition**:
   - Use Anki for interfaces, QoS, ACLs. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Topology**:
   - Switch (VLANs 10, 100, 200, PoE, LAG), WLC (CAPWAP, GUI), APs (access ports, Local mode).
2. **Switch Configuration**:
   - VLANs, SVIs, DHCP (Option 43), LAG (mode on), NTP.
3. **WLC Setup**:
   - CLI wizard: name, LAG, management IP (192.168.1.100), virtual IP, SSID, country, NTP.
4. **Interfaces**:
   - Management (CAPWAP), Virtual (DHCP relay), Dynamic (Internal: VLAN 100, Guest: VLAN 200).
5. **WLAN Configuration**:
   - WPA2 PSK via GUI, “Internal” (VLAN 100), “Guest” (VLAN 200), QoS Silver.
6. **CCNA Focus**:
   - Topics 2.7 (GUI access), 2.8 (WLAN components), 2.9/5.10 (WPA2 PSK via GUI).

These notes provide a complete foundation for the Wireless LAN Configuration lecture, preparing you for CCNA exam topics 2.7, 2.8, 2.9, and 5.10. Practice configuring WPA2 PSK, troubleshooting CAPWAP, and exploring WLC GUI to master WLAN setup for life!