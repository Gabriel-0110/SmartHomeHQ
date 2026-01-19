# Phased Migration Roadmap

**Date:** 2026-01-18
**Purpose:** Step-by-step migration plan with rollback procedures

---

## Migration Goals

1. **Consolidate HomeKit exposure** → Single HA bridge + native bridges
2. **Optimize device ownership** → Right platform for each device type
3. **Standardize naming** → Consistent entity IDs and friendly names
4. **Improve automation reliability** → Clear separation of concerns
5. **Document everything** → Enable easy troubleshooting and rollback

---

## Phase 1: Audit & Baseline (Week 1)

### Goal
Document current state and create rollback snapshots.

### Tasks

#### 1.1 Backup Current Configuration
- [ ] **HA Snapshot**: Create full backup via HA Supervisor
- [ ] **Export HomeKit Bridge config**: Copy current `configuration.yaml` HomeKit section
- [ ] **Screenshot HomeKit Home**: Document current Home app structure (rooms, scenes, automations)
- [ ] **Export Scrypted config**: Backup Scrypted settings
- [ ] **Document current automations**: Export all HA automations referencing exposed devices

**Rollback Procedure**: Restore HA snapshot to revert all changes.

---

#### 1.2 Audit Current HomeKit Bridges
- [ ] Identify all active HomeKit Bridge instances in HA
- [ ] List entities exposed by each bridge (check integrations page)
- [ ] Identify duplicate exposures (same device via multiple bridges)
- [ ] Document HomeKit accessory/bridge mode for each instance

**Expected Findings**:
- Multiple HomeKit Bridge instances (seen in screenshot)
- Possible duplicates: Hue lights exposed via both HA and Hue Bridge

**Deliverable**: `audit-report.md` with:
- Number of bridges
- Entities per bridge
- Duplicate exposure list

---

#### 1.3 Test Current Setup
- [ ] Test each HomeKit-exposed device (on/off, brightness, etc.)
- [ ] Verify Siri commands work for key devices
- [ ] Check HomeKit automation triggers
- [ ] Document any existing issues (latency, offline devices, etc.)

**Deliverable**: `baseline-test-results.md`

---

### Phase 1 Rollback
**If issues found**: Document issues but do not modify configuration yet. Phase 1 is read-only.

---

## Phase 2: Hue Bridge Cleanup (Week 2)

### Goal
Remove Hue devices from HA HomeKit Bridge and use Hue Bridge native HomeKit only.

### Pre-Requisites
- Hue Bridge already paired with HomeKit (should be active)
- Hue devices visible in Home app via Hue Bridge

### Tasks

#### 2.1 Identify Hue Devices in HA HomeKit Bridge
- [ ] Check HA HomeKit Bridge configuration for any Hue light entities
- [ ] Confirm Hue lights are exposed via Hue Bridge in Home app (look for bridge icon)

#### 2.2 Remove Hue Devices from HA Bridge
- [ ] Edit HA `configuration.yaml` HomeKit section
- [ ] Add exclusion filter for Hue lights:
```yaml
homekit:
  - name: Primary Bridge
    filter:
      exclude_domains:
        - light  # Exclude all lights, then re-include non-Hue below
      include_entities:
        - light.bedroom_tv_light_strip  # Govee, keep in HA
        - light.office_hex_panels  # Govee, keep in HA
```
OR use `exclude_entities` to specifically exclude Hue lights.

- [ ] Restart HA HomeKit Bridge integration
- [ ] Verify Hue lights still appear in Home app (via Hue Bridge, not HA)

#### 2.3 Test Hue Lights
- [ ] Test each Hue light via Home app (on/off, brightness, color)
- [ ] Test Siri control: "Turn on bedroom vanity light"
- [ ] Verify HA automations still work (HA controls Hue via Hue Bridge integration)
- [ ] Enable Adaptive Lighting in Home app for desired Hue lights

**Expected Outcome**:
- Hue lights controlled via Hue Bridge (fast, native)
- HA can still automate Hue lights via Hue Bridge integration
- No duplicate exposure in Home app

### Phase 2 Rollback
**If issues**:
1. Re-add Hue light entities to HA HomeKit Bridge include list
2. Restart HomeKit Bridge integration
3. Wait 5 minutes for Home app to re-sync
4. **Consequence**: Hue lights will be duplicated in Home app (one from HA, one from Hue Bridge). Remove Hue Bridge from Home app if needed.

**Rollback Time**: 15 minutes

---

## Phase 3: Consolidate HA HomeKit Bridges (Week 3)

### Goal
Merge multiple HA HomeKit Bridge instances into a single, filtered bridge.

### Pre-Requisites
- Phase 2 completed (Hue lights removed from HA bridge)
- Backup of all current HomeKit Bridge configurations

### Tasks

