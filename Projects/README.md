# NexaTech Solutions — Enterprise Network Simulation

> A multi-site enterprise network built in Cisco Packet Tracer, covering Layer 2 switching, inter-VLAN routing, OSPF multi-area, NAT/PAT, gateway redundancy (HSRP), and a full internal server infrastructure across HQ and two branch offices.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Topology](#topology)
- [IP Addressing](#ip-addressing)
- [VLAN Design](#vlan-design)
- [Features Implemented](#features-implemented)
- [Server Infrastructure](#server-infrastructure)
- [Security Implementation](#security-implementation)
- [Redundancy Design](#redundancy-design)
- [WAN & Routing](#wan--routing)
- [Test Results](#test-results)
- [Design Decisions](#design-decisions)
- [Tools Used](#tools-used)
- [Packet Tracer Limitations](#packet-tracer-limitations)

---

## Project Overview

NexaTech Solutions is a simulated enterprise network designed to reflect real-world corporate infrastructure. The network spans one headquarters site and two remote branch offices, connected via WAN links and managed through centralized HQ services.

| Attribute | Detail |
|---|---|
| Tool | Cisco Packet Tracer |
| Sites | HQ + Branch 1 + Branch 2 |
| Routers | 4 (HQ Primary, HQ Alternate/HSRP, Branch1, Branch2) + ISP |
| Switches | 9 (HQ Core, HR, Finance, IT, Sales, Management, Server, Branch×2) |
| Internal Servers | 5 (DHCP/DNS, HTTP/FTP, Email, Syslog, TFTP/NTP) |
| VLANs | 7 (HR, Finance, IT, Sales, Guest, Management, Servers) |
| Routing Protocol | OSPF Multi-Area (Area 0/1/2) |

---

## Topology

```
                          [ISP Router]
                         200.1.1.1/24
                         /           \
               10.0.0.16/30       10.0.0.20/30
                    /                     \
          [HQ Router]             [HQ Alternate Router]
          (Active - HSRP)         (Standby - HSRP)
                    \                     /
                     \                   /
                   [HQ Core Switch - Root Bridge]
                    |     |     |     |     |
                   HR  Finance  IT  Sales  Servers
                  SW    SW     SW    SW      SW
                                          (DHCP/DNS, HTTP/FTP,
                                           Email, Syslog, TFTP)

          10.0.0.0/30                  10.0.0.8/30
          10.0.0.4/30                  10.0.0.12/30
               |                            |
         [Branch1 Router]            [Branch2 Router]
               |                            |
         [Branch1 Switch]            [Branch2 Switch]
          192.168.110.0/24            192.168.120.0/24
```

---

## IP Addressing

### HQ VLANs (Virtual IPs — HSRP)

| VLAN | Name | Subnet | Virtual Gateway (HSRP) | HQ Primary | HQ Alternate |
|---|---|---|---|---|---|
| 10 | HR | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.2 | 192.168.10.3 |
| 20 | Finance | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.2 | 192.168.20.3 |
| 30 | IT | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.2 | 192.168.30.3 |
| 40 | Sales | 192.168.40.0/24 | 192.168.40.1 | 192.168.40.2 | 192.168.40.3 |
| 50 | Guest | 192.168.50.0/24 | 192.168.50.1 | 192.168.50.2 | 192.168.50.3 |
| 99 | Management | 192.168.99.0/24 | 192.168.99.1 | 192.168.99.2 | 192.168.99.3 |
| 100 | Servers | 192.168.100.0/24 | 192.168.100.1 | 192.168.100.2 | 192.168.100.3 |

### Branch Networks

| Site | Subnet | Gateway |
|---|---|---|
| Branch 1 | 192.168.110.0/24 | 192.168.110.1 |
| Branch 2 | 192.168.120.0/24 | 192.168.120.1 |

### WAN Links (/30 Point-to-Point)

| Link | Network | HQ End | Remote End |
|---|---|---|---|
| HQ Router ↔ Branch1 | 10.0.0.0/30 | 10.0.0.1 | 10.0.0.2 |
| HQ Alternate ↔ Branch1 | 10.0.0.4/30 | 10.0.0.5 | 10.0.0.6 |
| HQ Router ↔ Branch2 | 10.0.0.8/30 | 10.0.0.9 | 10.0.0.10 |
| HQ Alternate ↔ Branch2 | 10.0.0.12/30 | 10.0.0.13 | 10.0.0.14 |
| HQ Router ↔ ISP | 10.0.0.16/30 | 10.0.0.17 | 10.0.0.18 |
| HQ Alternate ↔ ISP | 10.0.0.20/30 | 10.0.0.21 | 10.0.0.22 |

### Switch Management IPs (VLAN 99)

| Device | IP | Default Gateway |
|---|---|---|
| Core Switch | 192.168.99.9 | 192.168.99.1 |
| HR Switch | 192.168.99.3 | 192.168.99.1 |
| Finance Switch | 192.168.99.4 | 192.168.99.1 |
| IT Switch | 192.168.99.5 | 192.168.99.1 |
| Sales Switch | 192.168.99.6 | 192.168.99.1 |
| Management Switch | 192.168.99.7 | 192.168.99.1 |
| Server Switch | 192.168.99.8 | 192.168.99.1 |
| Branch1 Switch | 192.168.110.2 | 192.168.110.1 |
| Branch2 Switch | 192.168.120.2 | 192.168.120.1 |

---

## VLAN Design

| VLAN ID | Name | Purpose | Access |
|---|---|---|---|
| 10 | HR | Human Resources department PCs | Internal only |
| 20 | Finance | Finance department PCs | Internal only |
| 30 | IT | IT department PCs + Admin | Internal only |
| 40 | Sales | Sales department PCs | Internal only |
| 50 | Guest | Wireless guest devices via AP | Internet only (ACL blocked from internal) |
| 99 | Management | SSH management of all network devices | Admin PC only (ACL enforced) |
| 100 | Servers | Internal server infrastructure | All internal VLANs |

---

## Features Implemented

| # | Feature | Status | Notes |
|---|---|---|---|
| 1 | Base device hardening | ✅ Done | Hostname, enable secret, console pw, banner, service password-encryption |
| 2 | Wireless Guest AP | ✅ Done | WPA2-PSK, AES, SSID: GuestWifi on VLAN 50, static IP guest clients |
| 3 | VLANs (7 total) | ✅ Done | Created and named on Core Switch |
| 4 | Trunk links | ✅ Done | Core Switch to all dept switches and both HQ routers |
| 5 | VTP | ✅ Done | Core = Server, dept switches = Client, domain: NEXATECH |
| 6 | Router-on-a-Stick | ✅ Done | 7 subinterfaces on HQ Router with dot1Q encapsulation |
| 7 | Management VLAN SVIs | ✅ Done | All 9 switches have VLAN 99 IP + default gateway |
| 8 | DHCP + DNS | ✅ Done | HQ VLAN DHCP pools on DHCP Server, Branch DHCP pools on Branch Routers, DNS records for enterprise services |
| 9 | HTTP + FTP Server | ✅ Done | Intranet page, 5 dept files uploaded via FTP from PCs |
| 10 | Email Server | ✅ Done | SMTP + POP3, nexatech.com domain, 9 user accounts |
| 11 | OSPF Multi-Area | ✅ Done | Area 0 (HQ), Area 1 (Branch1), Area 2 (Branch2), HQ Primary and HQ Alternate act as ABRs |
| 12 | Branch LAN + DHCP | ✅ Done | Local DHCP on branch routers, DNS points to HQ server |
| 13 | NAT/PAT | ✅ Done | PAT on both HQ routers, default route redistributed via OSPF |
| 14 | HSRP | ✅ Done | Active/Standby on all 7 VLANs, preempt enabled |
| 15 | SSH | ✅ Done | RSA v2, privilege 15, exec-timeout, VTY on all 10+ devices |
| 16 | ACL — Guest isolation | ✅ Done | ACL 110: Guest VLAN blocked from all internal subnets |
| 17 | ACL — SSH restriction | ✅ Done | ACL 10: Only admin PC (192.168.99.10) can SSH to switches |
| 18 | Port Security | ✅ Done | Sticky MAC, max 1, shutdown violation on access ports |
| 19 | Syslog | ✅ Done | Centralized logging to 192.168.100.40 on routers and Core Switch |
| 20 | NTP | ✅ Done | All devices sync to 192.168.100.50 |
| 21 | TFTP Backup | ✅ Done | Running-config backed up from all routers and Core Switch |
| 22 | STP — Root Bridge | ✅ Done | HQ Core Switch configured as Root Bridge for all VLANs |
| 23 | DHCP Snooping | ⚠️ Skipped | Planned — deferred due to Packet Tracer simulation instability |
| 24 | EtherChannel (LACP) | ⚠️ Skipped | Planned between Core and Server Switch — deferred due to PT bug |

---

## Server Infrastructure

| Server | IP | Services | Purpose |
|---|---|---|---|
| DHCP + DNS | 192.168.100.10 | DHCP, DNS | IP assignment for HQ VLANs and enterprise-wide DNS resolution |
| HTTP + FTP | 192.168.100.20 | HTTP, FTP | Intranet website + departmental file sharing |
| Email | 192.168.100.30 | SMTP, POP3 | Internal email for all departments (nexatech.com) |
| Syslog | 192.168.100.40 | Syslog | Centralized log monitoring from routers and switches |
| TFTP + NTP | 192.168.100.50 | TFTP, NTP | Config backup + time synchronization |
| External HTTP | 200.1.1.2 | HTTP | Simulates public internet (ISP side) |

### DNS Records

| Hostname | IP |
|---|---|
| www.nexatech.com | 192.168.100.20 |
| intranet.nexatech.com | 192.168.100.20 |
| ftp.nexatech.com | 192.168.100.20 |
| mail.nexatech.com | 192.168.100.30 |

### FTP Department Files

| Department | File | Purpose |
|---|---|---|
| HR | employee_records.txt | Employee details |
| Finance | budget_report.txt | Budget and accounts |
| IT | network_backup.txt | Network configurations |
| Sales | sales_targets.txt | Sales reports |
| Management | strategic_plan.txt | Company plans |

---

## Security Implementation

### ACL 110 — Guest VLAN Isolation (on both HQ routers)

Guest VLAN devices use static IP addressing.

Guest users are denied access to all internal VLANs, management devices, branch networks and internal application servers while retaining Internet connectivity.

DNS access to 192.168.100.10 is permitted for name resolution.

### ACL 10 — SSH Management Restriction (on all switches)

Only the admin PC at 192.168.99.10 is permitted to SSH into any switch. All other hosts are denied on VTY lines.

### Additional Security Features

- **Port Security** — sticky MAC address learning, max 1 device per access port, shutdown on violation
- **SSH v2** — RSA 1024-bit keys, exec-timeout 5 minutes, local authentication only
- **Service password-encryption** — all plaintext passwords encrypted in running-config
- **Banner MOTD** — "Unauthorized Access Prohibited" on all devices
- **Management VLAN 99** — dedicated out-of-band management plane, isolated from user traffic

---

## Redundancy Design

### HSRP — Hot Standby Router Protocol

Both HQ routers share a virtual IP per VLAN. PCs use only the virtual IP as their default gateway. If the active router fails, the standby takes over within the hold timer without any reconfiguration on end devices.

| VLAN | Virtual IP (HSRP) | Active Router | Priority | Standby Router | Priority |
|---|---|---|---|---|---|
| 10 | 192.168.10.1 | HQ Primary | 110 | HQ Alternate | 100 |
| 20 | 192.168.20.1 | HQ Primary | 110 | HQ Alternate | 100 |
| 30–100 | (same pattern) | HQ Primary | 110 | HQ Alternate | 100 |

Preempt is enabled — HQ Primary automatically reclaims active role when it recovers.

### STP — Spanning Tree Protocol

HQ Core Switch configured as Root Bridge for all VLANs. Prevents switching loops in the redundant link topology.

---

## WAN & Routing

### OSPF Multi-Area Design

| Area | Devices | Networks |
|---|---|---|
| Area 0 (Backbone) | HQ Primary Router, HQ Alternate Router, HQ Internal Networks | 192.168.x.x/24, 10.0.0.16/30, 10.0.0.20/30 |
| Area 1 | Branch1 Router | 192.168.110.0/24, 10.0.0.0/30, 10.0.0.4/30 |
| Area 2 | Branch2 Router | 192.168.120.0/24, 10.0.0.8/30, 10.0.0.12/30 |

HQ Primary and Alternate act as ABRs (Area Border Routers). `default-information originate` on both HQ routers redistributes the default route into OSPF so branch offices reach the internet via HQ NAT.

### NAT/PAT

Both HQ routers perform NAT Overload (PAT). All private subnets (192.168.0.0/16) are translated to the public WAN interface IP before traffic exits to the ISP.

```
ip nat inside  → all subinterfaces (g0/0.10 through g0/0.100)
ip nat outside → WAN interface toward ISP
```

### Branch Architecture

Branch offices have no dedicated servers.

DHCP services are provided locally by the Branch Routers.

DNS, HTTP, FTP, Email, Syslog, NTP and TFTP services are hosted centrally at HQ.

Branch routers use `dns-server 192.168.100.10` in their DHCP pools to provide enterprise-wide name resolution.

Internet traffic from branch users is routed through HQ using OSPF and exits through NAT/PAT.

---

## Test Results

| Test | Method | Result |
|---|---|---|
| VLAN isolation | PC in HR pings Finance PC | ✅ Pass |
| Inter-VLAN routing | Cross-VLAN ping | ✅ Pass |
| DHCP assignment | Desktop → IP Config → DHCP | ✅ Pass (HQ VLANs and Branch Routers) |
| DNS resolution | Browser → www.nexatech.com | ✅ Pass |
| Intranet web access | Browser → http://www.nexatech.com | ✅ Pass |
| FTP upload | PC → ftp ftp.nexatech.com → put file | ✅ Pass |
| Email send/receive | HR1 → compose → Finance1 | ✅ Pass |
| Branch ↔ HQ ping | Branch1 PC → HR PC | ✅ Pass (OSPF) |
| Internet access | Internal PC → 200.1.1.2 | ✅ Pass (NAT) |
| Guest blocked | Guest PC → HR PC ping | ✅ Blocked (ACL 110) |
| Guest internet | Guest PC → 200.1.1.2 | ✅ Pass (ACL 110 permit any) |
| SSH access | Admin PC → ssh 192.168.99.9 | ✅ Pass |
| SSH blocked | Non-admin PC → SSH to switch | ✅ Blocked (ACL 10) |
| HSRP failover | Disable HQ Primary → test gateway | ✅ Standby takes over |
| Syslog | Disable/enable interface → check server | ✅ Logs appear |
| TFTP backup | copy running-config tftp | ✅ Pass |

---

## Design Decisions

**Why Router-on-a-Stick instead of a Layer 3 switch?**
Router-on-a-Stick was chosen to explicitly demonstrate dot1Q subinterface configuration and VLAN-aware routing. A Layer 3 switch would consolidate these, but the goal was to show understanding of the underlying mechanism. In a production environment with high inter-VLAN traffic, an L3 switch would be preferred for performance.

**Why OSPF Multi-Area instead of single-area or static routing?**
Single-area OSPF works but doesn't scale. With HQ and two branches, multi-area design reduces LSA flooding between sites and demonstrates ABR functionality. Static routing would require manual updates on every topology change — unsuitable for a growing enterprise.

**Why two HQ routers instead of one?**
A single router is a single point of failure for the entire HQ gateway. HSRP with two routers means any PC's default gateway remains reachable even if one router goes down. This is standard enterprise design for critical sites.

**Why a dedicated Management VLAN?**
Separating administrative traffic (SSH) from user traffic prevents unauthorized users on HR or Finance VLANs from attempting SSH access to network devices. Combined with ACL 10 on all VTY lines, only the designated admin PC can manage any switch in the network.

**Why NAT on both HQ routers?**
Each HQ router has its own WAN link to the ISP. If only one performed NAT, internet traffic would always traverse the standby router — creating asymmetric routing and a bottleneck. Both routers are capable NAT gateways, consistent with the redundancy design.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Cisco Packet Tracer | 8.x | Network simulation |
| Cisco IOS (simulated) | 15.x | Router and switch OS |

---

## Packet Tracer Limitations

The following features were planned but deferred due to known Packet Tracer simulation issues:

- **DHCP Snooping** — `ip dhcp snooping` commands applied but binding table behavior is inconsistent on PT reopen. Feature deferred due to Packet Tracer simulation limitations.
- **EtherChannel (LACP)** — Port-channel between Core Switch and Server Switch was planned. PT occasionally marks LACP bundles as `err-disabled` on file reload even with correct config. Feature deferred; configuration logic is identical to production IOS.

These are simulator constraints, not design gaps. Both features are standard IOS configurations that work on physical Cisco hardware and higher-fidelity simulators.

---

*Built as part of placement portfolio — demonstrating enterprise network design, security, and redundancy concepts at CCNA level.*

