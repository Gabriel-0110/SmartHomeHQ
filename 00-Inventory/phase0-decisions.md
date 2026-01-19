# Phase 0 Architectural Decisions

**Date:** 2026-01-18
**Status:** Decisions finalized, ready to proceed to Phase 1

---

## Decision Summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Aqara Sensor Exposure | **Option B: Via HA HomeKit Bridge** | Single source of truth, unified control |
| Matter vs Zigbee Priority | **Matter for new devices, Zigbee stable** | Future-proof while maintaining stability |
| Adaptive Lighting | **HA Component (HA-first)** | Consistent approach across all lights |
| Google Assistant | **Keep (with Nest Mini)** | Bathroom speaker in use |
| Phase 4 Naming | **Proceed** | Building from scratch, organization priority |
| Network Rebuild | **Proceed (from scratch)** | Enhanced security, privacy, isolation |

---

## 1. Aqara Sensor HomeKit Exposure

**Decision**: **Option B - Expose via HA HomeKit Bridge**

**Devices Affected** (7 sensors exposed to HomeKit):
- Bedroom Window Left Contact
- Bedroom Window Right Contact
- Entrance Door Contact
- Kitchen Fridge Contact
- Hallway Wireless Switch
- Bathroom Motion Sensor (P1)
- Bedroom Safe Contact (NOT exposed - automation only)

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

## 2. Matter vs Zigbee Priority

**Decision**: **Prioritize Matter for new/Matter-native devices, keep Zigbee for existing stable devices**

**Matter Devices** (2):
- Bedroom TV Light Strip (Govee) - Matter-native, cross-platform
- Bedroom Motion Sensor P2 (Aqara) - Matter-only device, cannot connect to Hub M2

**Zigbee Devices** (maintained - 16):
- Hue lights (5) - Mature, stable, native HomeKit
- Aqara sensors (10: 6 contact, 1 motion P1, 1 switch, 3 NFC assigned) - Already paired, working well

**Rationale**:
- **Future-proofing**: Matter is the long-term standard for smart home
- **Stability**: Don't migrate working Zigbee devices unnecessarily
- **Device constraints**: P2 motion sensor can ONLY connect via Matter
- **Cross-platform**: Matter enables better interoperability

**Implementation**:
- Apple TV (Control Room) acts as primary Matter controller
- Aqara Hub M2 remains Zigbee controller for existing devices
- New device purchases should prioritize Matter when available

**Reference**: [matter-zigbee-decision.md](00-Inventory/matter-zigbee-decision.md)

---

## 3. Adaptive Lighting Strategy

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

## 4. Google Assistant - Keep

**Decision**: **Keep Google Assistant integration**

**Devices**:
- Nest Mini (1st Gen) - Bathroom speaker ✅ added to inventory
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

## 5. Phase 4 Entity Naming Standardization

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

## 6. Network Rebuild Strategy

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
4. ✅ Verify all Phase 0 documents for accuracy
5. ✅ Document Matter vs Zigbee priority decision
6. → **Phase 0 COMPLETE** - Ready to proceed to Phase 1: Network Segmentation planning

---

## Version History

- **v1.0.0** (2026-01-18) - Initial decisions documented
- **v1.1.0** (2026-01-18) - Added Matter vs Zigbee decision, updated device counts, marked Nest Mini as added
