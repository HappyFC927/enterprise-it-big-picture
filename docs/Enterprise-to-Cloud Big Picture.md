# 🌐 Enterprise-to-Cloud Big Picture (with Monitoring Points)

[User / Endpoint]
  └─(Power on → OS boot → **Defender for Endpoint** + Intune/MDM) 
    └─(802.1X / NAC posture)──[Access Switch/AP]──VLAN/QoS
      └──[Distribution/Core]──Routing/ACLs/NetFlow
        └──[**Palo Alto NGFW/IPS**]──(SNAT/NAT)──Threat/Traffic logs
          └──[**Zscaler SWG**]──URL filtering / DLP / SSL inspection / Sandbox
            └──[**Cisco Umbrella DNS**]──DNS filtering / resolution
              └──[Egress / SD-WAN]──ISP / Internet──Anycast
                └──[**AWS CloudFront (CDN Edge + Shield Advanced + WAF)**]
                  └──[Reverse Proxy / ALB]──[App Tier]──[DB / Cache / S3]
                    └──(IdP / SSO: Azure AD / Okta via OIDC/SAML)
                    └──(Secrets / PKI / HSM)
                    └──(**Splunk SIEM** for centralized log correlation)
                    └──(**Phantom SOAR** for automated response)
                    
# ☁️ Cloud Environments & What to Monitor

## Azure
- **Core Services**: VNets, NSGs, Azure Firewall, Application Gateway, Entra ID (SSO), Defender for Cloud.  
- **Monitoring**:  
  - Sign-in & Conditional Access logs (Entra ID).  
  - NSG flow logs / Firewall logs.  
  - Defender for Endpoint/Cloud alerts.  
  - Activity logs for RBAC/Key Vault access.  

## AWS
- **Core Services**: VPC, Security Groups, NACLs, ALB/NLB, CloudFront, S3, RDS, IAM roles, GuardDuty.  
- **Monitoring**:  
  - CloudTrail (API calls).  
  - VPC Flow Logs.  
  - GuardDuty findings.  
  - S3 access logs & KMS key usage.  
  - WAF/Shield logs.  

## GCP
- **Core Services**: VPC, Firewall rules, IAM, Cloud Armor (WAF/DDoS), GKE, Cloud Storage, BigQuery.  
- **Monitoring**:  
  - Cloud Audit Logs (Admin + Data Access).  
  - VPC Flow Logs.  
  - IAM role assignments / key usage.  
  - Cloud Armor logs (attack detections).  
  - GKE audit events.  

## Wiz (Cloud Security Posture / CNAPP)
- **Function**: Cross-cloud visibility & risk scoring.  
- **Monitors**:  
  - Misconfigurations (public buckets, open ports, weak IAM).  
  - Vulnerabilities in cloud workloads & containers.  
  - Toxic combinations (e.g., exposed VM + key in metadata).  
  - Compliance posture (CIS, NIST, SOC2).  

# 📊 Centralized Visibility
- **Splunk SIEM**: Ingests logs from Defender, Palo Alto, Umbrella, Zscaler, CloudFront, Azure, AWS, GCP.  
- **Phantom SOAR**: Automates response playbooks (block, isolate, revoke tokens).  
- **Dashboards**: Show e2e packet journey + cloud posture/risk.  
