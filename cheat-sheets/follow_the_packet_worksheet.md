# Follow-the-Packet Worksheet (From Endpoint to Web App)

**How to use:** For a single page load (e.g., `https://app.example.com/dashboard`), fill each section with concrete facts (IPs, certs, policy hits, logs). Repeat on corp LAN, VPN, and off-net.

> [!NOTE]
> the Follow-the-Packet Worksheet is a guided lab exercise + checklist to help you trace one single request (like opening https://app.example.com) through every layer of the enterprise stack. It’s not just theory — it’s a way to force yourself to look at real logs, configs, and telemetry step by step. Think of it as a detective sheet.

> ## 0) Scenario metadata
- Date/time (UTC & local):
- User & device name / asset tag:
- Network (LAN/VPN/Off-net/Guest):
- App URL/path:

## 1) Endpoint
- Process/Parent (e.g., `chrome.exe` PID):
- Local firewall rule matched:
- EDR network event ID(s):
- Local DNS cache entry / host file overrides:

## 2) Access & addressing
- 802.1X method & RADIUS result:
- VLAN / Subnet / Gateway:
- DHCP lease (incl. Option 252 PAC, if used):
- NTP source & skew:

## 3) Name resolution
- Resolver IP(s):
- Query/Answer(s) (A/AAAA/CNAME):
- Split-horizon? (internal vs public):
- DNS security/filter verdicts:

## 4) Proxy decision
- PAC/WPAD result: `DIRECT` or `PROXY host:port` (show PAC line):
- Authentication to proxy (Kerberos/NTLM/SAML):
- TLS inspection? (Yes/No, root CA thumbprint, bypass reason if any):

## 5) Egress security
- NGFW policy ID/app-ID verdict:
- NAT public IP / port:
- IPS signature(s) or threat logs:
- SWG/DLP verdicts; file hash if download:

## 6) Internet/CDN
- CDN POP / Anycast IP:
- RTT & path (traceroute/MTR):
- TLS version & cipher negotiated to origin:

## 7) Provider edge (if you own it or from logs)
- Reverse proxy/LB VIP:
- WAF rule(s) matched / bot score:
- X-Forwarded-For chain:

## 8) Application & data
- Auth subject (OIDC sub/UPN) & scopes/roles:
- HTTP status codes & latencies (TTFB, total):
- DB/caching events (if observable):
- Error traces / correlation IDs:

## 9) Observability & correlation
- Correlate timestamps across Endpoint → Proxy → NGFW → WAF → App → IdP.
- Attach log excerpts and dashboards/screenshots.
- Final verdict: Was policy correct? Performance bottleneck? Security risk?

**Pro tips:** Maintain a bypass list for pinned sites; always capture corp-root thumbprint; note when ZTNA/SASE steers traffic to a POP vs. on-prem egress.
