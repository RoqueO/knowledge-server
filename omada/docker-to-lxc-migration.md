# Docker to LXC Migration Guide

This document details the process of migrating your Omada Controller from a Docker container to a Proxmox LXC container, including backup, restore, and device re-adoption procedures.

## Overview

This migration process preserves all your Omada Controller settings, device configurations, and network settings by using the built-in backup and restore functionality. The migration involves:

1. Backing up the current Docker instance
2. Setting up the new LXC container (see [proxmox-lxc-setup.md](proxmox-lxc-setup.md))
3. Restoring the backup to the new LXC instance
4. Re-adopting devices if necessary
5. Verifying all functionality

## Prerequisites

- Existing Omada Controller running in Docker (on a different machine)
- Access to the Docker container's Omada Controller web interface
- New LXC container created and Omada Controller installed (see [proxmox-lxc-setup.md](proxmox-lxc-setup.md))
- Access to the new LXC container's Omada Controller web interface
- Network connectivity between both controllers and your Omada devices
- **Important**: Both controllers should be running the same or compatible Omada Controller version

## Pre-Migration Checklist

Before starting the migration, complete the following:

- [ ] Note the current Docker container's IP address
- [ ] Note the new LXC container's IP address
- [ ] Verify you can access both controller web interfaces
- [ ] Check the Omada Controller version on Docker instance
- [ ] Ensure the LXC container has the same or newer Omada Controller version
- [ ] Document all device IP addresses and MAC addresses (optional, for troubleshooting)
- [ ] Plan for brief downtime during migration
- [ ] Ensure you have admin access to both controllers

## Step 1: Verify Version Compatibility

**Critical**: Both controllers must be running the same or compatible versions for successful migration.

### Check Docker Container Version

1. Access your Docker container's Omada Controller web interface
2. Navigate to **Settings** → **About** (or **System Information**)
3. Note the **Controller Version** (e.g., `5.13.30`)

### Check LXC Container Version

1. Access your new LXC container's Omada Controller web interface
2. Navigate to **Settings** → **About** (or **System Information**)
3. Verify the version matches or is newer than the Docker version

**Note**: If versions don't match, update the LXC container's Omada Controller to match the Docker version, or update both to the latest version before migration.

## Step 2: Backup Current Docker Instance

### Create Backup via Web Interface

1. **Access Docker Controller**:
   - Open a web browser
   - Navigate to your Docker container's Omada Controller (e.g., `https://<DOCKER_IP>:8043`)
   - Log in with admin credentials

2. **Navigate to Backup Settings**:
   - Click **Settings** (gear icon in top right)
   - Go to **Maintenance** → **Backup & Restore**

3. **Create Backup**:
   - In the **Backup** section, you'll see backup options:
     - **Settings Only**: Backs up configurations without historical data (smaller file, faster)
     - **Settings + Statistics**: Includes historical statistics and logs (larger file)
   - Select your preferred backup type (recommended: **Settings + Statistics** for complete migration)
   - Click **Export** or **Backup** button
   - The backup file will download to your computer (filename format: `Omada_Controller_Backup_YYYYMMDD_HHMMSS.tar.gz`)

4. **Verify Backup File**:
   - Ensure the backup file downloaded successfully
   - Note the file location and name
   - File size should be reasonable (typically 1-50MB depending on configuration and statistics)

### Alternative: Backup via Command Line (Docker Container)

If you have SSH/console access to the Docker container:

```bash
# Access the Docker container
docker exec -it <container_name> bash

# Navigate to Omada data directory (location may vary)
cd /opt/tplink/EAPController/data

# Create manual backup (if needed)
tar -czf /tmp/omada_backup_$(date +%Y%m%d_%H%M%S).tar.gz .

# Copy backup out of container
docker cp <container_name>:/tmp/omada_backup_*.tar.gz ./
```

**Note**: The web interface backup method is recommended as it's the official supported method.

## Step 3: Prepare New LXC Container

Before restoring, ensure the new LXC container is ready:

1. **Verify Omada Controller is Running**:
   ```bash
   # SSH into LXC container
   ssh root@<LXC_IP>
   
   # Check service status
   systemctl status omada
   ```

2. **Access Web Interface**:
   - Open a web browser
   - Navigate to `https://<LXC_IP>:8043`
   - Complete initial setup if not already done (create admin account)

3. **Note the Controller IP**:
   - Write down the LXC container's IP address
   - This will be needed for device re-adoption if IP changed significantly

## Step 4: Restore Backup to LXC Container

### Restore via Web Interface

