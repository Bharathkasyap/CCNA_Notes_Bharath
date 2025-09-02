# TCP/UDP Lab with Wireshark Demo Notes for CCNA Beginners

## Lab Overview

This lab demonstrates the use of Wireshark to capture and analyze TCP and UDP traffic, reinforcing concepts from the TCP/UDP lecture for CCNA exam topic 1.5 ("Compare TCP to UDP"). The demo captures real network traffic from a PC, focusing on TCP (HTTPS, port 443) and UDP (DNS, port 53) packets, highlighting TCP’s three-way handshake, connection termination, and UDP’s simplicity. It serves as an introduction to packet capture software, showing how to observe Layer 4 protocols in action, with practical tasks to build familiarity.

**Lab Objectives**:
- **Understand Wireshark**: Learn basic packet capture and analysis, distinguishing it from Packet Tracer’s simulation mode.
- **Analyze TCP Traffic**: Observe the TCP three-way handshake (SYN, SYN-ACK, ACK), connection termination (FIN, ACK, FIN, ACK), sequence/acknowledgment numbers, and window size for HTTPS (port 443).
- **Analyze UDP Traffic**: Examine a DNS query (port 53) to highlight UDP’s simpler header and lack of reliability features.
- **Homework**: Download Wireshark, capture PC traffic, and identify TCP three-way handshake and connection termination.
- **Note**: The lab uses real PC traffic (e.g., YouTube browsing), not a simulated topology. No specific configuration is required, only Wireshark installation.

**Network Context**:
- **Devices**: PC capturing traffic via its network interface (e.g., Wi-Fi/Ethernet).
- **Traffic**:
  - **TCP**: HTTPS (port 443) to a web server (e.g., YouTube).
  - **UDP**: DNS query (port 53) to a DNS server.
- **Key Observations**:
  - TCP: Source port (ephemeral, e.g., 62652), destination port 443, three-way handshake, sequence/ACK numbers, window size, SSL encapsulation, connection termination.
  - UDP: Source port (ephemeral), destination port 53, no handshake or ACKs, simple header with DNS query.

**Analogy**: Wireshark is like a security camera recording postal deliveries (packets). TCP is a courier confirming delivery (handshake, ACKs) for HTTPS packages, while UDP is a quick postcard (DNS query) with no confirmation. The lab is like reviewing footage to see how TCP and UDP operate.

**Lifelong Retention Tip**: Visualize a PC sending TCP (HTTPS, port 443) and UDP (DNS, port 53) packets. Label TCP’s handshake (SYN, SYN-ACK, ACK) and UDP’s direct send. Download Wireshark, capture HTTPS/DNS traffic, and filter by `tcp.port == 443` and `udp.port == 53`. Create flashcards for TCP (handshake, sequence, ACK) and UDP (no handshake, port 53). Practice 3 times.

## Lab Steps and Analysis

### Step 1: Introduction to Wireshark
**Objective**: Understand Wireshark as a packet capture tool and its advantages over Packet Tracer.

- **Wireshark Overview**:
  - **Purpose**: Captures and analyzes real network traffic (frames, packets, segments) from a PC’s network interface (e.g., Wi-Fi, Ethernet).
  - **Contrast with Packet Tracer**: Packet Tracer simulates networks, offering limited packet analysis. Wireshark captures actual traffic, showing detailed headers (Layers 2–4).
  - **Interface**:
    - **Packet List**: Displays captured packets (protocol, source/destination ports, info).
    - **Packet Details**: Shows header fields (e.g., TCP SYN, sequence number) when a packet is selected.
  - **Download**: Free from wireshark.org.
- **Lab Context**:
  - Captures PC traffic during YouTube browsing (HTTPS, TCP 443) and DNS queries (UDP 53).
  - Filters applied (e.g., `tcp.port == 443`) to focus on specific protocols.

**Comparison to TCP/UDP Lecture**:
- **Lecture**: Introduced TCP (reliable, handshake) and UDP (best-effort, no handshake) concepts, focusing on headers and port numbers.
- **Lab**: Applies these concepts by showing real TCP (HTTPS) and UDP (DNS) packets in Wireshark, visualizing headers and handshakes.
- **Similarity**: Both emphasize TCP’s reliability (handshake, ACKs) vs. UDP’s simplicity (no ACKs).

**Comparison to Other Protocols**:
- **FHRP (HSRP)**: No packet capture; HSRP uses Layer 3 (virtual IP/MAC) vs. Wireshark’s Layer 4 focus (TCP/UDP ports).
- **OSPF**: Layer 3 protocol (IP protocol 89), no TCP/UDP ports, but Wireshark can capture OSPF packets (e.g., Hellos).
- **STP**: Layer 2, no ports; Wireshark captures BPDUs, not TCP/UDP.
- **RIP**: Uses UDP 520, observable in Wireshark, unlike TCP’s reliability.

