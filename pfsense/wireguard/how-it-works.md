# How WireGuard Works

This document explains the fundamental concepts of WireGuard and how it applies to your pfSense setup with tun_wg0 (site-to-site) and tun_wg1 (remote access).

## What is WireGuard?

WireGuard is a modern VPN protocol that creates encrypted tunnels between devices or networks. It's designed to be simpler, faster, and more secure than traditional VPN protocols like OpenVPN or IPSec.

**Key Advantages:**
- **Fast**: Minimal overhead, high throughput
- **Simple**: Small codebase, easier to audit
- **Secure**: Modern cryptography (ChaCha20, Poly1305, Curve25519)
- **Efficient**: Low CPU usage, good for mobile devices

## Core Concepts

### 1. Tunnels

A **tunnel** is a virtual network interface that encrypts and routes traffic. In your setup:
- `tun_wg0` is one tunnel (site-to-site)
- `tun_wg1` is another tunnel (remote access)

Each tunnel is independent and can have different configurations, peers, and purposes.

### 2. Peers

A **peer** is another device or network that can connect through the tunnel. Each peer has:
- **Public Key**: Like a username (shared with others)
- **Private Key**: Like a password (kept secret)
- **Endpoint**: The IP address and port where the peer can be reached
- **Allowed IPs**: Which IP addresses/ranges this peer can access

### 3. Public and Private Keys

WireGuard uses **public-key cryptography**:

- **Private Key**: Generated on each device, never shared
- **Public Key**: Derived from private key, shared with peers

**How it works:**
1. Your pfSense generates a private key (secret)
2. It derives a public key from the private key
3. You share the public key with peers
4. Peers share their public keys with you
5. Each side can encrypt messages that only the other can decrypt

**Security**: Even if someone intercepts your public key, they can't decrypt your traffic without the private key.

### 4. Handshakes

A **handshake** is when two peers establish a connection and exchange encrypted keys. WireGuard uses a "noise protocol" for this:

- **Latest Handshake**: Shows when peers last successfully connected
- **"Never"**: Means no successful connection yet, or peers are offline
- **Regular handshakes**: WireGuard re-establishes keys periodically for security

### 5. Allowed IPs

**Allowed IPs** define what traffic should be routed through the WireGuard tunnel:

- **0.0.0.0/0**: Route all internet traffic through the tunnel (full VPN)
- **10.0.0.0/28**: Route only traffic to this network range
- **10.0.0.18/32**: Route only traffic to this specific IP

This is how WireGuard knows which packets to encrypt and send through the tunnel.

## Site-to-Site vs Remote Access

### Site-to-Site (tun_wg0)

**Purpose**: Connect two entire networks together permanently.

**How it works:**
1. Two routers (like your pfSense and another router) each run WireGuard
2. They connect to each other as peers
3. Traffic between the two networks is encrypted and routed through the tunnel
4. Devices on either network can communicate as if they're on the same LAN

**Characteristics:**
- Both sides are "servers" (both listen for connections)
- No local IP assignment needed on the tunnel interface
- Network-to-network communication
- Usually permanent/always-on connection

**Example**: Your home network (192.168.1.0/24) connects to another location's network (10.0.0.0/28) through tun_wg0.

### Remote Access (tun_wg1)

**Purpose**: Allow individual devices to connect to your network from anywhere.

**How it works:**
1. Your pfSense runs a WireGuard server (tun_wg1)
2. Mobile devices/laptops run WireGuard clients
3. Clients connect to your pfSense server
4. Each client gets its own IP address (10.0.0.18, 10.0.0.19, etc.)
5. Clients can access your network resources securely

**Characteristics:**
- Server (pfSense) + multiple clients (devices)
- Tunnel interface has an IP address (10.0.0.17/28)
- Each peer gets a unique IP (/32 means single IP)
- On-demand connections (connect when needed)
- Clients can be anywhere with internet access

**Example**: Your Android phone connects to tun_wg1, gets IP 10.0.0.18, and can access your home network.

## Network Addressing

### Why tun_wg0 Has No IP Assignment

In site-to-site mode, the tunnel interface doesn't need its own IP because:
- It's a **bridge** between two networks
- Traffic is routed based on destination networks, not tunnel IPs
- The tunnel just forwards packets between networks

### Why tun_wg1 Has IP 10.0.0.17/28

