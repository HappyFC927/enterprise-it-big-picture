# 🔑 Step-by-Step Flow

## 1) Endpoint Lifecycle (Boot to Login)
- Firmware/UEFI runs Secure Boot, checks integrity (TPM).
- OS init: drivers, services, EDR/AV, firewall.
- Device identity: AD/Entra join, Intune/MDM policy enforcement.
- 802.1X + NAC: Device/user checked, assigned VLAN/ACL.
- DHCP & NTP:Get IP, DNS, PAC file (for proxy), and sync time.

## 2) Identity & SSO
- On-prem: Kerberos tickets via Domain Controller.
- Cloud: OIDC/SAML tokens via IdP (Okta, Entra ID, Ping).
- Policies: MFA, Conditional Access, device posture.

## 3) Outbound Web Request
- DNS resolution via enterprise resolver (with filtering).
- Proxy decision (PAC/WPAD → DIRECT/PROXY).
- Forward Proxy / Secure Web Gateway (SWG):
  - URL filtering, DLP, malware sandboxing.
  - TLS interception (corp root CA required).
- NGFW/IPS: App-ID, geo-block, SNAT, threat detection.
- SD-WAN/SASE: SaaS path optimization.

## 4) Internet & Destination
- Request hits ISP → CDN edge (Anycast).
- CDN routes traffic to nearest healthy POP.

## 5) Inbound to Application
- Reverse Proxy / Load Balancer terminates TLS, does routing.
- WAF filters HTTP(S) traffic (SQLi, XSS, bot mgmt) (OWASP Top 10).
- Service Mesh: secures east-west traffic in Kubernetes/microservices.
- App layer: Auth via tokens, queries DB/cache, returns data. Auth tokens validated, DB/cache accessed.

## 6) Observability & Response
- Endpoint logs: Sysmon/Defender network telemetry.
- Proxy logs: User, URL, action, SSL inspection status.
- Firewall logs: NAT, App-ID, threats.
- WAF logs: Rules triggered, request IDs.
- SIEM/SOAR: Correlates all layers, triggers alerts, automates response.

🔒 Key Security Components
- Forward Proxy (User → Internet): URL filtering, TLS intercept, DLP.
- reverse Proxy (Internet → App): TLS offload, routing, identity integration.
- NGFW: L3–L7 firewall, IPS, NAT.
- WAF: Protects web apps against OWASP Top 10.
- ZTNA / SASE: Identity-based app access for remote users.

📚 OSI Model Cheat Sheet
- L1–L2: Cables, Wi-Fi, VLANs, MAC, ARP, 802.1X.
- L3: IP, routing, ACLs, NAT.
- L4: TCP/UDP, ports, firewalls, load balancing.
- L5–6: TLS/SSL, encryption, certificates.
- L7: HTTP(S), REST APIs, WAF, proxies, auth.

✅ Summary
- Boot chain (TPM/Secure Boot) ensures trust.
- Endpoint joins directory & enforces policies via MDM.
- 802.1X/NAC gate network access and segmentation.
- Proxies and firewalls control and log outbound traffic.
- WAFs, reverse proxies, and service meshes secure inbound and east-west flows.
- SIEM/SOAR ties it all together for detection and response.


