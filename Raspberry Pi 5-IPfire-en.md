# English translation of the instructions for Raspberry Pi 5 with IPFire (Core 200) on Debian 13 (Trixie):

<b>Note:</b> This guide is intended for advanced users who are familiar with Linux and the command line. 
                Please ensure that all steps are followed carefully to avoid potential issues.

<b>Requirements for Raspberry Pi 5:</b>
- Raspberry Pi 5 with at least 4 GB RAM (recommended 8 GB or more)
- MicroSD card with at least 16 GB (recommended 32 GB or more)
- Debian 13 (Trixie) installed on the Raspberry Pi 5
- 2 network adapters (one for RED, one for GREEN)
- the original power supply of Raspberry Pi 5 (USB-C, 5.1v, 5A, 27W) 
        <b>Note:</b> A power supply with lower output may lead to instability, as the Raspberry Pi 5 requires more power, especially when using PCIe devices. A power supply failure can result in unexpected crashes or malfunctions. It is strongly recommended to use the official power supply to ensure stable power delivery.

<b>Requirements for the build:</b>
- A powerful computer (at least 16 GB RAM, recommended 32 GB or more)
- Sufficient storage space (at least 100 GB, recommended 200 GB or more)
- Internet connection for downloading packages and source code
- Basic knowledge of using Git, Patch, and the command line
- Debian 13 (Trixie) or a similar Linux distribution on the build computer
- Optional: An active cooling system or good ventilation for the build process, as it is very resource-intensive.

<b>Step-by-step instructions:</b>

1. Switch to root mode (enter password)
```
su -
```

2. Update the system to the latest version
```
apt update && apt upgrade -y
```

3. Install essential base tools
```
apt install git build-essential patch wget curl sudo -y
```

4. Install ARM cross-compiler
```
apt install gcc-aarch64-linux-gnu -y

sudo apt update && sudo apt install qemu-user-static binfmt-support -y
```

5. Add your regular user to the sudo group (replace 'your_username')
```
usermod -aG sudo pi
```

6. Create a directory for the IPFire source code and navigate to it
```
mkdir /home/ipfire_build
cd /home/ipfire_build
```

7. Download the IPFire source code
```
git clone https://git.ipfire.org/pub/git/ipfire-2.x.git
cd ipfire-2.x
```

8. Insert the patch code
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
with Ctrl + x to exit and Ctrl + y to save.

9. Apply the patch
```
patch -p1 < rpi5-changes.patch
```

10. Start the build for Aarch64 (Raspberry Pi 5)
```
sudo ./make.sh --target=aarch64 downloadsrc
sudo ./make.sh --target=aarch64 gettoolchain
sudo ./make.sh clean
sudo ./make.sh --target=aarch64 build
```
<b>Note:</b> The build environment is very resource-intensive. 
                It is recommended to have at least 16 GB RAM, a powerful processor, and sufficient storage space (at least 100 GB) to successfully complete the build process.
                Depending on system performance, the build can take several hours.
                On a GMKtek K11 with AMD Ryzen 9 8945HS and 32GB RAM, it took about 3 days.
                It is important to monitor the CPU temperature during the build to avoid overheating. 
                An active cooling system or good ventilation is therefore recommended.

11. After a successful build, the image will be created in images_aarch64/ipfire-2.29-core200-aarch64.img.xz.
12. Flash the image to the SD card
```
sudo apt install balena-etcher-electron -y
sudo balena-etcher-electron images_aarch64/ipfire-2.29-core200-aarch64.img.xz
```
13. Insert the SD card into the Raspberry Pi 5 and start it.
<b>Note:</b> For the initial configuration, it is recommended to connect a monitor (via Mini-HDMI), a keyboard, and a mouse.
If the Raspberry Pi 5 only shows a U-Boot bootloader, eject the SD card again and modify the files /uEnv.txt and /config.txt on the SD card:
/config.txt
```
enable_uart=0
hdmi_safe=0
```
/uEnv.txt
```
SERIAL-CONSOLE=OFF
```

