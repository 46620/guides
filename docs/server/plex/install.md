# Plex

!!! info "Notice"
    Make sure to read the [requirements](../../requirements.md) page before following this.

## Installation

To install PMS (Plex Media Server) with docker is a very easy thing to do. Just run the following command and replace the timezone and paths to match your setup.

```bash
docker run \
-d \
--name Plex \
--network=host \
-e TZ="EST" \
-v /opt/plexmediaserver/config:/config \
-v /opt/plexmediaserver/transcode:/transcode \
-v /media:/data \
plexinc/pms-docker:plexpass
```
> The plexpass tag will only be in effect if you have PlexPass, if not it will use the public release.