**Analogy**: Wireshark is like a network detective, inspecting TCP’s detailed delivery logs (handshake, ACKs) and UDP’s simple notes (no confirmation), unlike Packet Tracer’s staged rehearsal.

**Lifelong Retention Tip**: Install Wireshark, start a capture on your PC’s interface, and browse a website. Stop capture, explore packet list, and click a packet to see details. Memorize: “Wireshark = real traffic, Packet Tracer = simulated.” Create flashcard: “Wireshark purpose? Capture/analyze real packets.” Practice 3 times.

### Step 2: Analyze TCP Traffic (HTTPS, Port 443)
**Objective**: Capture and analyze TCP packets for HTTPS (port 443), observing the three-way handshake, sequence/ACK numbers, window size, and connection termination.

**Wireshark Capture**:
- **Filter**: `tcp.port == 443` (isolates HTTPS traffic).
- **Observations**:
  - **Protocol**: TCP (displayed as SSL for HTTPS due to encryption).
  - **Ports**:
    - Source: Ephemeral (e.g., 62652, in 49152–65535 range).
    - Destination: 443 (HTTPS, well-known port).
    - Reply: Source 443, destination 62652.
  - **Three-Way Handshake**:
    1. PC → Server: SYN (relative sequence 0, actual ~1.2 billion, SYN flag = 1).
    2. Server → PC: SYN-ACK (relative sequence 0, ACK 1, forward acknowledgment).
    3. PC → Server: ACK (relative sequence 1, ACK 1).
  - **Data Exchange**: SSL (HTTPS) data over TCP, with sequence/ACK numbers (relative for simplicity, e.g., 1, 2, 3).
  - **Connection Termination**:
    - FIN, ACK, FIN, ACK sequence (some nuances, e.g., extra ACKs, not CCNA-level).
    - Basic sequence: PC (FIN), Server (ACK), Server (FIN), PC (ACK).
  - **Key Fields**:
    - **Sequence Number**: Relative (0, 1, 2) for readability; actual (e.g., 1,224,315,781).
    - **Acknowledgment Number**: Forward ACK (e.g., receive 0, ACK 1).
    - **Window Size**: Controls flow (e.g., bytes before ACK).
    - **Flags**: SYN = 1 for first packet, others 0.

**Packet Details (First SYN Packet)**:
- **Encapsulation**: TCP segment in IP packet, in Ethernet frame (Layers 4, 3, 2).
- **Fields**:
  - Source Port: 62652 (ephemeral).
  - Destination Port: 443 (HTTPS).
  - Sequence Number: Relative 0, actual ~1.2 billion.
  - Flags: SYN = 1, others = 0.
  - Window Size: Displayed (e.g., 64240 bytes).
- **Note**: Wireshark simplifies sequence numbers (relative vs. actual) for analysis.

**Comparison to TCP/UDP Lecture**:
- **Lecture**: Described TCP’s three-way handshake (SYN, SYN-ACK, ACK), sequence/ACK numbers, window size, and termination (FIN, ACK).
- **Lab**: Visualizes these in Wireshark for HTTPS, showing real sequence numbers, flags, and SSL encapsulation.
- **Similarity**: Both emphasize TCP’s reliability (handshake, ACKs) and port usage (443 for HTTPS).

**Comparison to Other Protocols**:
- **FHRP (HSRP)**: No TCP/UDP; Wireshark could capture HSRP Hellos (UDP 1985 for v2).
- **OSPF**: No TCP/UDP, but Wireshark captures OSPF packets (protocol 89).
- **STP**: Layer 2 BPDUs, not TCP/UDP, visible in Wireshark.
- **RIP**: UDP 520, observable in Wireshark, unlike TCP’s HTTPS reliability.

**Analogy**: TCP in Wireshark is like watching a courier (HTTPS) confirm delivery with a handshake (SYN, SYN-ACK, ACK), track packages (sequence/ACK), and set delivery limits (window size). The footage (Wireshark) shows every step.

**Lifelong Retention Tip**: In Wireshark, capture HTTPS traffic (filter `tcp.port == 443`), identify SYN, SYN-ACK, ACK, and FIN packets. Note ports (443, ephemeral) and relative sequence (0, 1). Memorize: “TCP = handshake, sequence, ACK, window.” Create flashcard: “TCP HTTPS port? 443.” Practice 3 times.

### Step 3: Analyze UDP Traffic (DNS, Port 53)
**Objective**: Capture and analyze a UDP packet for a DNS query (port 53), highlighting its simplicity compared to TCP.

