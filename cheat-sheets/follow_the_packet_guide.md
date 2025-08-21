# 🧾 Follow-the-Packet Worksheet Guide

The **Follow-the-Packet Worksheet** is designed to help you trace **one single web request** (e.g., visiting `https://app.example.com`) through every layer of a corporate IT environment.  

It’s both a **training tool** and a **diagnostic playbook**: you’ll learn where to look for logs, how to validate controls, and how to correlate events end-to-end.

---

## 🎯 Goals of the Worksheet
- **Correlate everything**: from endpoint process → DNS → proxy → firewall → WAF → app.  
- **Validate controls**: confirm NAC, proxy, TLS interception, and policies are actually working.  
- **Spot failures**: pinpoint where traffic is blocked, broken, or delayed.  
- **Build SOC muscle memory**: train yourself to always ask *“where would I see this in logs?”*.

---

## 🛠️ Step-by-Step Sections

### 1. Endpoint
- Identify the process (`chrome.exe`) that initiated the request.  
- Check **EDR/Sysmon** logs for local connection details (PID, local IP, dest IP/port).  
- Look at the **local DNS cache** or `hosts` file.  

👉 *Proves*: which app made the connection and what it was trying to reach.

---

### 2. Network Access
- Was **802.1X authentication** successful?  
- Which **VLAN/subnet** were you placed in?  
- DHCP lease info (router, DNS, PAC file option).  
- NTP sync status.  

👉 *Proves*: device compliance and correct segmentation.

---

### 3. DNS
- Which **resolver** did you query?  
- What **record(s)** were returned (A/AAAA/CNAME)?  
- Was the domain blocked or allowed by DNS security?  

👉 *Proves*: correct resolution and DNS filtering control.

---

### 4. Proxy Decision
- Did the **PAC file** return `DIRECT` or `PROXY proxy.corp:3128`?  
- Did the proxy request authentication (Kerberos/NTLM/SAML)?  
- Was TLS **intercepted**? Which cert was presented?  

👉 *Proves*: whether outbound web traffic was inspected and tied to identity.

---

### 5. Egress Firewall / NGFW
- What **public IP (NAT)** was used?  
- Which **firewall rule** matched?  
- Was traffic identified correctly (App-ID)?  
- Any **IPS signatures** triggered?  

👉 *Proves*: how traffic leaves the enterprise and if policies applied.

---

### 6. Internet / CDN
- Which **CDN edge IP** did you hit?  
- Results of a **traceroute/MTR**.  
- TLS version and cipher negotiated.  

👉 *Proves*: external path and cryptographic security.

---

### 7. Reverse Proxy / WAF
- Which **VIP/load balancer** served the request?  
- Did the **WAF** trigger rules (SQLi/XSS/bot detection)?  
- Check the **X-Forwarded-For (XFF)** chain.  

👉 *Proves*: inbound protections are applied.

---

### 8. Application & Identity
- Was the **OIDC/JWT token** validated?  
- Which **user identity** was mapped?  
- App logs: HTTP status code, latency, DB/cache hits.  

👉 *Proves*: authentication and app-layer response.

---

### 9. Observability & SIEM
- Line up **timestamps** across all sources.  
- Can you build one **timeline** of the request?  
- Do SIEM/UEBA correlation rules match the event?  

👉 *Proves*: whether your detection and monitoring pipeline is complete.

---

## ✅ End Result
When completed, you’ll have:
- A **full request timeline** across endpoint → proxy → firewall → app.  
- Proof of which **security controls** enforced policy.  
- A map of where to look when something fails.  
- A **reusable SOC training exercise** for onboarding analysts.

---

## 📌 Example Scenario (Quick)
1. User opens `https://app.example.com`.  
2. Sysmon shows `chrome.exe` PID 2345 → proxy.corp:3128.  
3. DNS returns `app.cdn.net` → 203.0.113.45.  
4. Proxy log: user `lee.cole`, SSL inspected, category “Business”, action Allowed.  
5. NGFW log: NATed to 198.51.100.10, App-ID “HTTPS”.  
6. WAF log: Rule matched = none, response 200.  
7. App log: Auth subject `lee.cole@corp.com`, latency 120ms.
