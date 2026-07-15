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