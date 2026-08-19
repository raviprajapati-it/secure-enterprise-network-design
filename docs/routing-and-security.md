# Routing & Network Security Architecture

## Overview

This document describes the routing and security architecture demonstrated across the three enterprise network-design case studies.

The projects were developed for different scenarios and at different stages, so they do not use one identical routing or security model.

Together they demonstrate experience with:

- Static routing
- OSPF
- Inter-VLAN routing
- Access Control Lists (ACLs)
- Cisco ASA firewall concepts
- NAT/PAT
- Network segmentation
- DMZ architecture
- Secure SSH administration
- Site-to-site IPsec VPNs
- Port security
- Layered enterprise network security

The configurations and claims in this repository distinguish between technologies preserved in original configuration evidence and technologies discussed as architectural design elements.

---

# Case Study 1 — UK & Switzerland Multi-Site Routing

## Routing Objective

The first case study required communication between departmental networks located across:

```text
United Kingdom
        │
        │ WAN
        │
Switzerland
```

Each location contains multiple departmental VLANs.

Routing therefore needed to support:

- Local VLAN networks
- Inter-site communication
- Predictable remote-network reachability

---

## Inter-Site WAN

The preserved router configurations use the point-to-point network:

```text
10.1.1.0/30
```

with:

```text
UK Router:          10.1.1.1
Switzerland Router: 10.1.1.2
```

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

---

# Static Routing

The original configurations contain explicit static routes between departmental networks.

## UK Router

The UK router contains routes toward the Switzerland networks:

```text
ip route 192.168.40.0 255.255.255.0 10.1.1.2
ip route 192.168.50.0 255.255.255.0 10.1.1.2
ip route 192.168.60.0 255.255.255.0 10.1.1.2
```

This directs traffic for the Switzerland departmental networks toward:

```text
10.1.1.2
```

---

## Switzerland Router

The Switzerland router contains corresponding routes toward the UK networks:

```text
ip route 192.168.10.0 255.255.255.0 10.1.1.1
ip route 192.168.20.0 255.255.255.0 10.1.1.1
ip route 192.168.30.0 255.255.255.0 10.1.1.1
```

This directs traffic toward:

```text
10.1.1.1
```

for the UK departmental networks.

---

# OSPF

The original Case Study 1 configurations also contain:

```text
router ospf 1
```

on both core routers.

The UK configuration includes network statements such as:

```text
network 192.168.1.0 0.0.0.255 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 10.10.10.0 0.0.0.3 area 0
```

The Switzerland configuration includes:

```text
network 192.168.2.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.40.0 0.0.0.255 area 0
network 10.10.10.0 0.0.0.3 area 0
```

---

## Configuration Review

The preserved OSPF configuration contains inconsistencies.

For example, the actual inter-site interfaces use:

```text
10.1.1.1/30
10.1.1.2/30
```

while the OSPF configuration references:

```text
10.10.10.0 0.0.0.3
```

Similarly, not every departmental network appears consistently in the OSPF statements.

This repository intentionally preserves those original commands rather than silently correcting them.

The static routes provide clearer evidence of the intended inter-site routing behavior in the preserved configuration.

---

# Case Study 2 — Data Center Routing Architecture

## Routing Strategy

The secure data-center redesign uses multilayer switching to provide routing between segmented VLANs.

The architecture is:

```text
                   Core
                    │
             Distribution
                    │
          ┌─────────┼─────────┐
          │         │         │
       VLAN 10   VLAN 20   VLAN 30
          │         │         │
         IT        HR      Finance

          + VLAN 40 Admin
          + VLAN 50 Servers
          + VLAN 99 Management
```

Switched Virtual Interfaces provide Layer 3 gateways for each VLAN.

---

## Inter-VLAN Routing

The multilayer switches provide communication between departmental networks while allowing ACLs to control which traffic is permitted.

Conceptually:

```text
Source VLAN
     │
     ▼
Layer 3 SVI
     │
     ▼
Security Policy / ACL
     │
     ├── Permit ──► Destination VLAN
     │
     └── Deny   ──► Drop
```

This is fundamentally different from placing every endpoint in one Layer 2 broadcast domain.

---

## Static Routing

Case Study 2 documents static routing as the primary routing approach.

Static routes were selected for:

- Predictability
- Administrative control
- Low protocol overhead
- Suitability for the relatively stable lab topology

