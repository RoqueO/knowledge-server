# Proxmox Homelab Server Documentation

This documentation provides reference information for setting up and managing a Proxmox homelab server with multiple LXC containers and VMs for media services.

## Server Architecture

### LXC Containers
- **LXC 1 - Nginx**: Reverse proxy and web server
- **LXC 2 - Jellyfin**: Media server
- **LXC 3 - Download Clients**: qBittorrent and SABnzbd download clients (VLAN 20)

### Virtual Machines
- **VM1 - pfSense**: Router and firewall
- **VM2 - ArrStack**: Docker-based media automation stack

### ArrStack Services (VM2)
The ArrStack VM runs the following Docker containers:
- **Radarr**: Movie collection management
- **Sonarr**: TV series collection management
- **Prowlarr**: Indexer manager

**Note**: qBittorrent and SABnzbd have been moved to a dedicated LXC container on VLAN 20 for better isolation and security.

## Documentation Structure

- **[networking.md](networking.md)** - Network architecture, IP assignments, Docker networks, and firewall configuration
- **[lxc-nginx.md](lxc-nginx.md)** - Nginx LXC container setup and configuration
- **[lxc-jellyfin.md](lxc-jellyfin.md)** - Jellyfin LXC container setup and configuration
- **[lxc-download-clients.md](lxc-download-clients.md)** - Download clients LXC container setup (qBittorrent & SABnzbd)
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
- **Download Clients LXC**: 192.168.2.20

### Service Ports

**LAN Services:**
- **Nginx**: 80, 443
- **Jellyfin**: 8096
- **Radarr**: 7878 (http://192.168.1.20:7878)
- **Sonarr**: 8989 (http://192.168.1.20:8989)
- **Prowlarr**: 9696 (http://192.168.1.20:9696)

**VLAN 20 Services:**
- **qBittorrent**: 8080 (http://192.168.2.20:8080)
- **SABnzbd**: 8081 (http://192.168.2.20:8081)

## Implementation Order

1. Set up pfSense VM (VM1) - Configure networking and firewall
2. Create and configure Nginx LXC (LXC 1) - Reverse proxy setup
3. Create and configure Jellyfin LXC (LXC 2) - Media server setup
4. Create and configure ArrStack VM (VM2) - Docker environment
5. Deploy Docker Compose stack - ArrStack services (Radarr, Sonarr, Prowlarr)
6. Create and configure Download Clients LXC (LXC 3) - qBittorrent and SABnzbd on VLAN 20
7. Configure pfSense firewall rules - Allow ARR stack to access download clients
8. Configure ARR stack integration - Add download clients to Radarr and Sonarr
9. Configure Nginx reverse proxy - Point to all services
10. Set up monitoring and backups - Operational procedures

## Notes

- This is reference documentation for a homelab environment
- All configurations assume Debian-based LXC containers
- Download services (qBittorrent, SABnzbd) are hosted in a single LXC container on VLAN 20 for security and traffic separation
- The ARR stack (Radarr, Sonarr, Prowlarr) runs on the ArrStack VM on the LAN network
- pfSense handles routing and firewall rules for both networks, including inter-VLAN routing
- Proxmox uses VLAN-aware bridges (vmbr0) with VLAN tagging for VLAN 20 support
- Download clients communicate with the ARR stack via pfSense inter-VLAN routing
- Bind mounts are used to share storage between the Proxmox host and the download clients container