1. **Access LXC Controller**:
   - Open a web browser
   - Navigate to your new LXC container's Omada Controller (e.g., `https://<LXC_IP>:8043`)
   - Log in with admin credentials

2. **Navigate to Restore Settings**:
   - Click **Settings** (gear icon in top right)
   - Go to **Maintenance** → **Backup & Restore**

3. **Restore Backup**:
   - In the **Restore** section, click **Browse** or **Choose File**
   - Select the backup file you downloaded from the Docker container
   - Click **Restore** or **Import**
   - **Warning**: This will overwrite all current settings in the LXC controller

4. **Wait for Restore**:
   - The restore process may take several minutes depending on backup size
   - Do not close the browser or interrupt the process
   - You may see progress indicators or status messages

5. **Restore Complete**:
   - You may be automatically logged out
   - Log back in with your **original admin credentials** (from Docker container)
   - Verify settings have been restored

### Verify Restore Success

1. **Check Settings**:
   - Navigate through various settings sections
   - Verify sites, networks, and configurations are present
   - Check device lists (devices may show as disconnected initially)

2. **Check Logs** (if needed):
   ```bash
   # SSH into LXC container
   ssh root@<LXC_IP>
   
   # Check Omada Controller logs
   journalctl -u omada -n 100
   ```

## Step 5: Device Re-adoption

After restoring the backup, your devices may need to be re-adopted if:

- The controller IP address changed significantly
- Devices cannot reach the new controller IP
- Devices show as "Disconnected" or "Pending Adoption"

### Check Device Status

1. **Access LXC Controller**:
   - Navigate to `https://<LXC_IP>:8043`
   - Go to **Devices** or **Site** view

2. **Check Device Status**:
   - Devices should appear in the device list
   - Status may show as "Disconnected" or "Pending Adoption"

### Re-adoption Methods

#### Method 1: Automatic Re-adoption (Recommended)

If devices are on the same network and can discover the new controller:

1. **Wait for Auto-Discovery**:
   - Devices may automatically discover the new controller
   - This can take 5-15 minutes
   - Check device status periodically

2. **Manual Adoption** (if needed):
   - Select devices showing "Pending Adoption"
   - Click **Adopt** or **Re-adopt**
   - Wait for adoption process to complete

#### Method 2: Manual Re-adoption via Device Reset

If automatic re-adoption fails:

1. **Factory Reset Devices** (if necessary):
   - Physically access each device
   - Press and hold the reset button for 10 seconds
   - Device will reset to factory defaults

2. **Adopt Devices**:
   - In Omada Controller, go to **Devices**
   - Look for devices in "Pending Adoption" status
   - Click **Adopt** for each device
   - Follow the adoption wizard

3. **Reconfigure Devices** (if factory reset):
   - Since backup was restored, most settings should be preserved
   - Verify device configurations match previous setup
   - Update any device-specific settings if needed

#### Method 3: Update Controller IP on Devices (Advanced)

If you have SSH access to devices and want to avoid factory reset:

1. **SSH into Device**:
   ```bash
   ssh admin@<device_ip>
   ```

2. **Update Controller IP**:
   ```bash
   # Commands vary by device type - check device documentation
   # Example for some Omada devices:
   mca-cli
   set controller ip <NEW_LXC_IP>
   save
   reboot
   ```

**Note**: This method is device-specific and may not be available on all Omada devices.

## Step 6: Post-Migration Verification

After migration, verify everything is working correctly:

### Verify Controller Functionality

- [ ] Web interface is accessible
- [ ] All sites and networks are present
- [ ] Settings match previous configuration
- [ ] Statistics and logs are preserved (if included in backup)

### Verify Device Connectivity

- [ ] All devices show as "Connected" or "Online"
- [ ] Device configurations match previous setup
- [ ] Wireless networks are broadcasting correctly
- [ ] Client devices can connect to networks
- [ ] Internet connectivity works through devices

### Verify Network Features

- [ ] VLANs are configured correctly
- [ ] Firewall rules are active
- [ ] Guest networks are working (if applicable)
- [ ] Port forwarding rules are active (if applicable)
- [ ] QoS settings are applied (if applicable)

### Test Key Functions

1. **Client Connection Test**:
   - Connect a device to a wireless network
   - Verify internet connectivity
   - Test network speed

2. **Management Test**:
   - Make a small configuration change
   - Verify change is saved and applied
   - Revert change if needed

3. **Monitoring Test**:
   - Check device statistics
   - Verify logs are being recorded
   - Check system health indicators

## Step 7: Decommission Docker Container

Once migration is verified and stable:

