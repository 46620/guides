# After Linux install

## Poking around and getting used to the system

Welcome to Linux! Poke around and get used to the new file system. Below are a few path redirections if you're used to stuffs

`C:\` will be `/`

`C:\Users` will be `/home`

`%APPDATA%` or `C:\Users\%USERNAME%\AppData` will be `/home/<name>/.local/share` or `/home/<name>/.config`, depending on the app

## Update

After installing Linux, even if you know for a fact that the system is up to date, it's best practice to update the whole system.

## Make sure all your hardware is detected

If you use "exotic shit" like a GoXLR, Dual 10GbE card, or other random stuff most normal people don't have, you may need to go and grab some extra packages and drivers for them. [goxlr-on-linux](https://github.com/GoXLR-on-Linux/goxlr-on-linux/wiki/Out-of-Box-Support) solves the GoXLR issue and, at least on arch this is needed, you will need to install an [extra firmware](https://archlinux.org/packages/?q=linux-firmware) package depending on what kind of device it is.

## Getting the correct drivers (Nvidia)

As mentioned on the downsides page, Nvidia is annoying on Linux and the GPU driver in the kernel is ass, so we need to replace it with the proprietary driver and blacklist the open source one.

Depending on your distro, the command for installing will be different, below will be one for arch and one for ubuntu based distros. **Make sure to replace the numbers at the end of the ubuntu command with the latest driver.**

| Arch | Ubuntu |
| ---- | ------ |  
| `sudo pacman -S nvidia-dkms` | `sudo apt install nvidia-dkms-530` |

To blacklist nouveau, run `sudo nano /etc/modprobe.d/blacklist-nouveau.conf` and put the following in the file

```text
blacklist nouveau
options nouveau modeset=0
```