Dynamic routing such as OSPF was identified as a potential future enhancement rather than part of the primary documented routing design.

---

# Access Control Lists

ACLs form an important part of the data-center security model.

Rather than allowing unrestricted communication between VLANs, traffic policies are based on departmental requirements.

---

## Finance-to-Web Restriction

The original project preserves the following ACL:

```text
access-list 110 deny tcp 192.168.30.0 0.0.0.255 host 192.168.50.20 eq 80
access-list 110 permit ip any any
```

This policy blocks:

```text
Source:
192.168.30.0/24
Finance VLAN

Destination:
192.168.50.20
Web Server

Protocol:
TCP

Port:
80 / HTTP
```

while the following rule permits remaining IP traffic:

```text
access-list 110 permit ip any any
```

---

## Security Principle

This demonstrates an important network-security principle:

> Segmentation creates boundaries; ACLs determine what is allowed to cross those boundaries.

VLANs alone do not provide complete access-control policy.

Layer 3 filtering is required when traffic moves between networks.

---

# Least-Privilege Network Access

Case Study 2 defines different access requirements for different departments.

## IT

Designed for broader administrative access required for:

- Infrastructure management
- Troubleshooting
- System maintenance

## Admin

Permitted access to required internal business services while restricting unnecessary management access.

## Finance

Restricted from selected services while retaining access to approved internal resources.

## HR

Limited according to business-service requirements.

This demonstrates role-oriented network policy rather than universal inter-department connectivity.

---

# Cisco ASA Firewall

Case Studies 2 and 3 introduce Cisco ASA firewalls as perimeter security controls.

The firewall separates trusted enterprise networks from untrusted connectivity.

Conceptually:

```text
                 Internet
                    │
                    ▼
             ┌─────────────┐
             │ Cisco ASA   │
             │  Firewall   │
             └─────────────┘
                │       │
                │       │
              DMZ     Private
                │       │
             Servers   Users
```

---

# ASA Security Levels

Case Study 3 preserves an ASA example using:

```text
interface Et0/0
 nameif inside
 security-level 100
 ip address 192.168.0.2 255.255.255.0

interface Et0/1
 nameif outside
 security-level 0
 ip address 10.0.0.2 255.255.255.252
```

This defines:

```text
Inside  = Security Level 100
Outside = Security Level 0
```

The configuration establishes explicit trust boundaries between interfaces.

---

# NAT

The preserved ASA configuration includes:

```text
nat (inside,outside) source dynamic any interface
```

This provides dynamic translation of internal addresses when traffic leaves through the outside interface.

NAT provides address translation but should not be treated as a substitute for firewall policy.

---

# Inbound ACLs

Case Study 3 documents examples such as:

```text
access-list OUTSIDE_IN extended permit tcp any host 192.168.40.10 eq 80
access-list OUTSIDE_IN extended permit tcp any host 192.168.40.11 eq 443
```

These rules represent explicitly permitted web traffic toward selected hosts.

The ACL is applied using:

```text
access-group OUTSIDE_IN in interface outside
```

This illustrates the principle of exposing only explicitly required services.

---

# DMZ Architecture

Case Study 3 uses:

```text
VLAN 40
```

for the DMZ.

The objective is to separate public-facing services from private employee networks.

Conceptually:

```text
                    Internet
                       │
                       ▼
                    Firewall
                  /            \
                 /              \
                ▼                ▼
              DMZ              Private
            VLAN 40         VLAN 10 Admin
               │            VLAN 20 Sales
        Web / Email         VLAN 30 HR
          Services
```

---

## Why a DMZ?

A DMZ reduces the need to expose internal departmental networks directly to untrusted traffic.

If a public-facing service requires external connectivity, it can be placed within a separately controlled network zone.

This creates additional policy boundaries between:

```text
Internet
DMZ
Internal Network
```

---

# Secure Remote Administration

Case Study 2 documents SSH for remote network-device administration.

SSH was used instead of Telnet because SSH provides encrypted administrative sessions.

The documented implementation includes:

- Device domain configuration
- RSA key generation
- Local user authentication
- VTY-line configuration
- Access restrictions
- IT-VLAN-only administrative access

Conceptually:

```text
IT Administrator
      │
      │ SSH
      ▼
Management Policy
      │
      ▼
Router / Switch

Other VLAN
      │
      │ SSH
      ▼
    DENIED
```

---

# Management Plane Security

