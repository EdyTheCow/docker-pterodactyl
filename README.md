<p align="center">
  <img width="400" src="https://raw.githubusercontent.com/BeefBytes/Assets/master/Other/pterodactyl-docker/pterodactyl-docker_logo_png_text_625x347.png">
</p>

# About
There’s a lack of information about setting up and running Pterodactyl Panel inside docker using Traefik as reverse proxy. This guide focuses on the fastest and easiest way to do that! 

# Getting Started
We’ll be using docker, docker-compose and CloudFlare for DNS challenges to generate certificates. DNS challenges allow wildcard certificates allowing you to add any subdomains on the go. If you prefer using something else, have a look at [Traefik's Docs](https://docs.traefik.io/https/acme/).

### Requirements
- Basic command line knowledge
- A domain behind Cloudflare
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Understanding the data structure
By default the guide assumes you're cloning this repository into `/srv/docker/` directory! Current data structure looks like this:

```
├── /srv/docker/pterodactyl-docker/data
│   ├── daemon
│   │   ├── config
│   │   ├── packs
│   │   ├── data
│   ├── panel
│   ├── db
│   └── traefik
```
Allowing you to only worry about one directory when backing up or moving the whole setup. If you wish to use the default pterodactyl panel path for daemon change `DATA_DIR_DAEMON` to `/srv` and `CONTAINER_DAEMON_DATA` to `/srv/daemon-data` if you make these changes, you don't need to change `sftp path` in `core.json` when instructed later in the guide.

# Installation

### Basic configuration
Once you have met all of the requirements, start by cloning this repository into `/srv/docker/`
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```

Enter the compose directory and rename `.env.example` to `.env`. The most important variables to change right now are:

| Variable | Example | Description |
|-|:-:|-|
| DOMAIN | example.com | Enter a domain that is behind CloudFlare |
| CF_API_EMAIL | your@email.com | Your CloudFlare's account email |
| CF_API_KEY | - | Go to your CloudFlare's profile and navigate to "API Tokens". Copy the "Global API Key" |
| MYSQL_ROOT_PASSWORD | - | Use a password generator to create a strong password |
| MYSQL_PASSWORD | - | Don't reuse your root's password for this, generate a new one |

<b>Start Traefik</b><br />
 ```
docker-compose up -d traefik
 ```

### Setting up the panel

<b>Initialize the database container</b><br />
Allow around two minutes. Starting the panel before database is fully initialized will cause errors!
 ```
docker-compose up -d db 
 ```

<b>Start the panel container</b><br />
Allow around two minutes to be fully initialized.
 ```
docker-compose up -d panel
 ```
 
<b>Create a user</b><br />
 ```
docker-compose run --rm panel php artisan p:user:make
 ```
Login to the panel using newly created user at panel.DOMAIN you specified earlier
 
### Setting up the daemon
Navigate to admin control panel and add a new `Location`. Then navigate to `Nodes` and create a node.
- Set `FQDN` to `node.DOMAIN` you specified earlier
- Set the node to `Behind Proxy`
- Set the `Daemon Port` to `443`

Navigate to `Configuration` tab and copy the contents into `daemon/config/core.json`. Edit `core.json` and change `remote base` url from `http` to `https`

Change `sftp path` to `/srv/docker/pterodactyl-docker/data/daemon/data`. Full path is required so SFTP is able to access the daemon data. Make sure it matches your `CONTAINER_DAEMON_DATA` variable path!

<b>Start the daemon</b><br />
 ```
docker-compose up -d daemon
 ```

# Known issues
- 2-factor verification outputs invalid credentials error when trying to login using 2FA

# Credits
- Logo created for this project by Wob - [Dribbble.com/wob](https://dribbble.com/wob)
- Docker images for the panel and daemon created by [ccarney16/pterodactyl-docker ](https://github.com/ccarney16/pterodactyl-docker)
