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