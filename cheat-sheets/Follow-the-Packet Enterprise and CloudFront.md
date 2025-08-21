# 🧾 Follow-the-Packet (Enterprise + CloudFront)

---

## 1. Endpoint (Defender for Endpoint)

**Event**: `chrome.exe` → TCP 443 → Zscaler edge node.  

**Logs**:  
- `DeviceNetworkEvents` shows outbound session.  
- `DeviceDnsEvents` shows query for `app.example.com`.  
- Alerts if blocked by SmartScreen/Network Protection.  

👉 **First visibility**: who (user), what (process), where (dest).  

---

## 2. DNS (Umbrella)

- `app.example.com` resolves to a **CloudFront distribution domain** (e.g., `d123.cloudfront.net`).  
- Umbrella policy may allow or block depending on category/reputation.  

**Logs**:  
- Domain, source IP, identity, policy action.  

👉 **DNS-layer control**, shows which CDN hostname the app points to.  

---

## 3. Proxy / SWG (Zscaler Internet Access)

- Browser obeys PAC → forwards to Zscaler proxy.  

**Zscaler logs**:  
- URL: `https://app.example.com`.  
- Category: Business.  
- SSL inspection: ✅.  
- User: `lee.cole`.  
- DLP verdict: none.  
- Threat sandbox: if file downloaded, detonated in Zscaler Sandbox.  

👉 **Zscaler enforces outbound policy, DLP, and identity attribution.**  

---

## 4. Egress Firewall (Palo Alto NGFW)

- Sees GRE/IPSec tunnel traffic between Zscaler ↔ internet, or direct outbound sessions if Zscaler bypassed.  

**Logs**:  
- NAT public IP.  
- App-ID: `ssl` or `web-browsing`.  
- Rule hit.  
- IPS/Threat verdict.  

👉 **Confirms NAT mapping and perimeter enforcement.**  

---

## 5. Internet Transit

- Session leaves enterprise → ISP → public internet → **AWS CloudFront POP (edge node)** close to user.  
- CloudFront edge chosen by **Anycast routing + DNS**.  

👉 **First hop inside AWS global CDN network.**  

---

## 6. AWS CloudFront

- **TLS Termination**: CloudFront presents TLS cert for `app.example.com`.  

**Edge protections**:  
- Optional **AWS WAF** integrated with CloudFront (SQLi/XSS/bot mitigation).  
- **Shield Advanced**: DDoS protection.  
- **Rate limiting** rules (if configured).  

**Behavior**:  
- If object cached → serve from edge cache.  
- If miss → forward to origin (e.g., S3 bucket, ALB, API Gateway, or custom app server).  

**Logs**:  
- CloudFront access logs: timestamp, edge location, client IP, request, response, cache status.  
- WAF logs: rule matched, action taken.  

👉 **Critical control point for scaling, performance, and inbound security.**  

---

## 7. Application / Origin (Behind CloudFront)

Could be:  
- ALB → App servers → DB/cache.  
- S3 bucket (static content).  

**Auth**: Validates OIDC/JWT from Azure AD or Okta.  

**Logs**: App logs (200 OK, latency, user identity).  

---

## 8. Splunk (SIEM Correlation)

**Splunk stitches it all together**:  
- Defender connector → process, network telemetry.  
- Umbrella logs → DNS resolution (domain → CloudFront POP).  
- Zscaler logs → user identity, proxy verdict, SSL inspection.  
- Palo Alto logs → NAT, App-ID, rule enforcement.  
- CloudFront/WAF logs → request path, cache hit/miss, edge location, WAF verdict.  
- App logs → user authenticated, latency, DB calls.  

👉 **Correlation query in Splunk ties request by time + user + URL.**  

Example SPL:  

```spl
(index=defender OR index=umbrella OR index=zscaler OR index=panfw OR index=cloudfront OR index=app)
| eval user=coalesce(UserPrincipalName, user, src_user)
| search url="https://app.example.com" OR domain="app.example.com"
| stats earliest(_time) as first latest(_time) as last values(user) values(src_ip) values(nat_src_ip) values(edge_location) values(cache_status) values(appid) values(action) by url
