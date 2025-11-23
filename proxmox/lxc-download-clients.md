# Proxmox LXC Container Setup for Download Clients

This document details the creation and configuration of an LXC container for qBittorrent and SABnzbd download clients on your Proxmox server. The container is isolated on VLAN 20 for security and traffic separation.

## Overview

We'll create a single LXC container running Debian 12 to host both qBittorrent (BitTorrent client) and SABnzbd (Usenet client). The container is connected to VLAN 20 (192.168.2.0/24) and communicates with the ARR stack (Radarr, Sonarr) on the LAN network via pfSense routing.

## Prerequisites

- Proxmox server with LXC support enabled
- Access to Proxmox web interface
- Debian 12 template downloaded (if not already available)
- VLAN 20 configured on pfSense (192.168.2.0/24)
- VLAN-aware bridge (vmbr0) configured on Proxmox
- Host storage paths for downloads (incomplete and completed folders)

## Step 1: Download Template

If you don't already have a Debian 12 template:

1. Log into Proxmox web interface
2. Go to **local (pve)** → **CT Templates**
3. Click **Templates** tab
4. Search for "debian-12"
5. Click **Download** next to `debian-12-standard_*`
6. Wait for download to complete

## Step 2: Create LXC Container

1. In Proxmox web interface, click **Create CT** (top right)
2. Fill in the following:

### General Tab
- **Hostname**: `download-clients` (or your preferred name)
- **Password**: Set a secure root password
- **SSH Public Key**: (Optional) Add your SSH public key for key-based authentication

### Template Tab
- **Template**: Select `debian-12-standard_*`
- **Storage**: Select your storage (usually `local`)

### Root Disk Tab
- **Storage**: Select your storage
- **Disk Size**: `32GB` (minimum, 64GB+ recommended for active downloading)
- **SSD Emulation**: Enable if using SSD storage

### CPU Tab
- **Cores**: `2` (minimum, 2-4 recommended for concurrent downloads)
- **CPU Units**: `1024` (default)

### Memory Tab
- **Memory**: `2048` MB (2GB minimum, 4GB recommended)
- **Swap**: `1024` MB (recommended for download operations)

### Network Tab
- **IPv4/CIDR**: `192.168.2.20/24`
- **Gateway (IPv4)**: `192.168.2.1` (pfSense VLAN 20 interface)
- **Bridge**: `vmbr0` (your main bridge)
- **VLAN Tag**: `20` (critical - this connects to VLAN 20)
- **Firewall**: Enable if you want Proxmox firewall rules (pfSense handles most rules)

### DNS Tab
- **DNS Domain**: Leave empty or use your domain
- **DNS Server**: `192.168.2.1` (pfSense VLAN 20) or `8.8.8.8` (Google DNS)

### Confirm Tab
- Review all settings
- Click **Finish**

## Step 3: Configure Bind Mounts (Storage)

Before starting the container, configure bind mounts to share host storage:

1. **Stop the container** (if started): Right-click container → **Stop**

2. **Edit container configuration**:
   - Right-click container → **Shell** (or use SSH to Proxmox host)
   - Edit the container config file:
     ```bash
     nano /etc/pve/lxc/<CONTAINER_ID>.conf
     ```
   - Or use Proxmox web interface: Right-click container → **Resources** → **Add** → **Mount Point**

3. **Add mount points** (add these lines to the config file):
   ```
   mp0: /path/to/host/downloads,mp=/mnt/downloads
   mp1: /path/to/host/completed,mp=/mnt/completed
   ```
   
   Replace `/path/to/host/downloads` and `/path/to/host/completed` with your actual host paths.

   **Example:**
   ```
   mp0: /mnt/pve/storage/downloads,mp=/mnt/downloads
   mp1: /mnt/pve/storage/completed,mp=/mnt/completed
   ```

4. **Set proper permissions** (on Proxmox host):
   ```bash
   # Ensure directories exist
   mkdir -p /path/to/host/downloads
   mkdir -p /path/to/host/completed
   
   # Set ownership (adjust UID/GID as needed)
   chown -R 1000:1000 /path/to/host/downloads
   chown -R 1000:1000 /path/to/host/completed
   ```

5. **Start the container**: Right-click container → **Start**

## Step 4: Start Container and Initial Configuration

1. **Start the container**: Right-click container → **Start**

2. **Open console**: Right-click container → **Console** (or use SSH)

3. **Update system**:
   ```bash
   apt update && apt upgrade -y
   ```

4. **Install basic tools**:
   ```bash
   apt install -y curl wget nano htop net-tools
   ```

5. **Configure hostname** (if not set correctly):
   ```bash
   hostnamectl set-hostname download-clients
   ```

