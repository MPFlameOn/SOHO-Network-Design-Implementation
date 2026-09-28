# SOHO Network Design & Implementation

**Enterprise Network Project**

---

## 1. Case Study

XYZ company is a fast-growing company in Eastern Australia with more than 2 million customers globally. The company deals with selling and buying of food items, which are basically operated from the headquarters. The company is intending to open a branch near the local village Bonalbo. Thus, the company requires young IT graduates  to design the  network for the branch. The network is intended to operate separately from the HQ network.

## 2. Requirements

- One Cisco router and one Cisco switch
- Three departments: **Admin/IT**, **Finance/HR**, **Customer Service/Reception**
- Each department on a separate VLAN
- Each department has its own wireless network
- Hosts obtain IPv4 addresses automatically (DHCP)
- All departments can communicate with each other
- ISP-assigned base network: `192.168.1.0`

## 3. Technologies Implemented

1. Simple network topology (1 router, 1 switch)
2. Correct cabling (straight-through / crossover / trunk as needed)
3. VLAN creation and port assignment
4. Subnetting and IP addressing
5. Inter-VLAN routing (router-on-a-stick)
6. DHCP server configuration (router as DHCP server)
7. WLAN / access point configuration
8. Host device configuration
9. Testing and verification

---

## 4. Network Topology

![Network topology](screenshots/topology.png)

| Device | Role |
|---|---|
| Router0 | Router-on-a-stick, DHCP server |
| Switch (multilayer/L2) | VLAN trunking, access ports |
| Access Point 0/1/2 | Wireless per department |
| PC0, PC1, PC2 | Wired department hosts |
| Printer0/1/2 | Department printers |
| Laptop0/1/2, Smartphone0/1/2, Tablet PC0 | Wireless department clients |

---

## 5. IP Addressing Plan

Base network: `192.168.1.0/24`, subnetted into three `/26` blocks (2 borrowed bits → 4 subnets of 64 hosts each, 3 used).

| Department | VLAN | Subnet | Usable Range | Gateway (Router subinterface) | Broadcast |
|---|---|---|---|---|---|
| Admin/IT | 10 | 192.168.1.0/26 | .1 – .62 | 192.168.1.1 | 192.168.1.63 |
| Finance/HR | 20 | 192.168.1.64/26 | .65 – .126 | 192.168.1.65 | 192.168.1.127 |
| CS/Reception | 30 | 192.168.1.128/26 | .129 – .190 | 192.168.1.129 | 192.168.1.191 |


---

## 6. Configuration Summary

### 6.1 Switch — VLAN Creation & Access Ports

Created three VLANs and assigned access ports per department: Fa0/2–4 to
VLAN 10 (Admin/IT), Fa0/5–7 to VLAN 20 (Finance/HR), and Fa0/8–10 to VLAN 30
(CS/Reception). Packet Tracer auto-creates each VLAN the first time it's
referenced in a `switchport access vlan` command.

```
Switch>en
Switch#conf t
Switch(config)#int range fa0/2-4
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#exit
Switch(config)#int range fa0/5-7
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 20
Switch(config-if-range)#exit
Switch(config)#int range fa0/8-10
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 30
Switch(config-if-range)#exit
Switch(config)#do wr
```

### 6.2 Switch — Trunk Configuration

Set Fa0/1 (the uplink to the router) to trunk mode so all three VLANs' traffic
can pass to the router for inter-VLAN routing.

```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#int fa0/1
Switch(config-if)#switchport mode trunk
Switch(config-if)#do wr
```

### 6.3 Router — Subinterfaces (Router-on-a-Stick)

Configured one subinterface per VLAN on the router's Gi0/0 link, each with
802.1Q encapsulation matching its VLAN ID and the gateway address for that
department's subnet.

```
Router>en
Router#conf t
Router(config)#int g0/0
Router(config-if)#no sh
Router(config-if)#exit
Router(config)#int g0/0.10
Router(config-subif)#encapsulation dot1Q 10
Router(config-subif)#ip address 192.168.1.1 255.255.255.192
Router(config-subif)#exit
Router(config)#int g0/0.20
Router(config-subif)#encapsulation dot1Q 20
Router(config-subif)#ip address 192.168.1.65 255.255.255.192
Router(config-subif)#exit
Router(config)#int g0/0.30
Router(config-subif)#encapsulation dot1Q 30
Router(config-subif)#ip address 192.168.1.129 255.255.255.192
Router(config-subif)#do wr
```

### 6.4 Router — DHCP Pools

Configured one DHCP pool per department, scoped to its own /26 subnet, with
the router subinterface as both default gateway and DNS server.

