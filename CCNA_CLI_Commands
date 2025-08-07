# 📘 Cisco CLI Commands Guide for CCNA 

This guide lists Cisco IOS CLI commands topic-wise for CCNA preparation for each command: purpose, short form (if applicable), command mode, what happens if skipped, when to use it, and exam tips.

---

<details>
<summary>1. Networking Devices</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show version` | Shows IOS version, memory, and uptime | — | Privileged EXEC | You miss device details like platform, version, and config register | Initial checks, licensing validation, version upgrades | Exam may show output of this command to ask memory or IOS version |
| `show running-config` | Displays current active config in RAM | `show run` | Privileged EXEC | You won't know current config or settings | Verifying IP, VLAN, ACL, routing setups | Identify real-time changes vs saved configs |
| `show startup-config` | Displays saved config in NVRAM | `show start` | Privileged EXEC | You won't know what config will be applied after reboot | After saving config with `copy run start` to validate | Know the difference between this and `show running-config` |
| `reload` | Restarts the router/switch | — | Privileged EXEC | Device will not reboot, old config may remain active | After saving config or to apply new image | Be aware: can cause downtime unless confirmed |

</details>

<details>
<summary>2. Interfaces And Cables</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ip interface brief` | Summarized interface and IP status | `sh ip int brief` | Privileged EXEC | You won’t quickly see interface up/down and IP | Interface checks, troubleshooting | Best for quick overview of all ports and their IPs |
| `show controllers` | Shows physical status of interface hardware | — | Privileged EXEC | You miss cable or interface hardware issues | Physical layer troubleshooting | Useful when link is up/down inconsistently |
| `ping <ip>` | Sends ICMP echo to test reachability | — | Any EXEC | You won’t know if device is reachable | Verifying Layer 3 connectivity | Exam trick: ping fails may be ACL or wrong subnet mask |
| `traceroute <ip>` | Tracks hop path to destination | `tracert` (Windows equivalent) | Any EXEC | You won’t see where path breaks | Network path troubleshooting | Useful in route loop or asymmetry questions |

</details>

<details>
<summary>3. OSI Model and TCP/IP Suite</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show protocols` | Shows configured protocols and addresses | — | Privileged EXEC | You won't know which layer 3 protocols are running | Protocol troubleshooting, Layer 3 validation | Helps check protocol presence like IP, IPX |
| `telnet <ip>` | Opens remote session using TCP port 23 | — | User/Priv EXEC | You can’t remotely access devices | Older method to manage remote routers/switches | Use only in labs—SSH preferred in production |
| `ssh <user>@<ip>` | Secure remote access using TCP port 22 | — | User/Priv EXEC | Remote device cannot be securely accessed | Secure CLI management of Cisco devices | Required over Telnet in secure environments |

</details>

<details>
<summary>4. Intro to the CLI</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `enable` | Enters privileged EXEC mode from user EXEC mode | `en` | User EXEC | You won’t be able to access configuration commands like `conf t` or show advanced info | Before any configuration, like assigning IPs or setting passwords | First step in most tasks; know its position in command hierarchy |
| `configure terminal` | Enters global configuration mode | `conf t` | Privileged EXEC | You can't configure interfaces, routing, etc. | After `enable`, to start editing device settings | Commonly used before setting hostname, IP addresses, VLANs |
| `exit` | Exits current mode (e.g., interface back to global) | — | Varies | You’ll stay in deeper mode, may apply changes to wrong config area | Used after finishing config in sub-modes like `interface` or `line` | Know the difference between `exit` and `end` in exam |
| `end` | Exits all the way back to privileged EXEC mode | — | Global Config/Sub-Config | You stay in the current config mode unless `exit` is used multiple times | Useful after making multiple changes deep in config | Efficient way to exit quickly, especially in labs |

</details>


<details>
<summary>5. Ethernet LAN Switching - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show mac address-table` | Shows MAC addresses learned on switch ports | `show mac` | Privileged EXEC | You can’t verify which MACs are mapped to which ports | Useful for verifying switching behavior and troubleshooting | Key for troubleshooting unknown host connections |
| `clear mac address-table dynamic` | Clears all dynamically learned MACs | — | Privileged EXEC | Old MAC entries may persist, causing stale switching info | Before testing MAC learning or clearing network changes | Often used before lab tasks or test cases |

