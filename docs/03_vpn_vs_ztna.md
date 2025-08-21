# 🛡️ Legacy VPN vs Zero Trust (ZTNA)

| Feature         | Legacy VPN (e.g., Ivanti) | Zero Trust (e.g., Zscaler ZPA) |
|-----------------|---------------------------|--------------------------------|
| Access model    | Network-level (full tunnel) | App-level (per-app tunnel) |
| Trust model     | On VPN = trusted          | Never trust, always verify |
| Device posture  | Often not enforced        | Always verified |
| Risk            | Lateral movement possible | Scoped to specific app |
| Best for        | Legacy/internal apps      | SaaS, cloud, hybrid apps |
