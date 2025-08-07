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

✔️ Keep these notes close during practice and review to reinforce your muscle memory.

📌 Final Tip: CCNA is command-heavy. Practice commands every day. You'll remember what you *use*.


