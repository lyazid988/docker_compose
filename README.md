# TP Docker Compose
### B3 cybersécurité : Ynov Montpellier

## Introduction

L'idée du TP est de créer un fichier docker-compose, qui genere un ensemble de services (wireguard, samba, antivirus, ldap, gitlab, bitwarden et un serveur caddy). Une interconnection entre samba, ldap, gitlab et bitwarden est demandée.

## Sommaire:

### I - Docker-compose c'est quoi ? et comment ça fonctionne ?

### II - Trouver les images :

#### 1- Une image docker c'est quoi ?

#### 2- Les images des differents services 

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

### Les images des differents services:

#### 1- Image wireguard:


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
### 3- Partie ldap:

```yaml
  openldap:
    image: bitnami/openldap:2
    ports:
      - '1389:1389'
      - '1636:1636'
    environment:
      - LDAP_ADMIN_USERNAME=admin
      - LDAP_ADMIN_PASSWORD=adminpassword
      - LDAP_USERS=user01,user02
      - LDAP_PASSWORDS=password1,password2
    networks:
      - default
    volumes:
      - 'openldap_data:/bitnami/openldap'
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - my-network




```

### 4- Partie gitlab:

```yaml
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG:
        external_url 'https://gitlab.example.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
    

```
### 5- Bitwarden:

```yaml
  mssql:
    image: bitwarden/mssql:latest
    container_name: bitwarden-mssql
    restart: always
    stop_grace_period: 60s
    volumes:
      - ../mssql/data:/var/opt/mssql/data
      - ../logs/mssql:/var/opt/mssql/log
      - ../mssql/backups:/etc/bitwarden/mssql/backups
    env_file:
      - mssql.env
      - ../env/uid.env
      - ../env/mssql.override.env

  web:
    image: bitwarden/web:latest
    container_name: bitwarden-web
    restart: always
    volumes:
      - ../web:/etc/bitwarden/web
    env_file:
      - global.env
      - ../env/uid.env

  attachments:
    image: bitwarden/attachments:latest
    container_name: bitwarden-attachments
    restart: always
    volumes:
      - ../core/attachments:/etc/bitwarden/core/attachments
    env_file:
      - global.env
      - ../env/uid.env

  api:
    image: bitwarden/api:latest
    container_name: bitwarden-api
    restart: always
    volumes:
      - ../core:/etc/bitwarden/core
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/api:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  identity:
    image: bitwarden/identity:latest
    container_name: bitwarden-identity
    restart: always
    volumes:
      - ../identity:/etc/bitwarden/identity
      - ../core:/etc/bitwarden/core
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/identity:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  sso:
    image: bitwarden/sso:latest
    container_name: bitwarden-sso
    restart: always
    volumes:
      - ../identity:/etc/bitwarden/identity
      - ../core:/etc/bitwarden/core
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/sso:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  admin:
    image: bitwarden/admin:latest
    container_name: bitwarden-admin
    restart: always
    depends_on:
      - mssql
    volumes:
      - ../core:/etc/bitwarden/core
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/admin:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  icons:
    image: bitwarden/icons:latest
    container_name: bitwarden-icons
    restart: always
    volumes:
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/icons:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
    networks:
      - default
      - public

  notifications:
    image: bitwarden/notifications:latest
    container_name: bitwarden-notifications
    restart: always
    volumes:
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/notifications:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  events:
    image: bitwarden/events:latest
    container_name: bitwarden-events
    restart: always
    volumes:
      - ../ca-certificates:/etc/bitwarden/ca-certificates
      - ../logs/events:/etc/bitwarden/logs
    env_file:
      - global.env
      - ../env/uid.env
      - ../env/global.override.env
    networks:
      - default
      - public

  nginx:
    image: bitwarden/nginx:latest
    container_name: bitwarden-nginx
    restart: always
    depends_on:
      - web
      - admin
      - api
      - identity
    ports:
      - '80:8080'
      - '443:8443'
    volumes:
      - ../nginx:/etc/bitwarden/nginx
      - ../letsencrypt:/etc/letsencrypt
      - ../ssl:/etc/ssl
      - ../logs/nginx:/var/log/nginx
    env_file:
      - ../env/uid.env
    networks:
      - default
      - public

```





