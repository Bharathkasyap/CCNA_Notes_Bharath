# CCNA 200-301: Professional Revision Guide

This guide provides a comprehensive review of core concepts, commands, and protocols for the CCNA 200-301 exam. It is structured for efficient last-minute revision.

---

## 1. Network Fundamentals

This section covers the foundational building blocks of modern networks.

### OSI and TCP/IP Models

* **OSI Model:** A 7-layer conceptual model for understanding network interactions.
    1.  **Physical:** Bits, cables, connectors, hubs.
    2.  **Data Link:** Frames, MAC addresses, switches, bridges.
    3.  **Network:** Packets, IP addresses, routers.
    4.  **Transport:** Segments, TCP/UDP, Port numbers.
    5.  **Session:** Session management, establishment, termination.
    6.  **Presentation:** Data formatting, encryption (e.g., SSL/TLS).
    7.  **Application:** User-facing protocols (HTTP, FTP, DNS).
* **TCP/IP Model:** A 4-layer model used in practice.
    1.  **Network Access:** Combines OSI Physical and Data Link.
    2.  **Internet:** Maps to OSI Network Layer.
    3.  **Transport:** Maps to OSI Transport Layer.
    4.  **Application:** Combines OSI Session, Presentation, and Application.

### IP Addressing & Subnetting

* **MAC Address:** 48-bit, physical (Layer 2) address, burned into the NIC.
* **IPv4 Address:** 32-bit, logical (Layer 3) address used for routing.
* **IPv6 Address:** 128-bit, logical (Layer 3) address providing a vast address space.
    * **Link-Local Address (LLA):** `FE80::/10` prefix, automatically configured, for local segment communication.
    * **Global Unicast Address (GUA):** Routable on the public internet.

| CIDR  | Hosts | Wildcard Mask | Use Case            |
| :---- | :---- | :------------ | :------------------ |
| `/24` | 254   | `0.0.0.255`   | Common LAN          |
| `/27` | 30    | `0.0.0.31`    | Small subnet        |
| `/30` | 2     | `0.0.0.3`     | Point-to-point link |

### Key Protocols and Concepts

| Concept       | Purpose                                                       | Key Protocols / Commands                  |
| :------------ | :------------------------------------------------------------ | :---------------------------------------- |
| **TCP vs UDP** | TCP is connection-oriented and reliable; UDP is connectionless and fast. | TCP (HTTP, FTP), UDP (DNS, DHCP, VoIP)    |
| **ARP** | Resolves an IP address to a MAC address on a local network.     | `show ip arp`                             |
| **CDP/LLDP** | Cisco/Link Layer Discovery Protocols discover connected devices. | `show cdp neighbors`, `show lldp neighbors` |
| **DNS** | Resolves domain names to IP addresses.                        | `ip name-server 8.8.8.8`                  |
| **NTP** | Synchronizes device clocks across the network.                | `ntp server <ip_address>`                 |
| **Syslog** | Sends log messages to a central server.                       | `logging <ip_address>`                    |

---

## 2. LAN Switching

This section focuses on Layer 2 technologies that build local area networks.

### VLANs and Trunking

VLANs segment a switch into multiple broadcast domains. Trunks carry traffic for multiple VLANs between switches.

| Feature               | Configuration Commands                                                                                              |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------ |
| **Create VLAN** | `vlan 10`<br>`name Sales`                                                                                          |
| **Assign Access Port** | `interface Fa0/1`<br>`switchport mode access`<br>`switchport access vlan 10`                                       |
| **Configure Trunk Port** | `interface Gi0/1`<br>`switchport trunk encapsulation dot1q` (older switches)<br>`switchport mode trunk`<br>`switchport trunk allowed vlan 10,20,30` |
| **Inter-VLAN Routing** | `interface Gi0/1.10` (subinterface)<br>`encapsulation dot1q 10`<br>`ip address 10.0.10.1 255.255.255.0`               |

### Spanning Tree Protocol (STP & RSTP)

STP prevents Layer 2 loops. RSTP (`802.1w`) is a faster-converging evolution.

```bash
# Set switch priority to become Root Bridge for VLAN 10
spanning-tree vlan 10 priority 4096

# Enable Rapid PVST+ mode
spanning-tree mode rapid-pvst
```

### EtherChannel

Bundles multiple physical links into a single logical link for higher bandwidth and redundancy.

