# QoS Part 2 Lab Notes for CCNA Beginners

## Lab Overview

This lab provides hands-on experience with basic **Quality of Service (QoS)** configurations on a Cisco router (R1) using **Packet Tracer**, reinforcing concepts from the QoS Part 2 lecture for **CCNA exam topic 4.7** ("Describe the purpose of First Hop Redundancy Protocols and QoS"). Although QoS configuration is not required for the CCNA exam, this lab helps solidify understanding of **classification**, **marking**, **queuing**, and **per-hop behavior (PHB)** by marking and prioritizing traffic (HTTPS, HTTP, ICMP) on R1’s outbound interface (G0/0/0). The lab configures **class maps** to identify traffic, **policy maps** to mark and prioritize it, and a **service policy** to apply the rules, then verifies markings in simulation mode.

**Lab Objectives**:
- **Configure QoS on R1**: Use class maps, policy maps, and service policies to mark and prioritize traffic.
- **Mark Traffic**: Set DSCP values (AF31 for HTTPS, AF32 for HTTP, CS2 for ICMP).
- **Prioritize Traffic**: Assign HTTPS to a priority queue (10% bandwidth), HTTP and ICMP to non-priority queues (10% and 5% bandwidth).
- **Understand PHB**: Recognize that QoS markings apply per hop (R1 to R2); R2 needs its own QoS configuration.
- **Verify Markings**: Check DSCP values in Packet Tracer’s simulation mode.
- **Prepare for Exam**: Reinforce QoS concepts (classification, marking, queuing) without memorizing configurations (topic 4.7).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      R1        |-------|      R2        |
| 10.0.0.11      |       | G0/0           |       | G0/0/0         |       | G0/0/0         |
|                |       |                |       | (QoS Applied)  |       |                |
+----------------+       +----------------+       +----------------+       +----------------+
                                                                  |                |
                                                                  +----------------+       +----------------+
                                                                  |      SW2       |-------|      SRV1      |
                                                                  | G0/0           |       | 10.0.0.100     |
                                                                  |                |       | (jeremysitlab.com)
                                                                  +----------------+       +----------------+
```
- **Devices**:
  - **PC1**: Sends HTTPS, HTTP, and ICMP (ping) traffic to SRV1 (10.0.0.100, jeremysitlab.com).
  - **SW1**: Switch connecting PC1 to R1 (no QoS config in lab).
  - **R1**: Applies QoS on G0/0/0 (outbound): marks HTTPS (AF31, priority queue, 10% bandwidth), HTTP (AF32, 10% bandwidth), ICMP (CS2, 5% bandwidth).
  - **R2**: Receives traffic from R1; no QoS config (treats all traffic equally unless configured).
  - **SW2**: Connects R2 to SRV1 (no QoS config).
  - **SRV1**: Server at 10.0.0.100, responds to HTTPS, HTTP, and ICMP.
- **Note**: Network assumed congested (multiple PCs implied). R1’s QoS ensures HTTPS priority. PHB means R2 ignores R1’s markings unless configured. Default DSCP is 0 (DF) before QoS config.
- **Requirements**:
  1. Configure class maps to identify HTTPS, HTTP, and ICMP traffic.
  2. Configure policy map to mark HTTPS (AF31), HTTP (AF32), ICMP (CS2), and assign bandwidth/priority.
  3. Apply policy map outbound on R1’s G0/0/0.
  4. Verify DSCP markings (AF31=26, AF32=28, CS2=16) in simulation mode.
  5. Understand PHB: R1’s QoS applies only to R1–R2 hop.

**Analogy**: QoS configuration is like a chef (R1) labeling orders (marking: HTTPS=AF31, HTTP=AF32, ICMP=CS2) and prioritizing VIP orders (HTTPS in priority queue) in a busy kitchen (congested network), but the next restaurant (R2) needs its own rules to respect the labels.

**Lifelong Retention Tip**: Visualize PC1 → R1 → SRV1, with R1 marking/prioritizing traffic. In Packet Tracer, configure class/policy maps, ping SRV1, check DSCP in simulation mode. Create flashcards: “AF31? DSCP 26, HTTPS. CS2? DSCP 16, ICMP. Priority queue? LLQ for HTTPS.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Verify Default Markings
**Objective**: Confirm that PC1’s traffic (ICMP, HTTP, HTTPS) has no DSCP markings (default DSCP 0) before QoS configuration.

**Steps (on PC1)**:
1. Ping SRV1 (realtime mode):
```plaintext
PC1> ping jeremysitlab.com
Pinging 10.0.0.100 with 32 bytes of data:
Reply from 10.0.0.100: bytes=32 time=10ms TTL=254
Reply from 10.0.0.100: bytes=32 time=8ms TTL=254
Reply from 10.0.0.100: bytes=32 time=9ms TTL=254
Reply from 10.0.0.100: bytes=32 time=11ms TTL=254
Ping statistics: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
2. Ping SRV1 (simulation mode):
```plaintext
PC1> ping 10.0.0.100
```
- Check packet at R1 (outbound PDU details):
  - DSCP: 0x00 (hex, 0 in decimal, Default Forwarding).
