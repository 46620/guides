# Migrating from Plex

So remember how I never said Plex was the best solution for a media server? Yeah thank fuck I never did that and gave Jellyfin a second chance. Don't get me wrong, I love Plex and I still believe that it's the best looking self hosted media platform you can use. But it's pay to use mobile, limited users, and it's able to lose access to a graphics card while running in Docker. So for those reasons I will be describing how I managed to migrate 30+ users from Plex to Jellyfin with ~~little to no~~ a bit of effort.

## What won't transfer

Before I show you how to do this, lemme give a warning of what will not trasnfer.

Watch history will sorta-maybe work. This happened to a few of my users, so please make sure that they can see their watch history on Plex to mark what they have already watched.

User Icons will also not transfer, as it does not pull those.

Usernames will mostly pull, please note if someone doesn't have a username, the username will be their email address on the Plex account.

## How to do this

First clone [this](https://github.com/nwithan8/Plex2Jellyfin) script and `cd` into it.

```bash
git clone https://github.com/nwithan8/Plex2Jellyfin.git
cd Plex2Jellyfin
```

Next go to your Jellyfin instance you installed and go to `Administrator - Dashboard > Advanced - API Keys` and create a new API key, it doesn't matter the name, just make sure to **NEVER GIVE IT TO ANYONE**

Then install all pip requirements, go into the scripts folder, and make a copy of the creds.py file, and modify it to match what you need

```bash
pip install -r requirements.txt
cd scripts
cp creds.py.blank creds.py
```

Below is an example file. It will obviously not work as the info is not valid, please make sure to put your information in it
??? tldr "Example creds.py file"
    ```python
    PLEX_URL = "http://localhost:32400"
    PLEX_TOKEN = "plextokenyoucangetfromtautulli"
    PLEX_SERVER_NAME = "46620's Server" # As it appears on app.plex.tv
    JELLYFIN_URL = "https://jellyfin.domain.tld" # include /jellyfin if needed (whatever URL used for mobile apps)
    JELLYFIN_API_KEY = "abcdefghijklmnopqrstuvwxyz123456"
    JELLYFIN_ADMIN_USERNAME = 'USERNAME'
    JELLYFIN_ADMIN_PASSWORD = 'PASSWORD'
    JELLYFIN_USER_POLICY = {
        "IsAdministrator": "false",
        "IsHidden": "true",
        "IsHiddenRemotely": "true",
        "IsDisabled": "false",
        "EnableRemoteControlOfOtherUsers": "false",
        "EnableSharedDeviceControl": "false",
        "EnableRemoteAccess": "true",
        "EnableLiveTvManagement": "false",
        "EnableLiveTvAccess": "true",
        "EnableContentDeletion": "false",
        "EnableContentDownloading": "true",
        "EnableSyncTranscoding": "true",
        "EnableSubtitleManagement": "false",
        "EnableAllDevices": "true",
        "EnableAllChannels": "false",
        "EnablePublicSharing": "false",
        "InvalidLoginAttemptCount": 5,
        "BlockedChannels": [
            "IPTV",
            "TVHeadEnd Recordings"
        ],
        "AuthenticationProviderId": "Emby.Server.Implementations.Library.DefaultAuthenticationProvider",
        "PasswordResetProviderId": "Emby.Server.Implementations.Library.DefaultPasswordResetProvider"
    }
    ```

Afterwards run `python migrate_users.py`
