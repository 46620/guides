# GPU Passthrough setup
> Last update: 2022-05-26

So you wanna game but you also wanna use Linux? That's understandable. This guide should help you with that.

!!! warning "Before Installing Linux"
    If you are still on Windows while reading this, dump your GPUs vBIOS using [GPUz](https://www.techpowerup.com/gpuz/). You're gonna need it in a much later step. Just back it up and keep it somewhere you can get to later.

## Dependencies
GPU passthrough will need some stuff installed in order to work. Running the following list of commands will install everything for you.

> I've only fully tested on Arch and just swapped what packages needed to be installed for each distro, if there's an issue with one, please let me know.

??? info "Arch based distros"
    ```bash
    sudo pacman -S qemu libvirt edk2-ovmf virt-manager iptables-nft dnsmasq linux-headers
    sudo systemctl enable --now libvirtd.service
    sudo virsh net-autostart default
    sudo virsh net-start default
    sudo usermod -aG kvm,input,libvirt <your_name>
    ```

??? info "Ubuntu"
    ```bash
    sudo apt install qemu-kvm libvirt-daemon-system dnsmasq linux-headers-`uname -r` dkms
    sudo systemctl enable --now libvirtd.service
    sudo virsh net-autostart default
    sudo virsh net-start default
    sudo usermod -aG kvm,input,libvirt <your_name>
    ```

??? info "Debian"
    ```bash
    sudo apt install qemu-kvm libvirt-daemon-system iptables dnsmasq linux-headers-`uname -r` dkms
    sudo systemctl enable --now libvirtd.service
    sudo virsh net-autostart default
    sudo virsh net-start default
    sudo usermod -aG kvm,input,libvirt <your_name>
    ```

??? info "Fedora"
    ```bash
    sudo dnf install qemu libvirt edk2-ovmf virt-manager
    sudo systemctl enable --now libvirtd.service
    sudo virsh net-autostart default
    sudo virsh net-start default
    sudo usermod -aG kvm,input,libvirt <your_name>
    ```

## Enable & Verify IOMMU
Before you start setting up VMs, go to your BIOS and enable either Intel VT-d or AMD-Vi + IOMMU depending on your processor and motherboard.

After enabling that in your BIOS you're now gonna have to edit grub and add either one of the following depending on your processor:

| /etc/default/grub |
| ----- |
| `GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on iommu=pt video=efifb:off ..."` |
| OR |
| `GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on iommu=pt video=efifb:off..."` |

After you do that run `grub-mkconfig -o /boot/grub/grub.cfg` to update your config with iommu grouping enabled, and then reboot.

To verify the groups run `dmesg | grep 'IOMMU enabled'`. If it does not show up then you missed something or your system does not support it. (If that reports nothing, try running `journalctl -b 0 | grep -i iommu` instead).

After making sure that IOMMU grouping is enabled, run the following script to make sure that the groups are valid, make sure that the GPU is in it's own group with all of it's stuff:

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
If there are devices in the group that you don't wanna pass, you will need to do [ACS patching](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Bypassing_the_IOMMU_groups_(ACS_override_patch)), which is out of the scope of this project, follow the guide linked.

While doing this, copy all the text for your GPUs IOMMU group and save it to a file somewhere to easily find. You will need it for a later step.

## Setting up the VM
### Downloads
You're gonna need 2 things, A copy of [Windows](https://www.microsoft.com/en-us/software-download/windows10ISO) and the [virtio](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.215-2/virtio-win-0.1.215.iso) drivers. Clicking them will either download or bring you to a download page.

### Setting up virt manager and all that stuff
This is recommended to do before all of this is to enable XML editing in virt managers settings. Open "Virtual Machine Manager" and click "Edit". Enable XML editing and hit close.

#### Setting up the Virtual Machine
Aight this is the fun part, click `"New VM" > Local install media > Browse > Browse Local` and select the ISO you download for Windows 10. If it asks for search permission give it permission. Whatever it asks of you give it.

Next is memory and CPUs, I gave it half of my RAM and all but 2 cores of my CPU.

Then give it some storage, I gave mine 500GB just to survive more than an hour with the bloatware that is in win10.

After giving it a reasonable amount of storage, name your vm, or keep it default. Just make sure that you note down what it's called. Also check the box that says "Customize configuration before install".

In the "Overview" section set the firmware to `/usr/share/edk2-ovmf/x64/OVMF_CODE.fd`.

Then go to "CPUs" and uncheck "Copy host CPU configuration" and set the model to "Host passthrough".

Then open the dropdown for Topology and check the "Manually set CPU topology" option. Then set the information there so the vCPU allocation matches what you put in the base setup.

Go into "Boot Options" and check CDROM and move it to the top of the boot list (this should uhh.. really be on by default but okay).

Then go to the SATA Disk for the VM and set it to VirtIO.

Do the same for your NIC, just set it to VirtIO and it'll be fine.

Last add another CDROM for the virtio iso.

After that you can hit Begin Installation.

#### The First boot/Install
Once you hit begin installation, and when it asks you to press any button to boot from CD, press something. Go through the setup until you're asked how you want to do the install.

Select Custom and select your... oh wait there's no disk. Hit "Load driver" and install the win10 driver. Then select your disk and do your standard windows 10 install.

Once you're at the desktop, open file explorer and install the drivers from the virtio CD. Once that is complete you can shut down the VM.

#### Cleaning up the VM details
This is a small section to just make management later a bit easier.

In your VM details go to "Boot Options" and disable the CDROM options.

Next remove both CDROMS (you can delete the files if you want).

## What kind of VM
If you only have one GPU, it's recommended to do the [Single GPU passthrough](../1gpu_pass) guide

If you have 2 GPUs in the same build, the [Multi GPU](../2gpu_pass) guide is a good option