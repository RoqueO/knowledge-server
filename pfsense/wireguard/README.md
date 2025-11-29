# WireGuard VPN Documentation

This documentation provides reference information for the WireGuard VPN setup in pfSense, including two distinct tunnel configurations: a site-to-site tunnel (tun_wg0) and a remote access tunnel (tun_wg1).

## Overview

WireGuard is a modern, high-performance VPN protocol that provides secure, encrypted connections between networks and devices. This pfSense setup uses the WireGuard package to manage VPN tunnels for both site-to-site connectivity and remote access.

**Key Features:**
- Site-to-site VPN connection (tun_wg0) for connecting two networks
- Remote access VPN (tun_wg1) for individual devices to connect securely
- High-performance encryption with minimal overhead
- Simple key-based authentication
- Easy peer management through pfSense web interface

## Architecture

### Tunnel Types

**tun_wg0 (Home Tunnel - Site-to-Site)**
- **Purpose**: Connects two networks together (site-to-site)
- **Configuration**: No local IP assignment, acts as a bridge
- **Peers**: 1 peer (Beryll Peer)
- **Network**: 10.0.0.0/28
- **Use Case**: Permanent connection between two locations

**tun_wg1 (Remote users - Remote Access)**
- **Purpose**: Allows individual devices to connect remotely
- **Configuration**: Has local IP assignment (10.0.0.17/28)
- **Peers**: 2 peers (Remote-Android-R..., ARM-Laptop)
- **Networks**: 10.0.0.18/32, 10.0.0.19/32 (one IP per peer)
- **Use Case**: Mobile devices and laptops connecting from anywhere

## Quick Reference

### Tunnel Configuration

| Tunnel | Description | Port | MTU | IP Assignment | Peers |
|--------|-------------|------|-----|---------------|-------|
| tun_wg0 | Home Tunnel | 51820 | 1420 | (none) | 1 |
| tun_wg1 | Remote users | 51821 | 1420 | 10.0.0.17/28 | 2 |

### Network Addressing

**tun_wg0 (Site-to-Site):**
- **Allowed IPs**: 10.0.0.0/28
- **Local IP**: None (site-to-site mode)

**tun_wg1 (Remote Access):**
- **Tunnel IP**: 10.0.0.17/28
- **Peer IPs**: 
  - Remote-Android-R...: 10.0.0.18/32
  - ARM-Laptop: 10.0.0.19/32

### Package Information
- **Package**: pfSense-pkg-WireGuard
- **Version**: 0.2.9_5

## Documentation Structure

1. **[how-it-works.md](how-it-works.md)** - Conceptual explanation of WireGuard, key concepts, and how site-to-site vs remote access patterns work
2. **[configuration-analysis.md](configuration-analysis.md)** - Detailed walkthrough of pfSense WireGuard settings, comparison of tun_wg0 vs tun_wg1, and navigation guide
3. **[adding-clients.md](adding-clients.md)** - Step-by-step guide for adding new peers to tun_wg1, client setup instructions, and troubleshooting

## Current Status

### tun_wg0 (Home Tunnel)
- **Status**: Active (green indicator)
- **Peers**: 1
- **Latest Handshake**: Never (no recent activity)
- **Data Transfer**: 0 B RX / 0 B TX

### tun_wg1 (Remote users)
- **Status**: Active (green indicator)
- **Peers**: 2
- **Latest Handshake**: Never (no recent activity)
- **Data Transfer**: 0 B RX / 0 B TX

**Note**: "Never" handshakes and 0 B transfer indicate the tunnels are configured but may not be actively connected or in use at the time of status check.

## Key Differences: tun_wg0 vs tun_wg1

| Aspect | tun_wg0 (Site-to-Site) | tun_wg1 (Remote Access) |
|--------|------------------------|-------------------------|
| **Purpose** | Connect two networks | Connect individual devices |
| **IP Assignment** | None | 10.0.0.17/28 |
| **Peer Count** | 1 (network-to-network) | 2+ (device-to-server) |
| **IP per Peer** | Network range (10.0.0.0/28) | Single IP (/32) |
| **Use Case** | Permanent network link | On-demand device access |
| **Configuration** | Both sides are servers | Server (pfSense) + clients |

## Questions & Answers

### Common Questions

*Add your questions and answers here as you explore the configuration.*

**Example Questions to Explore:**
- Why does tun_wg0 have no IP assignment while tun_wg1 has 10.0.0.17/28?
- What's the difference between site-to-site and remote access configurations?
- How do I add a new peer to tun_wg1?
- What firewall rules are needed for WireGuard tunnels?
- How do I troubleshoot connection issues?

### Troubleshooting

**Common Issues:**
- **No handshake**: Check peer configuration, firewall rules, and network connectivity
- **Connection timeout**: Verify port forwarding and endpoint addresses
- **No data transfer**: Check routing rules and allowed IPs configuration

## Prerequisites

- pfSense router with WireGuard package installed
- Access to pfSense web interface
- Basic understanding of networking concepts
- Static public IP address (recommended for site-to-site)

## Implementation Order

1. **Understand Current Setup** - Review configuration-analysis.md to understand existing tunnels
2. **Learn WireGuard Concepts** - Read how-it-works.md for background knowledge
3. **Add New Clients** - Use adding-clients.md to expand tun_wg1 with additional devices
4. **Test Connectivity** - Verify tunnels are working and troubleshoot as needed


