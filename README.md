# opencloud-compose-project

[opencloud-eu/opencloud-compose](https://github.com/opencloud-eu/opencloud-compose) describes the official Docker Compose deployment for OpenCloud, but requires cloning that repo directly — which is not always practical. This project provides a self-contained Docker Compose stack deploying OpenCloud with Radicale (CalDAV/CardDAV) and Collabora (office document editing), using SWAG as a reverse proxy.

Created with Claude Code using the official repo as context, manually edited and running in production. So it's basically put together the scathered compose and config files from the OpenCloud repo.

## Services

| Service | Role | Subdomain |
|---|---|---|
| OpenCloud | Main file sync and collaboration platform | `OC_DOMAIN` |
| Collabora | In-browser office document editor | `COLLABORA_DOMAIN` |
| WOPI server | Collabora-to-OpenCloud bridge | `WOPISERVER_DOMAIN` |
| Radicale | CalDAV/CardDAV server for calendars and contacts | — (internal) |

## Prerequisites

- Docker with the Compose plugin (`docker compose`)
- A running [SWAG](https://docs.linuxserver.io/general/swag/) instance
- Subdomains configured for `OC_DOMAIN`, `COLLABORA_DOMAIN`, and `WOPISERVER_DOMAIN`

## Installation

1. Clone this repo.

2. Copy `.env.example` to `.env` and fill in your values:
   ```
   cp .env.example .env
   ```
   At minimum, set `OC_DOMAIN`, `COLLABORA_DOMAIN`, `WOPISERVER_DOMAIN`, and `INITIAL_ADMIN_PASSWORD`. Set the storage paths and `OC_CONTAINER_UID_GID` to match your host user (`id $(whoami)`).

3. Create the host directories referenced in `.env`:
   ```
   mkdir -p /path/to/opencloud/config
   mkdir -p /path/to/opencloud/data
   mkdir -p /path/to/opencloud-radicale/config   # directory only — the file goes inside
   mkdir -p /path/to/appdata/opencloud-radicale/data
   ```

4. Copy the OpenCloud config files into `OC_CONFIG_DIR`:
   ```
   cp config/opencloud/banned-password-list.txt /path/to/opencloud/config/
   cp config/opencloud/csp.yaml                 /path/to/opencloud/config/
   cp config/opencloud/proxy.yaml               /path/to/opencloud/config/
   ```

5. Copy the Radicale config file into `RADICALE_CONFIG_DIR`'s parent directory:
   ```
   cp config/radicale/config /path/to/opencloud-radicale/config/config
   ```
   Note: `RADICALE_CONFIG_DIR` points to the file itself (`.../config/config`), not to a directory. The outer `config` is the directory; the inner `config` is the file. Radicale will error if it finds a directory where it expects a file.

6. Set ownership and permissions of created folders so the container user (from `OC_CONTAINER_UID_GID` in `.env`) can read and write the data.

7. Copy the SWAG nginx configs from `swag/` to your SWAG proxy-confs directory (e.g. `/path/to/swag/nginx/proxy-confs`):
   ```
   cp swag/collabora.subdomain.conf  /path/to/swag/nginx/proxy-confs/
   cp swag/opencloud.subdomain.conf  /path/to/swag/nginx/proxy-confs/
   cp swag/wopi.subdomain.conf       /path/to/swag/nginx/proxy-confs/
   ```

8. Start the stack:
   ```
   docker compose up -d
   ```

9. Log in at `https://<OC_DOMAIN>` with username `admin` and the password set in `INITIAL_ADMIN_PASSWORD`.
