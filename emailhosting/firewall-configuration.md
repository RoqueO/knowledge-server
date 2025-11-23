# Firewall Configuration (pfSense)

This document details the firewall and port forwarding configuration required in pfSense to allow your self-hosted email server to send and receive emails.

## Overview

Your email server LXC container (e.g., `192.168.1.13`) needs several ports forwarded from your public IP to allow email traffic. pfSense will handle the NAT translation and firewall rules.

## Prerequisites

- pfSense router configured and running
- Access to pfSense web interface
- Static public IP address
- Email server LXC container IP address (e.g., `192.168.1.13`)
- Mail-in-a-Box installed and running

## Required Ports

The following ports need to be forwarded from your public IP to the mail server container:

| Port | Protocol | Purpose | Required |
|------|----------|---------|----------|
| 25 | TCP | SMTP (receiving mail) | Optional* |
| 587 | TCP | SMTP submission (sending mail) | **Required** |
| 993 | TCP | IMAPS (email client access) | **Required** |
| 995 | TCP | POP3S (optional email access) | Optional |
| 80 | TCP | HTTP (Let's Encrypt, webmail) | **Required** |
| 443 | TCP | HTTPS (webmail, admin panel) | **Required** |

\* Port 25 may be blocked by your ISP. If blocked, you can still send email via port 587, but receiving email from other servers may be limited. See troubleshooting section.

## Step 1: Access pfSense Web Interface

1. Open web browser
2. Navigate to your pfSense admin interface (typically `https://192.168.1.1`)
3. Log in with admin credentials

## Step 2: Configure Port Forwarding Rules

For each port, create a port forwarding (NAT) rule:

### Port 25 (SMTP) - Optional

**Note**: Many ISPs block port 25. Try forwarding it, but if it doesn't work, you can skip this.

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add** (top right)
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `25` to `25`
   - **Redirect Target IP**: `192.168.1.13` (your mail server container IP)
   - **Redirect Target Port**: `25`
   - **Description**: `Mail Server SMTP`
   - **NAT Reflection**: Enable (optional, for internal access)
4. Click **Save**
5. Click **Apply Changes**

### Port 587 (SMTP Submission) - Required

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add**
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `587` to `587`
   - **Redirect Target IP**: `192.168.1.13`
   - **Redirect Target Port**: `587`
   - **Description**: `Mail Server SMTP Submission`
   - **NAT Reflection**: Enable
4. Click **Save**
5. Click **Apply Changes**

### Port 993 (IMAPS) - Required

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add**
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `993` to `993`
   - **Redirect Target IP**: `192.168.1.13`
   - **Redirect Target Port**: `993`
   - **Description**: `Mail Server IMAPS`
   - **NAT Reflection**: Enable
4. Click **Save**
5. Click **Apply Changes**

### Port 995 (POP3S) - Optional

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add**
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `995` to `995`
   - **Redirect Target IP**: `192.168.1.13`
   - **Redirect Target Port**: `995`
   - **Description**: `Mail Server POP3S`
   - **NAT Reflection**: Enable
4. Click **Save**
5. Click **Apply Changes**

### Port 80 (HTTP) - Required

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add**
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `80` to `80`
   - **Redirect Target IP**: `192.168.1.13`
   - **Redirect Target Port**: `80`
   - **Description**: `Mail Server HTTP`
   - **NAT Reflection**: Enable
4. Click **Save**
5. Click **Apply Changes**

### Port 443 (HTTPS) - Required

1. Go to **Firewall** → **NAT** → **Port Forward**
2. Click **Add**
3. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: `443` to `443`
   - **Redirect Target IP**: `192.168.1.13`
   - **Redirect Target Port**: `443`
   - **Description**: `Mail Server HTTPS`
   - **NAT Reflection**: Enable
4. Click **Save**
5. Click **Apply Changes**

## Step 3: Configure Firewall Rules

pfSense should automatically create firewall rules for the port forwards, but verify they exist:

1. Go to **Firewall** → **Rules** → **WAN**
2. Verify rules exist for each forwarded port
3. Rules should allow traffic from WAN to the mail server container

If rules are missing, they should be created automatically when you apply the NAT rules. If not:

1. Go to **Firewall** → **Rules** → **WAN**
2. Click **Add**
3. Configure:
   - **Action**: Pass
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port**: (the port number, e.g., 587)
   - **Destination**: Single host or alias → `192.168.1.13`
   - **Description**: (e.g., `Allow SMTP Submission`)
4. Click **Save**
5. Click **Apply Changes**

## Step 4: Verify Port Forwarding

### From External Network

Use online tools to verify ports are open:

- [CanYouSeeMe.org](https://canyouseeme.org/) - Port checker
- [PortChecker.co](https://www.portchecker.co/) - Port checker
- [MXToolbox Port Check](https://mxtoolbox.com/PortCheck.aspx) - Port checker

Test each port:
- Port 25 (may show as closed if ISP blocks it)
- Port 587 (should be open)
- Port 993 (should be open)
- Port 443 (should be open)

### From Command Line

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

Replace `your-public-ip` with your actual static public IP address.

## Step 5: Configure Outbound NAT (if needed)

If your mail server needs to send emails, ensure outbound NAT is configured:

1. Go to **Firewall** → **NAT** → **Outbound**
2. Verify **Automatic outbound NAT** is enabled (default)
3. If using manual rules, ensure mail server container can access internet

## Troubleshooting

### Port 25 Blocked by ISP

**Symptom**: Port 25 shows as closed/blocked from external tests

**Impact**: 
- Cannot receive email from other mail servers directly
- Can still send email via port 587
- May need to use SMTP relay service for receiving

**Solutions**:
1. **Accept limitation**: Use port 587 for sending, webmail for receiving
2. **SMTP Relay**: Use a service like SendGrid, Mailgun, or AWS SES as relay
3. **Contact ISP**: Some ISPs will unblock port 25 for business/residential accounts (may require request)

### Ports Not Accessible

**Symptom**: Port checker shows ports as closed

**Checklist**:
1. Verify port forwarding rules are created and enabled
2. Check firewall rules allow traffic
3. Verify mail server container is running: `systemctl status mailinabox`
4. Check mail server is listening on ports: `netstat -tulpn | grep :587`
5. Test from LAN: `telnet 192.168.1.13 587`
6. Verify public IP is correct in port forward rules

### Can't Access Webmail from Internet

**Symptom**: `https://mail.obusan.me` doesn't load from external network

**Checklist**:
1. Verify port 443 is forwarded
2. Check DNS A record points to correct public IP
3. Verify SSL certificate is valid
4. Check firewall rules allow HTTPS traffic
5. Test with IP directly: `https://your-public-ip`

### NAT Reflection Issues

**Symptom**: Can access from external network but not from LAN

**Solution**:
1. Enable NAT Reflection in port forward rules
2. Or configure Split DNS (use internal IP when on LAN)
3. Or access via internal IP: `https://192.168.1.13`

## Security Considerations

1. **Firewall Rules**: Only allow necessary ports
2. **Fail2Ban**: Mail-in-a-Box includes fail2ban for brute force protection
3. **Regular Updates**: Keep pfSense updated
4. **Logging**: Monitor firewall logs for suspicious activity
5. **VPN Access**: Consider accessing admin panel via VPN instead of exposing to internet

## Port Forwarding Summary

After configuration, you should have the following port forwards:

```
WAN:25   → 192.168.1.13:25   (SMTP - optional)
WAN:587  → 192.168.1.13:587  (SMTP Submission - required)
WAN:993  → 192.168.1.13:993  (IMAPS - required)
WAN:995  → 192.168.1.13:995  (POP3S - optional)
WAN:80   → 192.168.1.13:80   (HTTP - required)
WAN:443  → 192.168.1.13:443  (HTTPS - required)
```

## Next Steps

After firewall configuration:
1. Verify ports are accessible from internet
2. Test email sending/receiving
3. Configure email accounts (see [user-management.md](user-management.md))
4. Set up Gmail client (see [gmail-integration.md](gmail-integration.md))

## Notes

- Port 587 is the standard port for email clients (SMTP submission)
- Port 25 is primarily for server-to-server communication
- If port 25 is blocked, you can still use your email server for sending and receiving via webmail
- NAT Reflection allows you to access services using the public domain name from within your LAN

