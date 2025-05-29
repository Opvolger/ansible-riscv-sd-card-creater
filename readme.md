# Create bootable partitions for RISCV

Start with a SD-card with no partitions and GPT-partition table!

I started this playbook to create SD-cards for the StarFive VisionFive 2 with an external GPU.
So I can run different distributions of Linux on my RISC-V device.

I started with making a Milk-V Jupiter version. But the drivers for the PCIe are to bad. I have a lot of crashes and instable kernels. But the StarFive VisionFive 2 are working fine. Maybe I will add more RISC-V devices in the future.

## StarFive VisionFive 2

Documentation used for this project:

U-boot and opensbi build can be find [here](https://github.com/u-boot/u-boot/blob/master/doc/board/starfive/visionfive2.rst)

## Setup variables

Maybe you have to change some variables in `host_vars/localhost`. For example the firmware files I add in `starfive_visionfive2_kernel.yaml` with the variable `CONFIG_EXTRA_FIRMWARE`.

Maybe your SD-Card device is not on `/dev/sdb`, so change that variable in `host_vars/localhost/target.yaml`

All the downloaded files are default in /tmp, not for:

- downloading img file of ubuntu see (change it) `host_vars/localhost/ubuntu_24_10.yaml`
- checkout kernel Milk-V see (change it) `host_vars/localhost/milkv_jupiter_kernel.yaml`

## Reset u-boot environment variables

Maybe there are alreasy saved environment variables in u-boot in your onboard flash. I had to reset the environment variables in u-boot!:

```bash
env default -a
saveenv
```

## Playbooks

The command to run them:

```bash
ansible-playbook [board]-[action].yaml --ask-become-pass -v 
```

Description for Starfive Visionfive2 playbooks:

Compile

- starfive-visionfive2-compile.yaml: Compiles U-boot, OpenSBI and the kernel

Boot partitions

- starfive-visionfive2-create-boot-partitions.yaml: creates flash/boot partitions + copy kernel
- starfive-visionfive2-update-boot.yaml: compiles U-boot, OpenSBI and flash the partitions and compiles and copy the kernel on the boot partition
- starfive-visionfive2-update-kernel.yaml: compiles and copy the kernel on the boot partition

Create rootfs

- starfive-visionfive2-opensuse.yaml: creates rootfs partition + create boot.src
- starfive-visionfive2-ubuntu-20-10.yaml: creates rootfs partition + create boot.src

All combined

- starfive-visionfive2-opensuse-all.yaml: Compiles U-boot, OpenSBI and the kernel + creates boot partitions + copy kernel + creates rootfs partition + create boot.src
- starfive-visionfive2-ubuntu-20-10-all.yaml: Compiles U-boot, OpenSBI and the kernel + creates boot partitions + copy kernel + creates rootfs partition + create boot.src

## Example only boot from SD, use for example EMMC rootfs

I have an EMMC on my Starfive Visionfive2 with multiple rootfs. If I want to boot from them I can do that with a fresh u-boot,opensbi andkernel with the next commands

```bash
ansible-playbook starfive-visionfive2-compile.yaml
# we need to be root the create partitions on SD-card
ansible-playbook starfive-visionfive2-create-boot-partitions.yaml --ask-become-pass -v
# we need to be root the create partitions on SD-card and mount stuff
ansible-playbook starfive-visionfive2-boot-emmc.yaml --ask-become-pass -v
```

## Switch from distribution

Delete only last partition of the SD-card and run one of the other playbook:

- starfive-visionfive2-opensuse.yaml
- starfive-visionfive2-ubuntu-20-10.yaml

## After First Boot

After creating the SD-Card, boot the device and do the next steps

### Fedora 42

Login with user `fedora` and password `linux`.

edit `/etc/default/grub` add amd radeon switch arguments on boot `GRUB_CMDLINE_LINUX_DEFAULT`

```bash
sudo nano /etc/default/grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

```ini
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="radeon.si_support=0 amdgpu.si_support=1 radeon.cik_support=0 amdgpu.cik_support=1 earlycon rootflags=subvol=root"
GRUB_GFXMODE=auto
GRUB_TERMINAL_INPUT="serial"
GRUB_TERMINAL_OUTPUT="serial"
GRUB_TIMEOUT=3
GRUB_TIMEOUT_STYLE=menu
```

```bash
sudo dnf install @kde-desktop-environment
# sddm is SLOW! on riscv with Fedora 42 (BUG?)
sudo dnf install gdm
sudo systemctl set-default graphical.target
sudo systemctl disable sddm
sudo systemctl enable gdm
sudo balooctl6 disable
sudo reboot
```

### Fedora 41

Login with user `fedora` and password `fedora_rocks!`. Yes NOT with `riscv`, that user will NOT work!.

```bash
sudo -i
# check for desktops
dnf4 group list -v --available | grep desktop
   KDE Plasma Workspaces (kde-desktop-environment)
   Xfce Desktop (xfce-desktop-environment)
   Phosh Desktop (phosh-desktop-environment)
   LXDE Desktop (lxde-desktop-environment)
   LXQt Desktop (lxqt-desktop-environment)
   Cinnamon Desktop (cinnamon-desktop-environment)
   MATE Desktop (mate-desktop-environment)
   Sugar Desktop Environment (sugar-desktop-environment)
   Deepin Desktop (deepin-desktop-environment)
   Budgie Desktop (budgie-desktop-environment)
   Basic Desktop (basic-desktop-environment)
   i3 desktop (i3-desktop-environment)
   Sway Desktop (sway-desktop-environment)
   Budgie (budgie-desktop)
   Budgie Desktop Applications (budgie-desktop-apps)
   Desktop accessibility (desktop-accessibility)
# KDE/LXQt is not working when i was making this document (problem with libQt6Core.so.6 / libQt6Gui.so.6)
# XORG libs where not in repo when i was making this document
# So I installed GNOME
dnf install @workstation-product-environment --skip-broken
reboot
```

We will see the Desktop!

### Ubuntu

First login: ubuntu/ubuntu

Change password

Now you can install what you want, for example install KDE-desktop with Firefox browser.

```bash
sudo add-apt-repository ppa:mozillateam/ppa
sudo apt update
sudo apt install kde-standard firefox
sudo reboot
```

Login and we will see the Desktop!

### OpenSUSE

Hit `Ctrl + Alt + F3` and login as `root` with password `linux`.

```bash
Welcome to openSUSE Tumbleweed 20241117 - Kernel 6.6.36+ (ttyS0).

end0: 192.168.2.28 2a02:a447:277b:1:e172:a638:be8a:114c
end1:  


fedora login: root
Password: 
Have a lot of fun...
2a02-a447-277b-1-e172-a638-be8a-114c:~ #
```

now create a user to login KDE! I create a user opvolger, and added the wheel-group. (and added opvolger to the wheel-group)

```bash
$ useradd opvolger
# set password
$ passwd opvolger
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: password updated successfully
# install wheel group
$ zypper install system-group-wheel
Retrieving repository 'Open H.264 Codec (openSUSE Tumbleweed)' metadata ..[done]
Building repository 'Open H.264 Codec (openSUSE Tumbleweed)' cache .......[done]
Retrieving repository 'openSUSE-Tumbleweed-Oss' metadata .................[done]
Building repository 'openSUSE-Tumbleweed-Oss' cache ......................[done]
Retrieving repository 'openSUSE-Tumbleweed-Update' metadata ..............[done]
Building repository 'openSUSE-Tumbleweed-Update' cache ...................[done]
Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following NEW package is going to be installed:
  system-group-wheel

1 new package to install.

Package download size:     8.6 KiB

Package install size change:
            |        38 B  required by packages that will be installed
      38 B  |  -      0 B  released by packages that will be removed

Backend:  classic_rpmtrans
Continue? [y/n/v/...? shows all options] (y): 
Retrieving: system-group-wheel-20170617-26.1.noarch (openSUSE-Tumbleweed-Oss)
                                                            (1/1),   8.6 KiB
Retrieving: system-group-wheel-20170617-26.1.noarch.rpm ..................[done]

Checking for file conflicts: .............................................[done]
/usr/bin/systemd-sysusers --replace=/usr/lib/sysusers.d/system-group-wheel.conf -
Creating group 'wheel' with GID 469.
(1/1) Installing: system-group-wheel-20170617-26.1.noarch ................[done]
Running post-transaction scripts .........................................[done]
# add user as sudo-er
$ usermod -a -G wheel opvolger
```

Now hit `Ctrl + Alt + F2`. Login with user opvolger and your password.

We will see the Desktop!

## TODO

Create a ramdisk with all the firmware in the correct kernel version.
For now OpenSUSE will boot without a ramdisk.
