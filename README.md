<p align="center">
  <img width="500" src="https://raw.githubusercontent.com/BeefBytes/Assets/master/Other/container_illustration/v2/dockerized_pterodactyl.png">
</p>

# 📚 About
There’s a lack of information about setting up and running Pterodactyl Panel inside docker using Traefik as a reverse proxy. This guide focuses on the fastest and easiest way to do that! This setup has been tested and is currently running in a production environment. All of images used in this setup are from official sources and even uses official compose files provided by Pterodactyl with addition of Traefik.

# 🧰 Getting Started
This guide assumes you have at least two servers, one for panel and one for wings. You're fine by using a cheap VPS for the panel, while wings may require a higher spec server depending on the game servers you're planning to run.

### Requirements
- Domain
- Two or more servers
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### DNS
Create `A record` pointing to panel's server IP, if you're using Cloudflare you can proxy this record through Cloudflare.

Create second `A record` pointing to wings server IP. If you're using Cloudflare, do not proxy this record! There's no advantages for proxying wings server. If it is proxied the server SFTP details in the panel will point to Cloudflare's IP rather than wings.

# 🏗️ Installation

### Create pterodactyl docker network
```bash
docker network create pterodactyl
```

### Preparations / Setting up Traefik
<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```

<b>Set correct acme.json permissions</b><br />

Navigate to `_base/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Start docker compose</b><br />
Inside of `_base/compose` run
 ```
docker-compose up -d
 ```

### Setting up Panel

<b>Configure variables</b><br />
Navigate to `panel/compose/.env` and set `PANEL_DOMAIN` to the domain you pointed to panel's server earlier.

Navigate to `panel/compose/docker-compose.yml` and set these variables


| Variable | Example | Description |
|-|:-:|-|
| MYSQL_ROOT_PASSWORD | - | Use a password generator to create a strong password |
| MYSQL_PASSWORD | - | Don't reuse your root's password for this, generate a new one |
| APP_URL | https://panel.example.com | Same as `PANEL_DOMAIN` but with `https://` included|

Rest of the variables can be set as desired, these three are required for panel's basic functionality.

<b>Start docker compose</b><br />
Inside of `panel/compose` run
 ```
docker-compose up -d
 ```
Navigate to the domain you've set for `PANEL_DOMAIN` earlier and make sure panel is up and running.

<b>Create a new user</b><br />
Inside of `panel/compose` run
 ```
docker-compose run --rm panel php artisan p:user:make
 ```
Login into the panel using newly created user.

<b>Create a new node</b><br />
Navigate to the admin control panel and add a new `Location`. Then navigate to `Nodes` and create a node.

| Setting | Set to | Description |
|-|:-:|-|
| FQDN | Wings domain | Domain you pointed to wings server|
| Behind Proxy | Behind Proxy | Set this to `Behind Proxy` for Traefik to work properly|
| Daemon Port | 443 | Change the default port |

Rest of the settings can be set as you desire. You can leave `Daemon Server File Directory` as is unless you want to store servers data in a specific location. In that case make sure to read instruction in `wings/compose/.env`. Otherwise proceed with the guide.

### Setting up Wings
The guide assumes you're setting this up on a different server than the panel is running on!
Go back to the `Preparations / Setting up Traefik` section and follow the same instruction for setting up Traefik.

<b>Configure variables</b><br />
Navigate to `wings/compose/.env` and set `WINGS_DOMAIN` to the domain you pointed to wings server earlier.

<b>Copying daemon's config</b><br />
Navigate to the panel in your web browser and find the node you created earlier. Click on `Configuration` tab and copy the contents into `wings/data/wings/etc/config.yml`.

<b>Start docker compose</b><br />
 ```
docker-compose up -d
 ```

# 🐛 Known issues
- None

# 📜 Credits
- Logo created by Wob - [Dribbble.com/wob](https://dribbble.com/wob)
