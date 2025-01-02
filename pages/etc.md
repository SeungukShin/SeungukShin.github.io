---
title: Etc
---

# Etc
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Gnome

### Use same style for QT5

Install `qt5ct` and `qt5-styleplugins`

Add system variables on `/etc/profile`
```sh
$ export QT_QPA_PLATFORMTHEME="qt5ct"
$ export QT_AUTO_SCREEN_SCALE_FACTOR=1
```

Execute qt5ct (qt5 settings)
* Select `gtk2` on `Style`
* Select `Default` on `Standard dialogs`

## KDE

### Remove title bar and borders on maximized windows

Edit `~/.kde4/share/config/kwinrc`
```conf
[Windows]
ActiveMouseScreen=true
AltTabStyle=KDE
AutoRaise=false
AutoRaiseInterval=750
BorderSnapZone=10
BorderlessMaximizedWindows=true
CenterSnapZone=0
```

Restart kwin
```sh
$ kwin --replace
```

### Enable the appmenu

Install appmenu
```sh
$ sudo apt-get install appmenu-qt appmenu-qt5 appmenu-gtk appmenu-gtk3
```

Edit `~/.config/gtk-3.0/settings.ini`
```conf
gtk-shell-shows-menubar = 1
```

enable the appmenu
* System Settings > Application Appearance > Style
* "Fine Tuning" tab
* "Menubar style" (bottom)
* select between: "in application", "title bar button" or "top screen menubar"

### Change fonts of konsole

Edit `~/.kde4/share/apps/konsole/xxx.profile`
```conf
[Appearance]
Font=NanumGothicCoding,14,-1,5,50,0,0,0,0,0
```

## Windows

### Change the key to change input method for Korean

* Open `regedit`
* Find `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters`
* Change a value of `LayerDriver KOR` from 'kbd101**a**.dll' to 'kbd101**c**.dl'

### Register / Unregister a service

Register a service
```sh
$ sc create "Service Name" binPath="C:\Path\to\execute.exe"
```
in case of msys64 (you have to run something in sh of msys64)
```sh
$ sc create "sshd" binPath="C:\msys64\usr\bin\sh.exe --login -c /usr/bin/sshd"
```

Unregister a service
```sh
$ sc delete "Service Name"
```

## Android

### Change immersive mode

Add apps for immersive mode
```sh
$ adb shell settings put global policy_control immersive.status=org.mozilla.firefox
$ adb shell settings put global policy_control immersive.navigation=org.mozilla.firefox
$ adb shell settings put global policy_control immersive.full=org.mozilla.firefox,org.torproject.torbrowser
```

Remove immersive mode
```sh
$ adb shell settings put global policy_control null*
```

# References

* [Hans Chen's Blog](https://blog.hanschen.org/2010/04/01/hide-window-border-for-maximized-windows/)
* [Reddit](https://www.reddit.com/r/firefox/comments/a4z3bl/instructions_to_hide_android_status_bar_when/)