In remote access mode, the tunnel interface needs an IP because:
- It acts as a **gateway** for connected clients
- Clients need to know where to send traffic
- The tunnel IP (10.0.0.17) is the "server" address clients connect to
- The /28 subnet (10.0.0.16-31) provides IPs for multiple clients

### IP Address Allocation

**tun_wg1 subnet: 10.0.0.17/28**
- **Range**: 10.0.0.16 - 10.0.0.31 (16 IPs total)
- **Network**: 10.0.0.16
- **Broadcast**: 10.0.0.31
- **Tunnel IP**: 10.0.0.17 (pfSense server)
- **Available for clients**: 10.0.0.18 - 10.0.0.30 (13 IPs)

**Current allocation:**
- 10.0.0.18 → Remote-Android-R...
- 10.0.0.19 → ARM-Laptop
- 10.0.0.20-30 → Available for new clients

## Routing and Traffic Flow

### How Traffic Gets Routed

1. **Device sends packet** → Checks destination IP
2. **Matches Allowed IPs?** → If yes, route through WireGuard tunnel
3. **Encrypt packet** → Using peer's public key
4. **Send through tunnel** → To peer's endpoint
5. **Peer receives** → Decrypts using private key
6. **Forward to destination** → On peer's network

### Example: Remote Access (tun_wg1)

**Scenario**: Android phone (10.0.0.18) wants to access Jellyfin (192.168.1.12)

1. Phone sends packet to 192.168.1.12
2. Phone's WireGuard config has Allowed IPs: 192.168.1.0/24
3. Packet matches, so it's encrypted and sent to pfSense (tun_wg1)
4. pfSense receives packet, decrypts it
5. pfSense routes packet to 192.168.1.12 on LAN
6. Response follows reverse path

### Example: Site-to-Site (tun_wg0)

**Scenario**: Device on your LAN (192.168.1.50) wants to access device on remote network (10.0.0.5)

1. Device sends packet to 10.0.0.5
2. pfSense routing table sees 10.0.0.0/28 should go through tun_wg0
3. Packet is encrypted and sent to remote peer
4. Remote router receives, decrypts, forwards to 10.0.0.5
5. Response follows reverse path

## Security Model

### Encryption

WireGuard uses:
- **ChaCha20**: Symmetric encryption (fast, secure)
- **Poly1305**: Authentication (prevents tampering)
- **Curve25519**: Key exchange (establishes shared secret)

### Key Management

- **Private keys**: Never leave the device, never shared
- **Public keys**: Safe to share, used for encryption
- **Pre-shared keys (optional)**: Additional layer of security

### Connection Security

- **No handshake = no connection**: If handshake is "never", peers aren't connected
- **Automatic rekeying**: Keys are rotated periodically
- **Perfect forward secrecy**: Old keys can't decrypt past traffic

## MTU (Maximum Transmission Unit)

**Your setup uses MTU 1420**

- **Standard Ethernet MTU**: 1500 bytes
- **WireGuard overhead**: ~80 bytes (encryption headers)
- **Effective MTU**: 1420 bytes (1500 - 80)

**Why it matters:**
- Packets larger than MTU get fragmented
- Fragmentation can cause performance issues
- Setting MTU to 1420 prevents fragmentation

## Listen Ports

Each tunnel listens on a different port:
- **tun_wg0**: Port 51820
- **tun_wg1**: Port 51821

**Why different ports?**
- Allows multiple tunnels on the same server
- Each tunnel needs its own port for incoming connections
- Ports must be forwarded in firewall/router if behind NAT

## Understanding Your Status Screen

From your pfSense status screen:

**"Latest Handshake: never"**
- Means peers haven't successfully connected yet
- Could mean: peers are offline, configuration issue, or firewall blocking

**"RX: 0 B / TX: 0 B"**
- No data has been transferred
- Normal if peers aren't connected or no traffic is flowing

**"Endpoint: (none)"**
- Peer hasn't established a connection yet
- Will show IP:port once connected

**Green up arrow**
- Tunnel interface is active and configured
- Doesn't mean peers are connected, just that the tunnel is ready

## Next Steps

Now that you understand how WireGuard works:
1. Review your specific configuration in [configuration-analysis.md](configuration-analysis.md)
2. Learn how to add new clients in [adding-clients.md](adding-clients.md)
3. Explore your pfSense settings to see these concepts in action


