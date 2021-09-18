# Tautulli Scripting

Aight now we get to the fun part, actually setting up scripts. I'm gonna go over one script here as I use it the most.

## PlexAniSync
My friends and I watch a lot of anime and keeping our Anilist accounts up to date is like 4 more clicks than we feel like. The solution? [PlexAniSync](https://github.com/RickDB/PlexAniSync).

### Installing

!!! note "Installing"
    You can set this up in 2 ways, manually, or use their docker image that has Tautulli installed in it. I'm gonna go over manually as that's how I have it set up and prefer it that way.

Installation is very easy, clone the repo somewhere and yell at tautulli to look at it. To make my life easier, I made a "scripts" folder in the Tautulli directory and cloned it in there, so my path looks like this `/opt/Tautulli/scripts/PlexAniSync`. Make sure to also install the pip deps by running `pip install --user -r requirements.txt` in the folder.

### Configs

??? info
    If you did not know already, any text inside of "<>" is a name that you put yourself without the "<>"

Fun thing: if you do this for multiple users, no config will ever be the same. Follow their [config guide](https://github.com/RickDB/PlexAniSync#step-3---configuration) to set up the configs. If you name your config file differently, you can call it by doing `python PlexAniSync.py <config.ini>`.

### Custom mapping
This shit fucking sucks. Lets say, hypohetically, for the sake of the argument, that you have have a large media library right? Now let's say that you combine shows into series with multiple seasons, that would be normal, no? Now let's say hypothetically if the script failed to detect that?

Aight out of the Ben Shapiro rant, this is something the script does do and it annoyed me. Luckily they have a solution with the `custom_mappings.yaml` file. This is a file that you manually have to write up yourself, but I have one [here](https://github.com/46620/custom-mappings) that has everything that I've ever had on my media server. I try to update it when I add stuff. If you want to add something, please send a PR to the repo as it would help out a lot. Just place the file in the root of the folder and it should be picked up everytime the scripts run.

### Setting up in Tautulli

In Tautulli go to `Settings > Notification Agents > Add a new notification agent`. Select `Script` and get ready to config a lot of stuff

First you want to set the script folder and the script to use, below is a screenshot that shows my setup, adapt the paths for yours.

![03_tautulli_pas_1](img/03_tautulli_pas_1.png)

Next go to triggers and check "Playback Stop" and optionally "Watched" if you want to guarantee that the script runs.

Next go to Conditions and set it up similar to the photo below. Make sure the Library Name is where you have your Anime.

![04_tautulli_pas_2](img/04_tautulli_pas_2.png)

Lastly go to the Arguments tab and in "Playback Stop" and "Watched" put in `<config.ini> "{show_name}"`

After you do that and save, just hope that the script works when you play, if not... something something change the python env in the `TautulliSyncHelper.py` to use py3.