#### 3.1 Document Desired Entity Exposure
Create final list of entities to expose via HA HomeKit Bridge:

**Switches/Plugs**:
- `switch.bedroom_tv_plug`
- `switch.bedroom_air_purifier_plug`

**Lights**:
- `light.bedroom_tv_light_strip`
- `light.office_hex_panels`

**Sensors** (optional):
- `binary_sensor.entrance_door_contact`
- `binary_sensor.hallway_wireless_switch`

**Cameras**: None (handled by Scrypted)

#### 3.2 Remove Old HomeKit Bridge Instances
- [ ] Identify all HomeKit integrations in HA (Settings → Devices & Services → HomeKit)
- [ ] Remove all except one (or remove all, will create new in next step)
- [ ] Remove HomeKit accessories from Home app (if prompted)

#### 3.3 Create New Consolidated Bridge
- [ ] Edit `configuration.yaml`:
```yaml
homekit:
  - name: HA Primary Bridge
    port: 21063
    filter:
      include_entities:
        - switch.bedroom_tv_plug
        - switch.bedroom_air_purifier_plug
        - light.bedroom_tv_light_strip
        - light.office_hex_panels
        - binary_sensor.entrance_door_contact
        - sensor.hallway_wireless_switch
    entity_config:
      switch.bedroom_tv_plug:
        name: Bedroom TV Plug
      switch.bedroom_air_purifier_plug:
        name: Bedroom Air Purifier
      light.bedroom_tv_light_strip:
        name: Bedroom TV Light Strip
      light.office_hex_panels:
        name: Office Hex Panels
```
- [ ] Restart HA
- [ ] Pair new HomeKit Bridge with Home app (scan QR code)

#### 3.4 Reorganize HomeKit Rooms
- [ ] In Home app, assign each device to correct room:
  - Bedroom TV Plug → Bedroom
  - Bedroom Air Purifier → Bedroom
  - Bedroom TV Light Strip → Bedroom
  - Office Hex Panels → Office
  - etc.
- [ ] Verify room structure matches HA areas

#### 3.5 Test Consolidated Bridge
- [ ] Test each device via Home app
- [ ] Test Siri commands
- [ ] Verify HomeKit automations still trigger
- [ ] Check HA automations still work

**Expected Outcome**:
- Single HA HomeKit Bridge
- All desired devices exposed
- No duplicates
- Organized rooms in Home app

### Phase 3 Rollback
**If issues**:
1. Remove new HomeKit Bridge integration from HA
2. Restore old HomeKit Bridge configuration from backup (Phase 1)
3. Restart HA
4. Re-pair HomeKit bridge(s) with Home app
5. **Consequence**: Back to multiple bridges, but all devices working

**Rollback Time**: 30 minutes

---

## Phase 4: Scrypted Camera Verification (Week 3)

### Goal
Verify Scrypted cameras are properly exposed with HKSV enabled.

### Pre-Requisites
- Scrypted already configured with Tapo cameras
- iCloud+ subscription active (for HKSV recording)

### Tasks

#### 4.1 Audit Scrypted Camera Configuration
- [ ] Open Scrypted web interface
- [ ] Verify all 4 Tapo cameras are added:
  - Kitchen Camera (192.168.1.205)
  - Hallway Camera
  - Entrance Camera
  - Office Camera
- [ ] Check Camera Bridge plugin is installed
- [ ] Verify HKSV is enabled for each camera

#### 4.2 Test Cameras in Home App
- [ ] View live feed for each camera
- [ ] Check HKSV recording status (Activity tab)
- [ ] Verify motion detection triggers
- [ ] Test camera tile in Home app (quick view)

#### 4.3 Verify RTSP Streams
- [ ] In Scrypted, check RTSP stream URLs for each camera
- [ ] Verify resolution settings (1080p recommended)
- [ ] Check frame rate (15-20 fps for HKSV)

**Expected Outcome**:
- All 4 cameras streaming to Home app
- HKSV recording enabled (10-day history with iCloud+)
- Motion detection working

### Phase 4 Rollback
**If issues**:
1. In Scrypted, remove camera from HomeKit
2. Re-add camera to Scrypted HomeKit Bridge
3. Re-pair camera in Home app (scan QR code in Scrypted)
4. **Alternative**: Add camera to HA Tapo integration and expose via HA HomeKit (no HKSV, but live view works)

**Rollback Time**: 15 minutes per camera

---

## Phase 5: Aqara Sensor Strategy (Week 4)

### Goal
Decide and implement Aqara sensor exposure strategy.

### Decision Point
**Option A**: Expose Aqara sensors via HA HomeKit Bridge (current recommendation)
**Option B**: Expose Aqara sensors via Aqara Hub M2 native HomeKit

### Tasks (Option A - Recommended)

