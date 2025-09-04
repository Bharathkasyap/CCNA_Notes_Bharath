# QoS Part 1 Lab Notes (Voice VLANs) for CCNA Beginners

## Lab Overview

This lab focuses on configuring **Voice VLANs** on a Cisco multilayer switch (SW1) to support IP phones and PCs, aligning with **CCNA exam topic 4.7** ("Describe the purpose of First Hop Redundancy Protocols and QoS"). Using Packet Tracer, it sets up voice VLANs (VLAN 20) and data VLANs (VLAN 10) on SW1’s access ports (G1/0/2–3) for two IP phones (PH1, PH2) and PCs (PC1, PC2). It also configures a router-on-a-stick on R1 (F0/0.10, F0/0.20) to enable communication, leveraging **Power over Ethernet (PoE)** to power the phones. The lab verifies configurations by pinging between PCs (untagged, VLAN 10) and initiating a call between phones (tagged, VLAN 20) in simulation mode, observing 802.1Q tags. This reinforces concepts from the QoS Part 1 lecture.

**Lab Objectives**:
- **Configure Voice VLANs**: Set up SW1’s access ports for data (VLAN 10) and voice (VLAN 20).
- **Configure Router-on-a-Stick**: Enable R1 to handle VLAN 10 and VLAN 20 traffic.
- **Leverage PoE**: Use SW1’s PoE to power IP phones without separate power cables.
- **Verify Traffic**: Confirm PC traffic is untagged (VLAN 10) and phone traffic is tagged (VLAN 20) using simulation mode.
- **Prepare for Exam**: Master voice VLAN configuration and verification (topic 4.7).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      PH1       |-------|                |-------|      R1        |
| 192.168.10.11  |       | (IP Phone,     |       |      SW1       |       | F0/0: Trunk    |
| VLAN 10        |       | VLAN 20, 2010) |       | G1/0/2: Access |       | F0/0.10: VLAN 10|
+----------------+       +----------------+       | G1/0/3: Access |       | F0/0.20: VLAN 20|
                                         | G1/0/1: Trunk  |       +----------------+
                                         |                |
                                         +----------------+
                                         |                |
                                         +----------------+       +----------------+
                                         |      PH2       |-------|      PC2       |
                                         | (IP Phone,     |       | 192.168.10.12  |
                                         | VLAN 20, 2010) |       | VLAN 10        |
                                         +----------------+       +----------------+
