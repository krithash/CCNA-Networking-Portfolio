# CCNA Cisco Packet Tracer Labs

Hands-on CCNA labs using Cisco Packet Tracer to build practical understanding of networking concepts.

---

## Week 1 – Switching Basics

### Topics

* Basic connectivity (PC ↔ PC)
* LAN setup using switches
* Multi-switch topologies
* Telnet and SSH access

### Key Learnings

* Connectivity verification using ping
* Switch forwarding using MAC address tables
* Difference between Telnet (insecure) and SSH (secure)

---

## Files

* Switching_Topologies_Basic_Connectivity_Week1.pkt
* Telnet_SSH_Switch_Config_Week1.pkt

---

## Week 2 – VTP & VLAN Management

### Topics

* VLAN Trunking Protocol (VTP)
* VTP Server mode
* VTP Client mode
* VTP Transparent mode
* Trunk link configuration
* VLAN propagation across switches
* Access port VLAN assignment

### Key Learnings

* VTP simplifies VLAN management in multi-switch networks
* Server mode distributes VLAN updates
* Client mode receives synchronized VLAN data
* Transparent mode forwards VTP advertisements while maintaining local VLAN control
* Trunk links are essential for VTP communication

---

## Files

* VTP.pkt

---

## Week 3 – DTP, STP & EtherChannel

### Topics

* Dynamic Trunking Protocol (Auto / Desirable)
* Manual trunk configuration
* Spanning Tree Protocol (STP)
* Root Bridge Priority configuration
* STP port roles (Root, Designated, Alternate)
* EtherChannel configuration using LACP
* Link aggregation between switches

### Key Learnings

* DTP automates trunk formation between switches
* STP prevents loops in redundant Layer 2 networks
* Root bridge determines the path for traffic flow
* Non-root switches block redundant links to avoid loops
* EtherChannel bundles multiple links into one logical link
* LACP ensures dynamic and reliable link aggregation
* EtherChannel removes STP blocking on bundled links and improves bandwidth utilization

---

## Files

* DTP_STP_EtherChannel_Lab_Week3.pkt

---

## Week 4 – Routing Protocols

### Topics

* Static Routing
* RIP Configuration
* OSPF Configuration
* EIGRP Configuration
* Serial WAN connections between routers
* Route verification using routing tables
* End-to-end connectivity testing

### Key Learnings

* Static routing manually defines packet forwarding paths
* RIP dynamically shares routes using hop count metric
* OSPF uses areas and wildcard masks for efficient routing
* EIGRP provides fast convergence and advanced route selection
* Routing tables help verify learned and connected networks
* Different routing protocols are identified using:

  * S → Static
  * R → RIP
  * O → OSPF
  * D → EIGRP
* Successful ping responses confirm inter-network communication

---

## Files

* Static_Dynamic_Routing.pkt

---

## Week 5 – DHCP, OSPF & NTP Configuration

### Topics

* Dynamic Host Configuration Protocol (DHCP)
* OSPF multi-router routing
* Network Time Protocol (NTP)
* Multi-router WAN topology
* Automatic IP assignment
* Time synchronization between routers
* End-to-end connectivity verification

### Key Learnings

* DHCP automates IP address assignment for end devices
* OSPF dynamically exchanges routes between interconnected routers
* OSPF neighbor relationships enable scalable routing
* NTP synchronizes time across all routers in the network
* Proper routing must exist before NTP synchronization works
* WAN links enable communication between multiple LAN networks
* Connectivity can be verified using ping and routing tables

---

## Week 6 – NAT, PAT, ACL & Port Security

### Topics
- Dynamic NAT using Public IP Pool
- PAT (NAT Overload) using Public IP Pool
- Access Control Lists (ACLs)
- Switch Port Security
- NAT Translation Verification
- End-to-End Connectivity Testing

### Key Learnings
- Dynamic NAT provides one-to-one translation using a pool of public IP addresses
- PAT (NAT Overload) allows multiple hosts to share public IP addresses through port/identifier translation
- ACLs can be used to identify traffic that should be translated
- NAT translation tables help visualize address translation processes
- Port Security enhances access-layer security by limiting device access on switch ports
- Sticky MAC addresses allow switches to dynamically learn and secure connected devices
- Verification commands are essential for validating NAT and security configurations

### Files
- NAT_PortSecurity.pkt

## Files

* DHCP_NTP.pkt

This repository will be updated as I progress through my CCNA journey.