| Protocol | Description                             | Configuration Mode  |
| :------- | :-------------------------------------- | :------------------ |
| **PAgP** | Cisco proprietary Port Aggregation Protocol. | `desirable`, `auto` |
| **LACP** | IEEE standard Link Aggregation Control Protocol. | `active`, `passive` |

```bash
# LACP EtherChannel Configuration
interface range Gi0/1-2
  channel-protocol lacp
  channel-group 1 mode active
  exit
interface Port-channel 1
  switchport mode trunk
  switchport trunk allowed vlan 10,20
```

### Switch Security

| Feature         | Purpose                           | Configuration Commands                                                                                                            |
| :-------------- | :-------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| **Port Security** | Limits MAC addresses on a port.   | `switchport port-security`<br>`switchport port-security maximum 2`<br>`switchport port-security violation shutdown`<br>`switchport port-security mac-address sticky` |
| **DHCP Snooping** | Prevents rogue DHCP servers.      | `ip dhcp snooping`<br>`ip dhcp snooping vlan 10`<br>`interface Gi0/1` (uplink to router)<br>`ip dhcp snooping trust`                  |
| **DAI** | Dynamic ARP Inspection prevents ARP spoofing. | `ip arp inspection vlan 10`<br>`interface Gi0/1`<br>`ip arp inspection trust`                                                     |


---

## 3. IP Routing

This section covers how routers make decisions to forward packets between networks.

### Static vs. Dynamic Routing

* **Static routes** are manually configured and are best for simple, predictable networks.
* **Dynamic routing protocols** automatically learn about remote networks and adapt to topology changes.

| Protocol       | Configuration Example                                                                                                   | Key Characteristics                                        |
| :------------- | :---------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------- |
| **Static Route** | `ip route 192.168.2.0 255.255.255.0 10.0.0.2`                                                                            | Manually configured, low overhead.                         |
| **OSPFv2** | `router ospf 1`<br>`router-id 1.1.1.1`<br>`network 10.0.0.0 0.0.0.255 area 0`<br>`passive-interface default`                 | Link-state, classless, uses Areas for scalability.         |
| **EIGRP** | `router eigrp 100`<br>`network 10.0.0.0`<br>`no auto-summary`                                                               | Cisco proprietary, fast convergence, classless.            |
| **IPv6 Static** | `ipv6 route 2001:DB8:ACAD::/48 2001:DB8:FEED::2`                                                                         | Same logic as IPv4.                                        |
| **OSPFv3 (for IPv6)** | `ipv6 router ospf 1`<br>`router-id 1.1.1.1`<br>`interface Gi0/0`<br>`ipv6 ospf 1 area 0`                          | Configured on interface.                                   |

### First Hop Redundancy Protocol (FHRP)

**HSRP (Hot Standby Router Protocol)** provides a virtual IP for gateway redundancy.

```bash
# HSRP Configuration on two routers
interface Gi0/0
  standby 1 ip 192.168.1.254
  standby 1 priority 110
  standby 1 preempt
```

---

## 4. IP Services & Security

### Access Control Lists (ACLs)

ACLs filter traffic based on source/destination IP, ports, and protocol.

| ACL Type            | Configuration Example                                | Application                       |
| :------------------ | :--------------------------------------------------- | :-------------------------------- |
| **Standard** | `access-list 10 permit 192.168.10.0 0.0.0.255`       | Place close to the destination.   |
| **Extended** | `access-list 101 permit tcp any host 172.16.5.10 eq 80` | Place close to the source.        |
| **Apply to Interface** | `interface Gi0/0`<br>`ip access-group 101 in`        | Activates the filter on an interface. |

### Network Address Translation (NAT)

**PAT (Port Address Translation)** translates many private IPs to one public IP.

```bash
access-list 1 permit 192.168.0.0 0.0.0.255

interface Gi0/0
  ip nat outside
interface Gi0/1
  ip nat inside

ip nat inside source list 1 interface Gi0/0 overload
```

### Secure Shell (SSH)

Encrypted remote access via terminal.

```bash
ip domain-name mylab.local
hostname R1
crypto key generate rsa
username admin secret MySecurePass
line vty 0 4
  login local
  transport input ssh
```

---

## 5. Wireless Networking

### Core Wireless Concepts

* **SSID (Service Set Identifier):** The network name.
* **BSS (Basic Service Set):** An AP and its connected clients.
* **ESS (Extended Service Set):** Multiple APs with the same SSID.
* **RF Channels:** Frequencies used for communication (e.g., 1, 6, 11 in 2.4GHz band).
* **Architectures:**
  - **Autonomous AP:** Individually managed.
  - **Controller-Based AP:** Managed by Wireless LAN Controller (WLC).

