# Active Directory Assessment & Hardening Portfolio

Rapport d'audit de sécurité offensive (*Assume Breach*) et implémentation de stratégies de durcissement défensif (*Hardening*) sur une infrastructure Active Directory d'entreprise.

---

## 🎯 Périmètre de l'Audit (Scope)

* **Environnement** : Infrastructure Active Directory d'entreprise simulant un réseau critique (`10.250.0.0/24`).
* **Cibles identifiées** :
  * `DC01` (`10.250.0.101`) : Contrôleur de Domaine principal (Windows Server).
  * `FILER01` (`10.250.0.112`) : Serveur de fichiers de production.
  * `DESKTOP01` (`10.250.0.117`) : Poste de travail d'un collaborateur du domaine.

---

## ⚡ Chaîne d'Attaque Validée (Kill Chain)

1. **Reconnaissance & Foothold** : Énumération des comptes Kerberos via `kerbrute` suivie d'une attaque par *Password Spraying* ciblée sur les comptes de service. Compromission initiale du compte de sauvegarde de l'infrastructure.
2. **Mouvements Latéraux & Escalade Interne** : Énumération automatisée des partages réseau menant à la découverte de clés d'authentification en clair dans un script de déploiement obsolète (`admin.ps1`).
3. **Pivoting & Propagation** : Exploitation de l'absence de Windows LAPS pour se propager horizontalement d'un poste à un autre via l'identifiant RID 500 local par usurpation d'empreinte de mot de passe (*Pass-the-Hash*).
4. **Domination de Domaine** : Extraction de tokens privilégiés en mémoire via dump du processus `LSASS.exe` sur le serveur de fichiers de production `FILER01`, menant à la compromission totale du groupe des Administrateurs du Domaine.

---

## 🛡️ Plan de Remédiation & Hardening Appliqué

* **Windows LAPS (Local Administrator Password Solution)** : Déploiement automatisé par GPO de LAPS pour forcer la rotation dynamique et le chiffrement des mots de passe des comptes Administrateurs locaux sur l'ensemble du parc.
* **Fine-Grained Password Policies (FGPP)** : Implémentation d'une stratégie de mots de passe personnalisée restrictive (objets PSO) pour bloquer les attaques de bruteforce en limitant les échecs et en exigeant une haute entropie (12 caractères complexes minimum).
* **Credential Guard & Durcissement LSASS** : Activation de la protection additionnelle LSA pour bloquer la lecture mémoire directe par les outils d'extraction non autorisés et isolement des sessions utilisateur privilégiées.

---

## 📂 Organisation des Dossiers

```text
.
├── docs/                        # Livrables d'ingénierie et rapports d'audit techniques
│   ├── audit_scope.md           # Définition du périmètre d'évaluation et règles d'engagement
│   ├── user_recon.md            # Énumération des identités et découverte d'annuaire
│   ├── initial_foothold.md      # Compromission initiale et vecteurs d'entrée
│   ├── lateral_movements.md     # Exploitation des partages réseau et d'accès locaux
│   ├── local_priv_escalation.md # Techniques d'élévation de privilèges locales (Postes/Serveurs)
│   ├── pivoting_and_tunneling.md# Pivotement réseau et rebond d'infrastructure
│   ├── target_systems.md        # Cartographie technique des cibles identifiées et faiblesses
│   ├── remediation_mechanisms.md# Rapport complet de remédiation et stratégies de durcissement AD
│   └── README.md                # Synthèse générale des livrables de sécurité
└── .gitignore                   # Fichier d'exclusion pour empêcher la fuite de données d'audit réelles
