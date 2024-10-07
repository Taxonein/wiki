---
title: Proxmox
description: 
published: true
date: 2024-10-07T10:11:13.777Z
tags: 
editor: markdown
dateCreated: 2024-09-22T18:23:29.722Z
---

# Proxmox
В данной статье собраны решения ошибок системы, и информация по работе с VM
## Ошибки системы
Иногда в системе proxmox случается ошибка, когда выключение VM не представляется возможным. В консоли же выдает ошибку о том, что статус машины "locked". Для решения вводим следующие команды, меняя ID машины:
```bash
rm -f /var/lock/qemu-server/lock-101.conf
qm unlock 101
qm stop 101
```
## Cloud Init VM
Cloud Init подрузумевает под собой процесс, при котором созданием виртуальной машины (а после ее настройки) занимается сам Proxmox с помощью "Агента". 

Для начала нам необходимо установить образ в систему Proxmox. Заходим на сайт: https://cloud.debian.org/images/cloud/bookworm/latest/ (В зависимости от даты скачивания релиз системы может поменяться. На момент написания Debian имеет 12 версию, и распространяется под кодовым названием "Bookworm"). На сайте нам необходимо скачать в систему Proxmox файл, который имеет расширение .qcow2

Делаем это следующей командой:
`wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2`

После установки образа системы на Proxmox устанавливаем следующий пакет в систему командой: `sudo apt install libguestfs-tools`

После необходимо с помощью ранее установленного пакета установить в образ системы агента. Агент и выполняет взаимодействие Proxmox с VM. Установить агента можно следующей командой: `virt-customize --install qemu-guest-agent -a debian-12-generic-amd64.qcow2`

После нам необходимо создать саму VM. Вводим команды по списку:
```bash
qm create 1001 --name DEBIAN-CLOUD -net0 virtio,bridge=vmbr0
qm importdisk 1001 debian-12-generic-amd64.qcow2 local-lvm
qm set 1001 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-1001-disk-0
qm set 1001 --serial0 socket --vga serial0
qm set 1001 --ide2 local-lvm:cloudinit
qm set 1001 --boot c --bootdisk scsi0
qm set 1001 --serial0 socket --vga serial0
qm set 1001 --agent enabled=1
```
После ввода указанных команд наш образ готов. Советую перевести его в темплейт.