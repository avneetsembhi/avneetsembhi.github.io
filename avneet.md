# FitLife Gym — Network Infrastructure Technical Documentation

**NSA630 Capstone Project**  
**Manitoba Institute of Trades and Technology (MITT)**  
**Date: April 2026**

---

## 1. Executive Summary

This document presents the complete design and implementation of the FitLife Gym network infrastructure. The network is built to support business operations with a strong focus on **security, redundancy, and scalability**.

The solution includes dual routers configured with HSRP for gateway redundancy, multiple switches interconnected using EtherChannel for link resilience, and VLAN segmentation to separate different types of users and services. Core services such as **Active Directory, DNS, DHCP, web hosting, and email services** are deployed within secure network zones.

Internet access is provided through Network Address Translation (NAT), while Access Control Lists (ACLs) enforce strict communication policies between VLANs.

---

## 2. Project Overview

FitLife Gym requires a reliable IT infrastructure to support:

- Staff operations (front desk, trainers, finance)
- IT administration and management
- Guest Wi-Fi access
- Internal services (authentication, file sharing)
- Public services (website and email)

Each user group has **different access requirements**, which are enforced using VLAN segmentation and ACL policies.

---

## 3. Network Design Overview

The network follows a **collapsed core/distribution architecture** with a dedicated access layer.

- **Routers (R-A, R-B)** provide inter-VLAN routing and internet connectivity  
- **Switches (S-A, S-B, S-C)** handle Layer 2 switching and connectivity  
- **HSRP** ensures continuous gateway availability  
- **EtherChannel** improves bandwidth and redundancy  

This design ensures that even if a device or link fails, the network continues to operate without disruption.

---

## 4. VLAN and IP Addressing Design

To ensure proper segmentation, the network is divided into multiple VLANs, each assigned a unique subnet.

### VLAN Structure

| VLAN ID | Name       | Subnet           | Purpose |
|--------|------------|------------------|--------|
| 10     | SERVER     | 10.10.10.0/24    | AD, DNS, DHCP |
| 15     | DMZ        | 10.10.15.0/24    | Web & Mail Servers |
| 20     | STAFF      | 10.10.20.0/24    | Employees |
| 30     | IT_ADMIN   | 10.10.30.0/24    | IT Management |
| 40     | GUEST      | 10.10.40.0/24    | Public Wi-Fi |
| 50     | MGMT       | 10.10.50.0/24    | Network Devices |

Each VLAN uses a virtual gateway (`10.10.X.254`) provided by HSRP.

---

## 5. Network Device Roles

Each device in the network plays a specific role:

| Device | Role |
|--------|------|
| R-A | Primary router (HSRP Active) |
| R-B | Secondary router (HSRP Standby) |
| S-A | Core/Distribution switch |
| S-B | Distribution switch |
| S-C | Access switch |

---

## 6. Redundancy and High Availability

### HSRP (Gateway Redundancy)

HSRP is configured on both routers to provide a virtual default gateway for all VLANs.

- R-A operates as the **active router**
- R-B remains in **standby mode**
- Virtual IP: `10.10.X.254`

If R-A fails, R-B automatically takes over without interrupting user connectivity.

---

### EtherChannel (Link Redundancy)

Switches are connected using LACP EtherChannel links.

| EtherChannel | Connection |
|-------------|-----------|
| Po1 | S-A ↔ S-B |
| Po2 | S-A ↔ S-C |
| Po3 | S-B ↔ S-C |

This setup increases bandwidth and prevents single points of failure.

---

## 7. Routing and NAT

The routers use a **router-on-a-stick** configuration to route traffic between VLANs.

### NAT Implementation

Currently, **PAT (Port Address Translation)** is implemented:

- All internal traffic (10.10.0.0/16) is translated to the WAN IP  
- Enables internet access for all users  

### Static NAT (Design)

| Public IP | Internal IP | Service |
|----------|------------|--------|
| 10.128.250.21 | 10.10.15.10 | Web |
| 10.128.250.22 | 10.10.15.20 | Mail |

---

## 8. DHCP and DNS Services

A centralized server (10.10.10.50) provides both DHCP and DNS services.

- DHCP assigns IP addresses to client devices  
- DNS resolves internal and external domain names  

### DHCP Relay

