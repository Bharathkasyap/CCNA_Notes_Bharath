# TFTP Lecture Notes for CCNA Beginners

## Lecture Overview

This lecture covers **File Transfer Protocol (FTP)** and **Trivial File Transfer Protocol (TFTP)**, focusing on their capabilities and functions in network file transfers, as outlined in **CCNA exam topic 4.9** ("Describe the capabilities and function of TFTP/FTP in the network"). It introduces FTP and TFTP, compares their features, explains Cisco IOS file systems, and demonstrates their use for transferring IOS images to a router (R1). FTP uses TCP ports 20 (data) and 21 (control), supports authentication, and offers advanced functions like directory navigation. TFTP, a simpler protocol, uses UDP port 69, lacks authentication and encryption, and is limited to basic file transfers. The lecture includes a practical example of copying an IOS image to R1’s flash memory, updating the boot system, and deleting the old image. A bonus practice question from Boson Software’s ExSim reinforces key concepts.

**Lecture Objectives**:
- **Understand FTP/TFTP Purpose**: Describe their role in transferring files over a network.
- **Compare FTP and TFTP**: Highlight differences in ports, security, and functionality.
- **Explain Cisco IOS File Systems**: Introduce flash, NVRAM, and network file systems.
- **Demonstrate File Transfers**: Show how to use FTP/TFTP to copy IOS images to a Cisco device.
- **Prepare for Exam**: Master capabilities and functions of FTP/TFTP (topic 4.9).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
| Cisco Server   |-------|      SRV1      |-------|      R1        |
| software.cisco.com |   | FTP/TFTP Server|       | Flash: IOS     |
| (IOS Source)   |       | 192.168.1.100  |       | 192.168.1.1    |
+----------------+       +----------------+       +----------------+
```
- **Devices**:
  - **Cisco Server**: Hosts IOS images (software.cisco.com).
  - **SRV1**: FTP/TFTP server (192.168.1.100), stores IOS image for transfer.
  - **R1**: Router (192.168.1.1), receives IOS image into flash memory.
- **Note**: Connectivity via OSPF/static routes (assumed from Day 34). FTP uses TCP 20/21; TFTP uses UDP 69.
- **Requirements**:
  1. Download IOS image from Cisco to SRV1.
  2. Use FTP/TFTP on R1 to copy IOS image to flash (`copy tftp: flash:` or `copy ftp: flash:`).
  3. Configure R1 to boot new IOS (`boot system`), save config, reload.
  4. Delete old IOS image (`delete flash:/old-image.bin`).
  5. Verify with `show version`, `show flash`.

**Analogy**: FTP/TFTP are like delivery services for files. FTP is a full-service courier (TCP 20/21, secure with authentication) handling complex tasks (directory navigation), while TFTP is a basic drop-off service (UDP 69, no security) for quick, simple deliveries (file copy only).

**Lifelong Retention Tip**: Visualize R1 downloading an IOS image from SRV1 (192.168.1.100) using TFTP (UDP 69) or FTP (TCP 21). In Packet Tracer, run `copy tftp: flash:`, enter SRV1’s IP and filename, verify with `show flash`. Create flashcards: “TFTP port? UDP 69. FTP ports? TCP 20/21. TFTP security? None.” Practice 3 times daily for a week.

## FTP and TFTP Purpose

**Purpose**: Transfer files over a network, primarily for upgrading Cisco IOS images or backing up configurations.

**Key Points**:
- **Client-Server Model**: Client (e.g., R1) requests files from/to a server (e.g., SRV1).
- **Primary Use**: Copy IOS images to device flash for upgrades or save configs to a server.
- **Other Uses**: Transfer any file type (e.g., configs, logs) between devices.
- **Example Workflow**:
  - Download IOS image from software.cisco.com to SRV1.
  - R1 uses FTP/TFTP to copy image to flash.
  - Configure R1 to boot new IOS, reload, delete old image.

**Analogy**: FTP/TFTP are like shipping files to a device’s storage (flash) or retrieving them for safekeeping (server), with IOS upgrades being the main delivery.

**Lifelong Retention Tip**: Flashcard: “FTP/TFTP purpose? Transfer files, e.g., IOS upgrades.” Practice transferring a file in Packet Tracer (`copy tftp: flash:`) 3 times.

## TFTP Overview

**Purpose**: Simple, lightweight protocol for basic file transfers in controlled environments.

**Key Points**:
- **Standardized**: 1981, post-FTP, named “Trivial” for simplicity.
- **Port**: UDP 69 (client to server initial request).
- **Features**:
  - Only supports file copy (to/from server).
  - No authentication (no username/password).
  - No encryption (plain-text data).
- **Reliability**: Uses “lock-step” communication (data-ACK pairs, retransmissions).
- **Connection Phases**:
  1. **Connection**: Client sends read/write request; server responds.
  2. **Data Transfer**: Data and ACK messages exchanged.
  3. **Termination**: Final ACK ends transfer.
- **Port Behavior**:
  - Client uses ephemeral port (TID) to server’s UDP 69.
  - Server responds with random TID (not 69); subsequent messages use TIDs.
- **Use Case**: Quick IOS image transfers in secure LANs (e.g., R1 to SRV1).
- **Limitations**: No directory navigation, file listing, or security; best for small files.

**Demonstration**:
- Client (PC) sends read request to server (UDP 69).
- Server sends data packet (random TID); client ACKs.
- If ACK lost, client retransmits; continues until file transfer completes.

**Observations**:
- **UDP-Based**: Connectionless but TFTP adds reliability via ACKs.
- **Security**: None; avoid over public networks.
- **Exam Note**: Know TFTP’s simplicity, UDP 69, no authentication (topic 4.9).

**Comparison to Prior Content**:
- **Syslog Lab**: TFTP (UDP 69) vs. Syslog (UDP 514); both lack encryption.
- **SSH Lecture**: TFTP insecure vs. SSH encrypted (TCP 22).
- **SNMP Lab**: TFTP transfers IOS; SNMP monitors device status.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: TFTP UDP 69 vs. FTP TCP 20/21, SSH TCP 22.
- **Security (Section 5.0)**: TFTP’s lack of security contrasts with SSH/FTPS.
- **Extended ACLs (Day 35)**: ACLs can block TFTP (e.g., `deny udp any host 192.168.1.100 eq 69`).

**Analogy**: TFTP is like a basic mail carrier (UDP 69) dropping off a package (file) without checking IDs or locking the box, using a simple handshake (ACKs) to ensure delivery.

**Lifelong Retention Tip**: In Packet Tracer, configure SRV1 as TFTP server, copy file to R1 (`copy tftp: flash:`), verify with `show flash`. Flashcard: “TFTP port? UDP 69. Security? None.” Practice 3 times.

## FTP Overview

**Purpose**: Complex protocol for file transfers with authentication and advanced functionality.

**Key Points**:
- **Standardized**: 1971, pre-TCP/IP, updated for TCP.
- **Ports**: TCP 21 (control connection), TCP 20 (data connection).
- **Features**:
  - Authentication (username/password).
  - No encryption (plain-text); FTPS/SFTP for security.
  - Commands for file copy, directory navigation, listing, creation/deletion.
- **Connections**:
  - **Control Connection**: TCP 21, sends commands/replies (e.g., “get file”).
  - **Data Connection**: TCP 20, transfers files.
- **Modes**:
  - **Active Mode**: Server initiates data connection (TCP 20).
  - **Passive Mode**: Client initiates data connection (firewall-friendly).
- **Use Case**: Secure file transfers (with FTPS) or complex tasks (e.g., IOS upgrades, config backups).

**Demonstration**:
- Client initiates control connection (TCP 21, SYN/SYN-ACK/ACK).
- Client sends command (e.g., “retrieve file”); server ACKs.
- **Active Mode**: Server opens data connection (TCP 20) to client.
- **Passive Mode**: Client opens data connection (firewall permits).

**Observations**:
- **TCP-Based**: Reliable, connection-oriented.
- **Security**: Plain-text; use FTPS (FTP over SSL/TLS) or SFTP for encryption.
- **Exam Note**: Know FTP’s TCP 20/21, authentication, passive mode (topic 4.9).

**Comparison to Prior Content**:
- **Syslog Lab**: FTP (TCP 21) vs. Syslog (UDP 514); FTP authenticates, Syslog doesn’t.
- **SSH Lecture**: FTP plain-text vs. SSH encrypted; SFTP uses SSH (TCP 22).
- **SNMP Lab**: FTP for IOS transfer; SNMP for device monitoring.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: FTP TCP 20/21 vs. TFTP UDP 69.
- **Security (Section 5.0)**: FTP insecure unless FTPS/SFTP used.
- **Extended ACLs (Day 35)**: ACLs block FTP (e.g., `deny tcp any host 192.168.1.100 eq 21`).

**Analogy**: FTP is like a full-service courier (TCP 20/21) requiring ID (username/password) and handling complex tasks (directory navigation), but without locks (no encryption) unless upgraded (FTPS).

**Lifelong Retention Tip**: Flashcard: “FTP ports? TCP 20 data, 21 control. Passive mode? Client initiates.” Practice FTP transfer in Packet Tracer (`copy ftp: flash:`) 3 times.

## FTP vs. TFTP

**Purpose**: Compare FTP and TFTP to understand their differences in functionality and use cases.

**Comparison**:
| Feature            | FTP                                  | TFTP                                |
|--------------------|--------------------------------------|-------------------------------------|
| **Port**           | TCP 20 (data), 21 (control)         | UDP 69                              |
| **Transport**      | TCP (reliable, connection-oriented) | UDP (connectionless, ACK-based)     |
| **Authentication** | Username/password                    | None                                |
| **Encryption**     | None (FTPS for encryption)          | None                                |
| **Functions**      | File copy, directory navigation, list, create/delete | File copy only               |
| **Complexity**     | Complex, feature-rich                | Simple, lightweight                 |
| **Use Case**       | Secure transfers (with FTPS), complex tasks | Quick, small file transfers in LAN |

**Observations**:
- **FTP**: More secure with authentication; supports FTPS/SFTP for encryption.
- **TFTP**: Faster but insecure; limited to file copy.
- **Exam Note**: Know FTP’s authentication, TCP 20/21, vs. TFTP’s simplicity, UDP 69 (topic 4.9).

**Comparison to Prior Content**:
- **Syslog Lab**: FTP/TFTP transfer configs; Syslog logs transfers (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: FTP/TFTP plain-text vs. SSH encrypted; SFTP uses SSH.
- **SNMP Lab**: FTP/TFTP for IOS upgrades; SNMP for monitoring.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: FTP TCP 20/21, TFTP UDP 69 vs. SSH TCP 22.
- **Security (Section 5.0)**: FTP/TFTP insecure vs. SSH/FTPS secure.
- **Extended ACLs (Day 35)**: ACLs block FTP (TCP 20/21) or TFTP (UDP 69).

**Analogy**: FTP is a full-featured delivery truck (TCP 20/21) with a driver ID (authentication), while TFTP is a bicycle courier (UDP 69) for quick, unsecured drops.

**Lifelong Retention Tip**: Create table comparing FTP (TCP 20/21, authentication) vs. TFTP (UDP 69, none). Flashcard: “FTP vs. TFTP? FTP complex, secure; TFTP simple, insecure.” Practice 3 times.

## Cisco IOS File Systems

**Purpose**: Understand storage locations for files (e.g., IOS, configs) on Cisco devices.

**Key Points**:
- **File System**: Manages data storage/retrieval (e.g., flash, NVRAM).
- **Types**:
  - **Flash (Disk)**: Stores IOS image; copied to RAM on boot.
  - **NVRAM**: Stores startup-config; retains data without power.
  - **Network**: External systems (e.g., FTP/TFTP servers).
  - **Opaque**: Internal logical systems (not physical storage).
- **Command**: `show file systems` lists available file systems.
- **Use Case**: Copy IOS to flash; save configs to NVRAM or server.

**Sample Output**:
```plaintext
R1# show file systems
File Systems:
  Size(b)       Free(b)      Type  Flags  Prefixes
