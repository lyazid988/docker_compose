# TP Docker Compose
### B3 cybersécurité : Ynov Montpellier

## Introduction

L'idée du TP est de créer un fichier docker-compose, qui genere un ensemble de services (wireguard, samba, antivirus, ldap, gitlab, bitwarden et un serveur caddy). Une interconnection entre samba, ldap, gitlab et bitwarden est demandée.

## Sommaire:

### I - Docker-compose c'est quoi ? et comment ça fonctionne ?

### II - Trouver les images :

#### 1- Une image docker c'est quoi ?

### III - Création du fichier docker-compose:

##  Docker-compose c'est quoi ? et comment ça fonctionne ?

Compose est un outil permettant de définir et d'exécuter des applications Docker multi-conteneurs. Avec Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, avec une seule commande, vous créez et démarrez tous les services de votre configuration.

Compose fonctionne dans tous les environnements : production, staging, développement, test, ainsi que les flux de travail CI. Il dispose également de commandes pour gérer l'ensemble du cycle de vie de votre application :

    1- Démarrer, arrêter et reconstruire les services
    
    2- Afficher l'état des services en cours d'exécution
    
    3- Diffuser la sortie du journal des services en cours d'exécution
    
    4- Exécuter une commande ponctuelle sur un service
    
## Trouver les images docker:

### Une image docker c'est quoi ?

Une image Docker est un modèle en lecture seule, utiliser pour créer des conteneurs Docker. Elle est composée de plusieurs couches empaquetant toutes les installations, dépendances, bibliothèques, processus et codes d'application nécessaires pour un environnement de conteneur pleinement opérationnel.



## Création du fichier docker-compose:

### 1- Partie wireguard:

```yaml
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add: // donner des capacités sau conteneurs
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=wireguard.domain.com #optional
      - SERVERPORT=51820 #optional
      - PEERS=1 #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=10.13.13.1/24,10.13.13.2/24,10.13.13.3/24,10.13.13.4/24 #optional
      - PERSISTENTKEEPALIVE_PEERS= #optional
      - LOG_CONFS=true #optional
    volumes:
      - /path/to/appdata/config:/config
      - /lib/modules:/lib/modules #optional
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1 // à creuser
    restart: unless-stopped
```
image : on precise l'image utilisé
- container_name : on donne un nom au conteneur
- cap_add : rajouter des capacités au conteneur
- environnement:
PUID,PGID: Les variables PUID et PGID sont un moyen de translater un utilisateur à l'intérieur du conteneur vers un utilisateur sur l'hôte. Prenons Linuxserver, ils utilisent généralement un utilisateur "abc" à l'intérieur du conteneur

### 2- Partie minio:
```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  minio:
    image: 'bitnami/minio:latest'
    ports:
      - '9000:9000'
      - '9001:9001'
    environment:
      - MINIO_ROOT_USER=minio-root-user
      - MINIO_ROOT_PASSWORD=minio-root-password
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
    environment:
      - MINIO_SERVER_ACCESS_KEY=minio-access-key
      - MINIO_SERVER_SECRET_KEY=minio-secret-key
```

