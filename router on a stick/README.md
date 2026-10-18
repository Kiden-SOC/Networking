# Inter-VLAN Routing (Router-on-a-Stick) with Dynamic DHCP Addressing

## Project Overview
This project demonstrates a scalable **Inter-VLAN routing** architecture using the **Router-on-a-Stick (ROAS)** design across a multi-switch corporate topology. To optimize network operations and administrative overhead, the central router is configured as a native **Cisco IOS DHCP Server**, dynamically provisioning IP configurations to endpoints across segregated department subnets.

## Network Architecture & Design Constraints
* **Core Topology:** 1 Central Cisco Router, 3 Interconnected Layer 2 Switches, and end-user host machines segmented by function.
* **Segmentation & Branding:** Broadcast domains split into **VLAN 10 (HR)** and **VLAN 20 (IT)**.
* **Encapsulation Protocol:** IEEE 802.1Q trunking links across switches and the router uplink.
* **IP Allocation:** Automated via localized DHCP pools with exclusion ranges to protect static gateway boundaries.

### IP Addressing & Trunking Schema

| VLAN | Subnet / Network | Department | Router Subinterface | Default Gateway | DHCP Pool Scope |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | 192.168.10.0/24 | HR (Human Resources) | Gig0/0.10 | 192.168.10.1 | 192.168.10.11 - .254 |
| **VLAN 20** | 192.168.20.0/24 | IT (Information Tech) | Gig0/0.20 | 192.168.20.1 | 192.168.20.11 - .254 |

---

## Core Configuration Highlights

### 1. Cisco IOS DHCP Pool Configurations
Configured on the central router to automatically assign IP addresses, subnet masks, and default gateways while excluding the first 10 addresses for network infrastructure safety:

```cisco
! Exclude static infrastructure addresses from allocation
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10

! DHCP Pool Definition for VLAN 10 (HR)
ip dhcp pool HR_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

! DHCP Pool Definition for VLAN 20 (IT)
ip dhcp pool IT_POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
```

### 2. Router Subinterface Setup (802.1Q Encapsulation)
```cisco
interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.10
 description --- Gateway for HR VLAN 10 ---
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 description --- Gateway for IT VLAN 20 ---
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

### 3. Switch-to-Switch & Switch-to-Router Trunking
To maintain VLAN visibility across all three switches, all interconnecting links are set to explicitly forward multi-VLAN tagged traffic:

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

---

## Verification & Testing Documentation
To ensure network functionality, the following verification commands were executed:

* **DHCP Lease Binding Inspection:** Checked active IP assignments via `show ip dhcp binding` to verify endpoints successfully received operational parameters.
* **Routing Table Status:** Executed `show ip route` to confirm directly connected subinterfaces populate the routing table natively.
* **Cross-VLAN Connectivity:** Verified end-to-end communication via ICMP echo requests (pings) initiated from HR endpoints (VLAN 10) directly reaching IT endpoints (VLAN 20).

---

## Skills & Technologies Demonstrated
* **IP Infrastructure Services:** Cisco IOS DHCP Server Configuration, IP Address Exclusions
* **Routing Architectures:** Router-on-a-Stick, Layer 3 Logical Subinterfaces
* **Switching & Layer 2 Protocols:** IEEE 802.1Q Trunking, Multi-Switch VLAN Propagation
* **Simulation Platforms:** Cisco Packet Tracer Framework