</details>

<details>
<summary>6. Ethernet LAN Switching - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `switchport mode access` | Sets port to access mode | — | Interface Config | VLAN assignment won't work | When assigning VLANs to end hosts | Access ports support only one VLAN |
| `switchport mode trunk` | Enables trunk mode on port | — | Interface Config | VLAN traffic won't pass between switches | To allow multiple VLANs over one link | Needed for inter-switch VLAN communication |

</details>

<details>
<summary>7. IPv4 Addressing - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip address <ip> <mask>` | Assign IP to interface | — | Interface Config | Interface won't have Layer 3 address | Basic IP configuration | Exam may test valid subnet assignments |
| `no shutdown` | Enables interface | `no shut` | Interface Config | Interface stays down | After assigning IP to activate interface | Always required after IP assignment |

</details>

<details>
<summary>8. IPv4 Addressing - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip default-gateway <ip>` | Sets default gateway | — | Global Config | Switch can't send traffic outside subnet | For Layer 2 switches needing remote management | Key for remote access to switches |

</details>

<details>
<summary>9. Switch Interfaces</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `interface <type> <number>` | Accesses specific interface config | — | Global Config | You can't modify port-level settings | Before assigning IP or enabling port | Required for interface-level tasks |
| `shutdown` | Disables interface | — | Interface Config | Port remains active | Used to turn off unused ports | Helps improve security |
| `no shutdown` | Enables interface | — | Interface Config | Port remains down | Brings up port after changes | Common mistake to skip in labs |

</details>

<details>
<summary>10. The IPv4 Header</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ip route` | Displays routing table | — | Privileged EXEC | You won’t see learned or static routes | Routing verification | Helps troubleshoot unreachable subnets |
| `debug ip packet` | Real-time packet debug | — | Privileged EXEC | You can’t monitor live traffic | Deep diagnostics | Use with caution—can overload CPU |

</details>

<details>
<summary>11. Routing Fundamentals - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip routing` | Enables routing on device | — | Global Config | Device won’t route packets | On Layer 3 switches or routers | Required for static/dynamic routing |
| `show ip route` | Shows routing info | — | Privileged EXEC | No visibility into route decisions | To check learned, static, or default routes | Exam often uses this output for questions |
| `show cdp neighbors` | Shows Cisco neighbors | — | Privileged EXEC | You won’t see connected Cisco devices | Device discovery | Useful when topology is unknown |

</details>

<details>
<summary>12. Static Routing - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip route <destination> <mask> <next-hop>` | Creates static route to remote network | — | Global Config | Router won't know how to reach network | Used when there's no dynamic protocol | Very common in exam topology questions |
| `no ip route <destination> <mask> <next-hop>` | Removes a static route | — | Global Config | Route remains in table | When you need to replace/remove a static route | Practice both adding and removing routes |

</details>

<details>
<summary>13. Life of a Packet</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `debug ip packet` | Real-time analysis of traffic | — | Privileged EXEC | You won't monitor traffic patterns | Deep packet inspection | Avoid enabling on production networks |
| `show interfaces` | View interface traffic/errors | `sh int` | Privileged EXEC | Can't verify port performance | Use when debugging traffic issues | Know output values like CRCs and input errors |

</details>

<details>
<summary>14. Subnetting - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip address <ip> <subnet>` | Assign subnet to interface | — | Interface Config | Interface won't communicate with subnet | After planning subnetting | Use VLSM/subnetting skills |
| `show ip interface brief` | Displays IP and interface state | `sh ip int brief` | Privileged EXEC | Can't confirm config status | After subnet assignment | This is your verification tool |

</details>