```
- **Devices**:
  - **PC1**: 192.168.10.11, VLAN 10 (untagged), connected to PH1’s downlink.
  - **PC2**: 192.168.10.12, VLAN 10 (untagged), connected to PH2’s downlink.
  - **PH1**: IP phone, VLAN 20 (tagged), phone number 2010, powered by PoE, connected to SW1 G1/0/2.
  - **PH2**: IP phone, VLAN 20 (tagged), phone number 2010, powered by PoE, connected to SW1 G1/0/3.
  - **SW1**: Multilayer switch (PoE-enabled), G1/0/2–3 (access, VLAN 10, voice VLAN 20), G1/0/1 (trunk to R1).
  - **R1**: Router-on-a-stick, F0/0.10 (192.168.10.1, VLAN 10), F0/0.20 (192.168.20.1, VLAN 20).
- **Note**: SW1 uses **CDP** to instruct PH1/PH2 to tag voice traffic in VLAN 20. R1 pre-configured with telephony settings (e.g., DHCP, TFTP for phone numbers/IPs, beyond CCNA scope). PoE powers PH1/PH2 via G1/0/2–3. No inter-subnet traffic in this lab; focus is on VLAN tagging.
- **Requirements**:
  1. Configure SW1’s G1/0/2–3 for VLAN 10 (data) and VLAN 20 (voice).
  2. Configure SW1’s G1/0/1 as a trunk (VLANs 10, 20) to R1.
  3. Configure R1’s F0/0.10 (VLAN 10) and F0/0.20 (VLAN 20) for router-on-a-stick.
  4. Verify PC1 pings PC2 (untagged, VLAN 10) and PH1 calls PH2 (tagged, VLAN 20) in simulation mode.
  5. Observe 802.1Q tags in Packet Tracer’s simulation mode.

**Analogy**: Voice VLANs are like a bus lane (VLAN 20) for ambulances (IP phones) and a regular lane (VLAN 10) for cars (PCs), sharing one road (switchport). PoE is like powering the ambulances through the road itself, and the router-on-a-stick is like a traffic hub directing both lanes.

**Lifelong Retention Tip**: Visualize PC1 → PH1 → SW1 → R1, with PH1 tagging voice traffic (VLAN 20). In Packet Tracer, configure `switchport voice vlan 20`, ping PC2 from PC1, call PH2 from PH1, and check tags in simulation mode. Create flashcards: “Voice VLAN? Tags voice traffic via CDP. Command? `switchport voice vlan 20`.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Configure Voice VLANs on SW1
**Objective**: Set up SW1’s G1/0/2–3 for data (VLAN 10, untagged) and voice (VLAN 20, tagged), and G1/0/1 as a trunk to R1.

**Steps (on SW1)**:
1. Configure access ports for IP phones:
```plaintext
SW1# configure terminal
SW1(config)# interface range g1/0/2 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# switchport voice vlan 20
SW1(config-if-range)# exit
```
2. Configure trunk to R1:
```plaintext
SW1(config)# interface g1/0/1
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
SW1(config-if)# exit
```

**Observations**:
- **Access Ports (G1/0/2–3)**:
  - `switchport mode access`: Sets ports as access (not trunk).
  - `switchport access vlan 10`: PC1/PC2 traffic untagged, placed in VLAN 10.
  - `switchport voice vlan 20`: PH1/PH2 tag voice traffic in VLAN 20 via CDP.
- **Trunk Port (G1/0/1)**:
  - `switchport trunk encapsulation dot1q`: Uses 802.1Q for VLAN tagging.
  - `switchport mode trunk`: Carries VLAN 10 and 20 traffic to R1.
  - `switchport trunk allowed vlan 10,20`: Restricts to VLANs 10 and 20 for security.
- **PoE**: SW1 (multilayer, PoE-enabled) powers PH1/PH2 via G1/0/2–3; no separate power cables.
- **CDP**: Instructs PH1/PH2 to tag voice traffic in VLAN 20.
- **Exam Note**: Know `switchport voice vlan` and trunk configuration for voice VLANs (topic 4.7).

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Lab applies voice VLANs to separate voice (VLAN 20) and data (VLAN 10) for QoS.
- **VLAN Lecture (Day 20)**: Voice VLAN extends access ports to carry tagged voice traffic.
- **CDP Lecture (Day 25)**: CDP enables voice VLAN tagging.
- **Syslog Lab**: Voice VLAN config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).

**Comparison to Other CCNA Topics**:
- **Day 20 (VLANs)**: Voice VLAN (tagged) vs. data VLAN (untagged).
- **Day 25 (CDP/LLDP)**: CDP required for voice VLAN functionality.
- **Day 30 (Ports)**: Voice traffic (e.g., SCCP, UDP 5060 for SIP) tagged in VLAN 20.

**Analogy**: Configuring voice VLANs is like setting up a bus lane (VLAN 20) for ambulances (PH1/PH2) and a car lane (VLAN 10) for PCs, with SW1 directing traffic and powering ambulances via PoE.

**Lifelong Retention Tip**: In Packet Tracer, configure `switchport voice vlan 20` on G1/0/2–3, verify with `show interfaces g1/0/2 switchport`. Flashcard: “Voice VLAN config? `switchport mode access`, `switchport access vlan 10`, `switchport voice vlan 20`.” Practice 3 times.

### Step 2: Configure Router-on-a-Stick on R1
**Objective**: Set up R1’s F0/0 subinterfaces for VLAN 10 (data) and VLAN 20 (voice) to support phone configuration.

**Steps (on R1)**:
```plaintext
R1# configure terminal
R1(config)# interface f0/0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface f0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface f0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config)# exit
```

**Observations**:
- **Subinterfaces**:
  - `f0/0.10`: Handles VLAN 10 (data, 192.168.10.0/24), IP 192.168.10.1.
  - `f0/0.20`: Handles VLAN 20 (voice, 192.168.20.0/24), IP 192.168.20.1.
- **Telephony Settings**: R1 pre-configured (beyond CCNA) with DHCP/TFTP to assign PH1/PH2 IPs and phone numbers (e.g., 2010 for PH2).
- **No Inter-Subnet Traffic**: Lab focuses on VLAN tagging, not routing between VLANs.
- **Exam Note**: Understand router-on-a-stick for VLAN support (topic 4.7).

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Router-on-a-stick supports voice VLANs for QoS.
- **VLAN Lecture (Day 20)**: Subinterfaces handle tagged VLAN traffic.
- **Syslog Lab**: Subinterface config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).
- **Inter-VLAN Routing (Day 22)**: Router-on-a-stick matches lab setup.

**Comparison to Other CCNA Topics**:
- **Day 20 (VLANs)**: Subinterfaces process VLAN 10/20 traffic.
- **Day 25 (CDP/LLDP)**: CDP on R1 aids phone configuration.
- **Day 10 (IP Addressing)**: Subinterfaces use 192.168.10.0/24, 192.168.20.0/24.

**Analogy**: Router-on-a-stick is like a traffic hub (R1) directing cars (VLAN 10) and ambulances (VLAN 20) to their lanes, with pre-set signs (telephony settings) guiding ambulances.

**Lifelong Retention Tip**: In Packet Tracer, configure `encapsulation dot1q 10` on F0/0.10, verify with `show running-config`. Flashcard: “Router-on-a-stick? `encapsulation dot1q`, subinterface IPs.” Practice 3 times.

### Step 3: Test and Verify Traffic
**Objective**: Verify PC traffic (untagged, VLAN 10) and phone traffic (tagged, VLAN 20) using pings and a call in simulation mode.

**Steps**:
1. Ping PC2 from PC1 (realtime mode):
```plaintext
PC1> ping 192.168.10.12
Pinging 192.168.10.12 with 32 bytes of data:
Reply from 192.168.10.12: bytes=32 time=10ms TTL=255
Reply from 192.168.10.12: bytes=32 time=8ms TTL=255
Reply from 192.168.10.12: bytes=32 time=9ms TTL=255
Reply from 192.168.10.12: bytes=32 time=11ms TTL=255
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
2. Ping R1 from PC1 (realtime mode):
```plaintext
PC1> ping 192.168.10.1
Pinging 192.168.10.1 with 32 bytes of data:
Reply from 192.168.10.1: bytes=32 time=12ms TTL=255
Reply from 192.168.10.1: bytes=32 time=10ms TTL=255
Reply from 192.168.10.1: bytes=32 time=11ms TTL=255
Reply from 192.168.10.1: bytes=32 time=9ms TTL=255
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
3. Ping PC2 from PC1 (simulation mode):
```plaintext
PC1> ping 192.168.10.12
```
- Check frame in simulation mode (outbound PDU details):
  - No 802.1Q tag (untagged, VLAN 10).
4. Call PH2 from PH1 (simulation mode):
- On PH2, check GUI: Phone number 2010 (assigned by R1).
- On PH1:
  - Dial 2010, pick up phone.
  - Observe SCCP message (Skinny Client Control Protocol, beyond CCNA).
- Check frame in simulation mode (outbound PDU details):
  - Source IP: PH1’s IP (e.g., 192.168.20.x, assigned by R1).
  - 802.1Q header: TPID 0x8100 (dot1q), VLAN ID 0x0014 (decimal 20).

**Observations**:
- **PC Traffic**: Untagged (VLAN 10), confirmed by no 802.1Q tag in ping to 192.168.10.12.
- **Phone Traffic**: Tagged (VLAN 20), confirmed by 802.1Q tag (VLAN ID 20) in SCCP message.
- **PoE**: SW1 powers PH1/PH2 via G1/0/2–3; no power cables needed.
- **R1 Role**: Assigns phone IPs/numbers (e.g., 2010 for PH2) via pre-configured telephony settings.
- **SW1 Behavior**: Sends untagged traffic to PCs (VLAN 10), tagged traffic to phones (VLAN 20).
- **Exam Note**: Verify voice VLAN tagging with simulation mode; understand untagged vs. tagged traffic (topic 4.7).

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Lab confirms voice VLANs separate voice (tagged, VLAN 20) for QoS.
- **VLAN Lecture (Day 20)**: Voice VLAN tags traffic; data VLAN untagged.
- **CDP Lecture (Day 25)**: CDP enables VLAN 20 tagging for phones.
- **Syslog Lab**: Pings, voice VLAN configs trigger Syslog (e.g., `%SYS-5-CONFIG_I`).

**Comparison to Other CCNA Topics**:
- **Day 20 (VLANs)**: Voice VLAN (tagged) vs. data VLAN (untagged).
- **Day 25 (CDP/LLDP)**: CDP instructs phones to tag VLAN 20.
- **Day 30 (Ports)**: Voice traffic (e.g., SCCP, UDP 5060) tagged in VLAN 20.

**Analogy**: Testing voice VLANs is like checking if ambulances (PH1/PH2) use the bus lane (VLAN 20, tagged) while cars (PC1/PC2) use the regular lane (VLAN 10, untagged), with SW1 directing traffic.

**Lifelong Retention Tip**: In Packet Tracer, ping PC2 from PC1 (untagged), call PH2 from PH1 (tagged), check tags in simulation mode. Flashcard: “Voice traffic? Tagged VLAN 20. PC traffic? Untagged VLAN 10.” Practice 3 times.

## Common Issues and Troubleshooting

- **Voice VLAN Tagging Failure**:
  - **Issue**: PH1/PH2 traffic not tagged in VLAN 20.
  - **Cause**: Missing `switchport voice vlan 20` or CDP disabled.
  - **Fix**: Add `switchport voice vlan 20`, enable CDP (`cdp run`), verify with `show interfaces g1/0/2 switchport`.
- **PoE Failure**:
  - **Issue**: PH1/PH2 not powered.
  - **Cause**: Non-PoE switch or PoE disabled on G1/0/2–3.
  - **Fix**: Use PoE switch, check `show power inline g1/0/2`.
- **Ping Failure**:
  - **Issue**: PC1 can’t ping PC2 (192.168.10.12).
  - **Cause**: Incorrect VLAN (e.g., missing `switchport access vlan 10`) or ARP delay.
  - **Fix**: Verify `switchport access vlan 10`, wait for ARP, check `show arp`.
- **Trunk Misconfiguration**:
  - **Issue**: Phones can’t reach R1.
  - **Cause**: Missing VLAN 20 on trunk (`switchport trunk allowed vlan`).
  - **Fix**: Add `switchport trunk allowed vlan 10,20`, verify with `show interfaces trunk`.
- **Phone Configuration Failure**:
  - **Issue**: PH2 has no phone number (2010).
  - **Cause**: R1 telephony settings incorrect or F0/0.20 down.
  - **Fix**: Check R1’s subinterface (`show ip interface brief`), verify telephony settings.

**Analogy**: Troubleshooting voice VLANs is like ensuring ambulances (phones) use the correct lane (VLAN 20) with proper signs (CDP) and power (PoE), while cars (PCs) stay in their lane (VLAN 10).

**Lifelong Retention Tip**: Simulate voice VLAN failure (disable CDP), fix with `cdp run`, verify tags. Checklist: “No VoIP? Check voice VLAN, CDP, PoE, trunk, R1 subinterfaces.” Practice 3 times.

## Practice Commands

- **Voice VLANs (SW1)**:
```plaintext
SW1# configure terminal
SW1(config)# interface range g1/0/2 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# switchport voice vlan 20
SW1(config-if-range)# exit
SW1(config)# interface g1/0/1
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
SW1(config-if)# exit
```
- **Router-on-a-Stick (R1)**:
```plaintext
R1# configure terminal
R1(config)# interface f0/0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface f0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface f0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config)# exit
```
- **Verify Configurations**:
```plaintext
SW1# show interfaces g1/0/2 switchport
SW1# show interfaces trunk
SW1# show power inline g1/0/2
R1# show ip interface brief
```
- **Test Traffic**:
```plaintext
PC1> ping 192.168.10.12
PC1> ping 192.168.10.1
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Voice VLAN: “Data Untagged, Voice Tagged” (DUVT).
   - Router-on-a-Stick: “Subinterface, Dot1Q, IP” (SDI).