- Note: HTTP/HTTPS also DSCP 0 by default (not shown for time).

**Observations**:
- **Default Markings**: PC1 sends ICMP, HTTP, HTTPS with DSCP 0 (DF, no priority).
- **No QoS**: R1/SW1 do not add markings by default.
- **DNS Query**: PC1 queries SRV1 (10.0.0.100) for jeremysitlab.com; Packet Tracer lacks DNS caching.
- **Exam Note**: Understand default DSCP 0 for unmarked traffic (topic 4.7).
- **PHB**: R1’s lack of QoS means no prioritization; R2 treats all traffic equally.

**Comparison to Prior Content**:
- **QoS Part 2 Lecture**: Default DSCP 0 aligns with DF (best effort, no QoS).
- **QoS Part 1 Lecture**: Unmarked traffic (DSCP 0) uses FIFO queue, prone to tail drop.
- **Syslog Lab**: Ping triggers Syslog (e.g., `%SYS-5-CONFIG_I` if interfaces change).

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: ICMP (ping), HTTP (TCP 80), HTTPS (TCP 443) unmarked by default.
- **Day 10 (IP Addressing)**: PC1 (10.0.0.11), SRV1 (10.0.0.100) in same subnet.
- **Day 30 (Ports)**: HTTP (80), HTTPS (443) identified for QoS classification.

**Analogy**: Default traffic (DSCP 0) is like unlabeled orders in a kitchen, treated equally (FIFO) with no priority until the chef (R1) labels and sorts them.

**Lifelong Retention Tip**: In Packet Tracer, ping SRV1, check DSCP 0 in simulation mode. Flashcard: “Default DSCP? 0, DF, no priority.” Practice 3 times.

### Step 2: Configure QoS on R1
**Objective**: Configure class maps to identify HTTPS, HTTP, and ICMP traffic; policy map to mark and prioritize; and service policy to apply outbound on G0/0/0.

**Steps (on R1)**:
1. Configure class maps:
```plaintext
R1# configure terminal
R1(config)# class-map HTTPS_MAP
R1(config-cmap)# match protocol https
R1(config-cmap)# exit
R1(config)# class-map HTTP_MAP
R1(config-cmap)# match protocol http
R1(config-cmap)# exit
R1(config)# class-map ICMP_MAP
R1(config-cmap)# match protocol icmp
R1(config-cmap)# exit
```
2. Configure policy map:
```plaintext
R1(config)# policy-map G0/0/0_OUT
R1(config-pmap)# class HTTPS_MAP
R1(config-pmap-c)# set ip dscp af31
R1(config-pmap-c)# priority percent 10
R1(config-pmap-c)# exit
R1(config-pmap)# class HTTP_MAP
R1(config-pmap-c)# set ip dscp af32
R1(config-pmap-c)# bandwidth percent 10
R1(config-pmap-c)# exit
R1(config-pmap)# class ICMP_MAP
R1(config-pmap-c)# set ip dscp cs2
R1(config-pmap-c)# bandwidth percent 5
R1(config-pmap-c)# exit
R1(config-pmap)# exit
```
3. Apply service policy:
```plaintext
R1(config)# interface g0/0/0
R1(config-if)# service-policy output G0/0/0_OUT
R1(config-if)# exit
R1(config)# exit
```
4. Verify configuration:
```plaintext
R1# show running-config | section class-map
class-map match-all HTTPS_MAP
 match protocol https
class-map match-all HTTP_MAP
 match protocol http
class-map match-all ICMP_MAP
 match protocol icmp
R1# show running-config | section policy-map
policy-map G0/0/0_OUT
 class HTTPS_MAP
  set ip dscp af31
  priority percent 10
 class HTTP_MAP
  set ip dscp af32
  bandwidth percent 10
 class ICMP_MAP
  set ip dscp cs2
  bandwidth percent 5
```

