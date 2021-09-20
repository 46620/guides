# Single GPU passthrough
> Last updated: 2021-09-19 9:05pm EDT

So you wanna game but you also wanna use Linux? That's understandable. This guide should help you with that.

# Notices

!!! warning
    This guide is less of a full "here's how everything works" and more of a jumpstart into this. **PLEASE DO RESEARCH AND DO NOT RELY ON THIS ALONE** All pages I use will be at the bottom in the credits. 

!!! info "Current setup"
    The following are what this guide is based on, please make sure to change certain steps for your build. Read the credits for help.

    *    A machine running an arch based distro
    *    An EVGA RTX 2080 Super Black
    *    An AMD Ryzen 5 5600x

    At some points I will provide the intel commands, but please make sure to read the credits for all the guides that go more in-depth.

!!! warning "Before Installing Linux"
    Aight before you even get to Linux (If you're still on Windows) dump your GPUs vBIOS using [GPUz](https://www.techpowerup.com/gpuz/). You're gonna need it in a much later step. Just back it up and keep it somewhere you can get to later.

Now that we got that out of the way, lets get to installing, crying, and a lot of screaming.

# Dependencies
Single GPU passthrough will need some stuff installed in order to work. Running the following list of commands will install everything for you.

```bash
sudo pacman -S qemu libvirt edk2-ovmf virt-manager iptables-nft dnsmasq
sudo systemctl enable --now libvirtd.service
sudo virsh net-autostart default
sudo virsh net-start default
sudo usermod -aG kvm,input,libvirt <your_name>
```

# Enable & Verify IOMMU
Before you start setting up VMs, go to your BIOS and enable either Intel VT-d or AMD-Vi + IOMMU depending on your processor and motherboard. If you're using AMD and you do not see the IOMMU option, you should be fine, but do research.

After enabling that in your BIOS you're now gonna have to edit grub and add either one of the following depending on your processor.

| /etc/default/grub |
| ----- |
| `GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on iommu=pt ..."` |
| OR |
| `GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on iommu=pt ..."` |

After you do that run `grub-mkconfig -o /boot/grub/grub.cfg` to rebuild your config with iommu grouping enabled, and then reboot.

To verify the groups run `dmesg | grep 'IOMMU enabled'`. If it does not show up then you missed something or your system does not support it. (There are instances where IOMMU will not report as enabled but still work, verify with the second part before thinking something is funky).

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
If the groups are not valid, you will need to do [ACS patching](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Bypassing_the_IOMMU_groups_(ACS_override_patch)), which is out of the scope of this project, follow the guide linked.

While doing this, copy all the text for your GPUs IOMMU group and save it to a file somewhere to easily find.

# Setting up the VM
This guide is going to assume that you want to set up a Windows 10 VM, In the future I will make a guide for a MacOS VM, but now is not the time.

## Downloading Drivers
Yes I'm telling you to download the drivers before downloading the OS, why? Cause fuck you, you're gonna forget to download them later and skip to installing. Anyways click [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso) for the latest virtio drivers.

## Downloading Windows 10
Aight now you need a shit OS to install huh? [This](https://www.microsoft.com/en-us/software-download/windows10ISO) page will be where you download Windows 10.

## Setting up virt manager and all that stuff
This is recommended to do before all of this is to enable XML editing in virt managers settings. Open "Virtual Machine Manager" and click "Edit". Enable XML editing and hit close.

### Base setup
Aight this is the fun part, click `"New VM" > Local install media > Browse > Browse Local` and select the ISO you download for Windows 10. If it asks for search permission give it permission. Whatever it asks of you give it.

Next is memory and CPUs, I gave it half of my RAM and all but 2 cores of my CPU.

Then give it some storage, I gave mine 500GB just to survive more than an hour with the bloatware that is in win10.

After giving it a reasonable amount of storage, name your vm, or keep it default. Just make sure that you note down what it's called. Also check the box that says "Customize configuration before install".

### Customizing Install
Aight in the "Overview" section set the firmware to the UEFI setting that does not have `secboot` in the name.

Then go to "CPUs" and uncheck "Copy host CPU configuration" and set the model to "Host passthrough".

Then open the dropdown for Topology and check the "Manually set CPU topology" option/ Then set the information there so the vCPU allocation matches what you put in the base setup.

Go into "Boot Options" and check CDROM and move it to the top of the boot list (this should uhh.. really be on by default but okay).

Then go to the SATA Disk for the VM and set it to VirtIO.

Do the same for your NIC, just set it to VirtIO and it'll be fine.

Last add another CDROM for the virtio iso.

After that you can hit Begin Installation.

### The First boot/Install
Once you hit begin installation, wait til it asks you to press any button to boot from CD. Press something and go through the setup until you're asked how you want to do the install

Select Custom and select your... oh wait there's no disk. Hit "Load driver" and install the win10 driver. Then select your disk and do your standard windows 10 install.

Once you're at the desktop, open file explorer and install the drivers from the virtio CD. Once that is complete you can shut down the VM.

### Cleaning up the VM details
This is a small section to just make management later a bit easier.

In your VM details go to "Boot Options" and disable the CDROM options.

Next remove both CDROMS (you can delete the files if you want)

You can also remove `Sound ich*` if you want. I have not noticed anything buggy yet with it removed.

# QEMU hooks
You ready to learn the wonders of qemu hooks? Cause if not you don't have a choice.

## Installing
To get started you need to make a few folders for the hooks so run `sudo mkdir -p /etc/libvirt/hooks` to make said folders.

Next thing you want to do is download the hook manager itself by running the following commands:

``` bash
sudo wget 'https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu' \
     -O /etc/libvirt/hooks/qemu
sudo chmod +x /etc/libvirt/hooks/qemu
sudo systemctl restart libvirtd
```
And there ya go, you've now installed the manager for the hooks.. time to write them

## Making your own hooks
This section is annoying but honestly kinda my favorite. We now have to write the scripts that will tell our computer to give the virtual machine the graphics card. 

I hope you know what your VM is called cause now would be the time to know the name

To get started you want to make the folder structure below: 
```
/etc/libvirt/hooks
├── qemu
└── qemu.d
    └── <vm_name>
        ├── prepare
        │   └── begin
        ├── release
        │   └── end
        └── stopped
            └── end
```
> The stopped folder is not required, but on my setup I need it because my release scripts never run.

With your folders created now we need to make some scripts. You need to put the scripts in the proper begin/end folder for their jobs. Below are some examples with the path of them as the name. You can use these examples but you need to remember to change certain things for your builds, like your dm, vtcons, and drivers.

??? info "/etc/libvirt/hooks/qemu.d/<vm_name\>/prepare/begin/start.sh"
    ``` bash
    # File based on "SomeOrdinaryGamers" scripts.
    # Anything labeled with a * in the comment needs to be edited by you to work with your setup

    # debugging
    set -x

    # load vars
    source "/etc/libvirt/hooks/kvm.conf"

    # kill the DM*
    systemctl stop sddm.service

    # Unbind VTconsoles*
    echo 0 > /sys/class/vtconsole/vtcon0/bind
    echo 0 > /sys/class/vtconsole/vtcon1/bind

    # Unbind EFI-framebuffer
    echo efi-framebuffer.0 > /sys/bus/platform/drivers/efi-framebuffer/unbind

    # Avoid race condition*
    sleep 5

    # Unload Nvidia Drivers*
    modprobe -r nvidia_uvm
    modprobe -r i2c_nvidia_gpu
    modprobe -r nvidia_drm
    modprobe -r nvidia_modeset
    modprobe -r drm_kms_helper
    modprobe -r nvidia
    modprobe -r drm

    # Unbind GPU*
    virsh nodedev-detach $VIRSH_GPU_VIDEO
    virsh nodedev-detach $VIRSH_GPU_AUDIO
    virsh nodedev-detach $VIRSH_GPU_USB
    virsh nodedev-detach $VIRSH_GPU_SERIAL

    # Isolate the CPU
    systemctl set-property --runtime -- user.slice AllowedCPUs=0,6
    systemctl set-property --runtime -- system.slice AllowedCPUs=0,6
    systemctl set-property --runtime -- init.scope AllowedCPUs=0,6


    # load vfio
    modprobe vfio
    modprobe vfio_pci
    modprobe vfio_iommu_type1
    ```

??? info "/etc/libvirt/hooks/qemu.d/<vm_name\>/release/end/revert.sh"
    ``` bash
    # File based on "SomeOrdinaryGamers" scripts.
    # Anything labeled with a * in the comment needs to be edited by you to work with your setup
    
    # debugging
    set -x
    
    # Restart linux host entirely (debug line)
    # reboot
    
    # load vars
    source "/etc/libvirt/hooks/kvm.conf"
    
    # Unload vfio
    modprobe -r vfio
    modprobe -r vfio_pci
    modprobe -r vfio_iommu_type1
    
    # Unisolate the CPU
    systemctl set-property --runtime -- user.slice AllowedCPUs=0-11
    systemctl set-property --runtime -- system.slice AllowedCPUs=0-11
    systemctl set-property --runtime -- init.scope AllowedCPUs=0-11
    
    # Rebind GPU*
    virsh nodedev-reattach $VIRSH_GPU_VIDEO
    virsh nodedev-reattach $VIRSH_GPU_AUDIO
    virsh nodedev-reattach $VIRSH_GPU_USB
    virsh nodedev-reattach $VIRSH_GPU_SERIAL
    
    # Rebind VTconsoles*
    echo 1 > /sys/class/vtconsole/vtcon0/bind
    echo 1 > /sys/class/vtconsole/vtcon1/bind
    
    # Read Nvidia x config* (remove if on AMD)
    nvidia-xconfig --query-gpu-info > /dev/null 2>&1
    
    # Rebind EFI-framebuffer
    echo "efi-framebuffer.0" > /sys/bus/platform/drivers/efi-framebuffer/bind
    
    # Reload Nvidia Drivers*
    modprobe nvidia_uvm
    modprobe i2c_nvidia_gpu # Key was rejected by service
    modprobe nvidia_drm # Key was rejected by service
    modprobe nvidia_modeset
    modprobe drm_kms_helper # Key was rejected by service
    modprobe nvidia
    modprobe drm

    # Start the DM*
    systemctl start sddm.service
    ```
But wait... what's this elusive `kvm.conf` file I see? Well lemme tell you, it's a small dictonary of which PCI devices are what device, lemme show you:

??? info "/etc/libvirt/hooks/kvm.conf"
    ```
    VIRSH_GPU_VIDEO=pci_0000_05_00_0
    VIRSH_GPU_AUDIO=pci_0000_05_00_1
    VIRSH_GPU_USB=pci_0000_05_00_2
    VIRSH_GPU_SERIAL=pci_0000_05_00_3
    ```
That's right, the file's sole purpose is to be a dictionary for your PCI devices you got in [this step](1gpu_pass.md#enable-verify-iommu).

Make sure that all the scripts are runable by doing `sudo chmod +x /path/to/each/script.sh` to all of the scripts.

Once that's all done you can test them by running the start script, if you're screen goes black then boom it worked and you did it right.. now go hold the power button down to restart it so you can continue.

# Hijacking a GPU
This part sucks the most IMHO.

## Patching the GPU
>This step is not required for all GPUs. It's needed for mine so I will go over that. Look up if you need to do it.

First thing you're gonna wanna do is get the file you dumped in [this step](1gpu_pass.md#before-installing-linux) and put it somewhere you will remember. I put mine in a `vbios` folder in my `Documents` directory.

Next you want to install a hex editor, for this guide I'm using Bless so run `sudo pacman -S bless` and open it.

Next make sure that you have a backup of the file before opening it in the hex editor. After you make a backup open the original and get ready for patching

Open the file and hit `CTRL+F` and type "VIDEO" and and search as Text. Find the closest "U" in front (hex 55) and delete **EVERYTHING** infront of it. (The file size difference for my GPU was ~130kb). Once you do that save the file and close bless.

## Attaching the GPU to the VM
This is my least favorite part as it just takes time and is annoying as shit to do.

To start this step go ahead and attach all of your GPU stuff to your VM by clicking `Add Hardware > PCI Host Device` and adding everything for the GPU.. one at a time. After you do that click on one of the devices and go to `XML` under `</source>` add a line similar to this for your patched vBIOS `<rom file="/home/mia/Documents/vbios/2080s.rom"/>`. It should look similar to the photo below

![01_gpu_vbios](img/1gpu_pass/01_gpu_vbios.png)

After you do it to one of them.. Do it to the rest of them. That's right folks you need to do it to all of them.

After you do this go to the option that says `Video QXL`, click it and replace "QXL" with "none".

After you do that start the VM and watch as your screens go black and you boot to Windows 10. 

# Making the VM stealthy
> This entire section is based on the videos [here](https://www.youtube.com/watch?v=rrlpg6F82S4) and [here](https://www.youtube.com/watch?v=VKh2eKPnmXs) I highly recommend watching those instead of following this part so you can make sure you do everything right for your system

You wanna play BattlEye games? Well follow the steps below until it gets officially supported on [Proton](https://fossbytes.com/steam-deck-valve-working-on-anti-cheat-support-for-proton/)

This is for if you already have a working windows VM. This is a very easy thing to set up. I have my entire xml file in this repo if you wanna base it off of that, but below is all I needed to add to get mine working.

## CPU

First things first, make sure that you set your CPU to be to pass the model to the VM so it doesn't show up as the wrong CPU type.

![02_cpu](img/1gpu_pass/02_cpu.png)

## XML

After you set that up click on "Overview" and go to XML. Make sure you have XML editing on because we need to do some manual edits in here.

First we need to edit HyperV, edit your HyperV section to look similar to this, if you need extra for your system to work, add it also

```xml
    <hyperv>
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <runtime state="on"/>
      <synic state="on"/>
      <stimer state="on"/>
      <reset state="on"/>
      <frequencies state="on"/>
    </hyperv>
``` 

Right below that add this section for KVM

```xml
    <kvm>
      <hidden state="on"/>
    </kvm>
```

And below `<vmport state="off"/>` add `<ioapic driver="kvm"/>`

In CPU we need to disable hypervisor and set cache mode to passthrough, so add these lines to the bottom of your CPU section

```xml
    <cache mode="passthrough"/>
    <feature policy="disable" name="hypervisor"/>
```

And finally we need to set the clock area.

```xml
  <clock offset="localtime">
    <timer name="pit" tickpolicy="delay"/>
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="hpet" present="no"/>
    <timer name="tsc" mode="native"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
```

Once all of that is set you should be good to start your VM. Check Task Manager to see if it's detecting it's in a VM and if it's not, then congrats, you now have a stealthy VM, but we're not done yet, just one more thing to do

Open Windows Search and type `Turn Windows features On/Off` and install everything in the first Hyper-V section. Reboot the VM and you're all done.

# Ending comments
This guide might not be the best or work for everyone, please see the credits below. I hope that this at least makes some sense to someone and they're able to use this to help them set up a "perfect" gaming VM.

!!! tldr "Credits"
    [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

    [QaidVoid/Complete-Single-GPU-Passthrough](https://github.com/QaidVoid/Complete-Single-GPU-Passthrough)

    [virtio drivers](https://github.com/virtio-win/virtio-win-pkg-scripts)

    [Passthrough Post](https://passthroughpo.st/simple-per-vm-libvirt-hooks-with-the-vfio-tools-hook-helper/)
