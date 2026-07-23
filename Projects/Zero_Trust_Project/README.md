# Zero Trust Enterprise Network Architecture
### Cisco Packet Tracer | Multi-Layer Switching | OSPF | HSRP | ACL | NAT | SSH | AAA | Syslog | NTP

> A fully simulated enterprise-grade Zero Trust network built in Cisco Packet Tracer, demonstrating multi-zone segmentation, dynamic routing, gateway redundancy, layered access control, and centralized monitoring — designed to mirror real-world security architecture principles.

Table of Contents
## Project Overview
## Network Topology
## Network Architecture
## IP Addressing Plan
## VLAN & Zone Design
## Zero Trust Security Architecture
## Features Implemented
## Security Implementation
## ACL Trust Matrix
## High Availability & Redundancy
## Routing & Internet Access
## Monitoring & Management
## Test Results
## Design Decisions
## Tools Used
## Packet Tracer Limitations
## Future Improvements
## Key Learning Outcomes
## Conclusion
## Project Overview

This project simulates an enterprise Zero Trust network segmented into four security zones — Management, Server Farm, Employee LAN, and DMZ — each isolated at Layer 2 and Layer 3 with explicit, policy-driven inter-zone communication rules. No device, user, or zone is trusted by default. Every inter-zone flow is governed by ACL policy, management access requires SSH v2 and AAA authentication, end devices are bound to ports via Port Security, and all activity is forwarded to a centralized Syslog server.

Attribute	Detail
Simulation Platform	Cisco Packet Tracer
Network Zones	Management, Server Farm, Employee LAN, DMZ
Layer 2 Switches	SW-MGMT, SW-SERVER, SW-EMPLOYEE, SW-DMZ, SW-Internet
Multilayer Switches	MLS1 (Primary), MLS2 (Secondary)
Routers	Border-Router, ISP-Router
End Devices	ADMIN-PC, APP-SERVER, EMP-PC1, EMP-PC2, WEB-SERVER, PUBLIC-PC
Routing Protocol	OSPF (Single Area — Area 0)
Gateway Redundancy	HSRP (Hot Standby Router Protocol)
Internet Access	NAT Overload (PAT) via Serial WAN link
Access Control	Named Extended ACLs applied inbound on VLAN SVIs
Management Security	SSH v2, AAA Local Authentication on all devices
Monitoring	Centralized Syslog, NTP Synchronization
Spanning Tree	Rapid PVST+ with Root Bridge election

## Network Topology
```
                        ┌──────────────┐
                        │  ISP-Router  │
                        │ 200.1.1.1/30 │
                        │ 100.1.1.1/24 │
                        └──────┬───────┘
                    Serial WAN │ 200.1.1.x/30
                               │
                        ┌──────┴───────┐
                        │Border-Router │
                        │ 200.1.1.2/30 │
                        │ 172.16.1.2   │
                        │ 172.16.1.6   │
                        └──────┬───────┘
               ┌───────────────┴───────────────┐
          /30  │ 172.16.1.1              172.16.1.5  │ /30
    ┌──────────┴──────┐               ┌──────────────┴──┐
    │      MLS1       │───────────────│      MLS2       │
    │  Primary L3 SW  │  Inter-switch │  Secondary L3 SW│
    │  HSRP Active    │     Link      │  HSRP Standby   │
    │  OSPF RID 1.1.1.1│             │  OSPF RID 2.2.2.2│
    └──┬──────┬───────┘               └──────┬──────┬───┘
       │      │                              │      │
  ┌────┘  ┌───┘                          ┌───┘  ┌───┘
  │       │                              │      │
┌─┴────┐ ┌┴──────┐                 ┌────┴─┐  ┌─┴────┐
│SW-   │ │SW-    │                 │SW-   │  │SW-   │
│MGMT  │ │SERVER │                 │EMPL  │  │DMZ   │
│VLAN10│ │VLAN20 │                 │VLAN30│  │VLAN40│
└──┬───┘ └───┬───┘                 └──┬───┘  └───┬──┘
   │         │                        │           │
┌──┴───┐ ┌───┴──────┐       ┌────┬───┘       ┌───┴──────┐
│ADMIN │ │APP-      │       │EMP-│EMP-        │WEB-      │
│PC    │ │SERVER    │       │PC1 │PC2         │SERVER    │
│.10   │ │.10       │       │.10 │.11         │.10       │
└──────┘ └──────────┘       └────┴────        └──────────┘

  VLAN 10               VLAN 20              VLAN 30    VLAN 40
  192.168.10.0/24       192.168.20.0/24      192.168.30.0/24
                                                         192.168.40.0/24

                ISP Side:
                ┌─────────────┐
                │  SW-Internet│
                └──┬───────┬──┘
                   │       │
              ┌────┘    ┌───┘
        ISP-Router   PUBLIC-PC
        100.1.1.1    100.1.1.10
```

