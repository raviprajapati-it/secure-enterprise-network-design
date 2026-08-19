# IP Addressing, Subnetting & VLAN Design

## Overview

This document describes the IP addressing, subnetting, and VLAN segmentation strategies used across the three enterprise network-design case studies in this repository.

The case studies were developed for different network scenarios and therefore use different addressing schemes.

They should not be interpreted as components of one production network.

The common design principles across the projects are:

- Dedicated subnets for departments and functional zones
- VLAN-based segmentation
- Predictable gateway addressing
- Separation of user and server networks
- Small point-to-point WAN subnets
- Hierarchical addressing for multi-site environments
- Inter-VLAN routing through Layer 3 devices
- Addressing plans designed to support troubleshooting and future expansion

---

# Case Study 1 — UK & Switzerland Multi-Site Network

## Addressing Strategy

The first case study separates departments across two geographical locations:

```text
United Kingdom
    │
    ├── IT Services
    ├── HR
    └── Training

Switzerland
    │
    ├── Technical Support
    ├── HR
    └── Network Operations
```

Each department receives a dedicated `/24` network.

---

## VLAN Allocation

| VLAN | Department | Location | Subnet |
| ---: | --- | --- | --- |
| 10 | IT Services | UK | `192.168.10.0/24` |
| 20 | HR | UK | `192.168.20.0/24` |
| 30 | Training | UK | `192.168.30.0/24` |
| 40 | Technical Support | Switzerland | `192.168.40.0/24` |
| 50 | HR | Switzerland | `192.168.50.0/24` |
| 60 | Network Operations | Switzerland | `192.168.60.0/24` |

This produces a simple relationship between VLAN identifiers and departmental networks.

---

## Default Gateways

The first usable address in each departmental subnet is used as the VLAN gateway in the preserved configurations.

### United Kingdom

```text
VLAN 10 — 192.168.10.1
VLAN 20 — 192.168.20.1
VLAN 30 — 192.168.30.1
```

### Switzerland

```text
VLAN 40 — 192.168.40.1
VLAN 50 — 192.168.50.1
VLAN 60 — 192.168.60.1
```

---

## DHCP Addressing

The router configurations preserve the first ten addresses of each departmental network before dynamically assigning client addresses.

Example:

```text
ip dhcp excluded-address 192.168.10.1 192.168.10.10

ip dhcp pool IT-Services
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

The same pattern is used for the other departmental networks.

This reserves addresses such as:

```text
.1 - .10
```

for gateways or infrastructure while leaving the remaining addresses available for DHCP clients.

---

## Inter-Site Link

The preserved router configurations use:

```text
UK:          10.1.1.1/30
Switzerland: 10.1.1.2/30
```

This creates the point-to-point network:

```text
10.1.1.0/30
```

with two usable host addresses.

Conceptually:

```text
R-UK-CORE-01
10.1.1.1
     │
     │ 10.1.1.0/30
     │
10.1.1.2
R-CH-CORE-01
```

A `/30` is suitable for this lab design because only two router interfaces require addresses.

---

## Segmentation Benefit

The design creates six separate departmental broadcast domains rather than placing all users into one large Layer 2 network.

Benefits include:

- Reduced broadcast scope
- Clear departmental boundaries
- Easier troubleshooting
- More predictable routing
- Improved policy enforcement
- Simpler DHCP administration

---

# Case Study 2 — Secure Data Center Redesign

## Original Problem

The legacy network used a flat:

```text
192.168.0.0/24
```

address space with a shared broadcast domain.

The redesign replaces that flat structure with dedicated departmental, server, and management networks.

---

## VLAN & Subnet Plan

| VLAN | Function | Subnet |
| ---: | --- | --- |
| 10 | IT | `192.168.10.0/24` |
| 20 | HR | `192.168.20.0/24` |
| 30 | Finance | `192.168.30.0/24` |
| 40 | Admin | `192.168.40.0/24` |
| 50 | Server Zone | `192.168.50.0/24` |
| 99 | Management | `192.168.99.0/24` |

This separates:

```text
Users
  │
  ├── IT
  ├── HR
  ├── Finance
  └── Admin

Infrastructure
  │
  ├── Server Zone
  └── Management
```

---

## Why VLAN 99 Matters

A dedicated management VLAN separates infrastructure administration from normal user traffic.

Conceptually:

```text
User VLANs
   │
   ├── Business traffic
   └── Application access

Management VLAN 99
   │
   └── Network-device administration
