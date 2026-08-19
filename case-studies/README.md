# Enterprise Network Design Case Studies

## Overview

This repository consolidates three related enterprise networking projects into a single technical portfolio.

Rather than representing the designs as one physical network, each project is preserved as a separate case study demonstrating progressively broader network-design and security concepts.

The three case studies cover:

1. Multi-site enterprise networking
2. Secure data-center redesign
3. Global multi-branch enterprise architecture

Together, they demonstrate practical experience with network architecture, Cisco configuration, segmentation, routing, access control, firewalling, secure administration, and site-to-site connectivity.

---

# Case Study 1 — Multi-Site Enterprise Network

## Scenario

The first design supports an organisation operating across:

- United Kingdom
- Switzerland

The organisation contains multiple departments distributed between the two locations and requires secure, scalable communication between sites.

The design follows a hierarchical enterprise architecture consisting of:

```text
Core
  │
Distribution
  │
Access
```

---

## Core Technologies

The design incorporates:

- Cisco routers
- Multilayer switches
- Access switches
- Departmental VLANs
- Inter-VLAN routing
- OSPF
- Static routing
- DHCP
- Cisco ASA firewalling
- WAN connectivity
- Network redundancy concepts

---

## VLAN Architecture

Departmental networks were separated using VLANs.

| VLAN | Department | Location |
| --- | --- | --- |
| 10 | IT Services | UK |
| 20 | HR | UK |
| 30 | Training | UK |
| 40 | Technical Support | Switzerland |
| 50 | HR | Switzerland |
| 60 | Network Operations | Switzerland |

Each VLAN was assigned a dedicated `/24` subnet.

This design reduces broadcast domains and provides clearer boundaries for routing and security policy.

---

## IP Addressing

Example departmental networks included:

```text
192.168.10.0/24 — IT Services
192.168.20.0/24 — HR
192.168.30.0/24 — Training
192.168.40.0/24 — Technical Support
192.168.50.0/24 — HR
192.168.60.0/24 — Network Operations
```

The addressing strategy maps departmental VLANs directly to dedicated IP subnets.

---

## DHCP

DHCP pools were configured to provide dynamic addressing to departmental endpoints.

Example UK pools included:

```text
IT-Services
192.168.10.0/24

HR
192.168.20.0/24

Training
192.168.30.0/24
```

Equivalent DHCP services were configured for the Switzerland networks.

---

## Routing

The implementation included both:

- OSPF
- Static routes

OSPF was configured to advertise selected internal networks, while static routes provided explicit reachability between remote departmental subnets.

The configuration evidence preserved in the original project includes router running configurations rather than only conceptual routing diagrams.

---

## Skills Demonstrated

This case study demonstrates:

- Hierarchical network design
- VLAN planning
- IPv4 subnetting
- DHCP configuration
- Inter-VLAN routing
- OSPF
- Static routing
- Cisco IOS configuration
- Multi-site network planning

---

# Case Study 2 — Secure Data Center Redesign

## Scenario

The second case study begins with a legacy data-center network based on a flat Layer 2 topology.

The original environment suffered from several weaknesses:

- Single broadcast domain
- No VLAN segmentation
- No Layer 3 routing
- No ACL enforcement
- No firewall
- Limited redundancy
- No secure remote administration
- Unrestricted access to internal services

The objective was to redesign the environment using a scalable and security-focused enterprise architecture.

---

## Redesigned Architecture

The redesigned network uses a three-tier model:

```text
             Core
              │
        Distribution
              │
            Access
```

### Core Layer

Implemented using Cisco routers responsible for high-level routing and resilient connectivity.

### Distribution Layer

Implemented using multilayer switches responsible for:

- Inter-VLAN routing
- Switched Virtual Interfaces
- ACL enforcement
- Traffic aggregation
- DHCP relay

### Access Layer

Provides departmental endpoint connectivity and VLAN membership.

---

## VLAN Segmentation

The redesigned environment uses:

