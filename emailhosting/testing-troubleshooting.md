# Testing and Troubleshooting

This document provides comprehensive testing procedures and troubleshooting guides for your self-hosted email server.

## Overview

After setting up your email server, it's crucial to test all functionality and verify email deliverability. This guide covers testing procedures, common issues, and solutions.

## Prerequisites

- Mail-in-a-Box installed and running
- DNS records configured (MX, A, SPF, DKIM, DMARC)
- Firewall port forwarding configured
- At least one email account created
- Access to admin panel and webmail

## Testing Procedures

### 1. DNS Record Verification

Verify all DNS records are properly configured and propagated.

#### Check MX Record

```bash
dig obusan.me MX
# or
nslookup -type=MX obusan.me
```

**Expected Result**: Should show `mail.obusan.me` with priority 10

#### Check A Record

```bash
dig mail.obusan.me A
# or
nslookup mail.obusan.me
```

**Expected Result**: Should resolve to your static public IP address

#### Check SPF Record

```bash
dig obusan.me TXT | grep spf
# or
nslookup -type=TXT obusan.me
```

**Expected Result**: Should show SPF record like `v=spf1 mx a:mail.obusan.me ~all`

#### Check DKIM Record

```bash
dig mail._domainkey.obusan.me TXT
# or
nslookup -type=TXT mail._domainkey.obusan.me
```

**Expected Result**: Should show DKIM record with public key

#### Check DMARC Record

```bash
dig _dmarc.obusan.me TXT
# or
nslookup -type=TXT _dmarc.obusan.me
```

**Expected Result**: Should show DMARC policy record

#### Online DNS Checkers

