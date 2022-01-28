# How to compile and install the latest linux kernel

The default kernel on Debian 11 bullseye is 5.10. In this guide, we want to upgrade to linux kernel 5.16. This newer kernel includes rtw89 wifi [drivers](https://git.kernel.org/pub/scm/linux/kernel/git/kvalo/wireless-drivers-next.git/commit/?id=e3ec7017f6a20d12ddd9fe23d345ebb7b8c104dd) for Realtek 8852AE 802.11ax wireless chip found in some ThinkPad laptops (ThinkPad L14 gen1). This kernel upgrade will also fix bluetooth problems with the same wireless chip.

Note: Use this at your own risk!

Verify the kernel version on your system with uname:
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

Download the latest kernel source with wget and extract it:
```
$ cd /usr/src/
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.16.3.tar.xz
$ tar xvf linux-5.16.3.tar.xz
$ cd linux-5.16.3/
```

Copy your current kernel's configuration file
```
$ sudo cp -v /boot/config-$(uname -r) .config
```

Create a config based on current config and loaded modules:
```
$ sudo make localmodconfig
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

Now compile the kernel:
```
$ sudo make bzImage
```

Install modules:
```
$ sudo make modules
$ sudo make modules_install
```

Next, install the compiled kernel:
```
$ sudo make install
```

The command will install the files below in the /boot directory.
```
vmlinuz-5.16.3
initramfs-5.16.3.img
System.map-5.16.3
```

Optional, configure GRUB to remeber the last kernel choice. Open `/etc/default/grub` and put the following:
```
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

For the changes made to take place, we need to update GRUB and reboot:
```
$ sudo update-grub
$ sudo reboot now
```
