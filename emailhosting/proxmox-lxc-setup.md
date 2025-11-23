# Proxmox LXC Container Setup

This document details the creation and configuration of an LXC container for the Mail-in-a-Box email server on your Proxmox server.

## Overview

We'll create an LXC container running Ubuntu 22.04 LTS, which is required for Mail-in-a-Box. LXC containers are preferred over VMs for this use case because they use fewer resources while providing sufficient isolation.

## Prerequisites

- Proxmox server with LXC support enabled
- Access to Proxmox web interface
- Ubuntu 22.04 LTS template downloaded (if not already available)
- Network configuration: LAN network (192.168.1.0/24)

## Step 1: Download Ubuntu 22.04 LTS Template

If you don't already have the Ubuntu 22.04 template:

1. Log into Proxmox web interface
2. Go to **local (pve)** → **CT Templates**
3. Click **Templates** tab
4. Search for "ubuntu-22.04"
5. Click **Download** next to `ubuntu-22.04-standard_*`
6. Wait for download to complete

## Step 2: Create LXC Container

1. In Proxmox web interface, click **Create CT** (top right)
2. Fill in the following:

### General Tab
- **Hostname**: `mail-server` (or your preferred name)
- **Password**: Set a secure root password
- **SSH Public Key**: (Optional) Add your SSH public key for key-based authentication

### Template Tab
- **Template**: Select `ubuntu-22.04-standard_*`
- **Storage**: Select your storage (usually `local`)

### Root Disk Tab
- **Storage**: Select your storage
- **Disk Size**: `20GB` (minimum, 30GB+ recommended for email storage)
- **SSD Emulation**: Enable if using SSD storage

### CPU Tab
- **Cores**: `2` (minimum, 4 recommended for better performance)
- **CPU Units**: `1024` (default)

### Memory Tab
- **Memory**: `2048` MB (2GB minimum, 4GB recommended)
- **Swap**: `512` MB (optional, but recommended)

### Network Tab
- **IPv4/CIDR**: `192.168.1.13/24` (or next available IP on your LAN)
- **Gateway (IPv4)**: `192.168.1.1` (your pfSense IP)
- **Bridge**: `vmbr0` (your main bridge)
- **VLAN Tag**: Leave empty (unless using VLANs)
- **Firewall**: Enable if you want Proxmox firewall rules

### DNS Tab
- **DNS Domain**: `obusan.me` (optional)
- **DNS Server**: `192.168.1.1` (your pfSense/DNS server)

### Confirm Tab
- Review all settings
- Click **Finish**

## Step 3: Start Container and Initial Configuration

1. **Start the container**: Right-click container → **Start**

2. **Open console**: Right-click container → **Console** (or use SSH)

3. **Update system**:
   ```bash
   apt update && apt upgrade -y
   ```

4. **Install basic tools** (optional but recommended):
   ```bash
   apt install -y curl wget nano htop
   ```

5. **Configure hostname** (if not set correctly):
   ```bash
   hostnamectl set-hostname mail-server
   ```

6. **Verify network connectivity**:
   ```bash
   ping -c 4 8.8.8.8
   ping -c 4 mail.obusan.me  # Should resolve to your public IP
   ```

## Step 4: Configure Container for Mail-in-a-Box

Mail-in-a-Box requires certain system configurations. The installation script will handle most of this, but we need to ensure the container is ready.

### Check System Requirements

```bash
# Verify Ubuntu version (must be 22.04)
lsb_release -a

# Check available disk space (should be at least 10GB free)
df -h

# Check memory (should be at least 2GB)
free -h
```

### Configure Firewall (if using Proxmox firewall)

If you enabled the firewall in Proxmox, you'll need to allow the required ports. However, it's recommended to handle firewall rules at the pfSense level instead.

### Set Static IP (if not already set)

The IP should already be static from the container creation, but verify:

```bash
ip addr show
```

The IP should match what you configured (e.g., `192.168.1.13`).

## Step 5: Prepare for Mail-in-a-Box Installation

1. **Ensure DNS resolution works**:
   ```bash
   # Test DNS resolution
   nslookup mail.obusan.me
   dig mail.obusan.me
   ```

2. **Verify time synchronization**:
   ```bash
   timedatectl status
   ```
   If not synchronized, enable it:
   ```bash
   timedatectl set-ntp true
   ```

3. **Note the container's IP address** for firewall configuration:
   ```bash
   hostname -I
   ```
   Write this down - you'll need it for pfSense port forwarding.

## Container Resource Recommendations

### Minimum Configuration
- **CPU**: 2 cores
- **RAM**: 2GB
- **Disk**: 20GB
- **Network**: Static IP on LAN

### Recommended Configuration
- **CPU**: 4 cores
- **RAM**: 4GB
- **Disk**: 50GB+ (for email storage)
- **Network**: Static IP on LAN

### For Multiple Users/High Volume
- **CPU**: 4-8 cores
- **RAM**: 8GB+
- **Disk**: 100GB+ (with expansion capability)
- **Network**: Static IP on LAN

## Network Configuration Details

### IP Assignment
- **Container IP**: `192.168.1.13` (example - use next available IP)
- **Gateway**: `192.168.1.1` (pfSense)
- **Subnet**: `192.168.1.0/24`
- **DNS**: `192.168.1.1` (pfSense) or `8.8.8.8` (Google DNS)

### Port Requirements
The following ports will need to be forwarded from pfSense to this container:
- **25**: SMTP
- **587**: SMTP submission
- **993**: IMAPS
- **995**: POP3S
- **80**: HTTP (for Let's Encrypt)
- **443**: HTTPS (webmail and admin)

## Security Considerations

1. **SSH Access**: Consider disabling password authentication and using SSH keys only
2. **Firewall**: Configure at pfSense level rather than container level
3. **Updates**: Keep the container updated regularly
4. **Backups**: Set up Proxmox backups for the container

## Troubleshooting

### Container Won't Start
- Check Proxmox logs: **Node** → **Syslog**
- Verify template is downloaded correctly
- Check resource availability (CPU, RAM, disk)

### Network Issues
- Verify IP configuration in container: `ip addr show`
- Check gateway is reachable: `ping 192.168.1.1`
- Verify DNS resolution: `nslookup google.com`

### DNS Resolution Problems
- Check `/etc/resolv.conf` contains correct DNS server
- Test with: `dig @8.8.8.8 mail.obusan.me`
- Verify DNS records are set at Hostinger

### Resource Constraints
- Monitor resource usage: `htop`
- Check disk space: `df -h`
- Increase resources in Proxmox if needed

## Next Steps

After the container is set up and running:
1. Proceed to [mail-in-a-box-installation.md](mail-in-a-box-installation.md) for Mail-in-a-Box installation
2. Configure firewall rules as documented in [firewall-configuration.md](firewall-configuration.md)

## Notes

- The container IP address (e.g., `192.168.1.13`) will be used in pfSense port forwarding rules
- Mail-in-a-Box will handle most system configuration automatically during installation
- Keep the container updated: `apt update && apt upgrade -y` (before Mail-in-a-Box installation)
- Consider setting up Proxmox backups for this container after Mail-in-a-Box is installed

