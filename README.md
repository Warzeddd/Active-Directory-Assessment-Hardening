This repository features the technical documentation and remediation playbooks from an *Assume Breach* security assessment against a simulated healthcare infrastructure (`10.250.0.0/24`). The objective was to evaluate the resilience of the Active Directory forest and pivot from an unprivileged position to Domain Admin.
* **DC01** (`10.250.0.101`) - Active Directory Domain Controller
* **FILER01** (`10.250.0.112`) - Enterprise File Server
* **DESKTOP01** (`10.250.0.117`) - Internal Workstation
1. **Reconnaissance & Foothold**: Kerberos Principal Enumeration via `kerbrute` followed by targeted Password Spraying to compromise the `backup` service account.
2. **Lateral Movement**: High-volume share enumeration leading to the discovery of cleartext credentials inside a legacy deployment script (`admin.ps1`).
4. **Privilege Escalation**: LSASS memory space dump on `FILER01` to extract volatile high-privileged tokens, achieving full Active Directory Domain Dominance.
## Repository Layout
* `/docs`: Step-by-step technical breakdown of each compromise phase.
* `/docs/README.md`: Consolidated full-length technical report.
