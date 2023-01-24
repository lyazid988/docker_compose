# TP Docker Compose
### B3 cybersécurité : Ynov Montpellier

## Introduction

L'idée du TP est de créer un fichier docker-compose, qui genere un ensemble de services (wireguard, samba, antivirus, ldap, gitlab et bitwarden). Une interconnection entre samba, ldap, gitlab et bitwarden à été demandée.

## Sommaire:

### I - Docker-compose c'est quoi ? et comment ça fonctionne ?

### II - Création du fichier docker-compose:

#### 1- Partie wireguard

#### 2- Partie samba

#### 3- Parite ldap

#### 4- Partie gitlab 

#### 5- Partie bitwarden

##  Docker-compose c'est quoi ? et comment ça fonctionne ?

Compose est un outil permettant de définir et d'exécuter des applications Docker multi-conteneurs. Avec Compose, vous utilisez un fichier YAML pour configurer les services de votre application. Ensuite, avec une seule commande, vous créez et démarrez tous les services de votre configuration.

Compose fonctionne dans tous les environnements : production, staging, développement, test, ainsi que les flux de travail CI. Il dispose également de commandes pour gérer l'ensemble du cycle de vie de votre application :

    1- Démarrer, arrêter et reconstruire les services
    
    2- Afficher l'état des services en cours d'exécution
    
    3- Diffuser la sortie du journal des services en cours d'exécution
    
    4- Exécuter une commande ponctuelle sur un service
    
    
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

### 2- Partie Samba:

```yaml
  samba:
    image: dperson/samba
    environment:
      TZ: 'EST5EDT'
    networks:
      - default
    ports:
      - "137:137/udp"
      - "138:138/udp"
      - "139:139/tcp"
      - "445:445/tcp"
    read_only: true
    tmpfs:
      - /tmp
    restart: unless-stopped
    stdin_open: true
    tty: true
    volumes:
      - /mnt:/mnt:z
      - /mnt2:/mnt2:z
    command: '-s "Mount;/mnt" -s "Bobs Volume;/mnt2;yes;no;no;bob" -u "bob;bobspasswd" -p'
```




