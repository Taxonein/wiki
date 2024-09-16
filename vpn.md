---
title: VPN
description: 
published: true
date: 2024-09-16T06:26:50.638Z
tags: 
editor: markdown
dateCreated: 2024-09-11T05:35:44.626Z
---

# VPN
Данная статья предоставляет информацию о том, как настроить VPN серверы по тому или иному протоколу.

## L2TP


Необходимый список предустановок для работы впн:
```bash
iptables -A INPUT -i eth0 -p tcp --dport 22 -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.10.1.0/24 -o eth0 -j MASQUERADE
iptables -A INPUT -i eth0 -p udp --dport 500 -j ACCEPT
iptables -A INPUT -i eth0 -p udp --dport 4500 -j ACCEPT
iptables -A INPUT -i eth0 -p 50 -j ACCEPT
iptables -A INPUT -i eth0 -p 51 -j ACCEPT
```

Открыть порты 500 UDP 1701 UDP 4500 UDP

Если подключение с Mikrotik, то добавить в Proposal (IP->IpSec)

В файле `nano /etc/sysctl.conf` добавляем строку `net.ipv4.ip_forward = 1` и применяем настройки командой `sysctl -p`


### Настройка


Устанавливаем пакет `apt-get install strongswan`

Редактируем файл `nano /etc/ipsec.conf`

Подводим его вид до такого состояния:

```bash
conn rw-base
    fragmentation=yes
    dpdaction=clear 
    dpdtimeout=90s
    dpddelay=30s

conn l2tp-vpn
    also=rw-base
    ike=aes128-sha256-modp3072
    esp=aes128-sha256-modp3072
    leftsubnet=%dynamic[/1701]
    rightsubnet=%dynamic
    leftauth=psk
    rightauth=psk
    type=transport
    auto=add
```

Редактируем файл `nano /etc/ipsec.secrets` и добавляем туда строку `%any %any : PSK "MYKEY"` Где "MYKEY" выставляем свой ключ, который будет использоваться для аутентификации

После, нам надо рестартнуть службу `systemctl restart strongswan-starter`

И устанавливаем `apt-get install xl2tpd`

Открываем файл настроек `nano /etc/xl2tpd/xl2tpd.conf` и приводим его к следующему виду:
```bash
[global]
port = 1701
auth file = /etc/ppp/chap-secrets
access control = no
ipsec saref = yes
force userspace = yes

[lns default]
exclusive = no
ip range = 10.10.1.100-10.10.1.200
hidden bit = no
local ip = 10.10.1.1
length bit = yes
require authentication = yes
name = l2tp-vpn
pppoptfile = /etc/ppp/options.xl2tpd
flow bit = yes
```

Дополнительно, нам необходимо скопировать опции ppp устройства. Заходим в `cd /etc/ppp` и копируем `cp options options.xl2tpd`

Редактируем файл `nano options.xl2tpd` и приводим его к следующему виду:
```bash
asyncmap 0
auth
crtscts
lock
hide-password
modem
mtu 1460
lcp-echo-interval 30
lcp-echo-failure 4
noipx
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
multilink
mppe-stateful
```
Далее добавляем пользователя. В файле `nano /etc/ppp/chap-secrets` добавляем строку `user l2tp-vpn password *`

И рестартим нашу службу.
`systemctl restart xl2tpd`