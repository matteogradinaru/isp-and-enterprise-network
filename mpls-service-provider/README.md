# Topology 2: MPLS L3VPN Backbone & IPsec Overlays

This lab simulates an ISP / Managed Service Provider backbone delivering multi-tenant L3VPN connectivity over an MPLS core, augmented with customer-managed route-based IPsec tunnels (IKEv2) for end-to-end zero-trust encryption across the provider cloud.

---

## Topology Architecture

![Topology 2 Architecture](screenshots/topology2.png)

---

## Design & Configuration Details

### 1. Provider Core Underlay (IS-IS + MPLS/LDP)

- **Core IGP:** The provider backbone (`PE-1`, `P-1`, `P-2`, `PE-2`) runs IS-IS as a single Level-2 routing domain with wide metrics, ensuring fast convergence and loop-free reachability between all loopback interfaces.
- **Label Distribution:** LDP is enabled on all core links (`mpls ip`). Labels are dynamically mapped to loopback endpoints, enabling Penultimate Hop Popping (PHP) on P-routers to reduce lookup overhead on egress PEs.

### 2. MP-BGP VPNv4 Control Plane

- **Multiprotocol BGP:** PE routers peer directly over iBGP via their `Loopback0` addresses under AS `65100`, activating the `vpnv4` address-family with extended communities (`send-community extended`).
- **VRF Routing & Route Targets:**
  - `KLANT-A`: Configured with Route Distinguisher `65100:100` and import/export Route Target `65100:100`.
  - `KLANT-B`: Configured with Route Distinguisher `65100:200` and import/export Route Target `65100:200`.
- **PE-CE Routing:** Customer routes and connected interfaces are redistributed into MP-BGP on the PE routers, allowing customer sites to communicate across the provider core while keeping their routing tables completely segregated.

### 3. Customer IPsec Overlay (Zero-Trust over L3VPN)

While MPLS L3VPN provides traffic segmentation, provider core nodes could theoretically inspect transit customer packets. To ensure full confidentiality, Tenant A establishes a route-based IPsec tunnel between CE routers:

- **Phase 1 (IKEv2):** Configured with AES-CBC-256 encryption, SHA-256 hashing, and Diffie-Hellman Group 14 using pre-shared keys.
- **Phase 2 (IPsec):** Configured with an `esp-aes 256 esp-sha256-hmac` transform-set applied to virtual tunnel interfaces (`Tunnel0`).
- **Encrypted Payload:** Traffic between internal tenant networks (`10.1.0.0/24` and `10.2.0.0/24`) is routed through `Tunnel0` (`10.99.0.0/30`), completely hiding internal customer data from the service provider.

---

## Verification & Test Results

### 1. MPLS Label Forwarding Information Base (LFIB)

Checking `show mpls forwarding-table` on `PE-1` verifies label assignments, label push/swap operations, and Penultimate Hop Popping (`Pop Label`) behavior toward P-routers:
![MPLS Forwarding Table](screenshots/mpls-forwarding-table.png)

### 2. MP-BGP VPNv4 Routes

Checking `show bgp vpnv4 unicast all` confirms that routes from both customer VRFs are tagged with their respective Route Distinguishers and propagated across the MP-BGP mesh:
![MP-BGP VPNv4](screenshots/bgp-vpnv4-unicast.png)

### 3. IKEv2 Phase 1 Status

Checking `show crypto ikev2 sa` on `CE1-A` verifies successful negotiation with `CE2-A` in the active `READY` state:
![IKEv2 SA](screenshots/crypto-ikev2.png)

### 4. IPsec Phase 2 SA & Packet Counters

Inspection of `show crypto ipsec sa` proves active encryption and decryption, with monotonically increasing `pkts encaps` and `pkts decaps` counters on `Tunnel0`:
![IPsec SA Counters](screenshots/ipsec-tunnel.png)

### 5. Data Plane Reachability & Multi-Tenant Isolation

Ping tests from `CE1-A` confirm smooth end-to-end reachability across the encrypted tunnel (`10.99.0.2`), while packets targeting Tenant B (`10.99.1.2`) are dropped, proving strict multi-tenant boundary enforcement:
![Ping Verification](screenshots/ping-ce1-a.png)

---

## Configuration Files

All device configurations are stored in the [`configs/`](./configs/) directory:

- **Provider Edge:** [`PE-1.cfg`](./configs/PE-1.cfg), [`PE-2.cfg`](./configs/PE-2.cfg)
- **Provider Core:** [`P-1.cfg`](./configs/P-1.cfg), [`P-2.cfg`](./configs/P-2.cfg)
- **Customer Edge (Tenant A):** [`CE1-A.cfg`](./configs/CE1-A.cfg), [`CE2-A.cfg`](./configs/CE2-A.cfg)
- **Customer Edge (Tenant B):** [`CE1-B.cfg`](./configs/CE1-B.cfg), [`CE2-B.cfg`](./configs/CE2-B.cfg)
