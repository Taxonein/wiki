---
title: Кастомный образ windows
description: 
published: true
date: 2024-09-16T08:32:52.071Z
tags: 
editor: markdown
dateCreated: 2024-09-16T08:13:23.579Z
---

# Кастомный образ windows
![windowscustom.png](/windowscustom.png)

```cmd
diskpart
lis vol
exit
```

Можно поменять формат с install.esd на install.wim
```cmd
dism /capture-Image /imageFile:<НА КАКОЙ КОПИРОВАТЬ>:\install.esd /capturedir:<ДИСК С ВИНДОЙ>:\ /name:windows
```

```cmd
Oscdimg /u2 /m /bootdata:2#p0,e,b<ДИСК>:\VINTES\boot\Etfsboot.com#pef,e,b<ДИСК>:\VINTES\efi\microsoft\boot\Efisys.bin <ДИСК>:\VINTES <ДИСК>:\WindowsCustom.iso
```

C:\Users\Default
cntrl+shift+f3
shift+f10