| VLAN | Department | Subnet |
| --- | --- | --- |
| 10 | IT | `192.168.10.0/24` |
| 20 | HR | `192.168.20.0/24` |
| 30 | Finance | `192.168.30.0/24` |
| 40 | Admin | `192.168.40.0/24` |
| 50 | Server Zone | `192.168.50.0/24` |
| 99 | Management | `192.168.99.0/24` |

This replaced the original flat trust model with departmental and functional segmentation.

---

## Server Infrastructure

Core services were placed inside the dedicated server network.

Examples included:

```text
DHCP Server — 192.168.50.10
Web Server  — 192.168.50.20
File Server — 192.168.50.30
```

Access to services was controlled through ACL policy.

---

## ACL Security Policy

Department-specific access policies were implemented.

Examples included:

- IT receiving broader administrative access
- Admin receiving access to required internal services
- Finance being restricted from specific web resources
- HR receiving only the services required for its role

An example ACL from the project:

```text
access-list 110 deny tcp 192.168.30.0 0.0.0.255 host 192.168.50.20 eq 80
access-list 110 permit ip any any
```

This demonstrates policy-based traffic filtering rather than unrestricted inter-VLAN communication.

---

## ASA Firewall

A Cisco ASA firewall was introduced between the enterprise network and untrusted external connectivity.

The design included:

- NAT/PAT
- Inbound traffic filtering
- Protocol-specific access rules
- Perimeter separation
- Protection of internal services

The firewall was configured to deny unsolicited inbound connections while permitting explicitly required services.

---

## Secure Management

SSH replaced insecure plaintext remote-management protocols.

The implementation included:

- RSA keys
- Local authentication
- VTY configuration
- Source restrictions
- IT-VLAN-only administrative access

This provided encrypted remote administration of network infrastructure.

---

## Validation

The redesigned network was tested using:

- Ping
- Inter-VLAN connectivity tests
- ACL validation
- DHCP scope validation
- Firewall behaviour tests
- SSH access tests
- Routing-table inspection
- VLAN inspection

The objective was not simply to design the topology but to verify that intended connectivity was allowed and restricted traffic was blocked.

---

## Skills Demonstrated

This case study demonstrates:

- Data-center network redesign
- Network segmentation
- Least-privilege traffic policy
- ACL implementation
- Cisco ASA concepts
- NAT/PAT
- SSH administration
- Server-zone design
- DHCP
- Network validation

---

# Case Study 3 — Global Secure Enterprise Network

## Scenario

The third design expands the enterprise architecture internationally.

The network connects:

```text
Cheltenham HQ
      │
      ├── San Francisco
      ├── Dubai
      └── Vancouver
```

The objective was to provide secure, scalable communication between geographically separated enterprise locations.

---

## Architecture

Each site includes a combination of:

- Router
- Cisco ASA firewall
- Multilayer switch
- Access switches
- Departmental VLANs
- Servers
- End-user systems

Headquarters acts as the central communication point for the branch architecture.

---

## Hierarchical Addressing

Each location was assigned a distinct internal address space.

Examples included:

```text
HQ            — 192.168.0.0/24
San Francisco — 192.168.1.0/24
Dubai         — 192.168.2.0/24
Vancouver     — 192.168.3.0/24
```

Departmental VLANs then received dedicated subnets within the broader addressing design.

---

## Point-to-Point Networks

WAN links used `/30` subnets.

Examples:

```text
HQ ↔ San Francisco — 10.0.0.0/30
HQ ↔ Dubai         — 10.0.0.4/30
HQ ↔ Vancouver     — 10.0.0.8/30
```

Using `/30` networks provides two usable addresses per point-to-point connection and avoids unnecessarily large WAN broadcast domains.

---

## Departmental Segmentation

Each branch used VLANs representing functions such as:

```text
VLAN 10 — Admin
VLAN 20 — Sales
VLAN 30 — HR
VLAN 40 — DMZ
```

