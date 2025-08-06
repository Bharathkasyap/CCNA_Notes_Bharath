# Cisco CLI Command Reference

This document summarizes key Cisco IOS commands aligned with the early portion of a 63‑day CCNA study plan. For each topic, representative commands are listed with an abbreviated form, purpose, and impact if omitted.

## Day 1. Networking Devices
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show version | sh ver | Display hardware/software details. | Hard to confirm device capabilities and versions. |
| show ip interface brief | sh ip int br | Summarize interface states and IPs. | Interface troubleshooting becomes slower. |
| enable | en | Enter privileged EXEC mode. | Cannot access configuration or advanced show commands. |
| configure terminal | conf t | Enter global configuration mode. | Cannot modify running configuration. |
| hostname <name> | host | Set device name for identification. | Device keeps default name, causing confusion. |

## Day 2. Interfaces and Cables
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| interface <type> <number> | int | Select an interface to configure. | Unable to apply settings to interfaces. |
| description <text> | desc | Document interface purpose. | Interface role may be unclear. |
| speed {10 \| 100 \| 1000} | speed | Set interface speed. | Auto‑negotiation may mismatch speeds. |
| duplex {auto \| full \| half} | duplex | Configure duplex mode. | Auto settings may cause collisions. |
| show interfaces | sh int | Detailed interface statistics. | Hard to troubleshoot physical issues. |

## Day 3. OSI Model and TCP/IP Suite
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show protocols | sh prot | Verify Layer‑3 protocol status. | Layer‑3 misconfigurations may go unnoticed. |
| ping <address> | ping | Test network reachability. | Connectivity issues harder to diagnose. |
| traceroute <address> | tracer | Display path to destination. | Cannot locate routing path problems. |
| show ip route | sh ip ro | Examine IP routing table. | Unknown or incorrect routes remain hidden. |

## Day 4. Intro to the CLI
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ? | ? | Context‑sensitive help. | Hard to learn syntax and options. |
| tab | tab | Auto‑complete commands. | Typing errors increase. |
| history | history | Display command history. | Re‑typing commands becomes tedious. |
| terminal length 0 | ter len 0 | Disable paging for long outputs. | Output pauses every 24 lines. |

## Day 5. Ethernet LAN Switching - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show mac address-table | sh mac add | View learned MAC addresses. | Hard to verify MAC learning and forwarding. |
| switchport mode access | sw mode acc | Set port as access port. | Port may negotiate as trunk causing VLAN leaks. |
| spanning-tree portfast | sp portfast | Accelerate host port transition. | Hosts wait for STP states, delaying connectivity. |
| no shutdown | no shut | Enable interface. | Interface remains administratively down. |

## Day 6. Ethernet LAN Switching - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| switchport mode trunk | sw mode tru | Set port as trunk. | VLAN tags not carried. |
| switchport trunk allowed vlan <list> | sw tru allow | Restrict VLANs on trunk. | All VLANs pass, exposing unnecessary traffic. |
| switchport trunk native vlan <id> | sw tru nat | Define native VLAN. | Mismatched native VLANs may cause loops. |
| show interfaces trunk | sh int tru | Validate trunk status. | Trunk misconfigurations remain undetected. |

## Day 7. IPv4 Addressing - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip address <address> <mask> | ip add | Assign IPv4 address to interface. | Interface cannot participate in Layer‑3 communication. |
| no shutdown | no shut | Activate L3 interface. | Interface stays down. |
| show ip interface | sh ip int | Detailed IPv4 interface status. | Hard to confirm addressing and ACLs. |

## Day 8. IPv4 Addressing - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip default-gateway <address> | ip def-gw | Set default gateway on L2 switch. | Switch cannot reach other subnets. |
| ip address dhcp | ip add dhcp | Obtain IP from DHCP server. | Interface requires manual IP configuration. |
| show ip dhcp binding | sh ip dhcp bind | Display DHCP address allocations. | DHCP assignments are unclear. |

## Day 9. Switch Interfaces
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show controllers | sh controllers | Display hardware controller info. | Physical issues may not be diagnosed. |
| show interface status | sh int status | Summary of port status and VLAN. | Hard to verify port quickly. |
| errdisable recovery cause <cause> | err rec | Re‑enable ports disabled by errors. | Ports stay down requiring manual intervention. |

