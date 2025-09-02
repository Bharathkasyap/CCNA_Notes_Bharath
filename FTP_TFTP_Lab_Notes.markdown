# FTP & TFTP Lab Notes for CCNA Beginners

## Lab Overview

This lab demonstrates the use of **File Transfer Protocol (FTP)** and **Trivial File Transfer Protocol (TFTP)** to transfer Cisco IOS images to routers R1 and R2, aligning with **CCNA exam topic 4.9** ("Describe the capabilities and function of TFTP/FTP in the network"). Using Packet Tracer, it configures IP addresses and static routes for connectivity, copies an IOS image from SRV1 to R1 (via TFTP) and R2 (via FTP), updates the boot system, reloads the routers, and deletes old IOS images. The lab verifies transfers using `show flash` and `show version`, reinforcing the FTP/TFTP lecture concepts.

**Lab Objectives**:
- **Configure Connectivity**: Set up IP addresses and static routes for SRV1, R1, R2.
- **Transfer IOS via TFTP**: Copy IOS image to R1’s flash using TFTP (`copy tftp: flash:`).
- **Transfer IOS via FTP**: Copy IOS image to R2’s flash using FTP (`copy ftp: flash:`).
- **Update IOS**: Configure `boot system`, save config, reload, and verify new IOS version.
- **Delete Old IOS**: Remove old IOS images from flash (`delete flash:/file`).
- **Prepare for Exam**: Master FTP/TFTP capabilities and commands (topic 4.9).

**Network Topology**:
```
+----------------+       +----------------+       +----------------+
|      SRV1      |-------|      R1        |-------|      R2        |
| FTP/TFTP Server|       | 192.168.1.1    |       | 192.168.1.2    |
| 10.0.0.1       |       | Flash: IOS     |       | Flash: IOS     |
+----------------+       +----------------+       +----------------+
```
- **Devices**:
  - **SRV1**: FTP/TFTP server (10.0.0.1), hosts IOS image (`c2900-universalk9-mz.SPA.155-1.T.bin`).
  - **R1**: Router (192.168.1.1), receives IOS via TFTP.
  - **R2**: Router (192.168.1.2), receives IOS via FTP.
- **Note**: IP addresses and static route to 10.0.0.0/24 pre-configured on R2. SRV1 has TFTP enabled (UDP 69) and FTP enabled (TCP 20/21, users: `cisco/cisco`, `jeremy/ccna`).
- **Requirements**:
  1. Verify SRV1’s TFTP/FTP services and files.
  2. Copy IOS to R1 via TFTP (`copy tftp: flash:`).
  3. Copy IOS to R2 via FTP (`copy ftp: flash:` with `jeremy/ccna`).
  4. Configure `boot system`, save, reload, verify IOS (15.5).
  5. Delete old IOS (15.1) from flash.

**Analogy**: TFTP/FTP are like delivery services for IOS images to R1/R2’s storage (flash). TFTP is a quick, unsecured drop-off (UDP 69), while FTP is a secure courier (TCP 20/21) requiring ID (`jeremy/ccna`). Updating IOS is like installing new software, and deleting old files clears storage space.

**Lifelong Retention Tip**: Visualize R1/R2 downloading IOS from SRV1 (10.0.0.1) using TFTP (UDP 69) or FTP (TCP 21). In Packet Tracer, run `copy tftp: flash:` on R1, `copy ftp: flash:` on R2, verify with `show version`. Create flashcards: “TFTP port? UDP 69. FTP ports? TCP 20/21. Boot command? `boot system`.” Practice 3 times daily for a week.

## Lab Steps and Analysis

### Step 1: Verify SRV1 Configuration
**Objective**: Confirm SRV1’s TFTP and FTP services and available files.

**Steps (on SRV1, Packet Tracer)**:
1. Go to Services → TFTP:
   - TFTP enabled (default).
   - Files listed, including `c2900-universalk9-mz.SPA.155-1.T.bin`.
2. Go to Services → FTP:
   - FTP enabled (default).
   - Users: `cisco/cisco`, `jeremy/ccna`.
   - Same files as TFTP.

