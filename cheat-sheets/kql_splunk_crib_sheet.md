# KQL + Splunk Crib Sheet (Trace a Web Request End-to-End)

> Replace placeholder fields with your environment's actual table names, product schemas, and labels.

---

## 1) Endpoint (Microsoft Defender for Endpoint — Advanced Hunting KQL)

**Process + network to proxy (by URL/domain)**
```kql
DeviceNetworkEvents
| where Timestamp between (ago(1h) .. now())
| where InitiatingProcessFileName =~ "chrome.exe"
| where RemotePort in (3128,8080,443)  // your proxy port(s)
| summarize by Timestamp, DeviceName, LocalIP, RemoteIP, RemotePort, InitiatingProcessCommandLine, ReportId
```

**Web protection / SmartScreen / Network Protection**
```kql
DeviceEvents
| where ActionType in ("SmartScreenUrlWarning", "NetworkProtectionBlocked")
| project Timestamp, DeviceName, FileName, AdditionalFields
```

**DNS lookups**
```kql
DeviceDnsEvents
| where Name has "example.com"
| project Timestamp, DeviceName, Name, IpAddresses
```

---

## 2) Proxy / SWG (generic fields — adapt to Zscaler, Blue Coat, Squid, Cloud SWG)
```
_time, user, src_ip, dst_ip, host, url, action, rule, bytes_in, bytes_out, category, ssl_inspect, threat, file_hash, xff
```
**Splunk search (proxy index)**
```
index=proxy (host="app.example.com" OR url="*app.example.com*")
| stats count sum(bytes_in) sum(bytes_out) values(action) by user, src_ip, xff, host, rule, ssl_inspect
```

**Who did this exact URL, when?**
```
index=proxy url="https://app.example.com/dashboard*"
| table _time user src_ip xff action rule category http_status server_ip ssl_protocol
```

---

## 3) NGFW / IDS / IPS (Palo Alto, Fortinet, etc.)

**NAT mapping + app-id**
```
index=firewall (dest_ip=<origin_or_proxy_ip>) action=allow
| stats values(nat_src_ip) values(app) values(rule) by src_ip user dst_ip
```

**Threats blocked**
```
index=firewall sourcetype=pan:threat OR sourcetype=fortinet:utm
| stats count by src_ip user threatid severity action
```

---

## 4) WAF / Reverse Proxy (F5/AWS WAF/NGINX/Envoy)

**Correlate by X-Forwarded-For + request id**
```
index=waf (host="app.example.com")
| table _time client_ip xff request_id uri status waf_rule rule_action
```

**Bot score / rate limiting**
```
index=waf bot_score>=80 OR rule="rate_limit*"
| stats count by client_ip xff uri rule status
```

---

## 5) Application + Identity (HTTP logs + IdP)

**App logs (JSON)**
```
index=applogs host="app.example.com" path="/dashboard*"
| spath input=request_id
| stats values(user) values(request_id) avg(latency_ms) by status
```

**Entra ID / Okta sign-ins (KQL for Sentinel)**
```kql
SigninLogs
| where AppDisplayName has "App Example"
| project TimeGenerated, UserPrincipalName, IPAddress, ConditionalAccessStatus, DeviceDetail
```

---

## 6) Full chain correlation (Splunk)
```
(index=endpoint OR index=proxy OR index=firewall OR index=waf OR index=applogs)
| eval cid=coalesce(request_id, session_id, correlation_id)
| stats earliest(_time) as first latest(_time) as last values(*) by cid user src_ip xff host url
| eval duration=last-first
| sort - duration
```

## 7) Handy artifacts to capture
- Corp root CA thumbprint used by TLS inspection
- NAT public IP used by your client session
- PAC file line that sent you to proxy
- IdP sign-in event ID tied to this session
- WAF request_id / correlation_id