## Network Architecture
Management Zone — VLAN 10 | 192.168.10.0/24

Hosts the ADMIN-PC and Syslog server (192.168.10.100). Unrestricted outbound access for administrative operations; all inbound traffic from Employee and DMZ zones is explicitly denied by ACL policy, creating a fully isolated management plane.

Server Farm Zone — VLAN 20 | 192.168.20.0/24

Hosts the internal APP-SERVER. Accepts inbound HTTPS from the Employee zone and HTTP/HTTPS from the DMZ. No public address is assigned; all access is mediated through zone ACL policy enforced at the VLAN 20 SVI.

Employee LAN Zone — VLAN 30 | 192.168.30.0/24

Contains end-user workstations EMP-PC1 and EMP-PC2. Permitted outbound HTTPS to the Server Farm and HTTP to the DMZ web server. Access to the Management zone is denied unconditionally.

DMZ — VLAN 40 | 192.168.40.0/24

Hosts the WEB-SERVER representing a public-facing service. Permitted to forward HTTP and HTTPS to the Server Farm only. Denied access to both the Management and Employee zones, containing any compromise within VLAN 40.

Border Router

Network edge device terminating the WAN serial link. Performs NAT overload (PAT), holds the static default route toward the ISP, distributes it into OSPF via default-information originate, and serves as NTP master. Connected to MLS1 (172.16.1.2/30) and MLS2 (172.16.1.6/30).

ISP Network

Simulated by ISP-Router and SW-Internet with PUBLIC-PC. ISP-Router holds static routes back to all four internal subnets via 200.1.1.2. PUBLIC-PC (100.1.1.10) validates end-to-end NAT translation.


## IP Addressing Plan
VLAN Subnets & Gateways
VLAN	Name	Subnet	MLS1 SVI	MLS2 SVI	HSRP Virtual GW
10	MANAGEMENT	192.168.10.0/24	192.168.10.1	192.168.10.2	192.168.10.254
20	SERVER_FARM	192.168.20.0/24	192.168.20.1	192.168.20.2	192.168.20.254
30	EMPLOYEE	192.168.30.0/24	192.168.30.1	192.168.30.2	192.168.30.254
40	DMZ	192.168.40.0/24	192.168.40.1	192.168.40.2	192.168.40.254
99	NATIVE_UNUSED	N/A	N/A	N/A	N/A
End Device Addressing
Device	Zone	IP Address	Subnet Mask	Default Gateway
ADMIN-PC	Management	192.168.10.10	255.255.255.0	192.168.10.254
APP-SERVER	Server Farm	192.168.20.10	255.255.255.0	192.168.20.254
EMP-PC1	Employee	192.168.30.10	255.255.255.0	192.168.30.254
EMP-PC2	Employee	192.168.30.11	255.255.255.0	192.168.30.254
WEB-SERVER	DMZ	192.168.40.10	255.255.255.0	192.168.40.254
Routed Point-to-Point Links (/30)
Link	Device	Interface	IP Address
MLS1 ↔ Border Router	MLS1	fa0/1	172.16.1.1/30
MLS1 ↔ Border Router	Border-Router	Gi0/0	172.16.1.2/30
MLS2 ↔ Border Router	MLS2	fa0/1	172.16.1.5/30
MLS2 ↔ Border Router	Border-Router	Gi0/1	172.16.1.6/30
WAN & ISP Addressing
Segment	Device	IP Address
WAN Serial Link	Border-Router	200.1.1.2/30
WAN Serial Link	ISP-Router	200.1.1.1/30
ISP LAN	ISP-Router	100.1.1.1/24
ISP LAN	PUBLIC-PC	100.1.1.10/24
OSPF Router IDs
Device	Router ID
MLS1	1.1.1.1
MLS2	2.2.2.2
Border-Router	3.3.3.3

