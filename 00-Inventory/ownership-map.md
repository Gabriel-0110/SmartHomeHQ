# Device Ownership Map

**Date:** 2026-01-18
**Purpose:** Define which platform owns/controls each device and how HomeKit access is provided

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         HomeKit                              │
│                    (User Interface)                          │
└───────────────────────┬─────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┬─────────────────┐
        │               │               │                 │
        ▼               ▼               ▼                 ▼
┌──────────────┐ ┌──────────┐  ┌──────────────┐ ┌──────────────┐
│ Home         │ │ Scrypted │  │ Homebridge   │ │ Native       │
│ Assistant    │ │          │  │              │ │ HomeKit      │
│ (via Bridge) │ │ (HKSV)   │  │ (Niche only) │ │ (Direct)     │
└──────┬───────┘ └────┬─────┘  └──────┬───────┘ └──────┬───────┘
       │              │               │                │
       │              │               │                │
  ┌────┴────┐    ┌────┴────┐    ┌────┴────┐     ┌─────┴─────┐
  │ Devices │    │ Cameras │    │ Niche   │     │ Hue       │
  │ (WiFi/  │    │ (Tapo)  │    │ Devices │     │ Bridge    │
  │ Zigbee) │    └─────────┘    └─────────┘     │ (direct)  │
  └─────────┘                                    └───────────┘
