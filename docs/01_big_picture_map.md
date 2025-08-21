# 🗺️ Big Picture Map

```text
[User/Endpoint]
  └─(Power on → OS boot → EDR/MDM) 
   └─(802.1X/NAC)──[Access Switch/AP]──VLAN/QoS
     └──[Distribution/Core]──Routing/ACLs/Telemetry
       └──[NGFW/IPS]──(SNAT)──[Forward Web Proxy]──URL/DLP/SSL inspect
         └──[Egress/SD-WAN]──ISP/Internet──CDN/Anycast
           └──[Authoritative DNS + CDN Edge]
             └──[Reverse Proxy/Load Balancer]──[WAF]──[App Tier]──[DB/Caches]
                                     └──(IdP/SSO: SAML/OIDC/Kerberos)
                                     └──(Secrets/PKI/HSM)
                                     └──(Observability/SIEM/SOAR)
```

👉 See [Step-by-Step Flow](02_step_by_step_flow.md) for details.
