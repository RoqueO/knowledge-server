# WireGuard Configuration Analysis

This document provides a detailed walkthrough of your pfSense WireGuard configuration, explaining each setting and the key differences between tun_wg0 (site-to-site) and tun_wg1 (remote access).

## Accessing WireGuard Configuration in pfSense

### Navigation Path

1. **Log into pfSense web interface** (typically `https://192.168.1.1`)
2. Navigate to **VPN** → **WireGuard** in the top menu
3. You'll see tabs: **Tunnels**, **Peers**, **Settings**, **Status**

### Key Sections

- **Tunnels**: Configure tunnel interfaces (tun_wg0, tun_wg1)
- **Peers**: Configure peer connections for each tunnel
- **Settings**: Global WireGuard settings
- **Status**: View current tunnel status and statistics (what you see in your screenshot)

## Current Configuration Overview

Based on your status screen, here's what's configured:

### tun_wg0 (Home Tunnel - Site-to-Site)

**Tunnel Details:**
- **Interface**: tun_wg0
- **Description**: Home Tunnel
- **Listen Port**: 51820
- **MTU**: 1420
- **Address/Assignment**: (none)
- **Public Key**: C4499Akim9vQKI5s... (truncated)
- **Status**: Active (green indicator)

**Peer Configuration:**
- **Peer 1**: Beryll Peer
- **Public Key**: IbNYTGIZbfVVPfyG... (truncated)
- **Allowed IPs**: 10.0.0.0/28 (+1)
- **Endpoint**: (none) - not currently connected
- **Latest Handshake**: never
- **Data Transfer**: 0 B RX / 0 B TX

### tun_wg1 (Remote users - Remote Access)

**Tunnel Details:**
- **Interface**: tun_wg1
- **Description**: Remote users
- **Listen Port**: 51821
- **MTU**: 1420
- **Address/Assignment**: 10.0.0.17/28
- **Public Key**: SIFOLTVVMSAELZkr... (truncated)
- **Status**: Active (green indicator)

**Peer Configuration:**
- **Peer 1**: Remote-Android-R...
  - **Public Key**: A9oyoSNSiYq5u35Y... (truncated)
  - **Allowed IPs**: 10.0.0.18/32
  - **Endpoint**: (none)
  - **Latest Handshake**: never
  - **Data Transfer**: 0 B RX / 0 B TX

- **Peer 2**: ARM-Laptop
  - **Public Key**: 1NwijLmHA0BhwdGm... (truncated)
  - **Allowed IPs**: 10.0.0.19/32
  - **Endpoint**: (none)
  - **Latest Handshake**: never
  - **Data Transfer**: 0 B RX / 0 B TX

## Detailed Configuration Walkthrough

### Tunnel Configuration Settings

#### 1. Interface Name
- **tun_wg0**: First tunnel interface
- **tun_wg1**: Second tunnel interface
- **Why**: Each tunnel needs a unique interface name

#### 2. Description
- **tun_wg0**: "Home Tunnel" - indicates site-to-site purpose
- **tun_wg1**: "Remote users" - indicates remote access purpose
- **Why**: Helps identify the tunnel's purpose in the interface

#### 3. Listen Port
- **tun_wg0**: 51820
- **tun_wg1**: 51821
- **Why**: Each tunnel must listen on a unique port. Ports must be:
  - Open in firewall rules
  - Forwarded if behind NAT (though pfSense typically handles this)
  - Accessible from the internet for incoming connections

#### 4. MTU (Maximum Transmission Unit)
- **Both**: 1420
- **Why**: Accounts for WireGuard encryption overhead (~80 bytes). Standard Ethernet MTU is 1500, so 1420 prevents packet fragmentation.

#### 5. Address/Assignment (KEY DIFFERENCE)

**tun_wg0: (none)**
- **Why no IP?**: Site-to-site tunnels don't need a local IP address
- The tunnel acts as a **bridge** between networks
- Routing is handled by network routes, not tunnel IP addresses
- Traffic is forwarded based on destination network, not tunnel interface IP

**tun_wg1: 10.0.0.17/28**
- **Why has IP?**: Remote access tunnels need a gateway IP
- The tunnel interface acts as a **gateway** for connected clients
- Clients connect to this IP (10.0.0.17)
- The /28 subnet (10.0.0.16-31) provides IP addresses for clients
- Each client gets a unique IP from this range

#### 6. Public Key
- **What**: Derived from the tunnel's private key
- **Purpose**: Shared with peers so they can encrypt traffic to this tunnel
- **Security**: Safe to share, can't be used to decrypt traffic

### Peer Configuration Settings

#### 1. Description
- Identifies the peer (device or network)
- **tun_wg0**: "Beryll Peer" - likely another router/network
- **tun_wg1**: "Remote-Android-R...", "ARM-Laptop" - individual devices

#### 2. Public Key
- The peer's public key
- Used to encrypt traffic sent to this peer
- Must match the peer's actual public key