<details>
<summary>15. Subnetting - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show running-config` | Displays current config | `sh run` | Privileged EXEC | You won't know what's applied | Confirm IP and subnet on interfaces | Always confirm before and after changes |

</details>

<details>
<summary>16. Subnetting (VLSM) - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip route <subnet> <mask> <next-hop>` | Manually route custom subnets | — | Global Config | VLSM-based networks fail to route | Required when using advanced subnetting | Know how to route overlapping/split subnets |

</details>

<details>
<summary>17. VLANs - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `vlan <id>` | Creates a VLAN | — | Global Config | VLAN cannot be assigned or used | Setup for segmentation | Default VLAN is 1, but use others |
| `name <vlan-name>` | Names VLAN for readability | — | VLAN Config | VLAN will remain unnamed | For documentation | Helps avoid confusion in large environments |

</details>


<details>
<summary>18. VLANs - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `interface <type> <num>` | Enters interface config mode | — | Global Config | Can't apply VLAN settings | Before applying VLAN to a port | Always enter interface mode first |
| `switchport access vlan <id>` | Assigns VLAN to port | — | Interface Config | Host won't be in correct VLAN | To segment traffic by department or zone | Confirm VLAN exists before applying |

</details>

<details>
<summary>19. VLANs - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `switchport mode access` | Forces port to access mode | — | Interface Config | Port may negotiate undesired trunking | For user-facing ports | Avoid DTP issues |
| `switchport trunk encapsulation dot1q` | Enables 802.1Q tagging | — | Interface Config | VLAN tagging may fail | For trunk links between switches | Cisco default is dot1q |
| `switchport mode trunk` | Sets port to trunk mode | — | Interface Config | Trunking won't occur | To carry multiple VLANs | Must match at both ends |

</details>

<details>
<summary>20. DTP / VTP</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `vtp mode transparent/server/client` | Sets VTP mode | — | Global Config | VLAN sync won't work (if client/server intended) | For central VLAN management | Transparent = manual VLAN mgmt |
| `vtp domain <name>` | Sets VTP domain name | — | Global Config | VTP sync fails between devices | Ensure all switches in same domain | Domain must match on all devices |
| `vtp password <pass>` | Secures VTP updates | — | Global Config | Unauthorized changes may occur | When security is required on VLAN updates | Password must match across switches |

</details>

<details>
<summary>21. Spanning Tree Protocol - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `spanning-tree vlan <id> priority <value>` | Sets STP priority | — | Global Config | Device might not become root | Designating root bridge | Lower priority = higher chance of being root |
| `show spanning-tree` | Displays STP info | — | Privileged EXEC | Can't validate STP roles and status | For STP troubleshooting | Learn terms like root port, blocking |

</details>

<details>
<summary>22. Spanning Tree Protocol - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `spanning-tree portfast` | Speeds up port transition | — | Interface Config | Ports take 30+ sec to forward | For access ports to PCs | Never use on trunk links |
| `spanning-tree bpduguard enable` | Blocks ports sending BPDUs | — | Interface Config | Rogue switches can affect STP | On edge ports to prevent loops | Often paired with portfast |

</details>

<details>
<summary>23. Rapid Spanning Tree Protocol</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `spanning-tree mode rapid-pvst` | Enables RSTP mode | — | Global Config | Switch runs old STP (slower) | To speed up convergence | Required for fast failover |

</details>

<details>
<summary>24. EtherChannel</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `interface range fa0/1 - 2` | Selects multiple interfaces | — | Global Config | Can't configure interfaces together | To group ports for EtherChannel | Syntax must be exact |
| `channel-group 1 mode active` | Forms EtherChannel using LACP | — | Interface Config | Channel won’t form | To bundle links for redundancy | Active uses LACP (not PAgP) |
| `show etherchannel summary` | Shows EtherChannel status | — | Privileged EXEC | Can't verify if channel is up | After configuring EtherChannel | Look for “P” (Port-channel) state |

</details>