2. **Visualize Topology**:
   - Draw PC1 → PH1 → SW1 → R1, with VLAN 10 (untagged) and VLAN 20 (tagged). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Voice VLAN? `switchport voice vlan 20`, tags via CDP. PC traffic? Untagged VLAN 10. Trunk? `switchport trunk allowed vlan 10,20`.”
4. **Practice in Packet Tracer**:
   - Configure voice VLANs, ping PC2, call PH2, check tags in simulation mode. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Voice VLAN (tagged, VLAN 20) vs. Data VLAN (untagged, VLAN 10).
   - Quiz: “Voice traffic? Tagged VLAN 20. PC traffic? Untagged VLAN 10.”
6. **Troubleshooting Practice**:
   - Disable CDP, fix with `cdp run`, verify tags. Checklist: “No VoIP? Check VLAN, CDP, PoE, trunk.”
7. **Teach Someone**:
   - Explain voice VLAN tagging to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, VLAN tagging, PoE. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Voice VLANs**:
   - Separate voice (tagged, VLAN 20) and data (untagged, VLAN 10) on access ports; use `switchport voice vlan 20`.
2. **PoE**:
   - SW1 powers PH1/PH2 via G1/0/2–3; no separate power cables.
3. **Router-on-a-Stick**:
   - R1’s F0/0.10 (VLAN 10), F0/0.20 (VLAN 20) support phone configuration.
4. **Traffic Verification**:
   - PC traffic untagged (VLAN 10), phone traffic tagged (VLAN 20, 802.1Q).
5. **CDP Role**:
   - Instructs phones to tag voice traffic in VLAN 20.
6. **CCNA Focus**:
   - Topic 4.7: Master voice VLAN config, verification, and tagging behavior.

These notes provide a complete foundation for the QoS Part 1 lab, reinforcing CCNA exam topic 4.7. Practice voice VLANs in Packet Tracer, verify tags in simulation mode, and use analogies/retention tips to master these concepts for life!