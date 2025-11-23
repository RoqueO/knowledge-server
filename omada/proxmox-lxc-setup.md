# Proxmox LXC Container Setup for Omada Controller

This document details the creation and configuration of an LXC container for the TP-Link Omada Controller on your Proxmox server.

## Overview

We'll create an LXC container running Debian 12 or Ubuntu 22.04 LTS to host the Omada Controller. LXC containers are preferred over VMs for this use case because they use fewer resources while providing sufficient isolation and performance for network management.

## Prerequisites

- Proxmox server with LXC support enabled
- Access to Proxmox web interface
- Debian 12 or Ubuntu 22.04 LTS template downloaded (if not already available)
- Network configuration: LAN network (192.168.1.0/24)
- Internet connectivity for downloading Omada Controller software

## Step 1: Download Template

If you don't already have a Debian or Ubuntu template:

1. Log into Proxmox web interface
2. Go to **local (pve)** → **CT Templates**
3. Click **Templates** tab
4. Search for "debian-12" or "ubuntu-22.04"
5. Click **Download** next to your preferred template
6. Wait for download to complete

**Note**: Debian 12 is recommended for better compatibility with MongoDB and Omada Controller dependencies.

## Step 2: Create LXC Container

1. In Proxmox web interface, click **Create CT** (top right)
2. Fill in the following:

### General Tab
- **Hostname**: `omada-controller` (or your preferred name)
- **Password**: Set a secure root password
- **SSH Public Key**: (Optional) Add your SSH public key for key-based authentication

### Template Tab
- **Template**: Select `debian-12-standard_*` or `ubuntu-22.04-standard_*`
- **Storage**: Select your storage (usually `local`)

### Root Disk Tab
- **Storage**: Select your storage
- **Disk Size**: `16GB` (minimum, 32GB+ recommended for logs and database growth)
- **SSD Emulation**: Enable if using SSD storage

### CPU Tab
- **Cores**: `2` (minimum, 2-4 recommended)
- **CPU Units**: `1024` (default)

### Memory Tab
- **Memory**: `2048` MB (2GB minimum, 4GB recommended)
- **Swap**: `512` MB (optional, but recommended)

### Network Tab
- **IPv4/CIDR**: `192.168.1.14/24` (or next available IP on your LAN - check existing IPs)
- **Gateway (IPv4)**: `192.168.1.1` (your pfSense IP)
- **Bridge**: `vmbr0` (your main bridge)
- **VLAN Tag**: Leave empty (unless using VLANs)
- **Firewall**: Enable if you want Proxmox firewall rules

### DNS Tab
- **DNS Domain**: Leave empty or use your domain
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
   hostnamectl set-hostname omada-controller
   ```

6. **Verify network connectivity**:
   ```bash
   ping -c 4 8.8.8.8
   ping -c 4 192.168.1.1  # Should reach pfSense gateway
   ```

## Step 4: Install Required Dependencies

### Install Java Runtime Environment

Omada Controller requires Java 8 or Java 11. Install OpenJDK:

```bash
# For Debian 12 or Ubuntu 22.04
apt install -y openjdk-11-jdk-headless

# Verify installation
java -version
```

**Note**: If you need Java 8 specifically, you can install `openjdk-8-jdk-headless` instead.

### Install Additional Dependencies

```bash
apt install -y jsvc curl gnupg wget
```

### Install MongoDB

Omada Controller uses MongoDB for its database. Install MongoDB 4.4 or compatible version:

**For Debian 12:**
```bash
# Import MongoDB GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | gpg --dearmor | tee /usr/share/keyrings/mongodb-server-4.4.gpg > /dev/null

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-4.4.gpg ] https://repo.mongodb.org/apt/debian bullseye/mongodb-org/4.4 main" | tee /etc/apt/sources.list.d/mongodb-org-4.4.list

# Update package list
apt update

# Install MongoDB (pin version to avoid automatic upgrades)
apt install -y mongodb-org=4.4.29 mongodb-org-server=4.4.29 mongodb-org-shell=4.4.29 mongodb-org-mongos=4.4.29 mongodb-org-tools=4.4.29

