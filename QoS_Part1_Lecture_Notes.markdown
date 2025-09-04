# QoS Part 1 Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture introduces **Quality of Service (QoS)**, **Power over Ethernet (PoE)**, and **Voice VLANs** for **CCNA exam topic 4.7** ("Describe the purpose of First Hop Redundancy Protocols and QoS"). QoS prioritizes network traffic (e.g., Voice over IP) to minimize delay, jitter, and packet loss, ensuring high-quality audio for IP phones. PoE delivers power over Ethernet cables to devices like IP phones, and voice VLANs separate voice and data traffic for efficient switchport use and QoS application. The lecture sets the foundation for QoS mechanics (covered in Part 2) and ties PoE/voice VLANs to IP phone deployments.

**Lecture Objectives**:
- **Understand IP Phones and Voice VLANs**: Learn how IP phones use VoIP and share switchports with PCs via voice VLANs.
- **Explore PoE**: Understand how switches (PSE) power devices (PDs) like IP phones over Ethernet.
- **Introduce QoS**: Cover basics (bandwidth, delay, jitter, loss) and queuing (FIFO, tail drop, RED/WRED).
- **Prepare for Exam**: Master voice VLAN config, PoE concepts, and QoS fundamentals (topic 4.7).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      PH1       |-------|      SW1       |
|  VLAN 10       |       | (IP Phone,     |       | G0/0: Access   |
|                |       |  VLAN 11)      |       | VLAN 10, Voice |
+----------------+       +----------------+       | VLAN 11        |
                                         +----------------+