```

This supports stronger control over access to routers, switches, and other management interfaces.

---

## Server Zone

VLAN 50 contains central infrastructure services.

Documented examples include:

| Service | Address |
| --- | --- |
| DHCP Server | `192.168.50.10` |
| Web Server | `192.168.50.20` |
| File Server | `192.168.50.30` |

Separating servers from user VLANs allows access policies to be enforced between departments and services.

For example, the project documents an ACL preventing the Finance subnet:

```text
192.168.30.0/24
```

from reaching:

```text
192.168.50.20
```

over HTTP.

This demonstrates how subnet design and security policy work together.

---

## Inter-VLAN Routing

Switched Virtual Interfaces on multilayer switches provide gateways for the VLANs.

Conceptually:

```text
VLAN 10 ──┐
VLAN 20 ──┤
VLAN 30 ──┤
VLAN 40 ──┼── Multilayer Switch ── Core
VLAN 50 ──┤
VLAN 99 ──┘
```

This enables controlled Layer 3 communication while preserving Layer 2 segmentation.

---

## DHCP Relay

The design uses:

```text
ip helper-address
```

on VLAN interfaces to forward DHCP requests to the central DHCP service.

Conceptually:

```text
Client
  │
  ▼
Access Switch
  │
  ▼
VLAN SVI
  │
  │ ip helper-address
  ▼
DHCP Server
192.168.50.10
```

This allows one central DHCP server to support clients in multiple broadcast domains.

---

# Case Study 3 — Global Enterprise Network

## Global Addressing Strategy

The third case study introduces a multi-site enterprise architecture connecting:

- Cheltenham Headquarters
- San Francisco
- Dubai
- Vancouver

Each location receives a distinct site-level address space.

| Site | Site Network |
| --- | --- |
| Headquarters | `192.168.0.0/24` |
| San Francisco | `192.168.1.0/24` |
| Dubai | `192.168.2.0/24` |
| Vancouver | `192.168.3.0/24` |

The project then uses additional dedicated departmental VLAN networks at each site.

---

# Headquarters VLANs

| VLAN | Function | Subnet | Gateway |
| ---: | --- | --- | --- |
| 10 | Admin | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Sales | `192.168.20.0/24` | `192.168.20.1` |
| 30 | HR | `192.168.30.0/24` | `192.168.30.1` |
| 40 | DMZ | `192.168.40.0/24` | `192.168.40.1` |

---

# San Francisco VLANs

| VLAN | Function | Subnet | Gateway |
| ---: | --- | --- | --- |
| 10 | Admin | `192.168.11.0/24` | `192.168.11.1` |
| 20 | Sales | `192.168.12.0/24` | `192.168.12.1` |
| 30 | HR | `192.168.13.0/24` | `192.168.13.1` |
| 40 | DMZ | `192.168.14.0/24` | `192.168.14.1` |

---

# Dubai VLANs

| VLAN | Function | Subnet | Gateway |
| ---: | --- | --- | --- |
| 10 | Admin | `192.168.21.0/24` | `192.168.21.1` |
| 20 | Sales | `192.168.22.0/24` | `192.168.22.1` |
| 30 | HR | `192.168.23.0/24` | `192.168.23.1` |
| 40 | DMZ | `192.168.24.0/24` | `192.168.24.1` |

---

# Vancouver VLANs

| VLAN | Function | Subnet | Gateway |
| ---: | --- | --- | --- |
| 10 | Admin | `192.168.31.0/24` | `192.168.31.1` |
| 20 | Sales | `192.168.32.0/24` | `192.168.32.1` |
| 30 | HR | `192.168.33.0/24` | `192.168.33.1` |
| 40 | DMZ | `192.168.34.0/24` | `192.168.34.1` |

---

# Site-Based Addressing Pattern

The departmental addressing creates a recognizable pattern.

```text
HQ
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24

San Francisco
192.168.11.0/24
192.168.12.0/24
192.168.13.0/24
192.168.14.0/24

Dubai
192.168.21.0/24
192.168.22.0/24
192.168.23.0/24
192.168.24.0/24

Vancouver
192.168.31.0/24
192.168.32.0/24
192.168.33.0/24
192.168.34.0/24
```

This provides a degree of visual predictability when troubleshooting or reviewing configurations.

---

# Global WAN Links

The documented global design uses `/30` point-to-point networks between headquarters and branches.

| Connection | WAN Network |
| --- | --- |
| HQ ↔ San Francisco | `10.0.0.0/30` |
| HQ ↔ Dubai | `10.0.0.4/30` |
| HQ ↔ Vancouver | `10.0.0.8/30` |

Conceptually:

```text
                    San Francisco
                     10.0.0.0/30
                          │
                          │
                          ▼
                        HQ
                    Cheltenham
                       /   \
                      /     \
        10.0.0.4/30  /       \ 10.0.0.8/30
                    /         \
                 Dubai      Vancouver