# Prevent automatic MongoDB updates
echo "mongodb-org hold" | dpkg --set-selections
echo "mongodb-org-server hold" | dpkg --set-selections
echo "mongodb-org-shell hold" | dpkg --set-selections
echo "mongodb-org-mongos hold" | dpkg --set-selections
echo "mongodb-org-tools hold" | dpkg --set-selections

# Start and enable MongoDB
systemctl start mongod
systemctl enable mongod

# Verify MongoDB is running
systemctl status mongod
```

**For Ubuntu 22.04:**
```bash
# Import MongoDB GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | gpg --dearmor | tee /usr/share/keyrings/mongodb-server-4.4.gpg > /dev/null

# Add MongoDB repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-4.4.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/4.4 main" | tee /etc/apt/sources.list.d/mongodb-org-4.4.list

# Update package list
apt update

# Install MongoDB (pin version to avoid automatic upgrades)
apt install -y mongodb-org=4.4.29 mongodb-org-server=4.4.29 mongodb-org-shell=4.4.29 mongodb-org-mongos=4.4.29 mongodb-org-tools=4.4.29

# Prevent automatic MongoDB updates
echo "mongodb-org hold" | dpkg --set-selections
echo "mongodb-org-server hold" | dpkg --set-selections
echo "mongodb-org-shell hold" | dpkg --set-selections
echo "mongodb-org-mongos hold" | dpkg --set-selections
echo "mongodb-org-tools hold" | dpkg --set-selections

# Start and enable MongoDB
systemctl start mongod
systemctl enable mongod

# Verify MongoDB is running
systemctl status mongod
```

## Step 5: Download and Install Omada Controller

### Find Latest Version

1. Visit TP-Link's official Omada Controller download page:
   - https://www.tp-link.com/us/support/download/omada-software-controller/
   - Or search for "Omada Controller download" on TP-Link's website

2. Download the latest Linux x64 `.deb` package
3. Note the version number (e.g., `5.13.30`)

### Download and Install

**Option 1: Direct Download (if you have the URL)**

```bash
# Replace VERSION with the actual version number (e.g., 5.13.30)
# Replace ARCH with your architecture (usually x64)
VERSION="5.13.30"
ARCH="x64"

# Download Omada Controller
wget https://static.tp-link.com/upload/software/$(date +%Y)/$(date +%m)/$(date +%d)/Omada_SDN_Controller_v${VERSION}_linux_${ARCH}.deb

# If the above URL doesn't work, check TP-Link's download page for the exact URL
# Then download directly:
# wget <EXACT_URL_FROM_TP_LINK>

# Install the package
dpkg -i Omada_SDN_Controller_v${VERSION}_linux_${ARCH}.deb

# Fix any dependency issues
apt-get install -f -y
```

**Option 2: Manual Download**

1. Download the `.deb` file from TP-Link's website to your local machine
2. Transfer to the container using SCP or Proxmox file upload:
   ```bash
   # From your local machine
   scp Omada_SDN_Controller_v*.deb root@192.168.1.14:/tmp/
   ```
3. Install in the container:
   ```bash
   cd /tmp
   dpkg -i Omada_SDN_Controller_v*.deb
   apt-get install -f -y
   ```

### Verify Installation

```bash
# Check if Omada Controller service is running
systemctl status omada

# Check service status
systemctl is-enabled omada
```

## Step 6: Initial Configuration

1. **Access the Omada Controller web interface**:
   - Open a web browser
   - Navigate to `https://192.168.1.14:8043` (replace with your container IP)
   - Accept the self-signed certificate warning

2. **Initial Setup Wizard**:
   - Create an admin account (default is `admin`/`admin` - **change this immediately**)
   - Set timezone
   - Configure basic settings

3. **Verify Service Status**:
   ```bash
   # Check if service is running
   systemctl status omada
   
   # Check logs if needed
   journalctl -u omada -f
   ```

## Container Resource Recommendations

### Minimum Configuration
- **CPU**: 2 cores
- **RAM**: 2GB
- **Disk**: 16GB
- **Network**: Static IP on LAN

