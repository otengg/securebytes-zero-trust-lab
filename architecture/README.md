# Architecture

## Objective

Provide secure, identity-based access to internal infrastructure using a Zero Trust model with no public exposure.

---

## High-Level Topology

\`\`\`
Remote Client
      │
      ▼
Tailscale (WireGuard Overlay)
      │
      ▼
Subnet Router (Pi-hole)
      │
      ▼
192.168.10.0/24 Network
      │
      ▼
Core Services
\`\`\`

---

## Core Components

### Remote Client
- Authenticated device (Mac, laptop, mobile)
- Connects via Tailscale identity — no credentials, no static keys

### Tailscale Overlay
- WireGuard-based encrypted mesh
- Enforces ACL-based access control per node tag
- No traffic permitted without valid identity

### Subnet Router (Pi-hole)
- Advertises \`192.168.10.0/24\` into the Tailscale overlay
- Performs NAT for return traffic back to remote clients
- Provides DNS filtering for all enrolled nodes

### Internal Network
- Subnet: \`192.168.10.0/24\`
- Private and non-routable from the internet
- Only reachable via authenticated overlay nodes

### Core Services

| Service | Role | IP |
|---|---|---|
| Proxmox | Infrastructure management | 192.168.10.10 |
| Grafana | Observability dashboard | 192.168.10.21 |
| Prometheus | Metrics collection | 192.168.10.21 |
| Pi-hole | DNS filtering + subnet router | 192.168.10.5 |

---

## Access Flow

\`\`\`
Client → Authenticate → Tailscale Tunnel → ACL Evaluation → Subnet Router → Internal Service
\`\`\`

---

## Access Control Model

Default policy: **deny all**

| Tag | Access |
|---|---|
| \`tag:admin\` | Full subnet — \`192.168.10.0/24:*\` |
| \`tag:dns\` | DNS only — \`192.168.10.5:53\` |
| \`tag:monitor\` | Observability — Grafana \`:3000\`, Prometheus \`:9090\` |

Full policy: [\`configs/tailscale/acl.json\`](../configs/tailscale/acl.json)

---

## Design Decisions

- **No traditional VPN** — subnet routing avoids full tunnel exposure
- **No port forwarding** — zero public-facing services or inbound rules
- **Centralized entry** — all overlay traffic enters through the Pi-hole subnet router
- **Identity-based segmentation** — access scoped per tag, not per IP or firewall rule

---

## Scope Note

This architecture documents the secure access and observability layer of the SecureBytes lab. Additional components — SIEM, Kubernetes, CML, and network automation — are maintained in separate repositories.