<details>
<summary>25. Dynamic Routing</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `router rip` | Starts RIP config mode | — | Global Config | RIP routes won't be exchanged | In legacy networks | RIP is rarely used today |
| `router eigrp <asn>` | Starts EIGRP config mode | — | Global Config | EIGRP won't run | In Cisco-heavy environments | Use same ASN on all routers |
| `router ospf <pid>` | Starts OSPF process | — | Global Config | OSPF won't work | Most common IGP in exams | PID can be any number |

</details>

<details>
<summary>26. RIP / EIGRP</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `network <network>` | Advertise network in routing protocol | — | Routing Config | Network won’t be advertised | Required for RIP, EIGRP, OSPF | Must match interface IP subnet |
| `no auto-summary` | Disables automatic summarization (RIP/EIGRP) | — | Routing Config | Routes may be incorrectly summarized | On discontiguous networks | Important in multi-subnet setups |

</details>

<details>
<summary>27. OSPF - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `router ospf <id>` | Start OSPF process | — | Global Config | OSPF not enabled | Used to configure OSPF | ID is locally significant |
| `network <ip> <wildcard> area <id>` | Add interfaces to OSPF | — | OSPF Config | Interfaces won’t participate in OSPF | Needed to form neighbors | Wildcard masks often tested in exams |

</details>

<details>
<summary>28. OSPF - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ip ospf neighbor` | Displays OSPF neighbor info | — | Privileged EXEC | Can't verify adjacency | After configuring OSPF | Look for FULL/DR/BDR state |

</details>

<details>
<summary>29. OSPF - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `passive-interface default` | Suppress OSPF hello messages | — | OSPF Config | Adjacencies may form unnecessarily | Improve security & control | Default disables all interfaces |
| `no passive-interface <intf>` | Enable hello on chosen interface | — | OSPF Config | Interface won’t form neighbors | For select interfaces | Enables adjacencies on edge routers |

</details>

<details>
<summary>30. First Hop Redundancy Protocol</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `standby <group> ip <ip>` | Configure virtual IP for HSRP | — | Interface Config | Hosts won’t failover | Provide redundancy for gateway | Know HSRP vs VRRP |
| `show standby` | Check HSRP status | — | Privileged EXEC | You won’t know current role | After setting up FHRP | Look for Active/Standby roles |

</details>

<details>
<summary>31. TCP and UDP</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ip protocols` | View active protocols and ports | — | Privileged EXEC | Can't see routing protocols | To confirm RIP/EIGRP running | Includes port and timer info |
| `show tcp brief` | Displays TCP connections | — | Privileged EXEC | You miss active TCP states | When checking connections | Use to verify sessions/ports |

</details>

<details>
<summary>32. IPv6 - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ipv6 unicast-routing` | Enables IPv6 routing | — | Global Config | IPv6 won’t route | Must be enabled for IPv6 | Always required for IPv6 networks |
| `ipv6 address <addr>/<prefix>` | Assigns IPv6 address | — | Interface Config | Interface won’t have L3 address | Needed to use IPv6 | Format: 2001:db8::1/64 |

</details>

<details>
<summary>33. IPv6 - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `router ospf <id> ipv6` | Starts OSPFv3 process | — | Global Config | No OSPFv3 routing | IPv6 dynamic routing | Required for OSPF on IPv6 |

</details>

<details>
<summary>34. IPv6 - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ipv6 route` | Displays IPv6 routes | — | Privileged EXEC | Can’t confirm routing decisions | After enabling IPv6 routing | Exam may include route types |
| `ping ipv6 <addr>` | Test IPv6 connectivity | — | Any EXEC | Can’t validate reachability | IPv6 network testing | Use full IPv6 address in ping |

</details>

<details>
<summary>35. Standard Access Control List</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `access-list 10 permit 192.168.1.0 0.0.0.255` | Create ACL to allow subnet | — | Global Config | Traffic may be blocked | Simple filtering | Standard ACLs filter by source only |
| `interface <intf>` | Access interface config mode | — | Global Config | ACL can’t be applied | Before applying ACL | Always apply on interface |
| `ip access-group 10 in` | Applies ACL inbound | — | Interface Config | ACL won't filter traffic | To enforce security policy | Know direction: in vs out |