## VLAN & Zone Design
VLAN	Name	Security Zone	Devices	Purpose	Trust Level
10	MANAGEMENT	Management	ADMIN-PC, Syslog Server	Out-of-band management plane. Hosts administrative workstation and centralized logging.	Highest
20	SERVER_FARM	Server Farm	APP-SERVER	Internal application services. Accepts controlled inbound traffic from Employee and DMZ only.	High
30	EMPLOYEE	Employee LAN	EMP-PC1, EMP-PC2	End-user access layer. Most likely compromise vector — tightly controlled outbound permissions.	Medium
40	DMZ	Demilitarized Zone	WEB-SERVER	Public-facing services. Isolated from internal zones. Can only communicate forward to Server Farm.	Low
99	NATIVE_UNUSED	N/A	None	Unused native VLAN. Assigned to all trunk ports to mitigate VLAN hopping attacks.	None

## Zero Trust Security Architecture
Never Trust, Always Verify

Every inter-zone communication is governed by an explicit ACL permit rule. No implicit trust relationships exist between zones — a device in the Employee zone cannot reach the Management zone under any circumstances, because the policy says so and the ACL enforces it unconditionally.

Least Privilege Access

Each zone is granted only the minimum access required for its operational role:

Employee → Server Farm: HTTPS (443) only
Employee → DMZ: HTTP (80) only
DMZ → Server Farm: HTTP (80) and HTTPS (443) only
Management: Outbound unrestricted; inbound blocked from all other zones
Network Segmentation

Four independent Layer 2 domains are enforced through VLAN isolation. Inter-VLAN routing occurs only through Multilayer Switch SVIs, where ACLs intercept all cross-zone traffic before any forwarding decision is made.

Secure Management Plane

All device management sessions require SSH v2 (Telnet is disabled on all devices), AAA local authentication with named credentials, exec-timeout after 10 minutes of inactivity, and a legal warning banner displayed before authentication.

Monitoring & Visibility

Centralized Syslog with millisecond-precision timestamps captures every network event. NTP synchronization across all devices guarantees log timestamp integrity for forensic correlation — implementing the "Assume Breach" pillar of Zero Trust.

Layered Defense
Layer	Control	Mechanism
Layer 1	Physical port control	Port Security (sticky MAC, max 1, shutdown violation)
Layer 2	STP protection	BPDU Guard on all access ports
Layer 2	Trunk security	Native VLAN 99 (unused), explicit VLAN allow-list
Layer 3	Zone enforcement	Named Extended ACLs on SVIs
Layer 3	Routing security	OSPF passive interfaces on all end-user VLANs
Layer 4+	Management encryption	SSH v2 + AAA
Management plane	Identity	Local AAA database, privilege 15 admin account
Visibility	Logging	Centralized Syslog + NTP

