---
title: VPN на Mikrotik
description: 
published: true
date: 2024-09-16T06:28:21.003Z
tags: 
editor: markdown
dateCreated: 2024-09-11T07:26:39.229Z
---

# VPN на Mikrotik

## Маршрутизация запросов из листа в гейтвей впн

Создаем в `Routing -> Tables` запись, название `rvpn`, галочку на `FIB`

Создаем интерфейс в ppp, к примеру l2tp (DEFAULT ROUTES убрать)

Создаем `IP -> Routes` dst: 0.0.0.0/0 gateway: l2tp-out, dist 1 scope 30 t.scope 10, routing table: rvpn

В  `IP -> Firewall -> NAT` создаем запись: srcnat,  dst.adress.list=VPN, out.interface=l2tp-out, routing mark =rvpn, action=masquerade

Идем в `IP -> Firewall -> Mangle` создаем запись: prerouting, dst.adress.list=VPN, action=mark routing, routing mark=rvpn

После чего в `IP -> Firewall -> Adress Lists` делаем запись с нашим листом "VPN" и доменом.

## Маршрутизация конкретного устройства в ВПН

Порядок действий тот же что и выше, кроме двух пунктов: NAT,MANGLE

В `IP -> Firewall -> NAT` создаем запись: srcnat, src.address.list=VPNPC, out.intefrace=l2tp-out, routing mark=rvpn, action=masquerade

Идем в `IP -> Firewall -> Mangle` создаем запись: prerouting, src.address.list=VPNPC, action=mark routing, routing mark=rvpn