- [MXToolbox](https://mxtoolbox.com/) - Comprehensive DNS checking
- [DNS Checker](https://dnschecker.org/) - Check DNS propagation globally
- [What's My DNS](https://www.whatsmydns.net/) - DNS propagation checker

### 2. Port Accessibility Testing

Verify ports are accessible from the internet.

#### Online Port Checkers

- [CanYouSeeMe.org](https://canyouseeme.org/)
- [PortChecker.co](https://www.portchecker.co/)
- [MXToolbox Port Check](https://mxtoolbox.com/PortCheck.aspx)

Test these ports:
- **Port 25**: May show as closed (ISP blocking is common)
- **Port 587**: Should be open (SMTP submission)
- **Port 993**: Should be open (IMAPS)
- **Port 443**: Should be open (HTTPS)

#### Command Line Testing

From an external network (not your home network):

```bash
# Test SMTP (port 25)
telnet your-public-ip 25

# Test SMTP submission (port 587)
telnet your-public-ip 587

# Test IMAPS (port 993)
openssl s_client -connect your-public-ip:993

# Test HTTPS (port 443)
curl -I https://your-public-ip
```

### 3. Email Deliverability Testing

#### Mail-Tester.com

1. Go to [Mail-Tester.com](https://www.mail-tester.com/)
2. Copy the test email address provided
3. Send an email from your Mail-in-a-Box account to that address
4. Click "Then check your score"
5. Review the score and recommendations

**Target Score**: 8/10 or higher (10/10 is ideal)

**What to Check**:
- SPF record is valid
- DKIM signature is valid
- DMARC policy is configured
- Reverse DNS (may fail - acceptable without ISP support)
- Blacklist status
- Spam score

#### MXToolbox Email Test

1. Go to [MXToolbox Email Health](https://mxtoolbox.com/emailhealth/)
2. Enter your domain: `obusan.me`
3. Review the report
4. Check for any issues or warnings

#### Send Test Emails

**Internal Test**:
1. Send email from `roque@obusan.me` to `matteo@obusan.me`
2. Check if received in webmail
3. Verify sender/receiver addresses are correct

**External Test - Send**:
1. Send email from your domain to:
   - Gmail account
   - Outlook account
   - Another email provider
2. Check if email arrives
3. Check spam folder if not in inbox
4. Verify sender address appears correctly

**External Test - Receive**:
1. Send email from external account (Gmail, etc.) to your domain
2. Check Mail-in-a-Box webmail
3. Verify email is received
4. Check spam filtering is working

### 4. Webmail Testing

#### Access Webmail

1. Navigate to: `https://mail.obusan.me/mail`
2. Login with email credentials
3. Verify interface loads correctly
4. Check SSL certificate is valid (no browser warnings)

#### Test Webmail Functions

- [ ] Compose new email
- [ ] Send email to external address
- [ ] Send email to internal address
- [ ] Receive email from external address
- [ ] Reply to email
- [ ] Forward email
- [ ] Create folders/labels
- [ ] Search emails
- [ ] Attach files

### 5. Email Client Testing (Gmail/IMAP/SMTP)

#### Test IMAP Connection

1. Configure email client with IMAP settings
2. Verify connection succeeds
3. Check emails sync correctly
4. Test sending email via SMTP

#### Test SMTP Connection

1. Configure SMTP settings in email client
2. Send test email
3. Verify email is sent successfully
4. Check recipient receives email

See [gmail-integration.md](gmail-integration.md) for detailed client setup.

### 6. SSL Certificate Verification

#### Check Certificate Validity

1. Access `https://mail.obusan.me`
2. Click padlock icon in browser
3. Verify certificate is valid
4. Check expiration date
5. Verify issuer is Let's Encrypt

#### Command Line Check

```bash
openssl s_client -connect mail.obusan.me:443 -servername mail.obusan.me
```

**Expected**: Should show valid certificate from Let's Encrypt

### 7. System Status Check

#### Mail-in-a-Box Admin Panel

1. Log into admin panel: `https://mail.obusan.me/admin`
2. Go to **System** → **Status Checks**
3. Review all status indicators
4. Address any warnings or errors

#### Command Line Status

```bash
# Check Mail-in-a-Box status
systemctl status mailinabox

# Check Postfix (SMTP)
systemctl status postfix

# Check Dovecot (IMAP)
systemctl status dovecot

# Check disk space
df -h

# Check memory
free -h
```

## Common Issues and Solutions

### Issue: Emails Going to Spam

**Symptoms**: Emails sent from your domain end up in spam folders

**Causes & Solutions**:

1. **Missing SPF/DKIM/DMARC**:
   - Verify all DNS records are configured
   - Use Mail-Tester to check record validity
   - Wait for DNS propagation

2. **Low IP Reputation**:
   - New IPs need warm-up period
   - Start with low email volume
   - Gradually increase sending
   - Avoid sending to spam traps

3. **Blacklisted IP**:
   - Check blacklist status: [MXToolbox Blacklist](https://mxtoolbox.com/blacklists.aspx)
   - If blacklisted, request removal from blacklist provider
   - Verify IP reputation: [Sender Score](https://www.senderscore.org/)

4. **Poor Email Content**:
   - Avoid spam trigger words
   - Don't send unsolicited emails
   - Include unsubscribe links
   - Use proper email formatting

### Issue: Can't Receive Emails

**Symptoms**: Emails sent to your domain don't arrive

**Causes & Solutions**:

1. **MX Record Not Configured**:
   - Verify MX record exists: `dig obusan.me MX`
   - Check MX record points to `mail.obusan.me`
   - Verify A record for `mail.obusan.me` is correct

2. **Port 25 Blocked**:
   - Test port 25 accessibility
   - If blocked by ISP, this limits receiving
   - Consider SMTP relay service
   - Port 587 works for sending but not receiving

3. **Firewall Blocking**:
   - Verify port forwarding in pfSense
   - Check firewall rules allow traffic
   - Test from external network

4. **Mail Server Not Running**:
   - Check service status: `systemctl status postfix`
   - Review logs: `journalctl -u postfix`
   - Restart services if needed

### Issue: Can't Send Emails

**Symptoms**: Emails fail to send or bounce

**Causes & Solutions**:

1. **SMTP Port Blocked**:
   - Verify port 587 is accessible
   - Test SMTP connection: `telnet mail.obusan.me 587`
   - Check firewall port forwarding

2. **Authentication Failed**:
   - Verify username is full email address
   - Check password is correct
   - Verify user account is active
   - Check mailbox quota

3. **SPF/DKIM Failures**:
   - Verify SPF record is configured
   - Check DKIM record exists
   - Use Mail-Tester to verify records

4. **Recipient Server Rejection**:
   - Check bounce messages for details
   - Verify recipient address is valid
   - Some servers may reject new IPs initially

### Issue: SSL Certificate Errors

**Symptoms**: Browser shows certificate warnings

**Causes & Solutions**:

1. **Certificate Not Obtained**:
   - Check DNS A record is correct
   - Verify port 80 is accessible (for Let's Encrypt)
   - Re-run certificate generation in admin panel

2. **Certificate Expired**:
   - Let's Encrypt certificates auto-renew
   - Check renewal status in admin panel
   - Manually renew if needed

3. **Wrong Domain**:
   - Verify accessing `mail.obusan.me` (not IP)
   - Check DNS A record points to correct IP

### Issue: Webmail Not Accessible

**Symptoms**: Can't access webmail interface

**Causes & Solutions**:

1. **DNS Not Resolving**:
   - Check DNS A record: `dig mail.obusan.me`
   - Wait for DNS propagation
   - Try accessing via IP (if firewall allows)

2. **Firewall Blocking**:
   - Verify port 443 is forwarded
   - Check firewall rules
   - Test from external network

3. **Service Not Running**:
   - Check Mail-in-a-Box status
   - Review system logs
   - Restart container if needed

### Issue: Slow Email Delivery

**Symptoms**: Emails take long time to send/receive

**Causes & Solutions**:

1. **Network Issues**:
   - Check internet connection
   - Test network speed
   - Verify firewall isn't throttling

2. **Server Resources**:
   - Check CPU usage: `htop`
   - Check memory: `free -h`
   - Check disk I/O: `iostat`

3. **Mail Queue**:
   - Check mail queue: `mailq` (in container)
   - Review queue for stuck messages
   - Clear queue if needed (with caution)

## Monitoring and Maintenance

### Regular Checks

- **Weekly**: Check system status in admin panel
- **Monthly**: Test email deliverability (Mail-Tester)
- **Quarterly**: Review DNS records and certificates
- **As Needed**: Check logs for errors

### Log Files

Important log locations:

```bash
# Mail-in-a-Box logs
/var/log/mailinabox.log

# Postfix logs (SMTP)
/var/log/mail.log
journalctl -u postfix

# Dovecot logs (IMAP)
journalctl -u dovecot

# System logs
journalctl -xe
```

### Monitoring Tools

- **Mail-in-a-Box Admin Panel**: Built-in status checks
- **System Monitoring**: `htop`, `df -h`, `free -h`
- **Email Testing**: Mail-Tester, MXToolbox
- **DNS Monitoring**: DNS checker tools

## Testing Checklist

Use this checklist after initial setup:

- [ ] DNS MX record resolves correctly
- [ ] DNS A record resolves to correct IP
- [ ] SPF record configured and valid
- [ ] DKIM record configured and valid
- [ ] DMARC record configured and valid
- [ ] Port 587 accessible from internet
- [ ] Port 993 accessible from internet
- [ ] Port 443 accessible from internet
- [ ] SSL certificate valid and not expired
- [ ] Webmail accessible and functional
- [ ] Admin panel accessible
- [ ] Can send email to external address
- [ ] Can receive email from external address
- [ ] Mail-Tester score 8/10 or higher
- [ ] Email client (Gmail) can connect via IMAP/SMTP
- [ ] System status checks show no critical errors
- [ ] No blacklist entries for your IP

## Next Steps

After successful testing:
1. Address any issues found
2. Set up regular monitoring
3. Configure backups (see [maintenance.md](maintenance.md))
4. Document any custom configurations
5. Share email account details with users

## Additional Resources

- **Mail-in-a-Box Documentation**: https://mailinabox.email/
- **Mail-Tester**: https://www.mail-tester.com/
- **MXToolbox**: https://mxtoolbox.com/
- **Let's Encrypt**: https://letsencrypt.org/

