# Nginx Reverse Proxy Setup

This document details the setup of the Nginx LXC container to act as a reverse proxy for services, specifically Jellyfin, and securing them with Let's Encrypt.

## 1. Prerequisites

*   **DNS**: `jellyfin.obusan.me` must point to your home public IP.
*   **Port Forwarding**: Ports `80` and `443` on your router (pfSense) must be forwarded to the Nginx LXC IP.
*   **LXC OS**: Debian/Ubuntu (Assumed).

## 2. Installation

Update repositories and install Nginx and Certbot:

```bash
apt update && apt upgrade -y
apt install nginx python3-certbot-nginx -y
```

## 3. Nginx Configuration for Jellyfin

Create a new configuration file for Jellyfin:

```bash
nano /etc/nginx/sites-available/jellyfin.obusan.me.conf
```

Add the following configuration. Replace `<JELLYFIN_IP>` with the actual IP address of your Jellyfin LXC (e.g., `192.168.1.12`).

```nginx
server {
    listen 80;
    server_name jellyfin.obusan.me;

    # Redirect HTTP to HTTPS (Certbot will configure this, but good to have placeholder)
    # location / {
    #     return 301 https://$host$request_uri;
    # }
    
    # Proxy Settings
    location / {
        proxy_pass http://192.168.1.12:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Websocket Support (Required for Jellyfin)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Buffering off for streaming
        proxy_buffering off;
    }
}
```

Enable the site:

```bash
ln -s /etc/nginx/sites-available/jellyfin.obusan.me.conf /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default  # Optional: Remove default if not needed
nginx -t  # Test configuration
systemctl reload nginx
```

## 4. SSL Certificate (Let's Encrypt)

Run Certbot to obtain the certificate and automatically configure SSL for Nginx:

```bash
certbot --nginx -d jellyfin.obusan.me
```

*   Enter your email address when prompted.
*   Agree to terms.
*   Choose to redirect HTTP traffic to HTTPS (Recommended).

## 5. Auto-Renewal

Certbot usually sets up a timer automatically. Verify it:

```bash
systemctl status certbot.timer
```

To test renewal:

```bash
certbot renew --dry-run
```

## 6. SSH Access Note

If you require SSH access to this container from outside:
1.  **Port Forwarding**: You would typically forward a non-standard port (e.g., `2222`) on your router to port `22` of this LXC.
2.  **Security**: Ensure you use SSH Key authentication and disable password login for external SSH access.
3.  **Domain**: You can use `ssh user@jellyfin.obusan.me -p 2222`. The domain simply resolves to your IP.

*Note: Let's Encrypt SSL certificates are for HTTPS (Web) traffic, not SSH traffic.*
