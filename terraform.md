---
title: Terraform
description: 
published: true
date: 2024-09-16T06:27:04.600Z
tags: 
editor: markdown
dateCreated: 2024-08-30T06:12:26.074Z
---

# Terraform
Terraform представляет из себя инструмент описания создания виртуальных машин с помощью IaC (Infrastructure as Code). С помощью данного инструмента мы можем создавать машины в автоматическом режиме, используя только код, и подготовленный темлейт системы в гипервизоре.

Инструмент не доступен в РФ, по этой причине скачиваем его через зеркало Яндекса:
https://hashicorp-releases.yandexcloud.net/terraform
Инструкция Яндекса:
https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart

## Proxmox credentials
Пример файла `credentials.auto.tfvars`:

```bash
proxmox_api_url = "https://10.10.10.10:8006/api2/json"
proxmox_api_token_id = "root@pam!terraform"
proxmox_api_token_secret = "YOUR-SECRET-HERE"
```
## Proxmox provider
Пример файла `provider.tf`:
```bash
terraform {
  required_version = ">= 0.13"

  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
    proxmox = {
        source = "telmate/proxmox"
        version = "3.0.1-rc1"
    }
  }
}
provider "yandex" {
  zone = "ru-central1-a"
}

variable "proxmox_api_url" {
    type = string
}

variable "proxmox_api_token_id" {
    type = string
    sensitive = true
}

variable "proxmox_api_token_secret" {
    type = string
    sensitive = true
}
provider "proxmox" {
    pm_api_url = var.proxmox_api_url
    pm_api_token_id = var.proxmox_api_token_id
    pm_api_token_secret = var.proxmox_api_token_secret

    pm_tls_insecure = true
}
```
## Proxmox main_copy
Пример файла `main.tf` при использовании обычного темплейта (НЕ cloud init):
```bash
resource "proxmox_vm_qemu" "main" {
    count = 2
    name = "prod-server-${count.index + 1}"
    desc = "Clone of machine DEBIAN-DOCKER for production"
    vmid = "40${count.index + 1}"
    target_node = "hosting"

    agent = 1
    clone = "DEBIAN-DOCKER"
    cores = 1
    sockets = 2
    cpu = "host"
    memory = 2048

    network {
        bridge = "vmbr0"
        model = "virtio"
    }


    disks {
        scsi {
            scsi0 {
                disk {
                    storage = "local-lvm"
                    size = "30"
                }
            }
        }
    }
}
```
## Proxmox main_cloud-init
Пример файла `main.tf` при использовании cloud init образа:
```bash
resource "proxmox_vm_qemu" "main" {

    count = 1
    name = "cloud-server-${count.index + 1}"
    desc = "Clone of machine DEBIAN-CLOUD for production"
    vmid = "50${count.index + 1}"
    target_node = "hosting"

    agent = 1

    clone = "DEBIAN-CLOUD"

    cores = 1
    sockets = 1
    cpu = "host"
    memory = 1024

    os_type = "cloud-init"

    network {
        bridge = "vmbr0"
        model = "virtio"
    }

    cloudinit_cdrom_storage = "local-lvm"
    scsihw = "virtio-scsi-pci"
    boot = "order=scsi0;ide2;net0"

    disks {
        scsi {
            scsi0 {
                disk {
                    storage = "local-lvm"
                    size = "10"
                }
            }
        }
    }

    ipconfig0 = "ip=10.10.10.${30 + count.index}/24,gw=10.10.10.1"
    nameserver = "10.10.10.1"
    ciuser = "root"
    sshkeys = <<EOF
    PUT-YOU-PUBLIC-KEY-HERE
    EOF
}
```