6. **Verify network connectivity**:
   ```bash
   ping -c 4 8.8.8.8
   ping -c 4 192.168.2.1  # Should reach pfSense VLAN 20 gateway
   ```

7. **Verify bind mounts**:
   ```bash
   ls -la /mnt/downloads
   ls -la /mnt/completed
   # Should show contents of host directories
   ```

## Step 5: Install qBittorrent

### Install qBittorrent-nox

qBittorrent-nox is the headless (no GUI) version suitable for server use:

```bash
apt install -y qbittorrent-nox
```

### Create Systemd Service

Create a systemd service file for qBittorrent:

```bash
nano /etc/systemd/system/qbittorrent-nox.service
```

Add the following content:

```ini
[Unit]
Description=qBittorrent-nox service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/qbittorrent-nox
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
systemctl daemon-reload
systemctl enable qbittorrent-nox
systemctl start qbittorrent-nox
```

### Initial Configuration

1. **Access WebUI**: Open a browser and navigate to `http://192.168.2.20:8080`

2. **First-time setup**:
   - Default username: `admin`
   - Default password: `adminadmin`
   - **Change these immediately!**

3. **Configure qBittorrent** (WebUI → Options):

   **Downloads Tab:**
   - **Default Save Path**: `/mnt/downloads/qbittorrent`
   - **Keep incomplete torrents in**: `/mnt/downloads/qbittorrent/incomplete`
   - **Append .!qB extension to incomplete files**: Enabled

   **Connection Tab:**
   - **Port used for incoming connections**: `6881` (or your preferred port)
   - **UPnP / NAT-PMP port forwarding**: Disabled (pfSense handles this)

   **WebUI Tab:**
   - **Web User Interface**: Enabled
   - **IP Address**: `0.0.0.0` (listen on all interfaces)
   - **Port**: `8080`
   - **Use UPnP / NAT-PMP to forward the port**: Disabled
   - **Authentication**: Enabled
   - **Username**: Set a secure username
   - **Password**: Set a secure password
   - **Click "Save"**

   **Advanced Tab:**
   - **API**: Enabled (required for ARR integration)
   - **API IP Address**: `0.0.0.0` (listen on all interfaces)
   - **API Port**: `8080` (same as WebUI, or use separate port like 8082)
   - **Click "Save"**

4. **Create download directories**:
   ```bash
   mkdir -p /mnt/downloads/qbittorrent/incomplete
   mkdir -p /mnt/completed/qbittorrent
   chmod 755 /mnt/downloads/qbittorrent
   chmod 755 /mnt/completed/qbittorrent
   ```

5. **Restart qBittorrent**:
   ```bash
   systemctl restart qbittorrent-nox
   ```

6. **Verify service**:
   ```bash
   systemctl status qbittorrent-nox
   ```

## Step 6: Install SABnzbd

### Install Python and Dependencies

SABnzbd requires Python 3:

```bash
apt install -y python3 python3-pip python3-dev python3-setuptools python3-wheel
```

### Install SABnzbd

Install SABnzbd using pip:

```bash
pip3 install --upgrade pip
pip3 install sabnzbd
```

### Create Systemd Service

Create a systemd service file for SABnzbd:

```bash
nano /etc/systemd/system/sabnzbd.service
```

Add the following content:

```ini
[Unit]
Description=SABnzbd service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/sabnzbd --browser 0 --server 0.0.0.0:8081
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
systemctl daemon-reload
systemctl enable sabnzbd
systemctl start sabnzbd
```

### Initial Configuration

1. **Access WebUI**: Open a browser and navigate to `http://192.168.2.20:8081`

2. **First-time setup wizard**:
   - Follow the setup wizard
   - **Language**: Select your preferred language
   - **Username**: Set a secure username
   - **Password**: Set a secure password
   - **Port**: `8081`
   - **HTTPS Port**: `9090` (optional, for HTTPS access)
   - **Click "Test Server"** and continue

3. **Configure SABnzbd** (WebUI → Config):

   **Folders Tab:**
   - **Temporary Download Folder**: `/mnt/downloads/sabnzbd/incomplete`
   - **Completed Download Folder**: `/mnt/completed/sabnzbd`
   - **Watched Folder**: Leave default or set custom path
   - **Click "Save"**

   **General Tab:**
   - **Host**: `0.0.0.0` (listen on all interfaces)
   - **Port**: `8081`
   - **HTTPS Port**: `9090` (optional)
   - **Username**: (as set during wizard)
   - **Password**: (as set during wizard)
   - **Click "Save"**

   **Switches Tab:**
   - **Enable HTTPS**: Optional (requires certificate setup)
   - **Click "Save"**

