# Proxmox LXC Container Setup for Download Clients

This document details the creation and configuration of an LXC container for qBittorrent and SABnzbd download clients on your Proxmox server. The container is isolated on VLAN 20 for security and traffic separation.

## Overview

We'll create a single LXC container running Debian 12 to host both qBittorrent (BitTorrent client) and SABnzbd (Usenet client). The container is connected to VLAN 20 (192.168.3.0/24) and communicates with the ARR stack (Radarr, Sonarr) on the LAN network via pfSense routing.

## Prerequisites

- Proxmox server with LXC support enabled
- Access to Proxmox web interface
- Debian 12 template downloaded (if not already available)
- VLAN 20 configured on pfSense (192.168.3.0/24)
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
- **IPv4/CIDR**: `192.168.3.20/24`
- **Gateway (IPv4)**: `192.168.3.1` (pfSense VLAN 20 interface)
- **Bridge**: `vmbr0` (your main bridge)
- **VLAN Tag**: `20` (critical - this connects to VLAN 20)
- **Firewall**: Enable if you want Proxmox firewall rules (pfSense handles most rules)

### DNS Tab
- **DNS Domain**: Leave empty or use your domain
- **DNS Server**: `192.168.3.1` (pfSense VLAN 20) or `8.8.8.8` (Google DNS)

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
   ping -c 4 192.168.3.1  # Should reach pfSense VLAN 20 gateway
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

1. **Access WebUI**: Open a browser and navigate to `http://192.168.3.20:8080`

2. **First-time setup**:
   - Default username: `admin`
   - Default password: qBittorrent generates a random password on first start
   - **Find the password in the logs**:
     ```bash
     # Check systemd journal for the password
     journalctl -u qbittorrent-nox | grep -i password
     
     # Or view recent logs
     journalctl -u qbittorrent-nox -n 100
     
     # The password will be displayed in the startup log output
     ```
   - **Alternative**: Check qBittorrent log file (if running as root):
     ```bash
     cat /root/.local/share/qBittorrent/logs/*.log | grep -i password
     ```
   - **Change these immediately after first login!**

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

SABnzbd requires Python 3 (included as a dependency):

```bash
apt install -y python3
```

### Install SABnzbd

Install SABnzbd using the Debian package:

```bash
apt install -y sabnzbdplus
```

**Note**: The `sabnzbdplus` package is the official Debian package for SABnzbd. It handles all dependencies automatically, includes a systemd service, and integrates with the system package manager.

### Enable and Start Systemd Service

The Debian package includes a systemd service. Enable and start it:

```bash
systemctl enable sabnzbdplus
systemctl start sabnzbdplus
```

**Note**: The service is named `sabnzbdplus` and is included with the package. If you need to modify the service configuration (e.g., change the port or add startup options), you can override it by creating a systemd override:

```bash
systemctl edit sabnzbdplus
```

This will create an override file at `/etc/systemd/system/sabnzbdplus.service.d/override.conf` where you can add custom ExecStart options.

### Initial Configuration

1. **Access WebUI**: Open a browser and navigate to `http://192.168.3.20:8081`

2. **First-time setup wizard**:
   - Follow the setup wizard
   - **Language**: Select your preferred language
   - **Username**: Set a secure username (required for initial setup)
   - **Password**: Set a secure password
   - **Port**: `8081`
   - **HTTPS Port**: `9090` (optional, for HTTPS access)
   - **Click "Test Server"** and continue
   
   **Important**: You must configure a username during the initial setup wizard. This is required to complete the configuration.

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
   systemctl restart sabnzbdplus
   ```

7. **Verify service**:
   ```bash
   systemctl status sabnzbdplus
   ```

## Step 7: Set Up Automatic Updates

Configure automatic weekly updates for both qBittorrent and SABnzbd to ensure you're always running the latest versions with security patches and bug fixes.

### Create Update Script

Create a script that will update both services:

```bash
nano /usr/local/bin/update-download-clients.sh
```

Add the following content:

```bash
#!/bin/bash
# Update script for qBittorrent and SABnzbd
# This script updates both services and restarts them if updates are available

LOG_FILE="/var/log/download-clients-update.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "[$DATE] Starting download clients update check..." >> "$LOG_FILE"

