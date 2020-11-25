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
- Two or more servers
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Setting up Cloudflare

TODO


# Installation
**Notes before installation**
- By default the guide assumes you're cloning this repository into `/srv/docker/` directory! 
- It's not recommended to run panel and daemon on the same server! While this may be possible, it may cause issues.

### Setting up Traefik
<b>Clone repository</b><br />
Start by cloning this repository into `/srv/docker/`. 
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```

<b>Set variables</b><br />
Navigate to `_base/compose/` directory, rename .env.example to .env and change these variables:

| Variable | Example | Description |
|-|:-:|-|
| CF_API_EMAIL | your@email.com | Your CloudFlare's account email |
| CF_API_KEY | - | Go to your CloudFlare's profile and navigate to "API Tokens". Copy the "Global API Key" |

<b>Start traefik container</b><br />
 ```
docker-compose up -d traefik
 ```

### Panel - (server A)
<b>Set variables</b><br />
Navigate to `panel/compose/` directory and rename .env.example to .env. The most important variables to change right now are:

| Variable | Example | Description |
|-|:-:|-|
| PANEL_DOMAIN | panel.example.com | Enter a domain that's behind CloudFlare |
| APP_URL | https://panel.example.com | Same as `PANEL_DOMAIN` but with `https://` included|
| MYSQL_ROOT_PASSWORD | - | Use a password generator to create a strong password |
| MYSQL_PASSWORD | - | Don't reuse your root's password for this, generate a new one |


<b>Initialize database container</b><br />
Allow around a minute or two before starting the panel. Starting it before database is fully initialized may cause errors!
 ```
docker-compose up -d mysql
 ```

<b>Start panel container</b><br />
Allow around two minutes to be fully initialized.
 ```
docker-compose up -d panel
 ```

<b>Start panel's worker and cron containers</b><br />
 ```
docker-compose up -d worker cron
 ```

<b>Create a new user</b><br />
 ```
docker-compose run --rm panel php artisan p:user:make
 ```
Login into the panel using newly created user by navigating to domain you've set in `PANEL_DOMAIN`

<b>Create a new node</b><br />
Navigate to admin control panel and add a new `Location`. Then navigate to `Nodes` and create a node.

| Setting | Set to | Description |
|-|:-:|-|
| FQDN | `DAEMON_DOMAIN` | This is your daemon's domain you'll have to specify later in guide. Example: `node.example.com`|
| Behind Proxy | Behind Proxy | Set this to `Behind Proxy` for Traefik to work properly|
| Daemon Port | 443 | Change the default port |
| Daemon Server File Directory | `DATA_DIR_DAEMON` | By default it should be set to `/srv/docker/pterodactyl-docker/daemon/data/daemon/servers`. This setting can be changed if desired and is found in `daemon/compose/.env-example` |

Rest of the settings can be set as you desire.

### Daemon - (Server B)

Follow the same steps from `Setting up Traefik` section on your second server for daemon.

<b>Setting variables</b><br />
Navigate to `panel/daemon/` directory and rename .env.example to .env and change these variables:

| Variable | Example | Description |
|-|:-:|-|
| DAEMON_DOMAIN | node.example.com | Enter a domain that's behind CloudFlare |

<b>Copying daemon's config</b><br />
Navigate to `PANEL_DOMAIN` and find the node you created earlier. Click on `Configuration` tab and copy the contents into `daemon/data/config/config.yml`. Sometimes the `remote` url inside `config.yml` may be set to `http://` change it to `https://`.

<b>Start daemon container</b><br />
 ```
docker-compose up -d daemon
 ```


# Known issues
- None

# Credits
- Logo created for this project by Wob - [Dribbble.com/wob](https://dribbble.com/wob)
- Docker images for the panel and daemon created by [ccarney16/pterodactyl-docker ](https://github.com/ccarney16/pterodactyl-docker)

