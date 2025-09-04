# Kali Linux Demo Lab Notes for CCNA Beginners

## Lab Overview

This lab demonstrates a **DHCP exhaustion attack** (also known as a DHCP starvation attack) using **Kali Linux** and **Yersinia** in **EVE-NG**, a network simulator, to reinforce concepts from the **Security Fundamentals lecture** (CCNA exam topic 5.1: "Describe key security concepts"). Unlike typical CCNA labs, this demo uses EVE-NG instead of Packet Tracer due to the need for Kali Linux, a cybersecurity-focused distribution with pre-installed penetration testing tools. The lab shows how an attacker depletes a DHCP server’s IP pool, causing a **Denial-of-Service (DoS)** attack, and highlights its impact on the **CIA triad** (Availability). While no Packet Tracer file is provided, the demo prepares you for the next lecture on **port security** to mitigate such attacks.

**Lab Objectives**:
- **Demonstrate DHCP Exhaustion**: Use Kali Linux’s Yersinia to send spoofed DHCP Discover messages, exhausting R1’s DHCP pool.
- **Understand Impact**: Observe how PC1 fails to obtain an IP address, affecting Availability (CIA triad).
- **Verify Attack**: Check R1’s DHCP pool and bindings to confirm depletion.
- **Recover Network**: Stop the attack, clear bindings, and restore PC1’s connectivity.
- **Prepare for CCNA**: Reinforce topic 5.1 (security concepts, DoS attacks) and connect to DHCP (Day 13) and Security Fundamentals.
- **Learn Mitigation (Future)**: Understand that port security (next lecture) prevents such attacks.

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      PC1       |-------|      SW1       |-------|      R1        |
| (DHCP Client)  |       | G0/1           |       | G0/0 (DHCP Server)
|                |       |                |       | 192.168.1.1    |
+----------------+       +----------------+       +----------------+
                       |
                       | G0/2
                       |
                       +----------------+
                       |   Attacker     |
                       | (Kali Linux)   |
                       +----------------+
```
- **Devices**:
  - **PC1**: DHCP client, attempts to obtain an IP from R1.
  - **SW1**: Switch, connects PC1 (G0/1) and Attacker (G0/2) to R1 (G0/0).
  - **R1**: DHCP server, pool 192.168.1.0/24 (253 assignable IPs, 192.168.1.2–254), default gateway 192.168.1.1, DNS 8.8.8.8.
  - **Attacker**: Kali Linux PC, uses Yersinia to send spoofed DHCP Discover messages.
- **Note**: Attacker spoofs MAC addresses, exhausting R1’s DHCP pool, preventing PC1 from getting an IP (DoS). Recovery involves stopping the attack and clearing bindings. No configuration is performed in this demo.
- **Requirements**:
  1. Run DHCP exhaustion attack using Yersinia in EVE-NG.
  2. Verify R1’s DHCP pool depletion (253 leased IPs).
  3. Confirm PC1’s failure to obtain an IP (autoconfigures 169.254.x.x).
  4. Stop attack, clear bindings, restore PC1’s connectivity.
  5. Understand attack’s impact on Availability (CIA) and mitigation via port security (future lecture).

**Analogy**: A DHCP exhaustion attack is like a prankster (Attacker) reserving all seats (IP addresses) in a theater (R1’s DHCP pool) with fake names (spoofed MACs), leaving no seats for real guests (PC1), blocking their entry (DoS).

**Lifelong Retention Tip**: Visualize Attacker flooding R1, PC1 stuck with 169.254.x.x. Create flashcards: “DHCP exhaustion? DoS, Availability. Yersinia? Kali Linux attack tool. 169.254? Link-local, DHCP failure.” Simulate in EVE-NG or Packet Tracer (if Kali unavailable, conceptualize). Review 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Verify R1’s DHCP Configuration
**Objective**: Confirm R1’s DHCP pool settings before the attack.

**Steps (on R1)**:
1. Check DHCP configuration:
```plaintext
R1# enable
R1# show running-config | section dhcp
ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
```
2. Check DHCP pool status:
```plaintext
R1# show ip dhcp pool
Pool POOL1 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 254
 Leased addresses               : 0
 Excluded addresses             : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.1          192.168.1.1 - 192.168.1.254         0
