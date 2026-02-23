# IT 381 – Assignment 3  
## Multi-Site Network Design Using VLANs and OSPF

---

## 1. Introduction

### What This Lab Is About

This lab simulates a real-world multi-site enterprise network using VLAN segmentation and OSPF dynamic routing.

The objective is to:

- Configure VLANs on switches
- Implement Router-on-a-Stick inter-VLAN routing
- Configure WAN point-to-point links between routers
- Establish OSPF Area 0 neighbor relationships
- Achieve full end-to-end connectivity across three separate sites

By the end of this lab, all PCs across HQ, Site-B, and Site-C should successfully communicate with each other.

---

### Skills This Lab Teaches

- VLAN creation and access port configuration  
- Trunk configuration using 802.1Q  
- Router subinterfaces (Router-on-a-Stick)  
- WAN point-to-point routing  
- OSPF configuration with different process IDs  
- OSPF neighbor verification  
- Routing table analysis  
- End-to-end network troubleshooting  

---

## 2. Network Topology Overview

This network consists of three sites:

- **HQ Site (R1 + SW1 + PC1)**
- **Site-B (R2 + SW2 + PC2)**
- **Site-C (R3 + SW3 + PC3)**

### WAN Links

- R1 ↔ R2 (10.12.1.0/30)
- R2 ↔ R3 (10.23.1.0/30)
- R1 ↔ R3 (10.13.1.0/30)

All routers participate in **OSPF Area 0**, forming dynamic neighbor relationships.

---

## 3. IP Addressing Table

### VLAN Networks

| VLAN | Network | Gateway |
|------|----------|----------|
| VLAN 100 | 142.100.64.0/24 | 142.100.64.1 |
| VLAN 200 | 142.100.65.0/24 | 142.100.65.1 |
| VLAN 300 | 142.100.66.0/24 | 142.100.66.1 |

---

### PCs

| Device | IP Address | Subnet Mask | Default Gateway |
|--------|------------|-------------|------------------|
| PC1 | 142.100.64.11 | 255.255.255.0 | 142.100.64.1 |
| PC2 | 142.100.65.10 | 255.255.255.0 | 142.100.65.1 |
| PC3 | 142.100.66.10 | 255.255.255.0 | 142.100.66.1 |

---

### WAN Links

| Link | Network |
|------|----------|
| R1–R2 | 10.12.1.0/30 |
| R2–R3 | 10.23.1.0/30 |
| R1–R3 | 10.13.1.0/30 |

---

### Loopbacks

| Router | Loopback |
|--------|----------|
| R1 | 142.1.64.254 |
| R2 | 142.1.65.254 |
| R3 | 142.1.66.254 |

---

## 4. Step-by-Step Configuration

---

### 4.1 Switch Configuration

#### VLAN Creation Example

vlan 100
name HQ-VLAN

#### Access Port Example
interface e0/1
switchport mode access
switchport access vlan 100

#### Trunk Configuration Example
interface e0/0
switchport mode trunk


### 4.2 Router Configuration

#### Router-on-a-Stick (R1 Example)
interface e0/0.100
encapsulation dot1Q 100
ip address 142.100.64.1 255.255.255.0


#### WAN Interface Example
interface e0/2
ip address 10.12.1.1 255.255.255.252
no shutdown

#### Loopback Example
interface loopback0
ip address 142.1.64.254 255.255.255.255


---

### 4.3 OSPF Configuration (Different Process IDs)

#### R1
router ospf 100
network 10.0.0.0 0.255.255.255 area 0
network 142.0.0.0 0.255.255.255 area 0
#### R2
router ospf 200
network 10.0.0.0 0.255.255.255 area 0
network 142.0.0.0 0.255.255.255 area 0

#### R3
router ospf 300
network 10.0.0.0 0.255.255.255 area 0
network 142.0.0.0 0.255.255.255 area 0


Note: OSPF process IDs are locally significant and do not need to match across routers.

---

## 5. Verification & Testing

---

### 5.1 Verify OSPF Neighbors
show ip ospf neighbor

All neighbors should show `FULL` state.

---

### 5.2 Verify OSPF Routes

show ip ospf neighbor

All neighbors should show `FULL` state.

---

### 5.2 Verify OSPF Routes
show ip route ospf

Remote networks should appear with `O` indicating OSPF-learned routes.

---

### 5.3 Verify Default Gateway (From PCs)

ping 142.100.64.1

Should return success.

---

### 5.4 End-to-End Testing

From PC1:
ping 142.100.65.10
ping 142.100.66.10

From PC2:
ping 142.100.64.11
ping 142.100.66.10
From PC3:
ping 142.100.64.11
ping 142.100.65.10

All pings should succeed.

---

### 5.5 Loopback Testing
ping 142.1.65.254
ping 142.1.66.254

Successful responses confirm full routing convergence.

---

## 6. Troubleshooting Section

### Problem 1 – OSPF Neighbors Not Forming

Cause: Incorrect network statements.  
Fix: Adjusted wildcard masks to properly advertise WAN interfaces.

---

### Problem 2 – PCs Could Not Reach Other Sites

Cause: `ip routing` was enabled on PC devices.  
Fix: Disabled `ip routing` and configured proper default gateways.

---

### Problem 3 – VLAN Traffic Not Passing

Cause: Trunk link misconfiguration.  
Fix: Verified trunk mode using:
show interfaces e0/0 switchport

Confirmed dot1Q trunk was operational.

---

## 7. Conclusion

This lab demonstrates how Layer 2 switching and Layer 3 routing interact to build scalable enterprise networks.

Key Takeaways:

- OSPF process IDs do not need to match across routers.
- VLAN and trunk misconfigurations can break routing even when OSPF is correct.
- Always verify interface status before troubleshooting routing.
- End-to-end testing confirms full network convergence.

Understanding protocol interaction across layers is more important than memorizing commands.

---

## Final Advice for Future IT-381 Students

- Use `show cdp neighbors` to verify physical connectivity first.
- Use `show ip interface brief` to confirm interfaces are up.
- Verify VLAN and trunk configuration before troubleshooting OSPF.
- Always test from PCs, not just routers.

Networking is about logical flow. Build it layer by layer.
