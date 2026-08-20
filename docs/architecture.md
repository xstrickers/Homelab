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
