# TP Docker Compose
### B3 cybersécurité : Ynov Montpellier

## Introduction

L'idée du TP est de créer un fichier docker-compose, qui genere un ensemble de services (wireguard, samba, antivirus, ldap, gitlab et bitwarden). Une interconnection entre samba, ldap, gitlab et bitwarden à été demandée.

## Sommaire:

### I - Docker-compose c'est quoi ? et comment ça fonctionne ?

### II - Création du fichier docker-compose demandé:

#### 1- Partie wireguard

#### 2- Partie samba

#### 3- Parite ldap

#### 4- Partie gitlab 

#### 5- Partie bitwarden

##  Docker-compose c'est quoi ? et comment ça fonctionne ?

Compose est un outil permettant de définir et d'exécuter des applications Docker multi-conteneurs. Avec Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, avec une seule commande, vous créez et démarrez tous les services de votre configuration.

Compose fonctionne dans tous les environnements : production, staging, développement, test, ainsi que les flux de travail CI. Il dispose également de commandes pour gérer l'ensemble du cycle de vie de votre application :

    Démarrer, arrêter et reconstruire les services
    
    Afficher l'état des services en cours d'exécution
    
    Diffuser la sortie du journal des services en cours d'exécution
    
    Exécuter une commande ponctuelle sur un service