# Update qBittorrent (installed via apt)
echo "[$DATE] Checking for qBittorrent updates..." >> "$LOG_FILE"
apt update >> "$LOG_FILE" 2>&1
QB_UPGRADE_OUTPUT=$(apt list --upgradable 2>/dev/null | grep -i qbittorrent)
if [ -n "$QB_UPGRADE_OUTPUT" ]; then
    echo "[$DATE] qBittorrent update available. Upgrading..." >> "$LOG_FILE"
    apt upgrade -y qbittorrent-nox >> "$LOG_FILE" 2>&1
    echo "[$DATE] Restarting qBittorrent service..." >> "$LOG_FILE"
    systemctl restart qbittorrent-nox
    echo "[$DATE] qBittorrent updated and restarted" >> "$LOG_FILE"
else
    echo "[$DATE] qBittorrent is up to date" >> "$LOG_FILE"
fi

# Update SABnzbd (installed via apt)
echo "[$DATE] Checking for SABnzbd updates..." >> "$LOG_FILE"
# Get current version
CURRENT_VERSION=$(dpkg -s sabnzbdplus 2>/dev/null | grep Version | awk '{print $2}')
# Check for updates
apt update >> "$LOG_FILE" 2>&1
SAB_UPGRADE_OUTPUT=$(apt list --upgradable 2>/dev/null | grep -i sabnzbdplus)
if [ -n "$SAB_UPGRADE_OUTPUT" ]; then
    echo "[$DATE] SABnzbd update available. Upgrading..." >> "$LOG_FILE"
    apt upgrade -y sabnzbdplus >> "$LOG_FILE" 2>&1
    echo "[$DATE] Restarting SABnzbd service..." >> "$LOG_FILE"
    systemctl restart sabnzbdplus
    echo "[$DATE] SABnzbd updated and restarted" >> "$LOG_FILE"
else
    echo "[$DATE] SABnzbd is up to date (version: $CURRENT_VERSION)" >> "$LOG_FILE"
fi

echo "[$DATE] Update check completed" >> "$LOG_FILE"
```

Make the script executable:

```bash
chmod +x /usr/local/bin/update-download-clients.sh
```

### Create Systemd Service

Create a systemd service file for the update script:

```bash
nano /etc/systemd/system/update-download-clients.service
```

Add the following content:

```ini
[Unit]
Description=Update qBittorrent and SABnzbd
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/update-download-clients.sh
StandardOutput=journal
StandardError=journal
```

### Create Systemd Timer

Create a systemd timer that runs the update script weekly:

```bash
nano /etc/systemd/system/update-download-clients.timer
```

Add the following content:

```ini
[Unit]
Description=Weekly update timer for download clients
Requires=update-download-clients.service

[Timer]
# Run every Monday at 3:00 AM
OnCalendar=Mon *-*-* 03:00:00
# Run immediately if missed (e.g., container was off)
Persistent=true
# Randomize start time by up to 30 minutes to avoid system load spikes
RandomizedDelaySec=30m

[Install]
WantedBy=timers.target
```

Enable and start the timer:

```bash
systemctl daemon-reload
systemctl enable update-download-clients.timer
systemctl start update-download-clients.timer
```

### Verify Timer Setup

Check that the timer is active and scheduled:

```bash
# Check timer status
systemctl status update-download-clients.timer

# List all timers to see when it will run next
systemctl list-timers update-download-clients.timer
```

### Manual Update

You can manually trigger an update at any time:

```bash
# Run the update script manually
/usr/local/bin/update-download-clients.sh

# Or trigger via systemd
systemctl start update-download-clients.service
```

### View Update Logs

Check the update log to see update history:

```bash
# View recent update logs
tail -n 50 /var/log/download-clients-update.log

# Or view via journalctl
journalctl -u update-download-clients.service -n 50
```

**Note**: The timer is set to run every Monday at 3:00 AM. The `Persistent=true` option ensures that if the container is off during the scheduled time, the update will run when the container starts next. The `RandomizedDelaySec` adds a random delay of up to 30 minutes to avoid system load spikes if you have multiple containers updating simultaneously.

## Step 8: Configure pfSense Firewall Rules

### Update LAN Interface Rules

Configure pfSense to allow the ARR stack (on LAN) to communicate with download clients (on VLAN 20):

1. **Access pfSense**: Navigate to `https://192.168.1.1` (or your pfSense IP)

2. **Firewall → Rules → LAN**:
   - Find or create the "Allow LAN to Downloaders" rule
   - **Action**: Pass
   - **Interface**: LAN
   - **Protocol**: TCP
   - **Source**: LAN Net (or specific IP: ArrStack VM `192.168.1.20`)
   - **Destination**: Single host or alias → `192.168.3.20`
   - **Destination Port**: `8080, 8081` (qBittorrent and SABnzbd)
   - **Description**: Allow ARR stack to access download clients
   - **Save** and **Apply Changes**