</details>


<details>
<summary>36. CDP and LLDP</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show cdp neighbors` | Displays Cisco neighbor devices | — | Privileged EXEC | You won’t discover connected Cisco devices | To verify device connectivity | Works only between Cisco devices |
| `show lldp neighbors` | Displays neighbors via LLDP | — | Privileged EXEC | Non-Cisco devices may not show | For multi-vendor environments | Use when CDP is unavailable |

</details>

<details>
<summary>37. NTP - Network Time Protocol</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ntp server <ip>` | Configures external NTP server | — | Global Config | Device clock won't sync | To maintain time accuracy | Required for logs and certificates |
| `show ntp status` | Displays NTP sync status | — | Privileged EXEC | You won’t know if synced | After setting NTP config | Look for “synchronized” output |

</details>

<details>
<summary>38. DNS - Domain Name System</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip name-server <ip>` | Sets DNS server IP | — | Global Config | Hostnames can’t be resolved | Required for name resolution | Needed for `ping hostname` to work |
| `ping www.cisco.com` | Test DNS and connectivity | — | Any EXEC | You won’t verify DNS + reachability | After DNS setup | Use if `ping IP` works but `ping domain` fails |

</details>

<details>
<summary>39. DHCP - Dynamic Host Configuration Protocol</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip dhcp pool <name>` | Creates DHCP pool | — | Global Config | DHCP won’t assign IPs | For auto IP assignment to clients | Name helps identify purpose |
| `network <ip> <mask>` | Sets IP range in pool | — | DHCP Pool Config | Pool won’t allocate IPs | Must match interface subnet | Often forgotten step |
| `default-router <ip>` | Sets default gateway for clients | — | DHCP Pool Config | Clients may have no gateway | Required for internet access | Matches router interface IP |

</details>

<details>
<summary>40. SNMP - Simple Network Management Protocol</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `snmp-server community public RO` | Creates read-only SNMP string | — | Global Config | SNMP tools can't query device | For monitoring interfaces, status | Default string is “public” but insecure |
| `show snmp` | Verifies SNMP status | — | Privileged EXEC | You won’t know if SNMP is active | After SNMP setup | Confirm community string and status |

</details>

<details>
<summary>41. SYSLOG</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `logging <server-ip>` | Sends logs to external syslog | — | Global Config | Logs won't be collected centrally | For monitoring and alerts | Required for centralized log review |
| `logging trap warnings` | Sets logging severity level | — | Global Config | Device may send too many/few logs | Controls log verbosity | Levels 0–7 (0 = emergency) |

</details>

<details>
<summary>42. SSH - Secure Shell</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `hostname <name>` | Sets device hostname | — | Global Config | SSH config won't proceed | First step in SSH setup | Hostname is tied to RSA key generation |
| `ip domain-name <domain>` | Sets domain for SSH key | — | Global Config | RSA key can’t be generated | Required before RSA command | Can be anything (e.g., localdomain) |
| `crypto key generate rsa` | Generates SSH keys | — | Global Config | SSH will not work | After hostname and domain are set | Use key size 1024+ |
| `username <user> password <pass>` | Creates login credentials | — | Global Config | SSH logins will fail | Needed for local authentication | Use strong password |
| `line vty 0 4` | Enters virtual line config | — | Global Config | Can’t apply login/auth methods | Required to enable SSH access | Must configure login, transport, local |

</details>

<details>
<summary>43. FTP and TFTP</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `copy running-config tftp` | Backs up config to TFTP server | — | Privileged EXEC | No backup of current config | Before major changes | Useful in lab or recovery |
| `copy tftp running-config` | Restores config from TFTP | — | Privileged EXEC | Can’t recover previous config | After config loss or reset | Watch for IP reachability to TFTP |

</details>

