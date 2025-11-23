# Mail-in-a-Box Installation

This document details the installation and initial configuration of Mail-in-a-Box on your Ubuntu 22.04 LXC container.

## Overview

Mail-in-a-Box is an all-in-one email server solution that automatically configures Postfix, Dovecot, SpamAssassin, ClamAV, and other necessary components. It provides a simple web-based administration interface.

## Prerequisites

- Ubuntu 22.04 LTS LXC container created and running
- Container has internet connectivity
- DNS A record for `mail.obusan.me` pointing to your public IP (configured at Hostinger)
- DNS MX record for `obusan.me` pointing to `mail.obusan.me` (configured at Hostinger)
- At least 2GB RAM and 20GB disk space available
- Root or sudo access to the container

## Step 1: Access the Container

1. **Via Proxmox Console**:
   - Right-click container → **Console**
   - Login with root credentials

2. **Via SSH** (if configured):
   ```bash
   ssh root@192.168.1.13  # Replace with your container IP
   ```

## Step 2: Update System

Before installation, ensure the system is up to date:

```bash
apt update && apt upgrade -y
```

If the kernel was updated, you may need to reboot (though LXC containers typically don't need kernel updates).

## Step 3: Download and Run Installation Script

Mail-in-a-Box installation is done via a single script:

```bash
curl -s https://mailinabox.email/setup.sh | sudo bash
```

**Important**: This script will:
- Install all required packages
- Configure the system
- Set up the web interface
- Generate SSL certificates (after DNS is verified)

The installation process takes approximately 15-30 minutes depending on your system speed and internet connection.

## Step 4: Initial Configuration

During installation, the script will prompt you for:

### 1. System Hostname
- Enter: `mail.obusan.me`
- This must match your DNS A record

### 2. Email Address for System Alerts
- Enter: `admin@obusan.me` (or your preferred admin email)
- This email will receive system notifications

### 3. DNS Configuration
- The script will check if DNS records are properly configured
- It will provide instructions if records are missing

### 4. SSL Certificate
- The script will attempt to obtain a Let's Encrypt certificate
- This requires DNS A record to be properly configured and propagated

## Step 5: Access Admin Panel

After installation completes:

1. **Open web browser** and navigate to:
   ```
   https://mail.obusan.me/admin
   ```

2. **Login** with:
   - **Username**: The email address you provided during setup (e.g., `admin@obusan.me`)
   - **Password**: The password will be displayed at the end of the installation script output

**Important**: Save the admin password securely! You can change it later in the admin panel.

## Step 6: Initial Admin Panel Configuration

### Change Admin Password

1. Log into admin panel
2. Go to **System** → **Status Checks**
3. Click **Change Password** (or go to **System** → **Users**)
4. Set a new secure password

### Verify System Status

1. Check **System** → **Status Checks**
2. Review any warnings or errors
3. Common issues:
   - DNS records not yet propagated (wait and check again)
   - Port forwarding not configured (see firewall-configuration.md)
   - SSL certificate issues (usually DNS-related)

### Configure Domain

1. Go to **System** → **External DNS**
2. Verify that `obusan.me` is listed
3. The system should show DNS records that need to be added

## Step 7: Get DNS Records from Mail-in-a-Box

Mail-in-a-Box will provide you with the exact DNS records to add:

1. **Log into admin panel**: `https://mail.obusan.me/admin`
2. **Navigate to**: **System** → **External DNS**
3. **Copy the following records**:
   - SPF record (TXT)
   - DKIM record (TXT) - usually named `mail._domainkey`
   - DMARC record (TXT) - usually named `_dmarc`

4. **Add these records at Hostinger** (see [dns-configuration.md](dns-configuration.md))

## Step 8: Verify Installation

### Check Services Status

In the container, verify services are running:

```bash
systemctl status mailinabox
systemctl status postfix
systemctl status dovecot
```

All services should show as `active (running)`.

### Test Web Interface

1. **Admin Panel**: `https://mail.obusan.me/admin` - Should load without SSL errors
2. **Webmail**: `https://mail.obusan.me/mail` - Should show login page (no users created yet)

### Check DNS Status

In the admin panel:
1. Go to **System** → **External DNS**
2. Review the status of DNS records
3. Green checkmarks indicate properly configured records
4. Red X marks indicate missing or incorrect records

## Step 9: Configure Firewall (pfSense)

Before emails can be sent/received, configure port forwarding in pfSense. See [firewall-configuration.md](firewall-configuration.md) for detailed instructions.

**Required ports to forward:**
- 25 (SMTP)
- 587 (SMTP submission)
- 993 (IMAPS)
- 995 (POP3S)
- 80 (HTTP)
- 443 (HTTPS)

## Post-Installation Checklist

- [ ] Installation completed without errors
- [ ] Admin panel accessible at `https://mail.obusan.me/admin`
- [ ] Webmail accessible at `https://mail.obusan.me/mail`
- [ ] SSL certificate obtained (no browser warnings)
- [ ] DNS records copied from admin panel
- [ ] SPF, DKIM, DMARC records added to Hostinger DNS
- [ ] Firewall port forwarding configured
- [ ] System status checks show no critical errors

## Common Installation Issues

### DNS Not Resolving

**Symptom**: Installation script can't verify DNS records

**Solution**:
1. Verify A record for `mail.obusan.me` is set at Hostinger
2. Wait for DNS propagation (15-30 minutes)
3. Check DNS from container: `dig mail.obusan.me`
4. Re-run installation if needed (script is idempotent)

### SSL Certificate Failed

**Symptom**: Let's Encrypt certificate generation fails

**Solution**:
1. Ensure DNS A record is properly configured and propagated
2. Verify port 80 is accessible from internet (for HTTP-01 challenge)
3. Check firewall allows port 80
4. Re-run: `mailinabox/management/ssl_certificates.py`

### Port Already in Use

**Symptom**: Installation fails because ports are already in use

**Solution**:
1. Check what's using the port: `netstat -tulpn | grep :25`
2. Stop conflicting services
3. Re-run installation script

### Low Disk Space

**Symptom**: Installation fails due to insufficient disk space

**Solution**:
1. Check disk space: `df -h`
2. Clean up unnecessary files: `apt autoremove && apt autoclean`
3. Increase container disk size in Proxmox if needed
4. Re-run installation

### Memory Issues

**Symptom**: Installation is slow or fails

**Solution**:
1. Check available memory: `free -h`
2. Increase container RAM in Proxmox (minimum 2GB recommended)
3. Re-run installation

## Security Notes

- The admin panel password is critical - keep it secure
- Enable 2FA in the admin panel if available
- Regularly update the system: `apt update && apt upgrade -y`
- Mail-in-a-Box handles automatic security updates
- Review system status checks regularly

## Next Steps

After successful installation:

1. **Complete DNS setup**: Add SPF, DKIM, DMARC records at Hostinger
2. **Configure firewall**: Set up port forwarding in pfSense
3. **Create users**: Add email accounts (see [user-management.md](user-management.md))
4. **Test email**: Send and receive test emails
5. **Configure Gmail**: Set up Gmail as email client (see [gmail-integration.md](gmail-integration.md))

## Maintenance

Mail-in-a-Box includes automatic updates. To manually check for updates:

```bash
cd /root/mailinabox
git pull
./setup/start.sh
```

The system will automatically:
- Update all packages
- Renew SSL certificates
- Apply security patches
- Update Mail-in-a-Box itself

## Additional Resources

- **Mail-in-a-Box Documentation**: https://mailinabox.email/
- **Community Forum**: https://discourse.mailinabox.email/
- **GitHub Repository**: https://github.com/mail-in-a-box/mailinabox