```
3. Check DHCP bindings:
```plaintext
R1# show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
```

**Observations**:
- **DHCP Pool**: 192.168.1.0/24, 254 total addresses, 253 assignable (192.168.1.2–254, 192.168.1.1 reserved for R1).
- **Default Gateway**: 192.168.1.1 (R1’s G0/0).
- **DNS Server**: 8.8.8.8 (Google’s DNS).
- **Status**: 0 leased addresses, no bindings (no clients yet).
- **Exam Note**: DHCP exhaustion targets Availability (CIA triad, topic 5.1). No excluded addresses configured (all 253 IPs available).

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: DHCP exhaustion is a spoofing/DoS attack, affecting Availability.
- **Day 13 (DHCP)**: Covers DHCP pool setup, Discover/Offer/Request/Ack (DORA) process.
- **Syslog Lab**: DHCP activity triggers Syslog (e.g., `%DHCPD-4-REQUEST`).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: 192.168.1.0/24 (private), 169.254.x.x (link-local for DHCP failure).
- **Day 18 (ARP)**: Spoofed MACs in DHCP attack similar to ARP spoofing.
- **Day 15 (TCP/UDP)**: DHCP uses UDP 67 (server), 68 (client).

**Analogy**: R1’s DHCP pool is like a ticket booth with 253 tickets (IPs). Before the attack, no tickets are taken (0 leased).

**Lifelong Retention Tip**: In Packet Tracer, configure R1 as DHCP server, check `show ip dhcp pool`. Flashcard: “DHCP pool? 192.168.1.2–254, 253 IPs. DORA? Discover, Offer, Request, Ack.” Practice 3 times.

### Step 2: Perform DHCP Exhaustion Attack
**Objective**: Use Kali Linux’s Yersinia to send spoofed DHCP Discover messages, depleting R1’s DHCP pool.

**Steps (on Attacker - Kali Linux)**:
1. Open terminal, launch Yersinia GUI:
```plaintext
root@kali:~# yersinia -G
```
2. In Yersinia GUI:
   - Click “Launch attack.”
   - Select “DHCP” → “sending DISCOVER packet” (marked as DoS).
   - Click OK to start attack.
3. Observe:
   - Right pane: DHCP Discover messages sent with spoofed MAC addresses.
   - Left pane: DHCP packet count increases rapidly.

**Observations**:
- **Attack Mechanism**:
  - Yersinia sends continuous DHCP Discover messages, each with a unique spoofed MAC address.
  - R1 responds with DHCP Offer, reserving an IP (192.168.1.2–254) for each.
  - Entire pool (253 IPs) is reserved, causing DoS.
- **Impact**: New clients (e.g., PC1) cannot obtain IPs.
- **Kali Linux**: Cybersecurity tool with pre-installed penetration testing software (e.g., Yersinia for DHCP, ARP attacks).
- **Exam Note**: DHCP exhaustion is a spoofing/DoS attack (topic 5.1), exploiting DHCP’s DORA process.

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: DHCP exhaustion targets Availability (CIA), uses spoofed MACs (spoofing attack).
- **Day 13 (DHCP)**: Attack abuses DORA (Discover → Offer reserves IP).
- **Syslog Lab**: Attack may trigger Syslog (e.g., `%DHCPD-4-POOL_EXHAUSTED`).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Spoofed MACs similar to ARP spoofing.
- **Day 15 (TCP/UDP)**: DHCP uses UDP 67/68, vulnerable to spoofing.
- **Day 10 (IP Addressing)**: Exhausts 192.168.1.2–254, forces link-local (169.254.x.x).

**Analogy**: Attacker is like a scammer reserving all theater tickets with fake IDs, leaving none for real guests (PC1).

**Lifelong Retention Tip**: Simulate DHCP attack in EVE-NG (or Packet Tracer, conceptual). Flashcard: “DHCP exhaustion? Spoofed MACs, DoS. Yersinia? Sends Discover packets.” Practice 3 times.

### Step 3: Verify Attack Impact on R1
**Objective**: Confirm R1’s DHCP pool is exhausted and bindings are full.

**Steps (on R1)**:
1. Check DHCP pool:
```plaintext
R1# show ip dhcp pool
Pool POOL1 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 254
 Leased addresses               : 253
 Excluded addresses             : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.1          192.168.1.1 - 192.168.1.254         253
