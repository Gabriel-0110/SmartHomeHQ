# Device Ownership Map

**Date:** 2026-01-18
**Purpose:** Define which platform owns/controls each device and how HomeKit access is provided

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         HomeKit                              │
│                    (User Interface)                          │
└──────┬────────────┬─────────────┬──────────────┬────────────┘
       │            │             │              │
       ▼            ▼             ▼              ▼
┌────────────┐ ┌─────────┐ ┌──────────┐ ┌──────────────┐
│ Home       │ │Scrypted │ │ Apple TV │ │ Hue Bridge   │
│ Assistant  │ │ (HKSV)  │ │ (Matter) │ │ (Native HK)  │
│ Bridge     │ └────┬────┘ └────┬─────┘ └──────┬───────┘
└─────┬──────┘      │           │               │
      │             │           │               │
  ┌───┴───┐    ┌────┴────┐ ┌────┴─────┐   ┌────┴─────┐
  │ WiFi/ │    │ Cameras │ │ Matter   │   │ Hue      │
  │Zigbee │    │ (Tapo)  │ │ Devices  │   │ Lights   │
  │Devices│    └─────────┘ └──────────┘   └──────────┘
  └───────┘
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
| Bedroom Vanity Light | Hue Bridge → HomeKit | Native | HA Adaptive Lighting via Hue integration |
| Office Floor Lamp | Hue Bridge → HomeKit | Native | HA Adaptive Lighting via Hue integration |
| Bathroom Mirror 1 | Hue Bridge → HomeKit | Native | HA Adaptive Lighting via Hue integration |
| Bathroom Mirror 2 | Hue Bridge → HomeKit | Native | HA Adaptive Lighting via Hue integration |
| Bathroom Mirror 3 | Hue Bridge → HomeKit | Native | HA Adaptive Lighting via Hue integration |

**Phase 0 Decision**: Use Hue Bridge native HomeKit for exposure, HA Adaptive Lighting Component for automation.

---

### 3. Matter Devices → Apple TV (Matter Hub) → HomeKit
**Rationale**: Future-proof protocol, direct local control, cross-platform compatibility.

| Device | Current Path | Exposure Method | Notes |
|--------|-------------|-----------------|-------|
| Bedroom TV Light Strip (Govee) | Govee → Apple TV Matter → HomeKit | Native Matter | TV mode sync via HA automation |
| Bedroom Motion Sensor P2 (Aqara) | Aqara → Apple TV Matter → HomeKit | Native Matter | Matter-only device, cannot pair to Zigbee |

**Key Point**: Matter devices connect directly to Apple TV, HA can still automate via Matter integration.

---

### 4. WiFi Plugs & Lights → Home Assistant → HomeKit Bridge
**Rationale**: HA provides automation logic (schedules, presence detection). HomeKit for UI control.

| Device | Current Path | Exposure Method | Automation in HA? |
|--------|-------------|-----------------|-------------------|
| Bedroom TV Plug (Kasa) | Kasa → HA → HK Bridge | HA Apple HomeKit | Yes |
| Bedroom Air Purifier (Kasa) | Kasa → HA → HK Bridge | HA Apple HomeKit | Yes (schedule/presence) |
| Office Hex Panels (Govee) | Govee → HA → HK Bridge | HA Apple HomeKit | Yes |
| Kitchen Microwave (Toshiba) | Toshiba → HA → HK Bridge | HA Apple HomeKit | Yes |

**Key Point**: These devices benefit from HA automations, so HA should remain the "owner."

---

### 5. Aqara Zigbee Sensors → HA HomeKit Bridge
**Decision Made**: **Option B** - Via HA bridge for unified control.

| Device | Current Owner | Current HK Exposure | Implementation |
|--------|---------------|---------------------|----------------|
| Bedroom Safe Sensor | Aqara Hub M2 | No | HA automation only (security trigger) |
| Bedroom Window Left Contact | Aqara Hub M2 | **Yes** | Via HA bridge (perimeter + automation) |
| Bedroom Window Right Contact | Aqara Hub M2 | **Yes** | Via HA bridge (perimeter + automation) |
| Entrance Door Sensor | Aqara Hub M2 | **Yes** | Via HA bridge (perimeter + automation) |
| Kitchen Fridge Contact | Aqara Hub M2 | **Yes** | Via HA bridge (monitoring) |
| Bathroom Motion Sensor (P1) | Aqara Hub M2 | **Yes** | Via HA bridge (lighting + user awareness) |
| Hallway Wireless Switch | Aqara Hub M2 | **Yes** | Via HA bridge (scene control) |
| NFC Tags (3 assigned) | Aqara Hub M2 | No | HA automation only (triggers) |

**Note**: P2 Motion Sensor is Matter-only (see section 3), not Zigbee.

**Rationale**:
- Single bridge configuration
- All automations in one place
- Easier to add custom sensor logic
- Consistent with "HA-first" architecture

---

### 6. Homebridge → Niche Devices Only
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
| **Apple TV Matter** | 2 | P2 motion sensor, Govee TV strip | Native Matter → HomeKit |
| **Home Assistant** | 11 | Kasa plugs (2), Govee panels (1), Aqara sensors (6: 4 contact, 1 motion P1, 1 switch), Microwave (1) | HA HomeKit Bridge |
| **Scrypted** | 4 | Tapo cameras | Native HKSV |
| **Hue Bridge** | 5 | Hue lights (Bedroom, Office, Bathroom x3) | Native HomeKit |
| **Aqara Hub M2** | 10 | Contact sensors (6), Motion sensor P1 (1), Switch (1), NFC tags (3), Safe sensor (1 not exposed) | Via HA for 7 exposed devices |
| **Homebridge** | 0 | Niche only | (future use) |
| **Non-Exposed** | 5 | Voice assistants (3: Echo x2, Nest Mini), Printer (1), NFC bundle (30-32 unassigned) | Not exposed to HomeKit |

---

## HomeKit Bridge Architecture

### Phase 0 Strategy
- **Single HA HomeKit Bridge** for all HA-managed devices (11 devices)
- **Separate native bridges**:
  - Hue Bridge (native HomeKit) - 5 lights
  - Scrypted (HKSV cameras) - 4 cameras
  - Apple TV Matter (native Matter) - 2 devices
- **Homebridge**: Reserved for niche devices only (currently none)

### Implementation (Phase 2/3)
1. Audit current HA HomeKit bridge configuration
2. Consolidate into single bridge with entity filtering
3. Ensure Hue devices NOT exposed via HA bridge
4. Configure Aqara sensors via HA bridge
5. Verify Matter integration for HA automations

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

## Phase 0 Decisions Complete ✅

**Ownership strategy finalized**:
1. ✅ Aqara sensor exposure strategy decided: **Via HA HomeKit Bridge**
2. ✅ Hue lights ownership: **Direct via Hue Bridge native HomeKit**
3. ✅ Matter devices ownership: **Direct via Apple TV Matter controller**
4. ✅ WiFi plugs/lights ownership: **Via HA HomeKit Bridge**
5. ✅ Cameras ownership: **Via Scrypted HKSV**
6. ✅ Homebridge usage: **Reserved for niche devices only**

**Phase 2/3 will implement**: Audit and configure HA HomeKit Bridge, verify Hue double-exposure, document Homebridge usage.