1. **Stop Docker Container** (optional, for testing):
   ```bash
   docker stop <container_name>
   ```
   - Monitor devices to ensure they remain connected to LXC controller
   - If issues occur, restart Docker container

2. **Remove Docker Container** (after verification period):
   ```bash
   # Stop container
   docker stop <container_name>
   
   # Remove container (optional - keep for backup period)
   docker rm <container_name>
   
   # Remove image (optional)
   docker rmi <image_name>
   ```

3. **Update DNS/Bookmarks**:
   - Update any bookmarks pointing to old Docker IP
   - Update any scripts or automation using the controller
   - Update reverse proxy configurations if applicable

## Troubleshooting

### Backup Fails

**Issue**: Cannot create backup from Docker container

**Solutions**:
- Verify you have admin access
- Check disk space on Docker container
- Try "Settings Only" backup if "Settings + Statistics" fails
- Check Docker container logs for errors
- Ensure controller is running and accessible

### Restore Fails

**Issue**: Backup restore fails on LXC container

**Solutions**:
- Verify backup file is not corrupted (try downloading again)
- Check version compatibility between controllers
- Ensure LXC container has sufficient disk space
- Check Omada Controller logs: `journalctl -u omada -n 100`
- Try restoring "Settings Only" backup first
- Verify MongoDB is running: `systemctl status mongod`

### Devices Not Re-adopting

**Issue**: Devices remain disconnected after restore

**Solutions**:
- Verify devices can reach new controller IP (ping test)
- Check if controller IP changed significantly
- Ensure port 8088 (UDP) is not blocked
- Wait longer for auto-discovery (can take 15+ minutes)
- Try manual adoption via web interface
- Factory reset devices if necessary
- Check device logs (if accessible via SSH)

### Settings Not Restored

**Issue**: Some settings missing after restore

**Solutions**:
- Verify backup included all desired settings
- Check if backup type was "Settings Only" (may exclude some data)
- Re-export backup from Docker container with "Settings + Statistics"
- Manually reconfigure missing settings
- Check Omada Controller version compatibility

### Performance Issues

**Issue**: LXC container performance is poor

**Solutions**:
- Check resource allocation (CPU, RAM) in Proxmox
- Increase container resources if needed
- Monitor resource usage: `htop` in container
- Check MongoDB performance: `systemctl status mongod`
- Verify disk I/O is not saturated

### Network Connectivity Issues

**Issue**: Cannot access controller or devices after migration

**Solutions**:
- Verify container IP configuration: `ip addr show`
- Check gateway connectivity: `ping 192.168.1.1`
- Verify firewall rules on pfSense
- Check if ports 8043 and 8088 are accessible
- Test from different network devices
- Verify DNS resolution if using hostnames

## Rollback Procedure

If migration fails and you need to rollback:

1. **Stop LXC Controller** (optional):
   ```bash
   systemctl stop omada
   ```

2. **Restart Docker Container**:
   ```bash
   docker start <container_name>
   ```

3. **Verify Docker Controller**:
   - Access web interface
   - Verify devices reconnect
   - Check settings are intact

4. **Re-adopt Devices** (if needed):
   - Devices should automatically reconnect to Docker controller
   - If not, follow re-adoption procedures

5. **Troubleshoot LXC Issues**:
   - Fix issues in LXC container
   - Retry migration when ready

## Best Practices

1. **Test Migration First**: If possible, test migration with a non-production controller first
2. **Keep Docker Container**: Don't delete Docker container immediately - keep it running for a few days as backup
3. **Document Changes**: Note any differences or issues encountered during migration
4. **Regular Backups**: Set up regular backups of the new LXC controller
5. **Monitor Closely**: Watch device connectivity and controller performance for the first week
6. **Version Matching**: Always ensure version compatibility before migration
7. **Network Planning**: Plan IP address changes if controller IP will change significantly

## Notes

- Migration downtime is typically 15-30 minutes depending on backup size and device count
- Device re-adoption may take additional time (15-60 minutes depending on device count)
- Some historical statistics may be lost if using "Settings Only" backup
- Device firmware versions are preserved in the backup
- Network configurations, VLANs, and firewall rules are all preserved
- User accounts and permissions are preserved
- Site configurations and device groupings are preserved

## Next Steps

After successful migration:

1. Set up regular Proxmox container backups
2. Configure Omada Controller automatic backups (Settings > Maintenance > Backup & Restore)
3. Update any automation scripts or monitoring tools
4. Update documentation with new IP addresses
5. Consider setting up reverse proxy for external access (if needed)
6. Monitor controller performance and adjust resources if needed


