# SmartHomeHQ

## Overview

SmartHomeHQ is a comprehensive documentation and configuration repository for managing a HomeKit-first smart home setup. This repository serves as the central knowledge base for all smart home infrastructure, devices, and configurations.

### Technology Stack

- **Home Assistant**: Main home automation hub running on a UTM VM on Mac mini
- **Homebridge**: Enables HomeKit integration for non-native devices and niche integrations
- **Scrypted**: Advanced camera management and NVR solution
- **HomeKit**: Primary smart home ecosystem for device control and automation

> **Note**: This repository contains documentation only—no application code.

## Repository Structure

```
SmartHomeHQ/
├── 00-Inventory/          # Device inventory, purchase records, and hardware tracking
├── 01-Network/            # Network topology, VLANs, firewall rules, and IoT network setup
├── 02-HomeAssistant/      # Home Assistant configuration, automations, and integrations
├── 03-Scrypted/           # Scrypted camera setup, plugins, and NVR configuration
├── 04-Homebridge/         # Homebridge plugins, accessories, and HomeKit bridges
├── 05-Dashboards/         # Dashboard configurations, layouts, and UI customization
├── 06-Security/           # Security policies, access control, and hardening guidelines
└── 99-Exports/            # Configuration exports, backups, and snapshots
```

## Quick Start

1. Browse the folders above to find documentation for specific components
2. Check the [Roadmap](roadmap.md) for planned improvements and current status
3. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on updating documentation
4. Review [SECURITY.md](SECURITY.md) for security policies and vulnerability reporting

## Documentation Standards

- Use Markdown (.md) format for all documentation
- Keep device inventory updated in CSV format
- Include diagrams for network topology and system architecture
- Document all configuration changes with date and reason
- Maintain version history for critical configurations

## Getting Help

- Check existing documentation in relevant folders
- Review the roadmap for planned features
- Open an issue for questions or suggestions

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.