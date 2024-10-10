---
title: Docker команды и compose файлы
description: 
published: true
date: 2024-10-10T05:14:35.740Z
tags: 
editor: markdown
dateCreated: 2024-09-22T18:23:13.581Z
---

# Docker
В этой статье описаны команды и docker compose файлы для продакшена
## Основные команды
`docker ps` - вывод списка активных контейнеров
`docker ps -a` - вывод списка всех контейнеров на хосте (даже неактивных)
`docker rm ###` - удаление контейнера (неактивного)
`docker stop ###` - остановка активного контейнера (soft stop)
`docker kill ###` - убийство активного контейнера (hard stop)
`docker build -t tag:v1 .` - билд образа docker из `Dockerfile`, который находится в текущей директории. Билд с именем образа "tag" и версией "v1"
`docker compose up` - запустить docker-compose.yml файл с активной консолью
`docker compose up -d` - запустить docker-compose.yml файл без консоли
`docker compose up -d --force-recreate` - запустить docker-compose.yml файл без консоли, предварительно перебилдив образ с измененными параметрами
`docker logs ###` - логи контейнера
`docker exec -it ### bash` - войти в bash терминал контейнера
`docker network ls` - вывод имеющихся в системе docker сетей
`docker inspect ###` - вывод подробной информации о контейнере
`docker network inspect ###` - вывод подробной информации о docker сети
`docker image ls` - вывод списка образов
`docker image rm` - удаление изображения
`docker system prune -a` - удаляет все остановленные контейнеры, изображения, volume, сети.
`docker image inspect ###` - вывод подробной информации о образе
## Системные файлы
`/var/lib/docker/volumes/` - по этому пути в системе хранятся docker volumes
`/etc/docker/daemon.json` - файл в котором применяются insecure регистры
## Одиночные контейнеры
Примеры запуска одиночных приложений:

`docker run -it --name nettool1 --ip 192.168.1.125 --net mainnet nicolaka/netshoot bash`

`docker run --name nginxserv1 -d -p 80:80 --net mymacvl -v nginx:/usr/share/nginx/html:ro nginx`

## Создание сети

`docker network create -d macvlan --subnet 192.168.1.0/24 --gateway 192.168.1.1 --ip-range 192.168.1.124/32 -o parent=enx00e04c3643f8 mainnet`

`docker network create --subnet 172.20.0.0/24 --gateway 172.20.0.1 nginxapp`
## DC Nginx
```yaml
services:
  nginx:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - nginx-data:/data
      - nginx-letsencrypt:/etc/letsencrypt

volumes:
  nginx-data:
    external: false
  nginx-letsencrypt:
    external: false
```
## DC Postrges
```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    environment:
      POSTGRES_DB=postgres
      POSTGRES_PASSWORD=password
      POSTGRES_USER=postgres
    restart: unless-stopped
    ports:
      - "5432:5432"
    volumes:
      - postgres:/var/lib/postgresql/data

volumes:
  postgres:
    external: false
```
## DC Zabbix MySQL
```yaml
services:
  mysql-server:
    container_name: mysql-server
    image: mysql:8.0-oracle
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.2
    volumes:
      - ./data:/var/lib/mysql:rw
    environment:
      MYSQL_DATABASE: "zabbix"
      MYSQL_USER "zabbix"
      MYSQL_PASSWORD "PASSWORD"
      MYSQL_ROOT_PASSWORD: "PASSWORD"
    restart: unless-stopped
    command: --character-set-server=utf8 --collation-server=utf8_bin --default-authentication-plugin=mysql_native_password

  zabbix-java-gateway:
    container_name: zabbix-java-gateway
    image: zabbix/zabbix-java-gateway:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.3
    restart: unless-stopped

  zabbix-server-mysql:
    container_name: zabbix-server-mysql
    image: zabbix/zabbix-server-mysql:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.4
    environment:
      DB_SERVER_HOST: "mysql-server"
      MYSQL_DATABASE "zabbix"
      MYSQL_USER "zabbix"
      MYSQL_PASSWORD: "PASSWORD"
      MYSQL_ROOT_PASSWORD: "PASSWORD"
      ZBX_JAVAGATEWAY=zabbix-java-gateway
    ports:
      - "10051:10051"
    restart: unless-stopped

  zabbix-web-nginx-mysql:
    container_name: zabbix-web-nginx-mysql
    image: zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.5
    environment:
      ZBX_SERVER_HOST: "zabbix-server-mysql"
      DB_SERVER_HOST: "mysql-server"
      MYSQL_DATABASE: "zabbix"
      MYSQL_USER: "zabbix"
      MYSQL_PASSWORD: "PASSWORD"
      MYSQL_ROOT_PASSWORD: "PASSWORD"
      ZBX_JAVAGATEWAY=zabbix-java-gateway
    ports:
      - "80:8080"
    restart: unless-stopped

networks:
  zabbix-net:
    ipam:
      driver: default
      config:
        - subnet: 172.30.0.0/24
```

