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