## Day 10. The IPv4 Header
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| debug ip packet | deb ip pack | Inspect IPv4 packet processing. | Detailed packet info unavailable. |
| undebug all | u all | Disable all debugging. | Debug continues, consuming resources. |
| show ip traffic | sh ip traff | Summarize IP packet statistics. | Lack visibility into protocol counters. |

## Day 11. Routing Fundamentals - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip routing | ip ro | Enable IP routing on L3 switch. | Device cannot route between subnets. |
| show ip route | sh ip ro | Display routing table. | Routing issues difficult to diagnose. |
| show cdp neighbors | sh cdp nei | Discover directly connected Cisco devices. | Topology discovery is manual. |

## Day 11. Static Routing - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip route <prefix> <mask> <next-hop> | ip ro | Create static route. | Traffic may not reach remote networks. |
| ip route 0.0.0.0 0.0.0.0 <next-hop> | ip ro 0 | Configure default route. | Unknown traffic is dropped. |
| show running-config | include ip route | sh run | inc ip ro | Verify static routes in configuration. | Misconfigurations remain hidden. |

## Day 12. Life of a Packet
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| debug ip packet detail | deb ip pack det | Track packet forwarding decisions. | Hard to follow packet path. |
| show processes cpu | sh proc cpu | Monitor CPU during testing. | High CPU usage from debugging may go unnoticed. |
| show interfaces switching | sh int switching | Examine packet forwarding methods. | Less insight into CEF vs process switching. |

## Day 13. Subnetting - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip subnet-zero | ip sub 0 | Allow use of first subnet. | Network wastes two subnets. |
| show ip interface brief | sh ip int br | Verify subnet assignments quickly. | Misassigned IPs go undetected. |
| show ip route connected | sh ip ro con | Display connected routes only. | Connected networks list is cluttered. |

## Day 14. Subnetting - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip classless | ip classless | Enable classless routing. | Router drops packets for unknown subnets. |
| show ip route static | sh ip ro sta | Display static routes only. | Harder to verify static design. |
| show ip protocols | sh ip prot | Examine routing protocol parameters. | Misconfigurations may persist. |

## Day 15. Subnetting (VLSM) - Part 3
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip route <subnet> <mask> <next-hop> | ip ro | Create routes for VLSM subnets. | VLSM design cannot be implemented. |
| summary-address <prefix> <mask> | sum-add | Aggregate routes on interface. | Routing table may be large. |
| show ip route summary | sh ip ro sum | Summarize routing table entries. | Harder to review VLSM efficiency. |

## Day 16. VLANs - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| vlan <id> | vlan | Create VLAN in database. | Cannot assign ports to new VLANs. |
| name <vlan-name> | name | Name VLAN for clarity. | VLANs identified only by numbers. |
| show vlan brief | sh vlan br | List VLANs and ports. | VLAN assignments are unclear. |

## Day 17. VLANs - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| switchport access vlan <id> | sw acc vlan | Assign access port to VLAN. | Port remains in VLAN 1. |
| switchport voice vlan <id> | sw voice | Configure voice VLAN for IP phones. | Voice/data share same VLAN. |
| show interfaces switchport | sh int sw | Display interface VLAN details. | Misassigned ports remain undetected. |

## Day 18. VLANs - Part 3
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| switchport nonegotiate | sw noneg | Disable DTP on trunk. | Unwanted trunk negotiations may occur. |
| vtp mode {server \| client \| transparent} | vtp mo | Control VTP role. | VLAN database may not sync. |
| vtp domain <name> | vtp dom | Set VTP domain name. | VTP advertisements ignored. |

## Day 19. DTP / VTP
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show dtp interface | sh dtp int | Check DTP negotiation status. | DTP issues may be missed. |
| vtp password <password> | vtp pass | Secure VTP updates. | Unauthorized VTP changes can occur. |
| show vtp status | sh vtp stat | Display VTP configuration and stats. | Difficult to verify synchronization. |

## Day 20. Spanning Tree Protocol - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| spanning-tree vlan <id> | sp vlan | Configure STP per VLAN. | Default STP settings may not suit topology. |
| spanning-tree mode {pvst \| rapid-pvst} | sp mode | Choose STP variant. | Device uses default PVST+, possibly suboptimal. |
| show spanning-tree | sh sp | View STP state. | STP topology issues go unnoticed. |

