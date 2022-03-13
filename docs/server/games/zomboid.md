# Project Zomboid

> I'm gonna keep it 100 with you if you actually use this guide. I spent 3 days working on making a server for my boyfriend and after those 3 days I decided to write this guide because I don't want someone else to go through the pain that is creating a fucking Project Zomboid server.

## Getting Started

For this you will need the following installed to the system:

  - Docker

That's it.

## Setting up the server

For the sake of making this guide simple I will be assuming the following:

 - The server data will be in `/opt/prozom/ZomboidConfig` and `/opt/prozom/ZomboidDedicatedServer`
 - That you're using linux.
 - You know how a monorail works.

Enter the `/opt/prozom` folder and run `mkdir -p Zomboid{Config,DedicatedServer}` to create the paths for the configs.

Next run `git clone https://github.com/Renegade-Master/zomboid-dedicated-server.git` and `cd` into `zomboid-dedicated-server` and edit the `docker-compose.yaml` file.

What you want to edit:

 - The user and group ID (optional: edit if you want to run the server as a certain user)
 - The admin username and password
 - The volumes (set the paths to be the absolute paths above)

An example of the volumes would look like this

```yaml
...
    volumes:
      - /opt/prozom/ZomboidDedicatedServer:/home/steam/ZomboidDedicatedServer
      - /opt/prozom/ZomboidConfig:/home/steam/Zomboid/
...
```

After running that command you can then run `docker-compose up --build --detach` and the server will start up.

## Adding mods
> This was a fucking bitch

So I don't want to admit how long this took me to figure out properly but just so you don't mess it up I'll explain it better than everyone else did.

Go back into the `docker-compose.yaml` file and edit the `MOD_NAMES` and `MOD_WORKSHOP_IDS` to include the IDs of each mod separated with a semicolon.

Just note one thing: `MOD_NAMES` (or modIDS as they're called in other guides) are not the same as the workshop IDs.. for some mods.

From my 5 minutes of looking, each mods workshop page will have the modID/MOD_NAME at the bottom, so you just need to grab all of those. Below is a snippit of what I mean.

**ALSO MAKE SURE THAT MODIDS AND WORKSHOP IDS ARE IN ORDER WITH EACHOTHER! IF CHAIRS IS FIRST THE WORKSHOP ID HAS TO BE FIRST!**

!!! info "Correct"
    ```yaml
    ...
      - "MOD_NAMES=Hydrocraft;Chair"
      - "MOD_WORKSHOP_IDS=2081538550;2664883430"
    ...
    ```

!!! warning "Incorrect (workshopID as modID)"
    ```yaml
    ...
      - "MOD_NAMES=2081538550;2664883430"
      - "MOD_WORKSHOP_IDS=2081538550;2664883430"
    ...
    ```

!!! warning "Incorrect (IDs not in order)"
    ```yaml
    ...
      - "MOD_NAMES=Hydrocraft;Chair"
      - "MOD_WORKSHOP_IDS=2664883430;2081538550"
    ...
    ```

Doesn't that sound simple and easy to explain? Yeah well no guides that I looked at actually fucking tell you that, and the official doesn't even describe how to add mods, or even have a linux section for configs. This entire guide was written out of being salty over a single java app and I am still mad. Literally a skill issue and I'm malding. Also my boyfriend spent 3 fucking hours manually copying modIDs and workshopIDs into a file for us to play with just for the game to still not let him join at all. Fuckin brits.