**Observations**:
- **Class Maps**:
  - Identify traffic: HTTPS (TCP 443), HTTP (TCP 80), ICMP.
  - Default `match-all`: Traffic must match all conditions (single condition here, so irrelevant).
  - Alternative: `match-any` for multiple conditions (e.g., match HTTP or HTTPS).
- **Policy Map**:
  - **HTTPS_MAP**: Marks AF31 (DSCP 26), priority queue (LLQ, 10% bandwidth).
  - **HTTP_MAP**: Marks AF32 (DSCP 28), non-priority queue (CBWFQ, 10% bandwidth).
  - **ICMP_MAP**: Marks CS2 (DSCP 16), non-priority queue (CBWFQ, 5% bandwidth).
  - Unmatched traffic: DSCP 0 (DF), no priority.
- **Service Policy**: Applies G0/0/0_OUT outbound on G0/0/0.
- **PHB**: R1 marks/prioritizes traffic for R1–R2 hop; R2 needs QoS config to honor markings.
- **Exam Note**: Configurations not required for CCNA, but understand class maps (classify), policy maps (mark/prioritize), and PHB (topic 4.7).

**Comparison to Prior Content**:
- **QoS Part 2 Lecture**: Lab applies classification (class maps), marking (AF31, AF32, CS2), queuing (LLQ for HTTPS, CBWFQ for HTTP/ICMP).
- **QoS Part 1 Lecture**: Builds on FIFO/RED/WRED; lab uses LLQ/CBWFQ for prioritization.
- **Syslog Lab**: QoS config triggers Syslog (e.g., `%SYS-5-CONFIG_I`).

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: HTTPS (TCP 443), HTTP (TCP 80), ICMP classified for QoS.
- **Day 44 (NAT)**: QoS may prioritize NAT’d traffic.
- **Day 30 (Ports)**: Protocol matching (http, https) uses port numbers.

**Analogy**: Configuring QoS is like a chef (R1) sorting orders into VIP (HTTPS, LLQ) and regular (HTTP/ICMP, CBWFQ) lines, labeling them (AF31, AF32, CS2) for the next restaurant (R2).

**Lifelong Retention Tip**: In Packet Tracer, configure class/policy maps, verify with `show running-config`. Flashcard: “Class map? Identifies traffic. Policy map? Marks/prioritizes. LLQ? HTTPS, 10%.” Practice 3 times.

### Step 3: Test and Verify Markings
**Objective**: Send HTTPS, HTTP, and ICMP traffic from PC1 to SRV1, verify DSCP markings in simulation mode.

**Steps (on PC1)**:
1. Ping SRV1 (simulation mode):
```plaintext
PC1> ping 10.0.0.100
```
- Check packet at R1 (outbound PDU details):
  - DSCP: 0x10 (hex, 16 decimal, CS2).
  - Binary: 010000 (3 left bits = CS2, 3 right bits = 0).
2. Send HTTP traffic (realtime, then simulation mode):
```plaintext
PC1> exit
[Open web browser, enter http://10.0.0.100]
[Switch to simulation mode, click Go]
```
- Check packet at R1 (outbound PDU details):
  - DSCP: 0x1c (hex, 28 decimal, AF32).
  - Binary: 011100 (class 3, drop precedence 2; 8*3 + 2*2 = 28).
3. Send HTTPS traffic (realtime, then simulation mode):
```plaintext
[In web browser, enter https://10.0.0.100]
[Switch to simulation mode, click Go]
```
- Check packet at R1 (outbound PDU details):
  - DSCP: 0x1a (hex, 26 decimal, AF31).
  - Binary: 011010 (class 3, drop precedence 1; 8*3 + 2*1 = 26).

