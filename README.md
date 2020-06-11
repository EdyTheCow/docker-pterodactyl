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

# Installation

### Basic configuration
Once you have met all of the requirements, start by cloning this repository
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```

Enter the compose directory and rename `.env.example` to `.env`. The most important variables to change right now are:

| Variable | Example | Description |
|-|:-:|-|
| DOMAIN | example.com | Enter a domain that is behind CloudFlare |
| CF_API_EMAIL | your@email.com | You CloudFlare's account email |
| CF_API_KEY | - | Go to your CloudFlare's profile and navigate to "API Tokens". Copy the "Global API Key" |
| MYSQL_ROOT_PASSWORD | - | Use a password generator to create a strong password |
| MYSQL_PASSWORD | - | Don't reuse your root's password for this, generate a new one |

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
Login to the panel using newly created user at the domain you specified earlier.
 
### Setting up the daemon
Navigate to admin control panel and add a new `Location`. Then navigate to `Nodes` to create a new one.
- Set `FQDN` to `node.YourDomain.com` 
- Set the node to `Behind Proxy`
- Set the `Daemon Port` to `443`

Navigate to `Configuration` tab and copy the contents to `daemon/daemon-config/core.json`. Edit `core.json` and change `remote base` url to `https`. 

Change `sftp path` to `/srv/docker/pterodactyl-docker/data/daemon/daemon-data`. This is to change the default location of daemon's data for more organization. I will add additional variables to make this more flexiable soon. As of right now, make sure your cloned directory is in `/srv/docker/` or edit the volume paths manually in `docker-compose.yml`.

<b>Start the daemon</b><br />
 ```
docker-compose up -d daemon
 ```

# Credits
- Logo created for this project by Wob - [Dribbble.com/wob](https://dribbble.com/wob)
- Docker images for the panel and daemon created by [ccarney16/pterodactyl-docker ](https://github.com/ccarney16/pterodactyl-docker)
