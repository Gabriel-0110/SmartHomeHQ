# 01-Network

## Overview

This folder contains network infrastructure documentation for the SmartHomeHQ dual-router architecture.

## Architecture Summary

- **Dual-router design**: Router #1 (Trusted) + Router #2 (IoT) in intentional double NAT
- **Two subnets**: 192.168.1.0/24 (Trusted) and 192.168.2.0/24 (IoT)
- **Mac mini is DUAL-HOMED**: One interface on each router for HA/Scrypted IoT access
- **NAT is NOT a security boundary**: Endpoint firewalls provide enforcement
- **No VLANs**: CR1000A routers do not support VLANs

## Contents

### Reference of Truth (Finalized)
- **PHASE1-START-HERE.md**: Entry point for Phase 1 implementation
- **phase1a-implementation-guide.md**: Network foundation and device migration
- **phase1b-implementation-guide.md**: Endpoint and platform hardening

### Supporting Documentation
- **current-network.md**: Current network state and target topology
- **phase1-decision-guide.md**: Architecture decisions and rationale
- **phase1-preflight-checklist.md**: Pre-implementation checklist
- **network-template.md**: Template for documenting network segments

### Router Configuration
- **router/**: Router configuration backups and audit results

## Network Segments

| Segment | Subnet | Purpose | Devices |
|---------|--------|---------|--------|
| Trusted | 192.168.1.0/24 | Control plane | Mac mini, HA, HomeKit hubs, phones, PCs |
| IoT | 192.168.2.0/24 | IoT and cameras | Hue, Aqara, Echo, cameras |

## Security Principles

- **Dual-homing**: Mac mini has interfaces on both subnets
- **Endpoint firewalls**: Primary enforcement (macOS/Windows firewalls)
- **Camera separation**: Internet blocked via Access Control
- **No magic isolation**: NAT reduces exposure but does not enforce security

## Documentation Standards

- Use network diagrams for visual representation
- Document all firewall rules with justification
- Keep IP address assignments updated
- Include device MAC addresses for critical systems
