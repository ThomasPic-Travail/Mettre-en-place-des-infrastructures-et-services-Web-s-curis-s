
# Prototype Extranet / Intranet  
*Service Informatique – Mairie de Valserac*

## Contenu

Ce dossier contient deux sites web :

- `extranet/` : site public, accessible à tous (accueil citoyens)
- `intranet/` : site réservé aux collaborateurs de la mairie

Chaque dossier contient une page `index.html`.

## Installation simple

1. Place le dossier `extranet` et le dossier `intranet` dans le dossier `/var/www/` de ton serveur web.
2. Configure deux virtual hosts sur Apache :
    - `extranet.valserac.local` → `/var/www/extranet/`
    - `intranet.valserac.local` → `/var/www/intranet/`
3. Ouvre les deux sites dans ton navigateur pour vérifier l’affichage.

## Sécurité

- Prévoir la configuration du pare-feu, CrowdSec, SSL et les droits d’accès selon les instructions internes.

---

*Contact : Service Informatique – DSI, Mairie de Valserac*