**Observations**:
- **Markings**:
  - ICMP: CS2 (DSCP 16, hex 0x10, binary 010000).
  - HTTP: AF32 (DSCP 28, hex 0x1c, binary 011100).
  - HTTPS: AF31 (DSCP 26, hex 0x1a, binary 011010).
- **Verification**:
  - Inbound DSCP: 0 (DF) for all traffic (PC1 sends unmarked).
  - Outbound DSCP: Set by R1’s policy map (AF31, AF32, CS2).
- **Queuing**:
  - HTTPS: LLQ (priority queue, 10% bandwidth, low delay).
  - HTTP: CBWFQ (10% bandwidth).
  - ICMP: CBWFQ (5% bandwidth).
- **PHB**: R2 ignores markings unless configured with QoS.
- **Packet Tracer Note**: Displays DSCP as 8-bit hex (includes 2-bit ECN, always 00); focus on 6-bit DSCP.
- **Exam Note**: Understand marking (DSCP values) and PHB (topic 4.7).

**Comparison to Prior Content**:
- **QoS Part 2 Lecture**: Lab confirms DSCP markings (AF31=26, AF32=28, CS2=16), LLQ for HTTPS, CBWFQ for HTTP/ICMP.
- **QoS Part 1 Lecture**: Builds on FIFO; lab uses LLQ/CBWFQ to avoid tail drop.
- **Syslog Lab**: Traffic triggers Syslog (e.g., `%QOS-5-PACKET_DROPPED` if congested).

**Comparison to Other CCNA Topics**:
- **Day 15 (TCP/UDP)**: HTTPS (TCP 443), HTTP (TCP 80), ICMP classified.
- **Day 10 (IP Addressing)**: Same subnet (10.0.0.0/24) for PC1, SRV1.
- **Day 30 (Ports)**: Protocol matching uses port numbers (80, 443).

**Analogy**: Verifying markings is like checking if the chef (R1) labeled orders correctly (HTTPS=AF31, HTTP=AF32, ICMP=CS2) before sending them to the next restaurant (R2).

**Lifelong Retention Tip**: In Packet Tracer, send HTTP/HTTPS/ICMP, check DSCP in simulation mode. Flashcard: “AF31? DSCP 26, binary 011010. CS2? DSCP 16, binary 010000.” Practice binary-to-hex (e.g., 011100 = 28 = 0x1c). Review 3 times.

## Common Issues and Troubleshooting

- **Incorrect Markings**:
  - **Issue**: HTTPS not marked AF31 (DSCP 26).
  - **Cause**: Wrong `match protocol https` or policy map error.
  - **Fix**: Verify `class-map HTTPS_MAP`, `set ip dscp af31` in policy map.
- **Priority Queue Failure**:
  - **Issue**: HTTPS not prioritized.
  - **Cause**: Missing `priority percent 10` or service policy not applied.
  - **Fix**: Check `policy-map G0/0/0_OUT`, `service-policy output G0/0/0_OUT`.
- **Traffic Not Matched**:
  - **Issue**: HTTP traffic unmarked (DSCP 0).
  - **Cause**: Incorrect `match protocol http` or `match-all` mismatch.
  - **Fix**: Verify `class-map HTTP_MAP`, consider `match-any` for multiple protocols.
- **PHB Issue**:
  - **Issue**: R2 ignores R1’s markings.
  - **Cause**: R2 lacks QoS config (normal for PHB).
  - **Fix**: Configure QoS on R2 (beyond lab scope).
- **Simulation Mode Error**:
  - **Issue**: DSCP not visible.
  - **Cause**: Checking inbound PDU instead of outbound.
  - **Fix**: Check outbound PDU details at R1’s G0/0/0.

**Analogy**: Troubleshooting QoS is like ensuring a chef (R1) correctly labels (marks) and prioritizes (queues) orders, checking that VIP orders (HTTPS) get served first, and confirming the next restaurant (R2) needs its own rules.

**Lifelong Retention Tip**: Simulate marking failure (wrong DSCP), fix policy map, verify in simulation mode. Checklist: “No priority? Check class map, policy map, service policy, PHB.” Practice 3 times.

