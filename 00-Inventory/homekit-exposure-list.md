# HomeKit Exposure Strategy

**Date:** 2026-01-18
**Purpose:** Define which devices are exposed to HomeKit and the method of exposure

---

## Exposure Principles

### 1. User-Facing Devices → Expose to HomeKit
Devices that users directly interact with via Siri, Home app, or physical automation.

### 2. Automation-Only Devices → Keep in HA Only
Sensors and triggers that only drive background automations (not user-controlled).

### 3. Native HomeKit First
If a device/hub has native HomeKit support, prefer direct connection over HA bridge.

### 4. Single Bridge Architecture
Avoid exposing the same device through multiple HomeKit bridges (causes conflicts).

---

## HomeKit Exposure List

### ✅ Expose to HomeKit (22 devices)

#### **Lights (7 devices)**
| Device | Exposure Method | Rationale |
|--------|----------------|-----------|
| Bedroom Vanity Light | Hue Bridge (native) | User control + Adaptive Lighting |
| Office Floor Lamp | Hue Bridge (native) | User control |
| Bathroom Mirror 1 | Hue Bridge (native) | User control |
| Bathroom Mirror 2 | Hue Bridge (native) | User control |
| Bathroom Mirror 3 | Hue Bridge (native) | User control |
| Bedroom TV Light Strip | Apple TV Matter | TV mode sync automation + user control (Matter device) |
| Office Hex Panels | HA HomeKit Bridge | User control |

#### **Plugs (2 devices)**
| Device | Exposure Method | Rationale |
|--------|----------------|-----------|
| Bedroom TV Plug | HA HomeKit Bridge | User control + HA schedules |
| Bedroom Air Purifier Plug | HA HomeKit Bridge | User control + presence automation |

#### **Cameras (4 devices)**
| Device | Exposure Method | Rationale |
|--------|----------------|-----------|
| Kitchen Camera | Scrypted (HKSV) | Security monitoring + HKSV recording |
| Hallway Camera | Scrypted (HKSV) | Security monitoring + HKSV recording |
| Entrance Camera | Scrypted (HKSV) | Security monitoring + HKSV recording |
| Office Camera | Scrypted (HKSV) | Security monitoring + HKSV recording |

#### **Sensors (7 devices)**
| Device | Exposure Method | Rationale |
|--------|----------------|-----------|
| Bedroom Window Left Contact | HA HomeKit Bridge | Perimeter security + ventilation automation |
| Bedroom Window Right Contact | HA HomeKit Bridge | Perimeter security + ventilation automation |
| Entrance Door Contact Sensor | HA HomeKit Bridge | Perimeter alert + Home/Away mode trigger |
| Kitchen Fridge Contact | HA HomeKit Bridge | Door left open monitoring |
| Hallway Wireless Switch | HA HomeKit Bridge | Scene control |
| Bathroom Motion Sensor (P1) | HA HomeKit Bridge | Lighting automation + user awareness |
| Bedroom Motion Sensor (P2) | Apple TV Matter | Presence detection + user awareness (Matter device) |

#### **Appliances (1 device)**
| Device | Exposure Method | Rationale |
|--------|----------------|-----------|
| Kitchen Microwave | HA HomeKit Bridge | Smart control + status monitoring |

#### **Matter Devices (2 devices - counted in categories above)**
- Bedroom Motion Sensor P2 (Aqara Matter)
- Bedroom TV Light Strip (Govee Matter)

---

### ❌ Do NOT Expose to HomeKit (16 devices)

#### **Automation-Only Sensors (3 devices)**
| Device | Kept In | Rationale |
|--------|---------|-----------|
| Bedroom Safe Contact Sensor | HA + Aqara Hub | Security automation trigger (not user-controlled) |
| NFC Tag 1 (Bedroom) | HA | Automation trigger |
| NFC Tag 2 (Office) | HA | Automation trigger |

#### **Voice Assistants (3 devices)**
| Device | Kept In | Rationale |
|--------|---------|-----------|
| Echo Show 8 (Bedroom) | Direct | Alarm clock + voice assistant |
| Echo Dot (Office) | Direct | Voice assistant |
| Nest Mini (Bathroom) | Direct | Voice assistant + speaker |

#### **Media Devices (3 devices)**
| Device | Kept In | Rationale |
|--------|---------|-----------|
| Apple TV (Bedroom) | Direct | Media player + HomeKit hub |
| Apple TV (Control Room) | Direct | Primary HomeKit/Thread/Matter hub |
| iPad 5th Gen Panel | Direct | Wall-mounted dashboard |

