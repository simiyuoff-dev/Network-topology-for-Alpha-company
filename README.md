# Alpha Enterprises — Enterprise Network Build
*Tool:* Cisco Packet Tracer  
*Scope:* Full enterprise network from scratch

# Overview
A complete three-tier hierarchical enterprise network built for 
a fictional company, Alpha Enterprises. Covers switching, routing, 
redundancy, services, WAN connectivity, and security hardening.

# Topology
uploaded on the files ,can open it via cisco packet tracer

# Addressing Scheme

# HQ Site — 10.10.0.0/18
Departments,VLANs,Subnets,Gateway (HSRP VIP)
Sales - vlan 10 - subnet 10.10.0.0/26 - gateway - 10.10.0.3 
IT - vlan 20 - subnet 10.10.0.64/27 - gateway - 10.10.0.67 
HR - vlan 30 - subnet 10.10.0.96/27 - gateway - 10.10.0.99
Finance - vlan 40 - subnet 10.10.0.128/27 - gateway - 10.10.0.131
Servers - vlan 50 - subnet 10.10.0.160/28 - gateway - 10.10.0.163
Management - vlan 99 - subnet 10.10.0.176/28 - gateway - 10.10.0.177 

# Branch Site — 10.10.64.0/18
Department,VLAN,Subnet,Gateway
Branch Sales - vlan 110 - subnet 10.10.64.0/28 - gateway - 10.10.64.1 |

# Point-to-Point WAN Links

Links and Subnet
HQ-ROUTER - ISP-ROUTER - 10.10.0.196/30 
HQ-ROUTER - BRANCH-ROUTER - 10.10.0.192/30 
CORE-SW1 - HQ-ROUTER -  10.10.0.200/30 

# Features Implemented

# Switching
- Three-tier hierarchical design: Core (3560), Distribution (2960), Access (2960)
- VLANs 10/20/30/40/50/99 across all switches
- 802.1Q trunking with native VLAN 99 on all inter-switch links
- Spanning Tree Protocol with CORE-SW1 elected as root bridge via priority manipulation
- EtherChannel (LACP) bundle between CORE-SW1 and CORE-SW2 — Po1(SU)

# Routing
- Inter-VLAN routing via SVIs on CORE-SW1 (ip routing)
- Static routing between HQ, branch, and ISP
- Default route propagation from HQ-ROUTER to CORE-SW1
- Serial WAN links with clock rate on DCE end

# Redundancy
- HSRP active/standby gateway redundancy on all five VLANs — CORE-SW1 active (priority 110), 
  CORE-SW2 standby (priority 90) with preempt

# Services
- DHCP server with per-VLAN pools and ip helper-address relay on CORE-SW1 SVIs
- DNS server with internal A records (*.alpha.local)
- FTP and HTTP servers in dedicated Server VLAN

#WAN and Internet
- Serial WAN link HQ ↔ Branch
- NAT/PAT on HQ-ROUTER — all 10.10.0.0/16 translated to serial interface IP via access-list overload

### Security
- Extended ACLs on VLAN 30 and 40 — HR/Finance cannot reach each other or Management VLAN
- Port security on access layer — sticky MAC, maximum 1, violation shutdown
- service password-encryption on all devices
- MOTD banner on all devices
- SSH-only remote access (vty transport input ssh)
- Native VLAN changed from default VLAN 1 to VLAN 99

# Verification Results
All tests passing from PC1 (Sales, 10.10.0.10):

Test with Result using ping command
Same-VLAN gateway  ok 4/4 
Cross-VLAN (IT, HR, Finance) ok 4/4 
Server farm  ok 4/4 
Branch site over WAN ok 4/4 
Simulated internet (8.8.8.8) ok 4/4 
DNS resolution (web.alpha.local) ok 4/4 
ACL — HR blocked from Finance ok 0/4 (correct) 
ACL — HR permitted to servers ok 3/4 (ARP timeout) 

# Troubleshooting Lessons Learned

*Swapped serial cable* — HQ-ROUTER's Serial0/3/0 and Serial0/3/1 were configured with IPs matching the wrong 
neighbors. Pings to the directly connected serial address failed even though interfaces showed up/up. 
Resolved using show cdp neighbors, which revealed the physical cable connected to BRANCH-ROUTER was actually 
on Serial0/3/0, opposite to what we assumed.

*Native VLAN mismatch flood* — Configuring trunk ports only on one end of inter-switch links causes 
CDP NATIVE_VLAN_MISMATCH errors and STP port blocking. Every switch-to-switch link requires matching trunk 
configuration on both ends. Fixed systematically by reading the mismatch messages — they identify 
exactly which local port and which neighbor port are mismatched.

*Wrong routed port on CORE-SW1* — Applied no switchport and ip address to GigabitEthernet0/1, but the cable 
to HQ-ROUTER was physically on FastEthernet0/4.show cdp neighbors revealed the true port before 
wasting further time on routing table issues.

*EtherChannel port suspension* — Port-channel interface must be configured with trunk settings 
before adding physical member ports. If physical ports are added first, their native VLAN (99) 
conflicts with Po1's default native VLAN (1), causing immediate suspension. Fix: configure 
interface port-channel 1 with full trunk settings first, then add members.

*DHCP relay* — DHCP broadcasts cannot cross 
router/Layer 3 boundaries. ip helper-address must be applied on every SVI interface that 
contains DHCP clients pointing to a server in a different VLAN.be applied on every SVI interface that 
contains DHCP clients pointing to a server 
in a different VLAN.
