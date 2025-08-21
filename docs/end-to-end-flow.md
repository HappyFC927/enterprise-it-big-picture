# 🌐 Enterprise IT Big Picture — End-to-End Flow

This document shows how everything works together in a corporate setup — from powering on a machine, to accessing a website, including proxies, firewalls, identity, and cloud.

---

## 1) From Power Button to a Managed, Networked Workstation

### 🔹 Firmware & Trust Anchor
- **UEFI/BIOS** initializes hardware, finds bootloader (disk or PXE/iPXE).
- **Secure Boot / Measured Boot** validates components; **TPM** stores measurements (used by BitLocker & health attestation).
- **Disk encryption** (BitLocker/FileVault/LUKS) unlocks via TPM-sealed keys + user PIN/policy.

### 🔹 OS Initialization
- Kernel loads drivers, mounts filesystems, starts services.
- **Device identity**: domain-joined (AD) or Entra/Azure AD joined.
- Endpoint registers with **MDM/EMM** (Intune, Workspace ONE) & config manager (SCCM/MECM).
- **EDR/AV** (Defender, CrowdStrike, etc.) loads early for tamper protection.

### 🔹 Network Bring-up & Access Control
- NIC/Wi-Fi associates; **802.1X (EAP-TLS/PEAP)** asks RADIUS (NPS/Ise/ClearPass) to validate certs.
- **NAC posture**: AV present? disk encrypted? patches current?  
  → Assign VLAN/ACLs. Fallback VLAN (guest/quarantine) if fails.
- **DHCP** leases IP + options (router, DNS, PAC file URL).
- **NTP** sync ensures accurate time (critical for Kerberos/TLS/tokens).

### 🔹 Directory, Certificates, and Policy
- **GPO/MDM policies** apply: firewall rules, BitLocker settings, proxy config, rights, app lists.
- **Enterprise PKI** (ADCS, SCEP/NDES) issues certs for 802.1X, mTLS, S/MIME, code signing.

### 🔹 User Sign-in & SSO
- **On-prem**: Kerberos (TGT via AS-REQ/AS-REP; service tickets via TGS).
- **Cloud/Hybrid**: OIDC/SAML with IdP (Entra/Okta/Ping) + MFA/Conditional Access.
- Result: tokens or Kerberos tickets enable SSO to apps, proxies, VPN/ZTNA, Wi-Fi.

---

## 2) How a Web Request Flows (with Proxy & Firewall)

Let’s say you browse `https://app.example.com` from the office.

### A) Name Resolution
- Browser → system resolver → **Enterprise recursive DNS**.  
- **Split-horizon**: internal names resolve internally, public go upstream.  
- DNS filtering may block malicious domains.

### B) Proxy Decision
- **PAC file** (auto-configured via GPO/MDM/DHCP/WPAD) returns `DIRECT` or `PROXY proxy.corp:3128`.
- **Modes**:
  - Explicit proxy: browser connects directly.
  - Transparent proxy: intercepted (rare for HTTPS due to pinning).
- Internal sites may bypass proxy, going straight to NGFW.

### C) Outbound Security Stack (Egress)
- **NGFW**: Stateful L3–L4, app-ID, enforces categories/ports/geo; performs SNAT.
- **Forward Proxy / Secure Web Gateway (SWG)**:
  - URL filtering, DLP, malware scanning, sandbox.
  - **TLS interception**: proxy presents a resigned cert (signed by corp root CA).
  - Exceptions: banking, pinned apps, privacy-sensitive.
  - Proxy auth: Kerberos/NTLM/SAML for identity-based policies.
- **IDS/IPS**: inline or NGFW-based.
- **SD-WAN/SASE**: SaaS traffic may be steered directly to nearest PoP.

### D) Internet & Destination
- Request → ISP → peering → **CDN edge (Anycast)** near user.
- DNS likely returned a CDN hostname.

### E) Inbound Stack at Provider
- **Reverse Proxy / Load Balancer** (NGINX/Envoy/ALB) terminates TLS (maybe mTLS).
- **WAF** enforces OWASP Top 10 protections, bot mgmt, API schema checks.
- **Service mesh** secures east-west in Kubernetes.
- **AuthN/AuthZ**: Tokens validated (OIDC JWT) or via auth proxy headers.
- **App tier**: caches (Redis), DB (SQL/NoSQL), object storage.
- **Logs/metrics/traces** → SIEM/Observability (Splunk/Sentinel/Datadog).

### F) Response Back
- Data: WAF → reverse proxy → CDN → Internet → egress → proxy/NGFW → browser.
- Every hop can log user/device identity for audit & IR.

---

## 3) Where Proxies & Firewalls Live

### 🔹 Forward Web Proxy (User → Internet)
- Controls outbound web.  
- Enforces URL filtering, DLP, malware scan, sandbox, identity-based rules.  
- **Placement**: after core/NGFW, before egress.  
- Requires **corp root CA** for TLS intercept.  
- Auth: seamless (Kerberos/NTLM) or explicit (SAML).

### 🔹 Reverse Proxy / Load Balancer (Internet → App)
- Front door for inbound traffic.  
- Handles TLS offload, L7 routing, rate limiting, WAF integration, mTLS to backends.

