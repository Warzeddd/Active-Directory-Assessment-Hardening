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