## Day 21. Spanning Tree Protocol - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| spanning-tree portfast | sp pf | Enables PortFast on access ports. | Edge ports take 30 seconds to forward, delaying host connectivity. |
| spanning-tree bpduguard enable | sp bpdug en | Protects PortFast ports from BPDUs. | Misconnected switches can form loops. |
| spanning-tree guard root | sp gr rt | Prevents the port from becoming root port. | A rogue switch could take over as root. |

## Day 22. Rapid Spanning Tree Protocol
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| spanning-tree mode rapid-pvst | sp mode rapid | Enables RSTP per-VLAN. | Device stays on PVST+, converging slower. |
| show spanning-tree detail | sh sp det | Displays RSTP port roles and timers. | Detailed RSTP status unavailable. |
| spanning-tree link-type point-to-point | sp link p2p | Sets port link type for RSTP. | Port may default to shared, slowing convergence. |

## Day 23. EtherChannel
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| interface range <range> | int ra | Selects multiple interfaces. | Each port must be configured separately. |
| channel-group <num> mode active | ch-gr act | Creates LACP EtherChannel. | Links remain independent and may loop. |
| show etherchannel summary | sh eth sum | Verifies EtherChannel status. | Bundling issues may go unnoticed. |

## Day 24. Dynamic Routing
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| router <protocol> | router | Enters routing-process config. | Cannot enable dynamic routing. |
| network <address> <wildcard> | net | Advertises networks in a protocol. | Routes are not shared with neighbors. |
| show ip protocols | sh ip prot | Lists active routing protocols. | Hard to confirm routing-process settings. |

## Day 25. RIP / EIGRP
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| router rip | router rip | Starts RIP process. | No RIP updates sent or received. |
| router eigrp <asn> | router eigrp | Starts EIGRP process. | No EIGRP neighbors form. |
| no auto-summary | no auto | Disables classful summarization. | Incorrect major-network routes may appear. |

## Day 26. OSPF - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| router ospf <process-id> | router ospf | Enables OSPF routing. | OSPF neighbors never form. |
| network <ip> <wildcard> area <id> | net ... area | Advertises networks into OSPF area. | Subnets stay unreachable to peers. |
| show ip ospf neighbor | sh ospf nei | Displays OSPF adjacency states. | Neighbor problems harder to detect. |

## Day 27. OSPF - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| area <id> stub | area stub | Configures stub area. | Extra LSAs may consume bandwidth. |
| ip ospf cost <value> | ip ospf cost | Adjusts OSPF interface cost. | Traffic may not use desired path. |
| show ip ospf interface | sh ospf int | Shows OSPF interface parameters. | Interface role and timers unclear. |

## Day 28. OSPF - Part 3
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| router-id <id> | rid | Manually sets OSPF router ID. | Automatic ID may change after reload. |
| passive-interface default | passive def | Suppresses OSPF on all interfaces. | Unwanted adjacencies may form. |
| default-information originate | def-info orig | Injects default route. | Peers lack a route to unknown networks. |

## Day 29. First Hop Redundancy Protocol
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| standby <grp> ip <addr> | stb ip | Configures HSRP virtual IP. | Hosts lack a gateway for redundancy. |
| standby <grp> priority <val> | stb pri | Sets HSRP priority. | Active router election uncontrollable. |
| show standby | sh stb | Displays HSRP state. | Failover status cannot be verified. |

## Day 30. TCP and UDP
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show control-plane host open-ports | sh cph ports | Lists listening TCP/UDP ports. | Hard to audit exposed services. |
| ip tcp adjust-mss <bytes> | tcp mss | Tweaks TCP MSS to avoid fragmentation. | Large segments may be dropped. |
| ip udp small-servers | udp ss | Enables built-in UDP test servers. | Cannot test simple UDP services. |

## Day 31. IPv6 - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ipv6 unicast-routing | ipv6 ur | Globally enables IPv6 routing. | Device only forwards IPv4 traffic. |
| interface <type> <num> | int | Enters interface config. | Cannot apply IPv6 settings. |
| ipv6 address <addr>/<prefix> | ipv6 addr | Assigns IPv6 address. | Interface lacks IPv6 connectivity. |