```
Router(config)#service dhcp
Router(config)#ip dhcp pool Admin-Pool
Router(dhcp-config)#network 192.168.1.0 255.255.255.192
Router(dhcp-config)#default-router 192.168.1.1
Router(dhcp-config)#dns-server 192.168.1.1
Router(dhcp-config)#domain-name Admin.com
Router(dhcp-config)#exit
Router(config)#ip dhcp pool Finance-Pool
Router(dhcp-config)#network 192.168.1.64 255.255.255.192
Router(dhcp-config)#default-router 192.168.1.65
Router(dhcp-config)#dns-server 192.168.1.65
Router(dhcp-config)#domain-name Finance.com
Router(dhcp-config)#exit
Router(config)#ip dhcp pool CS-Pool
Router(dhcp-config)#network 192.168.1.128 255.255.255.192
Router(dhcp-config)#default-router 192.168.1.129
Router(dhcp-config)#dns-server 192.168.1.129
Router(dhcp-config)#domain-name CS.com
Router(dhcp-config)#exit
Router(config)#do wr
```

### 6.5 Access Points

Each department's AP was configured with its own SSID, mapped to its VLAN via
the corresponding access port on the switch.

**Access Point0 — Admin/IT (VLAN 10)**

![Access Point0 config](screenshots/accesspoint0.png)

**Access Point1 — Finance/HR (VLAN 20)**

![Access Point1 config](screenshots/accesspoint1.png)

**Access Point2 — CS/Reception (VLAN 30)**

![Access Point2 config](screenshots/accesspoint2.png)

### 6.6 End Device Verification

Each PC and printer was set to obtain an IPv4 address automatically. The
screenshots below confirm every device received an address from its
department's DHCP pool.

| Device | Department | Verification |
|---|---|---|
| PC0 | Admin/IT | ![PC0 ipconfig](screenshots/pc0.png) |
| PC1 | Finance/HR | ![PC1 ipconfig](screenshots/pc1.png) |
| PC2 | CS/Reception | ![PC2 ipconfig](screenshots/pc2.png) |
| Printer0 | Admin/IT | ![Printer0 config](screenshots/printer0.png) |
| Printer1 | Finance/HR | ![Printer1 config](screenshots/printer1.png) |
| Printer2 | CS/Reception | ![Printer2 config](screenshots/printer2.png) |

All devices obtained IPv4 addresses automatically within their department's
subnet, confirming DHCP scoping is correct per VLAN.

## 7. Testing & Verification

The following tests confirm the network meets the case study's core
requirement: devices in all departments can communicate with each other,
while remaining logically separated by VLAN.

| # | Test | From | To | Expected Result | Actual Result |
|---|---|---|---|---|---|
| 1 | Same-VLAN ping | PC0 (Admin) | Printer0 (Admin) | Success | Success |
| 2 | Same-VLAN ping | PC1 (Finance) | Printer1 (Finance) | Success | Success |
| 3 | Cross-VLAN ping | PC0 (Admin) | PC1 (Finance) | Success (via router-on-a-stick) | Success |
| 4 | Cross-VLAN ping | PC2 (CS/Reception) | PC1 (Finance) | Success | Success |
| 5 | Cross-VLAN ping | PC0 (Admin) | PC2 (CS/Reception) | Success | Success |
| 6 | Wireless DHCP lease | Laptop0 (Admin, wireless) | — | Obtains 192.168.1.x address | |
| 7 | Wireless DHCP lease | Smartphone1 (Finance, wireless) | — | Obtains 192.168.1.64–126 address | |
| 8 | Wireless-to-wired ping | Smartphone2 (CS, wireless) | PC2 (CS, wired) | Success (same VLAN) | |
| 9 | Cross-VLAN traceroute | PC0 (Admin) | PC2 (CS/Reception) | Path routes through 192.168.1.1 | Success |

## 8. Key Learnings

- Router-on-a-stick requires the subinterface encapsulation VLAN ID to
  exactly match the VLAN tag the switch sends over the trunk — a mismatch
  here silently breaks inter-VLAN routing with no error message.
- Subnetting a /24 into three /26s made VLAN and DHCP scoping simple, but in
  a real deployment I'd size subnets closer to actual headcount per
  department rather than splitting evenly, to avoid wasting address space.
- Trunk ports must be configured on *both* the switch-to-router link; forgetting
  this on either end leaves VLANs unable to reach the router at all.
- Wireless client behavior in Packet Tracer (SSID, security mode, VLAN
  mapping via the access port) mirrors real AP configuration closely enough
  to be a genuinely useful proxy for hands-on wireless setup.

---

## 9. Summary

This project implements a segmented SOHO network for XYZ company's Bonalbo
branch, meeting all seven requirements from the case study: a single Cisco
router and switch form the core infrastructure; three departments (Admin/IT,
Finance/HR, CS/Reception) are isolated into separate VLANs with independent
/26 subnets; router-on-a-stick provides inter-VLAN routing so departments can
communicate despite the isolation; the router's DHCP server automatically
assigns addresses to all hosts; and each department has its own wireless
network via a dedicated access point. Testing via same-VLAN and cross-VLAN
pings, wireless DHCP leases, and a cross-VLAN traceroute confirms the design
works as intended.

---
