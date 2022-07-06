# Jellyfin

!!! info "Notice"
    Make sure to read the [requirements](../../requirements.md) page before following this.

## Installation

Setting up Jellyfin is actually a bit funny. I usually recommend setting things up with docker-compose + portainer for managing the stack. But this is the first time I'd recommend using just the docker commands, as for some reason it works better.

> Please make sure to change the paths to be what you want to use and your user/group ids are.

```bash
docker run -d \
 --name jellyfin \
 --user uid:gid \
 --net=host \
 --volume /opt/jellyfin/config:/config \
 --volume /opt/jellyfin/cache:/cache \
 --mount type=bind,source=/media,target=/media \
 --restart=unless-stopped \
 jellyfin/jellyfin:unstable
```
> I personally run unstable builds as they're just as stable as stable.. so shut your mouth

After this, spend some time to set up your libraries, metadata, and (if you want), set up a [custom theme](https://jellyfin.org/docs/general/clients/css-customization.html).