## Day 32. IPv6 - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ipv6 address <prefix> eui-64 | ipv6 eui | Generates interface ID automatically. | Manual addressing required for uniqueness. |
| show ipv6 interface brief | sh ipv6 int br | Summarizes IPv6 interfaces. | Hard to verify addressing at a glance. |
| ipv6 router ospf <pid> | ipv6 ospf | Starts OSPFv3 process. | OSPFv3 neighbors never form. |

## Day 33. IPv6 - Part 3
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ipv6 ospf <pid> area <id> | ipv6 ospf area | Enables OSPFv3 on interface. | Interface not advertised in OSPFv3. |
| ipv6 route ::/0 <next-hop> | ipv6 rt :: | Sets IPv6 default route. | Router lacks path to unknown networks. |
| show ipv6 neighbors | sh ipv6 nei | Displays IPv6 neighbor cache. | Duplicate-address or reachability issues go unseen. |

## Day 34. Standard Access Control List
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| access-list <1-99> permit <source> | acl perm | Creates standard ACL entry. | Traffic filtering not applied. |
| interface <type> <num> | int | Selects interface. | Cannot apply ACL to traffic. |
| ip access-group <acl> {in \| out} | ip acc-gr | Binds ACL to interface. | ACL rules never take effect. |

## Day 35. Extended Access Control List
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip access-list extended <name> | ip acl ext | Enters named extended ACL mode. | Fine-grained filtering impossible. |
| permit tcp <src> <dst> eq <port> | perm tcp | Allows TCP traffic on a port. | Desired TCP sessions blocked. |
| ip access-group <name> {in \| out} | ip acc-gr | Applies ACL to interface. | ACL rules not enforced. |

## Day 36. CDP and LLDP
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| cdp run | cdp run | Enables Cisco Discovery Protocol globally. | CDP neighbor info unavailable. |
| show cdp neighbors detail | sh cdp nei det | Lists detailed CDP data. | Troubleshooting neighbor devices harder. |
| lldp run | lldp run | Enables LLDP globally. | LLDP neighbor information absent. |

## Day 37. NTP - Network Time Protocol
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ntp server <ip> | ntp serv | Points device to NTP server. | Clock may drift, affecting logs and security. |
| clock timezone <zone> <offset> | clk tz | Sets local time zone. | Displayed times use default UTC. |
| show ntp status | sh ntp stat | Verifies NTP sync state. | Difficult to confirm time accuracy. |

## Day 38. DNS - Domain Name System
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip domain-lookup | dom look | Enables DNS lookups. | Hostnames cannot be resolved. |
| ip name-server <ip> | name-serv | Specifies DNS servers. | Device has nowhere to send DNS queries. |
| show hosts | sh hosts | Displays cached DNS entries. | Hard to verify resolved names. |

## Day 39. DHCP - Dynamic Host Configuration Protocol
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip dhcp pool <name> | dhcp pool | Creates DHCP scope. | No addresses are handed out. |
| network <net> <mask> | net | Defines subnet for pool. | Clients receive no valid addresses. |
| default-router <ip> | def-rtr | Sets gateway for clients. | Clients lack default route. |

## Day 40. SNMP - Simple Network Management Protocol
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| snmp-server community <str> ro | snmp com ro | Defines read-only community string. | Managers cannot poll device. |
| snmp-server host <ip> <str> | snmp host | Specifies trap destination. | Traps never reach management host. |
| show snmp | sh snmp | Displays SNMP configuration. | SNMP status is unknown. |

## Day 41. SYSLOG
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| logging <ip> | log ip | Sets remote syslog server. | Logs remain only locally. |
| logging trap <level> | log trap | Sets severity level for remote logging. | Too many or too few messages sent. |
| show logging | sh log | Displays log buffer contents. | Troubleshooting events harder. |

