# netbootxyz-dnsmasq-proxy

This Docker Compose setup leverages [netbootxyz](https://netboot.xyz) to manage network-based OS installations across multiple machines without physical installation media. The configuration includes `dnsmasq` in proxy mode, allowing PXE booting within your network while delegating IP assignments to your primary DHCP server.

## Overview

This setup includes two main components:

1. **netbootxyz** - Provides a comprehensive and flexible way to deploy operating systems via network boot. With this setup, you can customize boot options and select from various OS distributions and utilities available through `netbootxyz`.

2. **dnsmasq** - Acts as a DHCP proxy in this setup, handling PXE boot requests only. It communicates with the network’s existing DHCP server to provide IP addresses and enables PXE clients to boot via `netbootxyz`.

## Setup

### Prerequisites

- Docker and Docker Compose should be installed on your system.
- Create a `dnsmasq.conf` file in the same directory as `docker-compose.yml` to define DNS and DHCP proxy settings.

### Example `docker-compose.yml`

```yaml
version: '3.8'
 
services:
  netbootxyz:
    image: netbootxyz/netbootxyz:latest
    container_name: netbootxyz
    network_mode: host
    cap_add:
      - NET_ADMIN
    volumes:
      - ./dnsmasq.conf:/etc/dnsmasq.conf
    restart: unless-stopped

```
### Example `dnsmasq.conf`

```
# Disable DNS functionality
port=0

# Enable DHCP logging
log-dhcp

# Enable TFTP and set the root directory for boot files
enable-tftp
tftp-root=/config/menus/remote

# Set DHCP proxy mode
dhcp-range=192.168.1.0,proxy

# Define PXE services for BIOS and UEFI
pxe-service=X86PC, "Boot from network", netboot.xyz.kpxe
pxe-service=X86-64_EFI, "Boot from network (UEFI)", netboot.xyz.efi

# Prevent DHCP overrides
dhcp-no-override
```

### Explanation of Key `dnsmasq` Settings

- **port=0**: Disables DNS functionality.
- **log-dhcp**: Enables DHCP logging.
- **enable-tftp** and **tftp-root**: Enables TFTP server and sets the root directory.
- dhcp-range=192.168.1.0,proxy**: Configures `dnsmasq` in DHCP proxy mode, forwarding DHCP requests while intercepting PXE boot requests. Please edit `dhcp-range=` to reflect your network.
- **pxe-service**: Specifies PXE boot for BIOS (X86PC) and UEFI (X86-64_EFI) clients.
- **dhcp-no-override**: Prevents overriding existing PXE configurations.

### Usage

```
docker-compose up -d
```