## DC Zabbix Pgsql
```yaml
#Last edit: 10.10.24
services:
  zabbix-snmptraps:
    container_name: zabbix-snmptraps
    image: zabbix/zabbix-snmptraps:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.2
    volumes:
      - zabbix-snmptraps:/var/lib/zabbix/snmptraps
      - zabbix-mibs:/usr/share/snmp/mibs
    ports:
      - "162:1162"
    restart: unless-stopped

  zabbix-server-pgsql:
    container_name: zabbix-server-pgsql
    image: zabbix/zabbix-server-pgsql:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.3
    environment:
      - DB_SERVER_HOST=10.10.10.20
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=zabbix
      - ZBX_ENABLE_SNMP_TRAPS=true
    volumes:
      - zabbix-snmptraps:/var/lib/zabbix/snmptraps
      - zabbix-mibs:/usr/share/snmp/mibs
    ports:
      - "10051:10051"
    restart: unless-stopped

  zabbix-web-nginx-pgsql:
    container_name: zabbix-web-nginx-pgsql
    image: zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest
    networks:
      zabbix-net:
        ipv4_address: 172.30.0.4
    environment:
      - ZBX_SERVER_HOST=172.30.0.3
      - DB_SERVER_HOST=10.10.10.20
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=zabbix
      - ZBX_SERVER_NAME=zabbix.taxonein.ru
    volumes:
      - zabbix-nginx:/etc/ssl/nginx
    ports:
      - "80:8080"
      - "443:8443"
    restart: unless-stopped

volumes:
  zabbix-snmptraps:
    external: false
  zabbix-mibs:
    external: false
  zabbix-nginx:
    external: false

networks:
  zabbix-net:
    ipam:
      driver: default
      config:
        - subnet: 172.30.0.0/24
```

## DC Nexus
```yaml
services:
  nexus:
    image: sonatype/nexus3
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - ./data/nexus:/nexus-data
    networks:
      nexus-net:
        ipv4_address: 172.60.1.2

  nginx-nexus:
    image: nginx
    volumes:
      - ./data/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./data/nginx_crts:/etc/nginx/ssl/
    ports:
      - "443:443"
      - "8080:80"
    networks:
      nexus-net:
        ipv4_address: 172.60.1.3

networks:
  nexus-net:
    ipam:
      driver: default
      config:
        - subnet: "172.60.1.0/24"
```
## DC Nginx PrxMngr
```yaml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
  web:
    image: nginx
    volumes:
      - ./templates:/etc/nginx/templates
    ports:
      - "8080:80"
```
## DC GitLab
```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    restart: unless-stopped
    volumes:
      - ./data/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./data/nginx/certs:/etc/nginx/certs
    ports:
      - "443:443"
      - "80:80"
    networks:
      gitlab_net:
        ipv4_address: 172.60.0.2

  gitlab:
    image: 'gitlab/gitlab-ee:latest'
    container_name: gitlab
    restart: unless-stopped
    hostname: 'gitlab.taxonein.local'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.taxonein.local'
    volumes:
      - ./data/config:/etc/gitlab
      - ./data/logs:/var/log/gitlab
      - ./data/data:/var/opt/gitlab
    shm_size: 256m
    networks:
      gitlab_net:
        ipv4_address: 172.60.0.3

networks:
  gitlab_net:
    ipam:
      driver: default
      config:
        - subnet: 172.60.0.0/24
```
## DC WikiJS
```yaml
services:
  wikijs:
    image: ghcr.io/requarks/wiki:2
    container_name: wikijs
    environment:
      DB_TYPE: "postgres"
      DB_HOST: "10.10.10.20"
      DB_PORT: "5432"
      DB_USER: "wikijs"
      DB_PASS: "PASSWORD"
      DB_NAME: "wikijs"
    restart: unless-stopped
    ports:
      - "8081:3000"
```