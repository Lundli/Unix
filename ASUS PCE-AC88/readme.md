**NOTE: This is copied from https://tabre.com/post/2/, all credit due to Tabre. Uploading it here for safekeeping**
--------------------------------------

ASUS PCE-AC88 Wifi on Ubuntu
Broadcom Firmware Hack
Extracting the PCE-AC88 firmware from the RT-AC88 router firmware and loading it into the broadcom driver for Linux.
By Tabre

Dec. 18, 2017, 10:37 a.m.
The ASUS PC-AC88 is one of the top performing NICs on the market but when it comes to Linux support, the chipset manufacturer, Broadcom has been less than helpful in providing a proper firmware for this particular NIC. As such, Linux users are left to fend for themselves in making this NIC work on their systems. I spent a couple hours last night trying to make this thing work and after scouring the internet and combing information from various sources (which I will link at the end of this post) I finally came to a solution that works. Because there was no single source which contained all the information needed, I decided to create this guide, documenting what I've learned about making this card work with Ubuntu 17.10. I've split the guide into two sections: a TLDR version for those who want a simple and easy to follow set of instructions to download the firmware and get them up and running quickly, and a detailed guide for those who really want to understand how we were able to get this firmware and enjoy a little hacking around in a hex editor.

TLDR Section:

1. Download  brcmfmac4366c-pcie.bin

2. Create a copy of brcmfmac4366c-pcie.bin with the .txt extension: brcmfmac4366c-pcie.txt

3. Place brcmfmac4366c-pcie.bin and brcmfmac4366c-pcie.txt in the /lib/firmware/brcm directory

4. Reboot.

5. Your wifi adapter should now be able to detect wireless networks but will probably fail to connect if you're using WEP, WPA or WPA2, as most people do these days. Attempt to connect to your network using the network manager. This will cause the wpa_supplicant config file for that particular connection to be created in the /etc/NetworkManager/system-connections/

6. Navigate to the /etc/NetworkManager/system-connections/ directory and use nano (or your favorite text editor) to open the config file named after the SSID of your connection. This will probably require root permission so use sudo nano. At the end of the document, add the following line: 

p2p_disabled=1

7. Reboot. 

That's it. You should now be able to connect to your wireless network using your ASUS PCE-AC88. 



Extracting Your Own brcmfmac4366c-pcie.bin From the RT-AC88 Router Firmware:
(excerpt from n3roj2's post at ubuntuforums.org)

1.) Go to the Asus support page and download the "other" firmware for RT-AC88U.

- I'm using Version3.0.0.4.380.7266

2.) Extract the firmware using binwalk: binwalk -e RT-AC88U_3.0.0.4_380_7266-g6439257.trx

3.) Enter the _RT-AC88U_3.0.0.4_380_7266-g6439257.trx.extracted/squashfs-root/lib/modules/2.6.36.4brcmarm/kernel/drivers/net/dhd/ directory

- You should see dhd.ko

4.) Use read elf and search for 4366c0: readelf -s dhd.ko | grep 4366c0

219: 000fe818 4 OBJECT GLOBAL DEFAULT 35 active_cons_4366c0
228: 000fe7ec 18 OBJECT GLOBAL DEFAULT 35 dlimagever_4366c0
248: 000fe73c 175 OBJECT GLOBAL DEFAULT 35 dlimagename_4366c0
348: 000fe814 1 OBJECT GLOBAL DEFAULT 35 dlimagetag_4366c0
479: 000fe800 20 OBJECT GLOBAL DEFAULT 35 dlimagedate_4366c0
518: 000fe81c 0xe6450 OBJECT GLOBAL DEFAULT 35 dlarray_4366c0
625: 000022e4 4 OBJECT GLOBAL DEFAULT 45 debug_params_4366c0

5.) 0xe6450 is 943184 in decimal (this is the bin file size)

6.) Open up another brcmfmacXXXX.bin in bless to compare the start and stop tags for another firmware (I'm using brcmfmac4366b-pcie.bin)

7.) We see a start hex of "00 F2 3E B8" as a starting point and "FWID: 01-c47a91a4...." as ending (4 bytes past the denoted as periods

8.) Search in dhd.ko using bless for "00 F2 3E B8"; Address at 0x137bbc which is 1276860 in decimal

9.) Copy out the firmware using dd: dd if=dhd.ko skip=1276860 ibs=1 count=943184 of=brcmfmac4366c-pcie.bin

10.) Open up the new brcmfmac4366c-pcie.bin file in Bless to check it out and ensure that we start with "00 F2 3E B8" and end with the FWID: "FWID: 01-fed440e1...."



Sources:

- Firmware Extraction by n3roj2 at ubuntuforums.org: https://ubuntuforums.org/showthread.php?t=2337200
- Informative Discussion at vivaolinux.com: https://www.vivaolinux.com.br/topico/Rede-Wireless/PCE-AC88-Ubuntu
- P2P issue solution by Hellacopter: https://forums.fedoraforum.org/showthread.php?310626-Fedora-24-and-ASUS-PCE-AC-88
- wpa_supplicant.conf files: https://askubuntu.com/questions/497540/where-is-my-wpa-supplicant-conf

