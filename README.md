# wordpress-docker-setup

A simple, reproducible Docker Compose setup that runs WordPress together with
a MariaDB database, using persistent volumes, an isolated network, and
environment-based configuration.

## Table of Contents

- [Description](#description)
- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)

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

| File                  | Purpose                                                   |
|-----------------------|-----------------------------------------------------------|
| `docker-compose.yaml` | Defines and configures the `wordpress` and `db` services  |
| `example.env`         | Template for required/optional environment variables      |
| `.gitignore`          | Excludes secrets and irrelevant files from git             |
| `README.md`           | This documentation                                         |

> [!NOTE]
> `.env` itself is **not** part of this repository. It is your local,
> git-ignored copy of `example.env` containing your actual secrets — see
> Quickstart step 2 below.

## Quickstart

### Prerequisites

In order to run this project you need the following:

- **Git** – to clone the repository
- An **OCI-compliant container engine with Compose support** (e.g. Docker
  Engine + Compose plugin)
- A **host or VM with a public IP** and **port `8080` open**
- **Terminal/SSH access** to that host

### Installation

To get started, follow these steps:

1. Clone the repository:
```bash
git clone https://github.com/FabianRitzmann/wordpress-docker-setup.git
cd wordpress-docker-setup
```

2. Create your local `.env` file from the provided template:
```bash
cp example.env .env
```

3. Open `.env` and set `MYSQL_PASSWORD` and `MYSQL_ROOT_PASSWORD` to strong,
   unique values.
   > [!TIP]
   > You can generate a random, secure password with:
```bash
openssl rand -base64 24
```

4. Start the stack:
```bash
docker compose up -d
```

5. Open `http://<your-host-ip>:8080` in your browser and complete the
   WordPress setup wizard. The admin username and password you enter there
   are your actual WordPress login credentials — they are independent of the
   database credentials in `.env`.

## Usage

In this section you can read about the project configuration in more detail.

---

### 1. Environment Variables

All configuration is controlled through environment variables. `example.env`
is the template committed to this repository and documents every available
variable with its default value. Your local `.env` (created in Quickstart
step 2, and excluded from git via `.gitignore`) is where you set your actual
values. `docker-compose.yaml` itself contains no default values — every
variable is read from `.env` at runtime.

| Variable               | Required | Default     | Description                                          |
|-------------------------|----------|-------------|-------------------------------------------------------|
| `MYSQL_DATABASE`       | No       | `wordpress` | Name of the WordPress database                        |
| `MYSQL_USER`           | No       | `wordpress` | Database user used internally by WordPress             |
| `MYSQL_PASSWORD`       | **Yes**  | –           | Password for the database user (set in `.env`)         |
| `MYSQL_ROOT_PASSWORD`  | **Yes**  | –           | Root password for the MariaDB instance (`.env`)        |
| `WORDPRESS_DB_HOST`    | No       | `db:3306`   | Hostname/port of the database service                   |
| `WORDPRESS_PORT`       | No       | `8080`      | Host port on which WordPress is exposed                  |

> [!NOTE]
> `MYSQL_USER` / `MYSQL_PASSWORD` are only used for the internal connection
> between the `wordpress` and `db` containers. They are **not** the same as
> your WordPress admin login, which you set separately in the setup wizard
> on first launch.

To change any of the non-critical values (e.g. use a different database name
or expose WordPress on a different host port), edit your local `.env` file,
for example:

```env
MYSQL_PASSWORD=sicheresPasswort123
MYSQL_ROOT_PASSWORD=sicheresPasswort1234567
```

Sensitive values (`MYSQL_PASSWORD`, `MYSQL_ROOT_PASSWORD`) must always be set
in your local `.env` and are never hardcoded in `docker-compose.yaml`.

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