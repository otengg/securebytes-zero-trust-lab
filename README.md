# 🔐 SecureBytes — Zero Trust Lab

> Identity-based access to a private homelab using Tailscale.
> No inbound ports. No legacy VPN. No implicit trust.

![WireGuard](https://img.shields.io/badge/Transport-WireGuard-orange?style=flat-square)
![Tailscale](https://img.shields.io/badge/Overlay-Tailscale-blue?style=flat-square)
![Zero Trust](https://img.shields.io/badge/Model-Zero%20Trust-brightgreen?style=flat-square)
![Subnet](https://img.shields.io/badge/Subnet-192.168.10.0%2F24-lightgrey?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-success?style=flat-square)

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Design Principles](#design-principles)
- [Access Control Model](#access-control-model)
- [Implementation](#implementation)
- [Operational Notes](#operational-notes)
- [Current Capabilities](#current-capabilities)
- [Visual Proof](#visual-proof)
- [Next Steps](#next-steps)
- [Documentation](#documentation)

---

## Overview

This repository documents the design and implementation of a Zero Trust access model for a private homelab using Tailscale as the overlay network.

Access is granted based on **identity and policy** — not network location. Services remain private and unreachable from the public internet, with no inbound ports opened on the edge.

---

## Architecture

```
Remote Client
      │
      │  WireGuard encrypted tunnel (Tailscale overlay)
      ▼
Tailscale Coordination + Identity Verification
      │
      │  Tag-based ACL evaluated
      ▼
Subnet Router — pihole-01 (192.168.10.5)
      │
      │  Route advertisement + NAT
      ▼
192.168.10.0/24 — Internal Services
```

![Zero Trust Architecture](assets/diagrams/zt-architecture.png)

---

## Design Principles

| Principle | Description |
|---|---|
| **Zero Trust Access** | Access granted on identity and policy — never network location |
| **No Public Exposure** | Zero inbound ports opened on the edge network |
| **Minimal Attack Surface** | Services only reachable through the overlay network |
| **Policy Segmentation** | Tag-based ACLs enforce least-privilege at the service level |

---

## Access Control Model

Policies are enforced using Tailscale ACLs. The model is **deny-by-default** — all access is explicit.

| Tag | Scope | Services |
|---|---|---|
| `tag:admin` | Full subnet | `192.168.10.0/24` — all hosts, all ports |
| `tag:dns` | DNS only | Pi-hole — port `53` UDP/TCP |
| `tag:monitor` | Observability stack | Grafana `:3000`, Prometheus `:9090`, Alertmanager `:9093` |

ACL source: [`configs/tailscale/acl.json`](configs/tailscale/acl.json)

---

## Implementation

- [x] Tailscale deployed across all client and lab nodes
- [x] Subnet routing configured via Pi-hole node with IP forwarding enabled
- [x] NAT implemented for return traffic to overlay clients
- [x] Tag-based ACL policies applied and validated
- [x] Remote access confirmed from external networks

---

## Operational Notes

- Tailscale auth state is independent of system uptime — nodes may require re-auth after reimage
- Subnet routing requires route advertisement on the router **and** `--accept-routes` on each client
- NAT is required when upstream devices have no awareness of the `100.x.x.x` overlay range
- Tag assignments must be reapplied after node re-authentication — ACL enforcement depends on tag consistency

---

## Current Capabilities

- Remote SSH access to internal hosts via overlay
- Secure access to observability stack (Grafana, Prometheus)
- Identity-based network segmentation with tag-scoped ACLs
- DNS filtering via Pi-hole across all enrolled nodes
- No dependency on traditional VPN infrastructure

---

## Visual Proof

### Tailscale Network Overview
![Tailscale Overview](assets/screenshots/tailscale-overview.png)

### Subnet Routing Configuration
![Subnet Routing](assets/screenshots/tailscale-subnet-routing.png)

### Remote SSH Access
![Remote SSH](assets/screenshots/remote-ssh-access.png)

### Observability — Grafana
![Grafana](assets/screenshots/grafana-overview.png)

### DNS Layer — Pi-hole
![Pi-hole](assets/screenshots/pihole-dns-overview.png)

---

## Next Steps

| # | Item | Notes |
|---|---|---|
| 1 | Private service publishing | `securebytes.net` · custom domain routing |
| 2 | Reverse proxy integration | Caddy or NGINX · TLS termination |
| 3 | Identity provider integration | SSO / Entra ID · federated authentication |
| 4 | Multi-site expansion | Additional subnets · mesh topology |

---

## Documentation

| Document | Description |
|---|---|
| [`docs/overview.md`](docs/overview.md) | Full architecture overview, topology, and tech stack |
| [`docs/setup-guide.md`](docs/setup-guide.md) | Step-by-step deployment guide |
| [`docs/troubleshooting.md`](docs/troubleshooting.md) | Common issues and diagnostic commands |
| [`configs/tailscale/acl.json`](configs/tailscale/acl.json) | Tailscale ACL policy |
| [`architecture/README.md`](architecture/README.md) | Network diagrams and access flow |

---

## Author

**Gideon Oteng** — [github.com/otengg](https://github.com/otengg)