### Wireless Security

| Method                  | Description                                                          |
| :---------------------- | :------------------------------------------------------------------- |
| **WPA2/WPA3** | Modern Wi-Fi encryption standards. WPA3 is most secure.               |
| **PSK (Pre-Shared Key)** | Common password for all clients. Easy but less secure.        |
| **Enterprise (802.1X)** | Each user authenticates against RADIUS. Used in organizations. |

---

## 6. Automation & Programmability

| Component       | Description                                       | Example                                                  |
| :-------------- | :------------------------------------------------ | :------------------------------------------------------- |
| **Data Formats** | API payload structure.                            | JSON, YAML                                               |
| **REST API** | Interface for device configuration over HTTP.       | `GET /restconf/data/interfaces`                          |
| **Config Mgmt** | Automates deployment across multiple devices.     | Tools: Ansible, Puppet, Chef                             |
| **Python** | Scripting for device automation. Uses Netmiko, Requests | SSH automation, config changes                           |

```python
# Netmiko Python Example
from netmiko import ConnectHandler

device = {
    'device_type': 'cisco_ios',
    'host': '10.1.1.1',
    'username': 'admin',
    'password': 'password',
}

with ConnectHandler(**device) as net_connect:
    config_commands = ['interface loopback 0', 'ip address 1.1.1.1 255.255.255.255']
    output = net_connect.send_config_set(config_commands)
    print(output)
```

---

## 7. Troubleshooting Command Cheat Sheet

| Area            | Command                             | Purpose                                 |
|-----------------|--------------------------------------|-----------------------------------------|
| Interface       | `show ip interface brief`           | Verify interface status and IPs         |
|                 | `show interface status`             | Check duplex, speed, and connectivity   |
| Routing         | `show ip route`                     | Routing table overview                  |
|                 | `ping`, `traceroute`                | Check Layer 3 reachability              |
| OSPF            | `show ip ospf neighbor`             | Check adjacency status                  |
|                 | `show ip protocols`                 | View active protocols and networks      |
| VLAN            | `show vlan brief`                   | Port-to-VLAN mapping                    |
| STP             | `show spanning-tree vlan <id>`      | View root bridge and port states        |
| ARP             | `show ip arp`                       | IP-to-MAC resolution                    |
| ACL/NAT         | `show access-lists`                 | Validate ACL hits                       |
|                 | `show ip nat translations`          | See active NAT sessions                 |
| Discovery       | `show cdp neighbors detail`         | View directly connected Cisco devices   |

---

## CCNA Extended Revision – Extra Concepts & Syntax Cheat Sheet

This section includes **additional topics** often tested in the CCNA 200-301 exam and a **dedicated syntax reference** to speed up CLI recall.

---

## 🔧 Extended Topics

### 1. IPv6 Essentials

| Term        | Description                                  | Example                          |
|-------------|----------------------------------------------|----------------------------------|
| Link-Local  | Local segment-only addresses                 | `FE80::/10`                      |
| Global Unicast | Routable IPv6 addresses                  | `2001::/16`                      |
| EUI-64      | Interface ID auto-configured from MAC        | `ipv6 address 2001::/64 eui-64`  |
| Static Route | Manual IPv6 route                           | `ipv6 route 2001:db8::/64 ::1`   |

### 2. DHCP Configuration (Router as Server)

```bash
service dhcp
ip dhcp pool SALES
  network 192.168.10.0 255.255.255.0
  default-router 192.168.10.1
  dns-server 8.8.8.8
```

### 3. NAT Types

| Type     | Description                            |
|----------|----------------------------------------|
| Static   | One-to-one IP translation              |
| Dynamic  | Pool of public IPs to private IPs      |
| PAT      | Many-to-one (overload with ports)      |

### 4. Router on a Stick (Inter-VLAN Routing)

```bash
interface Gi0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```

### 5. VTP (VLAN Trunking Protocol)

| Mode    | Description                   |
|---------|-------------------------------|
| Server  | Can add/delete VLANs          |
| Client  | Receives from server only     |
| Transparent | Does not participate, forwards only |

```bash
vtp mode transparent
vtp domain CCNA
vtp password cisco
```

### 6. Telnet Access

```bash
line vty 0 4
 password cisco
 login
 transport input telnet
```

---

## 📘 CCNA Syntax Quick Reference