## Day 42. SSH - Secure Shell
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip domain-name <name> | dom-name | Sets domain for RSA key generation. | RSA keys cannot be created. |
| crypto key generate rsa modulus <bits> | crypto key gen | Generates RSA keys for SSH. | SSH server remains disabled. |
| line vty 0 4 | line vty | Enters VTY line configuration. | Cannot apply SSH-only settings. |
| transport input ssh | trans ssh | Restricts VTY access to SSH. | Telnet or no remote access allowed. |
## Day 43. FTP and TFTP
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip ftp username <user> | ftp user | Sets default FTP username. | Manual username needed each session. |
| ip ftp password <pass> | ftp pass | Sets default FTP password. | Password must be typed each time. |
| copy running-config tftp: | cp run tftp | Saves config to TFTP server. | Backups require different method. |

## Day 44. NAT (Static) - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip nat inside source static <local> <global> | nat stat | Creates one-to-one static mapping. | Inside host not reachable externally. |
| interface <type> <num> \n ip nat inside | int / nat in | Marks interface as inside. | NAT translations fail for that side. |
| interface <type> <num> \n ip nat outside | int / nat out | Marks interface as outside. | NAT translations fail for that side. |

## Day 45. NAT (Dynamic) - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip nat pool <name> <start> <end> netmask <mask> | nat pool | Defines address pool. | No addresses available for translation. |
| access-list <num> permit <source> | acl perm | Identifies inside traffic for NAT. | Packets not matched for translation. |
| ip nat inside source list <num> pool <name> overload | nat overload | Enables PAT using pool. | Multiple hosts share one address without overloading impossible. |

## Day 46. QoS (Voice VLAN) - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| switchport voice vlan <vlan> | sw voice | Assigns voice VLAN to access port. | Phones share data VLAN causing QoS issues. |
| mls qos trust cos | qos tr cos | Trusts CoS markings on port. | Upstream switches may rewrite voice priority. |
| show mls qos interface <int> | sh qos int | Displays QoS settings on interface. | Voice marking problems go unseen. |

## Day 47. QoS (Quality of Service) - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| class-map <name> | class-map | Defines traffic class. | Policy cannot identify traffic. |
| policy-map <name> | policy-map | Creates QoS policy. | No actions defined for classes. |
| service-policy output <name> | serv-pol out | Attaches policy to interface. | QoS rules never take effect. |

## Day 48. Security Fundamentals
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| enable secret <password> | en sec | Sets encrypted enable password. | Privileged access may be unprotected. |
| service password-encryption | svc pass-encr | Encrypts plain-text passwords. | Config passwords visible to all. |
| login block-for <sec> attempts <count> within <sec> | log blk | Thwarts brute-force logins. | Repeated login attempts allowed. |

## Day 49. Port Security
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| switchport port-security | sw ps | Enables port security on interface. | Unlimited MACs allowed on port. |
| switchport port-security maximum <n> | sw ps max | Limits MAC addresses. | Excess devices may connect freely. |
| show port-security interface <int> | sh ps int | Displays port-security status. | Violations or learnings unseen. |

## Day 50. DHCP Snooping
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip dhcp snooping | dhcp snoop | Globally enables DHCP Snooping. | Rogue servers can hand out IPs. |
| ip dhcp snooping vlan <vlan> | dhcp snoop vlan | Protects specific VLANs. | VLANs remain vulnerable to rogue DHCP. |
| interface <type> <num> \n ip dhcp snooping trust | int / dhcp trust | Marks port trusted for DHCP. | Valid server traffic dropped. |

## Day 51. Dynamic ARP Inspection
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip arp inspection vlan <vlan> | arp insp vlan | Enables DAI on VLAN. | ARP spoofing not prevented. |
| interface <type> <num> \n ip arp inspection trust | int / arp trust | Marks port trusted. | Legit ARP replies may be dropped. |
| show ip arp inspection | sh arp insp | Displays DAI status. | DAI misconfigs harder to spot. |

## Day 52. LAN Architectures
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show vlan brief | sh vlan br | Lists VLANs and ports. | Hard to view layer-2 segmentation. |
| show spanning-tree root | sh sp root | Identifies STP root bridge. | Root location unknown. |
| show ip route | sh ip ro | Displays routing table. | Layer-3 paths unclear. |

## Day 53. WAN Architectures
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| interface serial <num> | int ser | Enters serial interface config. | Cannot set WAN-specific options. |
| encapsulation ppp | encap ppp | Configures PPP encapsulation. | Default HDLC used, limiting features. |
| show interfaces serial <num> | sh int ser | Shows WAN link status. | Difficult to troubleshoot WAN issues. |