**Wireshark Capture**:
- **Filter**: `udp.port == 53` (isolates DNS traffic).
- **Observations**:
  - **Protocol**: UDP, encapsulating a DNS query.
  - **Ports**:
    - Source: Ephemeral (e.g., 49153).
    - Destination: 53 (DNS, well-known port).
  - **Header**: Simple (source/destination ports, length, checksum).
  - **No Handshake**: Direct query, no SYN/ACK.
  - **No Sequence/ACK**: No reliability or ordering.
  - **Encapsulation**: DNS query in UDP segment, IP packet, Ethernet frame.
- **Note**: UDP’s minimal header (8 bytes) reduces overhead, ideal for fast queries like DNS.

**Comparison to TCP/UDP Lecture**:
- **Lecture**: Described UDP as connectionless, best-effort, with a simple header (ports, length, checksum), used for DNS (port 53).
- **Lab**: Shows a real DNS query in Wireshark, confirming no handshake or sequence numbers.
- **Similarity**: Both highlight UDP’s simplicity and speed vs. TCP’s complexity.

**Comparison to Other Protocols**:
- **FHRP (HSRP)**: No UDP for HSRP (uses UDP 1985 for v2 Hellos, observable in Wireshark).
- **OSPF**: No UDP; Wireshark captures OSPF directly.
- **STP**: No UDP; Wireshark captures BPDUs.
- **RIP**: UDP 520, similar to DNS’s UDP usage.

**Analogy**: UDP in Wireshark is like watching a newsboy (DNS) toss a postcard (query) with just an address (port 53) and no delivery confirmation, unlike TCP’s detailed courier process.

**Lifelong Retention Tip**: In Wireshark, capture DNS traffic (filter `udp.port == 53`), note source (ephemeral), destination (53), and no handshake. Memorize: “UDP = no handshake, simple header, DNS 53.” Create flashcard: “DNS port? UDP 53.” Practice 3 times.

### Step 4: Homework Tasks
**Objective**: Practice Wireshark to reinforce TCP/UDP concepts.

- **Tasks**:
  1. Download Wireshark from wireshark.org.
  2. Capture PC traffic while browsing websites.
  3. Stop capture, analyze packets:
     - Find a TCP three-way handshake (SYN, SYN-ACK, ACK, e.g., HTTPS port 443).
     - Find a TCP connection termination (FIN, ACK, FIN, ACK).
- **Tips**:
  - Filter: `tcp.port == 443` for HTTPS, `udp.port == 53` for DNS.
  - Check packet details for ports, flags, sequence/ACK numbers.
  - Reference David Bombal’s Wireshark courses (Udemy, YouTube) for advanced learning.

**Comparison to TCP/UDP Lecture**:
- **Lecture**: Provided theoretical TCP (handshake, reliability) and UDP (best-effort) concepts.
- **Lab**: Applies theory by analyzing real TCP/UDP packets, encouraging hands-on practice.
- **Similarity**: Both focus on TCP’s handshake and UDP’s simplicity, using port numbers.

**Comparison to Other Protocols**:
- **FHRP (HSRP)**: No packet capture practice; HSRP focuses on configuration, not analysis.
- **OSPF**: Wireshark could analyze OSPF Hellos, but no TCP/UDP focus.
- **STP**: Wireshark captures BPDUs, not relevant to TCP/UDP.
- **RIP**: UDP-based, analyzable in Wireshark, like DNS.

**Analogy**: Homework is like practicing as a network detective, spotting TCP’s handshake (courier confirmation) and UDP’s quick send (postcard) in Wireshark’s footage.

**Lifelong Retention Tip**: Complete homework: capture HTTPS (TCP 443) and DNS (UDP 53) traffic, identify handshake (SYN, SYN-ACK, ACK) and termination (FIN, ACK). Save captures, review weekly. Memorize: “Wireshark homework = find TCP handshake, termination.” Create flashcard: “TCP termination? FIN, ACK, FIN, ACK.” Practice 3 times.

## Key Takeaways

1. **Wireshark Basics**:
   - Captures real traffic (frames, packets, segments) vs. Packet Tracer’s simulation.
   - Displays packet list and details (protocol, ports, flags, sequence/ACK).
   - Filters (e.g., `tcp.port == 443`, `udp.port == 53`) isolate TCP/UDP traffic.
2. **TCP Analysis (HTTPS, Port 443)**:
   - Ports: Source (ephemeral, e.g., 62652), destination (443).
   - Three-way handshake: SYN (sequence 0), SYN-ACK (ACK 1), ACK.
   - Sequence/ACK: Relative (0, 1) for readability, actual (e.g., 1.2 billion).
   - Window Size: Controls flow.
   - Termination: FIN, ACK, FIN, ACK (some nuances not CCNA-level).
   - Encapsulation: TCP/SSL in IP packet, Ethernet frame.
3. **UDP Analysis (DNS, Port 53)**:
   - Ports: Source (ephemeral), destination (53).
   - Header: Simple (ports, length, checksum).
   - No handshake, sequence, or ACKs; carries DNS query.
