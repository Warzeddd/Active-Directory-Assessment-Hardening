# Services

## Services ouverts
* **Port 21 (HTTP / Non-standard) :** 10.250.0.112 (Bannière IIS, pas de démon FTP)
* **Port 22 (SSH) :** 10.250.0.101, 10.250.0.112, 10.250.0.117
* **Port 53 (DNS) :** 10.250.0.101
* **Port 80 (HTTP) :** 10.250.0.112
* **Port 88 (Kerberos) :** 10.250.0.101
* **Port 135 (RPC) :** 10.250.0.101, 10.250.0.112, 10.250.0.117
* **Port 139 (NetBios) :** 10.250.0.101, 10.250.0.117
* **Port 389 / 636 (LDAP / LDAPS) :** 10.250.0.101
* **Port 445 (SMB) :** 10.250.0.101, 10.250.0.112, 10.250.0.117 (SMB Signing requis uniquement sur DC01)
* **Port 464 (Kpasswd5) :** 10.250.0.101
* **Port 3268 / 3269 (Global Catalog LDAP) :** 10.250.0.101
* **Port 3389 (RDP) :** 10.250.0.101, 10.250.0.112, 10.250.0.117
* **Port 5985 / 5986 (WinRM) :** 10.250.0.101, 10.250.0.112, 10.250.0.117

## ## Versions de services
- **HTTP (Port 21 & 80) :** Microsoft IIS httpd 10.0 / Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
  *Note : Le port 21 n'est pas un démon FTP, il héberge une instance HTTP restreinte.*
- **SSH :** OpenSSH protocol 2.0
- **DNS :** Simple DNS Plus
- **Kerberos :** Microsoft Windows Kerberos (server time: 2026-07-01)
- **RPC :** Microsoft Windows RPC
- **NetBios :** Microsoft Windows netbios-ssn
- **LDAP / GlobalLDAP :** Microsoft Windows Active Directory LDAP (Domain: travers.ic, Site: Default-First-Site-Name)
- **SMB :** Microsoft Windows SMB2/SMB3 (SMB Signing désactivé sur le Filer et le Desktop)
- **Kpasswd5 :** Kerberos Password Change Service
- **ncacn_http :** Microsoft Windows RPC over HTTP 1.0
- **Ms-wbt-server (RDP) :** Microsoft Terminal Services
- **WinRM :** Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

## ## Versions d'OS 
- Windows Server 2022 Build 20348 x64