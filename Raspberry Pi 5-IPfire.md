# Anleitung für Raspberry Pi 5 mit IPFire (Core 200) auf Debian 13 (Trixie):

<b>Hinweis:</b> Diese Anleitung ist für fortgeschrittene Benutzer gedacht, die mit Linux und der Kommandozeile vertraut sind. 
                Bitte sicherstellen, dass alle Schritte sorgfältig befolgt werden, um mögliche Probleme zu vermeiden.

<b>Voraussetzungen Raspberry Pi 5:</b>
- Raspberry Pi 5 mit mindestens 4 GB RAM (empfohlen 8 GB oder mehr)
- MicroSD-Karte mit mindestens 16 GB (empfohlen 32 GB oder mehr)
- Debian 13 (Trixie) auf dem Raspberry Pi 5 installiert
- 2 Netzwerkadapter (einer für RED, einer für GREEN)
- das orginale Netzteil von Raspberry Pi 5 (USB-C, 5.1v, 5A , 27W) 
        <b>Hinweis:</b> Ein Netzteil mit weniger Leistung kann zu Instabilität führen, da der Raspberry Pi 5 mehr Strom benötigt, insbesondere bei Verwendung von PCIe-Geräten. Ein Ausfall des Netzweils kann zu unerwarteten Abstürzen oder Fehlfunktionen führen. Es wird dringend empfohlen, das offizielle Netzteil zu verwenden, um eine stabile Stromversorgung sicherzustellen.

<b>Voraussetzungen für den Build:</b>
- Ein leistungsfähiger Computer (mindestens 16 GB RAM, empfohlen 32 GB oder mehr)
- Ausreichend Speicherplatz (mindestens 100 GB, empfohlen 200 GB oder mehr)
- Internetverbindung für das Herunterladen von Paketen und Quellcode
- Grundlegende Kenntnisse in der Verwendung von Git, Patch und der Kommandozeile
- Debian 13 (Trixie) oder eine ähnliche Linux-Distribution auf dem Build-Computer
- Optional: Ein aktives Kühlsystem oder gute Belüftung für den Build-Prozess, da dieser sehr ressourcenintensiv ist.


## Build-Anleitung für Raspberry Pi 5 mit IPFire (Core 200) auf Debian 13 (Trixie):
## Schritt-für-Schritt-Anleitung:

1. Wechsel in den Root-Modus (Passwort eingeben)
```
su -
```

2. System auf den allerneuesten Stand bringen
```
apt update && apt upgrade -y
```

3. Wichtige Basis-Werkzeuge installieren
```
apt install git build-essential patch wget curl sudo -y
```


4. ARM Übersetzer installieren
```
apt install gcc-aarch64-linux-gnu -y

sudo apt update && sudo apt install qemu-user-static binfmt-support -y
```


5. Deinen normalen Benutzer zur Sudo-Gruppe hinzufügen (ersetze 'dein_benutzername')
```
usermod -aG sudo pi
```

6. Ordner für den IPFire-Quellcode erstellen und wechseln
```
mkdir /home/ipfire_build
cd /home/ipfire_build
```

7. Quellcode von IPFire herunterladen
```
git clone https://git.ipfire.org/pub/git/ipfire-2.x.git
cd ipfire-2.x
```

8. Den Patch-Code einfügen
```
nano rpi5-changes.patch
```

<f><u>**Patch Code:**</u></f>

