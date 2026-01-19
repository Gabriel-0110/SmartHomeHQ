# SmartHomeHQ Roadmap

## Overview

This roadmap outlines the phased migration and improvement plan for the SmartHomeHQ smart home infrastructure. Each phase includes specific tasks, validation steps, and rollback procedures.

---

## Current Status

### ✅ Phase 0: Inventory & Target Architecture (COMPLETED)
- [x] Master device inventory created (35 devices across 6 areas)
- [x] Device ownership map defined (HA vs Scrypted vs Homebridge vs HomeKit)
- [x] HomeKit exposure strategy documented (20 devices exposed)
- [x] Screenshots captured from Home Assistant integrations
- [x] Baseline files established

**Outputs**:
- [device-inventory.csv](00-Inventory/device-inventory.csv) - Complete device list
- [ownership-map.md](00-Inventory/ownership-map.md) - Platform ownership strategy
- [homekit-exposure-list.md](00-Inventory/homekit-exposure-list.md) - HomeKit exposure decisions
- [phase0-finalized-inventory.md](00-Inventory/phase0-finalized-inventory.md) - Detailed inventory analysis

---

## 🚧 Phase 1: Network Segmentation (NEXT)

### Goal
Implement network segmentation with 3 logical zones: Trusted, IoT, and Cameras.

### Pre-Requisites
- [ ] Confirm Verizon router model and capabilities
- [ ] Verify VLAN support or SSID split options
- [ ] Document current network configuration

### Tasks
- [ ] Create [target-segmentation.md](01-Network/target-segmentation.md)
- [ ] Define firewall rule matrix (Trusted can initiate to all; IoT/Cameras cannot initiate to Trusted)
- [ ] Plan mDNS/Bonjour preservation strategy for HomeKit discovery
- [ ] Create step-by-step cutover plan with validation tests
- [ ] Document rollback procedures

### Risk Level
**Medium** - Network changes can disrupt device connectivity

### Rollback Strategy
- Revert to single network/SSID
- Restore original router configuration from backup
- Time: 15-30 minutes

---

## Phase 2: HomeKit Bridge Cleanup

### Goal
Remove Hue devices from HA HomeKit Bridge and use native Hue Bridge HomeKit only.

### Tasks
- [ ] Audit current HA HomeKit Bridge configuration
- [ ] Identify Hue devices exposed via HA
- [ ] Remove Hue lights from HA HomeKit Bridge filter
- [ ] Verify Hue lights still accessible via Hue Bridge native HomeKit
- [ ] Test Adaptive Lighting on Hue devices
- [ ] Verify HA automations still function

### Expected Outcome
- Hue lights controlled via native Hue Bridge (faster response)
- No duplicate devices in Home app
- HA can still automate Hue via Hue Bridge integration

### Risk Level
**Low** - Easy rollback available

### Rollback Strategy
- Re-add Hue entities to HA HomeKit Bridge
- Restart HomeKit integration
- Time: 15 minutes

---

## Phase 3: Consolidate HomeKit Bridges

### Goal
Merge multiple HA HomeKit Bridge instances into single, filtered bridge.

### Tasks
- [ ] Document all current HomeKit Bridge instances
- [ ] Create filtered entity list (WiFi devices + selected sensors)
- [ ] Remove old HomeKit Bridge integrations
- [ ] Create new consolidated bridge with proper filtering
- [ ] Re-pair with Home app
- [ ] Organize HomeKit rooms to match HA areas
- [ ] Test all devices and automations

### Expected Outcome
- Single HA HomeKit Bridge
- Clean, organized Home app structure
- All desired devices exposed without duplicates

### Risk Level
**Medium** - Requires re-pairing devices in Home app

### Rollback Strategy
- Remove new bridge
- Restore old configuration from backup
- Re-pair with Home app
- Time: 30 minutes

---

## Phase 4: Areas, Zones, Labels & Naming

### Goal
Standardize device naming and organization across HA and HomeKit.

### Status
**PROCEED** - User decision: Building from scratch, breaking automations is acceptable for long-term organization.

### Tasks
- [ ] Create [areas-zones-labels.md](00-Inventory/areas-zones-labels.md)
- [ ] Define naming convention (consistent with CLAUDE.md guidelines)
- [ ] Create safe rename plan that tracks automation dependencies
- [ ] Update entity IDs following: `<domain>.<room>_<function>[_<descriptor>]`
- [ ] Update friendly names for HomeKit: `<Room> <Category> <Descriptor>`
- [ ] Update all affected automations and scripts
- [ ] Test all automations after rename

### Expected Outcome
- Consistent naming across HA and HomeKit
- Better Siri recognition
- Cleaner automation code

### Risk Level
**High** - Entity ID changes break automations

**Risk Acceptance**: User acknowledges risk, prioritizes organization over short-term convenience.

### Rollback Strategy
- Restore entity IDs via HA entity registry
- Restore configuration.yaml from backup
- Time: 1 hour (manual verification needed)

---

## Phase 5: Scrypted Camera Verification

### Goal
Verify all 4 Tapo C110 cameras properly configured with HKSV.

### Tasks
- [ ] Audit Scrypted camera configuration
- [ ] Verify all 4 cameras added (Kitchen, Hallway, Entrance, Office)
- [ ] Confirm HKSV enabled on each camera
- [ ] Test live view in Home app
- [ ] Verify HKSV recording (Activity tab)
- [ ] Check motion detection triggers
- [ ] Verify RTSP stream settings (1080p, 15-20 fps)

