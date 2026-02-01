# Projet WordPress — Docker Compose & CI/CD

Ce projet met en place un **WordPress fonctionnel** basé sur **Docker Compose**, accompagné d’une **pipeline CI/CD GitHub Actions**.  
L’objectif n’est pas juste de “faire tourner WordPress”, mais de montrer une façon propre et réaliste de travailler, proche de ce qui se fait en environnement pro.

Tout est pensé pour être reproductible, automatisé et sécurisé (notamment sur la gestion des secrets).

## Docker Compose

Docker Compose permet de décrire toute l’architecture dans un seul fichier et de lancer l’application avec une seule commande.

Dans ce projet, on a deux services :

- **MariaDB** : la base de données
- **WordPress** : l’application web

Ils sont reliés entre eux via le réseau Docker, sans configuration manuelle compliquée.

```yaml
version: "3.8"
```

Cette version est volontairement utilisée car elle est stable, largement supportée et compatible avec les outils CI/CD et Kubernetes (via Kompose).

## Architecture mise en place :

- WordPress tourne sur le port 8080
- MariaDB est isolée dans son propre conteneur
- Les données de la base sont persistées via un volume Docker

**Résultat :**

On peut arrêter/redémarrer les conteneurs sans perdre les données, l’environnement est identique pour tout le monde.

## Gestion des variables et des secrets :

Même si ce projet est simple, la structure est pensée pour être safe.

**Dans Docker Compose :**

les variables d’environnement servent à configurer WordPress et la base, aucune configuration “en dur” dans le code applicatif

**Dans la pipeline GitHub :**

les identifiants FTP ne sont jamais écrits en clair, ils sont stockés dans GitHub Secrets

Exemples :

- FTP_HOST
- FTP_USER
- FTP_PASSWORD

C’est un point important :

Les secrets ne doivent jamais être commités dans un dépôt, même pour un TP.


## La pipeline CI/CD :

Une pipeline GitHub Actions est déclenchée à chaque push sur la branche main.

Elle est découpée en plusieurs étapes claires.

1. Génération des manifests Kubernetes

Le fichier docker-compose.yml est converti en manifests Kubernetes

L’outil utilisé est Kompose

Ca permet de partir d’un projet Docker classique vers Kubernetes sans tout réécrire

2. Packaging

-> Les manifests Kubernetes générés sont regroupés
-> Un fichier ZIP est créé automatiquement
-> Ce ZIP devient un artifact de la pipeline

3. Déploiement

-> Le ZIP est envoyé sur un serveur distant via FTP
-> Les identifiants FTP sont récupérés depuis les secrets GitHub
-> Le déploiement ne se fait que si on est sur la branche main

Tout est automatisé :
pas besoin de manipulation manuelle, pas de copier-coller.
