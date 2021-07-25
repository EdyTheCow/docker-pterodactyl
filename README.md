<p align="center">
  <img width="400" src="https://raw.githubusercontent.com/BeefBytes/Assets/master/Other/container_illustration/v2/dockerized_pterodactyl.png">
</p>

# 📚 About
There’s a lack of information about setting up and running Pterodactyl Panel inside docker using Traefik as reverse proxy. This guide focuses on the fastest and easiest way to do that! This setup has been tested and is currently running in a production environment. All of images used in this setup are from official sources and even uses official compose files provided by Pterodactyl with addition of Traefik. 

# 🧰 Getting Started
It is recommended to use Cloudflare but it's not a requirement. Cloudflare is only used to handle the A records and proxy panel through Cloudflare to take advantage of Cloudflare's network. The setup will still work as intended without use of Cloudflare. The guide however will use Cloudflare to explain the domain configuration which can be applied to any other service.

### Requirements
- Domain
- Two or more servers
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Cloudflare

<b>Settings</b><br />
Login into your CloudFlare account and navigate to the domain you'll be using for this setup. Click on `SSL/TLS` tab and make sure your `encryption mode` is set to `Full`.

<b>DNS</b><br />
Navigate to `DNS` tab and create `A record` pointing to panel's server, make sure the record is proxied through Cloudflare (orange clould is enabled).

Create second `A record` pointing to wings server. This time make sure the record is not proxied through Cloudflare! There's no advantage for proxying wings server. If it is proxied the server SFTP details in the panel will point to Cloudflare's IP rather than wings.


# 🏗️ Installation
**Notes before installation**
- It's not recommended to run panel and wings on the same server! While this may be possible, it may cause issues. This guide assumes panel and wings are running on two separate servers.

### Setting up Panel
<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```

<b>Set correct acme.json permissions</b><br />

Navigate to `panel/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Configure variables</b><br />
Navigate to `panel/compose/.env` and set `TRAEFIK_PANEL_DOMAIN` to the domain you pointed to panel's server earlier.

Navigate to `panel/compose/docker-compose.yml` and set these variables


| Variable | Example | Description |
|-|:-:|-|
| MYSQL_ROOT_PASSWORD | - | Use a password generator to create a strong password |
| MYSQL_PASSWORD | - | Don't reuse your root's password for this, generate a new one |
| APP_URL | https://panel.example.com | Same as `TRAEFIK_PANEL_DOMAIN` but with `https://` included|

Rest of the variables can be set as desired, these three are required for panel's basic functionality.

<b>Start docker compose</b><br />
Inside of `panel/compose` run
 ```
docker-compose up -d
 ```
Navigate to the domain you've set for `TRAEFIK_PANEL_DOMAIN` earlier and make sure panel is up and running

<b>Create a new user</b><br />
Inside of `panel/compose` run
 ```
docker-compose run --rm panel php artisan p:user:make
 ```
Login into the panel using newly created user

<b>Create a new node</b><br />
Navigate to admin control panel and add a new `Location`. Then navigate to `Nodes` and create a node.

| Setting | Set to | Description |
|-|:-:|-|
| FQDN | Wings domain | Domain you pointed to wings server|
| Behind Proxy | Behind Proxy | Set this to `Behind Proxy` for Traefik to work properly|
| Daemon Port | 80 | Change the default port |

Rest of the settings can be set as you desire. Do not change `Daemon Server File Directory` unless you change it to something else in wing's docker-compose.yml

### Setting up Wings
The guide assumes you're setting this up on a different server than the panel is running on!

<b>Clone repository</b><br />
```
git clone https://github.com/EdyTheCow/pterodactyl-docker.git
```
<b>Set correct acme.json permissions</b><br />

Navigate to `wings/data/traefik/` and run
```
sudo chmod 600 acme.json
```

<b>Configure variables</b><br />
Navigate to `wings/compose/.env` and set `TRAEFIK_WINGS_DOMAIN` to the domain you pointed to wings server earlier.


<b>Copying daemon's config</b><br />
Navigate to the panel in your web browser and find the node you created earlier. Click on `Configuration` tab and copy the contents into `wings/data/wings/etc/config.yml`. Sometimes the `remote` url inside `config.yml` may be set to `http://` change it to `https://`.

<b>Start docker compose</b><br />
 ```
docker-compose up -d
 ```


# 🐛 Known issues
- None

# 📜 Credits
- Logo created by Wob - [Dribbble.com/wob](https://dribbble.com/wob)