```
2. Check DHCP bindings:
```plaintext
R1# show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.2         00:11:22:33:44:55       [Expiration]            Automatic
192.168.1.3         00:11:22:33:44:56       [Expiration]            Automatic
...
192.168.1.254       00:11:22:33:45:aa       [Expiration]            Automatic
```

**Observations**:
- **Pool Status**: 253/253 IPs leased (192.168.1.2–254), 192.168.1.1 reserved.
- **Bindings**: Each IP tied to a unique spoofed MAC (e.g., 00:11:22:33:44:55).
- **Impact**: No IPs available for legitimate clients (DoS).
- **Exam Note**: Attack exploits DHCP’s trust in Discover messages, affects Availability (topic 5.1).

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: Confirms DHCP exhaustion as DoS/spoofing attack.
- **Day 13 (DHCP)**: `show ip dhcp binding` shows leased IPs; attack fills table.
- **Syslog Lab**: Pool exhaustion may trigger Syslog (e.g., `%DHCPD-4-POOL_EXHAUSTED`).

**Comparison to Other CCNA Topics**:
- **Day 18 (ARP)**: Spoofed MACs mimic ARP spoofing tactics.
- **Day 10 (IP Addressing)**: Depletes 192.168.1.2–254.
- **Day 15 (TCP/UDP)**: Attack uses DHCP’s UDP 67/68.

**Analogy**: R1’s binding table is like a full reservation list with fake names, blocking real guests (PC1).

**Lifelong Retention Tip**: Check `show ip dhcp pool` in Packet Tracer after simulated attack. Flashcard: “DHCP bindings? IP-MAC pairs. Exhaustion? 253 IPs leased.” Practice 3 times.

### Step 4: Test PC1’s DHCP Failure
**Objective**: Connect PC1 to SW1, confirm it cannot obtain an IP due to the attack.

**Steps (on PC1)**:
1. Verify PC1’s initial state (disconnected):
```plaintext
C:\> ipconfig /all
Ethernet adapter Ethernet0:
   Media State . . . . . . . . . . . : Media disconnected
   DHCP Enabled. . . . . . . . . . . : Yes
```
2. Connect PC1 to SW1 (G0/1) in EVE-NG (or Packet Tracer).
3. Check IP configuration:
```plaintext
C:\> ipconfig /all
Ethernet adapter Ethernet0:
   Connection-specific DNS Suffix  . :
   IPv6 Address. . . . . . . . . . . : fe80::260:97ff:fe01:1234%2
   IPv4 Address. . . . . . . . . . . : 169.254.123.45
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
```
4. Test connectivity:
```plaintext
C:\> ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```
5. Attempt DHCP renewal:
```plaintext
C:\> ipconfig /renew
[Times out, no IP assigned]
```

**Observations**:
- **IP Assignment**: PC1 autoconfigures 169.254.x.x (IPv4 link-local, /16) due to DHCP failure.
- **Connectivity**: Cannot ping R1 (192.168.1.1) or access Internet (e.g., google.com).
- **Impact**: DHCP exhaustion prevents PC1 from joining the network (DoS, Availability).
- **Exam Note**: Link-local 169.254.x.x indicates DHCP failure (topic 5.1, 2.2).

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: DHCP exhaustion blocks Availability (CIA).
- **Day 13 (DHCP)**: PC1’s 169.254.x.x confirms DORA failure.
- **Syslog Lab**: DHCP failure may trigger Syslog (e.g., `%DHCPD-4-NO_ADDRESS`).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: 169.254.x.x (APIPA) used when DHCP fails.
- **Day 15 (TCP/UDP)**: DHCP Discover (UDP 68) sent, no Offer received.
- **Day 18 (ARP)**: No ARP resolution without valid IP.

**Analogy**: PC1 is like a guest denied a ticket (IP) because the booth (R1) is out, stuck outside (169.254.x.x).

**Lifelong Retention Tip**: Simulate DHCP failure in Packet Tracer, observe 169.254.x.x. Flashcard: “169.254? Link-local, DHCP fail. DoS? Blocks Availability.” Practice 3 times.

### Step 5: Recover Network
**Objective**: Stop the attack, clear DHCP bindings, and restore PC1’s connectivity.

**Steps**:
1. **On Attacker**:
   - Stop Yersinia (close GUI or stop attack).
   - Disconnect Attacker from SW1 (delete G0/2 link in EVE-NG).
2. **On R1**:
   - Clear DHCP bindings:
```plaintext
R1# clear ip dhcp binding *
```
3. **On PC1**:
   - Renew IP:
```plaintext
C:\> ipconfig /renew
Ethernet adapter Ethernet0:
   IPv4 Address. . . . . . . . . . . : 192.168.1.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