<details>
<summary>44. NAT (Static) - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip nat inside source static <priv-ip> <pub-ip>` | Creates static NAT entry | — | Global Config | Private host won’t be reachable | For static public mappings | Good for servers (e.g., web) |

</details>

<details>
<summary>45. NAT (Dynamic) - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip nat pool <name> <start> <end> netmask <mask>` | Defines NAT address pool | — | Global Config | NAT won’t have public IPs to assign | For dynamic public IP translation | Pool must not overlap existing IPs |
| `access-list <num> permit <subnet>` | Defines who can use NAT | — | Global Config | No traffic will match NAT | Needed to link users to pool | Usually ACL 1 for simplicity |
| `ip nat inside source list <num> pool <name>` | Applies NAT config | — | Global Config | NAT won’t function | Final step in NAT setup | Ties pool to ACL-matching users |

</details>

<details>
<summary>46. QoS (Voice VLAN) - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `mls qos` | Enables QoS globally | — | Global Config | QoS settings won't apply | Required before any QoS settings | Must be turned on before port-level QoS |
| `switchport voice vlan <id>` | Assigns voice VLAN to port | — | Interface Config | IP phones won't get separate VLAN | For separating voice/data traffic | Always pair with access VLAN |

</details>


<details>
<summary>47. QoS (Quality of Service) - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `priority-queue out` | Enables egress priority queue | — | Interface Config | Voice traffic may face delay | Used for voice/video prioritization | Use with trust settings |
| `mls qos trust dscp` | Trusts DSCP from endpoints | — | Interface Config | Switch may reclassify traffic | When IP phone or trusted device sets DSCP | Trust only on edge or trunk ports |

</details>

<details>
<summary>48. Security Fundamentals</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `service password-encryption` | Encrypts plaintext passwords | — | Global Config | Passwords visible in plain text | Basic security best practice | Applies only to existing passwords |
| `enable secret <pwd>` | Sets encrypted enable password | — | Global Config | No secure access to privileged mode | To protect device from unauthorized access | Overrides `enable password` |

</details>

<details>
<summary>49. Port Security</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `switchport port-security` | Enables port security | — | Interface Config | No MAC filtering will apply | To restrict access to specific MACs | Must be on access port |
| `switchport port-security maximum <n>` | Limits MAC addresses | — | Interface Config | Unlimited devices allowed | To stop port sharing | Often set to 1 |
| `switchport port-security violation shutdown` | Shuts port on violation | — | Interface Config | Port stays active if MAC violated | To disable port after attack | Other options: restrict, protect |

</details>

<details>
<summary>50. DHCP Snooping</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip dhcp snooping` | Enables DHCP snooping globally | — | Global Config | Rogue DHCP servers may work | Prevent DHCP attacks | Needed to activate all DHCP snooping |
| `ip dhcp snooping vlan <id>` | Enables snooping on VLAN | — | Global Config | VLAN will not be protected | Must match user VLAN | Common to use on VLAN 1 or user VLANs |
| `ip dhcp snooping trust` | Trusts a port (uplink) | — | Interface Config | DHCP offers may be dropped | On uplinks to legit DHCP servers | Never trust user-facing ports |

</details>

<details>
<summary>51. Dynamic ARP Inspection</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip arp inspection vlan <id>` | Enables ARP inspection on VLAN | — | Global Config | ARP spoofing not prevented | Defends against ARP poisoning | Requires DHCP snooping |
| `ip arp inspection trust` | Trusts the port | — | Interface Config | ARP responses dropped | On trusted uplinks | Combine with DHCP snooping trust |

</details>

<details>
<summary>52. LAN Architectures</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show vlan brief` | Shows VLAN IDs and status | — | Privileged EXEC | You can’t verify VLANs | To validate VLAN setup | Know VLAN 1 is default |
| `show interfaces status` | Displays port status and speed | — | Privileged EXEC | Can't confirm port speed/Duplex | Port-level troubleshooting | Look for connected vs notconnect |

</details>

<details>
<summary>53. WAN Architectures</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show ip interface` | Shows interface protocol/IP info | — | Privileged EXEC | Can't check encapsulation/IP status | Useful in WAN and serial troubleshooting | Look for up/up or down/down |
| `encapsulation ppp` | Sets serial link protocol | — | Interface Config | Interface uses default (HDLC) | Needed for PPP features | Often used in labs for WAN practice |