## Features Implemented
Feature	Status	Description
VLAN Segmentation	✅ Implemented	Four security zones isolated via VLANs 10/20/30/40 with VLAN 99 as unused native.
802.1Q Trunking	✅ Implemented	Explicit VLAN allow-lists, native VLAN 99, and dot1q encapsulation on all inter-switch links.
Port Security	✅ Implemented	Sticky MAC learning, maximum 1 MAC per port, shutdown on violation on all access ports.
PortFast	✅ Implemented	Eliminates STP convergence delay on all access ports connected to end devices.
BPDU Guard	✅ Implemented	Blocks unauthorized switch insertion by error-disabling any access port receiving a BPDU.
Rapid PVST+	✅ Implemented	Sub-second STP convergence with MLS1 as primary root and MLS2 as secondary root.
Inter-VLAN Routing	✅ Implemented	Layer 3 SVIs on MLS1 and MLS2 with ip routing enabled for all four zones.
OSPF (Area 0)	✅ Implemented	Dynamic routing across MLS switches and Border Router with passive interfaces on all VLAN SVIs.
HSRP	✅ Implemented	Virtual gateways (.254) for all four VLANs; MLS1 active (priority 110), MLS2 standby (priority 100).
Named Extended ACLs	✅ Implemented	EMPLOYEE-ACL and DMZ-ACL applied inbound on VLAN 30 and VLAN 40 SVIs.
Device Hardening	✅ Implemented	Consistent security baseline applied to every device: enable secret, password-encryption, exec-timeout, banner.
SSH v2	✅ Implemented	SSH version 2 enforced on all devices; Telnet fully disabled on all VTY lines.
AAA Local Authentication	✅ Implemented	aaa new-model with local authentication and authorization enforced on all devices.
NAT Overload (PAT)	✅ Implemented	All internal RFC 1918 ranges translated to single public IP on Border Router serial interface.
Static Default Route	✅ Implemented	0.0.0.0/0 toward ISP redistributed into OSPF via default-information originate.
Syslog	✅ Implemented	Centralized logging to 192.168.10.100 at Informational level with millisecond timestamps.
NTP	✅ Implemented	Border Router as stratum 3 master with cascading synchronization to all internal and ISP devices.
ISP Simulation	✅ Implemented	ISP-Router with return static routes and PUBLIC-PC for end-to-end internet connectivity validation.
Routed P2P Links	✅ Implemented	/30 point-to-point subnets between MLS switches and Border Router using no switchport.

## Security Implementation
Device Hardening

A consistent security baseline is applied to every network device — both Multilayer Switches, all Layer 2 switches, the Border Router, and the ISP Router:

enable secret — type 5 MD5 hashing, not reversible type 7
service password-encryption — prevents plaintext password exposure in running configurations
no ip domain-lookup — prevents unintended DNS queries from the CLI
exec-timeout 10 0 — terminates idle VTY sessions after 10 minutes
Legal warning banner displayed at every login attempt
SSH v2

Telnet is disabled on every VTY line across every device. SSH v2 is the sole permitted management transport, with RSA keys generated at 1024-bit (Packet Tracer maximum) and ip ssh version 2 enforced to prevent downgrade to SSHv1. transport input ssh is applied to both line vty 0 4 and line vty 5 15 on every device.

AAA — Authentication, Authorization, Accounting

Local AAA replaces shared line passwords. Every management session is associated with a named identity (username admin privilege 15 secret). aaa new-model with aaa authentication login default local and aaa authorization exec default local is enforced on all devices. login local is applied to all VTY lines.

Port Security
Parameter	Value	Security Purpose
Maximum MAC addresses	1	Only one physical device per port
Violation action	Shutdown	Port error-disables immediately on second MAC detection
MAC learning	Sticky	First learned MAC is saved to running config automatically
Rapid PVST+

Rapid PVST+ is configured across all switches. MLS1 is elected primary root bridge for VLANs 10, 20, 30, 40, and 99 (bridge priority 24576 via root primary). MLS2 is secondary (priority 28672). Deterministic root bridge placement prevents any access switch from becoming root through default priority election.


## ACL Trust Matrix

Named extended ACLs (EMPLOYEE-ACL and DMZ-ACL) are applied inbound on VLAN 30 and VLAN 40 SVIs on both MLS1 and MLS2. Inline remark statements document the intent of every rule.

