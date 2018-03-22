## L4T on Nvidia Shield TV
### Transforming streaming device into a portable computer

---
### Intro
Shield TV is a powerful streaming device from Nvidia.  
It is the only streaming device in the market with Tegra X1 GPU at this moment, similar hardware spec like Jetson TX1, little lesser I/O options but much more portable. 

I decided to try this because I don't want display.
Many hotel rooms are equipped a TV with HDMI port, and I use a 55-inch TV as a monitor at home. So, there is no good reason for me to get a secondary full size computer.
On the other hand, most other set-top style computers are equipped with either weak or no GPU.

The purpose of this doc is to keep every relevant info in one place, rather than a code-by-code guideline.

### Preparation
1. Nvidia Shield TV
 - ***It is recommended to use "Pro version" (500GB)*** although it's more expensive than two other versions (original and 2017 version). This version is not only has bigger size but less vulnerable of getting bricked if anything goes wrong. Read "Warning/Backup" section for more detail.
 I already hard bricked two non-pro versions during this project and had to return.
2. If it's "Pro" version, you might want to buy OTG. Although there are multiple USB ports, it appears micro USB works better than other ports and there is only one micro USB ports. I experienced dropping signal or simply not detecting devices at all while micro USB could.
3. You might also want to get a USB splitter port like this one for the same reason you need OTG. You may need more than one USB ports for mouse/keyboard dongles, external HDD, USB, etc, but there's only one micro USB.
[https://www.amazon.com/Belkin-USB-2-0-4-Port-Ultra-Mini/dp/B000Q8UAWY/ref=sr_1_3?s=electronics&ie=UTF8&qid=1521745133&sr=1-3&keywords=belkin+usb+4+port](https://www.amazon.com/Belkin-USB-2-0-4-Port-Ultra-Mini/dp/B000Q8UAWY/ref=sr_1_3?s=electronics&ie=UTF8&qid=1521745133&sr=1-3&keywords=belkin+usb+4+port)
This is one I have and confirmed to work. However, it's pretty old and expensive (I bought it at $10). You'd want to look for a better, newer model.
4. Host computer with Linux

### Rooting with ADB and Fastboot
Just like every android rooting, we need ADB and Fastboot.
There is a clear, beginner friendly rooting guide of Nvidia Shield TV can be found here.
[https://nvidiashieldzone.com/shield-android-tv/](https://nvidiashieldzone.com/shield-android-tv/)

### Warning/Backup
Every Nvidia device, just like JetsonTX1, including Nvidia Shield TV, is built in aarch64 architecture with custom GPT partitioning. Because of this, the device will not be detected by any disk management/partition programs including popular GNOME disks and gparted. I will discuss about this partition in later/other section in more detail.

There are soft brick and hard brick.
You can recover soft brick with flashing OS image through adb/fastboot. There is a guide to flashing stock OS image in the link to the rooting guide provided above, too.

For hard brick, dd is the only option to recover hard bricked Shield TV as far as I know. Even HDD cloning with docking station didn't work on my case.

This is why you'd want to use Pro version. In Pro version you can disassemble Shield TV and manually recover 500GB sshd with backup image through dd.

To summarize, it's recommended to create and keep your own recovery image with dd in case anything goes wrong.

***MAKE SURE YOU UNDERSTAND THE RISK OF DD COMMAND.***  
***THIS CANNOT ONLY BRICK SHIELD TV BUT ALSO WIPE OUT THE HOST COMPUTER***  
When you do dd, it is wise to work on secondary computer if possible and make sure you unplug everything else, other than target HDD.  

See following link for dd example command.  
[https://wiki.archlinux.org/index.php/disk_cloning](https://wiki.archlinux.org/index.php/disk_cloning)  

Make sure you have enough storage.
You'd notice original SSHD equipped on Shield TV is 500GB.
This means backup image from dd will be also 500GB.

You may also create partial backup image.  
See this link for the guide on partial backup/HDD migration.  
[https://forum.xda-developers.com/shield-tv/general/guide-migrate-to-ssd-hdd-size-satv-pro-t3440195](https://forum.xda-developers.com/shield-tv/general/guide-migrate-to-ssd-hdd-size-satv-pro-t3440195)

### Flashing L4T
You can find the L4T flashing guide here:
[https://forum.xda-developers.com/shield-tv/general/build-kernel-source-boot-to-ubuntu-t3274632](https://forum.xda-developers.com/shield-tv/general/build-kernel-source-boot-to-ubuntu-t3274632)
L4T 24.2.1 is the latest version known to be working.
28+ versions could be flashed but with many problems according to many others already tried.

Also, you may need a little improvise after understanding the overall procedure.  
For example, flashing through USB flash drive never worked on me, and it happened "mmcblk0p1.img" worked on external SD card, not "mmcblk1p1.img"

### After flashing
Just like Jetson, type following in terminal before you do any update/upgrade to make sure "libglx.so" is not overwritten by apt.
`sudo apt-mark hold xserver-xorg-core`
You may unhold this temporarily whenever you need to install some non-system related packages.

### Next
You will notice you need SD card plugged in every time you boot L4T from Shield TV. I'm currently working on editing the partition table to add another boot partition. This may require GPT/hex editing on bin. I will update this doc once I come up with solid working solution.
