# Phase 0 Summary — Complete Device Inventory

**Date:** 2026-01-18 (Updated)
**Status:** Complete inventory with all devices discovered

---

## Summary Statistics

| Category | Count |
|----------|-------|
| **Total Devices** | 35 (excluding infrastructure) |
| **Infrastructure** | 3 (HA, Scrypted, Homebridge) |
| **Hubs/Controllers** | 4 (Apple TV, Aqara M2, Hue Bridge, NFC Bundle) |
| **HomeKit Exposed** | 20 devices (57%) |
| **Cameras** | 4 (Tapo C110) |
| **Lights** | 9 (Hue x7, Govee x2) |
| **Sensors** | 12 (contact x6, motion x2, NFC x4) |
| **Plugs/Switches** | 4 (Kasa x2, Govee strip x1, Aqara wireless x1) |
| **Voice Assistants** | 2 (Echo Show 8, Echo Dot) |
| **Media Devices** | 3 (Apple TV x2, iPad dashboard) |

---

## Devices by Area

### Control Room (Hub Central) — 4 devices
| Device | Type | Role | HomeKit |
|--------|------|------|---------|
| Apple TV (Hub) | Controller | HomeKit + Thread + Matter hub | No (infrastructure) |
| Aqara Hub M2 | Zigbee Hub | Aqara device controller | No (infrastructure) |
| Philips Hue Bridge | Zigbee Hub | Hue light controller | No (infrastructure) |
| NFC Tags Bundle | NFC Tags | 30-35 automation triggers | No (automation only) |

### Bedroom — 11 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Vanity Light | Philips Hue | Zigbee | Hue Bridge | Yes |
| TV Plug | Kasa | WiFi | Home Assistant | Yes |
| Air Purifier Plug | Kasa | WiFi | Home Assistant | Yes |
| TV Light Strip | Govee | WiFi | Home Assistant | Yes |
| Safe Contact Sensor | Aqara | Zigbee | Aqara M2 | No |
| Window Left Contact | Aqara | Zigbee | Aqara M2 | Yes |
| Window Right Contact | Aqara | Zigbee | Aqara M2 | Yes |
| Echo Show 8 | Amazon | WiFi | Direct | No |
| Apple TV | Apple | WiFi | Direct | No |
| Motion & Illumination (P2) | Aqara | Zigbee | Aqara M2 | No |
| NFC Tag 1 | NFC | NFC | HA | No |

### Office — 5 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Floor Lamp | Philips Hue | Zigbee | Hue Bridge | Yes |
| Hex Panels | Govee | WiFi | Home Assistant | Yes |
| Camera | Tapo C110 | WiFi | Scrypted | Yes |
| Echo Dot | Amazon | WiFi | Direct | No |
| NFC Tag 2 | NFC | NFC | HA | No |

### Kitchen — 2 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Camera | Tapo C110 | WiFi | Scrypted | Yes |
| Fridge Contact Sensor | Aqara | Zigbee | Aqara M2 | Yes |

### Hallway — 3 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Camera | Tapo C110 | WiFi | Scrypted | Yes |
| Wireless Switch | Aqara | Zigbee | Aqara M2 | Yes |
| iPad 5th Gen Panel | Apple | WiFi | Direct | No |

### Entrance — 2 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Camera | Tapo C110 | WiFi | Scrypted | Yes |
| Door Contact Sensor | Aqara | Zigbee | Aqara M2 | Yes |

### Bathroom — 4 devices
| Device | Type | Protocol | Owner | HomeKit |
|--------|------|----------|-------|---------|
| Baño Mirror 1 | Philips Hue | Zigbee | Hue Bridge | Yes |
| Baño Mirror 2 | Philips Hue | Zigbee | Hue Bridge | Yes |
| Baño Mirror 3 | Philips Hue | Zigbee | Hue Bridge | Yes |
| Motion & Illumination (P1) | Aqara | Zigbee | Aqara M2 | No |

---

## Platform Distribution

| Platform | Device Count | Device Types |
|----------|-------------|--------------|
| **Hue Bridge** | 7 | Lights (Bedroom x1, Office x1, Bathroom x3, plus 2 TBD) |
| **Home Assistant** | 10 | Kasa plugs x2, Govee lights x2, Aqara sensors (via HA) |
| **Scrypted** | 4 | Tapo C110 cameras |
| **Aqara Hub M2** | 11 | Contact sensors x6, Motion sensors x2, Wireless switch x1, NFC triggers |
| **Direct (no hub)** | 5 | Echo devices x2, Apple TV x2, iPad x1 |

---

## Protocol Breakdown

| Protocol | Count | Devices |
|----------|-------|---------|
| **Zigbee** | 18 | Hue lights (7) + Aqara sensors/switches (11) |
| **WiFi** | 14 | Cameras (4), Govee (2), Kasa (2), Echo (2), Apple TV (2), iPad (1) |
| **NFC** | 3+ | NFC Tags (3 assigned + 30-35 in bundle) |
| **LAN** | 3 | HA, Scrypted, Homebridge (infrastructure) |

