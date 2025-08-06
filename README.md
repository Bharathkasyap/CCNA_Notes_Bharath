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

<details>
<summary>Day 34. Standard Access Control List</summary>

| Command | Purpose |
|---|---|
| `access-list 10 permit 192.168.1.0 0.0.0.255` | Create standard ACL to permit a subnet |
| `interface <intf>` | Enter interface |
| `ip access-group 10 in` | Apply ACL to interface inbound |

</details>

<details>
<summary>Day 35. Extended Access Control List</summary>

| Command | Purpose |
|---|---|
| `access-list 101 permit tcp 192.168.1.0 0.0.0.255 any eq 80` | Permit HTTP from subnet |
| `ip access-group 101 out` | Apply ACL outbound |

</details>

<details>
<summary>Day 36. CDP and LLDP</summary>

| Command | Purpose |
|---|---|
| `show cdp neighbors` | View Cisco neighbors |
| `show lldp neighbors` | View LLDP neighbors |

</details>

<details>
<summary>Day 37. NTP - Network Time Protocol</summary>

| Command | Purpose |
|---|---|
| `ntp server <ip>` | Set NTP server |
| `show ntp status` | Verify NTP sync |

</details>

<details>
<summary>Day 38. DNS - Domain Name System</summary>

| Command | Purpose |
|---|---|
| `ip name-server <ip>` | Set DNS server |
| `ping www.cisco.com` | Test DNS resolution |

</details>

<details>
<summary>Day 39. DHCP - Dynamic Host Configuration Protocol</summary>

| Command | Purpose |
|---|---|
| `ip dhcp pool <name>` | Create DHCP pool |
| `network 192.168.1.0 255.255.255.0` | Define network |
| `default-router 192.168.1.1` | Set default gateway |

</details>

<details>
<summary>Day 40. SNMP - Simple Network Management Protocol</summary>

| Command | Purpose |
|---|---|
| `snmp-server community public RO` | Configure SNMP read-only community |
| `show snmp` | Displays SNMP status |

</details>

<details>
<summary>Day 41. SYSLOG</summary>

| Command | Purpose |
|---|---|
| `logging <server-ip>` | Set Syslog server |
| `logging trap warnings` | Set logging level |

</details>

<details>
<summary>Day 42. SSH - Secure Shell</summary>

| Command | Purpose |
|---|---|
| `hostname Router` | Set hostname |
| `ip domain-name local` | Set domain name |
| `crypto key generate rsa` | Generate RSA keys |
| `username admin password cisco` | Create user |
| `line vty 0 4` | Configure SSH access |

</details>

<details>
<summary>Day 43. FTP and TFTP</summary>

| Command | Purpose |
|---|---|
| `copy running-config tftp` | Backup config to TFTP |
| `copy tftp running-config` | Restore config from TFTP |

</details>

<details>
<summary>Day 44. NAT (Static) - Part 1</summary>

| Command | Purpose |
|---|---|
| `ip nat inside source static 192.168.1.10 203.0.113.10` | Configure static NAT |

</details>

<details>
<summary>Day 45. NAT (Dynamic) - Part 2</summary>

| Command | Purpose |
|---|---|
| `ip nat pool MYPOOL 203.0.113.100 203.0.113.110 netmask 255.255.255.0` | Define NAT pool |
| `access-list 1 permit 192.168.1.0 0.0.0.255` | Match inside network |
| `ip nat inside source list 1 pool MYPOOL` | Apply dynamic NAT |

</details>

<details>
<summary>Day 46. QoS (Voice VLAN) - Part 1</summary>

| Command | Purpose |
|---|---|
| `mls qos` | Enable QoS globally |
| `switchport voice vlan 10` | Assign voice VLAN to port |

</details>

<details>
<summary>Day 47. QoS (Quality of Service) - Part 2</summary>

| Command | Purpose |
|---|---|
| `priority-queue out` | Enable priority queuing |
| `mls qos trust dscp` | Trust DSCP values on port |

</details>

<details>
<summary>Day 48. Security Fundamentals</summary>

| Command | Purpose |
|---|---|
| `service password-encryption` | Encrypt passwords |
| `enable secret <pwd>` | Set secure privileged exec password |

