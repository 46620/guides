# Anime Metadata
Getting proper metadata for anime is a fucking bitch let me tell you. I've been hosting a media server since August 30th, 2018 and I still can't get this down properly, but this part is the knowledge I've gathered over the years to hopefully make it easier.

!!! info "Notice"
    This page will use the paths used in the [install](install.md) page. Please change them to your needs.

When doing this you have 2 options and I will go over both.

??? tldr "HAMA"
    [HAMA.bundle](https://github.com/ZeroQI/Hama.bundle) (HTTP Anidb Metadata Agent) was my first introduction to the world of custom Plex metadata agents and a really good one. Below are the installation commands.

    ``` bash
    docker stop Plex
    git clone --recursive https://github.com/ZeroQI/Hama.bundle.git '/opt/plexmediaserver/config/Library/Application Support/Plex Media Server/Plug-ins/Hama.bundle'
    docker start Plex
    ```

??? tldr "Anilist"
    [Anilist.bundle](https://github.com/sachaw/AniList.bundle) is the current metadata provider I use for my Plex server and I really think it's the best one anyone can use. It works well with a script that will be mentioned on another page. Below are the installation commands.

    ``` bash
    docker stop Plex
    git clone --recursive https://github.com/sachaw/AniList.bundle.git '/opt/plexmediaserver/config/Library/Application Support/Plex Media Server/Plug-ins/Anilist.bundle'
    docker start Plex
    ```

After installing either of the options you will need to also install [ASS](https://github.com/ZeroQI/Absolute-Series-Scanner) (Absolute Series Scanner). This is required for it to.. work (idk why, read the readme on their github if you wanna learn how). Below are the installation commands.

``` bash
docker stop Plex
wget -O '/opt/plexmediaserver/config/Library/Application Support/Plex Media Server/Scanners/Series/Absolute Series Scanner.py' https://raw.githubusercontent.com/ZeroQI/Absolute-Series-Scanner/master/Scanners/Series/Absolute%20Series%20Scanner.py
chmod 775 -R '/opt/plexmediaserver/config/Library/Application Support/Plex Media Server/Scanners'
docker start Plex
```
## Setting default metadata agent

After you wait for Plex to restart, you want to edit your library to use the custom agents, below is a gif on how to do so.

??? tip "Changing the agent"
    ![01_change_agent](img/01_change_agent.gif)