4. **Comparison to Lecture**:
   - Lecture: TCP (reliable, handshake, sequence, window), UDP (best-effort, no handshake).
   - Lab: Visualizes these in Wireshark, showing real headers and processes.
5. **CCNA Focus**:
   - Compare TCP (reliable, HTTPS 443) vs. UDP (fast, DNS 53).
   - Recognize port numbers (443, 53) and TCP handshake/termination.
   - Understand Wireshark’s role in analyzing Layer 4 traffic.

## Common Issues and Troubleshooting

- **No TCP Handshake Found**:
  - **Issue**: Wrong filter or no HTTPS traffic.
  - **Fix**: Use `tcp.port == 443`, browse a secure site (e.g., YouTube).
  - **Verify**: Check for SYN, SYN-ACK, ACK sequence.
- **No UDP Traffic**:
  - **Issue**: No DNS queries captured.
  - **Fix**: Use `udp.port == 53`, visit a new website to trigger DNS.
  - **Verify**: Check for UDP packet with destination port 53.
- **Overwhelming Traffic**:
  - **Issue**: Too many packets in capture.
  - **Fix**: Apply filters (e.g., `tcp.port == 443` or `udp.port == 53`) to focus on TCP/UDP.
  - **Verify**: Packet list shows only HTTPS or DNS traffic.
- **Missing Flags or Sequence Numbers**:
  - **Issue**: Incorrect packet selected.
  - **Fix**: Select TCP packets, check details for SYN/ACK/FIN flags and sequence/ACK fields.
  - **Verify**: Confirm handshake (SYN, SYN-ACK, ACK) or termination (FIN, ACK).

**Analogy**: Troubleshooting Wireshark is like sorting through security footage to find specific deliveries (TCP handshake, UDP query). Filters (e.g., `tcp.port == 443`) narrow the focus to the right packages.

**Lifelong Retention Tip**: In Wireshark, capture traffic, apply wrong filters (e.g., `tcp.port == 80`), then fix with `tcp.port == 443`. Check handshake and DNS packets. Create checklist: “No handshake? Check filter, port 443.” Practice 3 times.

## Practice Commands (Wireshark Filters)

- **TCP (HTTPS)**:
  ```plaintext
  tcp.port == 443
  ```
- **UDP (DNS)**:
  ```plaintext
  udp.port == 53
  ```
- **Display All TCP Traffic**:
  ```plaintext
  tcp
  ```
- **Display All UDP Traffic**:
  ```plaintext
  udp
  ```

## Study Tips for Lifelong Retention

1. **Mnemonics for Protocols**:
   - TCP: “Handshake, Sequence, ACK, Window, HTTPS 443.”
   - UDP: “No Handshake, Simple, DNS 53.”
   - Filters: “TCP 443, UDP 53.”
2. **Visualize the Traffic**:
   - Draw a PC sending TCP (HTTPS, port 443, SYN-ACK) and UDP (DNS, port 53, no handshake). Label sequence (0, 1), flags (SYN, ACK), and ports. Pin to study space.
3. **Memorize Key Values**:
   - TCP: Handshake (SYN, SYN-ACK, ACK), termination (FIN, ACK), port 443.
   - UDP: No handshake, port 53.
   - Flashcards: “HTTPS port? TCP 443. DNS port? UDP 53. TCP handshake? SYN, SYN-ACK, ACK.”
4. **Practice in Wireshark**:
   - Capture HTTPS (TCP 443) and DNS (UDP 53) traffic. Filter with `tcp.port == 443` and `udp.port == 53`. Identify handshake, termination, and DNS query. Save captures, review 3–5 times.
5. **Compare to Lecture**:
   - Create a table: Lecture (TCP reliable, UDP best-effort) vs. Lab (Wireshark TCP handshake, UDP DNS). Quiz yourself: “TCP in Wireshark? Handshake, port 443.”
6. **Troubleshooting Practice**:
   - Simulate issues (e.g., wrong filter, no HTTPS traffic), fix with correct filters (`tcp.port == 443`). Create checklist: “No DNS? Filter udp.port == 53, trigger query.”
7. **Teach Someone**:
   - Explain TCP handshake (SYN, SYN-ACK, ACK) and UDP’s simplicity (DNS query) using Wireshark captures. Teaching reinforces memory.
8. **Spaced Repetition**:
   - Use Anki to review ports (443, 53), TCP handshake, and UDP features after 1 day, 1 week, 1 month.

These notes provide a complete foundation for the TCP/UDP lab with Wireshark, preparing you for CCNA exam topic 1.5 and reinforcing lecture concepts. Practice capturing and analyzing packets, use the analogies, and apply the retention tips to master TCP and UDP for life!