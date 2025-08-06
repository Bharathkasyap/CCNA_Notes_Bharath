# 📘 Cisco CLI Commands Guide for CCNA

This guide lists Cisco IOS CLI commands topic-wise for CCNA preparation.

<details>
<summary>Day 1. Networking Devices</summary>

| Command | Purpose |
|---|---|
| `show version` | Displays device hardware, software version, and uptime |
| `show running-config` | Displays the active configuration |
| `show startup-config` | Displays configuration stored in NVRAM |
| `reload` | Reboots the device |

</details>

<details>
<summary>Day 2. Interfaces And Cables</summary>

| Command | Purpose |
|---|---|
| `show ip interface brief` | Shows interface status and IP addresses |
| `show controllers` | Displays interface hardware details |
| `ping <ip>` | Tests connectivity |
| `traceroute <ip>` | Traces path to destination |

</details>

<details>
<summary>Day 3. OSI Model and TCP/IP Suite</summary>

| Command | Purpose |
|---|---|
| `show protocols` | Shows configured protocols and addresses |
| `telnet <ip>` | Tests layer 4 TCP connectivity |
| `ssh <user>@<ip>` | Securely connects to remote device |

</details>

<details>
<summary>Day 4. Intro to the CLI</summary>

| Command | Purpose |
|---|---|
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `exit` | Exit current mode |
| `end` | Exit to privileged EXEC mode |

</details>

<details>
<summary>Day 5. Ethernet LAN Switching - Part 1</summary>

| Command | Purpose |
|---|---|
| `show mac address-table` | Displays MAC addresses learned |
| `clear mac address-table dynamic` | Clears dynamic MAC addresses |

</details>

<details>
<summary>Day 6. Ethernet LAN Switching - Part 2</summary>

| Command | Purpose |
|---|---|
| `switchport mode access` | Sets interface to access mode |
| `switchport mode trunk` | Sets interface to trunk mode |

</details>

<details>
<summary>Day 7. IPv4 Addressing - Part 1</summary>

| Command | Purpose |
|---|---|
| `ip address <ip> <mask>` | Assign IP address to interface |
| `no shutdown` | Enables the interface |

</details>

<details>
<summary>Day 8. IPv4 Addressing - Part 2</summary>

| Command | Purpose |
|---|---|
| `ip default-gateway <ip>` | Sets default gateway for switch |

</details>

<details>
<summary>Day 9. Switch Interfaces</summary>

| Command | Purpose |
|---|---|
| `interface <type> <number>` | Enters specific interface configuration mode |
| `shutdown` | Administratively disables an interface |
| `no shutdown` | Enables an interface |

</details>

<details>
<summary>Day 10. The IPv4 Header</summary>

| Command | Purpose |
|---|---|
| `show ip route` | Displays routing table |
| `debug ip packet` | Debugs packets in real time |

</details>

<details>
<summary>Day 11. Routing Fundamentals - Part 1</summary>

| Command | Purpose |
|---|---|
| `ip routing` | Enables routing on the router |
| `show ip route` | Displays the routing table |
| `show cdp neighbors` | Displays directly connected Cisco devices |

</details>

<details>
<summary>Day 12. Static Routing - Part 2</summary>

| Command | Purpose |
|---|---|
| `ip route <destination> <mask> <next-hop>` | Creates a static route |
| `no ip route <destination> <mask> <next-hop>` | Removes a static route |

</details>

<details>
<summary>Day 13. Life of a Packet</summary>

| Command | Purpose |
|---|---|
| `debug ip packet` | Displays live packet info (for analysis) |
| `show interfaces` | Displays traffic and errors on interfaces |

</details>

<details>
<summary>Day 14. Subnetting - Part 1</summary>

| Command | Purpose |
|---|---|
| `ip address <ip> <subnet>` | Assigns IP address with subnet |
| `show ip interface brief` | Verify IP and interface status |

</details>

<details>
<summary>Day 15. Subnetting - Part 2</summary>

| Command | Purpose |
|---|---|
| `show running-config` | Confirm subnet assignments in config |

</details>

<details>
<summary>Day 16. Subnetting (VLSM) - Part 3</summary>

| Command | Purpose |
|---|---|
| `ip route <subnet> <mask> <next-hop>` | Use custom subnets via static routes |

</details>

<details>
<summary>Day 17. VLANS - Part 1</summary>

