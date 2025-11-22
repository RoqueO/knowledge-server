# Proxmox Homelab Server Documentation

This documentation provides reference information for setting up and managing a Proxmox homelab server with multiple LXC containers and VMs for media services.

## Server Architecture

### LXC Containers
- **LXC 1 - Nginx**: Reverse proxy and web server
- **LXC 2 - Jellyfin**: Media server

### Virtual Machines
- **VM1 - pfSense**: Router and firewall
- **VM2 - ArrStack**: Docker-based media automation stack

### ArrStack Services (VM2)
The ArrStack VM runs the following Docker containers:
- **Radarr**: Movie collection management
- **Sonarr**: TV series collection management
- **Prowlarr**: Indexer manager
- **qBittorrent**: BitTorrent client
- **SABnzbd**: Usenet client

## Documentation Structure

- **[networking.md](networking.md)** - Network architecture, IP assignments, Docker networks, and firewall configuration
- **[lxc-nginx.md](lxc-nginx.md)** - Nginx LXC container setup and configuration
- **[lxc-jellyfin.md](lxc-jellyfin.md)** - Jellyfin LXC container setup and configuration
- **[vm-pfsense.md](vm-pfsense.md)** - pfSense VM installation and configuration
- **[vm-arrstack.md](vm-arrstack.md)** - ArrStack VM setup with Docker and Docker Compose
- **[docker-compose.yml](docker-compose.yml)** - Docker Compose configuration for all ArrStack services
- **[monitoring-maintenance.md](monitoring-maintenance.md)** - Backup, monitoring, and maintenance procedures

## Quick Reference

### Network Configuration
- **LAN Network**: 192.168.1.0/24
- **VLAN 20 Network**: 192.168.2.0/24 (Download services isolation)
- **pfSense WAN**: External interface (configured during setup)
- **pfSense LAN**: 192.168.1.1
- **pfSense VLAN 20**: 192.168.2.1

### Service IPs

**LAN Network (192.168.1.0/24):**
- **pfSense**: 192.168.1.1
- **Proxmox Host**: 192.168.1.10
- **Nginx LXC**: 192.168.1.11
- **Jellyfin LXC**: 192.168.1.12
- **ArrStack VM (Interface 1)**: 192.168.1.20

**VLAN 20 Network (192.168.2.0/24):**
- **pfSense (VLAN 20)**: 192.168.2.1
- **ArrStack VM (Interface 2)**: 192.168.2.30

### Service Ports

**LAN Services:**
- **Nginx**: 80, 443
- **Jellyfin**: 8096
- **Radarr**: 7878 (http://192.168.1.20:7878)
- **Sonarr**: 8989 (http://192.168.1.20:8989)
- **Prowlarr**: 9696 (http://192.168.1.20:9696)

**VLAN 20 Services:**
- **qBittorrent**: 8080 (http://192.168.2.30:8080)
- **SABnzbd**: 8081 (http://192.168.2.30:8081)

## Implementation Order

1. Set up pfSense VM (VM1) - Configure networking and firewall
2. Create and configure Nginx LXC (LXC 1) - Reverse proxy setup
3. Create and configure Jellyfin LXC (LXC 2) - Media server setup
4. Create and configure ArrStack VM (VM2) - Docker environment
5. Deploy Docker Compose stack - All ArrStack services
6. Configure Nginx reverse proxy - Point to all services
7. Set up monitoring and backups - Operational procedures

## Notes

- This is reference documentation for a homelab environment
- All configurations assume Debian-based LXC containers
- Docker services are segmented into two networks:
  - **arrstack_lan**: Radarr, Sonarr, Prowlarr (LAN network)
  - **arrstack_vlan20**: qBittorrent, SABnzbd (VLAN 20 network)
- Download services (qBittorrent, SABnzbd) are isolated on VLAN 20 for security and traffic separation
- pfSense handles routing and firewall rules for both networks, including inter-VLAN routing
- Proxmox uses VLAN-aware bridges (vmbr0.20) for VLAN 20 support

