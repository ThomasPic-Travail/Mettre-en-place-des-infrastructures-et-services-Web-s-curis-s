# Sécurisation d'infrastructures et services Web — Mairie (Extranet / Intranet)

## Contexte

Ce projet consiste à configurer un serveur prototype hébergeant deux espaces web distincts pour une commune :

- Un **extranet** destiné aux habitants.
- Un **intranet** réservé aux agents de la mairie.

L'objectif est de mettre en place une infrastructure Web fonctionnelle puis de la durcir progressivement : accès FTP sécurisé pour les mises à jour de contenu, pare-feu applicatif, protection contre les intrusions, et chiffrement des échanges.

## Périmètre technique

- **Serveur Web Apache** : hébergement de l'extranet (public) et de l'intranet (interne), configuration de hôtes virtuels (VirtualHost) et séparation des accès.
- **Accès FTP sécurisé** : mise en place d'un accès dédié aux mises à jour de contenu, avec restriction des droits et chiffrement des échanges (FTPS).
- **Pare-feu réseau (netfilter/iptables)** : filtrage des flux entrants/sortants, réduction de la surface d'exposition du serveur.
- **CrowdSec** : détection et blocage collaboratif des comportements malveillants (brute-force, scan de ports, tentatives d'intrusion).
- **Certificat SSL auto-signé** : chiffrement des échanges HTTPS entre les utilisateurs et le serveur.
- **Environnements virtuels** : isolation des services pour limiter l'impact d'une compromission.

## Structure du dépôt

```
├── ASRC-P4-main/
│   ├── extranet/                                  # Code source de l'espace public (habitants)
│   └── intranet/                                  # Code source de l'espace interne (agents)
├── Compte rendu de réunion Prototype EXTRANET.pdf  # Compte-rendu de cadrage du besoin
├── Guide pratique de sécurisation d'un serveur FTP.pdf
├── Guide pratique de sécurisation du serveur web Apache.pdf
└── README.md
```

## Démarche de sécurisation

Le projet suit une logique de durcissement progressif plutôt qu'une sécurisation "a posteriori" :

1. Mise en place du service (Apache, FTP) dans une configuration fonctionnelle minimale.
2. Restriction des droits d'accès et des permissions au strict nécessaire.
3. Ajout d'une couche de détection et de blocage des comportements suspects (CrowdSec).
4. Filtrage réseau pour ne laisser passer que les flux légitimes (netfilter).
5. Chiffrement des échanges (SSL/FTPS) pour protéger la confidentialité des données en transit.

Les deux guides pratiques inclus dans ce dépôt détaillent pas à pas la sécurisation du serveur Apache et du serveur FTP.

## Auteur

**Thomas Pic** — Formation Administrateur Systèmes, Réseaux et Cybersécurité (Titre RNCP 40356), OpenClassrooms — Projet 4.