Management Plane Security

All inbound traffic from the Employee and DMZ zones toward 192.168.10.0/24 is denied by ACL policy. The management plane is reachable only from within VLAN 10, providing an isolated administrative boundary equivalent to an out-of-band management network.


## ACL Trust Matrix
Policy Definition
Source Zone	Destination Zone	Protocol	Port	Action	Policy Rationale
Management	All	IP	Any	✅ Permit	Administrative zone requires unrestricted outbound access
Employee	Server Farm	TCP	443 (HTTPS)	✅ Permit	Encrypted application access
Employee	DMZ	TCP	80 (HTTP)	✅ Permit	Web content access
Employee	Management	IP	Any	❌ Deny	Management plane must not be reachable from user devices
Employee	Any	IP	Any	✅ Permit	General internet access via NAT
DMZ	Server Farm	TCP	80 (HTTP)	✅ Permit	Web-to-app tier communication
DMZ	Server Farm	TCP	443 (HTTPS)	✅ Permit	Encrypted web-to-app communication
DMZ	Management	IP	Any	❌ Deny	Public-facing zone must never reach management
DMZ	Employee	IP	Any	❌ Deny	DMZ has no legitimate reason to initiate connections to end users
DMZ	Any	IP	Any	✅ Permit	Internet access for updates and external communication
ACL Application Points
ACL Name	Applied Interface	Direction	Scope
EMPLOYEE-ACL	VLAN 30 SVI (MLS1 & MLS2)	Inbound	All traffic originating from Employee zone
DMZ-ACL	VLAN 40 SVI (MLS1 & MLS2)	Inbound	All traffic originating from DMZ zone

## High Availability & Redundancy
HSRP — Hot Standby Router Protocol

HSRP provides default gateway redundancy for all four security zones. End devices use a virtual IP (.254) not tied to any physical interface. If the active switch fails, the standby assumes the virtual IP within seconds — with no end-device reconfiguration required.

VLAN	Virtual Gateway	MLS1 Role	MLS1 Priority	MLS2 Role	MLS2 Priority
10	192.168.10.254	Active	110	Standby	100
20	192.168.20.254	Active	110	Standby	100
30	192.168.30.254	Active	110	Standby	100
40	192.168.40.254	Active	110	Standby	100
Preemption

standby preempt is configured exclusively on MLS1. When MLS1 recovers after a failure, it automatically reclaims the Active role without manual intervention. MLS2 does not have preempt configured, preventing role bounce during steady-state operation.

STP Root Bridge Election
Device	STP Role	Priority (all VLANs)
MLS1	Primary Root	24576 (root primary macro)
MLS2	Secondary Root	28672 (root secondary macro)

## Routing & Internet Access
OSPF — Open Shortest Path First

OSPF Process 1 operates across MLS1, MLS2, and the Border Router within a single Area 0. All four VLAN subnets and both /30 uplink networks are advertised. Passive interfaces on all VLAN SVIs prevent OSPF Hello packets from reaching end devices.

Device	Networks Advertised	Passive Interfaces
MLS1	192.168.10–40.0/24, 172.16.1.0/30	Vlan10, Vlan20, Vlan30, Vlan40
MLS2	192.168.10–40.0/24, 172.16.1.4/30	Vlan10, Vlan20, Vlan30, Vlan40
Border-Router	172.16.1.0/30, 172.16.1.4/30	None
Default Route Distribution

The Border Router holds a static default route (0.0.0.0/0 → 200.1.1.1) and redistributes it via default-information originate. All internal OSPF routers receive it as an O*E2 external route, enabling internet-bound traffic to reach the Border Router without per-device static configuration.

NAT Overload (PAT)

All RFC 1918 traffic matching access-list 1 (192.168.0.0/0.0.255.255) is translated to the Border Router's serial interface IP (200.1.1.2). Both Gi0/0 (toward MLS1) and Gi0/1 (toward MLS2) are marked ip nat inside; the serial interface is ip nat outside. This dual-inside configuration ensures NAT operates for traffic arriving from either distribution switch.

