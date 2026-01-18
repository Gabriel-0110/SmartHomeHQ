# 99-Exports

## Overview

This folder contains configuration exports, backups, and snapshots from various smart home systems. These exports serve as version history and recovery points.

## Contents

- **home-assistant/**: Home Assistant configuration exports
- **homebridge/**: Homebridge configuration backups
- **scrypted/**: Scrypted configuration exports
- **network/**: Network device configurations
- **snapshots/**: Point-in-time system snapshots

## Export Schedule

### Daily Exports
- Home Assistant configuration
- Homebridge configuration
- Critical automation rules

### Weekly Exports
- Full system backups
- Network device configurations
- Dashboard layouts

### Monthly Exports
- Complete system snapshots
- Historical configuration archives
- Performance baselines

## File Naming Convention

Use consistent naming for exports:
```
<system>-<type>-<YYYY-MM-DD>.ext

Examples:
homeassistant-config-2026-01-18.yaml
homebridge-backup-2026-01-18.json
network-switch-config-2026-01-18.cfg
```

## Backup Types

### Configuration Exports
- YAML configuration files
- JSON configuration files
- Device-specific configs

### Database Backups
- Home Assistant database
- History and statistics
- Long-term data

### Documentation Snapshots
- Documentation state at specific times
- Network diagrams
- Architecture documents

## Security Considerations

⚠️ **IMPORTANT**: 
- **NEVER** commit files containing passwords, API keys, or secrets
- Use placeholders in example exports
- Store sensitive backups in secure, encrypted locations
- Add sensitive export files to .gitignore

### Sanitizing Exports

Before committing exports:
1. Remove all passwords and API keys
2. Replace private IPs with placeholders
3. Remove MAC addresses
4. Remove personal information
5. Review for any sensitive data

## Storage Locations

### Git Repository
- Sanitized configuration examples
- Template configurations
- Documentation snapshots

### Secure Backup Location
- Full configuration backups with secrets
- Database backups
- Encrypted archives

### Offsite Backup
- Monthly full system backups
- Disaster recovery copies
- Long-term archives

## Recovery Procedures

### Configuration Restore
1. Identify required backup version
2. Stop relevant services
3. Restore configuration files
4. Verify configuration
5. Restart services
6. Test functionality

### System Restore
1. Identify backup snapshot
2. Prepare restore environment
3. Restore system image or files
4. Update network configurations
5. Verify all services
6. Test critical functions

## Version Control

- Use git for configuration tracking
- Tag major configuration changes
- Document reasons for changes
- Link to relevant issues or documentation

## Retention Policy

- **Daily backups**: Keep for 7 days
- **Weekly backups**: Keep for 4 weeks
- **Monthly backups**: Keep for 12 months
- **Major version snapshots**: Keep indefinitely

## Verification

### Regular Testing
- Test restore procedures quarterly
- Verify backup integrity monthly
- Validate export completeness
- Check backup accessibility

### Automation
- Automated backup verification
- Integrity checking
- Alert on backup failures
- Backup size monitoring

## Documentation

For each export, document:
- Date and time of export
- System state (versions, configurations)
- Reason for export (scheduled, pre-change, incident)
- Verification status
- Storage location
