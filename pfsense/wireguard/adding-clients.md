# Adding Clients to WireGuard (tun_wg1)

This guide provides step-by-step instructions for adding new peers (clients) to your tun_wg1 remote access tunnel. This is for connecting new devices like phones, laptops, or tablets to your network.

## Overview

When adding a new client to tun_wg1:
1. Create a new peer in pfSense
2. Assign an available IP from the 10.0.0.16/28 subnet
3. Generate or obtain the client's public key
4. Configure the client device
5. Test the connection

## Prerequisites

- Access to pfSense web interface
- WireGuard client installed on the device you want to connect
- Understanding of your current IP allocation (see below)

## Current IP Allocation

**tun_wg1 Subnet: 10.0.0.17/28**
- **Range**: 10.0.0.16 - 10.0.0.31 (16 total IPs)
- **Tunnel IP**: 10.0.0.17 (pfSense server)
- **In Use**:
  - 10.0.0.18 → Remote-Android-R...
  - 10.0.0.19 → ARM-Laptop
- **Available**: 10.0.0.20 - 10.0.0.30 (11 IPs available)

## Step-by-Step: Adding a New Peer in pfSense

### Step 1: Access WireGuard Peers

1. Log into pfSense web interface
2. Navigate to **VPN** → **WireGuard** → **Peers**
3. Click **Add** (top right) to create a new peer

### Step 2: Configure Peer Settings

**Basic Configuration:**

1. **Tunnel**: Select **tun_wg1** (Remote users)
   - This associates the peer with the remote access tunnel

2. **Description**: Enter a descriptive name
   - Example: "iPhone-UserName", "Windows-Laptop", "iPad-Home"
   - Use clear, identifiable names

3. **Public Key**: 
   - **Option A**: Generate on client first, then paste here
   - **Option B**: Generate in pfSense (less common, requires exporting)
   - **Recommended**: Generate on client device (more secure)

4. **Allowed IPs**: Enter the IP address for this client
   - Format: `10.0.0.X/32` (where X is an available IP)
   - Example: `10.0.0.20/32` for the next available IP
   - **/32 means**: Single IP address (not a range)

5. **Endpoint**: Leave empty (for most clients)
   - Clients initiate connection, so endpoint not needed on server
   - Only fill if client has a static public IP (rare)

6. **Persistent Keepalive**: 
   - **Recommended**: `25` (sends keepalive every 25 seconds)
   - Helps maintain connection through NAT/firewalls
   - Prevents connection timeout

7. **Pre-shared Key** (Optional):
   - Additional security layer
   - Generate if you want extra security
   - Must match on both server and client

### Step 3: Save and Apply

1. Click **Save**
2. Click **Apply Changes** (if prompted)
3. The peer is now configured on pfSense

## Step-by-Step: Configuring the Client Device

### Method 1: Using WireGuard Client (Recommended)

#### For Android/iOS:

1. **Install WireGuard app** from Play Store/App Store

