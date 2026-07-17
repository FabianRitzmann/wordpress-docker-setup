# wordpress-docker-setup
 
A simple, reproducible Docker Compose setup that runs WordPress together with
a MariaDB database, using persistent volumes, an isolated network, and
environment-based configuration.
 
## Table of Contents
 
- [Description](#description)
- [Quickstart](#quickstart)
- [Usage](#usage)
- [Testing](#testing)
- [Security Notes](#security-notes)
## Description
 
This repository contains everything needed to run a self-contained WordPress
instance using Docker Compose. It consists of two services:
 
- **`wordpress`** the WordPress application server, based on the official
  `wordpress` image.
- **`db`** a MariaDB database that stores all WordPress content, based on
  the official `mariadb` image.
Both services run in a shared Docker network (`wordpress_network`) so that
WordPress can reach the database by its service name (`db`). Database content
is persisted through a named Docker volume (`db_data`), so data survives
container restarts and recreations. The WordPress installation itself
(uploads, plugins, themes) is persisted through a second volume
(`wordpress_data`).
 
### Repository contents
 
| File                   | Purpose                                                   |
|------------------------|-----------------------------------------------------------|
| `docker-compose.yaml`  | Defines and configures the `wordpress` and `db` services  |
| `.env`                 | Template for required/optional environment variables      |
| `.gitignore`           | Excludes secrets and irrelevant files from git            |
| `README.md`            | This documentation                                        |
 
## Quickstart
 
### Installation
 
To get started, follow these steps:
 
1. Clone the repository:
```bash
   git clone https://github.com/FabianRitzmann/wordpress-docker-setup.git
   cd wordpress-docker-setup
```

2. Open `.env` and set `MYSQL_PASSWORD` and `MYSQL_ROOT_PASSWORD` to strong,
   unique values.
   > [!TIP]
   > You can generate a random, secure password with:
```bash
penssl rand -base64 24
```
 
3. Start the stack:
```bash
docker compose up -d
```

4. Open `http://<your-host-ip>:8080` in your browser and complete the
   WordPress setup wizard. The admin username and password you enter there
   are your actual WordPress login credentials — they are independent of the
   database credentials in `.env`.
## Usage
 
In this section you can read about the project configuration in more detail.
 
---
 
### 1. Environment Variables
 
All configuration is controlled through environment variables, either with
sensible defaults in `docker-compose.yaml` or via the `.env` file for
sensitive values. Variable naming follows the `UPPER_CASE_WITH_UNDERSCORE`
convention, and all references use the `${VARIABLE_NAME}` notation.
 
| Variable              | Required | Default        | Description                                     |
|------------------------|----------|----------------|---------------------------------------------------|
| `MYSQL_DATABASE`      | No       | `wordpress`    | Name of the WordPress database                    |
| `MYSQL_USER`          | No       | `wordpress`    | Database user used internally by WordPress         |
| `MYSQL_PASSWORD`      | **Yes**  | –              | Password for the database user (set in `.env`)     |
| `MYSQL_ROOT_PASSWORD` | **Yes**  | –              | Root password for the MariaDB instance (`.env`)    |
| `WORDPRESS_DB_HOST`   | No       | `db:3306`      | Hostname/port of the database service               |
| `WORDPRESS_PORT`      | No       | `8080`         | Host port on which WordPress is exposed              |
 
> [!NOTE]
> `MYSQL_USER` / `MYSQL_PASSWORD` are only used for the internal connection
> between the `wordpress` and `db` containers. They are **not** the same as
> your WordPress admin login, which you set separately in the setup wizard
> on first launch.
 
To change any of the non-critical values (e.g. use a different database name
or expose WordPress on a different host port), add the variable to your
`.env` file, for example:
 
```env
WORDPRESS_PORT=9090
MYSQL_DATABASE=my_custom_db
```
 
Sensitive values (`MYSQL_PASSWORD`, `MYSQL_ROOT_PASSWORD`) must always be set
in `.env` and are never hardcoded in `docker-compose.yaml`.
 
---
 
### 2. Networking
 
Both services are attached to a single bridge network, `wordpress_network`.
This allows the `wordpress` container to reach the database simply by using
the service name `db` as the hostname (see `WORDPRESS_DB_HOST`) — no manual
IP configuration required.
 
---
 
### 3. Persistence
 
Both services use named Docker volumes:
 
- `db_data` → mounted at `/var/lib/mysql` inside the `db` container
- `wordpress_data` → mounted at `/var/www/html` inside the `wordpress`
  container
This means running `docker compose down` (without `-v`) and then
`docker compose up -d` again will **not** delete any data — posts, users,
plugins, and themes remain intact. Only `docker compose down -v` removes the
volumes and resets the installation completely.
 
---
 
### 4. Restart Behavior
 
Both services are configured with `restart: unless-stopped`, so Docker
automatically restarts a container if it crashes or the host reboots, unless
it was explicitly stopped by the user.
 
---
 
### 5. Stopping the Stack
 
```bash
docker compose down
```
 