*  256487424    200000000    disk  rw     flash:
     2097152      1500000   nvram  rw     nvram:
         -            -  network  rw     tftp:
         -            -  network  rw     ftp:
```
- **Flash**: Stores `c2900-universalk9-mz.SPA.151-4.M4.bin`.
- **NVRAM**: Stores `startup-config`.

**Observations**:
- **Flash**: Primary storage for IOS; critical for upgrades.
- **NVRAM**: Retains startup-config; `copy running-config startup-config` saves here.
- **Exam Note**: NVRAM for startup-config; flash for IOS (topic 4.9, indirectly relevant).

**Comparison to Prior Content**:
- **Syslog Lab**: Syslog configs saved to NVRAM; logs to flash or server.
- **SSH Lecture**: SSH configs stored in NVRAM (`startup-config`).
- **SNMP Lab**: SNMP configs in NVRAM; IOS in flash.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: Network file systems use FTP (TCP 20/21), TFTP (UDP 69).
- **Security (Section 5.0)**: Configs in NVRAM secured via SSH access.
- **Extended ACLs (Day 35)**: ACLs protect network file systems (e.g., `deny udp any host 192.168.1.100 eq 69`).

**Analogy**: Cisco file systems are like a device’s filing cabinets: flash for software (IOS), NVRAM for permanent records (startup-config), and network for external storage (FTP/TFTP servers).

**Lifelong Retention Tip**: Run `show file systems` in Packet Tracer, note flash (IOS), NVRAM (startup-config). Flashcard: “IOS storage? Flash. Startup-config? NVRAM.” Practice 3 times.

## Using FTP/TFTP in Cisco IOS

**Purpose**: Transfer IOS images to R1’s flash using FTP/TFTP, configure boot, and delete old images.

**Steps (on R1)**:
1. **Check Current IOS**:
```plaintext
R1# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.1(4)M4
```
2. **Check Flash**:
```plaintext
R1# show flash
-#- --length-- -----date/time------ path
1   123456789  Jan 1 2025 12:00:00 c2900-universalk9-mz.SPA.151-4.M4.bin
```
3. **Copy IOS via TFTP**:
```plaintext
R1# copy tftp: flash:
Address or name of remote host []? 192.168.1.100
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
4. **Copy IOS via FTP**:
```plaintext
R1(config)# ip ftp username cisco
R1(config)# ip ftp password cisco
R1# copy ftp: flash:
Address or name of remote host []? 192.168.1.100
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
5. **Configure Boot**:
```plaintext
R1(config)# boot system flash:/c2900-universalk9-mz.SPA.155-1.T.bin
R1(config)# exit
R1# write memory
```
6. **Reload**:
```plaintext
R1# reload
```
7. **Verify New IOS**:
```plaintext
R1# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.5(1)T
```
8. **Delete Old IOS**:
```plaintext
R1# delete flash:/c2900-universalk9-mz.SPA.151-4.M4.bin
Delete filename [c2900-universalk9-mz.SPA.151-4.M4.bin]? [Enter]
```

**Verification**:
- **Show Flash** (after copy):
```plaintext
R1# show flash
-#- --length-- -----date/time------ path
1   123456789  Jan 1 2025 12:00:00 c2900-universalk9-mz.SPA.151-4.M4.bin
2   123456790  Jan 1 2025 12:05:00 c2900-universalk9-mz.SPA.155-1.T.bin
```
- **Show Version** (after reload): Confirms 15.5(1)T.
- **Show Flash** (after delete): Old file gone.

**Observations**:
- **TFTP**: `copy tftp: flash:` requires server IP, filename; no authentication.
- **FTP**: `copy ftp: flash:` requires `ip ftp username/password` matching server.
- **Boot System**: `boot system flash:/filename` ensures new IOS loads; requires `write memory`.
- **Packet Tracer**: Simplifies server setup; real-world uses dedicated FTP/TFTP servers.
- **Exam Note**: Know `copy tftp: flash:`, `copy ftp: flash:`, `boot system` (topic 4.9).

**Comparison to Prior Content**:
- **Syslog Lab**: File transfers logged (e.g., `%SYS-5-CONFIG_I` for `boot system`).
- **SSH Lecture**: SSH (TCP 22) secures CLI for running `copy` commands vs. FTP/TFTP plain-text.
- **SNMP Lab**: SNMP monitors device post-upgrade; FTP/TFTP delivers IOS.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: FTP TCP 20/21, TFTP UDP 69.
- **Security (Section 5.0)**: FTP/TFTP insecure; use FTPS/SFTP for security.
- **Extended ACLs (Day 35)**: ACLs block transfers (e.g., `deny tcp any host 192.168.1.100 eq 21`).

**Analogy**: Using FTP/TFTP is like R1 ordering a new IOS package from SRV1’s warehouse (192.168.1.100), delivered via secure courier (FTP) or basic drop-off (TFTP), then updating its software (boot) and discarding old files.

**Lifelong Retention Tip**: In Packet Tracer, copy IOS via `copy tftp: flash:`, set `boot system`, reload, verify with `show version`. Flashcard: “TFTP copy? `copy tftp: flash:`. Boot? `boot system`.” Practice 3 times.

## Common Issues and Troubleshooting

- **TFTP Copy Fails**:
  - **Issue**: `copy tftp: flash:` fails.
  - **Cause**: Wrong server IP, filename, or UDP 69 blocked.
  - **Fix**: Verify IP (192.168.1.100), filename, check ACLs (`show access-lists`).
- **FTP Copy Fails**:
  - **Issue**: `copy ftp: flash:` fails.
  - **Cause**: Incorrect username/password or TCP 20/21 blocked.
  - **Fix**: Verify `ip ftp username cisco`, `ip ftp password cisco`, check ACLs.
- **Wrong IOS Boots**:
  - **Issue**: Old IOS (15.1) boots after reload.
  - **Cause**: Missing `boot system` or unsaved config.
  - **Fix**: Configure `boot system flash:/new-image.bin`, `write memory`, reload.
- **Flash Full**:
  - **Issue**: Copy fails due to insufficient space.
  - **Cause**: Old IOS not deleted.
  - **Fix**: Run `show flash`, delete old file (`delete flash:/old-image.bin`).
- **Firewall Blocks FTP**:
  - **Issue**: FTP fails from behind firewall.
  - **Cause**: Active mode blocked; server can’t initiate TCP 20.
  - **Fix**: Use passive mode on FTP client (`copy ftp: flash:` supports passive).

**Analogy**: Troubleshooting FTP/TFTP is like ensuring a delivery truck (FTP/TFTP) reaches R1’s warehouse (flash) with the correct address (IP), package name (filename), and clear roads (no ACL blocks).

**Lifelong Retention Tip**: Simulate TFTP failure (wrong IP), retry with correct IP (192.168.1.100), verify with `show flash`. Checklist: “No transfer? Check IP, filename, ports, ACLs.” Practice 3 times.

## Practice Commands

- **Check IOS and Flash**:
```plaintext
R1# show version
R1# show file systems
R1# show flash
```
- **TFTP Transfer**:
```plaintext
R1# copy tftp: flash:
Address or name of remote host []? 192.168.1.100
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
- **FTP Transfer**:
```plaintext
R1(config)# ip ftp username cisco
R1(config)# ip ftp password cisco
R1# copy ftp: flash:
Address or name of remote host []? 192.168.1.100
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
- **Configure Boot**:
```plaintext
R1(config)# boot system flash:/c2900-universalk9-mz.SPA.155-1.T.bin
R1(config)# exit
R1# write memory
R1# reload
```
- **Delete Old IOS**:
```plaintext
R1# delete flash:/c2900-universalk9-mz.SPA.151-4.M4.bin
```

## Quiz Answers and Explanations

1. **Which statements about FTP are true? (Select two)**  
   - **A**: Uses UDP  
   - **B**: Control connections use TCP port 21  
   - **C**: No authentication  
   - **D**: Data connections use TCP port 20  
   - **Answer**: B, D (FTP uses TCP 21 for control, 20 for data).  

2. **Which command transfers a file from TFTP to flash?**  
   - **A**: `copy tftp: flash:`  
   - **B**: `copy flash: tftp:`  
   - **C**: `copy tftp: running-config`  
   - **D**: `copy running-config tftp:`  
   - **Answer**: A (Source: TFTP, destination: flash).  

3. **R1 behind firewall connects to FTP server. Which is true?**  
   - **A**: Active mode for control connection  
   - **B**: Active mode for data connection  
   - **C**: Passive mode for data connection  
   - **D**: No firewall impact  
   - **Answer**: C (Passive mode for firewall compatibility).  

4. **Which file system stores startup-config?**  
   - **A**: Flash  
   - **B**: RAM  
   - **C**: ROM  
   - **D**: NVRAM  
   - **Answer**: D (NVRAM for startup-config).  

5. **Which functions are NOT possible with TFTP? (Select two)**  
   - **A**: Copy files  
   - **B**: Create directory on server  
   - **C**: List server contents  
   - **D**: Delete files  
   - **Answer**: B, C (TFTP only copies files).  

**Bonus Question (Boson ExSim)**:
- **Note**: Transcript references but doesn’t provide; typical:
  - **Question**: TFTP uses which port?
    - **A**: TCP 20
    - **B**: TCP 21
    - **C**: UDP 69
    - **D**: TCP 22
    - **Answer**: C (TFTP uses UDP 69).

**Lifelong Retention Tip**: Create flashcards for quiz answers (e.g., “FTP ports? TCP 20/21. TFTP port? UDP 69”). Practice questions 3 times.

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Ports: “FTP 20/21, TFTP 69” (FTP TCP, TFTP UDP).
   - FTP Modes: “Active Server, Passive Client” (ASPC).
2. **Visualize Topology**:
   - Draw R1 → SRV1 (TFTP UDP 69, FTP TCP 21). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “TFTP port? UDP 69. FTP ports? TCP 20/21. Startup-config? NVRAM.”
4. **Practice in Packet Tracer**:
   - Configure SRV1 as TFTP/FTP server, run `copy tftp: flash:`, `copy ftp: flash:`, verify (`show flash`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: FTP (TCP 20/21, authentication) vs. TFTP (UDP 69, none).
   - Quiz: “TFTP function? File copy only. FTP mode for firewall? Passive.”
6. **Troubleshooting Practice**:
   - Use wrong IP for TFTP, retry with 192.168.1.100, verify. Checklist: “No transfer? Check IP, filename, ports, ACLs.”
7. **Teach Someone**:
   - Explain FTP vs. TFTP and IOS transfer to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, ports, file systems. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **FTP/TFTP Purpose**:
   - Transfer files (e.g., IOS images, configs) over a network.
2. **TFTP**:
   - Simple, UDP 69, no authentication/encryption, file copy only.
3. **FTP**:
   - Complex, TCP 20 (data)/21 (control), authentication, supports directory navigation.
4. **File Systems**:
   - Flash (IOS), NVRAM (startup-config), network (FTP/TFTP servers).
5. **Commands**:
   - `copy tftp: flash:`, `copy ftp: flash:`, `boot system`, `delete flash:/file`.
6. **CCNA Focus**:
   - Topic 4.9: Know FTP (TCP 20/21, authentication), TFTP (UDP 69, simple), and their functions.

These notes provide a complete foundation for the TFTP lecture, preparing you for CCNA exam topic 4.9. Practice file transfers in Packet Tracer, verify with `show flash` and `show version`, and use analogies/retention tips to master FTP/TFTP for life!