```

---

## Ownership Strategy by Device Type

### 1. Cameras → Scrypted (Owner) → HomeKit (HKSV)
**Rationale**: Scrypted provides RTSP stream management and HomeKit Secure Video recording.

| Device | Current Path | Exposure Method |
|--------|-------------|-----------------|
| Kitchen Camera (Tapo C110) | Tapo → Scrypted → HKSV | Scrypted Camera Bridge |
| Hallway Camera (Tapo C110) | Tapo → Scrypted → HKSV | Scrypted Camera Bridge |
| Entrance Camera (Tapo C110) | Tapo → Scrypted → HKSV | Scrypted Camera Bridge |
| Office Camera (Tapo C110) | Tapo → Scrypted → HKSV | Scrypted Camera Bridge |

**Rollback**: Remove camera from Scrypted, re-add to HA Tapo integration if needed.

---

### 2. Philips Hue Lights → Hue Bridge → Direct HomeKit
**Rationale**: Hue Bridge has native HomeKit support. Direct connection avoids double-hop through HA.

| Device | Current Path | Exposure Method | Notes |
|--------|-------------|-----------------|-------|
| Bedroom Vanity Light | Hue Bridge → HomeKit | Native | Adaptive Lighting in HomeKit |
| Office Floor Lamp | Hue Bridge → HomeKit | Native | - |
| Bathroom Lights | Hue Bridge → HomeKit | Native | - |

**Current Issue**: Listed as exposed via HA in inventory. Should verify if double-exposed.

**Migration**: Remove from HA HomeKit bridge if present, use Hue Bridge's native HomeKit only.

---

### 3. WiFi Plugs & Lights → Home Assistant → HomeKit Bridge
**Rationale**: HA provides automation logic (schedules, presence detection, TV sync). HomeKit for UI control.

| Device | Current Path | Exposure Method | Automation in HA? |
|--------|-------------|-----------------|-------------------|
| Bedroom TV Plug (Kasa) | Kasa → HA → HK Bridge | HA Apple HomeKit | Yes |
| Bedroom Air Purifier (Kasa) | Kasa → HA → HK Bridge | HA Apple HomeKit | Yes (schedule/presence) |
| Bedroom TV Light Strip (Govee) | Govee → HA → HK Bridge | HA Apple HomeKit | Yes (TV mode sync) |
| Office Hex Panels (Govee) | Govee → HA → HK Bridge | HA Apple HomeKit | Unknown |

**Key Point**: These devices benefit from HA automations, so HA should remain the "owner."

---

### 4. Aqara Zigbee Sensors → HA vs Direct HomeKit
**Current State**: Mixed exposure strategy.

| Device | Current Owner | Current HK Exposure | Recommendation |
|--------|---------------|---------------------|----------------|
| Bedroom Safe Sensor | Aqara Hub M2 | No | HA automation only (security trigger) |
| Entrance Door Sensor | Aqara Hub M2 | Yes | Direct via Aqara Hub HomeKit or HA bridge |
| Bathroom Motion Sensor | Aqara Hub M2 | No | HA automation only (lighting driver) |
| Hallway Wireless Switch | Aqara Hub M2 | Yes | Direct via Aqara Hub HomeKit (scene control) |

**Decision Needed**:
- **Option A**: Aqara Hub M2 has native HomeKit. Expose sensors directly from Aqara, use HA for automation only.
- **Option B**: Keep sensors in HA, expose via HA HomeKit bridge for unified control.

**Pros of Option A**:
- Faster response (direct HomeKit)
- Less complexity (no HA bridge for sensors)

**Pros of Option B**:
- Single bridge configuration
- All automations in one place
- Easier to add custom sensor logic

**Recommended**: Option B for consistency. Use HA as single source of truth for HomeKit exposure.

---

### 5. Homebridge → Niche Devices Only
**Current State**: Homebridge is listed but no devices assigned.

**Strategy**: Reserve for devices that:
- Cannot be integrated into HA
- Cannot connect directly to HomeKit
- Require community plugins not available in HA

**Examples of candidates**:
- Legacy IR devices (via Broadlink plugin)
- Non-standard WiFi devices without HA integration
- API-only services needing HomeKit exposure

**Action**: Document which devices (if any) currently use Homebridge. If none, consider deprecating.

---

## Summary: Device Ownership Matrix

| Platform | Device Count | Device Types | HomeKit Exposure |
|----------|-------------|--------------|------------------|
| **Home Assistant** | 10 | Kasa plugs, Govee lights, Aqara sensors | HA HomeKit Bridge |
| **Scrypted** | 4 | Tapo cameras | Native HKSV |
| **Hue Bridge** | 4 | Hue lights | Native HomeKit |
| **Aqara Hub M2** | 4 | Zigbee sensors/switches | Via HA (recommended) |
| **Homebridge** | 0 | Niche only | (future use) |

---

## HomeKit Bridge Architecture

### Current Setup (Inferred)
- **Multiple HomeKit Bridge instances** in HA (seen in screenshots)
- Possible issue: Multiple bridges can cause confusion and performance issues

### Recommended Architecture
- **Single HA HomeKit Bridge** for all HA-managed devices
- **Separate native bridges**:
  - Hue Bridge (native HomeKit)
  - Scrypted (HKSV cameras)
  - Aqara Hub M2 (decide: direct or via HA)
- **Homebridge instance**: Only if needed for niche devices

### Migration Action Items
1. Audit current HA HomeKit bridge instances
2. Consolidate into single bridge with proper entity filtering
3. Remove Hue devices from HA HomeKit bridge (if present)
4. Verify Aqara device exposure strategy
5. Document Homebridge usage (or deprecate if unused)

---

## Rollback Scenarios

### Rollback: Camera from Scrypted to HA
1. Remove camera from Scrypted
2. Re-add Tapo camera to HA via Tapo integration
3. Expose to HomeKit via HA bridge (no HKSV, just live view)
4. **Consequence**: Lose HKSV recording

### Rollback: Hue from Direct to HA
1. Add Hue Bridge integration to HA
2. Expose Hue devices via HA HomeKit bridge
3. Remove Hue Bridge from HomeKit (Home app)
4. **Consequence**: Extra hop, potential latency

### Rollback: WiFi Device from HA to Direct HomeKit
- **Not recommended**: Kasa/Govee devices don't have native HomeKit
- **Alternative**: Use manufacturer's app if HA fails, but lose automation

### Rollback: Aqara from HA to Direct
1. Remove Aqara devices from HA HomeKit bridge
2. Pair Aqara Hub M2 directly to HomeKit
3. Expose sensors via Aqara's native HomeKit
4. **Consequence**: HA automations must use Aqara hub integration, not HomeKit Controller

---

## Next Steps
1. **User Decision Required**: Aqara sensor exposure strategy (HA bridge vs direct)
2. **Audit**: Current HA HomeKit bridge configuration (how many instances? which entities?)
3. **Verify**: Are Hue devices currently double-exposed (HA + Hue Bridge)?
4. **Document**: Homebridge current usage (if any)