ISP Simulation

ISP-Router holds static routes for all four internal subnets via 200.1.1.2, plus a /16 summary route (192.168.0.0/255.255.0.0). PUBLIC-PC (100.1.1.10) provides a reachable internet-side endpoint for validating end-to-end NAT translation.


## Monitoring & Management
SSH v2 + AAA

All administrative access is secured through SSH v2 with AAA local database authentication and authorization. Per-session identity is tracked; shared credential risks are eliminated. Sessions terminate after 10 minutes of inactivity.

Syslog
Parameter	Value
Syslog Server IP	192.168.10.100
Syslog Severity Level	Informational (Level 6)
Timestamp Precision	Milliseconds (datetime msec)
Devices Logging	All switches, Border Router, ISP Router
Syslog Server Location	VLAN 10 (Management Zone) — SW-MGMT fa0/3

All devices forward log messages to 192.168.10.100 in the Management zone. Placing the Syslog server in the isolated Management VLAN ensures log records cannot be tampered with even if other zones are compromised.

NTP
Device	NTP Server	Stratum
Border-Router	Internal master	3
MLS1, MLS2, all L2 switches	172.16.1.2	4
ISP-Router	200.1.1.2	4
SW-Internet	100.1.1.1	5

Consistent timestamps across all devices enable accurate cross-device Syslog correlation and forensic integrity.


## Test Results
Test	Method	Expected Result	Actual Result
SSH Login — MLS1	SSH from ADMIN-PC to MLS1 SVI	Authenticated via AAA local database	✅ Pass
SSH Login — Layer 2 Switches	SSH from ADMIN-PC to SW-MGMT	Authenticated via AAA local database	✅ Pass
Telnet Blocked	Telnet attempt to any device	Connection refused	✅ Pass
VLAN Isolation	Ping EMP-PC1 → ADMIN-PC (different VLAN, same switch)	No L2 reachability without routing	✅ Pass
Port Security	Connect second device to access port	Port enters err-disabled state	✅ Pass
BPDU Guard	Connect unauthorized switch to access port	Port enters err-disabled state	✅ Pass
STP Root — MLS1	show spanning-tree vlan 10	MLS1 listed as Root Bridge	✅ Pass
STP Root — MLS2	show spanning-tree vlan 10 on MLS2	MLS2 listed as secondary	✅ Pass
Inter-VLAN Routing	Ping EMP-PC1 → APP-SERVER	Routed via MLS SVI, ICMP reply received	✅ Pass
OSPF Adjacency	show ip ospf neighbor on MLS1	FULL state with Border Router	✅ Pass
Default Route Propagation	show ip route on MLS1	O*E2 0.0.0.0/0 present	✅ Pass
HSRP Active State	show standby brief on MLS1	Active for VLANs 10/20/30/40	✅ Pass
HSRP Standby State	show standby brief on MLS2	Standby for VLANs 10/20/30/40	✅ Pass
HSRP Failover	Power off MLS1, ping from EMP-PC1	Gateway switches to MLS2, traffic resumes	✅ Pass
NAT Translation	Ping PUBLIC-PC from EMP-PC1, show ip nat translations	Inside → Outside translation entry created	✅ Pass
Internet Simulation	Ping 100.1.1.10 from EMP-PC1	Reply received via PAT	✅ Pass
EMPLOYEE-ACL — HTTPS to Server	EMP-PC1 → APP-SERVER TCP 443	Permitted by EMPLOYEE-ACL	✅ Pass
EMPLOYEE-ACL — HTTP to DMZ	EMP-PC1 → WEB-SERVER TCP 80	Permitted by EMPLOYEE-ACL	✅ Pass
EMPLOYEE-ACL — Block Management	EMP-PC1 → ADMIN-PC (ping)	Denied by EMPLOYEE-ACL deny rule	✅ Pass
DMZ-ACL — HTTP/HTTPS to Server Farm	WEB-SERVER → APP-SERVER TCP 80/443	Permitted by DMZ-ACL	✅ Pass
DMZ-ACL — Block Management	WEB-SERVER → ADMIN-PC (ping)	Denied by DMZ-ACL deny rule	✅ Pass
Syslog Reception	Generate interface event, check Syslog server	Log entry received with msec timestamp	✅ Pass
NTP Synchronization	show ntp status on MLS1	Clock synchronized, stratum 4	✅ Pass

