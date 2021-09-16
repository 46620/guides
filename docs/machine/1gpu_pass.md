# Single GPU passthrough
> Last updated: 2021-09-16

So you wanna game but you also wanna use Linux? That's understandable. This guide should help you with that.

!!! warning
    This guide is less of a full "here's how everything works" and more of a jumpstart into this. **PLEASE DO RESEARCH AND DO NOT RELY ON THIS ALONE** All pages I use will be at the bottom in a credits dropdown. 

## Requirements
The following are what this guide is based on, please make sure to change certain steps for your build. Read the credits for help.

*    A machine running an arch based distro
*    An EVGA RTX 2080 Super Black
*    An AMD Ryzen 5 5600x

At some points I will provide the intel commands, but please make sure to read the credits for all the guides that go more in-depth.

Now that we got that out of the way, lets get to installing, crying, and a lot of screaming.

## Installing deps
Single GPU passthrough will need some stuff installed in order to work. Running the following list of commands will install everything for you.

```bash
sudo pacman -S qemu libvirt edk2-ovmf virt-manager iptables-nft dnsmasq
sudo systemctl enable --now libvirtd.service
sudo virsh net-autostart default
sudo virsh net-start default
sudo usermod -aG kvm,input,libvirt $user
```

## Enable & Verify IOMMU

Before you start setting up VMs, go to your BIOS and enable either Intel VT-d or AMD-Vi + IOMMU depending on your processor and motherboard. If you're using AMD and you do not see the IOMMU option, you should be fine, but do research.

After enabling that in your BIOS you're now gonna have to edit grub and add either one of the following depending on your processor.

| /etc/default/grub |
| ----- |
| `GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on iommu=pt ..."` |
| OR |
| `GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on iommu=pt ..."` |

After you do that run `grub-mkconfig -o /boot/grub/grub.cfg` to rebuild your config with iommu grouping enabled, and then reboot.

To verify the groups run `dmesg | grep 'IOMMU enabled'`. If it does not show up then you missed something or your system does not support it. (There are instances where IOMMU will not report as enabled but still work, verify with the second part before thinking something is funky)

After making sure that IOMMU grouping is enabled, run the following script to make sure that the groups are valid, make sure that the GPU is in it's own group with all of it's stuff.

``` bash
#!/bin/bash
shopt -s nullglob
for g in `find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V`; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```
If the groups are not valid, you will need to do [ACS patching](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Bypassing_the_IOMMU_groups_(ACS_override_patch)) Which is out of the scope of this project, follow the guide linked.

## Setting up the VM
This guide is going to assume that you want to set up a Windows 10 VM, In the future I will make a guide for a MacOS VM, but now is not the time.

### Downloading Drivers
Yes I'm telling you to download the drivers before downloading the OS, why? Cause fuck you, you're gonna forget to download them later and skip to installing. Anyways click [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso) for the latest virtio drivers.

### Downloading windows 10
Aight now you need a shit OS to install huh? [This](https://www.microsoft.com/en-us/software-download/windows10ISO) page will be where you download Windows 10.

## Setting up virt manager and all that stuff

Under construction...

??? tldr "Credits"
    [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

    [QaidVoid/Complete-Single-GPU-Passthrough](https://github.com/QaidVoid/Complete-Single-GPU-Passthrough)

    [virtio drivers](https://github.com/virtio-win/virtio-win-pkg-scripts)