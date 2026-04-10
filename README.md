# Inception

A Docker-based WordPress stack with **Nginx (TLS)**, **WordPress (PHP-FPM)**, and **MariaDB**, orchestrated with Docker Compose.

This setup is designed for the 42 Inception project and runs the full stack through a single `make` workflow.

## Overview

The project builds and runs 3 containers:

- `nginx`: HTTPS reverse proxy on port `443`
- `wordpress`: PHP-FPM + WordPress core
- `mariadb`: database backend on port `3306`

Persistent data is stored through bind-mounted Docker volumes.

## Architecture

```text
Browser (HTTPS :443)
        |
        v
      Nginx
        |
        v
  WordPress (php-fpm :9000)
        |
        v
     MariaDB (:3306)
```

## Project Structure

```text
.
├── Makefile
├── README.md
└── srcs
    ├── .env
    ├── docker-compose.yml
    └── requirements
        ├── mariadb
        │   ├── Dockerfile
        │   └── conf/create_db.sh
        ├── nginx
        │   ├── Dockerfile
        │   └── conf/nginx.conf
        └── wordpress
            ├── Dockerfile
            ├── conf/wp-config-create.sh
            └── tools/make_dir.sh
```

## Prerequisites

Install:

- Docker
- Docker Compose (or Compose plugin compatible with `docker-compose` command)
- GNU Make

You may need `sudo` privileges for cleanup targets that remove host data folders.

## Environment Variables

Variables are defined in `srcs/.env`.

Current keys:

```env
DOMAIN_NAME=edalmis.42.fr
CERT_=./requirements/tools/edalmis.42.fr.crt
KEY_=./requirements/tools/edalmis.42.fr.key
DB_NAME=wordpress
DB_ROOT=rootpass
DB_USER=wpuser
DB_PASS=wppass
TITLE=Inception
ADMIN_USER=edalmis
ADMIN_PASSWORD=1244
ADMIN_EMAIL=edalmis@42.fr
```

## Important Path Note

Your compose file and helper script currently use host paths under:

- `/home/edalmis/data/wordpress`
- `/home/edalmis/data/mariadb`

If your Linux username is different, update:

- `srcs/docker-compose.yml` (`device:` paths in volumes)
- `srcs/requirements/wordpress/tools/make_dir.sh` (directory creation paths)

## Local Domain Setup

Nginx is configured for `edalmis.42.fr`.
Add this line to your `/etc/hosts`:

```text
127.0.0.1 edalmis.42.fr www.edalmis.42.fr
```

## Usage

From project root:

```bash
make
```

Useful targets:

- `make` or `make all`: start stack
- `make build`: build images and start
- `make down`: stop services
- `make re`: rebuild and restart
- `make clean`: remove containers/images (plus project data folders)
- `make fclean`: full Docker cleanup (very destructive)
- `make list`: list containers
- `make volumes`: list volumes

## Access

After startup:

- Main site: `https://edalmis.42.fr`
- Database service (internal + published): `localhost:3306`

Your TLS cert/key are expected in:

- `srcs/requirements/nginx/tools/edalmis.42.fr.crt`
- `srcs/requirements/nginx/tools/edalmis.42.fr.key`

## Troubleshooting

- If volumes fail to mount, verify host paths in `srcs/docker-compose.yml`.
- If `make` fails on missing data directories, run the helper script manually:
  - `bash srcs/requirements/wordpress/tools/make_dir.sh`
- If domain does not resolve, check `/etc/hosts` entry.
- If HTTPS fails, confirm cert/key files exist and names match `nginx.conf`.

## Notes

- The stack uses `restart: always` for resilience.
- WordPress debug log is enabled at `/var/www/wp-content/debug.log` inside the container.
