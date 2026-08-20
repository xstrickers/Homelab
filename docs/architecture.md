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
