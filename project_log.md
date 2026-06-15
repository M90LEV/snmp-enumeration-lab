# SNMP Enumeration Lab — Project Log

## Date: 15 June 2026

## Overview
Worked through a full SNMP enumeration lab simulator covering 39 commands 
across 6 phases. Lab designed for CREST CPSA exam preparation focusing on 
network protocol enumeration techniques.

## Target Network
| Host | IP | Role |
|------|----|------|
| CORE-RTR-01 | 192.168.1.1 | Cisco IOS Core Router (SNMPv2c) |
| ACCESS-SW-01 | 192.168.1.2 | Cisco Catalyst 2960 Switch (SNMPv1) |
| BACKUP-RTR-01 | 10.10.0.1 | Legacy Cisco Router (SNMPv2c) |
| TFTP-SRV-01 | 192.168.1.50 | Linux TFTP Server (no auth) |
| FW-01 | 192.168.10.5 | FortiGate Firewall |

## Phase 1 — Discovery
- Used nmap -sU -p161 to confirm SNMP running on UDP/161
- Learned that TCP scans miss SNMP entirely — must use -sU
- Used onesixtyone to bruteforce community strings
- Found three valid community strings: public, private, internal
- private confirmed as read-write access

## Phase 2 — Enumeration
- snmpwalk retrieved 847 OIDs from core router
- snmpbulkwalk retrieved 1,284 OIDs faster using GETBULK
- Extracted routing table revealing previously unknown networks
- Dumped ARP cache revealing live hosts without port scanning
- Enumerated running processes confirming telnetd and tftp running
- Used snmpget for targeted single OID queries
- Used snmp-check and snmp-enum for automated full audits
- Found user accounts: admin, monitor, readonly, guest

## Phase 3 — Exploitation and Write Access
- Confirmed read-write access using snmpset with private community string
- Successfully overwrote sysName and sysContact OIDs
- Confirmed public community string is read-only — SET operations blocked

## Phase 4 — Metasploit
- Loaded auxiliary/scanner/snmp/snmp_enum module
- Set RHOSTS and COMMUNITY options
- Module profiled all three Cisco devices automatically
- Results saved to Metasploit database automatically

## Phase 5 — TFTP Config Extraction
- Connected to TFTP server with zero authentication
- Downloaded all four device config files
- Config files contained Type 7 passwords — reversible obfuscation
- SNMP community strings visible in plaintext in config files
- Telnet confirmed as only VTY management method

## Phase 6 — Target Switching
- Practiced running commands against different targets
- Confirmed SNMPv1 limitations — no GETBULK available
- Enumerated legacy backup router separately

## Key Vulnerabilities Identified
- SNMPv1 and v2c in use — no encryption or real authentication
- Default community strings (public/private) valid on all devices
- Read-write community string exposed — SNMP SET possible
- Telnet enabled — plaintext credential transmission
- TFTP server unauthenticated — config files freely downloadable
- Type 7 passwords in config files — trivially reversible
- Outdated firmware on all devices

## Key Learnings
- SNMP enumeration can map an entire network from a single device
- Routing table, ARP cache and process list reveal more than a port scan
- Read-write SNMP access can trigger config copy to attacker controlled TFTP
- Type 7 passwords are obfuscation not encryption — not a security control
- SNMPv3 with authPriv is the only secure SNMP configuration

## Remediation Summary
- Migrate all devices to SNMPv3 authPriv with AES encryption
- Remove default community strings — use strong random strings
- Restrict SNMP to management VLAN only via ACL
- Disable Telnet — enforce SSH only (transport input ssh)
- Replace TFTP with SCP or SFTP
- Replace Type 7 passwords with enable secret (Type 5 minimum)
- Patch all devices to supported firmware versions

## Tools Used
nmap, onesixtyone, snmpwalk, snmpbulkwalk, snmpget, snmpset, 
snmp-check, snmp-enum, braa, tftp, msfconsole
