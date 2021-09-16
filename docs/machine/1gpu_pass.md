# Single GPU passthrough
> Last updated: 2021-09-16

So you wanna game but you also wanna use Linux? That's understandable. This guide should help you with that.

> This guide is a combination of reading countless guides over time, some of which I sadly can't find links to. I will make sure that I add credits here at some point with everyones page.

## Requirements
The following are what this guide is based on, please make sure to change certain steps for your build. Read the credits for help.

*    A machine with Manjaro 21.1.2 installed
*    An EVGA RTX 2080 Super Black
*    An AMD Ryzen 5 5600x

Now that we got that out of the way, lets get to installing, crying, and a lot of screaming.

## Installing deps
Single GPU passthrough will need some stuff installed in order to work. Running the following list of commands will install everything for you.

```bash
sudo pacman -S qemu libvirt edk2-ovmf virt-manager iptables-nft dnsmasq
sudo systemctl enable --now libvirtd.service
sudo virsh net-autostart default
sudo virsh net-start default
```

Under construction...