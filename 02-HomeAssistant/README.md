# 02-HomeAssistant

## Overview

This folder contains Home Assistant configuration, automations, integrations, and setup documentation.

## Contents

- **installation.md**: Home Assistant installation and initial setup
- **integrations.md**: List of all integrations and add-ons
- **automations.md**: Documentation of all automations
- **scripts.md**: Custom scripts and helpers
- **lovelace-ui.md**: Dashboard configuration and layouts
- **themes.md**: UI themes and customization
- **blueprints.md**: Automation blueprints
- **custom-components.md**: HACS and custom integrations

## Home Assistant Setup

### Installation Details
- **Platform**: UTM VM on Mac mini
- **OS**: Home Assistant Operating System
- **Version**: [Document current version]
- **Resources**: [CPU, RAM, Storage allocation]

### Key Integrations

Document all major integrations:
- HomeKit integration
- Device integrations (Zigbee, Z-Wave, WiFi, etc.)
- Cloud services (weather, TTS, etc.)
- Media players
- Energy monitoring
- Security systems

## Configuration Structure

```
configuration.yaml       # Main configuration file
automations.yaml        # Automation definitions
scripts.yaml           # Custom scripts
scenes.yaml            # Scene definitions
secrets.yaml           # Secrets (NOT committed to git)
```

## Automation Documentation

For each automation, document:
- **Purpose**: What it does and why
- **Trigger**: What starts the automation
- **Conditions**: When it should run
- **Actions**: What it does
- **Testing**: How to verify it works

## Backup Strategy

- Document backup schedule and procedures
- List backup locations
- Recovery testing procedures

## Maintenance

- Keep documentation updated with configuration changes
- Document all integration additions or removals
- Note any performance issues or optimizations
- Track Home Assistant version upgrades