### 🔹 Firewalls (NGFW)
- **Egress**: Stateful rules, app-ID, IPS, URL filtering, SNAT.  
- **Ingress (DMZ)**: Publishes only required services; pairs with WAF.  
- **Microsegmentation**: host-based or overlay (Illumio, NSX, etc.).

### 🔹 Other Security Layers
- **IDS/IPS/NDR**: Detect anomalies.  
- **CASB/SSE/SASE**: SaaS governance.  
- **ZTNA**: replaces VPN, per-app access.  
- **DLP**: monitor sensitive data.

---

## 4) Corporate Network Architecture

- **Access Layer**: switches/APs with 802.1X, VLAN, QoS.  
- **Distribution/Core**: routing (OSPF/BGP), VRFs, ACLs, NetFlow.  
- **Data Center / DMZ**: server VLANs, virtual clusters, internet-facing DMZ.  
- **WAN**: MPLS or SD-WAN with path steering.  
- **Internet Edge**: NGFW, SWG, DNS resolvers, mail gateways.  
- **Identity**: AD/Entra/Okta, PKI, HSMs.  
- **Mgmt Plane**: OOB networks, bastions, PAM.  
- **Logging & SOC**: Syslog/CEF to SIEM, SOAR automation.  

---

## 5) Cloud & SaaS Patterns

- **IaaS (AWS/Azure/GCP)**: VPC/VNet, SGs/NSGs, NACLs, IGWs, NAT GWs, private endpoints, transit gateways, cloud WAF, IAM/KMS.  
- **Kubernetes**: Ingress, service mesh (mTLS), secrets vaults, admission controllers.  
- **SaaS**: SAML/OIDC SSO, SCIM provisioning, CASB, audit exports.  
- **Zero Trust**: ZTNA broker validates identity + posture continuously.  

---

## 6) DevOps / IT Ops Flows

- **CI/CD**: Build, test, SAST/DAST, sign artifacts.  
- **IaC**: Terraform/Ansible, GitOps (ArgoCD/Flux).  
- **Secrets**: Vault/KMS, short-lived tokens.  
- **Release**: canary/blue-green, WAF rule updates.  
- **Observability**: Prometheus, ELK, OTel, Sentinel.  

---

## 7) OSI Model Cheat-Sheet

- **L1–L2**: Cabling, Wi-Fi, VLANs, MAC, ARP, 802.1X.  
- **L3**: IP, routing, VRFs, ACLs, NAT.  
- **L4**: TCP/UDP, ports, firewalls, load balancers.  
- **L5–L6**: TLS handshakes, certs, encryption.  
- **L7**: HTTP(S), REST, WAF, proxies, app auth.  

---

## 8) TLS & Interception

- **Direct TLS 1.3**: ClientHello (SNI/ALPN), server cert, key exchange → encrypted.  
- **Intercepted**:  
  - Client ↔ Proxy (proxy cert signed by corp root).  
  - Proxy ↔ Server (real public cert).  
  - Proxy inspects traffic, re-encrypts.  
- **Pinning/Privacy**: Some apps bypass due to pinning/cert transparency.  

---

## 9) Logs for a Single Request (`GET /dashboard`)

- **Endpoint**: Sysmon/Defender shows `chrome.exe` → TCP 443 → proxy.  
- **Proxy**: user `lee.cole`, device `WIN10-123`, category “Business”, action “Allowed”.  
- **NGFW**: NATed session, app-ID “HTTPS”, bytes in/out.  
- **WAF**: rules matched, bot score, TLS version, client IP.  
- **App**: HTTP 200, latency, OIDC subject.  
- **SIEM**: correlated timeline, UEBA, anomaly alerts.  

---

## 10) Remote Access Variants

- **Legacy VPN**: Full/split tunnel; NGFW policies; posture checks.  
- **ZTNA**: Broker verifies posture, creates app-specific tunnel; no flat access.  
- **SASE/SSE**: Roaming agent forwards traffic to cloud SWG/Firewall PoP.  

---

## 11) Data Protection & Resilience

- **Backups & snapshots**: on-prem + cloud, immutability, off-site.  
- **RPO/RTO** define recovery designs (active-active vs pilot light).  
- **Email security**: MX → secure gateway → spam/malware/DLP → user. Outbound: DKIM/DMARC/SPF.  

---

## 12) Quick Glossary

- **Forward Proxy**: Outbound web control.  
- **Reverse Proxy**: Inbound app routing/protection.  
- **NGFW**: Stateful firewall + IPS + app awareness.  
- **WAF**: L7 HTTP(S) firewall for apps.  
- **CASB/SSE/SASE**: SaaS & cloud security.  
- **ZTNA**: Per-app Zero Trust access.  
- **NAC/802.1X**: Gate device access to LAN.  
- **MDM/Intune**: Endpoint compliance & config.  
- **SIEM/SOAR**: Centralized detection & automated response.  
- **IdP (OIDC/SAML)**: SSO and identity tokens.  
- **PKI/CA/HSM**: Certificates, encryption, key custody.  
