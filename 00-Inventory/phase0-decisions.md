# Phase 0 Architectural Decisions

**Date:** 2026-01-18
**Status:** Decisions finalized, ready to proceed to Phase 1

---

## Decision Summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Aqara Sensor Exposure | **Option B: Via HA HomeKit Bridge** | Single source of truth, unified control |
| Adaptive Lighting | **HA Component (HA-first)** | Consistent approach across all lights |
| Google Assistant | **Keep (with Nest Mini)** | Bathroom speaker in use |
| Phase 4 Naming | **Proceed** | Building from scratch, organization priority |
| Network Rebuild | **Proceed (from scratch)** | Enhanced security, privacy, isolation |

---

## 1. Aqara Sensor HomeKit Exposure

**Decision**: **Option B - Expose via HA HomeKit Bridge**

**Devices Affected**:
- Bedroom Window Left Contact
- Bedroom Window Right Contact
- Entrance Door Contact
- Kitchen Fridge Contact
- Hallway Wireless Switch

**Implementation**:
- All Aqara sensors connected to Aqara Hub M2
- HA reads sensors via Aqara integration
- HA HomeKit Bridge exposes selected sensors to HomeKit
- Aqara Hub M2's native HomeKit NOT used

**Rationale**:
- Single source of truth (HA is the brain)
- Unified control and automation logic
- Consistent with "HA-first" architecture
- Easier to manage entity names and configurations

**Reference**: [ownership-map.md:94-103](00-Inventory/ownership-map.md)

---

## 2. Adaptive Lighting Strategy

**Decision**: **HA Adaptive Lighting Component (HA-first approach)**

**Implementation**:
- Install HA Adaptive Lighting integration (HACS)
- Apply to ALL lights (Hue + Govee)
- HomeKit native Adaptive Lighting NOT used (even for Hue)

**Rationale**:
- Consistent approach: HA manages all lighting logic
- More customization options (per-room settings, manual overrides)
- Aligns with "HA is the automation brain" principle
- Avoids conflicts between HomeKit and HA lighting control

**Note**: Since Hue lights are exposed via native Hue Bridge HomeKit, HA will control them via Hue Bridge integration for automation purposes.

**Reference**: [homekit-exposure-list.md:227-252](00-Inventory/homekit-exposure-list.md)

---

## 3. Google Assistant - Keep

**Decision**: **Keep Google Assistant integration**

**Devices**:
- Nest Mini (1st Gen) - Bathroom speaker (to be added to inventory)
- Echo Show 8 (Bedroom)
- Echo Dot (Office)

**Rationale**:
- Bathroom speaker actively in use
- Multi-ecosystem approach (HomeKit primary, Google/Alexa secondary)
- Different voice assistants have different strengths

**Implementation**:
- Keep HA Google Assistant integration active
- Expose relevant devices to Google Home
- Primary control remains HomeKit/Siri
- Google Assistant as secondary interface

**Reference**: [homekit-exposure-list.md:257-260](00-Inventory/homekit-exposure-list.md)

---

## 4. Phase 4 Entity Naming Standardization

**Decision**: **Proceed with full standardization**

**Rationale**:
- Starting fresh with clean slate
- Breaking existing automations is acceptable
- Organization and consistency are priorities
- Long-term maintainability over short-term convenience

**Scope**:
- Standardize ALL entity IDs: `<domain>.<room>_<function>[_<descriptor>]`
- Standardize ALL friendly names: `<Room> <Category> <Descriptor>`
- Update ALL automations to use new entity IDs
- Document ALL changes with before/after mapping

**Risk Acceptance**:
- HIGH RISK acknowledged
- Automations WILL break
- Plan includes comprehensive testing and validation
- Rollback plan documented in roadmap

**Reference**: [roadmap.md:111-139](roadmap.md)

---

## 5. Network Rebuild Strategy

**Decision**: **Complete network rebuild from scratch**

**Goals**:
- Enhanced security and isolation
- Privacy-focused segmentation
- Professional network architecture
- Future-proof configuration

**Current State**:
- Router: Verizon CR1000A
- 3 SSIDs already exist:
  - Primary: "EnterAtYourOwnRisk" (2.4/5/6 GHz)
  - Guest: "HotNeighborsCanConnect" (2.4 GHz)
  - IoT: "ByteMe" (2.4 GHz)

**Plan**:
- Leverage existing SSID structure
- Enhance with proper firewall rules
- Implement strict device isolation
- Preserve HomeKit/mDNS functionality

**Note**: Phase 1 will include research on CR1000A VLAN capabilities via Verizon app or router admin interface.

**Reference**: [current-network.md](01-Network/current-network.md)

---

## Additional Decisions

### Control Room Location
**Status**: TBD (still needed for network planning)

### Homebridge Usage
**Decision**: Reserve for future niche devices, currently unused

---

## Next Actions

1. ✅ Update network baseline with CR1000A details
2. ✅ Add new devices to inventory (Nest Mini, Microwave, Printer)
3. ✅ Update Phase 4 roadmap status to "Proceed"
4. → Proceed to Phase 1: Network Segmentation planning

---

## Version History

- **v1.0.0** (2026-01-18) - Initial decisions documented