#### 3. Allowed IPs (KEY DIFFERENCE)

**tun_wg0 - Beryll Peer: 10.0.0.0/28**
- **Network range**: Routes traffic for entire 10.0.0.0/28 network
- **Why**: Site-to-site connects entire networks, not single devices
- Traffic destined for any IP in 10.0.0.0/28 goes through this peer

**tun_wg1 - Peers: 10.0.0.18/32, 10.0.0.19/32**
- **Single IPs**: Each peer gets exactly one IP address
- **/32 means**: Single host (not a network range)
- **Why**: Remote access gives each device its own unique IP
- Traffic destined for that specific IP goes to that peer

#### 4. Endpoint
- **Current**: (none) for all peers
- **What it should be**: IP address:port of the peer
- **Why "none"**: Peer hasn't connected yet, or peer is behind NAT (client initiates)
- **For site-to-site**: Should have endpoint if other side has public IP
- **For remote access**: Often (none) because clients initiate connection

#### 5. Latest Handshake
- **Current**: "never" for all peers
- **What it means**: No successful connection established
- **Why**: Could be:
  - Peers are offline
  - Configuration mismatch
  - Firewall blocking
  - Network connectivity issues

#### 6. Data Transfer (RX/TX)
- **Current**: 0 B for all peers
- **RX**: Received bytes (data from peer)
- **TX**: Transmitted bytes (data to peer)
- **Why 0**: No traffic has flowed, likely because handshake is "never"

## Side-by-Side Comparison: tun_wg0 vs tun_wg1

| Configuration Aspect | tun_wg0 (Site-to-Site) | tun_wg1 (Remote Access) |
|---------------------|------------------------|-------------------------|
| **Purpose** | Connect two networks | Connect individual devices |
| **IP Assignment** | None | 10.0.0.17/28 |
| **Why IP Assignment** | Acts as bridge, routes by network | Acts as gateway, needs server IP |
| **Peer Type** | Network/router (1 peer) | Individual devices (2+ peers) |
| **Allowed IPs Pattern** | Network range (/28) | Single IPs (/32) |
| **Endpoint** | Usually configured (if public IP) | Often (none) - clients initiate |
| **Connection Pattern** | Both sides can initiate | Clients initiate to server |
| **Use Case** | Permanent network link | On-demand device access |
| **IP Allocation** | Network-to-network | One IP per device |

## Understanding the Differences

### Why tun_wg0 Has No IP Assignment

**Site-to-Site Architecture:**
```
Network A (192.168.1.0/24) ←→ [tun_wg0] ←→ Network B (10.0.0.0/28)
```

- The tunnel is a **transparent bridge**
- Devices on Network A can reach devices on Network B
- Routing rules say "traffic to 10.0.0.0/28 goes through tun_wg0"
- No IP needed on the tunnel interface itself

**Think of it like**: A bridge connecting two islands - the bridge doesn't need its own address, it just connects the islands.

### Why tun_wg1 Has IP 10.0.0.17/28

**Remote Access Architecture:**
```
pfSense (10.0.0.17) ←→ [tun_wg1] ←→ Client 1 (10.0.0.18)
                                  ←→ Client 2 (10.0.0.19)
```

- The tunnel is a **gateway server**
- Clients connect TO 10.0.0.17
- Each client gets an IP from the 10.0.0.16-31 range
- The tunnel IP is the "server" address

**Think of it like**: A hotel with address 10.0.0.17, and each guest (client) gets their own room number (10.0.0.18, 10.0.0.19, etc.).

### Allowed IPs: Network Range vs Single IPs

**tun_wg0 - 10.0.0.0/28:**
- Means: "Route all traffic for the 10.0.0.0/28 network to this peer"
- Covers: 10.0.0.0 through 10.0.0.15 (16 IPs)
- Used for: Network-to-network routing

**tun_wg1 - 10.0.0.18/32:**
- Means: "Route traffic for exactly 10.0.0.18 to this peer"
- Covers: Only 10.0.0.18 (single IP)
- Used for: Device-specific routing

## Navigating pfSense to Review Your Configuration

### Step 1: View Tunnel Configuration

1. Go to **VPN** → **WireGuard** → **Tunnels**
2. Click on **tun_wg0** to see its configuration
3. Note the settings:
   - **Enable**: Should be checked
   - **Description**: "Home Tunnel"
   - **Listen Port**: 51820
   - **Address/Assignment**: Should be empty or "none"
   - **MTU**: 1420

4. Click on **tun_wg1** to see its configuration
5. Note the differences:
   - **Description**: "Remote users"
   - **Listen Port**: 51821
   - **Address/Assignment**: Should show "10.0.0.17/28" (this is the key difference!)
   - **MTU**: 1420

### Step 2: View Peer Configuration

1. Go to **VPN** → **WireGuard** → **Peers**
2. Filter or find peers for **tun_wg0**:
   - Look for "Beryll Peer"
   - Check **Allowed IPs**: Should be 10.0.0.0/28
   - Check **Endpoint**: May be empty or have an IP:port

