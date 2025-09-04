# QoS Part 2 Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture dives deeper into **Quality of Service (QoS)**, building on QoS Part 1, and focuses on **CCNA exam topic 4.7** ("Describe the purpose of First Hop Redundancy Protocols and QoS"). It covers **classification**, **marking**, **queuing**, **congestion management**, **policing**, and **shaping**, emphasizing their roles in prioritizing traffic (e.g., voice/video) to minimize delay, jitter, and loss. The lecture explains how devices identify and prioritize traffic using fields like **PCP (CoS)** and **DSCP**, manage multiple queues with **CBWFQ** and **LLQ**, and control traffic rates with shaping and policing. No configuration is required for the CCNA, but understanding concepts is key.

**Lecture Objectives**:
- **Understand Classification and Marking**: Learn how devices identify (classify) and mark traffic (PCP, DSCP) for prioritization.
- **Explore Queuing and Congestion**: Understand multiple queues, scheduling (weighted round-robin, CBWFQ, LLQ), and congestion tools (RED/WRED).
- **Learn Policing and Shaping**: Grasp how these control traffic rates (shaping buffers, policing drops).
- **Prepare for Exam**: Master QoS concepts (topic 4.7) without needing configuration details.
- **Review Part 1**: Reinforce voice VLANs, PoE, and basic QoS (delay ≤150ms, jitter ≤30ms, loss ≤1%).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      PH1       |-------|      SW1       |-------|      R1        |
|  VLAN 10       |       | (IP Phone,     |       | G0/0: Access   |       | G0/0           |
|                |       |  VLAN 20)      |       | VLAN 10, Voice |       |                |
+----------------+       +----------------+       | VLAN 20        |       +----------------+
                                         | G0/1: Trunk    |       | G0/1           |
                                         +----------------+       +----------------+
                                                                  |      R2        |
                                                                  | G0/0           |
                                                                  +----------------+
