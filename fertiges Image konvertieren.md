### Fertiges Image konvertieren
Es gibt auch die Möglichkeit sich ein fertiges Image https://downloads.openwrt.org/releases/19.07.3/targets/x86/64/ herunter zu laden un zu konvertieren.
Es sollte `combines-ext4.img.gz` gewählt werden.

Anschliessend folgen 3 Schritte durchführen:

+ die Datei `combines-ext4.img.gz` entpacken
+ mit `qemu-img.exe` die Datei `openwrt-19.07.3-x86-64-combined-ext4.img` konvertieren

`qemu-img.exe convert -f raw -O vmdk a:\openwrt-x86-64-generic-ext4-combined.img a:\openwrt-ext4.vmdk`
+ auf den ESX kopieren und eie VMDK-flat-Datei erzeugen

`vmkfstools -i /vmfs/volumes/ssd/Openwrt/openwrt-ext4.vmdk -d thin /vmfs/volumes/ssd/Openwrt/Openwrt.vmdk`

#### Einschänkunger
Aus meiner Erfahrung funktionieren nicht alle Treiber die in so ein Image nachträglich installiert werden. z.B. wenn man WLAN Stick oder Karte betreibenmöchte.

Für die 'normale' Router oder VPN Funktionalität, sind diese Images uneingeschänkt nutzbar.