### 🔹 Interface Setup

```bash
enable
configure terminal
interface g0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

### 🔹 VLAN Commands

```bash
vlan 10
name Sales
interface f0/1
switchport mode access
switchport access vlan 10
```

### 🔹 Static Routing

```bash
ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

### 🔹 OSPF

```bash
router ospf 1
network 10.0.0.0 0.0.0.255 area 0
```

### 🔹 EIGRP

```bash
router eigrp 100
network 192.168.0.0
no auto-summary
```

### 🔹 NAT (PAT)

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface g0/0 overload
interface g0/0
 ip nat outside
interface g0/1
 ip nat inside
```

### 🔹 ACLs

```bash
access-list 100 permit tcp any any eq 80
interface g0/0
ip access-group 100 in
```

### 🔹 SSH

```bash
ip domain-name mydomain.com
crypto key generate rsa
username admin secret ccna123
line vty 0 4
 login local
 transport input ssh
```

---


## CCNA Additional Concepts 

This section continues to expand on topics often overlooked but crucial for deeper CCNA understanding and real-world job readiness.

---

## 1. Switchport Modes (Access vs Trunk)

| Mode          | Description                                   |
|---------------|-----------------------------------------------|
| Access        | Assigned to a single VLAN                     |
| Trunk         | Carries multiple VLANs using tagging (802.1Q) |
| Dynamic Auto  | Passive; forms trunk if the other end is set to trunk or desirable |
| Dynamic Desirable | Actively attempts to form a trunk        |

```bash
interface f0/1
 switchport mode access
 switchport access vlan 10

interface f0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

## 2. Native VLAN and VLAN Mismatches

- Native VLAN is the **untagged VLAN** on a trunk port.
- Must match on both sides of the trunk link.
- Mismatched native VLANs can cause **security and traffic issues**.

```bash
switchport trunk native vlan 99
```

---

## 3. STP Port States

| State        | Description                                      |
|--------------|--------------------------------------------------|
| Blocking     | Listens for BPDUs but does not forward frames    |
| Listening    | Accepts BPDUs, prepares to transition            |
| Learning     | Learns MACs but doesn't forward                  |
| Forwarding   | Forwards frames and learns MACs                  |
| Disabled     | Administratively shut down                       |

---

## 4. STP Port Roles

| Role        | Description                                         |
|-------------|-----------------------------------------------------|
| Root Port   | Path to Root Bridge (one per non-root switch)       |
| Designated  | One per segment; forwards traffic to/from that segment |
| Alternate   | Backup path (in RSTP)                               |
| Backup      | Redundant link on same segment (in RSTP)            |

---

## 5. Syslog Severity Levels

| Level | Name       | Description               |
|-------|------------|---------------------------|
| 0     | Emergency  | System is unusable        |
| 1     | Alert      | Immediate action needed   |
| 2     | Critical   | Critical condition        |
| 3     | Error      | Error condition           |
| 4     | Warning    | Warning condition         |
| 5     | Notice     | Normal but significant    |
| 6     | Informational | Informational messages |
| 7     | Debug      | Debug-level messages      |

---

## 6. DNS and DHCP Relay

- When a router forwards a DHCP request from a different subnet to a DHCP server.

```bash
interface g0/0
 ip helper-address 192.168.1.10
```

---

## 7. Router Boot Sequence

1. POST
2. Bootstrap runs
3. IOS loaded from flash
4. Startup-config from NVRAM loaded
5. If none found → setup mode

Useful commands:
```bash
show version
show flash:
show startup-config
```

---

## 8. Password Recovery Steps (Cisco)

1. Reboot and enter **ROMmon mode**
2. Change config register:
   ```bash
   confreg 0x2142
   ```
3. Restart and enter privileged exec
4. Copy startup-config to running-config
5. Reset password and save config:
   ```bash
   copy startup-config running-config
   enable secret NEWPASS
   config-register 0x2102
   write memory
   ```

---


## CCNA Extended– Exam Traps, Troubleshooting & Labs

This part focuses on **tricky exam questions**, **real-world issues**, and **lab-based guidance** that help reinforce practical knowledge.

---

## 🧠 1. Common Exam Traps to Watch For