Restricting SSH to the IT VLAN reduces the number of systems capable of reaching device-management interfaces.

This follows the principle:

> Administrative access should originate only from explicitly authorized management networks.

A stronger production design could extend this further through:

- Dedicated management VRFs
- TACACS+ or RADIUS
- MFA-supported administrative workflows
- Out-of-band management
- Centralized logging
- Management-plane ACLs

These are improvement recommendations and were not all implemented in the original projects.

---

# Case Study 3 — Site-to-Site IPsec VPN

The global enterprise design connects remote branches securely to headquarters.

The architecture is:

```text
San Francisco ── IPsec ──┐
                         │
Dubai ───────── IPsec ───┼── Cheltenham HQ
                         │
Vancouver ───── IPsec ───┘
```

The documented configuration provides an example for the HQ-to-San Francisco tunnel.

---

# IKE / ISAKMP Policy

The preserved configuration includes:

```text
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
```

This specifies:

| Parameter | Configuration |
| --- | --- |
| Encryption | AES |
| Integrity/Hash | SHA |
| Authentication | Pre-shared key |
| Diffie-Hellman | Group 2 |

---

# Pre-Shared Authentication

The original academic configuration used a plaintext lab pre-shared key.

The public portfolio intentionally replaces it with:

```text
<REDACTED_LAB_PSK>
```

rather than publishing the original value.

Conceptually:

```text
crypto isakmp key <REDACTED_LAB_PSK> address 10.0.0.2
```

Even though the original credential was only used in a lab, public repositories should avoid normalizing plaintext-secret publication.

---

# IPsec Transform Set

The documented configuration includes:

```text
crypto ipsec transform-set TS esp-aes esp-sha-hmac
```

This defines the IPsec protection used for VPN traffic.

---

# Crypto Map

The VPN configuration includes:

```text
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set TS
 match address 101
```

The crypto map combines:

- Remote peer
- Transform set
- Interesting-traffic ACL

---

# Interesting Traffic

The documented VPN ACL is:

```text
access-list 101 permit ip 192.168.0.0 0.0.0.255 192.168.11.0 0.0.0.255
```

This defines traffic between:

```text
HQ:
192.168.0.0/24

San Francisco:
192.168.11.0/24
```

as traffic intended for VPN processing in the example configuration.

---

# Crypto Map Application

The configuration applies the crypto map to:

```text
interface Se0/3/1
 crypto map VPN-MAP
```

Conceptually:

```text
HQ Internal Network
       │
       ▼
VPN ACL Match
       │
       ▼
IPsec Processing
       │
       ▼
WAN Interface
       │
======= Encrypted Tunnel =======
       │
       ▼
Remote Branch
```

---

# VPN Security Objectives

The site-to-site VPN architecture is intended to provide:

## Confidentiality

Encryption protects the contents of inter-site traffic.

## Integrity

Integrity mechanisms help detect modification of protected packets.

## Authentication

The VPN peers authenticate before establishing the protected connection.

---

# Legacy Cryptographic Design Note

The configuration reflects an academic Cisco lab design.

Some parameters, particularly:

```text
SHA
Diffie-Hellman Group 2
Pre-shared authentication
```

should not automatically be treated as preferred choices for a new production deployment.

A modern implementation should follow current vendor and organisational cryptographic standards.

Possible improvements could include:

- Stronger IKE versions/configurations
- Stronger Diffie-Hellman groups
- Modern integrity algorithms
- Certificate-based authentication
- Secure secret-management practices

These are review recommendations rather than claims about the original implementation.

---

# Layered Security Model

Across the three case studies, the strongest security architecture can be represented as:

```text
                    Internet
                       │
                       ▼
                 ASA Firewall
                       │
              ┌────────┴────────┐
              │                 │
             DMZ             Private
              │                 │
        Public Services    Department VLANs
                                │
                                ▼
                              ACLs
                                │
                     ┌──────────┼──────────┐
                     │          │          │
                   Admin      Sales       HR
                     │
                     ▼
             Restricted Management
                     │
                     ▼
                    SSH

Remote Branches
      │
      ▼
Site-to-Site IPsec
      │
      ▼
Headquarters
```

No single original case study implemented every element in exactly this combined form.

The diagram represents the security concepts demonstrated across the portfolio as a whole.

---

# Security Controls by Case Study

