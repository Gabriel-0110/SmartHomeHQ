# 01-Network

## Overview

This folder contains network infrastructure documentation including topology, VLANs, firewall rules, and IoT network segregation.

## Contents

- **network-topology.md**: Network diagram and overall architecture
- **vlan-configuration.md**: VLAN setup and device assignments
- **firewall-rules.md**: Firewall rules and access control lists
- **dhcp-reservations.md**: Static IP assignments and DHCP reservations
- **dns-configuration.md**: Local DNS setup and custom domains
- **vpn-setup.md**: VPN configuration for remote access
- **wifi-configuration.md**: WiFi networks and guest access

## Network Architecture

### Network Segments

- **Main Network**: Trusted devices (computers, phones, tablets)
- **IoT Network**: Smart home devices (isolated VLAN)
- **Cameras**: Security cameras (dedicated VLAN)
- **Guest Network**: Visitor access (internet only)
- **Management**: Infrastructure management (switches, APs, servers)

## Security Principles

- **Segregation**: IoT devices isolated from main network
- **Least Privilege**: Devices only access required services
- **No Internet**: Critical devices operate locally when possible
- **Monitoring**: Network traffic monitoring and alerts

## Documentation Standards

- Use network diagrams for visual representation
- Document all firewall rules with justification
- Keep IP address assignments updated
- Include device MAC addresses for critical systems
- Document port forwarding and NAT rules

## Tools and Resources

- Network diagramming tools
- IP address management (IPAM)
- Network monitoring tools
- Firewall management interface
