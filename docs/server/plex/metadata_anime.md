# Anime Metadata

Getting proper metadata for anime is a fucking bitch let me tell you. I've been hosting a media server since August 30th, 2018 and I still can't get this down properly, but this part is the knowledge I've gathered over the years to hopefully make it easier.

!!! info "Notice"
    This page will use the paths used in the [install](install.md) page. Please change them to your needs.

There's 2 different metadata agents that exist and both have their advantages, below is a command to install both and to install Absolute Series Scanner

!!! tldr "Install command"

    ``` bash
    bash <(curl -sL https://raw.githubusercontent.com/46620/Scripts/master/plex/aha_update.sh)
    ```

## Setting default metadata agent

After you wait for Plex to restart, you want to edit your library to use the custom agents, below is a gif on how to do so.

!!! tip "Changing the agent"
    ![01_change_agent](img/01_change_agent.gif)

!!! info "Github links"
    [HAMA.bundle](https://github.com/ZeroQI/Hama.bundle)

    [Anilist.bundle](https://github.com/sachaw/AniList.bundle)

    [ASS](https://github.com/ZeroQI/Absolute-Series-Scanner)