3. Find peers for **tun_wg1**:
   - Look for "Remote-Android-R..." and "ARM-Laptop"
   - Check **Allowed IPs**: Should be 10.0.0.18/32 and 10.0.0.19/32
   - Check **Endpoint**: Likely empty (clients initiate)

### Step 3: Compare Settings

Create a mental or written comparison:
- Which tunnel has an IP assignment? (tun_wg1)
- Which tunnel uses network ranges for Allowed IPs? (tun_wg0)
- Which tunnel uses single IPs for Allowed IPs? (tun_wg1)
- How many peers does each have? (1 vs 2)

## What Each Setting Means

### Tunnel Settings Explained

**Enable**: Turns the tunnel on/off. Both should be enabled.

**Description**: Human-readable name. Helps identify purpose.

**Listen Port**: Port WireGuard listens on for incoming connections. Must be unique per tunnel.

**Address/Assignment**: 
- **Empty (tun_wg0)**: Site-to-site mode, no gateway IP needed
- **Has IP (tun_wg1)**: Remote access mode, gateway IP for clients

**MTU**: Maximum packet size. 1420 accounts for encryption overhead.

**Public Key**: Your tunnel's public key. Share this with peers.

**Private Key**: (Hidden) Your tunnel's private key. Never share this.

### Peer Settings Explained

**Description**: Name identifying this peer.

**Public Key**: Peer's public key. Must match their actual key.

**Allowed IPs**: 
- **What traffic to route**: Tells WireGuard which destination IPs should go through this peer
- **tun_wg0**: Network range (10.0.0.0/28) - route entire network
- **tun_wg1**: Single IPs (/32) - route to specific device

**Endpoint**: 
- **IP:port of peer**: Where to send encrypted traffic
- **Empty**: Peer will initiate connection (common for clients)
- **Has value**: Direct connection to peer (common for site-to-site)

**Persistent Keepalive**: 
- **What**: Sends periodic packets to keep connection alive
- **When needed**: If peer is behind NAT/firewall
- **Your setup**: May or may not be configured

**Pre-shared Key**: 
- **Optional**: Additional encryption layer
- **Your setup**: May or may not be configured

## Firewall and Routing Considerations

### Firewall Rules Needed

**For tun_wg0 (Site-to-Site):**
- Allow traffic between your LAN (192.168.1.0/24) and remote network (10.0.0.0/28)
- Rules on both LAN and WireGuard interfaces

**For tun_wg1 (Remote Access):**
- Allow traffic from WireGuard network (10.0.0.16/28) to LAN (192.168.1.0/24)
- Allow return traffic from LAN to WireGuard network
- May need rules for specific services (e.g., allow access to Jellyfin)

### Routing

**tun_wg0:**
- Static route: "Traffic to 10.0.0.0/28 → route through tun_wg0"
- Usually configured automatically by pfSense WireGuard package

**tun_wg1:**
- Clients get routes via WireGuard configuration
- pfSense routes traffic from 10.0.0.16/28 to LAN automatically

## Troubleshooting Your Configuration

### If Handshake is "Never"

**Possible causes:**
1. **Peer is offline**: Device/router not running WireGuard
2. **Firewall blocking**: Port not open or forwarded
3. **Configuration mismatch**: Public keys don't match
4. **Network issue**: Can't reach endpoint

**How to check:**
1. Verify peer is online and WireGuard is running
2. Check firewall rules allow port 51820/51821
3. Verify public keys match on both sides
4. Test network connectivity to endpoint

### If Data Transfer is 0 B

**Possible causes:**
1. **No handshake**: Can't transfer data without connection
2. **No traffic**: Tunnels connected but no data being sent
3. **Routing issue**: Traffic not being routed through tunnel

**How to check:**
1. First fix handshake issue
2. Generate some traffic (ping, access service)
3. Check routing tables and firewall rules

## Questions to Explore in pfSense

As you review your configuration, consider these questions:

1. **Why does tun_wg0 have no IP but tun_wg1 has 10.0.0.17/28?**
   - Answer: Site-to-site vs remote access architecture difference

2. **Why does tun_wg0 use 10.0.0.0/28 for Allowed IPs but tun_wg1 uses /32?**
   - Answer: Network range routing vs device-specific routing

3. **What firewall rules exist for each tunnel?**
   - Check: Firewall → Rules → [Interface] to see what's allowed

4. **What routes are configured?**
   - Check: System → Routing to see static routes

5. **Why are endpoints (none) for tun_wg1 peers?**
   - Answer: Clients initiate connection, so endpoint not needed on server side

## Next Steps

1. **Review your actual pfSense configuration** using the navigation guide above
2. **Compare what you see** with the analysis in this document
3. **Document any differences** or questions you have
4. **Check firewall rules** to understand what traffic is allowed
5. **Review routing** to see how traffic flows

For adding new clients, see [adding-clients.md](adding-clients.md).


