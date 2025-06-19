# HumHub - Docker Image (Development Version)

The Docker package provides all the essential components for setting up your HumHub installation.

## Features

- HumHub Core Software (Apache2 + FPM)
- MariaDB (Database Server)
- Caddy (Reverse Proxy w/ automatic LetsEncrypt SSL certificates)
- Redis (Cache / Queue)

## Quick Start

### Installation

```
git clone git@github.com:humhub/docker-dev.git /opt/humhub
cd /opt/humhub

cp humhub.env.dist humhub.env
cp .env.dist .env
```

Then edit the `.env` file and set at least the `HUMHUB_DOCKER_DOMAIN`.

```
docker compose up -d
```

> On older Docker versions you may need to run `docker-compose up -d` instead. 

Open your HumHub installation at: https://YOUR-HUMHUB_DOCKER_DOMAIN

> After installation, an e-mail server must be configured at: `Administration` -> `Settings` -> `Advanced` -> `E-Mail`.

### Upgrading

Running following commands in the main directory.

```
cd /opt/humhub

git pull

docker compose pull
docker compose down
docker compose up -d
```

> Also check for new configuration options in `.env.dist` and `humhub.env.dist` files.

### Migration

If you want to migrate from a Non-Docker setup to this Docker image, you can find step-by-step guide in the [MIGRATE.md](MIGRATE.md) file. 

## Other Setup Options

### HumHub CLI 

You can use the wrapper script in the main directoy.

```
./yii.sh help
```

### Version Control

You can define the HumHub version using the variable `HUMHUB_DOCKER_VERSION`. 

The following tags are currently available:
- `master` - For the current stable version
- `develop` - For the current development version
- `next` - For the next development version

For older versions:
- `v1.17` (latest v17 branch)
- `v1.17.2`
- `v1.17.1`
- `v1.17.0`

### Custom Themes & Modules

You can store your own themes/modules in the  folders `/opt/humhub/humhub-data/themes` and `/opt/humhub/humhub-data/custom-modules`. 

### Existing Reverse Proxy

**NGINX Example:** 

If an NGINX web server is already running on the host on which the Docker container is started on the HTTPS port, a reverse proxy to the HumHub Docker container can be created in a virtual host via the following block. 

```
   	location / {
		proxy_pass http://127.0.0.1:8404;
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
	}
```    

**Apache2 Example:** 

TBD

### Without Bundled MariaDB Server

You can also use the Docker Compose setup without the integrated MariaDB server, for example to use an existing cluster
or locally installed MariaDB server.

First set the `HUMHUB_DOCKER_DB_*` variables in the `.env` file.

- `HUMHUB_DOCKER_DB_DSN` - The database connection string.
- `HUMHUB_DOCKER_DB_USER` - The database user.
- `HUMHUB_DOCKER_DB_PASSWORD` - The database password.

e.g.

```env
HUMHUB_DOCKER_DB_DSN="mysql:host=host.docker.internal;dbname=humhub_master"
HUMHUB_DOCKER_DB_USER="humhub-user"
HUMHUB_DOCKER_DB_PASSWORD="humhub-user-password"
```

To start the Docker container without the MariaDB server, run:

```bash
docker compose -f compose-without-db.yaml up -d
``` 

Note: The helper script `helper.sh` database commands are not available in this case.

### Docker Standalone

You can also run the Docker image standalone without Docker Compose and all dependencies. 
A MariaDB database server is required.

For this setup, you don't need to clone this Git repository, but may want to copy the `yii.sh` helper script.

Example command:

```bash
sudo docker run -d \
    --net=host \
    -v ./humhub-data-docker:/var/lib/humhub \
    -e HUMHUB_DEBUG=true \
    -e HUMHUB_CONFIG__COMPONENTS__DB__DSN="mysql:host=127.0.0.1;dbname=---your-db-name---" \
    -e HUMHUB_CONFIG__COMPONENTS__DB__USERNAME="---your-db-user---" \
    -e HUMHUB_CONFIG__COMPONENTS__DB__PASSWORD="---your-db-password---" \
    -e HUMHUB_FIXED_SETTINGS__BASE__BASE_URL="http://localhost:8404" \
    humhub/humhub-dev:develop
```

After starting, HumHub can be accessed at the URL http://localhost:8404.

A reverse proxy is recommended for this setup. In this case, the ENV variable `HUMHUB_FIXED_SETTINGS__BASE__BASE_URL` 
must be set to the correct host name. See section **Existing Reverse Proxy** for more details.
