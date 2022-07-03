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
| APP_URL | https://localhost | Same as `PANEL_DOMAIN` but with `https://` included|

Rest of the variables can be set as desired, these three are required for panel's basic functionality.

<b>Start docker compose</b><br />
Inside of `panel/compose` run
 ```
docker-compose up -d
 ```
Generates a new encryption key for the APP.
```
docker-compose run --rm panel php artisan key:generate
```
Criar o arquivo de configuração do pterodactyl
```
docker-compose run --rm panel cat > /etc/nginx/conf.d/pterodactyl.conf
```
Colar no console as configurações :
alterar o \<domain>
```xml
server_tokens off;

server {
    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name <domain>;

    root /var/www/pterodactyl/public;
    index index.php;

    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration - Replace the example <domain> with your domain
    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
CTL + C para salvar

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