#### 5.1 Configure Aqara Sensors in HA Bridge
- [ ] Verify Aqara Hub M2 integration is working in HA
- [ ] Identify Aqara sensor entities:
  - `binary_sensor.bedroom_safe_contact`
  - `binary_sensor.entrance_door_contact`
  - `binary_sensor.bathroom_motion`
  - `sensor.hallway_wireless_switch`
- [ ] Add desired sensors to HA HomeKit Bridge (entrance door sensor only?)
- [ ] Test in Home app

#### 5.2 Keep Automation Sensors in HA Only
- [ ] Verify these sensors are NOT exposed to HomeKit:
  - Bedroom Safe Contact (automation only)
  - Bathroom Motion (automation only)
- [ ] Test HA automations still trigger correctly

**Expected Outcome**:
- Entrance door sensor visible in Home app (for status checking)
- Automation-only sensors remain in HA backend
- All HA automations continue to work

### Tasks (Option B - Alternative)

#### 5.1 Pair Aqara Hub M2 with HomeKit
- [ ] In Home app, add Aqara Hub M2 as accessory
- [ ] All Aqara sensors will automatically appear
- [ ] Organize sensors into rooms

#### 5.2 Configure HA to Use Aqara Sensors
- [ ] Ensure HA automations reference Aqara sensor entities (via Aqara integration, not HomeKit Controller)
- [ ] Test automations

**Pros/Cons**:
- **Pro**: Faster sensor response in HomeKit
- **Con**: Sensors managed in two places (Home app + HA)

### Phase 5 Rollback
**Option A → Option B**:
1. Remove Aqara sensors from HA HomeKit Bridge
2. Pair Aqara Hub M2 with Home app
3. Update HA automations if needed

**Option B → Option A**:
1. Remove Aqara Hub M2 from Home app
2. Add Aqara sensors to HA HomeKit Bridge
3. Restart HA HomeKit integration

**Rollback Time**: 20 minutes

---

## Phase 6: Naming Standardization (Week 4)

### Goal
Standardize entity IDs and friendly names for consistency.

### Pre-Requisites
- All previous phases completed
- Devices working in HomeKit and HA

### Tasks

#### 6.1 Rename Entities in HA
- [ ] Follow naming convention: `[area]_[device_type]_[descriptor]`
- [ ] Update entity IDs:
  - `switch.bedroom_tv_plug` (already correct)
  - `light.bedroom_vanity_light` → `light.bedroom_light_vanity`
  - etc.
- [ ] Update friendly names to match HomeKit display:
  - "Bedroom TV Plug"
  - "Bedroom Vanity Light"

**Warning**: Renaming entity IDs will break automations, scripts, and dashboards that reference old IDs.

#### 6.2 Update Automations
- [ ] Search for all automations using renamed entities
- [ ] Update entity references
- [ ] Test each automation

#### 6.3 Update HomeKit Bridge Config
- [ ] Update `entity_config` section in HomeKit Bridge to use new entity IDs
- [ ] Restart HomeKit Bridge
- [ ] Verify devices still appear in Home app (may not require re-pairing)

**Expected Outcome**:
- Consistent naming across HA and HomeKit
- Easier Siri recognition
- Cleaner automation code

### Phase 6 Rollback
**If issues**:
1. Restore entity IDs to original names (via HA entity registry)
2. Restore `configuration.yaml` from backup
3. Restart HA
4. **Consequence**: Inconsistent naming remains

**Rollback Time**: 1 hour (tedious, involves checking all automations)

**Alternative**: Skip this phase if naming is "good enough" and not worth the risk.

---

## Phase 7: Homebridge Audit (Week 5)

### Goal
Determine if Homebridge is needed, and either configure or deprecate.

### Tasks

#### 7.1 Identify Homebridge Usage
- [ ] Check if Homebridge is currently running
- [ ] List any devices/plugins configured in Homebridge
- [ ] Determine if any devices are only accessible via Homebridge

#### 7.2 Option A - No Devices: Deprecate Homebridge
- [ ] Stop Homebridge service
- [ ] Remove from startup
- [ ] Document decision in architecture docs

#### 7.2 Option B - Has Devices: Configure Homebridge
- [ ] Document which devices use Homebridge
- [ ] Verify devices appear in Home app
- [ ] Add to device ownership map
- [ ] Keep Homebridge running

**Expected Outcome**:
- Clear decision on Homebridge usage
- No orphaned/unused services

### Phase 7 Rollback
**If Homebridge was needed**:
1. Restart Homebridge service
2. Re-pair with Home app (if removed)

**Rollback Time**: 10 minutes

---

## Phase 8: Final Testing & Documentation (Week 5-6)

### Goal
Comprehensive testing and user documentation.

### Tasks