## Practice Commands (Conceptual, Not Required for CCNA)

- **Class Maps**:
```plaintext
R1# configure terminal
R1(config)# class-map HTTPS_MAP
R1(config-cmap)# match protocol https
R1(config-cmap)# exit
R1(config)# class-map HTTP_MAP
R1(config-cmap)# match protocol http
R1(config-cmap)# exit
R1(config)# class-map ICMP_MAP
R1(config-cmap)# match protocol icmp
R1(config-cmap)# exit
```
- **Policy Map**:
```plaintext
R1(config)# policy-map G0/0/0_OUT
R1(config-pmap)# class HTTPS_MAP
R1(config-pmap-c)# set ip dscp af31
R1(config-pmap-c)# priority percent 10
R1(config-pmap-c)# exit
R1(config-pmap)# class HTTP_MAP
R1(config-pmap-c)# set ip dscp af32
R1(config-pmap-c)# bandwidth percent 10
R1(config-pmap-c)# exit
R1(config-pmap)# class ICMP_MAP
R1(config-pmap-c)# set ip dscp cs2
R1(config-pmap-c)# bandwidth percent 5
R1(config-pmap)# exit
R1(config-pmap)# exit
```
- **Service Policy**:
```plaintext
R1(config)# interface g0/0/0
R1(config-if)# service-policy output G0/0/0_OUT
R1(config-if)# exit
```
- **Verify Configurations**:
```plaintext
R1# show running-config | section class-map
R1# show running-config | section policy-map
```
- **Test Traffic**:
```plaintext
PC1> ping 10.0.0.100
PC1> [Web browser: http://10.0.0.100]
PC1> [Web browser: https://10.0.0.100]
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - QoS Config: “Classify, Mark, Queue, Apply” (CMQA).
   - Markings: “HTTPS AF31, HTTP AF32, ICMP CS2” (HAI).
   - PHB: “Per-Hop Behavior, Configure Each Device” (PHBC).
2. **Visualize Topology**:
   - Draw PC1 → R1 → SRV1, with R1 marking HTTPS (AF31), HTTP (AF32), ICMP (CS2). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “AF31? DSCP 26, binary 011010. AF32? DSCP 28, binary 011100. CS2? DSCP 16, binary 010000. HTTPS? LLQ, 10%.”
4. **Practice in Packet Tracer**:
   - Configure QoS, send HTTP/HTTPS/ICMP, check DSCP in simulation mode. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: Class map (classify) vs. Policy map (mark/queue) vs. Service policy (apply).
   - Quiz: “AF31? DSCP 26. LLQ? HTTPS priority. PHB? Per-hop config.”
6. **Troubleshooting Practice**:
   - Simulate wrong DSCP, fix policy map, verify. Checklist: “No markings? Check class map, policy map, service policy.”
7. **Teach Someone**:
   - Explain QoS config and PHB to a peer.
8. **Spaced Repetition**:
   - Use Anki for DSCP values (AF31=26, AF32=28, CS2=16), LLQ, PHB. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **QoS Configuration** (not required for CCNA):
   - **Class Maps**: Identify traffic (HTTPS, HTTP, ICMP).
   - **Policy Maps**: Mark (AF31, AF32, CS2) and prioritize (LLQ for HTTPS, CBWFQ for HTTP/ICMP).
   - **Service Policy**: Apply outbound on G0/0/0.
2. **Markings**:
   - HTTPS: AF31 (DSCP 26, hex 0x1a).
   - HTTP: AF32 (DSCP 28, hex 0x1c).
   - ICMP: CS2 (DSCP 16, hex 0x10).
3. **Queuing**:
   - HTTPS: LLQ (priority, 10% bandwidth).
   - HTTP/ICMP: CBWFQ (10%, 5% bandwidth).
4. **PHB**:
   - R1’s QoS applies only to R1–R2 hop; R2 needs own config.
5. **CCNA Focus**:
   - Topic 4.7: Understand classification, marking, queuing, PHB (not configuration).

These notes provide a complete foundation for the QoS Part 2 lab, reinforcing CCNA exam topic 4.7. Practice QoS config in Packet Tracer, verify DSCP markings in simulation mode, and use flashcards/analogies to master concepts for life!