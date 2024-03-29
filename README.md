# OpenWRT-x86-for-ESXi
Es werden hier zwei Optionen vorgestellt. Selber kompieleren und einfach eine VMDK herunterladen und eine virtuelle Maschine damit erstellen.

Es ist keine Vollständige und 100% funktionierende Anleitung. Ich werde es versuchen aktuell zu halten, aber es kommen immer wieder kleine Fehler/Probleme die man selbst lösen muss. Als beispiel, manchmal wird eine `openwrt-x86-64-combined-ext4-esxi.vmdk` und `openwrt-x86-64-combined-ext4-esxi-flat.vmdk` Datei erstellt und machmal nicht (*diese muss man vor dem Einsatz erst zurecht konverteren*)

#### Einsatzzweck
Virtueller Router mit einen WLAN Accespoint für einen ESX Server. Das heißt zwar eine VM die jedoch eine PCIe WLAN Karte durchgescheift bekommt oder ein USB WLAN Sick angehängt wird.

Ein Mal USB-Stick im AP Mode und ein mal Intel MiniPCIe Karte im AP-Mode. *(nicht alle WLAN Karten oder USB Stick lassen sich unter OpenWRT im AP Modus betreiben, obwohl sie unter Linux oder Windows dies tun)*

Ich habe zwei duch probieren gefunden
+ TP-Link
+ Intel PCIe 8260

>Falls Sie die 'Build Umgebung' in eine VM betreiben, dann vor jeden Update ein Snapshots erstellen !

## Selbst kompilieren
Wie man das Betriebssystem vorbereitet, steht auf der OpenWRT Seite hier: [Build system – Setup Linux!](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem).
Theoretisch folgende Zeile abfeuern `apt install subversion build-essential libncurses5-dev zlib1g-dev gawk git ccache gettext libssl-dev xsltproc zip qemu-utils` und fertig.

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
+ Kernel modules > Network Devices
  + kmod-vmxnet3 | **x**
+ Utilities
  + open-vm-tools | **x**
+ LuCI > 1. Collections
  + luci | **x**
+ Administration
  + htop | **x**


Falls man keinen OpenVPN Support oder WLAN Support benötigt, so kann man auf den Rest verzichten und direkt kompilieren.
Hier die benötigten Module für einen WLAN-Access Poiint

+ Kernel modules > USB Support
  + kmod-usb-core | **x**
+ Kernel modules > Wireless Drivers
  + kmod-iwlwifi | **x**
  + kmod-mt76x0u | **x**
+ Firmware
  + iwlwifi-firmware-iwl8260c | **x**
  + rtl8192eu-firmware | **x**
+ Network > WirelessAPD
  + wpad | **x**

Dies braucht man, wenn man OpenVPN Server oder Client auf den Router laufen lassen möchte.

+ LuCI > 3. Applications
  + luci-app-openvpn | **x**
  + luci-app-statistics | **x**
  + luci-app-wireguard | **x**
+ Network > VPN
  + openvpn-openssl | **x**
  + wireguard | **x**

>Das fette **x** bedeutet Sternchen. Ich weiß nicht, wie man es hier darstellt.

#### Balast entfernen
Bei der Gelegenheit unnütze Treiber entfernen. Den Stern entfernen

+ Kernel modules > Network Devices
  + kmod-bnx2
  + kmod-e1000e
  + kmod-forcedeth
  + kmod-igb
  + kmod-ixgbe
  + kmod-r8169
  + kmod-ph-realtek
  + kmod-mii
  + komd-libphy

+ Firmware
  + bnx2-firmware
  + r8192-firmware

### Kompilieren
Wenn alles angehackt ist wir mit dem Befehl `make` das Image kreirt. Wenn man z.B. 6 Kerne CPU besitzt kann man den Parameter -j6 hinzufügen. Das Ganze dauert 10-15 Minuten beim ersten Mal.

`make -j6`

Das fertige Image befindet sich in `~/openwrt/bin/targets/x86/64/`

### VMDK flat erzeugen
Damit der ESXi Server das Image akzeptiert muss die fertig kompilierte Datei `openwrt-x86-64-generic-ext4-combined.img` auf den ESX kopiert werden und eine VMDK-flat-Datei erzeugen werden.

`vmkfstools -i /vmfs/volumes/ssd/Openwrt/
openwrt-x86-64-generic-ext4-combined.img -d thin /vmfs/volumes/ssd/Openwrt/Openwrt.vmdk`

### Ein paar Bilder
![make_menuconfig](https://user-images.githubusercontent.com/35377000/86009497-95997680-ba1a-11ea-89e8-fca909e72438.png)

## Bei Problemen, alles wieder neu machen ;-( ##

```
make clean && make tools/clean && make toolchain/clean
make menuconfig
./scripts/feeds update -a
./scripts/feeds install -a
make tools/install
make toolchain/install
```
