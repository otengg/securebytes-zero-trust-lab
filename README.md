# 🔐 SecureBytes — Zero Trust Lab

> Identity-based access to internal infrastructure via Tailscale overlay network.
> No public exposure. No legacy VPN.

![WireGuard](https://img.shields.io/badge/WireGuard-Overlay-orange?style=flat-square)
![Tailscale](https://img.shields.io/badge/Tailscale-ZeroTrust-blue?style=flat-square)
![Subnet](https://img.shields.io/badge/Subnet-192.168.10.0%2F24-lightgrey?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## Overview

This repository documents the design and implementation of a Zero Trust access model for a private lab environment using Tailscale.

Access is granted based on **identity and policy** — not network location. Services remain private and unreachable from the public internet, with no inbound ports opened on the edge.

---

## Architecture

```
Remote Client
      │
      ▼
Tailscale (WireGuard Overlay)   ← identity-verified transport
      │
      ▼
Subnet Router (Pi-hole)         ← route advertisement + NAT
      │
      ▼
192.168.10.0/24
      │
      ▼
Internal Services
```

---

## Design Principles

| Principle | Description |
|---|---|
| **Zero Trust Access** | Access granted on identity and policy, never network location |
| **No Public Exposure** | Zero inbound ports opened on the edge network |
| **Minimal Attack Surface** | Services only reachable through the overlay network |
| **Policy Segmentation** | Tag-based ACLs restrict access at the service level |

---

## Network Scope

| Parameter | Value |
|---|---|
| Internal subnet | `192.168.10.0/24` |
| Access method | Tailscale overlay network |
| Entry point | Subnet router (Pi-hole) |

---

## Access Control Model

Access policies are enforced using Tailscale ACLs with tag-based segmentation.

| Tag | Scope | Services |
|---|---|---|
| `admin` | Full subnet | `192.168.10.0/24` — all hosts |
| `dns` | DNS service | Pi-hole — port `53` only |
| `monitor` | Observability stack | Grafana, Prometheus |

---

## Implementation

- [x] Tailscale deployed across all client and lab nodes
- [x] Subnet routing configured via Pi-hole node
- [x] NAT implemented for return traffic handling
- [x] Tag-based ACL policies applied for segmentation
- [x] Remote access validated from external networks

---

## Operational Notes

- Tailscale auth state is independent of system uptime — re-auth on reboot is expected behavior
- Subnet routing requires both route advertisement on the router **and** client-side acceptance
- NAT is required when upstream routing has no awareness of the overlay network space
- Tag consistency must be maintained when re-authenticating nodes — drift breaks ACL enforcement

---

## Current Capabilities

- Remote SSH access to internal hosts
- Secure access to observability services (Grafana, Prometheus)
- Identity-based network segmentation
- No dependency on traditional VPN infrastructure

---

## Next Steps

| # | Item | Notes |
|---|---|---|
| 1 | Private service publishing | `securebytes.net` · custom domain routing |
| 2 | Reverse proxy integration | Caddy or NGINX · TLS termination |
| 3 | Identity provider integration | SSO / Entra ID · federated auth |
| 4 | Multi-site expansion | Additional subnets · mesh topology |

---

## Author

**Gideon Oteng** — [SecureBytes](https://github.com/securebytes)