### Recommended Configuration
- **CPU**: 2-4 cores
- **RAM**: 4GB
- **Disk**: 32GB+ (for logs and database growth)
- **Network**: Static IP on LAN

### For Large Networks (50+ devices)
- **CPU**: 4 cores
- **RAM**: 8GB
- **Disk**: 64GB+
- **Network**: Static IP on LAN

## Network Configuration Details

### IP Assignment
- **Container IP**: `192.168.1.14` (example - use next available IP)
- **Gateway**: `192.168.1.1` (pfSense)
- **Subnet**: `192.168.1.0/24`
- **DNS**: `192.168.1.1` (pfSense) or `8.8.8.8` (Google DNS)

### Port Requirements

The Omada Controller uses the following ports:

| Port | Protocol | Purpose | External Access |
|------|----------|---------|----------------|
| 8043 | TCP | HTTPS web interface | Yes (if accessing remotely) |
| 8088 | TCP/UDP | Device discovery and HTTP redirect | Yes (UDP for discovery) |
| 27001 | TCP | MongoDB (internal) | No |
| 27002 | TCP | MongoDB (internal) | No |

**Note**: Ports 27001/27002 are only used internally by MongoDB and do not need external access or port forwarding.

### Firewall Configuration

If accessing from outside your network, configure pfSense to forward:
- **Port 8043** (HTTPS) to the container IP
- **Port 8088** (UDP) to the container IP (for device discovery)

For internal access only, no port forwarding is needed.

## Security Considerations

1. **Change Default Password**: Immediately change the default `admin`/`admin` credentials
2. **SSH Access**: Consider disabling password authentication and using SSH keys only
3. **Firewall**: Configure at pfSense level rather than container level
4. **Updates**: Keep the container and Omada Controller updated regularly
5. **Backups**: Set up Proxmox backups for the container and use Omada Controller's built-in backup feature
6. **HTTPS**: The controller uses self-signed certificates by default - consider using a reverse proxy with Let's Encrypt if accessing externally

## Troubleshooting

### Container Won't Start
- Check Proxmox logs: **Node** → **Syslog**
- Verify template is downloaded correctly
- Check resource availability (CPU, RAM, disk)

### Network Issues
- Verify IP configuration in container: `ip addr show`
- Check gateway is reachable: `ping 192.168.1.1`
- Verify DNS resolution: `nslookup google.com`
- Check if ports are accessible: `netstat -tlnp | grep -E '8043|8088'`

### Omada Controller Won't Start
- Check service status: `systemctl status omada`
- Check logs: `journalctl -u omada -n 50`
- Verify MongoDB is running: `systemctl status mongod`
- Check Java installation: `java -version`
- Verify disk space: `df -h`

### MongoDB Issues
- Check MongoDB status: `systemctl status mongod`
- Check MongoDB logs: `journalctl -u mongod -n 50`
- Verify MongoDB is listening: `netstat -tlnp | grep mongod`

### Web Interface Not Accessible
- Verify the service is running: `systemctl status omada`
- Check if port 8043 is listening: `netstat -tlnp | grep 8043`
- Try accessing via IP address: `https://<CONTAINER_IP>:8043`
- Check firewall rules on pfSense
- Verify network connectivity from your client

### Device Discovery Issues
- Ensure port 8088 (UDP) is not blocked
- Check if devices are on the same network segment
- Verify controller IP hasn't changed (devices may need re-adoption)

## Next Steps

After the container is set up and Omada Controller is installed:

1. Proceed to [docker-to-lxc-migration.md](docker-to-lxc-migration.md) to migrate settings from your Docker container
2. Configure firewall rules if accessing externally
3. Set up regular backups (both Proxmox container backups and Omada Controller backups)

## Notes

- The container IP address (e.g., `192.168.1.14`) will be used to access the Omada Controller web interface
- Keep the container updated: `apt update && apt upgrade -y`
- Consider setting up Proxmox backups for this container
- Use Omada Controller's built-in backup feature regularly (Settings > Maintenance > Backup & Restore)
- If the controller IP changes, devices may need to be re-adopted
- MongoDB version is pinned to prevent automatic upgrades that might break compatibility

