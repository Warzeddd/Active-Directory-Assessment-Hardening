# Accès initial obtenu

## Machine compromise
* **Hostname :** `DC01` (Point de validation de l'authentification réseau)
* **IP :** `10.250.0.101`
* **Compte exécuté :** `travers.ic\backup`

## Collecte locale immédiate
* **Version de l'OS & Architecture :** Windows Server 2022 x64 (détecté via l'écriture des paquets SMB).
* **Connexions réseau actives (netstat -ano) :** *Non exécuté* (Pas d'accès shell à ce stade, interaction limitée aux connexions RPC/SMB distantes).
* **Processus en cours (tasklist / ps) :** *Non exécuté*.
* **Énumération des partages distants (Impact immédiat) :** * `\\10.250.0.101\SYSVOL` et `NETLOGON` (Accès `READ`)
    * `\\10.250.0.112\Configuration` (Accès `READ` critique sur ce partage masqué de `FILER01`).