---

## HomeKit Exposure Strategy

### ✅ Expose to HomeKit (20 devices)

#### Lights (7 devices)
- Bedroom Vanity Light
- Office Floor Lamp
- Bathroom Mirror 1, 2, 3
- Bedroom TV Light Strip
- Office Hex Panels

#### Cameras (4 devices)
- Kitchen Camera (HKSV)
- Office Camera (HKSV)
- Hallway Camera (HKSV)
- Entrance Camera (HKSV)

#### Sensors (4 devices)
- Bedroom Window Left Contact
- Bedroom Window Right Contact
- Entrance Door Contact
- Kitchen Fridge Contact

#### Plugs/Switches (3 devices)
- Bedroom TV Plug
- Bedroom Air Purifier Plug
- Hallway Wireless Switch

### ❌ Do NOT Expose to HomeKit (15 devices)

#### Automation-Only Sensors
- Bedroom Safe Contact (security trigger only)
- Bedroom Motion & Illumination (P2) (presence automation)
- Bathroom Motion & Illumination (P1) (lighting automation)
- NFC Tags x3 (automation triggers via HA)

#### Voice Assistants
- Echo Show 8 (Bedroom)
- Echo Dot (Office)

#### Media/Infrastructure
- Apple TV x2 (HomeKit hubs, not exposed as accessories)
- iPad 5th Gen Panel (dashboard only)
- Control Room hubs (infrastructure)

---

## Key Architectural Decisions

### 1. Dual Apple TV Setup
- **Bedroom Apple TV**: Media player + secondary HomeKit hub
- **Control Room Apple TV**: Primary HomeKit/Thread/Matter hub (no TV usage)

**Rationale**: Dedicated hub for HomeKit reliability, separate from media usage.

### 2. Window Sensors Exposed
**Decision**: Expose bedroom window contacts to HomeKit
**Rationale**:
- Perimeter security monitoring via Home app
- Automation triggers for ventilation/climate control
- User awareness of open windows

### 3. Fridge Sensor Exposed
**Decision**: Expose fridge contact to HomeKit
**Rationale**:
- User notification if door left open
- Visible in Home app for quick status check

### 4. Echo Devices NOT Exposed
**Decision**: Keep Alexa ecosystem separate from HomeKit
**Rationale**:
- Echo Show 8 = alarm clock + secondary voice assistant
- Echo Dot = office voice control
- No HomeKit integration needed (HA can control if necessary)

### 5. iPad Dashboard
**Decision**: Wall-mounted iPad for HA dashboard
**Rationale**:
- Central control panel for whole home
- Kiosk mode with HA Companion app
- No need to expose as HomeKit accessory

### 6. NFC Tag Strategy
**Decision**: All NFC tags managed via HA, not exposed to HomeKit
**Rationale**:
- HA provides more flexible NFC automation options
- iOS native NFC shortcuts can trigger HA automations if needed
- 30-35 tags for room-specific and task-specific triggers

---

## Network Implications (For Phase 1)

### WiFi Device Count: 14
- 4x Cameras (Tapo C110) → **Camera VLAN**
- 2x Echo devices → **IoT VLAN**
- 2x Govee lights → **IoT VLAN**
- 2x Kasa plugs → **IoT VLAN**
- 2x Apple TV → **Trusted VLAN** (HomeKit hubs)
- 1x iPad → **Trusted VLAN** (dashboard)
- 1x Govee TV light strip → **IoT VLAN**

### Zigbee Device Count: 18
- Zigbee devices don't need VLAN segregation (isolated via hubs)
- Hubs (Hue Bridge, Aqara M2) → **Trusted or IoT VLAN** (to be decided in Phase 1)

---

## Missing Information / Next Steps

### 1. IP Addresses
- Only Kitchen Camera has documented IP (192.168.1.205)
- Need IPs for:
  - HA VM
  - Scrypted host
  - Homebridge host (if used)
  - Other 3 cameras
  - WiFi devices (for static assignment in Phase 1)

### 2. Control Room Physical Setup
- Is "Control Room" a dedicated equipment room/closet?
- Or is it collocated with another area (Office, Bedroom)?
- Helps with network planning (cable runs, WiFi coverage)

### 3. Router/Network Details
- Still need from Phase 1: Verizon router model, VLAN support, current SSID setup

---

## Files Generated

- [device-inventory.csv](device-inventory.csv) - Complete device list (35 devices)
- [ownership-map.md](ownership-map.md) - Platform ownership (HA, Scrypted, Hue, Aqara)
- [homekit-exposure-list.md](homekit-exposure-list.md) - 20 devices exposed
- [phase0-finalized-inventory.md](phase0-finalized-inventory.md) - Original analysis
- [phased-migration-roadmap.md](phased-migration-roadmap.md) - Migration plan
- **This file**: Updated summary with all discovered devices

---

## Phase 0 Status: ✅ COMPLETE (with noted gaps to fill)

**Next Action**: Fill in missing information (see above), then proceed to Phase 1 (Network Segmentation).