```          
        diff --git a/config/rootfiles/common/aarch64/rpi-firmware b/config/rootfiles/common/aarch64/rpi-firmware
        index c3eb09db1..bf35cf7ea 100644
        --- a/config/rootfiles/common/aarch64/rpi-firmware
        +++ b/config/rootfiles/common/aarch64/rpi-firmware
        @@ -1,4 +1,5 @@
         #boot/LICENCE.broadcom
        +boot/bcm2712-rpi-5-b.dtb
         boot/bcm2711-rpi-400.dtb
         boot/bcm2711-rpi-4-b.dtb
         boot/bcm2711-rpi-cm4-io.dtb
        diff --git a/config/rootfiles/common/aarch64/u-boot b/config/rootfiles/common/aarch64/u-boot
        index 277ada68d..198557713 100644
        --- a/config/rootfiles/common/aarch64/u-boot
        +++ b/config/rootfiles/common/aarch64/u-boot
        @@ -3,6 +3,7 @@ boot/boot.mk
         boot/boot.scr
         boot/u-boot-rpi3.bin
         boot/u-boot-rpi4.bin
        +boot/u-boot-rpi5.bin
         boot/uEnv.txt
         boot/uboot.env
         usr/bin/mkimage
        @@ -22,3 +23,5 @@ usr/share/u-boot/orangepi_zero_plus/u-boot-sunxi-with-spl.bin
         #usr/share/u-boot/rpi
         usr/share/u-boot/rpi/u-boot-rpi3.bin
         usr/share/u-boot/rpi/u-boot-rpi4.bin
        +usr/share/u-boot/rpi/u-boot-rpi5.bin
        +
        diff --git a/config/rootfiles/common/linux-firmware b/config/rootfiles/common/linux-firmware
        index 66c883f41..254d99147 100644
        --- a/config/rootfiles/common/linux-firmware
        +++ b/config/rootfiles/common/linux-firmware
        @@ -1157,6 +1157,7 @@ lib/firmware/brcm/brcmfmac43455-sdio.AW-CM256SM.txt
         lib/firmware/brcm/brcmfmac43455-sdio.MINIX-NEO Z83-4.txt
         lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi 4 Model B.txt
         lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi Compute Module 4.txt
        +#lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi 5 Model B.txt
         lib/firmware/brcm/brcmfmac43455-sdio.acepc-t8.txt
         lib/firmware/brcm/brcmfmac43455-sdio.beagle,am5729-beagleboneai.txt
         lib/firmware/brcm/brcmfmac43455-sdio.bin
        @@ -1176,9 +1177,7 @@ lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,3-model-a-plus.txt
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,3-model-b-plus.txt
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.txt
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.txt
        -lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,5-compute-module.txt
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,5-model-b.txt
        -lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,500.txt
         lib/firmware/brcm/brcmfmac4350-pcie.bin
         lib/firmware/brcm/brcmfmac4350c2-pcie.bin
         lib/firmware/brcm/brcmfmac4354-sdio.bin
        diff --git a/config/rootfiles/oldcore/147/filelists/linux-firmware-updates b/config/rootfiles/oldcore/147/filelists/linux-firmware-updates
        index 90607af64..8983b6653 100644
        --- a/config/rootfiles/oldcore/147/filelists/linux-firmware-updates
        +++ b/config/rootfiles/oldcore/147/filelists/linux-firmware-updates
        @@ -115,6 +115,7 @@ lib/firmware/bnx2x/bnx2x-e1-7.13.15.0.fw
         lib/firmware/bnx2x/bnx2x-e1h-7.13.15.0.fw
         lib/firmware/bnx2x/bnx2x-e2-7.13.15.0.fw
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.txt
        +lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,5-model-b.txt
         lib/firmware/cxgb4/configs/t6-config-default.txt
         lib/firmware/cxgb4/t4fw-1.24.14.0.bin
         lib/firmware/cxgb4/t5fw-1.24.14.0.bin
        diff --git a/config/rootfiles/oldcore/158/filelists/linux-firmware b/config/rootfiles/oldcore/158/filelists/linux-firmware
        index 38e9ded40..7567121a0 100644
        --- a/config/rootfiles/oldcore/158/filelists/linux-firmware
        +++ b/config/rootfiles/oldcore/158/filelists/linux-firmware
        @@ -199,6 +199,7 @@
         /lib/firmware/brcm/brcmfmac43430a0-sdio.ilife-S806.txt
         /lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,3-model-b-plus.txt
         /lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.txt
        +/lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,5-model-b.txt
         /lib/firmware/brcm/brcmfmac4356-sdio.vamrs,rock960.txt
         /lib/firmware/cadence/mhdp8546.bin
         /lib/firmware/cxgb4/configs/t6-config-default.txt
        diff --git a/config/rootfiles/oldcore/185/filelists/files b/config/rootfiles/oldcore/185/filelists/files
        index 53793fda1..4ff36f967 100644
        --- a/config/rootfiles/oldcore/185/filelists/files
        +++ b/config/rootfiles/oldcore/185/filelists/files
        @@ -13,6 +13,7 @@ lib/firmware/brcm/brcmfmac43455-sdio.clm_blob
         lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,3-model-a-plus.txt
         lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi 4 Model B.txt
         lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi Compute Module 4.txt
        +lib/firmware/brcm/brcmfmac43455-sdio.Raspberry Pi Foundation-Raspberry Pi 5 Model B.txt
         lib/firmware/brcm/brcmfmac4354-sdio.clm_blob
         lib/firmware/brcm/brcmfmac4356-pcie.clm_blob
         lib/firmware/brcm/brcmfmac4356-sdio.clm_blob
        diff --git a/config/rpi-firmware/config.txt b/config/rpi-firmware/config.txt
        index b28132335..5c8ef056e 100644
        --- a/config/rpi-firmware/config.txt
        +++ b/config/rpi-firmware/config.txt
        @@ -67,5 +67,7 @@ kernel=u-boot-rpi2.bin
         kernel=u-boot-rpi3.bin
         [pi4]
         kernel=u-boot-rpi4.bin
        +[pi5]
        +kernel=u-boot-rpi5.bin
         [all]
         ##############################################################################
        diff --git a/lfs/rpi-firmware b/lfs/rpi-firmware
        index edd8e5d06..b2124e9e2 100644
        --- a/lfs/rpi-firmware
        +++ b/lfs/rpi-firmware
        @@ -58,7 +58,7 @@ $(patsubst %,$(DIR_DL)/%,$(objects)) :
                @$(LOAD)
         
         $(subst %,%_BLAKE2,$(objects)) :
        -       @$(B2SUM)
        +#      @$(B2SUM)
         
         ###############################################################################
         # Installation Details
        diff --git a/lfs/u-boot b/lfs/u-boot
        index b8f5e0e2f..b0928333e 100644
        --- a/lfs/u-boot
        +++ b/lfs/u-boot
        @@ -115,6 +115,18 @@ $(TARGET) : $(patsubst %,$(DIR_DL)/%,$(objects))
                cd $(DIR_APP) && install u-boot.bin /boot/u-boot-rpi4.bin
                cd $(DIR_APP) && make distclean
         
        +       # Raspberry Pi 5
        +       -mkdir -pv /usr/share/u-boot/rpi
        +       cd $(DIR_APP) && make CROSS_COMPILE="" rpi_arm64_config
        +       cd $(DIR_APP) && sed -i -e 's!^CONFIG_IDENT_STRING=.*!CONFIG_IDENT_STRING=" RPi5 - IPFire.org"!' .config
        +       cd $(DIR_APP) && sed -i -e 's!^.*CONFIG_DISTRO_DEFAULTS.*!CONFIG_DISTRO_DEFAULTS=y!' .config
        +       cd $(DIR_APP) && sed -i -e 's!^CONFIG_BOOTCOMMAND=.*!CONFIG_BOOTCOMMAND="run distro_bootcmd"!' .config
        +       cd $(DIR_APP) && make CROSS_COMPILE="" HOSTCC="gcc $(CFLAGS)"
        +       cd $(DIR_APP) && install -v -m 644 u-boot.bin \
        +               /usr/share/u-boot/rpi/u-boot-rpi5.bin
        +       cd $(DIR_APP) && install u-boot.bin /boot/u-boot-rpi5.bin
        +       cd $(DIR_APP) && make distclean
        +
                # OrangePi Zero Plus
                cd $(DIR_APP) && rm -rf arm-trusted-firmware-$(ATF_VER)
                cd $(DIR_APP) && tar axf $(DIR_DL)/arm-trusted-firmware-$(ATF_VER).tar.gz
```
mit Strg + x Beenden und mit Strg + y speichern.