## Design Decisions
Why Zero Trust?

Traditional perimeter models trust devices once inside the network, offering no protection against lateral movement after a compromise. Zero Trust requires explicit policy for every inter-zone communication — the only model that contains an attacker to the zone of initial access.

Why OSPF Instead of Static Routing?

With dual MLS switches and dual Border Router uplinks, static routing requires complex floating routes and manual maintenance. OSPF dynamically adapts to topology changes, recalculates paths on failure, and redistributes the default route automatically across all routers.

Why HSRP Instead of a Single Gateway?

A single physical gateway IP is a single point of failure. HSRP abstracts the gateway into a virtual address shared between two switches, with failover transparent to end devices and complete within seconds.

Why ACLs Applied on SVIs?

Inbound SVI application filters traffic at the Layer 2-to-Layer 3 boundary — the earliest point at which source, destination, protocol, and port are all visible — before any forwarding decision propagates traffic further into the network.

Why a Dedicated Management VLAN?

A shared management VLAN allows any zone with network reachability to attempt access to device interfaces. A dedicated Management VLAN combined with explicit inbound ACL denies creates an isolated administrative plane aligned with NIST and CIS separation recommendations.

Why a DMZ?

The DMZ contains public-facing services in a semi-trusted intermediate zone. A compromised WEB-SERVER is contained within VLAN 40 — ACL policy blocks all pivot attempts toward Management and Employee zones.

Why NAT Overload (PAT)?

Internal RFC 1918 addresses are non-routable on the public internet. PAT translates all outbound sessions to a single public IP using unique port numbers, preserving internal addressing confidentiality while enabling internet access for all zones.

Why Syslog in the Management Zone?

Placing the Syslog server inside the ACL-isolated Management VLAN ensures logs cannot be reached or tampered with even if Employee or DMZ zones are compromised — a fundamental requirement for forensic integrity and compliance.

Why NTP?

Without clock synchronization, cross-device log correlation is impossible. NTP with millisecond precision enables accurate event timelines across every device, making security incident analysis reliable.

Why Port Security With Sticky MAC?

Sticky learning automatically saves the first MAC seen on each port, providing rogue device prevention without requiring pre-configured MAC inventories. Any subsequent device triggers an immediate shutdown violation.

Why Rapid PVST+?

Standard STP takes 30–50 seconds to converge. Rapid PVST+ converges in 1–2 seconds — essential in a network where HSRP failover completes in seconds. Slow STP would block the recovered path and extend outages unnecessarily.

Why /30 Subnets for Routed Links?

A /30 provides exactly two usable addresses for a point-to-point link. Larger subnets waste address space and unnecessarily inflate broadcast domains on two-endpoint interfaces.


## Tools Used
Tool	Version	Purpose
### Cisco Packet Tracer	8.x	Network simulation and validation
Cisco IOS (Multilayer Switch)	12.x / 15.x	Layer 2/3 switching, VLAN, OSPF, HSRP, ACL
Cisco IOS (Router)	15.x	NAT, OSPF, NTP, routing
Cisco 3560 Multilayer Switch	Simulated	Distribution layer with Layer 3 capabilities
Cisco 2960 Switch	Simulated	Access layer switching
Cisco 2911 Router	Simulated	Border Router and ISP Router