| Command | Purpose |
|---|---|
| `vlan <id>` | Creates a VLAN |
| `name <vlan-name>` | Assigns a name to VLAN |

</details>

<details>
<summary>Day 18. VLANS - Part 2</summary>

| Command | Purpose |
|---|---|
| `interface <type> <num>` | Access interface |
| `switchport access vlan <id>` | Assigns VLAN to access port |

</details>

<details>
<summary>Day 19. VLANS - Part 3</summary>

| Command | Purpose |
|---|---|
| `switchport mode access` | Sets interface to access mode |
| `switchport trunk encapsulation dot1q` | Sets trunking to dot1q |
| `switchport mode trunk` | Sets trunk mode |

</details>

<details>
<summary>Day 20. DTP / VTP</summary>

| Command | Purpose |
|---|---|
| `vtp mode transparent/server/client` | Sets VTP mode |
| `vtp domain <name>` | Sets VTP domain |
| `vtp password <pass>` | Sets VTP password |

</details>

<details>
<summary>Day 21. Spanning Tree Protocol - Part 1</summary>

| Command | Purpose |
|---|---|
| `spanning-tree vlan <id> priority <value>` | Set priority for STP root |
| `show spanning-tree` | Displays spanning-tree info |

</details>

<details>
<summary>Day 22. Spanning Tree Protocol - Part 2</summary>

| Command | Purpose |
|---|---|
| `spanning-tree portfast` | Enables fast transition for ports |
| `spanning-tree bpduguard enable` | Protects STP topology from loops |

</details>

<details>
<summary>Day 23. Rapid Spanning Tree Protocol</summary>

| Command | Purpose |
|---|---|
| `spanning-tree mode rapid-pvst` | Enables RSTP mode |

</details>

<details>
<summary>Day 24. Etherchannel</summary>

| Command | Purpose |
|---|---|
| `interface range fa0/1 - 2` | Select multiple interfaces |
| `channel-group 1 mode active` | Creates EtherChannel using LACP |
| `show etherchannel summary` | View EtherChannel status |

</details>

<details>
<summary>Day 25. Dynamic Routing</summary>

| Command | Purpose |
|---|---|
| `router rip` | Enters RIP configuration |
| `router eigrp <asn>` | Enters EIGRP configuration |
| `router ospf <pid>` | Enters OSPF config |

</details>

<details>
<summary>Day 26. RIP / EIGRP</summary>

| Command | Purpose |
|---|---|
| `network <network>` | Advertise network in dynamic protocol |
| `no auto-summary` | Disable automatic summarization |

</details>

<details>
<summary>Day 27. OSPF - Part 1</summary>

| Command | Purpose |
|---|---|
| `router ospf <id>` | Starts OSPF config |
| `network <ip> <wildcard> area <id>` | Assign network to OSPF area |

</details>

<details>
<summary>Day 28. OSPF - Part 2</summary>

| Command | Purpose |
|---|---|
| `show ip ospf neighbor` | Displays OSPF neighbors |

</details>

<details>
<summary>Day 29. OSPF - Part 3</summary>

| Command | Purpose |
|---|---|
| `passive-interface default` | Prevents OSPF hellos on all interfaces |
| `no passive-interface <intf>` | Allows hello on selected interface |

</details>

<details>
<summary>Day 30. First Hop Redundancy Protocol</summary>

| Command | Purpose |
|---|---|
| `standby <group> ip <ip>` | Configures HSRP virtual IP |
| `show standby` | Displays HSRP status |

</details>

<details>
<summary>Day 31. TCP and UDP</summary>

| Command | Purpose |
|---|---|
| `show ip protocols` | Shows active routing protocols and ports |
| `show tcp brief` | Displays TCP connections |

</details>

<details>
<summary>Day 32. IPv6 - Part 1</summary>

| Command | Purpose |
|---|---|
| `ipv6 unicast-routing` | Enables IPv6 routing |
| `ipv6 address <addr>/<prefix>` | Assigns IPv6 to interface |

</details>

<details>
<summary>Day 33. IPv6 - Part 2</summary>

| Command | Purpose |
|---|---|
| `router ospf <id> ipv6` | Enable OSPFv3 for IPv6 |

</details>

<details>
<summary>Day 34. IPv6 - Part 3</summary>

| Command | Purpose |
|---|---|
| `show ipv6 route` | View IPv6 routing table |
| `ping ipv6 <addr>` | Ping using IPv6 |

</details>