| Trap Type          | What to Watch Out For                                             |
|--------------------|-------------------------------------------------------------------|
| **Wrong mask**     | CIDR vs. subnet mismatch (`/25` vs. `/24`)                        |
| **Wrong native VLAN** | Trunk links with mismatched native VLANs cause issues         |
| **Unidirectional link** | Switch A sees Switch B, but not vice-versa – likely trunk issue |
| **ACL direction**  | Make sure it's `in` or `out` on the correct interface             |
| **OSPF wildcard**  | Exam may show `10.0.0.0 0.0.255.255` instead of `10.0.0.0/16`     |
| **No `no shutdown`** | Interfaces stay down unless manually enabled                   |

---

## 🔍 2. Real-World Troubleshooting Use Cases

### A. PC Cannot Ping Gateway

```bash
# Step-by-step
show ip interface brief         # Check IP assigned and interface status
show running-config interface x # Verify VLAN and IP
show vlan brief                 # Is the port in the correct VLAN?
```

### B. Inter-VLAN Routing Fails

```bash
# Troubleshooting
show ip route                   # Routes to VLANs must exist
show interfaces trunk           # Check if trunk links are active
show vlan brief                 # Ports assigned to correct VLANs?
```

### C. OSPF Neighbors Not Forming

```bash
show ip ospf neighbor           # Should be FULL state
show ip protocols               # Confirm router-id, networks
show ip ospf interface brief    # Check OSPF on correct interface
```

---

## 🧪 3. Lab Ideas with Minimum Setup

### Lab 1: VLAN & Trunking

**Devices**: 2 Switches, 2 PCs  
**Objective**:
- Create VLANs 10 and 20
- Assign PC1 to VLAN 10, PC2 to VLAN 20
- Trunk between switches

### Lab 2: Router on a Stick

**Devices**: 1 Switch, 1 Router, 2 PCs  
**Objective**:
- Configure subinterfaces on router
- Enable VLAN-based communication across PCs

### Lab 3: Static + OSPF Routing

**Devices**: 2 Routers, 1 Switch  
**Objective**:
- Use static route between local LANs
- Configure OSPF to share remote routes

---

## 🛠 4. High-Yield Commands for Labs

```bash
show ip interface brief
show interface trunk
show vlan brief
show ip route
show ip ospf neighbor
show spanning-tree
show access-lists
debug ip icmp
debug ip routing
```

---

## CCNA Professional Revision: Difficulties and Exam Traps

This section outlines the **most common difficulties**, **trick questions**, and **conceptual pitfalls** that candidates face in the CCNA 200-301 exam. Awareness of these will help reduce silly mistakes and improve pass rates.

---

## 🔥 1. Subnetting Confusion

| Difficulty | Explanation |
|------------|-------------|
| **CIDR → Subnet Mask** | Candidates struggle converting `/26` → `255.255.255.192` |
| **Host Calculation** | Not subtracting 2 from usable hosts (for network/broadcast) |
| **Wildcard Masks** | Mistaking wildcards in ACLs (`0.0.0.255` is for `/24`) |
| **Binary Math** | Binary-to-decimal conversion errors in subnetting |

> **Trap**: Not practicing enough to do subnetting in 15–20 seconds.

---

## 🚨 2. ACL Direction and Placement

| Trap | Description |
|------|-------------|
| Wrong Interface | Applying ACL on the wrong interface (inside vs outside) |
| Wrong Direction | Using `in` instead of `out`, or vice versa |
| Order Matters | ACLs are top-down, first match wins |
| Forgetting `deny` all | Implicit deny at the end isn't visible but always active |

> **Tip**: Always simulate traffic and direction before applying ACL.

---

## ⚙️ 3. NAT Misconfiguration

| Error | Why it Happens |
|-------|----------------|
| `ip nat inside` / `ip nat outside` mismatch | Interface roles incorrectly assigned |
| Access list mismatch | Access list doesn't match the intended internal network |
| Forgetting Overload | Missing `overload` in PAT setup causes NAT to fail |

---

## 🧭 4. Routing Protocol Mistakes

| Mistake | Notes |
|---------|-------|
| OSPF wildcard mask confusion | Students write `10.0.0.0 255.255.255.0` instead of `10.0.0.0 0.0.0.255` |
| Not setting router ID | OSPF requires a router ID manually or from highest loopback |
| EIGRP `no auto-summary` | Needed to disable classful behavior |
| Routing loops in RIP | Caused by forgetting hop count or update intervals |

---

## 🧱 5. STP & VLAN Problems

| Issue | Explanation |
|-------|-------------|
| Native VLAN mismatch | Causes untagged traffic to be dropped silently |
| VLAN not allowed on trunk | `switchport trunk allowed vlan` missing VLAN |
| Wrong STP mode | Using PVST+ instead of RPVST causes delay in convergence |
| Port security shutdown | MAC address violations can cause port shutdowns |