```
   - Test connectivity:
```plaintext
C:\> ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:
Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Reply from 192.168.1.1: bytes=32 time=1ms TTL=255
Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
   - Access Internet:
     - Open Firefox, navigate to google.com (works).

**Observations**:
- **Recovery**:
  - Stopping attack frees R1’s DHCP pool.
  - `clear ip dhcp binding *` removes spoofed bindings.
  - PC1 gets 192.168.1.2, connects to R1, accesses Internet.
- **Exam Note**: `clear ip dhcp binding *` resets DHCP server (topic 2.2). Attack highlights DoS impact (topic 5.1).
- **Mitigation**: Next lecture covers port security to prevent spoofed MACs (e.g., limit SW1’s G0/2 MAC addresses).

**Comparison to Prior Content**:
- **Security Fundamentals Lecture**: Recovery restores Availability (CIA).
- **Day 13 (DHCP)**: `clear ip dhcp binding *` from DHCP lecture.
- **Syslog Lab**: Clearing bindings may trigger Syslog (e.g., `%DHCPD-5-BINDING_CLEARED`).

**Comparison to Other CCNA Topics**:
- **Day 10 (IP Addressing)**: PC1 gets valid IP (192.168.1.2) post-recovery.
- **Day 15 (TCP/UDP)**: DHCP DORA succeeds post-recovery (UDP 67/68).
- **Day 18 (ARP)**: Valid IP enables ARP for R1’s MAC.

**Analogy**: Stopping the scammer (Attacker) and clearing fake reservations (bindings) lets the guest (PC1) get a ticket (IP) and enter the theater (network).

**Lifelong Retention Tip**: Simulate recovery in Packet Tracer, use `clear ip dhcp binding *`. Flashcard: “Clear bindings? `clear ip dhcp binding *`. Recovery? Restores Availability.” Practice 3 times.

## Common Issues and Troubleshooting

- **DHCP Pool Exhaustion**:
  - **Issue**: PC1 gets 169.254.x.x, cannot connect.
  - **Cause**: Attacker’s spoofed Discover messages fill R1’s pool.
  - **Fix**: Stop attack, `clear ip dhcp binding *`.
- **Persistent Attack**:
  - **Issue**: Pool refills after clearing bindings.
  - **Cause**: Yersinia still running on Attacker.
  - **Fix**: Stop Yersinia, disconnect Attacker (G0/2).
- **PC1 Connectivity Failure**:
  - **Issue**: PC1 pings fail post-recovery.
  - **Cause**: Incorrect default gateway or DNS (not set to 192.168.1.1, 8.8.8.8).
  - **Fix**: Verify PC1’s DHCP settings (`ipconfig /all`).
- **Simulation Error**:
  - **Issue**: EVE-NG or Packet Tracer freezes during attack.
  - **Cause**: High packet rate from Yersinia.
  - **Fix**: Stop attack, restart simulation.

**Analogy**: Troubleshooting is like fixing a sold-out theater: clear fake reservations (bindings), stop the scammer (Attacker), and ensure guests (PC1) get tickets (IPs).

**Lifelong Retention Tip**: Simulate DHCP failure, recover with `clear ip dhcp binding *`. Checklist: “No IP? Check pool, bindings, attack status.” Practice 3 times.

## Additional CCNA-Relevant Information

**Why This Matters for CCNA**:
- **Exam Topic 5.1**: DHCP exhaustion is a DoS/spoofing attack, targeting Availability (CIA triad). Understanding attack mechanisms prepares you for questions on security concepts.
- **Exam Topic 2.2**: DHCP configuration (`show ip dhcp pool`, `clear ip dhcp binding *`) reinforces DHCP server operations.
- **Future Mitigation**: Port security (next lecture) limits MAC addresses per switch port, preventing spoofed DHCP Discover messages (topic 5.2: "Configure and verify switch security features").

**Key CCNA Concepts Reinforced**:
- **DHCP Process (DORA)**:
  - Discover (client broadcasts), Offer (server reserves IP), Request (client accepts), Ack (server confirms).
  - Attack exploits Offer phase, reserving IPs without Request/Ack.
- **CIA Triad**: DHCP exhaustion blocks Availability (no IPs for legitimate clients).
- **Spoofing**: Attacker’s fake MACs mimic legitimate clients, a common attack vector (e.g., ARP spoofing).
- **Link-Local Addresses**: 169.254.x.x (APIPA) used when DHCP fails, critical for troubleshooting (topic 2.2).