```
- **Devices**:
  - **PC1**: Data traffic, VLAN 10 (untagged), connected to PH1’s downlink.
  - **PH1**: IP phone, voice traffic, VLAN 20 (tagged, CoS 5, DSCP EF), connected to SW1 G0/0.
  - **SW1**: Switch, G0/0 (access, VLAN 10, voice VLAN 20), G0/1 (trunk to R1, VLANs 10, 20).
  - **R1, R2**: Routers, no dot1q tags between them (no PCP, only DSCP).
- **Note**: PH1 marks voice traffic as CoS 5 (PCP) and DSCP EF (46). SW1 trusts PH1’s markings (trust boundary at phone) but not PC1’s (re-marked to DSCP 0). Queuing (CBWFQ, LLQ) prioritizes voice on SW1/R1. Shaping/policing may limit traffic (e.g., 300 Mbps on R1 G0/1).
- **Requirements**:
  1. Understand classification (PCP, DSCP) and marking (e.g., CoS 5, DSCP EF for voice).
  2. Grasp queuing (multiple queues, CBWFQ, LLQ) and congestion (RED/WRED).
  3. Know policing (drops excess) vs. shaping (buffers excess).
  4. Recognize trust boundaries (trust PH1, not PC1).
  5. Verify with flashcards (e.g., AF11 = DSCP 10, formula 8X+2Y).

**Analogy**: QoS is like a restaurant prioritizing VIP orders (voice traffic, EF/CoS 5) by sorting orders (classification), labeling them (marking), queuing them in priority lines (CBWFQ/LLQ), and managing rush hours (policing/shaping) to ensure VIPs get served fast.

**Lifelong Retention Tip**: Visualize PC1 → PH1 → SW1 → R1, with PH1 marking voice as CoS 5/DSCP EF. Create flashcards: “CoS 5? Voice. DSCP EF? 46, voice. LLQ? Strict priority queue.” Practice in Packet Tracer (simulate voice traffic, observe queues). Review 3 times daily for a week.

## Classification and Marking

**Purpose**: Identify (classify) and label (mark) traffic to prioritize it (e.g., voice over data) during congestion.

**Key Points**:
- **Classification**: Organizes packets into traffic classes (e.g., voice, video, data) using:
  - **ACLs**: Identify traffic (e.g., permit voice traffic for high priority).
  - **NBAR**: Deep packet inspection (Layer 7, beyond CCNA scope).
  - **Header Fields**: PCP (Layer 2, dot1q tag) and DSCP (Layer 3, IP header).
- **Marking**: Sets values in PCP or DSCP fields to indicate priority.
- **PCP (Priority Code Point, CoS)**:
  - In 802.1Q tag (3 bits, values 0–7, defined by IEEE 802.1p).
  - **Standard Values**:
    - 0: Best effort (default, no QoS guarantees).
    - 3: Call signaling (e.g., IP phone setup).
    - 4: Video.
    - 5: Voice (audio packets).
  - Used on trunk links (G0/1) or access links with voice VLANs (G0/0).
  - **Limitation**: Requires dot1q tag; unavailable between R1–R2.
- **DSCP (Differentiated Services Code Point)**:
  - In IPv4 ToS byte (6 bits, values 0–63, RFC 2474).
  - **History**: ToS byte had IP Precedence (IPP, 3 bits, 0–7, similar to PCP).
  - **Current Use**: 6-bit DSCP + 2-bit ECN (Explicit Congestion Notification).
  - **Standard Markings** (RFC 4594):
    - **DF (Default Forwarding)**: DSCP 0, best effort (e.g., email, file downloads).
    - **EF (Expedited Forwarding)**: DSCP 46 (binary 101110), low delay/jitter/loss (voice).
    - **AF (Assured Forwarding)**: 12 values, 4 classes (1–4, higher = better priority), 3 drop precedences (1–3, higher = more likely to drop via WRED).
      - Format: AFXY (X = class, Y = drop precedence).
      - Formula: DSCP = 8X + 2Y (e.g., AF11 = 8*1 + 2*1 = 10).
      - Examples: AF11 (DSCP 10), AF12 (DSCP 12), AF23 (DSCP 22), AF32 (DSCP 28), AF43 (DSCP 38, best service).
    - **CS (Class Selector)**: 8 values (0–7), backward compatible with IPP.
      - Formula: DSCP = 8 * CS number (e.g., CS3 = 24, CS6 = 48).
    - Recommendations: Voice = EF (46), interactive video = AF4x, streaming video = AF3x, high-priority data = AF2x, best effort = DF (0).
- **Trust Boundaries**:
  - Define where markings are trusted (forwarded unchanged) or untrusted (re-marked).
  - Best Practice: Trust IP phone markings (e.g., EF/CoS 5 for voice), untrust PC markings (e.g., re-mark EF to DF).
  - Configured on switchport (e.g., SW1 G0/0 trusts PH1’s CoS 5).

**Demonstration**:
- **Topology**: PC1 (VLAN 10) → PH1 (VLAN 20, CoS 5, DSCP EF) → SW1 G0/0 → G0/1 (trunk) → R1.
- **Marking**:
  - PH1 marks voice traffic: CoS 5 (PCP), DSCP EF (46).
  - PC1 traffic untagged (no CoS), marked DSCP 0 (DF) by SW1.
- **Classification**:
  - SW1 classifies PH1’s traffic (CoS 5/EF) as high priority, PC1’s (DF) as low priority.
- **Trust Boundary**:
  - SW1 trusts PH1’s markings (CoS 5/EF), re-marks PC1’s traffic to DSCP 0.
- **Verification** (conceptual, no config for CCNA):
```plaintext
SW1# show class-map
Class Map match-all TEST
  Match dscp ef
