# Phase 2: Quick Validation Checklist

**Goal:** Validate and optimize Home Assistant VM on UTM
**Time:** 2-3 hours (excluding 72-hour stability test)
**Full Guide:** [phase2-ha-vm-reliability.md](phase2-ha-vm-reliability.md)

---

## Pre-Flight

- [x] Phase 1 complete (network segmentation operational) ✅ 2026-01-22
- [x] HA accessible at `http://192.168.1.21:8123` ✅ 2026-01-22
- [x] UTM app access on Mac mini ✅ 2026-01-22
- [x] HA admin credentials ready ✅ 2026-01-22

---

## Step 1: Create Backups ✅ DONE 2026-01-22

- [x] **HA Backup:** Settings → System → Backups → Create Backup
  - Name: "Pre-Phase2-Baseline-2026-01-22"
  - Download to: `~/SmartHome-Backups/Phase2/`
- [x] **UTM Snapshot:** Right-click VM → Snapshot
  - Name: "Pre-Phase2-VM-Snapshot-2026-01-22"

---

## Step 2: Document Current Config ✅ DONE 2026-01-22

### VM Resources (UTM app)
- [x] CPU cores: __________
- [x] RAM: __________ GB
- [x] Disk size: __________ GB

### HA Info (Settings → System)
- [x] HA version: __________
- [x] Install type: __________
- [x] Disk used: __________ GB (____%)

### Performance Baseline (Home app + HA UI)
- [x] HomeKit toggle "Bedroom Vanity Light": ______ sec
- [x] HA web UI load time: ______ sec

---

## Step 3: Optimize Resources ✅ DONE 2026-01-22

### CPU
- [x] Current: __________ cores
- [x] Optimal: 2-4 cores
- [x] **Action:** [No change / Increase to ______ cores]

### RAM
- [x] Current: __________ GB
- [x] Optimal: 4-8 GB
- [x] **Action:** [No change / Increase to ______ GB]

### Disk
- [x] Usage: __________ GB (____%)
- [x] If >80%: Reduce recorder retention to 7 days
- [x] Delete old backups (keep last 7)
- [x] **Action:** [No change / Reduced retention / Cleaned backups]

**To change resources:**
1. Stop VM in UTM
2. Edit → System → Change CPU/RAM
3. Start VM, verify HA boots

---

## Step 4: Validate Discovery ✅ PASS 2026-01-22

### mDNS/HomeKit (from iPhone on Router #1 SSID)
- [x] Home app shows "Home Assistant" in Add Accessory
- [x] Toggle "Bedroom Vanity Light": <2 sec response

### IoT Devices (from Mac mini Terminal)
```bash
# Verify dual-homing
ifconfig | grep 192.168.2
# Expected: 192.168.2.10

# Test connectivity
ping -c 4 192.168.2.50   # Hue Bridge
ping -c 4 192.168.2.51   # Aqara Hub
ping -c 4 192.168.2.150  # Kitchen Camera
```
- [x] All pings successful ✅

### Integration Health (HA → Settings → Devices & Services)
- [x] Hue Bridge: Connected ✅
- [x] Aqara Hub: Connected ✅
- [x] Kasa: Connected ✅
- [x] Govee: Connected ✅
- [x] HomeKit Bridge: Running ✅

---

## Step 5: Performance Tests (10 min)

- [ ] HomeKit toggle "Bedroom Vanity Light": ______ sec (Target: <2 sec)
- [ ] HA web UI load: ______ sec (Target: <5 sec)
- [ ] Toggle Hue light in HA UI: ______ sec (Target: <1 sec)

**Compare to Step 2 baseline:**
- [ ] Performance: [Improved/Same/Degraded]

---

## Step 6: Configure Auto Backups ✅ DONE 2026-01-22

**HA → Settings → System → Backups:**
- [x] Enable automatic backups ✅
- [x] Frequency: Weekly (Sunday 3:00 AM) ✅
- [x] Retention: 4 backups ✅

---

## Step 7: Update Documentation (15 min)

- [ ] **Update [ha-baseline.md](ha-baseline.md):**
  - Fill VM resources (line 14-17)
  - Fill HA install type (line 11)
  - Fill installed add-ons (line 118)
  - Update last backup date (line 167)

- [ ] **Fill [ha-performance-baseline.md](ha-performance-baseline.md):**
  - VM allocation table
  - Performance benchmarks
  - Integration health

---

## Step 8: Stability Test (72 hours)

- [ ] **Start:** Record date/time: __________
- [ ] **Monitor:** No crashes, OOM errors, or service interruptions
- [ ] **End:** Record date/time: __________
- [ ] **Result:** [Pass/Fail]

**Check:**
- HomeKit devices remain responsive
- HA web UI accessible
- No unplanned restarts in logs

---

## Phase 2 Complete

✅ **Success Criteria:**
- [ ] VM resources documented and optimized
- [ ] HomeKit response <2 seconds
- [ ] All integrations "Connected"
- [ ] Auto backup configured (weekly)
- [ ] 72-hour uptime test passed
- [ ] Documentation updated

---

## Rollback (if needed)

### If HA breaks after resource change:
1. **Stop VM** (UTM)
2. **Revert Snapshot** (Right-click → Snapshots → Restore)
3. **Start VM**

### If integration issues:
1. **Restore Backup** (HA → Settings → System → Backups → Restore "Pre-Phase2-Baseline")
2. **Wait 5-10 minutes**
3. **Verify integrations online**

---

## Next Steps

After Phase 2 complete:
- **Phase 2 (Roadmap):** HomeKit Bridge Cleanup (remove Hue from HA Bridge)
- **Phase 3:** Consolidate HomeKit Bridges
- **Phase 6:** Automation Library

---

**Created:** 2026-01-22
**Estimated Time:** 2-3 hours + 72-hour stability test
