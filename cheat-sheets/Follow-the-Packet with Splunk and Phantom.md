# 🧾 Follow-the-Packet with Splunk + Phantom (SOAR)

This guide shows how telemetry is collected at each layer and how Phantom automates response playbooks.  

---

## 1. Endpoint (Defender for Endpoint)
- **Telemetry**: Process + network events (e.g., `chrome.exe` → proxy).  
- **Phantom automation**:  
  - If malicious hash/URL seen → auto-isolate host (`isolate machine` action).  

---

## 2. DNS (Umbrella)
- **Telemetry**: DNS query for `app.example.com` → CloudFront edge.  
- **Phantom automation**:  
  - If domain flagged → auto-add to Umbrella blocklist (`enforce policy`).  
  - Notify SOC channel.  

---

## 3. Proxy / SWG (Zscaler)
- **Telemetry**: Proxy log (user `lee.cole`, SSL inspected, Allowed).  
- **Phantom automation**:  
  - If URL in risky category or sandbox verdict = malicious → auto-block in Zscaler (`add URL category block`).  

---

## 4. Firewall (Palo Alto NGFW)
- **Telemetry**: NAT session, App-ID `ssl`, IPS logs.  
- **Phantom automation**:  
  - Push dynamic block lists (EDL) for malicious IPs.  
  - If repeated alerts → auto-quarantine VLAN rule.  

---

## 5. CDN/Edge (AWS CloudFront)
- **Telemetry**: CloudFront access/WAF logs → Splunk.  
- **Phantom automation**:  
  - If repeated attack from same IP → auto-create AWS WAF IP block rule via Phantom AWS connector.  
  - If cache poisoning attempt → alert cloud team.  

---

## 6. Application / Identity
- **Telemetry**: App log shows user `lee.cole@corp.com` authenticated, response 200.  
- **Phantom automation**:  
  - If suspicious login (geo anomaly) → trigger Azure AD conditional access update (force reauth/MFA).  

---

## 7. Splunk + Phantom (SIEM + SOAR Integration)
- **Splunk aggregates**: Defender, Umbrella, Zscaler, Palo Alto, CloudFront, App logs.  
- **Phantom consumes alerts via Splunk → runs playbooks**:  
  - **Enrichment**: WHOIS, VirusTotal, Censys lookup on suspicious IP/URL.  
  - **Containment**: Isolate device, block domain, update firewall/proxy rules.  
  - **Notification**: Auto-create ticket (ServiceNow/Jira) + Slack/Teams alert.  
  - **Orchestration**: Chain multiple systems (e.g., Defender isolate + Palo block + Umbrella block simultaneously).  

---

## 📌 Example Timeline with Phantom in Action

- **Defender**: `chrome.exe` → TCP 443 → Zscaler. *No alert.*  
- **Umbrella**: Query `app.example.com` → `d123.cloudfront.net`. *Allowed.*  
- **Zscaler**: URL allowed, SSL inspected.  
- **Palo Alto**: Allowed, NAT to 198.51.100.10.  
- **CloudFront**: Edge `IAD53-C1`, cache MISS, forwarded to ALB.  
- **App logs**: User `lee.cole@corp.com`, 200 OK.  

✅ **Incident scenario**: `app.example.com/dashboard` starts serving malware.  
- Defender flags suspicious file hash → sends to Splunk.  
- Splunk → Phantom playbook triggers:  
  - Auto-isolate endpoint in Defender.  
  - Block `app.example.com` in Umbrella + Zscaler.  
  - Push IP block to Palo Alto.  
  - Add IP to AWS WAF via CloudFront integration.  
  - Notify SOC via Slack & create ServiceNow incident.  
