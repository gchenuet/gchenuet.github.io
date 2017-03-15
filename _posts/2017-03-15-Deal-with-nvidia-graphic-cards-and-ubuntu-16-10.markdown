---
layout: post
title:  "System - Deal with NVIDIA Graphic Card & Ubuntu 16.10"
date:   2017-03-15 14:00:51
categories: ubuntu 16.10 nvidia graphic-card bumblebee tutorial
tags: ubuntu 16.10 nvidia graphic-card bumblebee tutorial
---

# Preface

At **[leboncoin](https://leboncoin.fr)**, we're using **Lenovo Thinkpad T460p** laptop with a serious hardware config (Intel Core i7 vPro, 16Go RAM, Dual GFX Intel/Nvidia & SSD) which is comfy for our daily tasks.
	
But last week, some artefacts appeared for black pixels on one of my display screens *(finally due to a bad cable...)* and I also noticed that I was using only the Intel card and never the other one (Nvidia).
	
So I decided to setup the Nvidia graphic card and install drivers and tools as Bumblebee for example.
Obviously, the Internets has a lot of informations about setup, etc, but I faced some troubles with **Ubuntu 16.10**.
	
This setup guide are based on **Nvidia GM108M [GeForce 940MX]** and **Ubuntu 16.10**.
	
# Graphic Card setup

#### Add PPA repository

```bash
$ sudo add-apt-repository ppa:graphics-drivers/ppa
   $ sudo apt update
```
	
#### Install the latest version

*Currently, the version 375 is the most stable, but feel free to use a newer one*
	
```bash
$ sudo apt-get install nvidia-375 nvidia-opencl-icd-375 nvidia-common 
     nvidia-prime nvidia-settings
```
	
#### Activate the driver

From *System Settings* or directly from the menu / Dash, open *Software & Updates*, click on the *Additional Drivers* tab, select the driver you want to use, and click "Apply changes"

<div style="text-align:center">
    <img src="https://raw.githubusercontent.com/gchenuet/gchenuet.github.io/master/images/2017-03-15-Deal-with-nvidia-graphic-cards-and-ubuntu-16-10/software-updates.png" width="500">
</div>

#### (Opt) Disable Power Management

*This step is mandatory if:*
	
`Kernel >=4.8, with a system for which the nvidia card is not the primary output and a BIOS released >=2015.`
	
Currently, there's a bug with Nvidia graphic cards and Kernel >= 4.8 and the workaround consists to **disable the Power Management for PCI Express cards** in **GRUB**

#### Add the kernel option
	
Add `pcie_port_pm=off` kernel option on `GRUB_CMDLINE_LINUX_DEFAULT` line in `/etc/default/grub`
	
``` bash
[...]

   GRUB_DEFAULT=0
   GRUB_HIDDEN_TIMEOUT=0
   GRUB_HIDDEN_TIMEOUT_QUIET=true
   GRUB_TIMEOUT=10
   GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_port_pm=off"
   GRUB_CMDLINE_LINUX=""
   
   [...]
```

#### Update GRUB

```bash
sudo update-grub
```
	
If interested, issues are opened for [Nvidia](https://devtalk.nvidia.com/default/topic/971733/linux/-370-28-with-kernel-4-8-on-gt-2015-machines-driver-claims-card-not-supported-if-nvidia-is-not-primary-card/) and [Bumblebee](https://github.com/Bumblebee-Project/Bumblebee/issues/810)
	
**Once done, restart your system !**

#### Verify !

To ensure your Nvidia card is correctly installed, you can try these commands:

- **dmesg**

```bash
dmesg | egrep -ni "bbswitch|nvidia"
```
- **nvidia-setup**

```bash
nvidia-setup
```
And verify informations about your graphic card.

- **glxinfo**

```bash
glxinfo|egrep "OpenGL vendor|OpenGL renderer"
```

# Power Saving

#### Set Intel card as default

To avoid black screen login, set the intel card as default:
``` bash
sudo prime-select intel
```

#### Install Bumblebee
Bumblebee is available in Ubuntu default repository
	
```bash
sudo apt install bumblebee
```
	
####  Blacklist your driver
	
Open **/etc/modprobe.d/bumblebee.conf** with a text editor (as root) and add your driver a the end of file:
```bash
# 370
blacklist nvidia-370
blacklist nvidia-370-updates
blacklist nvidia-experimental-370
```
	
> Note: If you install some newer Nvidia graphics drivers (nvidia-378, etc.), you'll need to add them too in the same way.

#### Configure Bumblebee

Open **/etc/bumblebee/bumblebee.conf** and modify some parts:
	
- In `bumblebeed` marker
```bash
[bumblebeed]
     Driver=nvidia
```
	
- In `driver-nvidia` marker
```bash
[driver-nvidia]
     KernelDriver=nvidia-375
     LibraryPath=/usr/lib/nvidia-375:/usr/lib32/nvidia-375
     XorgModulePath=/usr/lib/nvidia-375/xorg,/usr/lib/xorg/modules
```
	
> Note: Same as above, if you install a newer drivers, don't forget to update paths with the correct version number`
	
#### Upgrade XORG

- Find the bus ID for the Nvidia graphic card
```bash
lspci | egrep 'VGA|3D'
```

- Uncomment and modify the `busID` parameter in `/etc/bumblebee/xorg.conf.nvidia`
```bash
    BusID "PCI:02:00.0"
```

#### Reboot

That's all !
	
After rebooting, you can verify the state of you graphic card with:
```bash
$ cat /proc/acpi/bbswitch
```


