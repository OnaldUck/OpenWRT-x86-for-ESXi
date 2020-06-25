# OpenWRT-x86-for-ESXi
Es werden hier zwei Optinen vorgestellt. Selber kompieleren und einfach eine VMDK herunterladen und eine virtuelle Maschine damit erstellen.

Es ist keine Vollständige und 100% funktionierende Anleitung. Ich werde es versuchen aktuell zu halten, aber es kommen immer wieder kleine Fehler/Probleme die man selbst lösen muss. Als beispiel, manchmal wird eine **combined-esxi.vmdk** Datei erstellt und machmal nicht (diese muss man vor dem Einsatz erst konverteren)

#### Einsatzzweck
Router mit einen OpenVPN Client und WLAN Accespoint.

Ein Mal USB-Stick im AP Mode und ein mal Intel MiniPCIe Karte im AP-Mode. *(nicht jede WLAN Karte oder USB Stick lassen sich unter OpenWRT im AP Mode betreiben obwohl sie unter Linux oder Windows dies tum)*

Ich habe zwei duch probieren gefunden
+ TP-Link
+ Intel PCIe 8260

>Falls Sie die 'Build Umgebung' in eine VM betreiben, dann vor jeden Update ein Snapshots erstellen !

## Selber kompilieren
Wie man das Betriebssystem vorbereitet, steht auf der OpenWRT Seite hier: [Build system – Setup Linux!](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem).
Theoretisch folgende Zeile abfeuern `apt install subversion build-essential libncurses5-dev zlib1g-dev gawk git ccache gettext libssl-dev xsltproc zip` und fertig.

Danach werden folgende Kommandos ausgeführt:
```
git clone https://github.com/openwrt/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

### OpenWrt Konfiguration
Anschliessend starten wir die Menügeführte / Textbasierende Konfiguration mit `make menuconfig` und wählen folgende Punkte:

+ Target System > **x86**
+ Subtarget > **x86_64**
+ Target Profile > **Generic x86/64**
+ Target Images
  + Build VMware image files (VMDK) > **x**
  + Seconds to wait before booting the default entry | **1**
+ Administration
  + htop | **x**

Falls man keinen OpenVPN Support oder WLAN Support benötigt, so kann man auf den Rest versichten und direkt kompilieren.
  
+ Kernel modules > Network Devices
  + kmod-vmxnet3 | **x**
+ Kernel modules > USB Support
  + kmod-usb-core | **x**
+ Kernel modules > Wireless Drivers
  + kmod-iwlwifi | **x**
  + kmod-mt76x0u | **x**
+ Firmware
  + iwlwifi-firmware-iwl8260c | **x**
  + rtl8192eu-firmware | **x**
  + wireles-regdb | **x**
+ Network > VPN
  +openvpn-openssl | **x**
+ Network > WirelessAPD
  + wpad | **x**
+ Utilities
  + open-vm-tools | **x**
+ LuCI > 3. Applications
  + luci-app-openvpn | **x**
  + luci-app-statistics | **x**

### Kompilieren
Wenn alles angehackt ist wir mit dem Befehl `make` das Image kreirt. Wenn man z.B. 6 Kerne CPU besitzt kann man den Parameter -j6 hinzufügen. Das dauert 10-15 Minuten beim ersten Mal.

`make -j6`

Das fertige Image befindet sich in `~/openwrt/bin/targets/x86/64/`

### git Ordner aktualisieren und frisch kompilieren
git pull

### Fertiges Image 
https://downloads.openwrt.org/releases/19.07.3/targets/x86/64/
