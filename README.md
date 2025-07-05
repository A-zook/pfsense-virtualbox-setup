# pfSense Virtual Lab Setup Guide

A comprehensive guide for setting up a pfSense firewall with DMZ and internal network segmentation using VirtualBox.

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation Process](#installation-process)
- [Configuration](#configuration)
- [Testing and Validation](#testing-and-validation)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)
- [Contributing](#contributing)

## Overview

This guide demonstrates how to create a virtualized network environment using pfSense as a firewall/router with proper network segmentation. The setup includes:

- **WAN Interface**: Simulated internet connection
- **LAN Interface**: Internal private network for applications
- **DMZ Interface**: Isolated network for public-facing services

Perfect for learning network security, testing firewall rules, and understanding network segmentation concepts.

## Architecture

### Network Topology
```
[Internet/NAT] <--> [pfSense WAN] | [pfSense LAN] <--> [Internal App VM]
                                  |
                                  | [pfSense DMZ] <--> [DMZ Server]
```

### Interface Mapping
| Interface | VM | VirtualBox Network | Purpose |
|-----------|----|--------------------|---------|
| WAN | pfSense | NAT | Internet simulation |
| LAN | pfSense, AppVM | intnet | Private internal network |
| DMZ (optional) | pfSense, DMZVM | dmznet | Isolated public-facing zone |

## Prerequisites

### Software Requirements
- **VirtualBox** (latest version recommended)
- **pfSense ISO** - Download from [pfSense.org](https://www.pfsense.org/download/)
- **Guest OS ISOs** (Ubuntu Server, Windows, etc.)

### System Requirements
- **Host Machine**: Minimum 8GB RAM, 4 CPU cores
- **pfSense VM**: 2GB RAM, 1 CPU, 20GB storage
- **Guest VMs**: 1-2GB RAM each, depending on OS

## Installation Process

### Step 1: Create pfSense VM

1. **Create New VM**
   ```
   Name: pfSense
   Type: BSD
   Version: FreeBSD (64-bit)
   RAM: 2048 MB
   Storage: 20 GB (dynamically allocated)
   ```

2. **Configure Network Adapters**
   - **Adapter 1 (WAN)**:
     - Enable Network Adapter: ✓
     - Attached to: NAT
   - **Adapter 2 (LAN)**:
     - Enable Network Adapter: ✓
     - Attached to: Internal Network
     - Name: `intnet`
   - **Adapter 3 (DMZ)** *(optional)*:
     - Enable Network Adapter: ✓
     - Attached to: Internal Network
     - Name: `dmznet`

3. **Attach pfSense ISO**
   - Settings → Storage → Controller: IDE
   - Add pfSense ISO to optical drive

### Step 2: Create Internal Application VM

1. **Create VM** (Ubuntu/Windows)
   ```
   Name: AppVM
   Type: Linux/Windows
   Version: Ubuntu (64-bit) or Windows 10 (64-bit)
   RAM: 2048 MB
   Storage: 20 GB
   ```

2. **Network Configuration**
   - **Adapter 1**:
     - Attached to: Internal Network
     - Name: `intnet`

### Step 3: Create DMZ VM *(Optional)*

1. **Create VM** (Ubuntu Server recommended)
   ```
   Name: DMZVM
   Type: Linux
   Version: Ubuntu (64-bit)
   RAM: 1024 MB
   Storage: 20 GB
   ```

2. **Network Configuration**
   - **Adapter 1**:
     - Attached to: Internal Network
     - Name: `dmznet`

### Step 4: Install pfSense

1. **Boot pfSense VM** and follow installation wizard
2. **Interface Assignment**:
   - Identify interfaces by MAC addresses (check VirtualBox settings)
   - Assign interfaces:
     ```
     WAN: em0 (first adapter - NAT)
     LAN: em1 (second adapter - intnet)
     DMZ: em2 (third adapter - dmznet)
     ```
3. **Complete Installation** and reboot
4. **Remove ISO** from optical drive

## Configuration

### Initial pfSense Setup

1. **Access Web Interface**
   - From AppVM, open browser
   - Navigate to: `https://192.168.1.1`
   - Default credentials: `admin` / `pfsense`

2. **Run Setup Wizard**
   - Set hostname: `pfsense-lab`
   - Configure timezone
   - Set DNS servers (8.8.8.8, 8.8.4.4)
   - Configure LAN IP range (default: 192.168.1.0/24)

3. **Change Default Password**
   - System → User Manager
   - Edit admin user
   - Set strong password

### Network Configuration

#### LAN Interface
```
IP Address: 192.168.1.1
Subnet Mask: 255.255.255.0 (/24)
DHCP Range: 192.168.1.100-192.168.1.199
```

#### DMZ Interface *(if configured)*
```
IP Address: 192.168.2.1
Subnet Mask: 255.255.255.0 (/24)
DHCP Range: 192.168.2.100-192.168.2.199
```

### Firewall Rules

#### WAN Rules
```
Protocol: TCP
Port: 80, 443 (HTTP/HTTPS)
Action: Allow
Source: Any
Destination: DMZ Network
```

#### LAN Rules
```
Protocol: Any
Port: Any
Action: Allow
Source: LAN Network
Destination: Any
```

#### DMZ Rules
```
Protocol: TCP
Port: 22, 80, 443
Action: Allow
Source: Any
Destination: DMZ Network

Protocol: Any
Port: Any
Action: Block
Source: DMZ Network
Destination: LAN Network
```

## Testing and Validation

### Connectivity Tests

1. **Internet Access from LAN**
   ```bash
   # From AppVM
   ping 8.8.8.8
   curl -I https://google.com
   ```

2. **DMZ Isolation Test**
   ```bash
   # From DMZVM - should fail
   ping 192.168.1.1
   ```

3. **DMZ Service Access**
   ```bash
   # Install web server on DMZVM
   sudo apt update && sudo apt install apache2
   
   # Test from AppVM
   curl http://192.168.2.100
   ```

### pfSense Monitoring

1. **Traffic Analysis**
   - Status → Traffic Graph
   - Monitor interface utilization

2. **Firewall Logs**
   - Status → System Logs → Firewall
   - Check blocked/allowed connections

3. **DHCP Leases**
   - Status → DHCP Leases
   - Verify VM IP assignments

## Troubleshooting

### Common Issues

#### VMs Not Getting IP Addresses
- Check DHCP service: Services → DHCP Server
- Verify interface assignments
- Restart network services on guest VMs

#### Cannot Access pfSense Web Interface
- Verify LAN interface IP: `ifconfig em1`
- Check firewall rules on LAN interface
- Ensure AppVM is on correct network (intnet)

#### Interface Assignment Problems
- Use `ifconfig` to identify interface names
- Check VirtualBox network adapter order
- Reassign interfaces: Option 1 from pfSense console

### Network Interface Identification

```bash
# From pfSense console
ifconfig

# Common interface names:
# em0, em1, em2 (Intel)
# vtnet0, vtnet1, vtnet2 (VirtIO)
# re0, re1, re2 (Realtek)
```

## Advanced Configuration

### Optional Enhancements

#### VPN Server Setup
- Configure OpenVPN server
- Generate client certificates
- Test remote access

#### Intrusion Detection
- Install Suricata package
- Configure IDS rules
- Monitor security events

#### Content Filtering
- Install pfBlockerNG
- Configure DNS filtering
- Block malicious domains

#### High Availability
- Configure CARP (Virtual IP)
- Set up pfSync for state synchronization
- Test failover scenarios

### Performance Optimization

#### Resource Allocation
```
pfSense VM: 2GB RAM, 2 CPU cores
Guest VMs: Adjust based on workload
Enable VT-x/AMD-V in BIOS
```

#### Network Performance
- Enable VirtIO network drivers
- Adjust network buffer sizes
- Monitor interface statistics

## Security Best Practices

### pfSense Hardening
- Change default passwords
- Enable two-factor authentication
- Regular security updates
- Disable unnecessary services

### Network Segmentation
- Implement least privilege access
- Use VLANs for additional segmentation
- Regular firewall rule audits
- Monitor network traffic

### Backup and Recovery
- Regular configuration backups
- Test restore procedures
- Document configuration changes
- Version control for configs

## Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Reporting Issues
- Use GitHub Issues
- Provide detailed description
- Include configuration details
- Add screenshots if helpful

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- pfSense team for the excellent firewall solution
- VirtualBox team for virtualization platform
- Community contributors and testers

---

**Note**: This setup is intended for educational and testing purposes. For production environments, consider additional security measures and proper hardware sizing.