</details>

<details>
<summary>54. Virtualization and Cloud - Part 1</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show version` | Shows virtualization platform | — | Privileged EXEC | Can't confirm virtual support | To check Cisco platform type | Check for IOx or virtual capabilities |
| `show platform` | Displays hardware/software info | — | Privileged EXEC | Can't validate supported features | For compatibility checks | Used before upgrades or deployment |

</details>

<details>
<summary>55. Virtualization (Containers) - Part 2</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `docker ps` | Lists running containers | — | Exec shell | Can't see container status | Used in IOx-enabled Cisco devices | Similar to Linux Docker |

</details>


<details>
<summary>56. Virtualization (VRF) - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip vrf <name>` | Creates a new VRF instance | — | Global Config | Routing domains will not be separated | For creating multiple logical routers | Useful in ISPs and multi-tenant setups |
| `ip route vrf <name> <dest> <mask> <next-hop>` | Adds route within specific VRF | — | Global Config | No route added in that VRF | For separate routing tables | Remember to assign interfaces to VRF |

</details>

<details>
<summary>57. Wireless Fundamentals</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show controllers dot11Radio 0` | Displays wireless radio info | — | Privileged EXEC | Can’t troubleshoot wireless issues | To verify radio settings or errors | Useful in wireless tuning |
| `show wlan summary` | Displays all WLANs | — | Privileged EXEC | Can’t verify WLAN configs | For visibility into all SSIDs | Look for Admin/Operational status |

</details>

<details>
<summary>58. Wireless Architectures</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show wireless stats` | Shows AP/client stats | — | Privileged EXEC | Can't monitor usage or health | For analyzing wireless performance | Useful in large WLAN deployments |

</details>

<details>
<summary>59. Wireless Security</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `security wpa psk set-key ascii 0 <key>` | Configures WPA PSK | — | WLAN Config | Clients can’t connect securely | When configuring secure WLAN | WPA2 is most used in exams |

</details>

<details>
<summary>60. Wireless Configuration</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `interface dot11Radio 0` | Accesses wireless radio | — | Global Config | Wireless settings not applied | To create SSIDs or apply settings | Always start from radio interface |
| `ssid <name>` | Creates SSID | — | Dot11Radio Config | WLAN won’t broadcast | For defining network name | SSID is broadcast to clients |
| `authentication open` | Sets open authentication | — | SSID Config | Clients can’t connect | When configuring open networks | Use with caution; not secure |

</details>

<details>
<summary>61. Introduction to Network Automation</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show version | json` | Displays version in JSON | — | Privileged EXEC | JSON parsing won't be possible | For API/automation tools | Supported in newer IOS XE |
| `restconf` | Enables RESTCONF feature | — | Global Config | RESTCONF APIs won’t work | Required for modern APIs | Pair with HTTPS server config |

</details>

<details>
<summary>62. JSON, XML, and YAML</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show interfaces | include json` | Show interface data in JSON | — | Privileged EXEC | Data won’t be structured | When integrating with scripts | Not supported in all IOS versions |

</details>

