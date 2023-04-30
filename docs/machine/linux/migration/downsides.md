# Downsides

Yes, like every Operating system on the planet, there are downsides to using it over another one, let's talk about those here.

## Fragmentation

Like NTFS on Windows, Linux has a lot of fragmentation. With different distros, package managers, folder structures for config files, extra methods of installing apps (Ubuntu with their snaps), and the list goes on. While that helps keep different distributions unique and quirky from each other.

### Package managers

My favorite part of Linux is what gives some people their most pain, the package manager. Instead of hunting down sites for random exe/msi files, you literally have it built into your system, most of the time. If you're on Ubuntu you may need to add a few PPAs throughout your usage, and on Fedora, you kinda need to add a [whole extra repo to use your system](https://rpmfusion.org). It also is annoying if you're following a guide that only has commands and packages for `apt` on Ubuntu, but you're running arch^b^^t^^w^, the package names will be very different.

### Ubuntu Snaps

Snaps fucking suck, I'm not gonna write my whole dislike for them since the last time I used snap was 2019 setting up nextcloud on my server, so here's a few links and you can draw your conclusion.

[Firefox as a Snap is slow as molasses in ubuntu Reddit post](https://www.reddit.com/r/firefox/comments/u8wnux/firefox_as_a_snap_is_slow_as_molasses_in_ubuntu)

[Differences between snap, appimage, and flatpak](https://askubuntu.com/questions/866511/what-are-the-differences-between-snaps-appimage-flatpak-and-others)

[Reddit post "what's up with snapd"](https://www.reddit.com/r/Ubuntu/comments/ih6snd/comment/g2yxndh/?utm_source=share&utm_medium=web2x&context=3)

## Little app support from large companies

This is mostly targetting Adobe, there are a lot of large companies that make Linux ports/don't write their app in a way that makes it impossible to use in wine. [Adobe sadly isn't one of them](https://cdn.discordapp.com/attachments/635205267172360234/1016762613968936980/unknown.png).

I used to use Photoshop and Premiere all the time back when I was on Windows. Since moving to Linux I tried several different methods to be able to continue to use them, but seeing as the only viable way was through a VM, I was basically forced to swap off and go to other programs. Companies push the "We don't support x OS cause it's not popular", while not giving applications to that OS to make it popular, keeping it stuck at the bottom.

## Gaming

> Anything in this section about games not working is from tests on my system, YMMV.

Yeah yeah yeah shut up we knew this was gonna be added. I'm putting it down here since Gaming on Linux has gotten much better since 2018, with [proton](https://github.com/ValveSoftware/Proton) being the literal best thing in the world, to devs slowly [adding anticheat support](https://areweanticheatyet.com). Even with those, some games just don't function at all. [Fortnite](https://twitter.com/TimSweeneyEpic/status/1490565925648715781) and [Destiny 2](https://help.bungie.net/hc/en-us/articles/360049024592-Destiny-2-Steam-Guide) both denied access for Linux users, some games like Trove and Stealth Inc 2 literally just don't load, and Watch Dogs just runs at 18fps. If you want a more accurate page for your library, check out [ProtonDB](https://www.protondb.com) for real people testing games.

## Nvidia

Oh I actually have to describe this one? Nvidia, at the time of writing, does not have a usable [open source video driver](https://github.com/NVIDIA/open-gpu-kernel-modules). The current open source implementation, [Nouveau](https://nouveau.freedesktop.org/), is actually terrible. That's to be expected though, since they're reverse engineering a black box with zero help from Nvidia, it's good for what it is, but as a usable driver, it can't do much. There is a [proprietary driver](https://www.nvidia.com/Download/driverResults.aspx/191961/en-us/) that works really well, but depending on your distro, it may be annoying to install.

---

There are 100% more downsides, but those are the mains ones that people should be aware of. If you still want to move over, continue to the [preparing](../prep) page to get your installation going.
