# Configuration

## Active Directory
* **Nom du domaine :** `travers.ic`
* **Niveau fonctionnel :** Windows Server 2016 ou supérieur (`domainFunctionality: 7`)
* **Relations d'approbation :** Aucune (Forêt unique isolée)
* **Coercition d'authentification :** `DC01` vulnérable à **PetitPotam** (MS-EFSR) et **PrinterBug** (MS-RPRN) en session nulle.

## Autre
* **Politique de verrouillage :** Seuil restrictif actif. Le password spray provoque un `STATUS_ACCOUNT_LOCKED_OUT` (Contradiction flagrante avec le brief initial du DSI).
* **EDR / Antivirus :** Windows Defender & Windows Firewall actif (détecté via l'UUID de `MPSSVC.dll` sur le mappeur RPC).