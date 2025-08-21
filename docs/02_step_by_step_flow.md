# 🔑 Step-by-Step Flow

## 1) Endpoint Lifecycle (Boot to Login)
- Firmware/UEFI with Secure Boot + TPM.
- OS init: drivers, services, EDR/AV, firewall.
- Device identity: AD/Entra join, MDM enforcement.
- 802.1X + NAC: VLAN/ACL assignment.
- DHCP & NTP: IP, DNS, PAC, and time sync.

## 2) Identity & SSO
- On-prem: Kerberos tickets.
- Cloud: OIDC/SAML tokens via IdP (Okta, Entra ID).
- Policies: MFA, Conditional Access, device posture.

## 3) Outbound Web Request
- DNS resolution via corp resolver.
- Proxy decision (PAC/WPAD → DIRECT/PROXY).
- Forward Proxy / SWG: filtering, DLP, TLS interception.
- NGFW/IPS: App-ID, NAT, geo-blocking.
- SD-WAN/SASE: SaaS path optimization.

## 4) Internet & Destination
- ISP → CDN edge (Anycast) → origin servers.

## 5) Inbound to Application
- Reverse Proxy / LB terminates TLS.
- WAF filters HTTP/S (OWASP Top 10).
- Service Mesh: mTLS east-west.
- App: Auth tokens validated, DB/cache accessed.

## 6) Observability & Response
- Endpoint logs (Sysmon, EDR).
- Proxy logs (URL, SSL inspect, user).
- Firewall logs (NAT, App-ID, threats).
- WAF logs (rules, request IDs).
- SIEM/SOAR correlation.
