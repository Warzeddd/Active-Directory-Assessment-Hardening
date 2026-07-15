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