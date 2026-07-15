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