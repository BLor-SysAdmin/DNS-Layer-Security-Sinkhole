# DNS-Layer Security Sinkhole & Core Threat Mitigation Pipeline

## Executive Architecture Overview
This security infrastructure portfolio showcases the deployment of a centralized, network-wide DNS-layer sinkhole engineered to intercept protocol-level vulnerabilities before malicious traffic can enter host OS spaces. In modern enterprise environments, client endpoints routinely leak data via silent tracking telemetry, adware insertion schemes, and known malicious command-and-control (C2) domains. This project resolves these issues by deploying a high-efficiency caching DNS proxy that matches outbound lookup queries against strict blocklist matrices, dropping threat vectors at the root protocol tier without introducing network latency.

## System Topology Blueprint
*   **Hosting Virtual Fabric:** Type-1 Proxmox VE Hypervisor Environment
*   **Compute Node State:** Low-overhead Linux Linux Container (Ubuntu Linux LXC)
*   **Core DNS Core Service Engine:** Pi-hole Core Domain Sinkhole Architecture
*   **Static IP Core Binding:** `10.0.1.53/24` (Corporate Management LAN Zone)
*   **Upstream Forwarder Routing:** Quad9 Cryptographic Resolvers (`9.9.9.9` with DNSSEC validation enabled)

## Technical Features Implemented
1.  **Microservices Container Isolation:** Deployed the service within a dedicated, unprivileged Linux LXC container structure, ensuring minimal host resource foot-printing while maintaining strict operational isolation from adjacent server nodes.
2.  **DHCP Option 6 Core Scope Injection:** Integrated the sinkhole into the core network routing domain by overwriting the **DHCP Option 6 (dns-server)** parameter scope inside the OPNsense core network gateway. This forces all newly leased client workstations to dynamically inherit `10.0.1.53` as their sole name resolution target.
3.  **Adlist Matrix Aggregation:** Compiled and synchronized comprehensive corporate security blocklists containing over 100,000 unique malicious entries, configuring wild-card blocking heuristics to trap malicious subdomains automatically.
4.  **Permit All Origins Perimeter Control:** Tuned the local interface listening behavior (`Permit all origins`) to securely handle lookup requests migrating inward from the multi-subnet isolated zones (`10.0.50.x` VLAN 50) and authenticated remote access tunnels (`10.50.0.x` WireGuard pool).

## DNS Name Resolution Flow Engine

```text
[Client Computer Workstation]
       │ (Outbound Web Lookup: e.g., ://malicious.com)
       ▼
[OPNsense DHCP Router Gateway] 
       │
       ▼
(Forces DNS Route via Option 6)
       │
       ▼
[Ubuntu Linux LXC Container (10.0.1.53)]
                                 │
        ┌────────────────────────┴────────────────────────┐  ▼                                                 ▼
[Domain on Security Blocklist]             [Safe Verified Domain]
        │                                                 │
(Packet Instantly Dropped)            (Forwarded to Upstream DNS)
        ▼                                                 ▼
Returns: 0.0.0.0 (Null)                Resolves to Public Web WAN
```