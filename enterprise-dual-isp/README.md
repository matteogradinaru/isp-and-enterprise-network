# Topology 1: Enterprise Multi-Homed Network & VRF-Lite

This lab models an enterprise campus network with dual upstream Internet service providers, an IS-IS IGP backbone, dynamic BGP traffic engineering, and VRF-Lite network virtualization for tenant isolation.

---

## Topology Architecture

![Topology 1 Architecture](screenshots/topology1.png)

---

## Design & Configuration Details

### 1. IS-IS Underlay Routing

- **Hierarchy:** Implemented a two-tier IS-IS routing domain (`NetworkByte`). Distribution routers (`DIST-R1`, `DIST-R2`) operate strictly as **Level-1**, while edge routers (`EDGE-R1`, `EDGE-R2`) operate as **Level-2**.
- **L1/L2 Boundary:** Core routers act as Level-1-2 border routers. `CORE-R1` generates the default route and summarizes internal distribution routes (`10.10.0.0/16`) into Level-2.
- **Metrics:** Configured `metric-style wide` across all nodes to support 32-bit link metrics and facilitate modern traffic engineering.

### 2. Dual-Homed BGP & Traffic Engineering

The enterprise resides in private `AS 65100` and connects to two distinct providers:

- **ISP-A Peering (`AS 65001`):** Serves as the primary upstream connection. Inbound routes received on `EDGE-R1` are tagged with `Local-Preference 200`.
- **ISP-B Peering (`AS 65002`):** Serves as backup transit. Regular routes are tagged with `Local-Preference 100`. Inbound paths are discouraged by prepending the enterprise AS three times (`65100 65100 65100`) on advertisements out to ISP-B.
- **Policy Exception:** A specific prefix policy sends traffic destined for `9.0.0.0/8` out via ISP-B by raising its `Local-Preference` to `300`.
- **Internal Peering:** Full iBGP peering between `EDGE-R1`, `EDGE-R2`, and `CORE` nodes using Loopback addresses and `next-hop-self`.

### 3. Multi-Tenant VRF-Lite Isolation

To support multi-tenancy without leaking routes between customers:

- `CORE-R1` defines three independent VRF instances: `KLANT-A`, `KLANT-B`, and `KLANT-C`.
- Each customer router connects on dedicated subnets (`192.168.100.0/30`, `192.168.200.0/30`, `192.168.30.0/30`), yet all three customers can safely use the exact same internal subnet (`10.1.0.0/24`) without IP conflicts.

---

## Verification & Test Results

### BGP Peering Status

Internal iBGP and external eBGP peering sessions established and exchanging prefixes:<br>
![BGP Summary](screenshots/bgp-sessions.png)

### IS-IS Adjacencies and Routing Table

Checking IS-IS neighbor states confirms the expected Level-1 and Level-2 adjacencies across the campus:<br>
![IS-IS Neighbors](screenshots/isis-neighbors.png)

Routing table on `EDGE-R1` displaying learned IS-IS inter-area routes and summarized subnets:<br>
![IS-IS Routes](screenshots/ip-route-isis.png)

### Outbound Traffic Engineering Verification

- **Primary Route Test:** A traceroute to `8.8.8.1` confirms outbound traffic flows through `EDGE-R1` toward `ISP-A`:<br>
  ![Traceroute to 8.8.8.1](screenshots/traceroute-8-8-8-1.png)

- **Engineered Policy Route Test:** A traceroute to `9.9.9.1` verifies that the `Local-Preference 300` policy takes effect, steering traffic across the internal core over to `EDGE-R2` and out via `ISP-B`:<br>
  ![Traceroute to 9.9.9.1](screenshots/traceroute-9-9-9-1.png)

### VRF Route Isolation

Displaying the VRF routing tables on `CORE-R1` validates that customer routes remain strictly contained within their respective VRF instances:<br>
![VRF Table](screenshots/vrf-table.png)

---

## Configuration Files

All running configurations for this topology are available in the [`configs/`](./configs/) directory:

- **Edge Layer:** [`EDGE-R1.cfg`](./configs/EDGE-R1.cfg), [`EDGE-R2.cfg`](./configs/EDGE-R2.cfg)
- **Core Layer:** [`CORE-R1.cfg`](./configs/CORE-R1.cfg), [`CORE-R2.cfg`](./configs/CORE-R2.cfg)
- **Distribution Layer:** [`DIST-R1.cfg`](./configs/DIST-R1.cfg), [`DIST-R2.cfg`](./configs/DIST-R2.cfg)
- **ISP Upstreams:** [`ISP-A.cfg`](./configs/ISP-A.cfg), [`ISP-B.cfg`](./configs/ISP-B.cfg)
- **Customer Premises:** [`Router-A.cfg`](./configs/Router-A.cfg), [`Router-B.cfg`](./configs/Router-B.cfg), [`Router-C.cfg`](./configs/Router-C.cfg)