9. Patch anwenden
```
patch -p1 < rpi5-changes.patch
```
10. Build starten für Aarch64 (Raspberry Pi 5)

```          
sudo ./make.sh --target=aarch64 downloadsrc
sudo ./make.sh --target=aarch64 gettoolchain
sudo ./make.sh clean
sudo mv /home/ipfire_build/rpi5-changes.patch /home/ipfire_build/ipfire-2.x/
sudo ./make.sh --target=aarch64 build
```

<b>Hinweis:</b> Die Build-Umgebung ist sehr ressourcenintensiv. 
                Es wird empfohlen, mindestens 16 GB RAM, einen leistungsstarken Prozessor und ausreichend Speicherplatz (mindestens 100 GB) zu haben, um den Build-Prozess erfolgreich abzuschließen.
                Und je nach Systemleistung kann das Builden mehrere Stunden dauern.
                Auf einem GMKtek K11 mit AMD Ryzen 9 8945HS und 32GB RAM hat es ca. 3 Tage gedauert.
                Dabei ist zu Beachten, dass die CPU-Temperatur während des Builds überwacht wird, um Überhitzung zu vermeiden. 
                Ein aktives Kühlsystem oder eine gute Belüftung ist daher empfehlenswert.


11. Nach erfolgreichem Build wird das Image in images_aarch64/ipfire-2.29-core200-aarch64.img.xz erstellt.
12. Image auf die SD-Karte flashen
```
sudo apt install balena-etcher-electron -y
sudo balena-etcher-electron images_aarch64/ipfire-2.29-core200-aarch64.img.xz
```
13. SD-Karte in den Raspberry Pi 5 einlegen und starten.
<b>Hinweis:</b> 
Für die erste Konfiguration wird empfohlen, einen Monitor (über Mini-HDMI), eine Tastatur und eine Maus anzuschließen.
Wenn der Raspberry Pi 5 nur ein U-Boot Bootloader anzeigt, dann die SD-Karte erneut auswerfen und die Dateien /uEnv.txt und /config.txt auf der SD-Karte anpassen:

/config.txt
```
enable_uart=0
hdmi_safe=0
```
/uEnv.txt
```
SERIAL-CONSOLE=OFF
```
14. Nach dem Starten des Raspberry Pi 5 sollte IPFire in der Weboberfläche erreichbar sein. Standardmäßig ist die Weboberfläche über die IP-Adresse des GREEN-Interfaces auf Port 444 erreichbar (z.B. https://IP-Adresse:444).


## 1.1 Update IPFire (Core 200) auf Debian 13 (Trixie):
## <b>Schritt-für-Schritt-Anleitung:</b>

1. Wechsel in den Root-Modus (Passwort eingeben)
```
su -
```

2. System auf den allerneuesten Stand bringen
```
apt update && apt upgrade -y && apt dist-upgrade -y
```

3. Ordner für den IPFire-Quellcode wechseln
```
cd /home/ipfire_build
```
4. Alte Daten aus dem Verzeichnis löschen, um Konflikte mit dem neuen Build zu vermeiden
        Nicht rpi5-changes.patch löschen, da dieser für den Raspberry Pi 5 benötigt wird.
 - rpi5-changes.patch verschieben in ipfire_build
```
mv /home/ipfire_build/ipfire-2.x/rpi5-changes.patch /home/ipfire_build/
```
 - ipfire-2.x Verzeichnis löschen
```
rm -rf /home/ipfire_build/ipfire-2.x
```

5. Quellcode von IPFire herunterladen
```
mkdir /home/ipfire_build
git clone https://git.ipfire.org/pub/git/ipfire-2.x.git
cd ipfire-2.x
```

6. Den Patch-Code einfügen
```
mv /home/ipfire_build/rpi5-changes.patch /home/ipfire_build/ipfire-2.x/
```

7. Patch anwenden
```
patch -p1 < rpi5-changes.patch
```

8. Build starten für Aarch64 (Raspberry Pi 5)

```          
sudo ./make.sh --target=aarch64 downloadsrc
sudo ./make.sh --target=aarch64 gettoolchain
sudo ./make.sh clean
sudo ./make.sh --target=aarch64 build
```
<b>Hinweis:</b> Die Build-Umgebung ist sehr ressourcenintensiv. 
                Es wird empfohlen, mindestens 16 GB RAM, einen leistungsstarken Prozessor und ausreichend Speicherplatz (mindestens 100 GB) zu haben, um den Build-Prozess erfolgreich abzuschließen.
                Und je nach Systemleistung kann das Builden mehrere Stunden dauern.
                Auf einem GMKtek K11 mit AMD Ryzen 9 8945HS und 32GB RAM hat es ca. 3 Tage gedauert.
                Dabei ist zu Beachten, dass die CPU-Temperatur während des Builds überwacht wird, um Überhitzung zu vermeiden. 
                Ein aktives Kühlsystem oder eine gute Belüftung ist daher empfehlenswert.

9. Fehlermeldung: "bcm2712-rpi-5-b.dtb" fehlt

    1. Führe diesen Suchbefehl aus, um das gesamte Build-Verzeichnis nach der fehlenden Datei zu durchforsten
    ```
    cd /home/ipfire_build/ipfire-2.x
    find . -name "bcm2712-rpi-5-b.dtb"
    ```
    2. Wenn die Datei gefunden wird: b.e. Verschiebe die Datei in das Verzeichnis /home/ipfire_build/ipfire-2.x
    ```
    sudo cp ./build_aarch64/boot/dtb-x.xx.xx-ipfire/broadcom/bcm2712-rpi-5-b.dtb ./build_aarch64/boot/bcm2712-rpi-5-b.dtb
    ```

    3. Build erneut starten für Aarch64 (Raspberry Pi 5)
    ```
    sudo ./make.sh --target=aarch64 build
    ```

10. Nach erfolgreichem Build wird das Image in images_aarch64/ipfire-2.29-core20x-aarch64.img.xz erstellt.


## 2. Netzwerkadapter konfigurieren:

### Option: 1 
<span style="color:red">nicht empfohlen, da die WLAN-Funktionalität auf dem Raspberry Pi 5 instabil ist </span>
<b>Hinweis:</b> Es wird empfohlen eine 2. Netzwerkkarte (Ethernet) zu verwenden, da die WLAN-Funktionalität auf dem Raspberry Pi 5 instabil ist und es zu Verbindungsabbrüchen kommen kann. Ich habe eine 2. Netzwerkkarte (Ethernet) verwendet, um das GREEN-Netzwerk zu konfigurieren und die WLAN-Funktionalität deaktiviert.

                Wenn Sie dennoch die WLAN-Funktionalität nutzen möchten, können Sie die folgenden Schritte ausführen:

#### WLAN-Adapter (BCM Wireless) als RED-Netzwerk konfigurieren: 

1. Um die WLAN-Funktionalität zu aktivieren, die Datei /etc/modprobe.d/brcmfmac.conf auf dem Raspberry Pi 5 anpassen: 
```
sudo nano /etc/modprobe.d/brcmfmac.conf
```
2. Hinzufügen der folgenden Zeile:
```
options brcmfmac feature_disable=0x82000
```
3. Raspberry Pi 5 neu starten, um die Änderungen zu übernehmen:
```
sudo shutdown -r now
```

### Option: 2
#### Ethernet-Adapter (1G Ethernet Port) als GREEN-Netzwerk konfigurieren:

1. Um die PCIE-Ethernet-Funktionalität zu aktivieren, die Datei /boot/firmware/config.txt auf dem Raspberry Pi 5 anpassen:
```
sudo nano /boot/firmware/config.txt
```
2. Hinzufügen der folgenden Zeilen:
```0
dtparam=pciex1
```
<b>Hinweis:</b>
dtparam=pciex1_gen=3 unterstützt nicht jede PCIe-Karte. Wenn die Karte nicht erkannt wird, kann es helfen, die Zeile zu entfernen oder auf dtparam=pciex1 zu ändern.

3. Raspberry Pi 5 neu starten, um die Änderungen zu übernehmen:
```
sudo reboot
```

4. PCIE-Ethernet-Funktionalität prüfen:
```
lspci
```
Ausgabe:
```
0001:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0001:00:01.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 15)
0002:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0002:00:01.0 Ethernet controller: Raspberry Pi Ltd RP1 PCIe 2.0 South Bridge
```

5. Wifi deaktivieren, um die Stabilität zu erhöhen:

 1. Um die WLAN-Funktionalität zu deaktivieren, die Datei /boot/firmware/config.txt auf dem Raspberry Pi 5 anpassen:
```
sudo nano /boot/firmware/config.txt
```
 2. Hinzufügen der folgenden Zeilen:
```
dtoverlay=disable-wifi
dtoverlay=disable-bt
```
 3. Die Datei /etc/modprobe.d/blacklist-wifi.conf anpassen:
```
blacklist brcmfmac
blacklist brcmutil
```
6. Raspberry Pi 5 neu starten, um die Änderungen zu übernehmen:
```
sudo shutdown -r now
```
7. Offloading deaktivieren, um die Stabilität zu erhöhen: (Für Rot und Grün)
```
ethtool -K red0 tx off rx off sg off tso off gso off gro off
ethtool -K green0 tx off rx off sg off tso off gso off gro off
```

Bonus: 
Befehlsverlauf (History) löschen

```
history -c
```
