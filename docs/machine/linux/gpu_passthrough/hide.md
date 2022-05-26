# Hiding the Virtual Machines (Unsupported)
> Last updated: 2022-05-26

!!! warning "NOTICE"
    Everything on this page is not officially supported or endorsed. If you get banned from a game trying to hide the fact you're on a VM, that is fully on you.

## QEMU configs
This is for if you already have a working windows VM. This is a very easy thing to set up. I have my entire xml file in [this](https://github.com/46620/kvm-passthrough) repo if you wanna base it off of that, but below is all I needed to add to get mine working.

First things first, make sure that you set your CPU to be to pass the model to the VM so it doesn't show up as the wrong CPU type.

![01_cpu](../img/hide/01_cpu.png)

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

Once all of that is set you should be good to start your VM. Open Windows Search and type `Turn Windows features On/Off` and install everything in the first Hyper-V section. Reboot the VM and you're all done. You can use [this](../apps/pafish.exe) tool to check how stealthy the Virtual Machine is.

## Patching the Kernel and qemu (Arch only)
**USE THESE AT YOUR OWN RISK!! IF YOU GET BANNED FROM GAMES THAT DETECT THIS STUFF IT IS NOT MY FAULT**

The guide will be split into multiple sub guides in this one drop down to keep it all together.

### Kernel Patching
> The following guide is for patching your kernel and passing rdtscp checks on PAFish.
        
> This guide will be going over Arch's kernel build method as seen on the [Arch Wiki](https://wiki.archlinux.org/title/Kernel/Arch_Build_System) and the patch file is for the zen kernel.

#### Recompiling your kernel
First thing is grabbing the source code. Luckily on arch there are very easy ways to do this. Running `asp update linux-zen` will grab the latest build files and doing `asp export linux-zen`. You can replace linux with any package to grab the latest build files of said package, just in case you wanna compile everything yourself.
        
After grabbing the source next you want to make a little build directory for everything, I have mine at `~/Projects/kernel/linux-zen`
    
Then download [this patch](https://raw.githubusercontent.com/46620/patches/master/kvm.patch) and put it in the folder with the `PKGBUILD` file. (Note that the patch was made for version 5.15.13-zen1, but should work on the latest version)

!!! info
    The patch is only updated when it actually breaks. Please check when it was last modified in my [patch repo](https://github.com/46620/patches) to see how old it might be.
    
Next you're going to edit the `PKGBUILD` file and add the patch into it (example below) and [disable building docs](https://wiki.archlinux.org/title/Kernel/Arch_Build_System#Avoid_creating_the_doc), after you add it run `updpkgsums` and then `makepkg -si`.
    
```
    source=(
        "$_srcname::git+https://github.com/zen-kernel/zen-kernel?signed#tag=$_srctag"
        config         # the main kernel config file
        kvm.patch # add this line
    )
```

After you reboot, edit your VM's XML and add `<feature policy="disable" name="rdtscp"/>` to the CPU block.

#### Using my repo
This is a lot less secure as I don't sign my packages, but this is for the few that don't wanna manually recompile their kernel

edit your `/etc/pacman.conf` and add the following above all other repos (so it gets priority when downloading the package)

```
[46620-repo]
SigLevel = Optional
Server = https://gitlab.com/46620/$repo/-/raw/master/$arch
```

After that, reinstall `linux-zen` and `linux-zen-headers` and you should have the patch applied, then just add the XML line and everything should work.

### QEMU Patching
> The following guide will try to mask some generic QEMU names so the Virtual Machine won't detect them.

For this we actually wanna use the [qemu-git](https://aur.archlinux.org/packages/qemu-git/) AUR package as it is easier to work with.

After cloning it locally, add [this patch](https://raw.githubusercontent.com/46620/patches/master/qemu.patch) and [this patch](https://raw.githubusercontent.com/46620/patches/master/edk2.patch) to the root folder and edit the PKGBUILD to make source and prepare similar to the following.

!!! info
    The patches are only updated when they actually break. Please check when they were last modified in my [patch repo](https://github.com/46620/patches) to see how old they might be.

```
...
source=(git://git.qemu.org/qemu.git
    qemu-guest-agent.service
    65-kvm.rules
    qemu.patch # added
    edk2.patch # added
)
...
prepare() {
    cd "${srcdir}/${_gitname}"
    mkdir build-{full,headless}
    git submodule init; git submodule update # added
    cd .. # added
    patch -p1 < qemu.patch # added
    patch -p1 --binary < edk2.patch # added
      cd "${srcdir}/${_gitname}" # added
    mkdir -p extra-arch-{full,headless}/usr/{bin,share/qemu}
}
```

After doing that run `updpkgsums` and then `makepkg -s`. Install the packages you need from what it builds, you can't use `makepkg -si` as qemu-headless and qemu conflict with each other.

!!! warning
    After compiling a new version of qemu, restart libvirtd and remake your Virtual Machine, you can leave the old one there, just set the storage to the old drive and copy any XML edits you made, this is just to make sure that if the patch breaks anything, you still have your original to restore to.

!!! info "Compile faster"
    Arch's default compile flags are kinda dog water and slow, you can edit `/etc/makepkg.conf` and uncomment the `MAKEFLAGS` line and change it to match your needs.
        
    Also for compiling the kernel, you should set up [modprobed-db](https://wiki.archlinux.org/title/Modprobed-db) to cut the compile time (and size) by at least half.