# Architecture

## Network Topology

```
                    ┌─────────────────────────────────────┐
                    │       TAILSCALE OVERLAY MESH        │
                    │        100.x.x.x / WireGuard        │
                    └──────────────┬──────────────────────┘
                                   │
                        ┌──────────▼──────────┐
                        │    Remote Client     │
                        │    (any location)    │
                        └──────────┬──────────┘
                                   │ Tailscale tunnel
                        ┌──────────▼──────────┐
                        │     pihole-01        │
                        │   192.168.10.5       │
                        │   Subnet Router      │
                        │   + DNS filtering    │
                        └──────────┬──────────┘
                                   │ 192.168.10.0/24
       ┌───────────────────────────┼───────────────────────────┐
       │                           │                           │
┌──────▼────────┐     ┌────────────▼──────────┐   ┌───────────▼───────────┐
│  ava-dc-01    │     │  Observability Stack   │   │  CML / Automation     │
│ 192.168.10.10 │     │  .20 Security Onion    │   │  .40 CML 2.9          │
│  Proxmox VE   │     │  .21 Wazuh             │   │  .50 python-auto      │
│  Hypervisor   │     │  .22 Splunk            │   └───────────────────────┘
└───────────────┘     └────────────────────────┘

┌──────────────────────────────────────────┐
│  Kubernetes Cluster                      │
│  k8s-01  192.168.10.30                  │
│  k8s-02  192.168.10.31                  │
│  k8s-03  192.168.10.32                  │
└──────────────────────────────────────────┘
```

---

## Zero Trust Access Flow

```
1. Remote client authenticates to Tailscale (identity provider)
         │
         ▼
2. Tailscale issues WireGuard keypair, verifies node tags
         │
         ▼
3. Encrypted WireGuard tunnel established (overlay)
         │
         ▼
4. ACL evaluated — tag:admin / tag:monitor / tag:dns
         │
         ▼
5. Subnet router (pihole-01) forwards into 192.168.10.0/24
         │
         ▼
6. NAT applied for return traffic back to overlay client
         │
         ▼
7. Internal service reached — SSH / Grafana / Prometheus / DNS
```

---

## ACL Segmentation

```
┌──────────────────────────────────────────────────────┐
│               TAILSCALE ACL (deny by default)        │
│                                                      │
│  tag:admin   ────────────────► 192.168.10.0/24:*    │
│                                                      │
│  tag:dns     ────────────────► 192.168.10.5:53      │
│                                                      │
│  tag:monitor ────────────────► :3000  :9090  :9093  │
│                                                      │
│  (untagged)  ────────────────► DENY                 │
└──────────────────────────────────────────────────────┘
```

---

## Node Reference

| Hostname | Role | IP |
|---|---|---|
| `pihole-01` | Subnet router + DNS | 192.168.10.5 |
| `ava-dc-01` | Proxmox hypervisor | 192.168.10.10 |
| `sec-onion-01` | Security Onion (IDS/NSM) | 192.168.10.20 |
| `wazuh-01` | SIEM / EDR manager | 192.168.10.21 |
| `splunk-01` | Log aggregation | 192.168.10.22 |
| `k8s-01` | Kubernetes node | 192.168.10.30 |
| `k8s-02` | Kubernetes node | 192.168.10.31 |
| `k8s-03` | Kubernetes node | 192.168.10.32 |
| `cml-01` | Cisco Modeling Labs | 192.168.10.40 |
| `python-automation` | Network automation VM | 192.168.10.50 |
