# Omada Controller Self-Hosting Documentation

This documentation provides a complete guide for migrating TP-Link Omada Controller from a Docker container to a Proxmox LXC container, including container setup, installation, and settings migration.

## Overview

The **TP-Link Omada Controller** is a centralized management platform for Omada access points, switches, and routers. This setup migrates the controller from a Docker container (on a different machine) to a lightweight LXC container on your Proxmox server.

**Key Features:**
- Centralized network device management
- Web-based administration interface
- Device adoption and configuration
- Network monitoring and statistics
- Firmware management
- Guest network management

## Architecture

- **Container Type**: LXC (lightweight, low resource usage)
- **OS**: Debian 12 or Ubuntu 22.04 LTS
- **Controller Software**: TP-Link Omada SDN Controller
- **Network**: LAN (192.168.1.0/24)
- **Database**: MongoDB (included with controller)

## Quick Reference

### Omada Controller Details
- **Web Interface**: `https://<LXC_IP>:8043`
- **Default Admin**: `admin` / `admin` (change on first login)
- **Device Discovery**: Port 8088 (UDP)
- **HTTPS Port**: 8043 (TCP)
- **HTTP Port**: 8088 (TCP, redirects to HTTPS)

### Required Ports
- **Port 8043**: HTTPS web interface (primary access)
- **Port 8088**: Device discovery (UDP) and HTTP redirect (TCP)
- **Port 27001/27002**: MongoDB (internal, no external access needed)

### Network Configuration
- **IP Assignment**: Static IP on LAN network (192.168.1.0/24)
- **Gateway**: 192.168.1.1 (pfSense)
- **DNS**: 192.168.1.1 (pfSense) or 8.8.8.8

## Documentation Structure

1. **[proxmox-lxc-setup.md](proxmox-lxc-setup.md)** - LXC container creation and Omada Controller installation
2. **[docker-to-lxc-migration.md](docker-to-lxc-migration.md)** - Backup, restore, and migration procedures from Docker to LXC

## Implementation Order

1. **Backup Current Docker Instance** - Export settings from existing Docker container
2. **Proxmox LXC Setup** - Create Debian/Ubuntu LXC container
3. **Omada Controller Installation** - Install controller software in LXC
4. **Settings Migration** - Restore backup to new LXC instance
5. **Device Re-adoption** - Re-adopt devices if needed (may require factory reset)
6. **Verification** - Verify all devices and settings are working correctly

## Important Notes

- **Version Compatibility**: Ensure both Docker and LXC instances use the same Omada Controller version for smooth migration
- **Device Adoption**: If the controller IP changes significantly, devices may need to be re-adopted
- **Backup First**: Always create a backup before migration
- **Network Access**: Devices must be able to reach the controller on the new IP address
- **Downtime**: Plan for brief downtime during migration

## Prerequisites

- Proxmox server with LXC support
- Existing Omada Controller running in Docker (on different machine)
- Access to Omada Controller web interface
- Network access to Proxmox server
- Basic familiarity with command line and web interfaces
- Static IP available on LAN network (192.168.1.0/24)

## Resource Requirements

### Minimum Configuration
- **CPU**: 2 cores
- **RAM**: 2GB
- **Disk**: 8GB
- **Network**: Static IP on LAN

### Recommended Configuration
- **CPU**: 2-4 cores
- **RAM**: 4GB
- **Disk**: 16GB+ (for logs and database growth)
- **Network**: Static IP on LAN

