# Architecture

The Homelab is a self-hosted infrastructure designed to provide media
management and streaming services while keeping storage, compute and
networking responsibilities separated.

The infrastructure is built around:

- A small Ubuntu Server used for compute and service orchestration
- A NAS used for bulk media storage
- Docker and Docker Compose for service deployment
- A dedicated Docker network for the media stack
- Caddy as a reverse proxy and HTTPS entry point
- Gluetun to isolate qBittorrent traffic through a VPN
- NFS to connect the server to the NAS
- Intel hardware acceleration for Jellyfin transcoding

The infrastructure is composed of two main Docker Compose stacks:

1. The media stack, containing Jellyfin, Seerr, Sonarr, Radarr,
   Prowlarr, qBittorrent and Gluetun.
2. The Caddy stack, containing the reverse proxy and its custom
   DuckDNS-enabled Caddy build.
   
## Hardware & Infrastructure

### Server

The main server is a Fujitsu Esprimo Q556/2 running Ubuntu Server.

Its current hardware configuration is:

- Intel Core i5-7400
- 8 GB RAM
- 240 GB SSD
- Intel integrated GPU

The SSD is used for the operating system, Docker and local application
configuration. Media data is stored on the NAS rather than on the server's
local storage.

### NAS

The NAS is an OpenMediaVault-based storage server located on the local
network.

It provides approximately 22 TB of storage to the Homelab through NFS.

The server currently uses three NFS exports:

- `/export/permanent`
- `/export/downloads`
- `/export/temp`

These are mounted on the Ubuntu server as:

- `/mnt/permanent`
- `/mnt/downloads`
- `/mnt/temp`

The separation between compute and storage allows the server to run the
applications while the NAS provides the bulk storage required by the media
library.

## Docker

Docker is used to deploy and isolate the services running on the server.

The media services are managed through Docker Compose and share a dedicated
bridge network named `docker_jelly_network`.

The current Docker network uses the following configuration:

- Network: `docker_jelly_network`
- Driver: `bridge`
- Subnet: `172.18.0.0/16`
- Gateway: `172.18.0.1`

The following services are connected to this network:

- Jellyfin
- Seerr
- Sonarr
- Radarr
- Prowlarr
- Gluetun

qBittorrent uses a different networking configuration. It shares Gluetun's
network namespace through Docker's `network_mode: service:gluetun` mechanism.

This ensures that qBittorrent uses the same network path as Gluetun and can
therefore operate through the VPN tunnel.

### Docker Compose Stacks

The infrastructure is split into two Docker Compose projects.

#### Media stack

The main Compose project contains:

- Gluetun
- qBittorrent
- Sonarr
- Radarr
- Prowlarr
- Jellyfin
- Seerr

#### Caddy stack

Caddy is deployed separately using its own Docker Compose project.

This separation keeps the reverse proxy configuration independent from the
media services.

## Networking

The Homelab uses several network layers to separate external access,
container-to-container communication and VPN traffic.

### Host Network

The Ubuntu server runs Docker's default bridge interface:

- Interface: `docker0`
- Subnet: `172.17.0.0/16`
- Gateway: `172.17.0.1`

The Docker host is reachable from containers connected to this bridge
through the gateway address.

Caddy reaches Jellyfin and Seerr through ports published on the Docker host,
using the Docker host gateway at `172.17.0.1`.

For example:

```text
Caddy
  │
  ▼
172.17.0.1:8096
  │
  ▼
Docker port mapping
  │
  ▼
Jellyfin
```

### Media Network

The media stack uses a dedicated Docker bridge network:

- Network: `docker_jelly_network`
- Subnet: `172.18.0.0/16`
- Gateway: `172.18.0.1`

Services connected to this network can communicate using Docker's internal
DNS and service names rather than relying on container IP addresses.

For example:

```text
Sonarr → Radarr
Sonarr → Prowlarr
Seerr → Sonarr
Seerr → Radarr
```

### Caddy Network

Caddy runs as part of a separate Docker Compose project and therefore uses
its own Docker network.

Caddy does not directly join `docker_jelly_network`.

Instead, it reaches Jellyfin and Seerr through ports published on the
Ubuntu host:

```text
Caddy
  │
  ├── 172.17.0.1:8096 → Jellyfin
  │
  └── 172.17.0.1:5055 → Seerr
```

### VPN Network Path

qBittorrent uses Gluetun's network namespace:

```text
qBittorrent
     │
     │ network_mode: service:gluetun
     ▼
  Gluetun
     │
     ▼
    VPN
     │
     ▼
  Internet
```

## Storage & NFS

The NAS provides the bulk storage used by the media infrastructure.

The Ubuntu server accesses the NAS through NFS mounts over the local network.

The NAS exposes three separate NFS exports:

- `/export/permanent` for permanent media storage
- `/export/downloads` for downloaded content
- `/export/temp` for temporary or rotating media storage

These exports are mounted on the Ubuntu server as:

- `/mnt/permanent`
- `/mnt/downloads`
- `/mnt/temp`

The current mounts use NFS version 3 over TCP.

The mounts are configured in `/etc/fstab` so they are restored automatically
when the server starts.

The NFS mounts use the `_netdev` option to indicate that the filesystems
depend on network availability.

The storage architecture is therefore:

```text
NAS
  │
  │ NFS
  ▼
Ubuntu Server
  │
  ├── /mnt/permanent
  ├── /mnt/downloads
  └── /mnt/temp
  │
  ▼
Docker containers
```

The separation between permanent, downloaded and temporary data allows the
media services to use different storage paths depending on the stage of the
media lifecycle.
