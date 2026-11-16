# Digital Library Portal

This project includes a Docker Compose deployment for a centralized WordPress
digital library portal backed by MySQL. The containers communicate over the
custom `library_network` bridge network, while named volumes preserve the
database and WordPress files across container restarts and replacements.

## Prerequisites

- Docker Engine with the Compose plugin
- A browser

## Configure and start

```bash
cp .env.example .env
# Edit .env and set non-default passwords.
docker pull mysql:8.0
docker pull wordpress:latest
docker compose up -d
docker compose ps
```

Open [http://localhost:8080](http://localhost:8080) and complete the
WordPress setup. If `WORDPRESS_PORT` is changed in `.env`, use that port in the
browser URL.

The Compose file explicitly configures:

- `library_network`, a custom bridge network used by both services.
- `WORDPRESS_DB_HOST=db:3306`, so WordPress resolves MySQL by its Compose
  service name rather than by a container IP.
- `library_db_data`, mounted at `/var/lib/mysql`, for persistent database data.
- `library_wordpress_data`, mounted at `/var/www/html`, for persistent
  WordPress files.
- Host port `8080` mapped to WordPress port `80`.
- `restart: always` on both services and a MySQL health check to support
  automatic recovery.

## Verify connectivity and recovery

Show the custom network and persistent volumes:

```bash
docker network inspect library_network
docker volume inspect library-management_library_db_data
docker volume inspect library-management_library_wordpress_data
```

Verify that WordPress can resolve and reach MySQL:

```bash
docker compose exec wordpress getent hosts db
docker compose exec db mysqladmin ping -h localhost -u root -p
```

To demonstrate container recovery, stop WordPress and wait for Docker to
restart it:

```bash
docker stop library-wordpress
docker compose ps
curl -I http://localhost:8080
```

To demonstrate database persistence, create or update content in WordPress,
then restart the stack:

```bash
docker compose down
docker compose up -d
```

The named volumes are intentionally retained by `docker compose down`, so the
WordPress installation and MySQL records remain available. To remove the
application and its stored data explicitly, run:

```bash
docker compose down -v
```