</details>

<details>
<summary>Day 49. Port Security</summary>

| Command | Purpose |
|---|---|
| `switchport port-security` | Enable port security |
| `switchport port-security maximum 1` | Limit MAC addresses |
| `switchport port-security violation shutdown` | Shutdown on violation |

</details>

<details>
<summary>Day 50. DHCP Snooping</summary>

| Command | Purpose |
|---|---|
| `ip dhcp snooping` | Enable DHCP snooping |
| `ip dhcp snooping vlan 1` | Enable for VLAN 1 |
| `ip dhcp snooping trust` | Trust port (uplink) |

</details>

<details>
<summary>Day 51. Dynamic Arp Inspection</summary>

| Command | Purpose |
|---|---|
| `ip arp inspection vlan 1` | Enable DAI for VLAN 1 |
| `ip arp inspection trust` | Trust specific port |

</details>

<details>
<summary>Day 52. LAN Architectures</summary>

| Command | Purpose |
|---|---|
| `show vlan brief` | Displays VLAN info |
| `show interfaces status` | Displays interface and speed info |

</details>

<details>
<summary>Day 53. WAN Architectures</summary>

| Command | Purpose |
|---|---|
| `show ip interface` | Displays IP and encapsulation on interfaces |
| `encapsulation ppp` | Set PPP on serial interface |

</details>

<details>
<summary>Day 54. Virtualization and Cloud - Part1</summary>

| Command | Purpose |
|---|---|
| `show version` | Confirms platform and virtualization support |
| `show platform` | Displays hardware capabilities |

</details>

<details>
<summary>Day 55. Virtualization (Containers) - Part 2</summary>

| Command | Purpose |
|---|---|
| `docker ps` | Lists running containers (Cisco IOx environments) |

</details>

<details>
<summary>Day 56. Virtualization (VRF) - Part3</summary>

| Command | Purpose |
|---|---|
| `ip vrf <name>` | Creates VRF |
| `ip route vrf <name> <dest> <mask> <next-hop>` | Routing inside VRF |

</details>

<details>
<summary>Day 57. Wireless Fundamentals</summary>

| Command | Purpose |
|---|---|
| `show controllers dot11Radio 0` | Wireless radio details |
| `show wlan summary` | List of WLANs |

</details>

<details>
<summary>Day 58. Wireless Architectures</summary>

| Command | Purpose |
|---|---|
| `show wireless stats` | Wireless client and AP statistics |

</details>

<details>
<summary>Day 59. Wireless Security</summary>

| Command | Purpose |
|---|---|
| `security wpa psk set-key ascii 0 <key>` | Set WPA key |

</details>

<details>
<summary>Day 60. Wireless Configuration</summary>

| Command | Purpose |
|---|---|
| `interface dot11Radio 0` | Access wireless interface |
| `ssid <name>` | Create SSID |
| `authentication open` | Set auth type |

</details>

<details>
<summary>Day 61. Introduction to Network Automation</summary>

| Command | Purpose |
|---|---|
| `show version | json` | Show output in JSON (NX-OS) |
| `restconf` | Enable RESTCONF on modern IOS XE |

</details>

<details>
<summary>Day 62. JSON, XML, and YAML</summary>

| Command | Purpose |
|---|---|
| `show interfaces | include json` | Use JSON output format if supported |

</details>

<details>
<summary>Day 63. REST APIs</summary>

| Command | Purpose |
|---|---|
| `ip http server` | Enable HTTP server |
| `ip http secure-server` | Enable HTTPS server |

</details>

<details>
<summary>Day 64. SOFTWARE DEFINED NETWORKING</summary>

| Command | Purpose |
|---|---|
| `sdm prefer lanbase-routing` | Optimize for SDN-like routing |
| `show license` | Show SDN or DNA licensing |

</details>

<details>
<summary>Day 65. ANSIBLE, PUPPET, and CHEF</summary>

| Command | Purpose |
|---|---|
| `netconf-yang` | Enable NETCONF/YANG for automation |
| `show management api` | Verify APIs for automation tools |

</details>