```
- **Devices**:
  - **PC1**: Data traffic, VLAN 10 (untagged), connected to IP phone.
  - **PH1**: Cisco IP phone, voice traffic, VLAN 11 (tagged), internal 3-port switch.
  - **SW1**: Switch with PoE, G0/0 as access port (VLAN 10) and voice VLAN 11.
- **Note**: PC1 connects to PH1’s downlink; PH1’s uplink connects to SW1’s G0/0. SW1 uses CDP to instruct PH1 to tag voice traffic in VLAN 11. PoE powers PH1 via G0/0. QoS prioritizes PH1’s voice traffic.
- **Requirements**:
  1. Configure voice VLAN on G0/0 for PC1 (VLAN 10) and PH1 (VLAN 11).
  2. Understand PoE power delivery and policing (e.g., `power inline police`).
  3. Learn QoS basics: bandwidth, delay (150ms), jitter (30ms), loss (1%), FIFO, tail drop, RED/WRED.
  4. Verify voice VLAN with `show interfaces g0/0 switchport`.

**Analogy**: QoS is like a traffic cop prioritizing ambulances (voice traffic) over cars (data traffic) on a busy road (network). Voice VLANs are like dedicated lanes for ambulances, and PoE is like fueling them through the road itself.

**Lifelong Retention Tip**: Visualize PC1 → PH1 → SW1, with PH1 tagging voice traffic (VLAN 11) and SW1 powering PH1 via PoE. In Packet Tracer, configure `switchport voice vlan 11`, verify with `show interfaces g0/0 switchport`. Create flashcards: “Voice VLAN? Tags voice traffic. PoE? Power over Ethernet. QoS? Prioritizes delay-sensitive traffic.” Practice 3 times daily for a week.

## IP Phones and Voice VLANs

**Purpose**: Introduce IP phones (VoIP) and voice VLANs to separate voice and data traffic, optimizing switchport usage and QoS.

**Key Points**:
- **IP Phones**: Use Voice over IP (VoIP) to send audio in IP packets over IP networks (e.g., Internet), unlike PSTN/POTS.
- **Internal 3-Port Switch**: Uplink to switch, downlink to PC, internal port for phone.
- **Voice VLAN**: Separates voice (tagged, e.g., VLAN 11) from data (untagged, e.g., VLAN 10) on one switchport.
- **Configuration**: Access port with `switchport voice vlan` for IP phone traffic.
- **CDP**: Switch uses Cisco Discovery Protocol to instruct phone to tag voice traffic.

**Demonstration**:
- Setup: PC1 → PH1 (downlink) → SW1 G0/0 (uplink).
- Config on SW1:
```plaintext
SW1# configure terminal
SW1(config)# interface g0/0
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# switchport voice vlan 11
SW1(config-if)# exit
```
- Traffic:
  - PC1: Untagged (VLAN 10, default access VLAN).
  - PH1: Tags voice traffic with VLAN 11 (via CDP).
- Verification:
```plaintext
SW1# show interfaces g0/0 switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Access Mode VLAN: 10
Voice VLAN: 11
```
```plaintext
SW1# show interfaces trunk
<no output, G0/0 is not a trunk>
```
```plaintext
SW1# show interfaces g0/0 trunk
Name: Gi0/0
Status: not-trunking
VLANs allowed on trunk: 10, 11
```

**Observations**:
- **Efficiency**: One switchport (G0/0) supports PC1 and PH1, halving switchport needs.
- **Access Port**: G0/0 is access (not trunk), despite carrying VLAN 10 (untagged) and VLAN 11 (tagged).
- **CDP Role**: Instructs PH1 to tag voice traffic in VLAN 11.
- **Exam Note**: Know `switchport voice vlan` and verification commands (topic 4.7).

**Comparison to Prior Content**:
- **VLAN Lecture (Day 20)**: Voice VLAN extends access ports to carry tagged voice traffic.
- **CDP Lecture (Day 25)**: CDP enables voice VLAN tagging.
- **Syslog Lab**: Voice VLAN config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).

**Comparison to Other CCNA Topics**:
- **Day 20 (VLANs)**: Voice VLAN (tagged) vs. data VLAN (untagged).
- **Day 25 (CDP/LLDP)**: CDP required for voice VLAN functionality.
- **Day 30 (Ports)**: Voice traffic (e.g., UDP 5060 for SIP) prioritized by QoS.

**Analogy**: Voice VLANs are like a highway lane reserved for ambulances (PH1’s voice traffic), while cars (PC1’s data) use the regular lane, sharing one road (switchport).

**Lifelong Retention Tip**: In Packet Tracer, configure `switchport voice vlan 11`, check `show interfaces g0/0 switchport` for VLAN 10 (access) and 11 (voice). Flashcard: “Voice VLAN? `switchport voice vlan 11`, tags voice via CDP.” Practice 3 times.

## Power over Ethernet (PoE)

**Purpose**: Deliver power to devices (e.g., IP phones) over Ethernet cables, eliminating separate power cables.

**Key Points**:
- **PoE**: Power Sourcing Equipment (PSE, e.g., switch) provides DC power to Powered Devices (PDs, e.g., IP phones, cameras, APs) via Ethernet.
- **Single Cable**: Same cable for data and power (uses unused wire pairs in FastEthernet).
- **Power Negotiation**: PSE sends low-power signals to detect PD’s power needs, avoiding damage.
- **Power Policing**: Limits PD power draw; default is `err-disable`, alternative is `log`.
- **Standards**:
  - Cisco Inline Power (ILP): 7W, 2 wire pairs (4, 5, 7, 8).
  - 802.3af (PoE, Type 1): 15.4W.
  - 802.3at (PoE+, Type 2): 30W.
  - Cisco UPoE: Non-standard, up to 60W.
  - 802.3bt (Type 3/4): 60W/100W.

**Demonstration**:
- Setup: SW1 (PSE) powers PH1 (PD) via G0/0.
- Config (Power Policing):
```plaintext
SW1# configure terminal
SW1(config)# interface g0/0
SW1(config-if)# power inline police
SW1(config-if)# exit
```
- Alternative:
```plaintext
SW1(config)# interface g0/0
SW1(config-if)# power inline police action log
SW1(config-if)# exit
```
- Verification:
```plaintext
SW1# show power inline police g0/0
Interface  Admin  Oper   Power(Watts)  Device        Class
           State  State  Available     Used