4. **Create download directories**:
   ```bash
   mkdir -p /mnt/downloads/sabnzbd/incomplete
   mkdir -p /mnt/completed/sabnzbd
   chmod 755 /mnt/downloads/sabnzbd
   chmod 755 /mnt/completed/sabnzbd
   ```

5. **Get API Key** (for ARR integration):
   - WebUI → Config → General
   - Copy the **API Key** (you'll need this for Radarr/Sonarr configuration)

6. **Restart SABnzbd**:
   ```bash
   systemctl restart sabnzbd
   ```

7. **Verify service**:
   ```bash
   systemctl status sabnzbd
   ```

## Step 7: Configure pfSense Firewall Rules

### Update LAN Interface Rules

Configure pfSense to allow the ARR stack (on LAN) to communicate with download clients (on VLAN 20):

1. **Access pfSense**: Navigate to `https://192.168.1.1` (or your pfSense IP)

2. **Firewall → Rules → LAN**:
   - Find or create the "Allow LAN to Downloaders" rule
   - **Action**: Pass
   - **Interface**: LAN
   - **Protocol**: TCP
   - **Source**: LAN Net (or specific IP: ArrStack VM `192.168.1.20`)
   - **Destination**: Single host or alias → `192.168.2.20`
   - **Destination Port**: `8080, 8081` (qBittorrent and SABnzbd)
   - **Description**: Allow ARR stack to access download clients
   - **Save** and **Apply Changes**

### Verify VLAN 20 Interface Rules

Ensure VLAN 20 rules are configured:

1. **Firewall → Rules → VLAN 20**:
   - **Block VLAN 20 to LAN**: Should exist (blocks download clients from initiating connections to LAN)
   - **Allow Internet Access**: Should exist (allows download clients to reach internet)

## Step 8: Configure ARR Stack Integration

### Radarr Configuration

1. **Access Radarr**: Navigate to `http://192.168.1.20:7878` (or your Radarr URL)

2. **Settings → Download Clients → Add Client → qBittorrent**:
   - **Name**: `qBittorrent-VLAN20`
   - **Host**: `192.168.2.20`
   - **Port**: `8080`
   - **Username**: (qBittorrent WebUI username)
   - **Password**: (qBittorrent WebUI password)
   - **Category**: `radarr` (optional, for organization)
   - **Click "Test"** to verify connection
   - **Save**

### Sonarr Configuration

1. **Access Sonarr**: Navigate to `http://192.168.1.20:8989` (or your Sonarr URL)

2. **Settings → Download Clients → Add Client → qBittorrent**:
   - **Name**: `qBittorrent-VLAN20`
   - **Host**: `192.168.2.20`
   - **Port**: `8080`
   - **Username**: (qBittorrent WebUI username)
   - **Password**: (qBittorrent WebUI password)
   - **Category**: `sonarr` (optional, for organization)
   - **Click "Test"** to verify connection
   - **Save**

3. **Settings → Download Clients → Add Client → SABnzbd**:
   - **Name**: `SABnzbd-VLAN20`
   - **Host**: `192.168.2.20`
   - **Port**: `8081`
   - **API Key**: (from SABnzbd WebUI → Config → General)
   - **Category**: `sonarr` (optional, for organization)
   - **Click "Test"** to verify connection
   - **Save**

## Container Resource Recommendations

### Minimum Configuration
- **CPU**: 2 cores
- **RAM**: 2GB
- **Disk**: 32GB (container) + bind mounts for storage
- **Network**: Static IP on VLAN 20 (192.168.2.20)

### Recommended Configuration
- **CPU**: 2-4 cores
- **RAM**: 4GB
- **Disk**: 64GB+ (container) + bind mounts for storage
- **Network**: Static IP on VLAN 20 (192.168.2.20)

### For Heavy Download Usage
- **CPU**: 4 cores
- **RAM**: 8GB
- **Disk**: 128GB+ (container) + bind mounts for storage
- **Network**: Static IP on VLAN 20 (192.168.2.20)

## Network Configuration Details

### IP Assignment
- **Container IP**: `192.168.2.20`
- **Gateway**: `192.168.2.1` (pfSense VLAN 20 interface)
- **Subnet**: `192.168.2.0/24`
- **DNS**: `192.168.2.1` (pfSense) or `8.8.8.8` (Google DNS)

### Port Requirements

| Port | Protocol | Service | Purpose | Access |
|------|----------|---------|---------|--------|
| 8080 | TCP | qBittorrent | WebUI and API | From LAN (via pfSense routing) |
| 8081 | TCP | SABnzbd | WebUI and API | From LAN (via pfSense routing) |
| 9090 | TCP | SABnzbd | HTTPS WebUI (optional) | From LAN (via pfSense routing) |
| 6881 | TCP/UDP | qBittorrent | BitTorrent incoming connections | Internet (via pfSense NAT) |

**Note**: Ports 8080 and 8081 are accessible from the LAN network (192.168.1.0/24) via pfSense inter-VLAN routing. The download clients cannot initiate connections to the LAN due to firewall rules.

## Security Considerations

1. **Change Default Passwords**: Immediately change default credentials for both qBittorrent and SABnzbd
2. **Use Strong Passwords**: Use complex passwords for WebUI access
3. **API Security**: Keep API keys secure and don't share them
4. **Firewall**: pfSense handles network isolation - download clients cannot reach LAN
5. **Updates**: Keep the container and applications updated regularly
6. **Backups**: Set up Proxmox backups for the container configuration
7. **Bind Mount Permissions**: Ensure proper file permissions on bind mount directories

## Troubleshooting

### Container Won't Start
- Check Proxmox logs: **Node** → **Syslog**
- Verify template is downloaded correctly
- Check resource availability (CPU, RAM, disk)
- Verify bind mount paths exist on host

### Network Issues
- Verify IP configuration: `ip addr show`
- Check gateway is reachable: `ping 192.168.2.1`
- Verify DNS resolution: `nslookup google.com`
- Check VLAN tag is set to `20` in container network config
- Verify pfSense VLAN 20 interface is configured

### Bind Mount Issues
- Verify mount points in container: `mount | grep mnt`
- Check host directory permissions
- Ensure directories exist on host
- Verify container config file has correct mount point entries

### qBittorrent Issues
- Check service status: `systemctl status qbittorrent-nox`
- Check logs: `journalctl -u qbittorrent-nox -n 50`
- Verify WebUI is accessible: `curl http://192.168.2.20:8080`
- Check API is enabled in qBittorrent settings
- Verify download paths are writable

### SABnzbd Issues
- Check service status: `systemctl status sabnzbd`
- Check logs: `journalctl -u sabnzbd -n 50`
- Verify WebUI is accessible: `curl http://192.168.2.20:8081`
- Check Python installation: `python3 --version`
- Verify download paths are writable

### ARR Stack Cannot Connect
- Verify pfSense firewall rule allows LAN → VLAN 20 on ports 8080 and 8081
- Test connectivity from ArrStack VM: `curl http://192.168.2.20:8080`
- Verify qBittorrent/SABnzbd credentials are correct
- Check API is enabled in both applications
- Verify API key is correct (for SABnzbd)

### Download Path Issues
- Verify bind mounts are accessible: `ls -la /mnt/downloads`
- Check file permissions: `ls -la /mnt/completed`
- Ensure directories exist: `mkdir -p /mnt/downloads/qbittorrent`
- Check disk space: `df -h`

## Verification Steps

1. **Container network connectivity**:
   ```bash
   ping -c 4 192.168.2.1  # pfSense VLAN 20 gateway
   ping -c 4 8.8.8.8      # Internet connectivity
   ```

2. **Bind mounts accessible**:
   ```bash
   ls -la /mnt/downloads
   ls -la /mnt/completed
   ```

3. **qBittorrent WebUI accessible**:
   - From LAN: `http://192.168.2.20:8080`
   - Should prompt for login

4. **SABnzbd WebUI accessible**:
   - From LAN: `http://192.168.2.20:8081`
   - Should show SABnzbd interface

5. **API connectivity from ArrStack VM**:
   ```bash
   # From ArrStack VM (192.168.1.20)
   curl http://192.168.2.20:8080/api/v2/app/version
   curl http://192.168.2.20:8081/api?mode=version
   ```

6. **ARR stack can add and test download clients**:
   - Add qBittorrent in Radarr/Sonarr
   - Add SABnzbd in Sonarr
   - Click "Test" for each client
   - Should show "Connection successful"

7. **Test download workflow**:
   - Add a movie/show in Radarr/Sonarr
   - Monitor download in qBittorrent/SABnzbd
   - Verify completed file appears in `/mnt/completed`

## Next Steps

After the container is set up and applications are configured:

1. Configure Usenet providers in SABnzbd (if using Usenet)
2. Set up categories in qBittorrent for better organization
3. Configure post-processing scripts if needed
4. Set up monitoring and alerts
5. Configure Proxmox backups for the container
6. Test complete download workflow end-to-end

## Notes

- The container IP address (192.168.2.20) is used to access both WebUIs and for ARR integration
- Keep the container updated: `apt update && apt upgrade -y`
- Consider setting up Proxmox backups for this container
- Download clients are isolated on VLAN 20 and cannot initiate connections to LAN
- All communication from ARR stack to download clients is routed through pfSense
- Bind mounts allow sharing storage between host and container
- Both applications run as root in this setup - consider creating dedicated users for production use

