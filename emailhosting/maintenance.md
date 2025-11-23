# Maintenance and Backup Procedures

This document covers ongoing maintenance, backup procedures, and monitoring for your self-hosted email server.

## Overview

Mail-in-a-Box includes automatic updates and backup systems, but regular monitoring and maintenance ensures optimal performance and data protection.

## Prerequisites

- Mail-in-a-Box installed and running
- Access to admin panel
- Access to Proxmox for container backups
- Basic understanding of system administration

## Automatic Maintenance

Mail-in-a-Box handles many maintenance tasks automatically:

### Automatic Updates

- **System packages**: Updated automatically
- **Mail-in-a-Box**: Updated automatically
- **Security patches**: Applied automatically
- **SSL certificates**: Renewed automatically (Let's Encrypt)

### Automatic Backups

Mail-in-a-Box includes a backup system that can be configured in the admin panel.

## Manual Maintenance Tasks

### Weekly Tasks

1. **Check System Status**
   - Log into admin panel: `https://mail.obusan.me/admin`
   - Go to **System** → **Status Checks**
   - Review any warnings or errors
   - Address any issues

2. **Review Email Deliverability**
   - Send test email to [Mail-Tester.com](https://www.mail-tester.com/)
   - Target score: 8/10 or higher
   - Address any issues found

### Monthly Tasks

1. **Review Disk Usage**
   ```bash
   df -h
   ```
   - Check available disk space
   - Monitor mailbox growth
   - Clean up if needed

2. **Check Logs**
   ```bash
   journalctl -u postfix -n 100
   journalctl -u dovecot -n 100
   ```
   - Review for errors
   - Check for suspicious activity

3. **Verify DNS Records**
   - Use [MXToolbox](https://mxtoolbox.com/) to verify DNS
   - Check SPF, DKIM, DMARC records
   - Verify certificates are valid

4. **Test Email Functionality**
   - Send test email to external address
   - Receive test email from external address
   - Verify webmail is accessible

### Quarterly Tasks

1. **Review User Accounts**
   - Check active users
   - Disable unused accounts
   - Review mailbox quotas

2. **Security Audit**
   - Review firewall rules
   - Check for unauthorized access
   - Verify backups are working

3. **Performance Review**
   - Check system resource usage
   - Review email delivery times
   - Optimize if needed

## Backup Procedures

### Mail-in-a-Box Built-in Backups

1. **Configure Backups in Admin Panel**
   - Log into admin panel
   - Go to **System** → **Backups**
   - Configure backup destination (local or remote)
   - Set backup schedule

2. **Backup Contents**
   - Email data (mailboxes)
   - User accounts
   - System configuration
   - DNS settings

3. **Backup Locations**
   - Local storage (container)
   - Remote storage (S3, SFTP, etc.)
   - External drive (if configured)

### Proxmox Container Backups

1. **Configure Proxmox Backup**
   - In Proxmox, select the mail server container
   - Go to **Backup** tab
   - Click **Add** to create backup job
   - Configure schedule (daily recommended)
   - Select storage destination

2. **Backup Settings**
   - **Mode**: Snapshot (for running container)
   - **Compression**: ZSTD (recommended)
   - **Retention**: Keep 7-30 days of backups

3. **Manual Backup**
   - Right-click container → **Backup**
   - Select storage
   - Click **Backup now**

### Backup Verification

1. **Test Restore**
   - Periodically test restoring from backup
   - Verify data integrity
   - Document restore procedure

2. **Monitor Backup Success**
   - Check backup logs in Proxmox
   - Verify backup files exist
   - Test backup accessibility

## Monitoring

### System Monitoring

#### Resource Usage

```bash
# CPU and Memory
htop

# Disk usage
df -h

# Disk I/O
iostat -x 1
```

#### Service Status

```bash
# Mail-in-a-Box
systemctl status mailinabox

# Postfix (SMTP)
systemctl status postfix

# Dovecot (IMAP)
systemctl status dovecot
```

### Email Monitoring

#### Mail Queue

```bash
# Check mail queue
mailq

# View queue details
postqueue -p
```

#### Email Logs

```bash
# Postfix logs
tail -f /var/log/mail.log

# Recent mail activity
journalctl -u postfix -f
```

### Admin Panel Monitoring

1. **Status Checks**
   - Regularly review status in admin panel
   - Address warnings promptly
   - Monitor system health

2. **User Activity**
   - Review user login activity
   - Monitor mailbox sizes
   - Check for unusual patterns

## Security Maintenance

### Regular Security Tasks

1. **Update Passwords**
   - Change admin password periodically
   - Encourage users to update passwords
   - Use strong, unique passwords

2. **Review Access Logs**
   - Check for failed login attempts
   - Monitor for suspicious activity
   - Review firewall logs

3. **SSL Certificate Monitoring**
   - Verify certificates are valid
   - Check expiration dates
   - Ensure auto-renewal is working

### Security Updates

Mail-in-a-Box handles security updates automatically, but verify:

```bash
# Check for pending updates
apt list --upgradable

# Review security updates
unattended-upgrades --dry-run
```

## Troubleshooting Common Issues

### High Disk Usage

**Symptoms**: Disk space running low

**Solutions**:
1. Check mailbox sizes in admin panel
2. Increase mailbox quotas or ask users to clean up
3. Increase container disk size in Proxmox
4. Archive old emails if needed

### Service Failures

**Symptoms**: Email service not working

**Solutions**:
1. Check service status: `systemctl status mailinabox`
2. Review logs for errors
3. Restart services if needed: `systemctl restart mailinabox`
4. Check system resources (CPU, memory, disk)

### Certificate Renewal Issues

**Symptoms**: SSL certificate warnings

**Solutions**:
1. Check DNS A record is correct
2. Verify port 80 is accessible (for Let's Encrypt)
3. Manually renew certificate in admin panel
4. Review certificate logs

### Backup Failures

**Symptoms**: Backups not completing

**Solutions**:
1. Check disk space for backup destination
2. Verify backup destination is accessible
3. Review backup logs
4. Test backup configuration

## Disaster Recovery

### Recovery Procedures

1. **Container Failure**
   - Restore from Proxmox backup
   - Verify services start correctly
   - Test email functionality

2. **Data Loss**
   - Restore from Mail-in-a-Box backup
   - Verify email data is restored
   - Test user access

3. **Complete System Failure**
   - Restore Proxmox container backup
   - Reconfigure if needed
   - Restore Mail-in-a-Box data
   - Verify DNS and firewall configuration

### Recovery Testing

- Test restore procedures quarterly
- Document recovery steps
- Keep recovery documentation accessible
- Verify backup integrity regularly

## Maintenance Checklist

### Daily
- [ ] (Automatic) System updates
- [ ] (Automatic) Security patches

### Weekly
- [ ] Check system status in admin panel
- [ ] Review error logs
- [ ] Test email sending/receiving

### Monthly
- [ ] Review disk usage
- [ ] Check backup success
- [ ] Verify DNS records
- [ ] Test email deliverability
- [ ] Review user accounts

### Quarterly
- [ ] Security audit
- [ ] Performance review
- [ ] Test backup restore
- [ ] Review firewall rules
- [ ] Update documentation

## Additional Resources

- **Mail-in-a-Box Documentation**: https://mailinabox.email/
- **Proxmox Backup Documentation**: https://pve.proxmox.com/wiki/Backup_and_Restore
- **System Monitoring**: Use built-in admin panel status checks

## Notes

- Mail-in-a-Box handles most maintenance automatically
- Regular monitoring prevents issues from becoming critical
- Backups are essential - test them regularly
- Keep documentation updated with any custom configurations
- Monitor disk space closely - email storage can grow quickly