#### **Infrastructure (3 devices)**
| Device | Kept In | Rationale |
|--------|---------|-----------|
| Home Assistant | - | Backend hub |
| Scrypted | - | Backend camera processor |
| Homebridge | - | Backend bridge (if used) |

#### **Appliances (1 device)**
| Device | Kept In | Rationale |
|--------|---------|-----------|
| HP Printer (Office) | HA | Printing service (no useful HomeKit entities) |

#### **NFC Bundle**
| Device | Count | Rationale |
|--------|-------|-----------|
| NFC Tags (unassigned) | 30-32 | Available for future automation triggers |

---

## Exposure Method Details

### Method 1: Native HomeKit (Hue Bridge)
**Devices**: 5x Hue lights

**Setup**:
1. Hue Bridge already paired to HomeKit
2. All Hue devices automatically appear in Home app
3. HA can still control via Hue Bridge integration (for automations)

**Pros**:
- Fastest response time
- No HA dependency for HomeKit control
- Adaptive Lighting support

**Cons**:
- Must manage in two places (Home app + HA)

**Action**: Remove Hue devices from HA HomeKit Bridge if currently exposed there.

---

### Method 2: Apple TV Matter
**Devices**: 2x Matter devices (P2 sensor, Govee TV light strip)

**Setup**:
1. Apple TV acts as Matter controller
2. Matter devices paired directly to Apple TV
3. Automatically appear in HomeKit
4. HA can still automate via Matter integration

**Pros**:
- Future-proof protocol
- Fast, local, reliable
- No bridge needed
- Cross-platform compatible

**Cons**:
- Requires Matter-compatible Apple TV
- HA automation requires Matter integration

---

### Method 3: HA HomeKit Bridge (WiFi Devices + Sensors)
**Devices**: Kasa plugs, Govee panels, Aqara sensors, Microwave

**Setup**:
1. Single HA HomeKit Bridge integration
2. Entity filtering to expose only desired devices
3. Accessory mode vs Bridge mode selection

**Configuration** (in HA `configuration.yaml`):
```yaml
homekit:
  - name: Primary Bridge
    port: 21063
    filter:
      include_entities:
        # Plugs
        - switch.bedroom_tv_plug
        - switch.bedroom_air_purifier_plug
        # Lights
        - light.office_hex_panels
        # Contact Sensors
        - binary_sensor.bedroom_window_left_contact
        - binary_sensor.bedroom_window_right_contact
        - binary_sensor.entrance_door_contact
        - binary_sensor.kitchen_fridge_contact
        # Motion Sensors
        - binary_sensor.bathroom_motion_p1
        # Switches
        - sensor.hallway_wireless_switch
        # Appliances
        - switch.kitchen_microwave
```

**Pros**:
- Single management point (HA)
- Advanced automation capabilities
- Custom entity configuration

**Cons**:
- Requires HA to be online
- Slightly higher latency vs native

---

### Method 4: Scrypted HKSV (Cameras)
**Devices**: 4x Tapo C110 cameras

**Setup**:
1. Tapo cameras added to Scrypted
2. Scrypted Camera Bridge plugin
3. HKSV enabled for 24/7 recording

**Pros**:
- Native HKSV recording
- RTSP stream optimization
- 10-day recording with iCloud+

**Cons**:
- Requires iCloud+ subscription
- Requires Scrypted always-on server

---

## HomeKit Room/Zone Mapping

### Recommended HomeKit Home Structure
```
SmartHomeHQ (Home)
├── Bedroom
│   ├── Vanity Light (Hue native)
│   ├── TV Plug (Kasa via HA)
│   ├── Air Purifier Plug (Kasa via HA)
│   ├── TV Light Strip (Govee Matter)
│   ├── Window Left Contact (Aqara via HA)
│   ├── Window Right Contact (Aqara via HA)
│   └── Motion Sensor P2 (Aqara Matter)
├── Office
│   ├── Floor Lamp (Hue native)
│   ├── Hex Panels (Govee via HA)
│   └── Camera (Scrypted HKSV)
├── Kitchen
│   ├── Camera (Scrypted HKSV)
│   ├── Fridge Contact (Aqara via HA)
│   └── Microwave (Toshiba via HA)
├── Hallway
│   ├── Camera (Scrypted HKSV)
│   └── Wireless Switch (Aqara via HA)
├── Entrance
│   ├── Camera (Scrypted HKSV)
│   └── Door Contact Sensor (Aqara via HA)
└── Bathroom
    ├── Mirror 1 (Hue native)
    ├── Mirror 2 (Hue native)
    ├── Mirror 3 (Hue native)
    └── Motion Sensor P1 (Aqara via HA)
```

