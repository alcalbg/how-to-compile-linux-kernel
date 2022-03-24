# How to compile and install the latest linux Kernel on Debian 11

The default Kernel on Debian 11 bullseye is 5.10. In this guide, we want to upgrade to linux kernel 5.16.

## Why?
The new Kernel 5.16 includes rtw89 wifi [drivers](https://git.kernel.org/pub/scm/linux/kernel/git/kvalo/wireless-drivers-next.git/commit/?id=e3ec7017f6a20d12ddd9fe23d345ebb7b8c104dd) for Realtek 8852AE 802.11ax PCIe wireless network adapter found on some Lenovo ThinkPad laptops (ThinkPad L14 gen1). This kernel upgrade will also fix bluetooth problems with the same wireless chip.

## Disclaimer
Use this guide at your own risk! Please backup everything before you begin.

## Instructions
Verify the Kernel version on your system with uname:
```
$ uname -r
5.10.0-11-amd64
```

Install required packages and update your system:
```
$ sudo apt update
$ sudo apt install wget build-essential dwarves python3 libncurses-dev flex bison libssl-dev bc libelf-dev
$ sudo apt upgrade
```

Download official firmware for Realtek 8852AE
```
$ sudo mkdir /lib/firmware/rtl_bt/
$ cd /lib/firmware/rtl_bt
$ sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_bt/rtl8852au_config.bin
$ sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_bt/rtl8852au_fw.bin
$ sudo mkdir /lib/firmware/rtw89/
$ cd /lib/firmware/rtw89
$ sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtw89/rtw8852a_fw.bin
```

Download the latest Kernel [source code](https://cdn.kernel.org/pub/linux/kernel/v5.x/) with wget and extract it:
```
$ cd /usr/src/
$ sudo wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.16.5.tar.xz
$ sudo tar xvf linux-5.16.5.tar.xz
$ cd linux-5.16.5/
```

Copy your current Kernel's configuration file (Debian stock Kernel config):
```
$ sudo cp -v /boot/config-$(uname -r) .config
```

Create a new config based on your current config (you can hit enter till the end):
```
$ sudo make oldconfig
```

Load the tool to configure the linux source:
```
$ sudo make menuconfig
```
Make sure to add Realtek 802.11ax 8852AE drivers, and to save the new config, you can find the drivers here:
```
- device drivers
-- network device support
--- wireless lan
---- realtek devices -> realtek 802.11ax -> 8852AE
```

Fix potential signing errors by setting this in your `.config` (or set your own keys):
```
CONFIG_SYSTEM_TRUSTED_KEYS=""
```

Now compile the Kernel (this will take some time, you can speed it up by using the `make` in parallel mode with -j flag, for example `make -j4` to use 4 cores):
```
$ sudo make clean
$ sudo make bzImage
```

Install modules:
```
$ sudo make modules
$ sudo make modules_install
```

Strip unnecessary files to reduce the size of initrd and /lib/modules/5.16.5/ directory:
```
$ sudo find /lib/modules/5.16.5/ -name *.ko -exec strip --strip-unneeded {} +
```

Next, install the compiled Kernel:
```
$ sudo make install
```

The command will install the files below in the /boot/ directory:
```
vmlinuz-5.16.5
initramfs-5.16.5.img
System.map-5.16.5
```

Optional, configure GRUB to remeber the last Kernel selected. Open `/etc/default/grub` and put the following in:
```
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

For the changes made to take place, we need to update GRUB and reboot:
```
$ sudo update-grub
$ sudo reboot now
```

## Credits
Thanks to all who figured this out and posted useful bits of info around the web. Enjoy!

[![IMAGE ALT TEXT](http://img.youtube.com/vi/Q4MymPStabI/0.jpg)](http://www.youtube.com/watch?v=Q4MymPStabI "Blind (Frankie Knuckles Remix)")
