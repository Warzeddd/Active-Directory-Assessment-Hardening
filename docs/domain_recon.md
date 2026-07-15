# Élévation Domaine (Active Directory)

## Chemins d'attaque (BloodHound)
- **Relations d'abus (GenericAll, WriteDacl, Owner) :** *Non exploité* (Analyse graphique BloodHound abandonnée suite à une panne de la JVM Neo4j locale).
- **Délégation Kerberos (Unconstrained, Constrained, RBCD) :** *Aucune*.
- **Vulnérabilités GPO :** *Aucune* (Le script défensif `admin.ps1` présent dans SYSVOL/Configuration contenait des identifiants en clair mais n'était pas une vulnérabilité de GPO de restriction).

## Résultat
- **Compte compromis :** `travers.ic\pclerc` (Domain Admin implicite validé par un succès PtH global sur `DC01`), suivi de la compromission suprême du compte `travers.ic\Administrator` (RID 500) et du secret d'infrastructure `krbtgt` (RID 502) via une attaque DCSync (`--ntds`).