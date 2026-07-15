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