The design separates employee departments from public-facing server infrastructure.

---

## Security Zones

Cisco ASA firewalls were designed around multiple trust zones.

### Private

Internal departmental networks.

### Public

External or internet-facing connectivity.

### DMZ

Public-facing services isolated from internal enterprise systems.

This creates stronger security boundaries than placing all systems within the same trust domain.

---

## Access Control

ACLs were used to control permitted traffic between:

- Internal VLANs
- Branch networks
- DMZ services
- External networks

The design follows a restrictive policy model in which only required traffic should be permitted.

---

## Site-to-Site IPsec VPN

Secure branch connectivity was designed using IPsec VPN tunnels between headquarters and remote locations.

The documented configuration includes:

```text
crypto isakmp policy
AES encryption
SHA hashing
pre-shared authentication
IPsec transform sets
crypto maps
VPN ACLs
```

Conceptually:

```text
San Francisco ── IPsec ──┐
                         │
Dubai ───────── IPsec ───┼── Cheltenham HQ
                         │
Vancouver ───── IPsec ───┘
```

This protects inter-site traffic for:

- Confidentiality
- Integrity
- Authentication

---

## Switching

Multilayer switches provide:

- VLAN SVIs
- Inter-VLAN routing
- Trunk connectivity
- Departmental gateways

Access switches provide:

- Endpoint connectivity
- VLAN membership
- Trunk uplinks
- Port-security controls

---

## DMZ Design

Public-facing services such as web or email servers are separated from private departmental systems.

This reduces direct exposure of internal enterprise resources while still allowing required public services to remain accessible.

---

## Skills Demonstrated

This case study demonstrates:

- Global enterprise network architecture
- Multi-site IP planning
- VLAN segmentation
- Layer 3 switching
- Inter-VLAN routing
- Cisco ASA security zones
- ACLs
- NAT concepts
- DMZ architecture
- Site-to-site IPsec VPN
- Enterprise security design

---

# Design Progression

The three case studies demonstrate a progression in network architecture and security.

| Area | Case Study 1 | Case Study 2 | Case Study 3 |
| --- | --- | --- | --- |
| Multi-site networking | ✓ | — | ✓ |
| Hierarchical architecture | ✓ | ✓ | ✓ |
| VLAN segmentation | ✓ | ✓ | ✓ |
| Inter-VLAN routing | ✓ | ✓ | ✓ |
| DHCP | ✓ | ✓ | ✓ |
| OSPF | ✓ | Future enhancement | Present in broader design/configuration context |
| Static routing | ✓ | ✓ | ✓ |
| ACLs | Security design | ✓ | ✓ |
| ASA firewall | Security design | ✓ | ✓ |
| NAT/PAT | Security design | ✓ | ✓ |
| SSH management | — | ✓ | Device-management security |
| DMZ | — | Server zone | ✓ |
| Site-to-site IPsec | Secure inter-site requirement | — | ✓ |
| International branches | UK/Switzerland | — | HQ + 3 branches |

---

# Important Scope Note

These case studies were developed at different stages and use different:

- Network scenarios
- Addressing schemes
- Device selections
- Routing approaches
- Security requirements

They should therefore **not** be interpreted as three parts of one production network.

They are preserved as related network-engineering case studies demonstrating progressively broader design and security concepts.

Where a technology was discussed conceptually but not supported by preserved implementation evidence, the repository documentation avoids presenting it as experimentally verified.

---

# Overall Skills Demonstrated

Across the three case studies, the project demonstrates practical knowledge of:

- Cisco IOS
- Enterprise Network Architecture
- IPv4 Addressing
- Subnetting
- VLANs
- Trunking
- Inter-VLAN Routing
- OSPF
- Static Routing
- DHCP
- ACLs
- Cisco ASA
- NAT/PAT
- SSH
- DMZ Design
- IPsec VPN
- Port Security
- Network Segmentation
- Network Validation
- Packet Tracer
- Secure Network Design
