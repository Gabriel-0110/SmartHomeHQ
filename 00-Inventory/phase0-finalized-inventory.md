# Phase 0: Finalized Device Inventory

**Date:** 2026-01-18
**Status:** Initial inventory from HA screenshots + device-inventory.csv

---

## Infrastructure Components

| Component | Type | Platform | Location | Purpose |
|-----------|------|----------|----------|---------|
| Home Assistant | Hub/Controller | UTM VM on Mac mini M4 | Primary | Central automation & device management |
| Scrypted | Camera Hub | Scrypted Host | Primary | Camera RTSP streams + HKSV |
| Homebridge | Bridge | Homebridge Host | Primary | Niche device HomeKit exposure |
| Aqara Hub M2 | Zigbee Hub | Hardware | Multiple locations | Aqara Zigbee devices |
| Philips Hue Bridge | Zigbee Hub | Hardware | - | Hue lighting ecosystem |

---

## Devices by Area

### Bedroom
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Vanity Light | Philips Hue | - | Zigbee | Hue Bridge | Yes | Adaptive Lighting |
| TV Plug | Kasa | - | WiFi | Home Assistant | Yes | Power control |
| Air Purifier Plug | Kasa | - | WiFi | Home Assistant | Yes | Schedules + presence |
| TV Light Strip | Govee | - | WiFi | Home Assistant | Yes | TV mode sync |
| Safe Contact Sensor | Aqara | Contact Sensor | Zigbee | Aqara Hub M2 | No | Security trigger |

### Office
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Floor Lamp | Philips Hue | - | Zigbee | Hue Bridge | Yes | - |
| Hex Panels | Govee | - | WiFi | Home Assistant | Yes | - |
| Camera | Tapo | C110 | WiFi | Scrypted | Yes | RTSP stream via Scrypted, HKSV |

### Kitchen
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Camera | Tapo | C110 | WiFi | Scrypted | Yes | RTSP stream via Scrypted, HKSV |

### Hallway
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Camera | Tapo | C110 | WiFi | Scrypted | Yes | RTSP stream via Scrypted, HKSV |
| Wireless Switch | Aqara | Wireless Switch | Zigbee | Aqara Hub M2 | Yes | Scene control |

### Entrance
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Camera | Tapo | C110 | WiFi | Scrypted | Yes | RTSP stream via Scrypted, HKSV |
| Door Contact Sensor | Aqara | Contact Sensor | Zigbee | Aqara Hub M2 | Yes | Perimeter alert |

### Bathroom
| Device | Brand | Model | Protocol | Current Owner | HomeKit | Notes |
|--------|-------|-------|----------|---------------|---------|-------|
| Lights | Philips Hue | - | Zigbee | Hue Bridge | Yes | - |
| Motion Sensor | Aqara | Motion Sensor | Zigbee | Aqara Hub M2 | No | HA automation driver |

---

## Integration Summary from HA Screenshots

### Active Integrations (from screenshots)
- **Apple HomeKit Bridge** (multiple instances) - Primary HomeKit exposure method
- **Aqara** - 2x Hub M2 instances for Zigbee devices
- **Philips Hue Bridge** - Lighting control
- **Scrypted** - Camera management + HKSV
- **Govee** - Light strips and panels
- **TP-Link Kasa Smart** - WiFi plugs
- **Tapo** - Cameras (via Scrypted)
- **Google Assistant/GCP** - Voice control (secondary to HomeKit)
- **GitHub** - Automation version control
- **OpenAI Conversation** - AI assistant integration
- **Telegram** - Notifications
- **Terminal & SSH** - Remote management
- **HACS** - Community integrations

### Ecosystem Distribution
- **Zigbee devices**: 8 devices (via Hue Bridge + Aqara Hub M2)
- **WiFi devices**: 7 devices (Kasa, Govee, Tapo cameras)
- **HomeKit-exposed**: 13 of 19 devices (68%)
- **Cameras**: 4x Tapo C110 (all via Scrypted → HKSV)

---

## Device Count Summary
- **Total Devices**: 19 (excluding infrastructure)
- **Sensors**: 4 (contact x2, motion x1, safe x1)
- **Lights**: 6 (Hue x4, Govee x2)
- **Plugs/Switches**: 3 (Kasa x2, Aqara wireless switch x1)
- **Cameras**: 4 (Tapo C110)
- **Other**: 2 (TV light strip, hex panels)

---

## Missing Information (To Be Collected)

1. **Daily Routines** - Need user input on:
   - Morning routine (wake up, lights, climate)
   - Evening routine (arrive home, lights, security)
   - Night routine (sleep, security arm)
   - Work-from-home routines

2. **Additional Devices** - Possible gaps:
   - Climate control (thermostats, HVAC)
   - Media devices (Apple TV, speakers, TV integrations)
   - Locks/security system
   - Blinds/shades
   - Additional sensors not yet catalogued

3. **Network Details**:
   - Current VLAN setup (if any)
   - WiFi network segregation
   - IP address ranges for IoT devices

4. **HomeKit Homes/Rooms Structure** - Current organization in HomeKit app

---

## Next Steps
1. User to provide daily routine details
2. User to confirm if any devices are missing from this inventory
3. Review device ownership map (next document)
4. Define HomeKit exposure strategy
5. Create phased migration roadmap
