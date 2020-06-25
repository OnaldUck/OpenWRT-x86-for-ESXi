# OpenWRT-x86-for-ESXi
Es werden hier zwei Optinen vorgestellt. Selber kompieleren und einfach eine VMDK herunterladen und eine virtuelle Maschine damit erstellen.

Es ist kein Vollständige 100% funktionierende Anleitung. Ich werde es versuchen aktuell zu halten, aber es kommen immer wieder kleine Fehler/Probleme die man selbst lösen muss.
Manchmal wird eine **combined-esxi.vmdk** Datei erstellt und machmal nicht (diese muss man vor dem Einsatz erst konverteren)

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
Anschliessend starten wir die Menügeführte / Textbasierende Konfiguration mit `make menuconfig` und wählen folgende Punkte:

+ Target System > **x86**
+ Subtarget > **x86_64**
+ Target Profile > **Generic x86/64**
+ Target Images
  + Build VMware image files (VMDK) > X
  + Seconds to wait before booting the default entry | 1

Wenn alles erledigt ist wir mit dem Befehl `make` das Image kreirt. Wenn man z.B. 6 Kerne CPU besitzt kann man den Parameter -j6 hinzufügen. Das dauert 10-15 Minuten beim ersten Mal.

`make -j6`

### git Ordner aktualisieren und frisch kompilieren
git pull

### Fertiges Image 
https://downloads.openwrt.org/releases/19.07.3/targets/x86/64/