**Observations**:
- **TFTP**: No authentication; responds to all clients (UDP 69).
- **FTP**: Requires username/password (`jeremy/ccna` used for R2).
- **Files**: IOS image (`c2900-universalk9-mz.SPA.155-1.T.bin`) available for transfer.
- **Packet Tracer**: Services enabled by default; real-world servers require manual setup.
- **Exam Note**: Understand TFTP’s simplicity (no authentication) vs. FTP’s authentication (topic 4.9).

**Comparison to Prior Content**:
- **TFTP Lecture**: Lecture noted TFTP (UDP 69, no security) and FTP (TCP 20/21, authentication); lab uses both.
- **Syslog Lab**: File transfers may trigger Syslog (e.g., `%SYS-5-CONFIG_I` for `boot system`).
- **SSH Lecture**: SSH secures CLI for running `copy` commands; FTP/TFTP are plain-text.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: TFTP UDP 69, FTP TCP 20/21 vs. SSH TCP 22.
- **Security (Section 5.0)**: FTP authenticates; TFTP doesn’t; both lack encryption.
- **Extended ACLs (Day 35)**: ACLs can block TFTP (UDP 69) or FTP (TCP 20/21).

**Analogy**: SRV1 is like a warehouse stocking IOS files, with TFTP as an open drop-off (no ID check) and FTP as a secure pickup requiring ID (`jeremy/ccna`).

**Lifelong Retention Tip**: In Packet Tracer, check SRV1’s TFTP/FTP services, note IOS filename. Flashcard: “TFTP security? None. FTP user? `jeremy/ccna`.” Practice 3 times.

### Step 2: TFTP Transfer to R1
**Objective**: Copy IOS image to R1’s flash using TFTP, update boot, reload, delete old IOS.

**Steps (on R1)**:
1. Verify current IOS:
```plaintext
R1# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.1(4)M4
```
2. Check flash:
```plaintext
R1# show flash
-#- --length-- -----date/time------ path
1   123456789  Jan 1 2025 12:00:00 c2900-universalk9-mz.SPA.151-4.M4.bin
```
3. Copy IOS via TFTP:
```plaintext
R1# copy tftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
4. Configure boot:
```plaintext
R1# configure terminal
R1(config)# boot system flash:/c2900-universalk9-mz.SPA.155-1.T.bin
R1(config)# exit
R1# write memory
```
5. Reload:
```plaintext
R1# reload
```
6. Verify new IOS:
```plaintext
R1# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.5(1)T
```
7. Delete old IOS:
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
- **TFTP**: `copy tftp: flash:` requires server IP (10.0.0.1), filename; no authentication.
- **Boot System**: `boot system flash:/file` ensures new IOS loads; requires `write memory`.
- **Packet Tracer**: TFTP transfer is fast; real-world may vary.
- **Exam Note**: Know `copy tftp: flash:`, `boot system` for IOS upgrades (topic 4.9).

**Comparison to Prior Content**:
- **TFTP Lecture**: Lab mirrors lecture’s TFTP process (`copy tftp: flash:`).
- **Syslog Lab**: `boot system`, `write` trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: SSH secures CLI for `copy` commands; TFTP is plain-text.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: TFTP UDP 69 vs. FTP TCP 20/21.
- **Security (Section 5.0)**: TFTP lacks security; use in controlled LAN.
- **Extended ACLs (Day 35)**: ACLs block TFTP (e.g., `deny udp any host 10.0.0.1 eq 69`).

**Analogy**: TFTP on R1 is like a quick, unsecured delivery (UDP 69) dropping an IOS package from SRV1’s warehouse to R1’s storage (flash), then updating software (`boot system`).

**Lifelong Retention Tip**: In Packet Tracer, run `copy tftp: flash:`, set `boot system`, reload, verify with `show version`. Flashcard: “TFTP copy? `copy tftp: flash:`. Port? UDP 69.” Practice 3 times.

### Step 3: FTP Transfer to R2
**Objective**: Copy IOS image to R2’s flash using FTP, update boot, reload, delete old IOS.

**Steps (on R2)**:
1. Verify current IOS:
```plaintext
R2# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.1(4)M4
```
2. Check flash:
```plaintext
R2# show flash
-#- --length-- -----date/time------ path
1   123456789  Jan 1 2025 12:00:00 c2900-universalk9-mz.SPA.151-4.M4.bin
```
3. Configure FTP credentials:
```plaintext
R2# configure terminal
R2(config)# ip ftp username jeremy
R2(config)# ip ftp password ccna
R2(config)# exit
```
4. Copy IOS via FTP:
```plaintext
R2# copy ftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
5. Configure boot:
```plaintext
R2# configure terminal
R2(config)# boot system flash:/c2900-universalk9-mz.SPA.155-1.T.bin
R2(config)# exit
R2# write memory
```
6. Reload:
```plaintext
R2# reload
```
7. Verify new IOS:
```plaintext
R2# show version
Cisco IOS Software, C2900 Software (C2900-UNIVERSALK9-M), Version 15.5(1)T
```
8. Delete old IOS:
```plaintext
R2# delete flash:/c2900-universalk9-mz.SPA.151-4.M4.bin
Delete filename [c2900-universalk9-mz.SPA.151-4.M4.bin]? [Enter]
```

