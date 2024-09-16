---
title: Sonatype nexus
description: 
published: true
date: 2024-09-16T06:28:37.659Z
tags: 
editor: markdown
dateCreated: 2024-08-30T06:49:49.545Z
---

# Sonatype nexus
В статье будут описаны инструкции по подготовке хранилища секретов, решения проблем при подготовке и работы с репозиторием.

При ошибке доступа к файлам репозитория пишем на хосте следующее `chown -R 200 /nexus-data`


При ошибке входа в локальный репозиторий nexus мы заходим в файл `/etc/docker/daemon.json` и прописываем в нем следующее:

```bash
{
"insecure-registries" : [
"http://172.60.1.2:8000",
"https://172.60.1.2:8000",
"http://10.10.10.17:8000",
"https://10.10.10.17:8000"
]
}
```
После чего рестартим сервис docker командой `service docker restart` и входим в локальный репозиторий командой `docker login`

