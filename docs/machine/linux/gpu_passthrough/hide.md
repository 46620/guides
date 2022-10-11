# Hiding the Virtual Machines (Unsupported)
> Last updated: 2022-09-30

!!! error "NOTICE"
    BattlEye and EAC will ban if you're caught with these. Don't bitch and complain if you're banned, just comlain that they made their linux compatibility opt in, not opt out.

## QEMU configs
This is for if you already have a working windows VM. This is a very easy thing to set up. I have my entire xml file in [this](https://github.com/46620/kvm-passthrough) repo if you wanna base it off of that, but below is all I needed to add to get mine working.

First things first, make sure that you set your CPU to be to pass the model to the VM so it doesn't show up as the wrong CPU type.

![01_cpu](img/hide/01_cpu.png)

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
      <vendor_id state="on" value="buttplug"/>
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

Once all of that is set you should be good to start your VM. Open Windows Search and type `Turn Windows features On/Off` and install everything in the first Hyper-V section. Reboot the VM and you're all done. You can use [this](apps/pafish.exe) tool to check how stealthy the Virtual Machine is.

## Patching the Kernel

Wanna pass an extra PAFish test? Luckily for you this table here will let you pick which setup you want to do, (the repo one is suggested as you don't have to do much work)

=== "Use my repo"

    So you don't wanna compile your own kernel? I gotchu, I made an arch repo with the zen-kernel built and ready with the patches needed.

    First run the following commands to add my keyring to your system.

    ```bash
    sudo pacman-key --recv-keys 71060F3E4998281A
    sudo pacman-key --lsign 12D08400A54A5B2F
    wget https://cdn.discordapp.com/attachments/983887063113945088/1029189673953800234/46620-keyring-20220930-1-any.pkg_1.tar.zst
    sudo pacman -U 46620-keyring-20220930-1-any.pkg.tar.zst
    ```

    Edit your `/etc/pacman.conf` and add the following block

    ```
    [46620-repo]
    Server = https://repo.46620.moe/$arch
    ```

    After that, install `linux-zen-kvm` and `linux-zen-kvm-headers`, update your grub config `sudo grub-mkconfig -o /boot/grub/grub.cfg` to pick up the new kernel, and then reboot and select the kernel on startup.

=== "Manually compiling your kernel"

    > This will be going over Arch's kernel build method as seen on the [Arch Wiki](https://wiki.archlinux.org/title/Kernel/Arch_Build_System) and the patch file is for the zen kernel, but should work on any kernel with a tweak of the file path.

    First install the `pacman-contrib` package to grab the `asp` command, it'll help with source code management. Next grab the source code. Running `asp update linux-zen` will grab the latest build files and doing `asp export linux-zen`. You can replace linux with any package to grab the latest build files of said package, just in case you wanna compile everything yourself.

    After grabbing the source next you want to make a little build directory for everything, I have mine at `~/Projects/kernel/linux-zen`

    Then download [this patch](https://raw.githubusercontent.com/46620/patches/master/kvm.patch) and put it in the folder with the `PKGBUILD` file.

    Note that the patch was made for version 5.18.x-zen1, and still works as of writing (5.19.9-zen1). If/when it breaks I will attempt to update the patch within a day.

    !!! info "Compile faster"
        Arch's default compile flags are kinda dog water and slow, you can edit `/etc/makepkg.conf` and uncomment the `MAKEFLAGS` line and change it to match your needs.

    Also for compiling the kernel, you should set up [modprobed-db](https://wiki.archlinux.org/title/Modprobed-db) to cut the compile time (and size) by at least half.

    Next you're going to edit the `PKGBUILD` file and add the patch into it (example below) and [disable building docs](https://wiki.archlinux.org/title/Kernel/Arch_Build_System#Avoid_creating_the_doc), after you add it run `updpkgsums` and then `makepkg -si`.

    ```
        source=(
            "$_srcname::git+https://github.com/zen-kernel/zen-kernel?signed#tag=$_srctag"
            config         # the main kernel config file
            kvm.patch # add this line
        )
    ```

---

After you reboot, edit your VM's XML and add `<feature policy="disable" name="rdtscp"/>` to the CPU block and you should pass yet another PAFish test.