```

---

## Why `/30` Was Used

A traditional IPv4 `/30` subnet provides:

```text
4 total addresses
2 usable host addresses
1 network address
1 broadcast address
```

For a point-to-point lab connection requiring only two router interfaces, this provides a simple and predictable addressing structure.

---

# VLAN Trunking

Case Study 3 explicitly documents trunking between switching layers.

Example:

```text
interface Fa0/5
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
```

This allows multiple VLANs to cross one physical switch-to-switch link while retaining logical separation.

Conceptually:

```text
Access Switch
     │
     │ VLANs 10,20,30,40
     │
     ▼
   Trunk
     │
     ▼
Multilayer Switch
```

---

# Access Ports

Departmental access ports are assigned to individual VLANs.

Example:

```text
interface Fa0/1
 switchport mode access
 switchport access vlan 10
```

A device attached to this interface becomes part of VLAN 10.

Equivalent assignments are documented for VLANs 20, 30, and 40.

---

# Inter-VLAN Routing

The multilayer switch provides VLAN gateway interfaces.

Example:

```text
interface vlan10
 ip address 192.168.10.1 255.255.255.0

interface vlan20
 ip address 192.168.20.1 255.255.255.0

interface vlan30
 ip address 192.168.30.1 255.255.255.0

interface vlan40
 ip address 192.168.40.1 255.255.255.0

ip routing
```

This allows the Layer 3 switch to route between VLAN networks subject to the applicable security policies.

---

# DMZ Segmentation

Case Study 3 assigns:

```text
VLAN 40
```

to the DMZ function.

The objective is to separate public-facing services from internal employee networks.

Conceptually:

```text
                  Internet
                     │
                     ▼
                  Firewall
                 /        \
                /          \
             DMZ          Private
           VLAN 40      VLANs 10-30
              │
        Public Services
```

This reduces direct exposure of internal departmental systems.

---

# Addressing Design Principles

Across the three case studies, several recurring principles are visible.

## 1. Separate Functions into Separate Subnets

Departments and infrastructure roles receive dedicated address spaces.

This improves:

- Broadcast control
- Policy enforcement
- Troubleshooting
- Security boundaries

---

## 2. Use Predictable Gateway Addresses

The projects generally use:

```text
x.x.x.1
```

as the VLAN gateway.

This makes gateway addressing easy to identify during troubleshooting.

---

## 3. Separate Servers from User Networks

Case Studies 2 and 3 explicitly separate server infrastructure from ordinary user endpoints.

Examples include:

```text
Server VLAN
DMZ VLAN
Management VLAN
```

---

## 4. Use Small WAN Subnets

Point-to-point connections use `/30` addressing where documented.

This avoids assigning an unnecessarily large subnet to a two-interface connection.

---

## 5. Align Segmentation with Security Policy

VLANs are not treated only as performance mechanisms.

They provide boundaries to which:

- ACLs
- Firewall rules
- Management restrictions
- Server-access policies

can be applied.

---

# Technical Review

The three case studies were completed at different stages and contain some differences and inconsistencies in their addressing documentation.

Examples include:

- Different site-addressing strategies
- Different VLAN numbering schemes
- Different WAN ranges
- Differences between some configuration examples and narrative descriptions
- Routing statements that do not always align perfectly with configured interface networks

These differences are preserved rather than silently rewritten.

The purpose of this portfolio is to demonstrate both the original networking work and the ability to review that work critically.

---

# Production Design Considerations

If these designs were being rebuilt for a production enterprise environment, additional considerations would include:

- Address summarisation strategy
- Formal IP Address Management (IPAM)
- IPv6 planning
- Reserved infrastructure ranges
- Loopback addressing
- Dedicated transit networks
- Dedicated management networks
- Redundant default gateways
- Dynamic routing design
- Route filtering
- DHCP redundancy
- First-hop redundancy
- High-availability firewall pairs
- Network monitoring and telemetry

These are improvement considerations and should not be interpreted as technologies implemented in every original case study.

---

# Key Takeaways

Across the three projects, the addressing and VLAN work demonstrates:

- IPv4 subnet planning
- Departmental segmentation
- VLAN design
- Gateway planning
- DHCP integration
- Server-network separation
- Management-network separation
- DMZ segmentation
- Point-to-point addressing
- VLAN trunking
- Inter-VLAN routing
- Multi-site address planning

The progression moves from basic departmental segmentation toward a more structured global enterprise addressing and security architecture.

---

## Related Resources

- [Enterprise Network Case Studies](../case-studies/README.md)
- [UK Core Router Configuration](../configs/case-study-01/uk-core-router-original.cfg)
- [Switzerland Core Router Configuration](../configs/case-study-01/switzerland-core-router-original.cfg)
- [Data Center Security Controls](../configs/case-study-02/security-controls-original.txt)
- [Global Security & VPN Configuration](../configs/case-study-03/global-security-and-vpn-config.txt)