#### 8.1 End-to-End Testing
- [ ] **Device Control**: Test every HomeKit device (on/off, dim, color)
- [ ] **Siri Commands**: Test voice control for each room
- [ ] **Home App Automations**: Verify triggers and actions
- [ ] **HA Automations**: Test all automations referencing HomeKit devices
- [ ] **Scrypted Cameras**: Verify live view and HKSV recording
- [ ] **Adaptive Lighting**: Verify Hue lights change temperature throughout day

#### 8.2 Performance Testing
- [ ] Measure response time for each device (Home app tap → action)
- [ ] Check for any "No Response" devices in Home app
- [ ] Verify HA is not overloaded (check CPU/memory usage)

#### 8.3 Create User Documentation
- [ ] **Device List**: Which devices are in HomeKit, which are HA-only
- [ ] **Siri Commands**: Cheat sheet for voice control
- [ ] **Home App Guide**: Room structure, scenes, automations
- [ ] **Troubleshooting**: Common issues and fixes
- [ ] **Rollback Procedures**: Quick reference for each phase

#### 8.4 Final Backup
- [ ] Create HA snapshot (post-migration)
- [ ] Export HomeKit Bridge config (final version)
- [ ] Screenshot Home app (final structure)
- [ ] Backup Scrypted config (final version)

**Deliverable**: `migration-complete.md` with:
- Final architecture diagram
- Device ownership summary
- Known issues (if any)
- Maintenance notes

---

## Phase 9: Optimization & Cleanup (Ongoing)

### Goal
Fine-tune performance and remove cruft.

### Tasks

#### 9.1 Remove Unused Integrations
- [ ] Audit HA integrations list
- [ ] Remove integrations no longer needed (e.g., old HomeKit bridges)
- [ ] Clean up unused entities

#### 9.2 Optimize Automations
- [ ] Review all automations for efficiency
- [ ] Consolidate duplicate automations
- [ ] Add conditions to prevent unnecessary triggers

#### 9.3 Monitor Performance
- [ ] Check HA logs for errors
- [ ] Monitor HomeKit responsiveness
- [ ] Track HKSV recording usage (iCloud storage)

---

## Rollback: Full Migration Revert

### If Everything Fails
**Nuclear Option**: Restore HA snapshot from Phase 1, Backup 1.1.

**Procedure**:
1. HA Supervisor → Backups → Restore snapshot from Phase 1
2. Wait for HA to restart (5-10 minutes)
3. Remove all HomeKit bridges from Home app
4. Re-pair original HomeKit bridge(s) from restored HA
5. Verify all devices return to original state

**Consequence**: All migration work lost, back to original configuration.

**Time**: 30 minutes

---

## Risk Assessment

| Phase | Risk Level | Consequences if Failed | Rollback Difficulty |
|-------|------------|------------------------|---------------------|
| Phase 1 | Low | None (read-only) | N/A |
| Phase 2 | Low | Hue lights offline in HomeKit | Easy (15 min) |
| Phase 3 | Medium | All HA devices offline in HomeKit | Medium (30 min) |
| Phase 4 | Low | Cameras offline in HomeKit | Easy (15 min) |
| Phase 5 | Low | Sensors not visible in HomeKit | Easy (20 min) |
| Phase 6 | High | Broken automations | Hard (1 hour) |
| Phase 7 | Low | Homebridge devices offline | Easy (10 min) |
| Phase 8 | Low | None (testing only) | N/A |

**Recommendation**:
- Phases 1-5 are low-risk, proceed with confidence.
- **Phase 6 (naming)**: Consider skipping if current naming is acceptable. High effort, high risk.
- Always create backups before each phase.

---

## Timeline Summary

| Week | Phases | Focus |
|------|--------|-------|
| Week 1 | Phase 1 | Audit & Baseline |
| Week 2 | Phase 2 | Hue Bridge Cleanup |
| Week 3 | Phase 3-4 | Consolidate Bridges + Cameras |
| Week 4 | Phase 5-6 | Aqara Strategy + Naming (optional) |
| Week 5-6 | Phase 7-8 | Homebridge + Testing |
| Ongoing | Phase 9 | Optimization |

**Total Estimated Time**: 5-6 weeks (assuming 2-3 hours per week)

**Fast-Track Option**: Phases 1-5 can be completed in 1 weekend (6-8 hours) if desired.

---

## Next Steps

1. **User Approval**: Review this roadmap and approve phases
2. **Schedule Migration**: Choose start date and time windows
3. **Backup Everything**: Execute Phase 1 backup tasks
4. **Begin Phase 2**: Start with low-risk Hue cleanup

**Questions for User**:
- Preferred timeline: Slow and steady (5-6 weeks) or fast-track (1-2 weekends)?
- Phase 6 (naming): Worth the risk, or skip?
- Homebridge: Currently in use, or can we deprecate?
- Daily routines: Do you want to add routine documentation before migration?