**Verification**:
- **Show Flash** (after copy):
```plaintext
R2# show flash
-#- --length-- -----date/time------ path
1   123456789  Jan 1 2025 12:00:00 c2900-universalk9-mz.SPA.151-4.M4.bin
2   123456790  Jan 1 2025 12:05:00 c2900-universalk9-mz.SPA.155-1.T.bin
```
- **Show Version** (after reload): Confirms 15.5(1)T.
- **Show Flash** (after delete): Old file gone.

**Observations**:
- **FTP**: `copy ftp: flash:` requires `ip ftp username/password` matching SRV1 (`jeremy/ccna`).
- **Speed**: FTP slower than TFTP in Packet Tracer; real-world varies.
- **Boot System**: `boot system` and `write memory` ensure new IOS loads.
- **Exam Note**: Know `copy ftp: flash:`, `ip ftp username/password` (topic 4.9).

**Comparison to Prior Content**:
- **TFTP Lecture**: Lab mirrors FTP process (`copy ftp: flash:`) with authentication.
- **Syslog Lab**: FTP transfer, `boot system` trigger Syslog (e.g., `%SYS-5-CONFIG_I`).
- **SSH Lecture**: SSH secures CLI; FTP uses plain-text authentication.

**Comparison to Other CCNA Topics**:
- **Day 30 (Ports)**: FTP TCP 20/21 vs. TFTP UDP 69.
- **Security (Section 5.0)**: FTP authenticates; no encryption unless FTPS.
- **Extended ACLs (Day 35)**: ACLs block FTP (e.g., `deny tcp any host 10.0.0.1 eq 21`).

**Analogy**: FTP on R2 is like a secure courier (TCP 20/21) delivering an IOS package from SRV1 to R2’s storage (flash), requiring ID (`jeremy/ccna`), then updating software.

**Lifelong Retention Tip**: In Packet Tracer, configure `ip ftp username jeremy`, run `copy ftp: flash:`, set `boot system`, verify with `show version`. Flashcard: “FTP copy? `copy ftp: flash:`. Credentials? `ip ftp username/password`.” Practice 3 times.

## Common Issues and Troubleshooting

- **TFTP Copy Fails**:
  - **Issue**: `copy tftp: flash:` fails on R1.
  - **Cause**: Wrong IP (not 10.0.0.1), filename, or UDP 69 blocked.
  - **Fix**: Verify IP, filename (`c2900-universalk9-mz.SPA.155-1.T.bin`), check ACLs (`show access-lists`).
- **FTP Copy Fails**:
  - **Issue**: `copy ftp: flash:` fails on R2.
  - **Cause**: Incorrect `ip ftp username/password` or TCP 20/21 blocked.
  - **Fix**: Verify `ip ftp username jeremy`, `ip ftp password ccna`, check ACLs.
- **Wrong IOS Boots**:
  - **Issue**: Old IOS (15.1) boots after reload.
  - **Cause**: Missing `boot system` or unsaved config.
  - **Fix**: Configure `boot system flash:/new-image.bin`, `write memory`, reload.