## Day 54. Virtualization and Cloud - Part 1
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| virtual-service install name <app> package <url> | vs install | Loads a virtual service. | Service unavailable on router. |
| virtual-service start <app> | vs start | Starts installed service. | Virtual service remains stopped. |
| show virtual-service list | sh vs list | Displays installed services. | Hard to verify service status. |

## Day 54. Virtualization (Containers) - Part 2
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| iox | iox | Enables IOx container environment. | Cannot host containerized apps. |
| app-hosting install appid <id> package <url> | app inst | Installs a container app. | Application not available. |
| app-hosting start appid <id> | app start | Launches container app. | App remains offline. |

## Day 54. Virtualization (VRF) - Part 3
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| vrf definition <name> | vrf def | Creates VRF instance. | Segmented routing tables unavailable. |
| ip vrf forwarding <name> | ip vrf for | Binds interface to VRF. | Interface stays in global table. |
| show vrf | sh vrf | Lists VRF configurations. | VRF membership unclear. |

## Day 55. Wireless Fundamentals
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| dot11 ssid <name> | ssid | Defines wireless SSID. | WLAN cannot be broadcast. |
| interface wlan <id> | int wlan | Enters WLAN interface. | Cannot apply SSID to radio. |
| show wlan summary | sh wlan sum | Displays WLAN configurations. | WLAN setup verification difficult. |

## Day 56. Wireless Architectures
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ap name <AP> | ap name | Sets AP's name for management. | AP identification confusing. |
| capwap ap controller ip address <ip> | capwap ip | Points AP to WLC. | AP fails to join controller. |
| show ap summary | sh ap sum | Lists APs and status. | AP join issues unnoticed. |

## Day 57. Wireless Security
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| security wpa psk set-key ascii <key> | wpa key | Configures WPA2-PSK. | WLAN lacks encryption. |
| authentication open | auth open | Sets open authentication (used before key). | Authentication defaults may block access. |
| show wireless security summary | sh wlan sec | Displays wireless security settings. | Hard to verify WLAN protection. |

## Day 58. Wireless Configuration
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| wlan <id> <ssid> | wlan id | Creates WLAN on controller. | Clients cannot associate. |
| client vlan <vlan> | client vlan | Maps WLAN to VLAN. | Wireless clients placed in wrong subnet. |
| no shutdown | no sh | Activates WLAN or interface. | WLAN stays disabled. |

## Day 59. Introduction to Network Automation
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| guest-shell enable | gsh en | Activates Linux shell on device. | On-box scripting unavailable. |
| netconf-yang | netconf | Enables NETCONF/YANG agent. | Controllers cannot manage device programmatically. |
| restconf | restconf | Enables RESTCONF interface. | RESTful automation cannot occur. |

## Day 60. JSON, XML, and YAML
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| show running-config \| format json | sh run \| json | Outputs config in JSON. | Automation tools lack JSON data. |
| show running-config \| format xml | sh run \| xml | Outputs config in XML. | XML-based parsing impossible. |
| show platform software yang-management process | sh yang proc | Displays YANG process info. | YANG service issues hidden. |

## Day 61. REST APIs
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ip http secure-server | http ssl | Enables HTTPS server. | RESTCONF cannot operate securely. |
| restconf | restconf | Activates RESTCONF API. | REST API unreachable. |
| show restconf | sh restconf | Displays RESTCONF status. | API troubleshooting difficult. |

## Day 62. SOFTWARE DEFINED NETWORKING
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| sdn enable | sdn en | Turns on SDN features. | Device cannot be SDN-controlled. |
| controller <ip> | controller | Specifies SDN controller. | Device fails to connect to controller. |
| show sdn controller | sh sdn ctrl | Shows controller connection status. | SDN connectivity issues unseen. |

## Day 63. ANSIBLE, PUPPET, and CHEF
| Command | Short Form | Purpose | If Omitted |
| --- | --- | --- | --- |
| ansible-playbook <file> | ansible | Runs Ansible playbook. | Automated configs not applied. |
| puppet agent -t | puppet | Executes Puppet agent run. | Desired state not enforced. |
| chef-client | chef | Applies Chef cookbook to node. | Configuration drift persists. |