Gi0/0      police on     30.0          15.4          IP Phone      2
Action: err-disable
```
```plaintext
SW1# show power inline police g0/0
Interface  Admin  Oper   Power(Watts)  Device        Class
           State  State  Available     Used
Gi0/0      police on     30.0          15.4          IP Phone      2
Action: log
```

**Observations**:
- **Power Policing**:
  - `power inline police`: Default, err-disables port if PD draws too much power; Syslog message sent.
  - `power inline police action log`: Logs event, restarts port/PD, re-negotiates power.
  - Recovery: For err-disable, use `shutdown`, `no shutdown`.
- **Power Delivery**: SW1 converts AC (wall outlet) to DC, supplies PH1 (e.g., 15.4W for 802.3af).
- **Standards**: CCNA focuses on concepts, not memorizing wattages (flashcards optional).
- **Exam Note**: Understand PoE purpose, power policing (`err-disable` vs. `log`) (topic 4.7).

**Comparison to Prior Content**:
- **VLAN Lecture (Day 20)**: PoE powers IP phones in voice VLANs.
- **Syslog Lab**: Power policing triggers Syslog (e.g., `%PM-4-ERR_DISABLE`).
- **CDP Lecture (Day 25)**: CDP negotiates PoE power levels.

**Comparison to Other CCNA Topics**:
- **Day 10 (Cables)**: PoE uses unused FastEthernet wires (4, 5, 7, 8).
- **Day 25 (CDP/LLDP)**: CDP/LLDP aids PoE negotiation.
- **Day 35 (Port Security)**: Err-disable similar to port security violations.

**Analogy**: PoE is like a power strip embedded in an Ethernet cable, fueling PH1 while carrying data, with policing as a circuit breaker to prevent overload.

**Lifelong Retention Tip**: In Packet Tracer, configure `power inline police`, simulate high power draw, check Syslog. Flashcard: “PoE? Power over Ethernet, PSE to PD. Policing? `err-disable` or `log`.” Practice 3 times.

## Quality of Service (QoS)

**Purpose**: Prioritize delay-sensitive traffic (e.g., VoIP) to ensure low delay, jitter, and loss in converged networks.

**Key Points**:
- **Converged Networks**: Voice, video, and data share IP networks, competing for bandwidth (unlike PSTN for voice).
- **QoS Tools**: Prioritize traffic (e.g., voice over data) to manage bandwidth, delay, jitter, and loss.
- **Traffic Characteristics**:
  - **Bandwidth**: Link capacity (bps); QoS reserves portions (e.g., 20% for voice).
  - **Delay**: One-way (source to destination, e.g., PH1 to PH2) or two-way (round-trip); ≤150ms for audio.
  - **Jitter**: Variation in one-way delay; ≤30ms for audio (jitter buffer mitigates).
  - **Loss**: Dropped packets; ≤1% for audio.
- **Queuing**: FIFO (default, first-in-first-out); tail drop when queue full.
- **Tail Drop Issues**: Causes TCP global synchronization (all TCP hosts slow down/increase in waves).
- **Random Early Detection (RED)**: Drops packets randomly when queue nears threshold, avoiding global sync.
- **Weighted RED (WRED)**: Drops lower-priority traffic (e.g., HTTP) sooner than high-priority (e.g., VoIP).

**Demonstration**:
- Setup: Router R1, packets arrive at G0/0, G0/1 faster than G0/2 can forward.
- Queue Behavior:
  - FIFO: Packets queued in order received; no priority.
  - Tail Drop: Full queue drops new packets (e.g., VoIP packet lost).
  - TCP Global Sync: All TCP hosts reduce/increase window size, causing congestion waves.
  - RED: Randomly drops packets (e.g., HTTP) before queue fills.
  - WRED: Drops low-priority packets (e.g., HTTP) sooner than VoIP.
- Standards for Audio:
  - One-way delay: ≤150ms.
  - Jitter: ≤30ms.
  - Loss: ≤1%.

**Observations**:
- **Converged Networks**: VoIP (PH1) competes with PC1’s data; QoS ensures voice quality.
- **Queuing**: FIFO treats all traffic equally; tail drop harms VoIP (packet loss).
- **RED/WRED**: Mitigates tail drop; WRED prioritizes VoIP over HTTP/FTP.
- **Exam Note**: Know QoS purpose, characteristics (delay, jitter, loss), FIFO, tail drop, RED/WRED (topic 4.7).

**Comparison to Prior Content**:
- **NAT Lecture (Day 44)**: QoS may prioritize NAT’d VoIP traffic.
- **Syslog Lab**: QoS drops trigger Syslog (e.g., `%QOS-5-PACKET_DROPPED`).
- **TCP Lecture (Day 15)**: Tail drop affects TCP sliding window.

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: VoIP uses UDP (e.g., RTP); TCP global sync affects TCP traffic.
- **Day 20 (VLANs)**: Voice VLANs isolate VoIP for QoS.
- **Day 30 (Ports)**: VoIP uses UDP ports (e.g., 5060 for SIP, RTP for audio).

**Analogy**: QoS is like a chef prioritizing urgent orders (VoIP) in a busy kitchen (network), ensuring they’re served fast (low delay) and intact (low loss), while RED/WRED discards less urgent orders (HTTP) to avoid chaos.

**Lifelong Retention Tip**: In Packet Tracer, simulate VoIP traffic, observe FIFO queue. Flashcard: “QoS? Prioritizes VoIP. Standards? Delay ≤150ms, jitter ≤30ms, loss ≤1%. Tail drop? TCP global sync.” Practice 3 times.

## Common Issues and Troubleshooting

- **Voice VLAN Misconfiguration**:
  - **Issue**: PH1’s voice traffic not tagged.
  - **Cause**: Missing `switchport voice vlan 11` or CDP disabled.
  - **Fix**: Add `switchport voice vlan 11`, enable CDP (`cdp run`).
- **PoE Failure**:
  - **Issue**: PH1 not powered.
  - **Cause**: Non-PoE switch or port not enabled.
  - **Fix**: Use PoE switch, check `show power inline g0/0`.
- **Power Policing**:
  - **Issue**: G0/0 err-disabled.
  - **Cause**: PH1 draws too much power (`power inline police`).
  - **Fix**: Use `shutdown`, `no shutdown` or switch to `power inline police action log`.
- **QoS Issues**:
  - **Issue**: Poor VoIP quality (high delay/jitter).
  - **Cause**: No QoS; FIFO queue drops VoIP packets.
  - **Fix**: Apply QoS to prioritize VoIP (covered in Part 2).

**Analogy**: Troubleshooting QoS is like fixing a traffic jam by ensuring ambulances (VoIP) get priority lanes (voice VLANs) and power (PoE) to move smoothly.

**Lifelong Retention Tip**: Simulate voice VLAN failure (disable CDP), fix, verify with `show interfaces g0/0 switchport`. Checklist: “No VoIP? Check voice VLAN, CDP, PoE, QoS.” Practice 3 times.

## Practice Commands

- **Voice VLAN (SW1)**:
```plaintext
SW1# configure terminal
SW1(config)# interface g0/0
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# switchport voice vlan 11
SW1(config-if)# exit
```
- **PoE Policing (SW1)**:
```plaintext
SW1(config)# interface g0/0
SW1(config-if)# power inline police
SW1(config-if)# exit
```
- **Alternative PoE Policing**:
```plaintext
SW1(config)# interface g0/0
SW1(config-if)# power inline police action log
SW1(config-if)# exit
```
- **Verify Voice VLAN/PoE**:
```plaintext
SW1# show interfaces g0/0 switchport
SW1# show interfaces g0/0 trunk
SW1# show power inline police g0/0
```

## Quiz Answers and Explanations

1. **Examine G0/0’s config: `interface g0/0`, `switchport mode access`, `switchport voice vlan 99`. Which are true? (Select two)**  
   - **A**: Voice traffic received by G0/0 should be tagged in VLAN 99.  
   - **B**: Interface is a trunk port.  
   - **C**: Data traffic tagged in VLAN 99.  
   - **D**: Data traffic untagged.  
   - **E**: Voice traffic untagged.  
   - **Answer**: A, D (PH1 tags voice in VLAN 99 via CDP; no access VLAN set, so data is untagged in VLAN 1).  

2. **What happens if a device draws too much power with `power inline police`?**  
   - **A**: Interface restarts.  
   - **B**: Device reboots.  
   - **C**: Interface err-disabled, Syslog generated.  
   - **D**: Power reduced.  
   - **Answer**: C (`power inline police` = `action err-disable`; requires `shutdown`, `no shutdown`).  

3. **Recommended standards for interactive audio quality? (Select three)**  
   - **A**: Delay ≤300ms.  
   - **B**: Delay ≤150ms.  
   - **C**: Jitter ≤30ms.  
   - **D**: Loss ≤5%.  
   - **E**: Loss ≤1%.  
   - **Answer**: B, C, E (Delay ≤150ms, jitter ≤30ms, loss ≤1% for VoIP quality).  

4. **Negative effect of tail drop?**  
   - **A**: TCP sliding window.  
   - **B**: RED.  
   - **C**: WRED.  
   - **D**: TCP global synchronization.  
   - **Answer**: D (Tail drop causes all TCP hosts to slow/increase in waves).  

5. **Default manner of forwarding queued packets?**  
   - **A**: FIFO.  
   - **B**: CBWFQ.  
   - **C**: RED.  
   - **D**: WRED.  
   - **Answer**: A (FIFO = first-in-first-out; no priority).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: What is PoE’s purpose?
    - **A**: Prioritize VoIP traffic.  
    - **B**: Power devices over Ethernet.  
    - **C**: Tag voice traffic.  
    - **D**: Configure VLANs.  
    - **Answer**: B (PoE powers PDs like IP phones).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “QoS standards? Delay ≤150ms, jitter ≤30ms, loss ≤1%. Tail drop? TCP global sync”). Practice 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - QoS: “Bandwidth, Delay, Jitter, Loss” (BDJL).
   - PoE: “Power Sourcing Equipment to Powered Device” (PSE-PD).
2. **Visualize Topology**:
   - Draw PC1 → PH1 → SW1 G0/0 with VLAN 10 (data), VLAN 11 (voice). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “Voice VLAN? `switchport voice vlan`. PoE policing? `err-disable` or `log`. QoS? Delay ≤150ms, jitter ≤30ms, loss ≤1%.”
4. **Practice in Packet Tracer**:
   - Configure voice VLAN, PoE, simulate VoIP traffic. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Voice VLAN (tags VoIP) vs. PoE (powers phones) vs. QoS (prioritizes VoIP).
   - Quiz: “Voice VLAN? Tags via CDP. Tail drop? TCP global sync.”
6. **Troubleshooting Practice**:
   - Disable CDP, fix voice VLAN, verify. Checklist: “No VoIP? Check VLAN, CDP, PoE, QoS.”
7. **Teach Someone**:
   - Explain voice VLANs and QoS to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, QoS standards, PoE policing. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Voice VLANs**:
   - Separate voice (tagged) and data (untagged) on one access port; use `switchport voice vlan`.
2. **PoE**:
   - Powers IP phones via Ethernet; policing prevents overload (`err-disable` or `log`).
3. **QoS**:
   - Prioritizes VoIP in converged networks; manages bandwidth, delay (≤150ms), jitter (≤30ms), loss (≤1%).
4. **Queuing**:
   - FIFO (default); tail drop causes TCP global sync; RED/WRED mitigate by dropping low-priority packets.
5. **CCNA Focus**:
   - Topic 4.7: Master voice VLAN config, PoE purpose, QoS basics (delay, jitter, loss, RED/WRED).

These notes provide a complete foundation for QoS Part 1, preparing you for CCNA exam topic 4.7. Practice voice VLANs and PoE in Packet Tracer, visualize QoS queuing, and use analogies/retention tips to master these concepts for life!