---

## 🧪 6. Configuration Order Errors

| Misstep | Why It Fails |
|---------|--------------|
| Skipping `no shutdown` | Interfaces stay down unless enabled |
| Applying settings to wrong port | Easily done when working with ranges |
| Not saving config | `write memory` or `copy run start` forgotten |

---

## 🧠 7. Mental Traps During Exam

- Overthinking simple questions
- Not checking **default behavior** (e.g., ACLs deny all by default)
- Not practicing **real CLI commands** – leads to panic during sim questions
- Getting stuck on **Subnetting early in the exam**, wasting 20+ minutes
- **Skipping drag-and-drop** at first – come back later if unsure

---



## CCNA Professional Revision: Final Checklist Before Exam

This section provides a focused list of **must-know concepts**, **commands**, and **mental strategies** that candidates should revise **on the day before the exam**.

---

## ✅ 1. Subnetting

- Be able to **calculate subnets within 20 seconds**
- Memorize:
  - `/24` → 256 IPs → 254 usable
  - `/30` → 4 IPs → 2 usable
  - `Wildcard mask` = Inverse of subnet mask
- **Practice quick math**:
  ```
  /26 = 64 block size
  /27 = 32
  /28 = 16
  /29 = 8
  /30 = 4
  ```

---

## ✅ 2. Routing Protocols (Syntax + Concepts)

- **Static Routing**:
  ```bash
  ip route 192.168.2.0 255.255.255.0 10.0.0.1
  ```

- **OSPF**:
  ```bash
  router ospf 1
  network 10.0.0.0 0.0.0.255 area 0
  router-id 1.1.1.1
  ```

- **EIGRP**:
  ```bash
  router eigrp 100
  network 192.168.1.0
  no auto-summary
  ```

---

## ✅ 3. VLANs & Trunks

- Know how to:
  - Create VLAN
  - Assign port
  - Configure trunk
- Watch for:
  - **Native VLAN mismatch**
  - VLAN not allowed on trunk

---

## ✅ 4. ACLs & NAT (Trickiest)

- **Standard ACL** close to destination  
- **Extended ACL** close to source
- NAT Overload:
  ```bash
  ip nat inside source list 1 interface g0/0 overload
  ```

---

## ✅ 5. Show & Debug Commands

| Purpose                | Command                        |
|------------------------|--------------------------------|
| IP & interface status  | `show ip interface brief`      |
| VLAN config            | `show vlan brief`              |
| Trunking info          | `show interfaces trunk`        |
| OSPF neighbor state    | `show ip ospf neighbor`        |
| NAT entries            | `show ip nat translations`     |
| MAC table              | `show mac address-table`       |
| IP to MAC              | `show ip arp`                  |
| Active ACLs            | `show access-lists`            |

---

## ✅ 6. Wireless Quick Facts

- 2.4GHz → Channels: 1, 6, 11 (non-overlapping)
- 5GHz → More channels, less interference
- WPA3 is most secure
- PSK vs 802.1X (Enterprise)

---

## ✅ 7. IPv6 Key Points

- **LLA**: FE80::/10
- **GUA**: 2000::/3
- Static route: `ipv6 route 2001::/64 ::1`

---

## ✅ 8. Lab Simulation Tips

- Always run `show run` and `show ip int brief` first
- If ping fails, check:
  1. Interface up?
  2. IP configured?
  3. Route exists?
  4. ACL/NAT applied?

---

## ✅ 9. Exam Day Strategy

- Skip subnetting-heavy questions initially, come back later
- Flag long questions, answer all short ones first
- Drag-and-drop questions are **not timed separately**
- Timebox each section:
  - 50 Qs → 45–50 mins max
  - Leave time for review

---




📌 Tip: When in doubt, check Layer 1 first, then interface config, then routing/NAT/ACL.

📌 **Pro Tip**: During exam, flag confusing questions. Focus on quick wins first, then come back.

📌 Final Tip: CCNA is command-heavy. Practice commands every day. You'll remember what you *use*.

✅ Bookmark this file as your go-to syntax reference during hands-on lab practice.

✅ With awareness of these traps, you'll be better equipped to pass with confidence.

⏱ Practice under timed conditions. 15–20 seconds max per subnetting or ACL question.

✔️ Keep these notes close during practice and review to reinforce your muscle memory.




