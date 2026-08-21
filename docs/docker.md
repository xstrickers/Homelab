# Docker

Docker is used as the main service orchestration platform for the Homelab.

The infrastructure is split into two independent Docker Compose projects:

- The media stack
- The Caddy reverse proxy stack

This separation keeps the media services and reverse proxy configuration
independent from each other.

## Docker Compose Projects

### Media Stack

The media stack is defined in:

`~/docker/docker-compose.yml`

It contains the following services:

- Gluetun
- qBittorrent
- Sonarr
- Radarr
- Prowlarr
- Jellyfin
- Seerr

The services are connected to the dedicated `docker_jelly_network` Docker
network, except for qBittorrent, which shares Gluetun's network namespace.

### Caddy Stack

The Caddy stack is defined separately in:

`~/docker/caddy/docker-compose.yml`

It contains the Caddy reverse proxy.

Caddy uses its own Docker network and does not directly join the media
network.

## Directory Structure

The Docker configuration is stored under:

`~/docker`

The current structure is:

```text
~/docker/
│
├── docker-compose.yml
├── .env
│
├── gluetun/
├── qbittorrent/
├── radarr/
├── sonarr/
├── prowlarr/
├── jellyfin/
├── jellyseerr/
│
└── caddy/
    ├── docker-compose.yml
    ├── Dockerfile
    └── Caddyfile
```

The individual service directories contain persistent configuration data
for their respective containers.

The `.env` file contains environment variables used by the media stack.
Sensitive values must not be committed to the repository.

## Media Stack

The main Docker Compose project defines a dedicated bridge network:

`docker_jelly_network`

The network allows the media services to communicate with each other using
Docker's internal DNS and service names.

The services connected to this network are:

- Jellyfin
- Seerr
- Sonarr
- Radarr
- Prowlarr
- Gluetun

qBittorrent is handled differently because it uses Gluetun's network
namespace.

## qBittorrent and Gluetun

qBittorrent uses:

```text
network_mode: service:gluetun
```

This means that qBittorrent does not create its own network namespace.

Instead, it uses the same network namespace as Gluetun:

```
qBittorrent
     │
     │ network_mode: service:gluetun
     ▼
  Gluetun
     │
     ▼
    VPN
```

This design ensures that qBittorrent's network traffic follows Gluetun's
VPN connection.

Gluetun is responsible for establishing and maintaining the VPN connection,
while qBittorrent uses that connection for its network traffic.

## Persistent Storage

The containers use Docker bind mounts to store their configuration outside
of the containers themselves.

Examples include:

```text
./jellyfin:/config
./sonarr:/config
./radarr:/config
./prowlarr:/config
./qbittorrent:/config
```

Media-related directories from the NAS are also mounted into the relevant
containers.

For example, Sonarr and Radarr use:

```text
/mnt/permanent:/data/permanent
/mnt/temp:/data/rotating
/mnt/downloads:/downloads
```

Jellyfin uses:

```text
/mnt/permanent:/data/permanent
/mnt/temp:/data/rotating
```

This allows the containers to access the NAS using consistent paths.

## Service Ports

The main services expose the following ports on the Ubuntu host:

| Service | Port |
|---|---:|
| Jellyfin | 8096 |
| Seerr | 5055 |
| Sonarr | 8989 |
| Radarr | 7878 |
| Prowlarr | 9696 |
| qBittorrent | 8080 |

The Caddy stack uses a separate external HTTPS port:

```text
34443 → Caddy :443
```

The exact network routing and purpose of these ports are documented in
`networking.md`.

## Docker Images

The current services use the following images:

| Service | Image |
|---|---|
| Gluetun | `qmcgaw/gluetun` |
| qBittorrent | `linuxserver/qbittorrent` |
| Sonarr | `linuxserver/sonarr` |
| Radarr | `linuxserver/radarr` |
| Prowlarr | `linuxserver/prowlarr` |
| Jellyfin | `jellyfin/jellyfin` |
| Seerr | `ghcr.io/seerr-team/seerr` |
| Caddy | Custom-built image based on `caddy:2` |

The Caddy image is custom-built to include the DuckDNS DNS provider plugin.

## Environment Variables

The media stack uses an environment file:

```text
~/docker/.env
```

The file contains configuration values used by the containers.

Sensitive information such as:

- passwords
- API keys
- VPN credentials
- authentication tokens
- DuckDNS tokens

must never be committed to the repository.

When configuration examples are included in this documentation, sensitive
values are represented as:

```text
<SECRET>
```
