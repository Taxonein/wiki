---
title: GitLab
description: 
published: true
date: 2024-09-16T06:27:51.592Z
tags: 
editor: markdown
dateCreated: 2024-08-30T06:37:25.627Z
---

# GitLab
Тут будут описаны примеры файлов пайплайнов, nginx, и других систем косвенно связанных с GitLab. По сути статья будет представлять из себя неструктурированный мусор.

Создание самоподписанного сертификата для GitLab Runner:
```bash
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.example.com" -out server.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:example.com,DNS:www.example.com") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

gitlab-runner register  --url [https://gitlab.taxonein.local](https://gitlab.taxonein.local/)  --token glrt-X5z45Z9RtpHdZDMevkiC  --tls-ca-file /root/server.crt
```

Пример пайплайна:
```bash
stages:
    - deps
    - check
    - build
    - cleanup
    - deploy

deps: #Подготавливает зависимости, и устанавливает их в venv python 3.11.6
    stage: deps
    script:
        - python -m venv venv
        - ./venv/bin/pip install flake8
    artifacts:
        paths:
            - venv/
    tags:
        - api-test-node-shell

check: #Запускает unit  тесты, удалить allow_failure когда тесты появятся
    stage: check
    script:
        - ./venv/bin/python -m pip install -r requirements.txt
        - cp ${SCRIPTS_FOLDER}/testing.sh .
        - ./testing.sh
        - ./venv/bin/python -B -m unittest discover -s src -p *_test.py
    tags:
        - api-test-node-shell

build: #Выполняется на ветке develop, билдит образ (а после удаляет)
    stage: build
    script: 
        - docker build -t sebot-${CI_COMMIT_BRANCH}:latest .
        - docker image rm sebot-${CI_COMMIT_BRANCH}
    tags:
        - api-test-node

cleanup: #Выключает и удаляет предыдущие контейнеры/образы, для того чтобы подготовить машину к следующей пайпе
    stage: cleanup
    script: 
        - cp ${SCRIPTS_FOLDER}/test.sh .
        - ./test.sh
    tags:
        - api-test-node-shell
    only:
        - master
        - develop

deploy: #Билдит и включает контейнер.
    stage: deploy
    script: 
        - docker build -t sebot-${CI_COMMIT_BRANCH}:latest .
        - docker compose --project-name develop up -d --force-recreate
    tags:
        - api-test-node
    only:
        - master
        - develop
```
Пример nginx конфига для GitLab
```bash
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    server {
        listen 443 ssl;
        server_name gitlab.taxonein.local;

        ssl_certificate /etc/nginx/certs/server.crt;
        ssl_certificate_key /etc/nginx/certs/server.key;

        location / {
            proxy_pass http://172.60.0.3:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $host;
        }
    }
}





config/gitlab.rb

nginx['listen_port'] = 80

##! **Override only if your reverse proxy internally communicates over HTTP**
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#supporting-proxied-ssl
nginx['listen_https'] = false

##! **Override only if you use a reverse proxy with proxy protocol enabled**
##! Docs: https://docs.gitlab.com/omnibus/settings/nginx.html#configuring-proxy-protocol
nginx['proxy_protocol'] = false

# nginx['custom_gitlab_server_config'] = "location ^~ /foo-namespace/bar-project/raw/ {\n deny all;\n}\n"
# nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/example.conf;"
# nginx['proxy_read_timeout'] = 3600
# nginx['proxy_connect_timeout'] = 300
nginx['proxy_set_headers'] = {
  "Host" => "$http_host_with_default",
  "X-Real-IP" => "$remote_addr",
  "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on",
  "Upgrade" => "$http_upgrade",
  "Connection" => "$connection_upgrade"
 }
```