2. **Generate key pair**:
   - Open WireGuard app
   - Tap **+** or **Add Tunnel**
   - Tap **Create from scratch** or **Create empty tunnel**
   - The app will generate a key pair automatically
   - **Copy the Public Key** (you'll need this for pfSense)

3. **Configure tunnel**:
   - **Name**: Give it a name (e.g., "Home VPN")
   - **Private Key**: Already generated (don't share this)
   - **Public Key**: Copy this to pfSense (Step 2 above)
   - **Address**: Enter the IP assigned in pfSense (e.g., `10.0.0.20/32`)
   - **DNS**: Optional - can set to your pfSense IP (192.168.1.1) or leave default
   - **Listen Port**: Leave empty (client doesn't need to listen)

4. **Add peer (pfSense server)**:
   - Tap **Add Peer** or **Peers** section
   - **Public Key**: Enter tun_wg1's public key (from pfSense)
   - **Endpoint**: Enter your public IP:51821
     - Format: `your.public.ip:51821`
     - Example: `203.0.113.45:51821`
   - **Allowed IPs**: Enter what you want to access
     - **Full VPN**: `0.0.0.0/0` (route all traffic)
     - **Home network only**: `192.168.1.0/24` (access LAN only)
     - **Both**: `192.168.1.0/24, 10.0.0.16/28` (LAN + WireGuard network)
   - **Persistent Keepalive**: `25` (should match pfSense setting)

5. **Save and connect**:
   - Save the configuration
   - Toggle the switch to connect
   - Check status - should show handshake time and data transfer

#### For Windows:

1. **Install WireGuard** from [wireguard.com](https://www.wireguard.com/install/)

2. **Generate key pair**:
   - Open WireGuard
   - Click **Add Tunnel** → **Add empty tunnel**
   - Click **Generate** next to Private Key
   - **Copy the Public Key** (shown below Private Key)

3. **Configure tunnel** (same as mobile, but in GUI):
   - **Name**: "Home VPN" or similar
   - **Address**: `10.0.0.20/32` (your assigned IP)
   - **DNS**: Optional

4. **Add peer**:
   - In the peer section, add:
     - **Public Key**: tun_wg1's public key
     - **Endpoint**: `your.public.ip:51821`
     - **Allowed IPs**: `192.168.1.0/24` (or your preference)
     - **Persistent Keepalive**: `25`

5. **Activate**: Click **Activate** to connect

#### For macOS:

1. **Install WireGuard**:
   - **Option A**: Download from [wireguard.com](https://www.wireguard.com/install/)
   - **Option B**: Install from Mac App Store (search for "WireGuard")
   - Both options provide the same GUI application

2. **Generate key pair**:
   - Open WireGuard application
   - Click **Add Tunnel** → **Add empty tunnel**
   - Click **Generate** next to Private Key
   - **Copy the Public Key** (shown below Private Key)

3. **Configure tunnel**:
   - **Name**: "Home VPN" or similar
   - **Address**: `10.0.0.20/32` (your assigned IP)
   - **DNS**: Optional - can set to your pfSense IP (192.168.1.1) or leave default

4. **Add peer**:
   - In the peer section, add:
     - **Public Key**: tun_wg1's public key (from pfSense)
     - **Endpoint**: `your.public.ip:51821`
       - Format: `your.public.ip:51821`
       - Example: `203.0.113.45:51821`
     - **Allowed IPs**: Enter what you want to access
       - **Full VPN**: `0.0.0.0/0` (route all traffic)
       - **Home network only**: `192.168.1.0/24` (access LAN only)
       - **Both**: `192.168.1.0/24, 10.0.0.16/28` (LAN + WireGuard network)
     - **Persistent Keepalive**: `25` (should match pfSense setting)

5. **Activate**: 
   - Click the toggle switch or **Activate** button to connect
   - Check status - should show handshake time and data transfer
   - The WireGuard icon in the menu bar will indicate connection status

**Note**: macOS also supports command-line WireGuard configuration (similar to Linux). See the Linux section for CLI instructions if you prefer that method.

#### For Linux:

1. **Install WireGuard**:
   ```bash
   # Ubuntu/Debian
   sudo apt install wireguard

   # Fedora
   sudo dnf install wireguard-tools
   ```

2. **Generate key pair**:
   ```bash
   # Generate private key
   wg genkey | tee privatekey | wg pubkey > publickey
   
   # View public key (copy this to pfSense)
   cat publickey
   ```

3. **Create configuration file**:
   ```bash
   sudo nano /etc/wireguard/wg0.conf
   ```

4. **Add configuration**:
   ```ini
   [Interface]
   PrivateKey = <paste private key here>
   Address = 10.0.0.20/32
   DNS = 192.168.1.1  # Optional

   [Peer]
   PublicKey = <tun_wg1 public key from pfSense>
   Endpoint = your.public.ip:51821
   AllowedIPs = 192.168.1.0/24
   PersistentKeepalive = 25
   ```

5. **Start WireGuard**:
   ```bash
   sudo wg-quick up wg0
   ```

6. **Check status**:
   ```bash
   sudo wg show
   ```

### Method 2: Using QR Code (Mobile Only)

Some WireGuard implementations support QR codes for easy setup:

1. **In pfSense**: After creating peer, you may see a QR code option
2. **On mobile**: Use WireGuard app's QR scanner
3. **Scan code**: App will import configuration automatically
4. **Complete setup**: May need to add endpoint and allowed IPs manually

## Configuration Templates

### pfSense Peer Configuration Template

When adding a new peer in pfSense, use these settings:

```
Tunnel: tun_wg1
Description: [Device Name]
Public Key: [From client device]
Allowed IPs: 10.0.0.X/32  (where X is next available IP)
Endpoint: (leave empty)
Persistent Keepalive: 25
Pre-shared Key: (optional)
```

### Client Configuration Template

**For accessing home network only:**
```ini
[Interface]
PrivateKey = <client private key>
Address = 10.0.0.X/32

[Peer]
PublicKey = <tun_wg1 public key>
Endpoint = your.public.ip:51821
AllowedIPs = 192.168.1.0/24
PersistentKeepalive = 25
```

**For full VPN (route all traffic):**
```ini
[Interface]
PrivateKey = <client private key>
Address = 10.0.0.X/32
DNS = 192.168.1.1

[Peer]
PublicKey = <tun_wg1 public key>
Endpoint = your.public.ip:51821
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Finding Your pfSense Public Key

To get tun_wg1's public key for client configuration:

1. Go to **VPN** → **WireGuard** → **Tunnels**
2. Click on **tun_wg1**
3. Find **Public Key** field
4. Copy this key to use in client configuration

## Finding Your Public IP and Port

**Public IP:**
- Check in pfSense: **Status** → **Dashboard** → WAN IP
- Or use external service: visit `whatismyip.com` from your network

**Port:**
- **tun_wg1 uses port**: 51821
- Must be forwarded in firewall if behind another router (unlikely with pfSense as main router)

## Firewall Rules for New Clients

After adding a client, ensure firewall rules allow traffic:

1. Go to **Firewall** → **Rules** → **WireGuard** (or tun_wg1 interface)
2. Verify rules exist to allow:
   - Traffic from WireGuard network (10.0.0.16/28) to LAN (192.168.1.0/24)
   - Return traffic from LAN to WireGuard network
3. If rules don't exist, create them:
   - **Action**: Pass
   - **Protocol**: Any
   - **Source**: WireGuard network (10.0.0.16/28)
   - **Destination**: LAN network (192.168.1.0/24)

## Testing the Connection

### Step 1: Check pfSense Status

1. Go to **VPN** → **WireGuard** → **Status**
2. Find your new peer under **tun_wg1**
3. Check:
   - **Latest Handshake**: Should show recent time (not "never")
   - **Endpoint**: May show client's IP
   - **RX/TX**: Should show data transfer if connected

### Step 2: Test from Client

**Ping test:**
```bash
# From client device
ping 192.168.1.1  # Should reach pfSense
ping 192.168.1.12  # Should reach devices on LAN (if firewall allows)
```

**Connection test:**
- Try accessing a service on your LAN (e.g., Jellyfin at 192.168.1.12:8096)
- Check if you can reach other devices

### Step 3: Verify Routing

**On client (if possible):**
```bash
# Check routing table includes WireGuard routes
ip route  # Linux
route print  # Windows
```

## Troubleshooting

### Handshake Shows "Never"

**Possible causes:**
1. Client not connected/active
2. Public keys don't match
3. Firewall blocking port 51821
4. Wrong endpoint address
5. Network connectivity issue

**Solutions:**
1. Verify client WireGuard is active and connected
2. Double-check public keys match exactly (no extra spaces)
3. Check firewall allows port 51821 (UDP)
4. Verify public IP and port are correct
5. Test network connectivity to your public IP

### Can Connect But No Data Transfer

**Possible causes:**
1. Firewall rules blocking traffic
2. Wrong Allowed IPs configuration
3. Routing issue

**Solutions:**
1. Check firewall rules on WireGuard interface
2. Verify Allowed IPs include networks you want to access
3. Check routing tables in pfSense

### Client Can't Reach LAN Devices

**Possible causes:**
1. Firewall rules not configured
2. Allowed IPs doesn't include LAN network
3. LAN devices blocking WireGuard network

**Solutions:**
1. Add firewall rule: Allow 10.0.0.16/28 → 192.168.1.0/24
2. Update client Allowed IPs to include 192.168.1.0/24
3. Check LAN device firewalls

### Connection Drops Frequently

**Possible causes:**
1. Persistent keepalive not set or too high
2. NAT timeout
3. Mobile network changing IPs

**Solutions:**
1. Set Persistent Keepalive to 25 seconds
2. Ensure keepalive is set on both server and client
3. This is normal on mobile networks - connection will re-establish

## IP Address Management

### Tracking Used IPs

Keep a record of assigned IPs:

| IP Address | Device | Description | Date Added |
|------------|--------|-------------|------------|
| 10.0.0.18 | Remote-Android-R... | Android phone | Existing |
| 10.0.0.19 | ARM-Laptop | Laptop | Existing |
| 10.0.0.20 | [New Device] | [Description] | [Date] |

### Next Available IPs

- **Next available**: 10.0.0.20
- **Remaining**: 10.0.0.21 - 10.0.0.30 (10 IPs)
- **Reserved**: 10.0.0.16 (network), 10.0.0.17 (server), 10.0.0.31 (broadcast)

## Best Practices

1. **Use descriptive names**: Makes management easier
2. **Document IP assignments**: Keep track of which device has which IP
3. **Set persistent keepalive**: Prevents connection timeouts
4. **Test immediately**: Verify connection works right after setup
5. **Use appropriate Allowed IPs**: Don't route all traffic unless needed
6. **Keep keys secure**: Never share private keys
7. **Regular updates**: Keep WireGuard clients updated

## Removing a Client

If you need to remove a client:

1. Go to **VPN** → **WireGuard** → **Peers**
2. Find the peer to remove
3. Click **Delete** or **Remove**
4. **Apply Changes**
5. Document the IP is now available for reuse

## Next Steps

After adding a client:
1. Test connectivity to your network
2. Verify you can access desired services
3. Document the new client in your records
4. Consider setting up automatic connection on client device

For understanding the configuration differences, see [configuration-analysis.md](configuration-analysis.md).


