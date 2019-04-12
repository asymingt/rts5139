# Overview #

This is a temporary fix for RTS5129/RTS5139 USB MMC card reader on Linux 3.16+ kernels. On my Lenovo Yoga 13 running Ubuntu 15.10 with a ```4.2.0-25-generic``` kernel I get the following errors.

```
Jan 27 22:27:53 ideapad-yoga-13 kernel: [   48.108682] mmc0: tuning execution failed
Jan 27 22:27:53 ideapad-yoga-13 kernel: [   48.108699] mmc0: error -22 whilst initialising SD card
```

This ocurred during a transition from ```3.15``` to ```3.16``` kernel, as a result of the ```staging/rts5139``` driver (which worked with the RTS5129/RTS5139) being replaced by the newer ```rtsx``` driver (which does not work with the RTS5129/RTS5139). This project reverts back to the old drivers as a temporary measure to get things up and running again.

# Compilation instructions #

Firstly, make sure you have kernel headers and the ```build-essential``` package installed.

Checkout, build and install the replacement driver.

```
cd /tmp
git clone https://github.com/asymingt/rts5139.git
cd rts5139
make
sudo make install
```

Blacklist the problematic modules by adding the following lines to ```/etc/modprobe.d/blacklist.conf```.

```
blacklist rtsx_usb_sdmmc
blacklist rtsx_usb_ms
blacklist rtsx_usb
```

Then, make sure you generate modules.dep and map files.

```
sudo depmod -a
```

Finally, disable module autoloading (and, optionally, also in the initial RAM filesystem)

Blacklist rtsx mmc modules via Grub.  For example, on Fedora (with grub.cfg in EFI):

```
# modify GRUB_CMDLINE_LINUX in /etc/default/grub and module_blacklist the rtsx_usb_sdmmc, rtsx_usb_ms, and rtsx_usb modules
grep GRUB_CMDLINE_LINUX /etc/default/grub
GRUB_CMDLINE_LINUX="resume=/dev/mapper/os-swap rd.lvm.lv=os/root rd.lvm.lv=os/swap rhgb quiet module_blacklist=rtsx_usb_sdmmc,rtsx_usb_ms,rtsx_usb"
grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

[OPTIONAL] If wanted/needed, update the initramfs:

```
sudo update-initramfs -u
```

Reboot, and check to see if the card reader works.