### Verify VLAN 20 Interface Rules

Ensure VLAN 20 rules are configured:

1. **Firewall → Rules → VLAN 20**:
   - **Block VLAN 20 to LAN**: Should exist (blocks download clients from initiating connections to LAN)
   - **Allow Internet Access**: Should exist (allows download clients to reach internet)

## Step 9: Configure ARR Stack Integration

### Radarr Configuration

1. **Access Radarr**: Navigate to `http://192.168.1.20:7878` (or your Radarr URL)

2. **Settings → Download Clients → Add Client → qBittorrent**:
   - **Name**: `qBittorrent-VLAN20`
   - **Host**: `192.168.3.20`
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
   - **Host**: `192.168.3.20`
   - **Port**: `8080`
   - **Username**: (qBittorrent WebUI username)
   - **Password**: (qBittorrent WebUI password)
   - **Category**: `sonarr` (optional, for organization)
   - **Click "Test"** to verify connection
   - **Save**

3. **Settings → Download Clients → Add Client → SABnzbd**:
   - **Name**: `SABnzbd-VLAN20`
   - **Host**: `192.168.3.20`
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
- **Network**: Static IP on VLAN 20 (192.168.3.20)

### Recommended Configuration
- **CPU**: 2-4 cores
- **RAM**: 4GB
- **Disk**: 64GB+ (container) + bind mounts for storage
- **Network**: Static IP on VLAN 20 (192.168.3.20)

### For Heavy Download Usage
- **CPU**: 4 cores
- **RAM**: 8GB
- **Disk**: 128GB+ (container) + bind mounts for storage
- **Network**: Static IP on VLAN 20 (192.168.3.20)

## Network Configuration Details

### IP Assignment
- **Container IP**: `192.168.3.20`
- **Gateway**: `192.168.3.1` (pfSense VLAN 20 interface)
- **Subnet**: `192.168.3.0/24`
- **DNS**: `192.168.3.1` (pfSense) or `8.8.8.8` (Google DNS)

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
- Check gateway is reachable: `ping 192.168.3.1`
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
- **Can't find default password?** Check the logs for the randomly generated password:
  ```bash
  journalctl -u qbittorrent-nox | grep -i password
  # Or view full startup logs
  journalctl -u qbittorrent-nox -n 100
  ```
- Verify WebUI is accessible: `curl http://192.168.3.20:8080`
- Check API is enabled in qBittorrent settings
- Verify download paths are writable

### SABnzbd Issues
- Check service status: `systemctl status sabnzbdplus`
- Check logs: `journalctl -u sabnzbdplus -n 50`
- Verify WebUI is accessible: `curl http://192.168.3.20:8081`
- Check Python installation: `python3 --version`
- Verify download paths are writable

### ARR Stack Cannot Connect
- Verify pfSense firewall rule allows LAN → VLAN 20 on ports 8080 and 8081
- Test connectivity from ArrStack VM: `curl http://192.168.3.20:8080`
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
   ping -c 4 192.168.3.1  # pfSense VLAN 20 gateway
   ping -c 4 8.8.8.8      # Internet connectivity
   ```

2. **Bind mounts accessible**:
   ```bash
   ls -la /mnt/downloads
   ls -la /mnt/completed
   ```

3. **qBittorrent WebUI accessible**:
   - From LAN: `http://192.168.3.20:8080`
   - Should prompt for login

4. **SABnzbd WebUI accessible**:
   - From LAN: `http://192.168.3.20:8081`
   - Should show SABnzbd interface

5. **API connectivity from ArrStack VM**:
   ```bash
   # From ArrStack VM (192.168.1.20)
   curl http://192.168.3.20:8080/api/v2/app/version
   curl http://192.168.3.20:8081/api?mode=version
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

- The container IP address (192.168.3.20) is used to access both WebUIs and for ARR integration
- Keep the container updated: `apt update && apt upgrade -y`
- Consider setting up Proxmox backups for this container
- Download clients are isolated on VLAN 20 and cannot initiate connections to LAN
- All communication from ARR stack to download clients is routed through pfSense
- Bind mounts allow sharing storage between host and container
- Both applications run as root in this setup - consider creating dedicated users for production use

