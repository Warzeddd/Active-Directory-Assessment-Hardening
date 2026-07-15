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