### Expected Outcome
- All 4 cameras streaming reliably
- HKSV 10-day recording active (with iCloud+)
- Motion detection working

### Risk Level
**Low** - Cameras already functional

### Rollback Strategy
- Remove camera from Scrypted HomeKit
- Re-add and re-pair
- Time: 15 minutes per camera

---

## Phase 6: Automation Library

### Goal
Document and standardize all automations with focus on daily routines, security, and maintenance.

### Tasks
- [ ] Create [automations-pack.md](02-HomeAssistant/automations-pack.md)
- [ ] Document daily routine automations (morning, evening, night)
- [ ] Security automations (door sensors, away mode, safe monitoring)
- [ ] Maintenance automations (device availability, backup verification)
- [ ] Include triggers, conditions, and actions for each
- [ ] Create testing plan for each automation
- [ ] Document failure modes and recovery

### Focus Areas
- Morning routine (wake up, lights, climate control)
- Evening routine (arrive home, lights, security disarm)
- Night routine (sleep mode, security arm)
- Presence detection (home/away states)
- Security triggers (perimeter alerts, safe opening)

### Risk Level
**Low** - Documentation only, no system changes

---

## Phase 7: Aqara Sensor Strategy

### Goal
Finalize Aqara sensor exposure strategy (via HA vs native HomeKit).

### Decision Point
**Option A** (Recommended): Expose via HA HomeKit Bridge for unified control
**Option B**: Expose via Aqara Hub M2 native HomeKit for faster response

### Tasks
- [ ] Decide exposure strategy based on user preference
- [ ] Configure sensors accordingly
- [ ] Test HA automations with chosen approach
- [ ] Verify HomeKit accessibility for exposed sensors

### Risk Level
**Low** - Easy to switch between approaches

### Rollback Strategy
- Switch between Option A and Option B
- Time: 20 minutes

---

## Phase 8: Dashboard & UI

### Goal
Create modern, centralized Home Assistant dashboard.

### Tasks
- [ ] Design dashboard layout (rooms, status, quick actions)
- [ ] Implement dashboard configuration
- [ ] Create mobile-friendly views
- [ ] Add camera tiles with HKSV links
- [ ] Include system health monitoring
- [ ] Document dashboard organization

### Risk Level
**None** - UI changes only

---

## Phase 9: Network Hardening & Monitoring

### Goal
Implement ongoing monitoring and security best practices.

### Tasks
- [ ] Set up device availability monitoring
- [ ] Configure HA backup automation (local + offsite)
- [ ] Create network performance tracking
- [ ] Set up security alerts (unauthorized access, device offline)
- [ ] Document maintenance cadence
- [ ] Regular security audit schedule

### Risk Level
**Low** - Monitoring additions

---

## Timeline & Priorities

| Phase | Status | Priority | Estimated Time | Risk Level |
|-------|--------|----------|----------------|------------|
| Phase 0: Inventory | ✅ Complete | - | - | - |
| Phase 1: Network | 🔜 Next | High | 2-4 hours | Medium |
| Phase 2: Hue Cleanup | Planned | Medium | 1 hour | Low |
| Phase 3: Bridge Consolidation | Planned | High | 2-3 hours | Medium |
| Phase 4: Naming | Planned | Low | 3-4 hours | High |
| Phase 5: Cameras | Planned | Low | 1 hour | Low |
| Phase 6: Automations | Planned | High | 4-6 hours | Low |
| Phase 7: Aqara | Planned | Low | 1 hour | Low |
| Phase 8: Dashboard | Planned | Medium | 3-4 hours | None |
| Phase 9: Monitoring | Ongoing | Medium | 2-3 hours | Low |

### Recommended Execution Order
1. **Phase 0** ✅ (Foundation)
2. **Phase 1** (Network security)
3. **Phase 2-3** (HomeKit cleanup)
4. **Phase 5-6** (Cameras + Automations)
5. **Phase 8** (Dashboard)
6. **Phase 4** (Optional: Naming - only if needed)
7. **Phase 7** (Optional: Aqara strategy)
9. **Phase 9** (Ongoing maintenance)

---

## Success Criteria

### Phase 0 ✅
- Complete device inventory
- Clear ownership strategy
- HomeKit exposure decisions documented

### Overall Project
- HomeKit shows only stable, user-friendly devices (20 devices)
- HA owns all automation logic
- Cameras isolated in Scrypted with HKSV
- Network segmented (Trusted/IoT/Cameras)
- All devices have single source of truth (no double-bridging)
- Repeatable process for adding new devices

---

## Rollback: Full Project Revert

### Nuclear Option
If entire migration fails, restore to pre-migration state.

**Procedure**:
1. Restore HA snapshot from Phase 0 backup
2. Remove all HomeKit bridges from Home app
3. Re-pair original configurations
4. Revert network changes (single SSID)
5. Verify all devices return to original state

**Time**: 30-60 minutes
**Consequence**: All migration work lost

---

## Version History

- **v2.0.0** (January 2026) - Phase 0 complete, refined roadmap with rollback procedures
- **v1.0.0** (January 2026) - Initial repository setup

---

## Notes

- Always create backups before each phase
- Test in small increments
- Validate after each step
- Keep rollback procedures handy
- Update this roadmap as phases complete
- Document any deviations or issues in phase-specific files