| Control | Case Study 1 | Case Study 2 | Case Study 3 |
| --- | --- | --- | --- |
| VLAN segmentation | Implemented | Implemented | Implemented |
| Inter-VLAN routing | Implemented | Implemented | Implemented |
| Static routing | Implemented | Primary design | Implemented |
| OSPF | Configuration preserved | Future enhancement | Limited documented example |
| ACLs | Security design | Implemented/tested | Configuration examples |
| ASA firewall | Design component | Implemented/tested | Configuration examples |
| NAT/PAT | Design component | Documented/tested | Configuration preserved |
| SSH | Not central | Implemented/tested | Security concept |
| DMZ/server separation | Limited | Server zone | DMZ design |
| IPsec VPN | Secure inter-site requirement | Not central | Configuration documented |

---

# Validation

Security controls should be evaluated based on whether intended traffic succeeds and prohibited traffic fails.

Case Study 2 explicitly documents validation using:

- Ping tests
- Gateway connectivity
- Inter-VLAN connectivity
- ACL testing
- DHCP validation
- Firewall behavior testing
- SSH access testing

Example policy validation:

```text
Finance VLAN
     │
     ├── File Server ──► PERMITTED
     │
     └── Web Server HTTP ──► DENIED
```

This is stronger evidence than merely documenting an ACL because it connects configuration intent with expected network behavior.

---

# Technical Review Findings

Reviewing the three projects together reveals several useful lessons.

## 1. Routing configuration must match interface addressing

OSPF network statements should correspond to the networks actually configured on participating interfaces.

The original Case Study 1 configuration contains mismatches that would need correction during a production implementation.

---

## 2. VLAN segmentation requires Layer 3 policy

Creating VLANs reduces Layer 2 broadcast scope but does not automatically enforce least privilege between routed networks.

ACLs or firewall policies are required for controlled inter-VLAN access.

---

## 3. NAT is not a security policy

NAT changes address representation.

Firewall rules and ACLs remain necessary to define which traffic is permitted.

---

## 4. Management access should be isolated

SSH is preferable to plaintext management protocols, but encryption alone is not sufficient.

Administrative access should also be restricted by:

- Source network
- Authentication
- Authorization
- Logging

---

## 5. Public services should be separated

DMZ architecture reduces direct exposure of private enterprise systems.

Public services should not normally share the same unrestricted trust domain as employee endpoints.

---

## 6. VPN configuration requires more than encryption

A complete site-to-site design includes:

- Peer identification
- Authentication
- Encryption
- Integrity
- Interesting-traffic definition
- Routing
- Firewall policy
- Key management

---

# Production Improvement Opportunities

If these academic designs were evolved into a modern production architecture, improvements could include:

- OSPF design cleanup and route summarisation
- HSRP/VRRP for gateway redundancy
- High-availability firewall pairs
- Redundant WAN providers
- IKEv2
- Modern cryptographic suites
- Certificate-based VPN authentication
- AAA using TACACS+ or RADIUS
- Dedicated management networks or VRFs
- Centralized Syslog
- SNMPv3
- NetFlow/IPFIX
- IDS/IPS integration
- Network Access Control
- 802.1X
- DHCP snooping
- Dynamic ARP Inspection
- BPDU Guard
- Root Guard
- Infrastructure configuration backups
- Automated configuration validation

These are recommendations derived from reviewing the original designs and are not presented as features implemented in every case study.

---

# Key Takeaways

Across the three network-design projects, the routing and security work demonstrates practical understanding of:

- Static routing
- OSPF
- Layer 3 switching
- Inter-VLAN routing
- ACL design
- Least-privilege network access
- Cisco ASA security levels
- NAT/PAT
- DMZ segmentation
- SSH management
- Site-to-site IPsec
- VPN traffic selection
- Enterprise network segmentation
- Security-policy validation
- Configuration review

The progression moves from multi-site routing toward increasingly layered enterprise security and secure global connectivity.

---

## Related Resources

- [Enterprise Network Case Studies](../case-studies/README.md)
- [Addressing & VLAN Design](addressing-and-vlans.md)
- [UK Core Router Configuration](../configs/case-study-01/uk-core-router-original.cfg)
- [Switzerland Core Router Configuration](../configs/case-study-01/switzerland-core-router-original.cfg)
- [Data Center Security Controls](../configs/case-study-02/security-controls-original.txt)
- [Global Security & VPN Configuration](../configs/case-study-03/global-security-and-vpn-config.txt)
