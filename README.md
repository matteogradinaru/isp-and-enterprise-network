# Enterprise & Service Provider Network Engineering Lab

This is a hands-on network engineering labe that I have made during a school project emulating service provider backbones, multi-homed BGP routing, multi-tenant segmentation, and cryptographic overlays built in GNS3.

---

## 📌 Architectures Overview

This repository documents two separate, network topologies designed and verified using Cisco IOSv appliances:

| Section                                                           | Key Protocols                                   | Core Objectives                                                                                                                  |
| :---------------------------------------------------------------- | :---------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| **[Topology 1: Dual-ISP & VRF-Lite](./enterprise-dual-isp/)**     | IS-IS, eBGP, iBGP, VRF-Lite                     | Multi-level IGP backbone, multi-homed ISP failover with Local Preference/AS-Prepending, and isolated customer VRFs.              |
| **[Topology 2: MPLS Backbone & IPsec](./mpls-service-provider/)** | IS-IS, MPLS, LDP, MP-BGP (VPNv4), IKEv2 / IPsec | Scalable label switching core, multi-tenant L3VPN connectivity with RD/RT route targets, and end-to-end IPsec tunnel encryption. |

---

## 🛠️ Technology Stack & Protocols

- **Interior Gateway Protocols (IGP):** IS-IS (Multi-level Level-1/Level-2 hierarchy, Wide Metrics, Route Summarization)
- **Exterior Gateway Protocols (EGP):** eBGP (Inter-domain peering, Inbound Prefix Filtering), iBGP (Full mesh / Loopback peering with Next-Hop-Self)
- **Traffic Engineering (TE):** Outbound policy via BGP `Local-Preference`, Inbound policy via `AS-Path Prepending`
- **MPLS & VPN Architecture:** Multi-Protocol Label Switching (MPLS), Label Distribution Protocol (LDP), Penultimate Hop Popping (PHP), Multi-Protocol BGP (MP-BGP VPNv4), Virtual Routing and Forwarding (VRF-Lite & MPLS L3VPN)
- **Network Security & Encryption:** Route-based IKEv2 / IPsec site-to-site VPNs, AES-CBC-256, SHA-256, DH Group 14

---

## 📂 Repository Structure

```text
isp-and-enterprise-network/
├── README.md
├── .gitignore
├── enterprise-dual-isp/
│   ├── README.md
│   ├── configs/            # Cisco IOS running configs (.cfg)
│   └── screenshots/        # Screenshot verification
└── mpls-service-provider/
    ├── README.md
    ├── configs/            # Clean Cisco IOS running configs (.cfg)
    └── screenshots/        # Screenshot verification
```