## Packet Tracer Limitations
Limitation	Impact on This Project
EtherChannel negotiation bug on 3560 model	LACP EtherChannel between MLS1 and MLS2 was designed (Gi0/2–Gi0/3) but could not be implemented due to a Packet Tracer 3560 simulation bug. In production, channel-group 1 mode active on both switches aggregates both links into Port-Channel 1, verified with show etherchannel summary.
Stateless ACL simulation	Packet Tracer does not fully honour the established keyword. A production implementation would use a stateful firewall (ASA/Palo Alto) to enforce server-to-client session return policy.
RSA key size	Packet Tracer limits RSA generation to an interactive prompt with a maximum of 1024 bits. Production environments use crypto key generate rsa modulus 2048.
SNMP polling	Active SNMP polling from a simulated NMS cannot be demonstrated within Packet Tracer.

## Future Improvements
Enhancement	Description	Production Benefit
802.1X Port Authentication	IEEE 802.1X with RADIUS for port-level identity verification	Replaces MAC-based port security with credential-based access control
RADIUS / TACACS+	External AAA server replacing local database	Centralized identity management, per-command authorization, full accounting
IP SLA + HSRP Tracking	HSRP priority decrement driven by IP SLA upstream probes	Failover triggered by reachability loss, not just interface state
DHCP Snooping + Dynamic ARP Inspection	Enable on all access VLANs	Blocks rogue DHCP servers and prevents ARP spoofing at Layer 2
Stateful Firewall (ASA)	ASA between Border Router and MLS distribution layer	True stateful inspection, connection tracking, advanced zone policy
SNMP v3	SNMPv3 with AuthPriv security model on all devices	Authenticated and encrypted SNMP, eliminates community string exposure
EtherChannel (LACP)	Bundle MLS1–MLS2 inter-switch links as Port-Channel 1	Eliminates STP blocking on redundant links, doubles inter-switch bandwidth
Site-to-Site VPN	IPsec tunnel from Border Router to remote site or cloud	Encrypted WAN connectivity for branch office or cloud workload integration

## Key Learning Outcomes
Concept	Demonstration in This Project
Network Segmentation	Four isolated security zones with independent Layer 2 domains and controlled Layer 3 boundaries
VLAN Design	Zone-to-VLAN mapping, trunk configuration with explicit allow-lists, unused native VLAN hardening
Layer 3 Switching	Multilayer SVI configuration, ip routing, inter-VLAN forwarding
Dynamic Routing	OSPF single-area with explicit Router IDs, passive interfaces, and external default route distribution
Gateway Redundancy	HSRP virtual IP design, priority-based election, preemption behaviour
Access Control	Named extended ACL design from policy matrix, inbound SVI application, inline remark documentation
NAT/PAT	RFC 1918 to public IP translation, dual inside interface NAT, PAT overload
Management Security	SSH v2, AAA local authentication and authorization, VTY hardening
STP Optimization	Rapid PVST+ convergence, deterministic root bridge election, PortFast and BPDU Guard
Port Security	Sticky MAC learning, violation policy, rogue device prevention
Centralized Monitoring	Syslog with timestamp precision, NTP hierarchy, management zone isolation
Zero Trust Principles	Explicit-deny inter-zone policy, least privilege ACL design, secure management plane, assume breach monitoring
WAN Simulation	Serial link addressing, static default route, ISP return routing
Documentation	Physical connection tables, IP addressing plans, ACL trust matrices, design decision rationale

## Conclusion

This project delivers a complete, multi-technology enterprise network simulation built on Zero Trust principles — from VLAN boundary definitions and SVI-applied ACLs through HSRP preemption, OSPF passive interfaces, dual inside NAT, AAA-enforced management access, and centralized Syslog monitoring. Every technology choice reflects a deliberate engineering decision with a clear security or availability rationale, and the implementation maps directly to the patterns used in production Cisco enterprise campus and branch network designs.

Built with Cisco Packet Tracer | Cisco IOS | Enterprise Zero Trust Architecture Principles
