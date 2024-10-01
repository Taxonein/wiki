---
title: Кастомный образ windows
description: 
published: true
date: 2024-10-01T08:46:40.788Z
tags: 
editor: markdown
dateCreated: 2024-09-22T18:23:45.875Z
---

# Кастомный образ windows
## Добавление драйверов в образ

Скачиваем драйвера (в папку) с помощью Snappy, распаковываем все архивы.

Распаковываем образ windows в папку, пишем:
```
Dism /Get-ImageInfo /imagefile:"D:\ChangedISO\WinFolder\sources\install.esd"
```

Вывод даст нам информацию о имеющихся в образе версий windows. Нам нужна Windows 10 Pro.

```
Индекс: 4
Имя : Windows 10 Pro
Описание : Windows 10 Pro
Размер: 15 571 487 266 байт
```

Дополнительно, нам необходимо из .esd файла получить .wim файл. Делаем это командой:

```
dism /export-image /SourceImageFile:D:\ChangedISO\WinFolder\sources\install.esd /SourceIndex:4 /DestinationImageFile:D:\ChangedISO\WinFolder\sources\install.wim /Compress:max /CheckIntegrity
```

В папке с образом винды мы получим .wim файл. Его будем редактировать.

Монтируем образ в папку следующей командой:

```
dism /mount-wim /wimfile:"D:\ChangedISO\WinFolder\sources\install.wim" /index:1 /mountdir:D:\ChangedISO\mount
```

Далее добавляем необходимые нам драйвера в образ командой:

```
dism /image:D:\ChangedISO\mount /Add-Driver /driver:D:\ChangedISO\Drivers\ /recurse
```











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