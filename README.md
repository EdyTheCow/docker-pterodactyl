# About
This guide provides step by step instruction in setting up pterodactyl panel using docker containers and traefik.

## Getting Started
As long as docker and docker compose is correctly installed, it doesn’t matter what distro is being used for this setup. This guide will be using ubuntu.

### Requirements
- Basic command line knowledge
- Cloudflare account (free)
- Domain behind Cloudflare

### Installing docker & docker-compose
Skip this step if you already have docker and compose installed.

<b>Installing docker</b><br />
```
wget -qO- https://get.docker.com/ | sh
```

<b>Adding your user to docker group</b><br />
```
sudo usermod -aG docker username
```

<b>Installing docker-compose</b><br />
```
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

<b>Apply executable permissions to the binary</b><br />
```
sudo chmod +x /usr/local/bin/docker-compose
```

## Setting up services
First you’ll need to clone the repository<br />
```
git clone https://github.com/EdyTheCow/docker-pterodactyl.git
```

# TODO
- Add variables for directory paths and domain
- Remove unnecessary networks
- Update docker compose version
- Add more information about panel's configuration and traefik's setup
