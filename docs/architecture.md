# Architecture

The Homelab is a self-hosted infrastructure designed to provide media
management and streaming services while keeping storage, compute and
networking responsibilities separated.

The infrastructure is built around:

- A small Ubuntu Server used for compute and service orchestration
- A NAS used for bulk media storage
- Docker and Docker Compose for service deployment
- Caddy as a reverse proxy and HTTPS entry point
- Gluetun to route qBittorrent traffic through a VPN
- NFS to connect the server to the NAS
- Intel hardware acceleration for Jellyfin transcoding

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

The NAS provides three main storage areas:

- Permanent media storage
- Download storage
- Temporary or rotating media storage

The separation between compute and storage allows the server to run the
applications while the NAS provides the bulk storage required by the media
library.

## Docker Infrastructure

The Homelab uses two separate Docker Compose projects.

### Media Stack

The main media stack contains:

- Jellyfin
- Seerr
- Sonarr
- Radarr
- Prowlarr
- qBittorrent
- Gluetun

These services handle media management, downloading and streaming.

### Caddy Stack

Caddy is deployed separately using its own Docker Compose project.

It acts as the reverse proxy and HTTPS entry point for externally accessible
services.

Keeping Caddy in a separate stack makes the reverse proxy independent from
the media services.

## Network Architecture

The infrastructure uses separate network layers for container
communication, external access and VPN traffic.

The media services communicate through a dedicated Docker network.

Caddy runs independently and reaches the required services through ports
published on the Ubuntu host.

qBittorrent shares Gluetun's network namespace so that its network traffic
is routed through the VPN.

A detailed description of the Docker networks, ports and traffic flows is
available in [Networking](networking.md).

## Storage Architecture

The Ubuntu server accesses the NAS through NFS.

The NAS provides separate storage areas for permanent media, downloads and
temporary data.

These storage areas are mounted on the Ubuntu server and then exposed to
the relevant Docker containers.

A detailed description of the NAS, NFS configuration and storage paths is
available in [Storage](storage.md).

## Infrastructure Overview

The overall architecture can be summarized as:

```text
                         Internet
                            │
                            ▼
                          Caddy
                            │
                    ┌───────┴───────┐
                    ▼               ▼
                 Jellyfin         Seerr
                    │
                    │
              Docker Media Stack
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Sonarr       Radarr     Prowlarr
        │
        ▼
    qBittorrent
        │
        ▼
     Gluetun
        │
        ▼
       VPN

        Ubuntu Server
             │
             │ NFS
             ▼
            NAS
```
