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

## Exposure Summary

**Total exposed to HomeKit**: 22 devices (58% of 38 total)

**Exposure methods**:
- **5 devices** via Hue Bridge (native HomeKit)
- **2 devices** via Apple TV Matter
- **11 devices** via HA HomeKit Bridge
- **4 devices** via Scrypted (HKSV)

---

## Next Steps

### Phase 0 - Complete ✅
1. ✅ **User Input**: Adaptive Lighting strategy confirmed (HA Component for ALL lights)
2. ✅ **User Input**: Google Assistant confirmed (Keep with Nest Mini)

### Phase 1 - Network Segmentation (Next)
- See [phase1-preflight-checklist.md](../01-Network/phase1-preflight-checklist.md) for user actions

### Phase 2/3 - HomeKit Bridge Implementation (After Phase 1)
- **Technical Task**: Audit current HA HomeKit Bridge configuration
- **Technical Task**: Implement consolidated HomeKit Bridge with HA Adaptive Lighting

### Phase 4 - Entity Naming Standardization (After Phase 2/3)
- **Technical Task**: Standardize all entity IDs and friendly names