14. After starting the Raspberry Pi 5, IPFire should be accessible in the web interface. By default, the web interface is accessible via the IP address of the GREEN interface on port 444 (e.g., https://IP-address:444).


## 1.1 Update IPFire (Core 200) to Debian 13 (Trixie):
## <b>Step-by-step instructions:</b>

1. Switch to root mode (enter password)
```
su -
```

2. Update the system to the latest version
```
apt update && apt upgrade -y
```

3. Change to the folder for the IPFire source code
```
cd /home/ipfire_build
```

4. Delete old data from the directory to avoid conflicts with the new build
        Do not delete rpi5-changes.patch, as it is needed for the Raspberry Pi 5.
 - Move rpi5-changes.patch to ipfire_build
```
mv /home/ipfire_build/ipfire-2.x/rpi5-changes.patch /home/ipfire_build/
```
 - Delete the ipfire-2.x directory
```
rm -rf /home/ipfire_build/ipfire-2.x
```
5. Download the IPFire source code
```
mkdir /home/ipfire_build
git clone https://git.ipfire.org/pub/git/ipfire-2.x.git
cd ipfire-2.x
```

6. Insert the patch code
```
mv /home/ipfire_build/rpi5-changes.patch /home/ipfire_build/ipfire-2.x/
```

7. Apply the patch
```
patch -p1 < rpi5-changes.patch
```
8. Start the build for Aarch64 (Raspberry Pi 5)
```          
sudo ./make.sh --target=aarch64 downloadsrc
sudo ./make.sh --target=aarch64 gettoolchain
sudo ./make.sh clean
sudo mv /home/ipfire_build/rpi5-changes.patch /home/ipfire_build/ipfire-2.x/
sudo ./make.sh --target=aarch64 build
```
<b>Note:</b> The build environment is very resource-intensive. 
                It is recommended to have at least 16 GB RAM, a powerful processor, and sufficient storage space (at least 100 GB) to successfully complete the build process.
                Depending on system performance, the build can take several hours.
                On a GMKtek K11 with AMD Ryzen 9 8945HS and 32GB RAM, it took about 3 days.
                It is important to monitor the CPU temperature during the build to avoid overheating. 
                An active cooling system or good ventilation is therefore recommended.

9. After a successful build, the image will be created in images_aarch64/ipfire-2.29-core200-aarch64.img.xz.


## 2. Configure network adapters:
### Option: 1
<span style="color:red">not recommended, as WLAN functionality on the Raspberry Pi 5 is unstable </span>
<b>Note:</b> It is recommended to use a second network card (Ethernet), as WLAN functionality on the Raspberry Pi 5 is unstable and may lead to connection drops. I used a second network card (Ethernet) to configure the GREEN network and disabled WLAN functionality. 

                If you still want to use WLAN functionality, you can follow these steps:

#### Configure WLAN adapter (BCM Wireless) as RED network:

1. To enable WLAN functionality, modify the file /etc/modprobe.d/brcmfmac.conf on the Raspberry Pi 5:
```
sudo nano /etc/modprobe.d/brcmfmac.conf
```
2. Add the following line:
```
options brcmfmac feature_disable=0x82000
```

3. Restart the Raspberry Pi 5 to apply the changes:
```
sudo reboot
```
        

### Option: 2

#### Configure Ethernet adapter (1G Ethernet Port) as GREEN network:

1. To enable PCIe Ethernet functionality, modify the file /boot/firmware/config.txt on the Raspberry Pi 5:
```
sudo nano /boot/firmware/config.txt
```

2. Add the following lines:
```
dtparam=pciex1
```

<b>Note:</b>
dtparam=pciex1_gen=3 does not support every PCIe card. If the card is not recognized, it may help to remove the line or change it to dtparam=pciex1.

3. Restart the Raspberry Pi 5 to apply the changes:
```
sudo reboot
```

4. Check PCIe Ethernet functionality:
```
lspci
```
Output:
```
0001:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0001:00:01.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 15)
0002:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0002:00:01.0 Ethernet controller: Raspberry Pi Ltd RP1 PCIe 2.0 South Bridge
```

5. Disable Wi-Fi to increase stability:

 1. To disable WLAN functionality, modify the file /boot/firmware/config.txt on the Raspberry Pi 5:
```
sudo nano /boot/firmware/config.txt
```

2. Add the following lines:
```
dtoverlay=disable-wifi
dtoverlay=disable-bt
```
 3. Modify the file /etc/modprobe.d/blacklist-wifi.conf:
```
blacklist brcmfmac
blacklist brcmutil
```
6. Restart the Raspberry Pi 5 to apply the changes:
```
sudo reboot
```
7. Disable offloading to increase stability: (For RED and GREEN)
```
ethtool -K red0 tx off rx off sg off tso off gso off gro off
ethtool -K green0 tx off rx off sg off tso off gso off gro off
```

<b>Bonus:</b>
Clear command history
```
history -c
```