```
- **Observations**:
  - CoS (PCP) used on G0/0 (voice VLAN) and G0/1 (trunk).
  - DSCP used across all links (e.g., R1–R2).
  - AF formula: DSCP = 8X + 2Y (e.g., AF23 = 8*2 + 2*3 = 22).
  - CS formula: DSCP = 8 * CS number (e.g., CS5 = 40).

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Introduced voice VLANs (VLAN 20, tagged), PoE, and QoS basics (delay ≤150ms, jitter ≤30ms, loss ≤1%). Part 2 adds PCP/DSCP for classification/marking.
- **VLAN Lecture (Day 20)**: PCP in 802.1Q tag (voice VLANs, trunks).
- **CDP Lecture (Day 25)**: CDP enables voice VLAN tagging, aiding CoS marking.
- **Syslog Lab**: QoS markings trigger Syslog (e.g., `%QOS-5-PACKET_DROPPED`).

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: Voice uses UDP (e.g., RTP), marked EF for low jitter.
- **Day 20 (VLANs)**: Voice VLANs enable CoS marking.
- **Day 44 (NAT)**: QoS may prioritize NAT’d voice traffic.

**Analogy**: Classification is like sorting mail (voice, video, data) into priority bins, and marking is like labeling them “urgent” (EF/CoS 5) or “standard” (DF/CoS 0). Trust boundaries ensure only trusted senders (PH1) keep their labels.

**Lifelong Retention Tip**: In Packet Tracer, simulate PH1 marking voice as CoS 5/EF, PC1 as DF. Flashcards: “CoS 5? Voice. DSCP EF? 46. AF23? DSCP 22 (8*2+2*3).” Practice binary-to-decimal (e.g., 101110 = 46). Review 3 times.

## Queuing and Congestion Management

**Purpose**: Manage traffic during congestion using multiple queues, scheduling (CBWFQ, LLQ), and congestion avoidance (RED/WRED).

**Key Points**:
- **Queuing Review (Part 1)**:
  - Packets queued when ingress rate exceeds egress rate.
  - Default: FIFO (first-in-first-out), single queue, tail drop when full.
  - Tail drop causes TCP global synchronization (all TCP hosts slow/increase in waves).
  - RED/WRED: Drop packets early to avoid tail drop (WRED prioritizes high-priority traffic).
- **Multiple Queues**:
  - Traffic classified (e.g., by DSCP) and placed in separate queues (e.g., voice, video, data).
  - Scheduler decides which queue to forward from (one packet at a time).
- **Scheduling Methods**:
  - **Weighted Round-Robin (WRR)**: Cycles through queues, sending more data from high-priority queues.
  - **CBWFQ (Class-Based Weighted Fair Queuing)**: WRR with guaranteed bandwidth per queue (e.g., 20% for voice).
  - **LLQ (Low Latency Queuing)**: Strict priority queue (e.g., for voice, EF); sends all packets before others, but risks starving other queues.
- **Process**:
  1. Ingress traffic classified (e.g., DSCP EF for voice).
  2. Routed to egress interface (e.g., R1 G0/1).
  3. Placed in queues (e.g., voice in LLQ, data in CBWFQ).
  4. Scheduler forwards packets (LLQ first, then WRR for others).
  5. RED/WRED prevents tail drop within queues.
- **Example**:
  - Voice (EF) in strict priority queue (LLQ).
  - Video (AF4x), data (AF2x, DF) in CBWFQ queues with bandwidth guarantees.
  - LLQ ensures low delay/jitter for voice but needs policing to avoid starving others.

**Demonstration**:
- **Topology**: PC1 → PH1 → SW1 → R1 → R2.
- **Scenario**: R1’s G0/1 congested, ingress from G0/0 exceeds egress capacity.
- **Queuing**:
  - PH1’s voice traffic (EF) in LLQ (strict priority).
  - PC1’s data (DF) in CBWFQ queue (low priority).
  - Scheduler sends all LLQ packets, then WRR for CBWFQ queues.
- **Congestion**:
  - RED/WRED drops low-priority packets (e.g., DF) before LLQ queue fills.
- **Verification** (conceptual):
  - Observe voice packets (EF) forwarded first, low delay/jitter.
  - Data packets (DF) queued, possibly dropped by WRED.

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Introduced FIFO, tail drop, RED/WRED. Part 2 adds multiple queues, CBWFQ, LLQ.
- **TCP Lecture (Day 15)**: Tail drop affects TCP sliding window; LLQ prioritizes UDP-based voice.
- **Syslog Lab**: Queue drops trigger Syslog (e.g., `%QOS-5-PACKET_DROPPED`).

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: LLQ prioritizes UDP voice (e.g., RTP).
- **Day 20 (VLANs)**: Voice VLANs (VLAN 20) feed LLQ.
- **Day 30 (Ports)**: Voice traffic (e.g., UDP 5060 for SIP) in LLQ.

**Analogy**: Queuing is like a supermarket with priority lanes (LLQ for voice) and regular lanes (CBWFQ for data), with a cashier (scheduler) serving VIPs first and WRED tossing low-priority items during a rush.

**Lifelong Retention Tip**: Simulate congestion in Packet Tracer, prioritize voice (EF) in LLQ. Flashcards: “LLQ? Strict priority, voice. CBWFQ? Bandwidth guarantee, WRR. WRED? Drops low-priority.” Practice 3 times.

## Policing and Shaping

**Purpose**: Control traffic rates to prevent congestion or align with service agreements (e.g., ISP limits).

**Key Points**:
- **Policing**:
  - Drops traffic exceeding configured rate (e.g., 300 Mbps).
  - Allows short bursts (configurable) for bursty data (e.g., HTTP).
  - Example: ISP polices R1’s G0/1 inbound to 300 Mbps.
- **Shaping**:
  - Buffers traffic exceeding configured rate, queuing it for later.
  - Example: R1 shapes G0/1 outbound to 300 Mbps to match ISP’s policing.
- **Use Case**:
  - R1 (customer) shapes outbound to 300 Mbps to avoid ISP dropping packets.
  - ISP polices inbound to enforce 300 Mbps contract.
- **Classification**: Applies different rates to different traffic (e.g., voice vs. data).
- **LLQ Protection**: Policing limits LLQ traffic to prevent starving other queues.

**Demonstration**:
- **Topology**: R1 (customer) → R2 (ISP).
- **Scenario**: R1’s G0/1 (GigabitEthernet) shaped to 300 Mbps outbound; R2 polices inbound to 300 Mbps.
- **Shaping (R1)**:
  - Buffers excess traffic (e.g., voice bursts) in queues.
- **Policing (R2)**:
  - Drops traffic above 300 Mbps, allows short bursts.
- **Observation**:
  - Voice (EF) prioritized in LLQ, shaped/policed to avoid overuse.
  - Data (DF) queued or dropped if exceeding 300 Mbps.

**Comparison to Prior Content**:
- **QoS Part 1 Lecture**: Policing similar to PoE policing (`power inline police`, err-disable for excess).
- **Syslog Lab**: Policing/shaping drops trigger Syslog (e.g., `%QOS-5-PACKET_DROPPED`).
- **TCP Lecture (Day 15)**: Policing affects TCP retransmissions; shaping buffers TCP traffic.

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: Policing impacts bursty TCP; shaping benefits UDP voice.
- **Day 44 (NAT)**: Shaping/policing may apply to NAT’d traffic.
- **Day 30 (Ports)**: Policing/shaping on voice ports (e.g., UDP 5060).

**Analogy**: Policing is like a strict bouncer (drops excess guests), while shaping is a polite usher (queues excess guests) at a concert (network) with limited seats (bandwidth).

**Lifelong Retention Tip**: Simulate shaping/policing in Packet Tracer, limit voice traffic rate. Flashcards: “Policing? Drops excess. Shaping? Buffers excess. Use? ISP bandwidth limits.” Practice 3 times.

## Common Issues and Troubleshooting

- **Incorrect Marking**:
  - **Issue**: Voice traffic not prioritized (not EF/CoS 5).
  - **Cause**: PC1 marks traffic as EF; SW1 trusts it.
  - **Fix**: Set trust boundary at PH1, re-mark PC1 traffic to DF (conceptual).
- **Queue Starvation**:
  - **Issue**: Data traffic delayed (LLQ overused).
  - **Cause**: Voice traffic fills LLQ, starves CBWFQ queues.
  - **Fix**: Police LLQ traffic to limit bandwidth (conceptual).
- **Congestion Drops**:
  - **Issue**: Voice packets dropped despite EF marking.
  - **Cause**: No LLQ; FIFO queue with tail drop.
  - **Fix**: Use LLQ for EF traffic, WRED for others (conceptual).
- **Shaping/Policing Mismatch**:
  - **Issue**: R1’s traffic dropped by R2.
  - **Cause**: R1 shapes to 500 Mbps, R2 polices to 300 Mbps.
  - **Fix**: Align R1’s shaping to 300 Mbps.

**Analogy**: Troubleshooting QoS is like fixing a VIP event where VIPs (voice) need priority service (LLQ), labels (EF) must be trusted, and crowd control (policing/shaping) prevents chaos.

**Lifelong Retention Tip**: Simulate LLQ starvation in Packet Tracer, apply policing. Checklist: “No voice priority? Check markings, LLQ, policing.” Practice 3 times.

## Practice Commands (Conceptual, Not Required for CCNA)

- **Class Map for Classification**:
```plaintext
SW1# configure terminal
SW1(config)# class-map TEST
SW1(config-cmap)# match dscp ef
SW1(config-cmap)# exit
```
- **Verify Markings** (simulation mode):
  - Check PH1’s traffic: CoS 5, DSCP EF (46).
  - Check PC1’s traffic: DSCP 0 (DF).
- **Verify Queuing** (conceptual):
```plaintext
R1# show queueing interface g0/1
```

## Quiz Answers and Explanations

1. **Which CoS markings are consistent with standard practice? (Select three)**  
   - **A**: CoS 1 for best effort.  
   - **B**: CoS 0 for best effort.  
   - **C**: CoS 3 for video.  
   - **D**: CoS 5 for voice.  
   - **E**: CoS 4 for video.  
   - **Answer**: B, D, E (CoS 0 = best effort, CoS 4 = video, CoS 5 = voice).  

2. **What bit pattern is in the DSCP field for EF?**  
   - **A**: 000 000.  
   - **B**: 001 010.  
   - **C**: 101 010.  
   - **D**: 101 110.  
   - **Answer**: D (EF = DSCP 46, binary 101110).  

3. **Which AF marking provides the best service?**  
   - **A**: AF43.  
   - **B**: AF41.  
   - **C**: AF51.  
   - **D**: AF61.  
   - **Answer**: B (AF41 = class 4, drop precedence 1; highest priority, lowest drop chance).  

4. **Which statement represents QoS best practice?**  
   - **A**: Trust markings from IP phones, don’t trust PCs.  
   - **B**: Trust all markings.  
   - **C**: Don’t trust any markings.  
   - **D**: Trust PC markings, not IP phones.  
   - **Answer**: A (Trust PH1’s EF/CoS 5, re-mark PC1 to DF).  

5. **Which creates a strict priority queue for low delay/jitter/loss?**  
   - **A**: CBWFQ.  
   - **B**: LLQ.  
   - **C**: WRED.  
   - **D**: FIFO.  
   - **Answer**: B (LLQ prioritizes voice, minimizes delay/jitter).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: What is the purpose of LLQ?
    - **A**: Drop low-priority packets.  
    - **B**: Guarantee bandwidth for all queues.  
    - **C**: Prioritize low-latency traffic.  
    - **D**: Buffer excess traffic.  
    - **Answer**: C (LLQ ensures low delay/jitter for voice).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “CoS 5? Voice. EF? DSCP 46. LLQ? Strict priority”). Practice binary conversions (e.g., AF23 = 001011 = 22). Review 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Classification/Marking: “PCP, DSCP, Classify, Mark” (PDCM).
   - Queuing: “CBWFQ, LLQ, Prioritize Voice” (CLPV).
   - Shaping/Policing: “Shape Buffers, Police Drops” (SBPD).
2. **Visualize Topology**:
   - Draw PC1 → PH1 → SW1 → R1, with voice (EF/CoS 5) in LLQ. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “CoS 5? Voice. DSCP EF? 46. AF41? DSCP 34, best service. LLQ? Strict priority.”
4. **Practice in Packet Tracer**:
   - Simulate voice (EF) in LLQ, data (DF) in CBWFQ. Observe prioritization. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: CoS (Layer 2, voice VLANs) vs. DSCP (Layer 3, all links) vs. LLQ (voice priority).
   - Quiz: “EF? DSCP 46, voice. LLQ? Low latency for voice.”
6. **Troubleshooting Practice**:
   - Simulate PC1 marking EF, re-mark to DF. Checklist: “No voice priority? Check markings, LLQ, trust.”
7. **Teach Someone**:
   - Explain LLQ and trust boundaries to a peer.
8. **Spaced Repetition**:
   - Use Anki for CoS/DSCP values, AF formula, LLQ. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **Classification and Marking**:
   - Classify traffic (ACLs, NBAR, PCP, DSCP); mark with CoS (0–7, e.g., 5 for voice) or DSCP (0–63, e.g., EF = 46).
   - AF: 4 classes, 3 drop precedences (DSCP = 8X+2Y, e.g., AF23 = 22).
   - CS: Backward compatible with IPP (DSCP = 8 * CS, e.g., CS5 = 40).
2. **Trust Boundaries**:
   - Trust IP phone markings (EF/CoS 5), re-mark PC traffic to DF.
3. **Queuing and Congestion**:
   - Multiple queues, CBWFQ (bandwidth guarantees), LLQ (strict priority for voice).
   - RED/WRED prevent tail drop; LLQ risks starving other queues.
4. **Policing and Shaping**:
   - Policing drops excess traffic; shaping buffers it.
   - Used to match ISP bandwidth limits (e.g., 300 Mbps).
5. **CCNA Focus**:
   - Topic 4.7: Understand CoS/DSCP, CBWFQ/LLQ, policing/shaping concepts.

These notes provide a comprehensive foundation for QoS Part 2, preparing you for CCNA exam topic 4.7. Practice marking/queuing in Packet Tracer, use flashcards for DSCP/AF values, and apply analogies/retention tips to master these concepts for life!