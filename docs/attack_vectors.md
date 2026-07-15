# Vecteurs d'accès initiaux

## Tentatives infructueuses
* **Password Spraying global (Wordlist 10k) :** Déclenchement de la GPO de protection Active Directory avec verrouillage des comptes (`STATUS_ACCOUNT_LOCKED_OUT`).
* **Fuzzing Web / VHosts (Gobuster/FFUF) :** Recherche de répertoires sur `FILER01` (ports 80 et 21 IIS). Retour systématique en `403 Forbidden` ou `400 Bad Request`.
* **Coercition d'authentification en session nulle :** Tentative de relais NTLM via le module `coerce_plus` (PrinterBug) vers un écouteur `ntlmrelayx`. Aucun jeton intercepté.

## Vecteur réussi
* **Type :** Recherche de politiques de mots de passe par défaut / Attaque par dictionnaire ciblé de type "Identifiant = Mot de passe".
* **Payload / Exploit utilisé :** Script de boucle Bash séquentiel interrogeant l'interface d'authentification SMB de NetExec :
    `for u in $(cat users.txt); do nxc smb 10.250.0.101 -u "$u" -p "$u" | grep "+"; done`
* **Point d'entrée (IP / FQDN) :** `10.250.0.101` / `dc01.travers.ic` (Port TCP 445).