<details>
<summary>63. REST APIs</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip http server` | Enables HTTP server | — | Global Config | REST and web UI won’t work | Required for basic web access | Combine with secure-server |
| `ip http secure-server` | Enables HTTPS server | — | Global Config | No secure REST access | Required for RESTCONF or secure GUI | HTTPS is best practice |

</details>

<details>
<summary>64. Software Defined Networking (SDN)</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `sdm prefer lanbase-routing` | Enables routing features in SDN | — | Global Config | Routing features limited | On LAN-based SDN switches | Requires reload |
| `show license` | Displays license details | — | Privileged EXEC | Can’t confirm SDN or DNA support | Before activating SDN | Check for required feature licenses |

</details>

<details>
<summary>65. Ansible, Puppet, and Chef</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `netconf-yang` | Enables NETCONF/YANG | — | Global Config | Device can’t use NETCONF | Used with Ansible/Puppet/Chef | Required for data modeling |
| `show management api` | Verifies API support | — | Privileged EXEC | API tools may fail | After enabling automation features | Look for enabled: yes |

</details>


<details>
<summary>56. Virtualization (VRF) - Part 3</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip vrf <name>` | Creates a new VRF instance | — | Global Config | Routing domains will not be separated | For creating multiple logical routers | Useful in ISPs and multi-tenant setups |
| `ip route vrf <name> <dest> <mask> <next-hop>` | Adds route within specific VRF | — | Global Config | No route added in that VRF | For separate routing tables | Remember to assign interfaces to VRF |

</details>

<details>
<summary>57. Wireless Fundamentals</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show controllers dot11Radio 0` | Displays wireless radio info | — | Privileged EXEC | Can’t troubleshoot wireless issues | To verify radio settings or errors | Useful in wireless tuning |
| `show wlan summary` | Displays all WLANs | — | Privileged EXEC | Can’t verify WLAN configs | For visibility into all SSIDs | Look for Admin/Operational status |

</details>

<details>
<summary>58. Wireless Architectures</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show wireless stats` | Shows AP/client stats | — | Privileged EXEC | Can't monitor usage or health | For analyzing wireless performance | Useful in large WLAN deployments |

</details>

<details>
<summary>59. Wireless Security</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `security wpa psk set-key ascii 0 <key>` | Configures WPA PSK | — | WLAN Config | Clients can’t connect securely | When configuring secure WLAN | WPA2 is most used in exams |

</details>

<details>
<summary>60. Wireless Configuration</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `interface dot11Radio 0` | Accesses wireless radio | — | Global Config | Wireless settings not applied | To create SSIDs or apply settings | Always start from radio interface |
| `ssid <name>` | Creates SSID | — | Dot11Radio Config | WLAN won’t broadcast | For defining network name | SSID is broadcast to clients |
| `authentication open` | Sets open authentication | — | SSID Config | Clients can’t connect | When configuring open networks | Use with caution; not secure |

</details>

<details>
<summary>61. Introduction to Network Automation</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show version | json` | Displays version in JSON | — | Privileged EXEC | JSON parsing won't be possible | For API/automation tools | Supported in newer IOS XE |
| `restconf` | Enables RESTCONF feature | — | Global Config | RESTCONF APIs won’t work | Required for modern APIs | Pair with HTTPS server config |

</details>

<details>
<summary>62. JSON, XML, and YAML</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `show interfaces | include json` | Show interface data in JSON | — | Privileged EXEC | Data won’t be structured | When integrating with scripts | Not supported in all IOS versions |

</details>

<details>
<summary>63. REST APIs</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `ip http server` | Enables HTTP server | — | Global Config | REST and web UI won’t work | Required for basic web access | Combine with secure-server |
| `ip http secure-server` | Enables HTTPS server | — | Global Config | No secure REST access | Required for RESTCONF or secure GUI | HTTPS is best practice |

</details>

<details>
<summary>64. Software Defined Networking (SDN)</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `sdm prefer lanbase-routing` | Enables routing features in SDN | — | Global Config | Routing features limited | On LAN-based SDN switches | Requires reload |
| `show license` | Displays license details | — | Privileged EXEC | Can’t confirm SDN or DNA support | Before activating SDN | Check for required feature licenses |

</details>

<details>
<summary>65. Ansible, Puppet, and Chef</summary>

| Command | Purpose | Short Form | Mode | What Happens If Skipped | When to Use | Exam Tip |
|---|---|---|---|---|---|---|
| `netconf-yang` | Enables NETCONF/YANG | — | Global Config | Device can’t use NETCONF | Used with Ansible/Puppet/Chef | Required for data modeling |
| `show management api` | Verifies API support | — | Privileged EXEC | API tools may fail | After enabling automation features | Look for enabled: yes |

</details>
