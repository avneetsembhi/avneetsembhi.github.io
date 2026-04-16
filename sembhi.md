# FitLife Gym — Network Infrastructure Technical Documentation

**NSA630 Capstone Project**  
**MITT (Manitoba Institute of Trades and Technology)**  
**Date: April 2026**

---

## 1. Executive Summary

This document describes the implementation of the FitLife Gym network. The network is designed to provide **secure access, redundancy, and proper segmentation** using VLANs, HSRP, and EtherChannel.

The topology includes:
- 2 Routers (HSRP)
- 3 Switches (Layer 2)
- Multiple VLANs for separation
- Internal + DMZ servers
- Wireless access (Staff + Guest)

---

## 2. Network Topology Overview

The network is built using a **collapsed core design**:

- **R-A and R-B** → Gateway routers with HSRP
- **S-A** → Core switch (main switching point)
- **S-B** → Distribution switch
- **S-C** → Access switch (end devices)

### Key Features:
- HSRP between R-A and R-B
- EtherChannel between all switches
- VLAN-based segmentation
- DHCP + DNS centralized server

---

## 3. VLAN Design (Based on Diagram)

Each colored section in your diagram represents a VLAN:

| VLAN | Name      | Network         | Devices |
|------|----------|-----------------|--------|
| 10   | SERVER   | 10.10.10.0/24   | DB1, HV1 |
| 15   | DMZ      | 10.10.15.0/24   | WEB1, MAIL1 |
| 20   | STAFF    | 10.10.20.0/24   | Front Desk PC |
| 30   | IT ADMIN | 10.10.30.0/24   | Management PC |
| 40   | GUEST    | 10.10.40.0/24   | Access Point |
| 50   | MGMT     | 10.10.50.0/24   | Admin PC |

---

## 4. IP Addressing (From Diagram)

### Routers

| Device | Interface | IP |
|--------|----------|----|
| R-A | WAN | 10.128.250.193 |
| R-B | WAN | 10.128.250.195 |

### Management Network

| Device | IP |
|--------|----|
| PC (MGMT) | 10.10.50.250 |
| S-A | 10.10.50.5 |
| S-B | 10.10.50.3 |
| S-C | 10.10.50.6 |

### Servers

| Server | IP | Role |
|--------|----|-----|
| DB1 | 10.10.10.50 | AD + DHCP + DNS |
| HV1 | 10.10.10.150 | Hyper-V |
| WEB1 | 10.10.15.10 | Web |
| MAIL1 | 10.10.15.20 | Mail |

---

## 5. Device Connections (MATCHED TO IMAGE)

### Core Switch (S-A)
- Connected to R-A (trunk)
- Connected to servers (VLAN 10 + 15)
- Connected to S-B (EtherChannel)
- Connected to S-C (EtherChannel)

### Distribution Switch (S-B)
- Connected to R-B
- Connected to S-A (EtherChannel)
- Connected to S-C (EtherChannel)

### Access Switch (S-C)
- Staff PC → VLAN 20
- IT PC → VLAN 30
- Guest AP → VLAN 40
- Connected to S-A and S-B (EtherChannel)

---

## 6. Redundancy Design

### HSRP (Gateway Redundancy)

- Virtual Gateway: `10.10.X.254`
- R-A → Active
- R-B → Standby

If R-A fails → R-B becomes active automatically.

---

### EtherChannel Links

| Link | Between |
|------|--------|
| Po1 | S-A ↔ S-B |
| Po2 | S-A ↔ S-C |
| Po3 | S-B ↔ S-C |

This prevents link failure and increases bandwidth.

---

## 7. DHCP and DNS (IMPORTANT MATCH)

- Server: `10.10.10.50`
- Provides:
  - DHCP
  - DNS
  - Active Directory

### DHCP Relay (Router Config)
interface g0/0/0.20
ip helper-address 10.10.10.50

interface g0/0/0.30
ip helper-address 10.10.10.50

interface g0/0/0.40
ip helper-address 10.10.10.50

---

## 8. NAT Configuration

### PAT (Internet Access)

All internal users use NAT to access internet:
access-list 1 permit 10.10.0.0 0.0.255.255

ip nat inside source list 1 interface g0/0/1 overload


---

### Static NAT

| Public | Private | Service |
|--------|--------|--------|
| 10.128.250.21 | 10.10.15.10 | Web |
| 10.128.250.22 | 10.10.15.20 | Mail |

---

## 9. Wireless Network (From Diagram)

| AP | VLAN | Purpose |
|----|------|--------|
| AP-Guest | 40 | Guest Internet |

- Guests only access internet
- No access to internal network

---

## 10. Security Implementation

### Layer 2
- DHCP Snooping
- ARP Inspection
- BPDU Guard

### Layer 3
- ACLs restrict VLAN access

### Device Access
- SSH only
- Admin login from VLAN 50

---

## 11. Key Design Benefits

- ✔ High Availability (HSRP)
- ✔ Link Redundancy (EtherChannel)
- ✔ Strong Security (ACL + VLAN)
- ✔ Scalable Design

---

## 12. Conclusion

The implemented network successfully matches the designed topology and ensures:

- Secure communication
- Network redundancy
- Efficient traffic management