- **Flash Full**:
  - **Issue**: Copy fails due to insufficient space.
  - **Cause**: Old IOS not deleted.
  - **Fix**: Run `show flash`, delete old file (`delete flash:/old-image.bin`).
- **Slow FTP Transfer**:
  - **Issue**: FTP transfer takes long in Packet Tracer.
  - **Cause**: Packet Tracer simulation delay.
  - **Fix**: Wait for completion; real-world performance varies.

**Analogy**: Troubleshooting is like ensuring a delivery (TFTP/FTP) reaches R1/R2’s warehouse (flash) with correct address (IP), package name (filename), and clear roads (no ACL blocks).

**Lifelong Retention Tip**: Simulate TFTP failure (wrong IP), retry with 10.0.0.1, verify with `show flash`. Checklist: “No transfer? Check IP, filename, credentials, ports, ACLs.” Practice 3 times.

## Practice Commands

- **Verify IOS and Flash**:
```plaintext
R1# show version
R1# show flash
```
- **TFTP Transfer (R1)**:
```plaintext
R1# copy tftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
- **FTP Transfer (R2)**:
```plaintext
R2# configure terminal
R2(config)# ip ftp username jeremy
R2(config)# ip ftp password ccna
R2# copy ftp: flash:
Address or name of remote host []? 10.0.0.1
Source filename []? c2900-universalk9-mz.SPA.155-1.T.bin
Destination filename [c2900-universalk9-mz.SPA.155-1.T.bin]? [Enter]
```
- **Configure Boot (R1/R2)**:
```plaintext
R1# configure terminal
R1(config)# boot system flash:/c2900-universalk9-mz.SPA.155-1.T.bin
R1(config)# exit
R1# write memory
R1# reload
```
- **Delete Old IOS (R1/R2)**:
```plaintext
R1# delete flash:/c2900-universalk9-mz.SPA.151-4.M4.bin
```

## Study Tips for Lifelong Retention

1. **Mnemonics**:
   - Ports: “FTP 20/21, TFTP 69” (FTP TCP, TFTP UDP).
   - FTP Credentials: “Jeremy’s CCNA for FTP” (`jeremy/ccna`).
2. **Visualize Topology**:
   - Draw SRV1 → R1 (TFTP), R2 (FTP). Pin to study space.
3. **Memorize Key Values**:
   - Flashcards: “TFTP port? UDP 69. FTP ports? TCP 20/21. Boot command? `boot system`.”
4. **Practice in Packet Tracer**:
   - Configure SRV1, run `copy tftp: flash:` (R1), `copy ftp: flash:` (R2), verify (`show version`). Repeat 3–5 times.
5. **Compare to Prior Content**:
   - Table: TFTP (UDP 69, no authentication) vs. FTP (TCP 20/21, `jeremy/ccna`).
   - Quiz: “TFTP function? File copy only. FTP credentials? `ip ftp username/password`.”
6. **Troubleshooting Practice**:
   - Use wrong IP for TFTP, retry with 10.0.0.1, verify. Checklist: “No transfer? Check IP, filename, credentials, ports.”
7. **Teach Someone**:
   - Explain TFTP vs. FTP and IOS upgrade process to a peer.
8. **Spaced Repetition**:
   - Use Anki for commands, ports, credentials. Review after 1 day, 1 week, 1 month.

## Key Takeaways

1. **TFTP**:
   - Simple, UDP 69, no authentication, `copy tftp: flash:`.
2. **FTP**:
   - Complex, TCP 20/21, requires `ip ftp username/password`, `copy ftp: flash:`.
3. **IOS Upgrade**:
   - Copy IOS to flash, set `boot system`, save (`write`), reload, delete old IOS.
4. **Verification**:
   - `show version` (new IOS), `show flash` (files), `show file systems`.
5. **CCNA Focus**:
   - Topic 4.9: Master TFTP (UDP 69, simple), FTP (TCP 20/21, authentication), and commands.

These notes provide a complete foundation for the FTP/TFTP lab, reinforcing CCNA exam topic 4.9. Practice transfers in Packet Tracer, verify with `show flash` and `show version`, and use analogies/retention tips to master FTP/TFTP for life!