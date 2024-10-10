# Satisfactory Server

![Satisfactory](https://raw.githubusercontent.com/wolveix/satisfactory-server/main/.github/logo.png "Satisfactory logo")

![Release](https://img.shields.io/github/v/release/wolveix/satisfactory-server)

This is a Dockerized version of the [Satisfactory](https://store.steampowered.com/app/526870/Satisfactory/) dedicated
server.

## Setup

The server may run on less than 8GB of RAM, though 8GB - 16GB is still recommended per
the [the official wiki](https://satisfactory.wiki.gg/wiki/Dedicated_servers#Requirements). You may need to increase the
container's defined `--memory` restriction as you approach the late game (or if you're playing with many 4+ players)

You'll need to bind a local directory to the Docker container's `/config` directory. This directory will hold the
following directories:

- `/backups` - the server will automatically backup your saves when the container first starts
- `/gamefiles` - this is for the game's files. They're stored outside the container to avoid needing to redownload
  8GB+ every time you want to rebuild the container
- `/logs` - this holds Steam's logs, and contains a pointer to Satisfactory's logs (empties on startup unless
  `LOG=true`)
- `/saved` - this contains the game's blueprints, saves, and server configuration

Before running the server image, you should find your user ID that will be running the container. This isn't necessary
in most cases, but it's good to find out regardless. If you're seeing `permission denied` errors, then this is probably
why. Find your ID in `Linux` by running the `id` command. Then grab the user ID (usually something like `1000`) and pass
it into the `-e PGID=1000` and `-e PUID=1000` environment variables.

Run the Satisfactory server image like this (this is one command, make sure to copy all of it):<br>

```bash
docker run \
--detach \
--name=satisfactory-server \
--hostname satisfactory-server \
--restart unless-stopped \
--volume ./satisfactory-server:/config \
--env MAXPLAYERS=4 \
--env PGID=1000 \
--env PUID=1000 \
--env ROOTLESS=false \
--env STEAMBETA=false \
--memory-reservation=4G \
--memory 8G \
--publish 57923:57923/udp \
--publish 57923:57923/tcp \
wolveix/satisfactory-server:latest
```

<details>
<summary>Explanation of the command</summary>

* `--detach` -> Starts the container detached from your terminal<br>
  If you want to see the logs replace it with `--sig-proxy=false`
* `--name` -> Gives the container a unqiue name
* `--hostname` -> Changes the hostname of the container
* `--restart unless-stopped` -> Automatically restarts the container unless the container was manually stopped
* `--volume` -> Binds the Satisfactory config folder to the folder you specified
  Allows you to easily access your savegames
* For the environment (`--env`) variables please
  see [here](https://github.com/wolveix/satisfactory-server#environment-variables)
* `--memory-reservation=4G` -> Reserves 4GB RAM from the host for the container's use
* `--memory 8G` -> Restricts the container to 6GB RAM
* `--publish` -> Specifies the ports that the container exposes<br>

</details>

### Docker Compose

If you're using [Docker Compose](https://docs.docker.com/compose/):

```yaml
services:
  satisfactory-server:
    container_name: 'satisfactory-server'
    hostname: 'satisfactory-server'
    image: 'wolveix/satisfactory-server:latest'
    ports:
      - '57923:57923/udp'
      - '57923:57923/tcp'
    volumes:
      - './satisfactory-server:/config'
    environment:
      - MAXPLAYERS=4
      - PGID=1000
      - PUID=1000
      - ROOTLESS=false
      - STEAMBETA=false
    restart: unless-stopped
    healthcheck:
      test: [ "CMD", "bash", "/healthcheck.sh" ]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s
    deploy:
      resources:
        limits:
          memory: 8G
        reservations:
          memory: 4G
```

## Environment Variables

| Parameter               |  Default  | Function                                                  |
|-------------------------|:---------:|-----------------------------------------------------------|
| `AUTOSAVENUM`           |    `5`    | number of rotating autosave files                         |
| `DEBUG`                 |  `false`  | for debugging the server                                  |
| `DISABLESEASONALEVENTS` |  `false`  | disable the FICSMAS event (you miserable bastard)         |
| `LOG`                   |  `false`  | disable Satisfactory log pruning                          |
| `LOGNETLEVEL`           |  `Error`  | print network logs at or above this log level             |
| `LOGNETTRAFFICLEVEL`    | `Warning` | print network traffic logs at or above this log level     |
| `MAXOBJECTS`            | `2162688` | set the object limit for your server                      |
| `MAXPLAYERS`            |    `4`    | set the player limit for your server                      |
| `MAXTICKRATE`           |   `30`    | set the maximum sim tick rate for your server             |
| `PGID`                  |  `1000`   | set the group ID of the user the server will run as       |
| `PUID`                  |  `1000`   | set the user ID of the user the server will run as        |
| `ROOTLESS`              |  `false`  | run the container as a non-root user                      |
| `SERVERGAMEPORT`        |  `57923`   | set the game's port                                       |
| `SERVERIP`              | `0.0.0.0` | set the game's ip (usually not needed)                    |
| `SERVERSTREAMING`       |  `true`   | toggle whether the game utilizes asset streaming          |
| `SKIPUPDATE`            |  `false`  | avoid updating the game on container start/restart        |
| `STEAMBETA`             |  `false`  | set experimental game version                             |
| `TIMEOUT`               |   `30`    | set client timeout (in seconds)                           |
| `VMOVERRIDE`            |  `false`  | skips the CPU model check (should not ordinarily be used) |

For the log level variables, the following values are supported by the game:

- `Fatal`
- `Error`
- `Warning`
- `Display`
- `Log`
- `Verbose`
- `VeryVerbose`