**Note**: HomeKit room names should match HA areas for consistency.

**Summary**:
- **22 total devices** exposed to HomeKit
- **5** via Hue Bridge (native)
- **2** via Apple TV Matter
- **11** via HA HomeKit Bridge
- **4** via Scrypted HKSV

---

## Entity Naming Convention

### Current Naming (from inventory)
- Inconsistent: Some use area prefix, some don't
- Examples: "Bedroom Vanity Light" vs "Safe Contact Sensor"

### Recommended Naming in HA (for HomeKit)
**Format**: `[area]_[device_type]_[descriptor]`

Examples:
- `bedroom_light_vanity`
- `bedroom_switch_tv_plug`
- `entrance_sensor_door_contact`
- `hallway_button_wireless_switch`

**Rationale**:
- Siri recognition: "Turn on bedroom vanity light"
- Sorting/filtering in automations
- Consistency across platforms

**Action**: Standardize entity IDs in HA before exposing to HomeKit.

---

## Automation-Only Devices: Why NOT Expose?

### Bedroom Safe Contact Sensor
- **Purpose**: Trigger "Safe opened while Away" security alert
- **User Interaction**: None (passive monitoring)
- **HomeKit Benefit**: Minimal (would just show "Open/Closed" in Home app)
- **Recommendation**: Keep in HA automations only

### Bathroom Motion Sensor
- **Purpose**: Trigger bathroom light automation
- **User Interaction**: None (passive detection)
- **HomeKit Benefit**: None (HomeKit motion sensors are for automation triggers, not control)
- **Recommendation**: Keep in HA automations only

**Key Principle**: If a device is never manually controlled or queried by the user, don't expose it to HomeKit.

---

## Adaptive Lighting Strategy

### Current Setup
- "Use Adaptive Lighting in HomeKit" noted for Bedroom Vanity Light

### Adaptive Lighting Options

#### Option 1: HomeKit Native Adaptive Lighting
- **Requirement**: Light must be exposed directly to HomeKit (not via HA bridge)
- **Supported**: Hue lights (via Hue Bridge native)
- **Configuration**: Enable in Home app for each light
- **Pros**: Native iOS integration, works without HA
- **Cons**: Cannot use HA's Adaptive Lighting component simultaneously

#### Option 2: HA Adaptive Lighting Component
- **Requirement**: HA Adaptive Lighting integration (HACS)
- **Supported**: All lights controlled by HA
- **Configuration**: Per-light or per-area automations
- **Pros**: More customization, automation-based triggers
- **Cons**: Requires HA online, conflicts with HomeKit Adaptive Lighting

**Recommendation**:
- **Hue lights**: Use HomeKit native Adaptive Lighting (already exposed directly)
- **WiFi lights** (Govee): Use HA Adaptive Lighting component (exposed via HA bridge)

**Action**: Verify Hue lights are NOT exposed via HA HomeKit Bridge to avoid conflicts.

---

## Special Considerations

### Google Assistant Integration
- Seen in HA screenshots
- **Decision Needed**: Keep Google Assistant, or migrate fully to HomeKit/Siri?
- **Recommendation**: If migrating to "HomeKit-first" strategy, deprecate Google Assistant to reduce complexity.

### Telegram/Notification Services
- Used for security alerts (e.g., safe opened, door contact)
- **Keep**: These are backend automation services, not affected by HomeKit strategy

---

## Next Steps

### Phase 0 - Complete ✅
1. ✅ **User Input**: Adaptive Lighting strategy confirmed (HA Component for ALL lights)
2. ✅ **User Input**: Google Assistant confirmed (Keep with Nest Mini)

### Phase 1 - Network Segmentation (Next)
- See [phase1-preflight-checklist.md](../01-Network/phase1-preflight-checklist.md) for user actions

### Phase 2/3 - HomeKit Bridge Implementation (After Phase 1)
- **Technical Task**: Audit current HA HomeKit Bridge configuration
- **Technical Task**: Implement consolidated HomeKit Bridge (see Implementation Checklist above)

### Phase 4 - Entity Naming Standardization (After Phase 2/3)
- **Technical Task**: Standardize all entity IDs and friendly names