**Mitigation Techniques (Preview for Next Lecture)**:
- **Port Security** (topic 5.2):
  - Limit MAC addresses on SW1’s G0/2 (e.g., `switchport port-security maximum 1`).
  - Restrict to known MACs (e.g., `switchport port-security mac-address <PC1-MAC>`).
  - Violation actions: shutdown, restrict, protect.
- **DHCP Snooping**:
  - Trusts only specific ports (e.g., G0/0 to R1).
  - Filters spoofed DHCP messages from G0/2 (Attacker).
- **Rate Limiting**: Limits DHCP Discover rate on SW1 to prevent flooding.

**Practice Commands (Conceptual, Not in Demo)**:
- Verify DHCP status:
```plaintext
R1# show ip dhcp pool
R1# show ip dhcp binding
R1# clear ip dhcp binding *
```
- Check PC1’s IP:
```plaintext
C:\> ipconfig /all
C:\> ipconfig /renew
C:\> ping 192.168.1.1
```
- Preview port security (next lecture):
```plaintext
SW1# configure terminal
SW1(config)# interface g0/2
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security violation shutdown
```

**Simulation in Packet Tracer** (if EVE-NG unavailable):
- Packet Tracer lacks Kali/Yersinia, but you can:
  1. Configure R1 as DHCP server (192.168.1.0/24).
  2. Connect multiple PCs (simulate Attacker) to request IPs until pool exhausts.
  3. Connect PC1, observe 169.254.x.x.
  4. Clear bindings (`clear ip dhcp binding *`), verify PC1 gets IP.

**Additional Resources for CCNA**:
- **Cloudflare Articles**: Search “DHCP exhaustion attack” or “DNS amplification attack” for real-world examples.
- **Cisco Documentation**: Review DHCP (topic 2.2) and switch security (topic 5.2) in Cisco’s CCNA study guide.
- **Boson ExSim**: Practice questions on DoS, spoofing, DHCP, and port security.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - DHCP Exhaustion: “Discover, Offer, Reserve, Deny” (DORD).
   - CIA Impact: “DHCP attack? Availability” (DA).
   - Recovery: “Stop, Clear, Renew” (SCR).
2. **Visualize Topology**:
   - Draw PC1 → SW1 → R1 with Attacker flooding DHCP. Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “DHCP exhaustion? DoS, spoofed MACs. 169.254? Link-local. `clear ip dhcp binding *`? Resets pool.”
4. **Practice in Packet Tracer**:
   - Simulate DHCP server, exhaust pool with multiple PCs, recover with `clear ip dhcp binding *`. Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: DHCP exhaustion (DoS, Availability) vs. ARP spoofing (MITM, Confidentiality/Integrity).
   - Quiz: “DHCP attack? Blocks Availability. 169.254? DHCP fail. Port security? Limits MACs.”
6. **Troubleshooting Practice**:
   - Simulate 169.254.x.x, fix with `clear ip dhcp binding *`. Checklist: “No IP? Check pool, bindings, attack.”
7. **Teach Someone**:
   - Explain DHCP exhaustion and recovery to a peer.
8. **Spaced Repetition**:
   - Use Anki for DHCP terms (DORA, 169.254.x.x), attack types, port security. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **DHCP Exhaustion Attack**:
   - Spoofed DHCP Discover messages with fake MACs exhaust R1’s pool (253 IPs).
   - Causes DoS, blocking Availability (CIA triad).
2. **Impact**:
   - PC1 gets 169.254.x.x (link-local), cannot connect (topic 2.2, 5.1).
3. **Verification**:
   - `show ip dhcp pool`: 253/253 IPs leased.
   - `show ip dhcp binding`: Full table with spoofed MACs.
   - PC1: `ipconfig /all` shows 169.254.x.x.
4. **Recovery**:
   - Stop attack, `clear ip dhcp binding *`, PC1 gets 192.168.1.x.
5. **CCNA Focus**:
   - Topic 5.1: DHCP exhaustion as DoS/spoofing attack.
   - Topic 2.2: DHCP operations, link-local addresses.
   - Topic 5.2 (future): Port security mitigates attack.
6. **Kali Linux**: Cybersecurity tool for penetration testing (e.g., Yersinia for DHCP attacks).

These notes provide a comprehensive foundation for the Kali Linux Demo Lab, reinforcing CCNA topics 5.1 and 2.2. Practice DHCP setup and recovery in Packet Tracer, use flashcards for attack terms, and apply analogies/retention tips to master concepts for life!