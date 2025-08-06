# Cisco CLI Command Reference

This repository provides a curated list of Cisco IOS commands aligned with an extended 63-day CCNA study plan. Use it as a quick study reference and starting point for hands-on practice.

## How to Use
- The master catalog lives in [Cisco_CLI_Commands.md](Cisco_CLI_Commands.md).
- Each topic section lists commands with their short forms, purpose, and the impact of omitting them.
- Topics beyond the first portion of the course may still need additional commands; contributions are welcome.

## Topic Index
1. [Networking Devices](Cisco_CLI_Commands.md#day-1-networking-devices)
2. [Interfaces and Cables](Cisco_CLI_Commands.md#day-2-interfaces-and-cables)
3. [OSI Model and TCP/IP Suite](Cisco_CLI_Commands.md#day-3-osi-model-and-tcp-ip-suite)
4. [Intro to the CLI](Cisco_CLI_Commands.md#day-4-intro-to-the-cli)
5. [Ethernet LAN Switching - Part 1](Cisco_CLI_Commands.md#day-5-ethernet-lan-switching-part-1)
6. [Ethernet LAN Switching - Part 2](Cisco_CLI_Commands.md#day-6-ethernet-lan-switching-part-2)
7. [IPv4 Addressing - Part 1](Cisco_CLI_Commands.md#day-7-ipv4-addressing-part-1)
8. [IPv4 Addressing - Part 2](Cisco_CLI_Commands.md#day-8-ipv4-addressing-part-2)
9. [Switch Interfaces](Cisco_CLI_Commands.md#day-9-switch-interfaces)
10. [The IPv4 Header](Cisco_CLI_Commands.md#day-10-the-ipv4-header)
11. [Routing Fundamentals - Part 1](Cisco_CLI_Commands.md#day-11-routing-fundamentals-part-1)
12. [Static Routing - Part 2](Cisco_CLI_Commands.md#day-11-static-routing-part-2)
13. [Life of a Packet](Cisco_CLI_Commands.md#day-12-life-of-a-packet)
14. [Subnetting - Part 1](Cisco_CLI_Commands.md#day-13-subnetting-part-1)
15. [Subnetting - Part 2](Cisco_CLI_Commands.md#day-14-subnetting-part-2)
16. [Subnetting (VLSM) - Part 3](Cisco_CLI_Commands.md#day-15-subnetting-vlsm-part-3)
17. [VLANs - Part 1](Cisco_CLI_Commands.md#day-16-vlans-part-1)
18. [VLANs - Part 2](Cisco_CLI_Commands.md#day-17-vlans-part-2)
19. [VLANs - Part 3](Cisco_CLI_Commands.md#day-18-vlans-part-3)
20. [DTP / VTP](Cisco_CLI_Commands.md#day-19-dtp-vtp)
21. [Spanning Tree Protocol - Part 1](Cisco_CLI_Commands.md#day-20-spanning-tree-protocol-part-1)
22. [Spanning Tree Protocol - Part 2](Cisco_CLI_Commands.md#day-21-spanning-tree-protocol-part-2)
23. [Rapid Spanning Tree Protocol](Cisco_CLI_Commands.md#day-22-rapid-spanning-tree-protocol)
24. [EtherChannel](Cisco_CLI_Commands.md#day-23-etherchannel)
25. [Dynamic Routing](Cisco_CLI_Commands.md#day-24-dynamic-routing)
26. [RIP / EIGRP](Cisco_CLI_Commands.md#day-25-rip-eigrp)
27. [OSPF - Part 1](Cisco_CLI_Commands.md#day-26-ospf-part-1)
28. [OSPF - Part 2](Cisco_CLI_Commands.md#day-27-ospf-part-2)
29. [OSPF - Part 3](Cisco_CLI_Commands.md#day-28-ospf-part-3)
30. [First Hop Redundancy Protocol](Cisco_CLI_Commands.md#day-29-first-hop-redundancy-protocol)
31. [TCP and UDP](Cisco_CLI_Commands.md#day-30-tcp-and-udp)
32. [IPv6 - Part 1](Cisco_CLI_Commands.md#day-31-ipv6-part-1)
33. [IPv6 - Part 2](Cisco_CLI_Commands.md#day-32-ipv6-part-2)
34. [IPv6 - Part 3](Cisco_CLI_Commands.md#day-33-ipv6-part-3)
35. [Standard Access Control List](Cisco_CLI_Commands.md#day-34-standard-access-control-list)
36. [Extended Access Control List](Cisco_CLI_Commands.md#day-35-extended-access-control-list)
37. [CDP and LLDP](Cisco_CLI_Commands.md#day-36-cdp-and-lldp)
38. [NTP - Network Time Protocol](Cisco_CLI_Commands.md#day-37-ntp-network-time-protocol)
39. [DNS - Domain Name System](Cisco_CLI_Commands.md#day-38-dns-domain-name-system)
40. [DHCP - Dynamic Host Configuration Protocol](Cisco_CLI_Commands.md#day-39-dhcp-dynamic-host-configuration-protocol)
41. [SNMP - Simple Network Management Protocol](Cisco_CLI_Commands.md#day-40-snmp-simple-network-management-protocol)
42. [SYSLOG](Cisco_CLI_Commands.md#day-41-syslog)
43. [SSH - Secure Shell](Cisco_CLI_Commands.md#day-42-ssh-secure-shell)
44. [FTP and TFTP](Cisco_CLI_Commands.md#day-43-ftp-and-tftp)
45. [NAT (Static) - Part 1](Cisco_CLI_Commands.md#day-44-nat-static-part-1)
46. [NAT (Dynamic) - Part 2](Cisco_CLI_Commands.md#day-45-nat-dynamic-part-2)
47. [QoS (Voice VLAN) - Part 1](Cisco_CLI_Commands.md#day-46-qos-voice-vlan-part-1)
48. [QoS (Quality of Service) - Part 2](Cisco_CLI_Commands.md#day-47-qos-quality-of-service-part-2)
49. [Security Fundamentals](Cisco_CLI_Commands.md#day-48-security-fundamentals)
50. [Port Security](Cisco_CLI_Commands.md#day-49-port-security)
51. [DHCP Snooping](Cisco_CLI_Commands.md#day-50-dhcp-snooping)
52. [Dynamic ARP Inspection](Cisco_CLI_Commands.md#day-51-dynamic-arp-inspection)
53. [LAN Architectures](Cisco_CLI_Commands.md#day-52-lan-architectures)
54. [WAN Architectures](Cisco_CLI_Commands.md#day-53-wan-architectures)
55. [Virtualization and Cloud - Part 1](Cisco_CLI_Commands.md#day-54-virtualization-and-cloud-part-1)
56. [Virtualization (Containers) - Part 2](Cisco_CLI_Commands.md#day-54-virtualization-containers-part-2)
57. [Virtualization (VRF) - Part 3](Cisco_CLI_Commands.md#day-54-virtualization-vrf-part-3)
58. [Wireless Fundamentals](Cisco_CLI_Commands.md#day-55-wireless-fundamentals)
59. [Wireless Architectures](Cisco_CLI_Commands.md#day-56-wireless-architectures)
60. [Wireless Security](Cisco_CLI_Commands.md#day-57-wireless-security)
61. [Wireless Configuration](Cisco_CLI_Commands.md#day-58-wireless-configuration)
62. [Introduction to Network Automation](Cisco_CLI_Commands.md#day-59-introduction-to-network-automation)
63. [JSON, XML, and YAML](Cisco_CLI_Commands.md#day-60-json-xml-and-yaml)
64. [REST APIs](Cisco_CLI_Commands.md#day-61-rest-apis)
65. [SOFTWARE DEFINED NETWORKING](Cisco_CLI_Commands.md#day-62-software-defined-networking)
66. [ANSIBLE, PUPPET, and CHEF](Cisco_CLI_Commands.md#day-63-ansible-puppet-and-chef)

## Contributing
1. Edit `Cisco_CLI_Commands.md` for any new commands or corrections.
2. Run `pytest` to ensure consistency.
3. Open a pull request describing your additions.

## License
This project is released under the MIT License.
