# Phase 2: Home Assistant VM Reliability & Optimization

**Last Updated:** 2026-01-22
**Status:** ✅ COMPLETE
**Completed:** 2026-01-22
**Goal:** Validate and optimize Home Assistant VM (UTM) for reliable, performant operation

---

## Overview

Phase 2 validates and optimizes your Home Assistant VM running on Mac mini M4 via UTM. This includes:
- Documenting current VM configuration
- Optimizing VM resource allocation
- Validating mDNS/discovery functionality
- Creating backup & snapshot strategy
- Establishing performance baselines
- Verifying integrations post-Phase 1

---

## Pre-Requisites

### Completed
- ✅ Phase 1 (Network Segmentation) complete
- ✅ HA VM accessible at `http://192.168.1.21:8123`
- ✅ Mac mini dual-homed (192.168.1.20 + 192.168.2.10)

### Required Access
- [ ] UTM app on Mac mini (to view/modify VM settings)
- [ ] Home Assistant admin credentials (with MFA)
- [ ] SSH access to Mac mini (optional, for advanced diagnostics)

---

## Phase 2 Checklist

### Step 1: Document Current VM Configuration

#### 1.1 VM Resource Allocation
**Location:** UTM app → Select HA VM → Edit → System

- [ ] **CPU Cores:** __________ cores allocated
  - **Recommendation:** 2-4 cores for typical home setup (35 devices)
  - **Current Host:** Mac mini M4 (10 total cores)
  - **Optimal:** 2 cores (balanced) or 4 cores (if HA feels sluggish)

- [ ] **RAM:** __________ GB allocated
  - **Recommendation:** 4 GB minimum, 8 GB optimal
  - **Too low:** HA slowness, OOM errors in logs
  - **Too high:** Wastes host RAM, no performance benefit beyond 8 GB

- [ ] **Storage Size:** __________ GB allocated
  - **Recommendation:** 32 GB minimum, 64 GB optimal
  - **Used Space:** Check in HA → **Settings → System → Storage**
  - **Growth rate:** ~1-2 GB/month (depends on recorder retention, backups)

#### 1.2 VM Network Configuration
**Location:** UTM app → Select HA VM → Edit → Network

- [ ] **Network Mode:** [Select one]
  - ⚪ Shared Network (NAT) - VM gets IP via UTM's internal DHCP
  - ⚪ Bridged Network (Recommended) - VM gets IP from Router #1 DHCP
  - ⚪ Host Only - VM isolated from LAN
  - ⚪ Other: __________

- [ ] **Current Configuration:** __________ mode
  - **Expected for Phase 1:** Bridged mode (VM at 192.168.1.21 on Router #1 LAN)
  - **Verify:** HA accessible from other devices on 192.168.1.x subnet

- [ ] **MAC Address:** __________ (static or auto-generated)
  - **Note:** If using DHCP reservation, this must match Router #1 reservation

#### 1.3 HA Install Type & Version
**Location:** Home Assistant → **Settings → System → Repairs/Info**

- [ ] **Install Type:** [Select one]
  - ⚪ Home Assistant Operating System (HA OS) - Full OS, includes Supervisor
  - ⚪ Home Assistant Supervised - HA Supervisor on Debian/Ubuntu
  - ⚪ Home Assistant Container - Docker container only
  - ⚪ Home Assistant Core - Python venv (no Supervisor)

- [ ] **Current Install:** __________
  - **Expected:** HA OS (most common for UTM VMs)

- [ ] **HA Version:** __________
  - **Check for updates:** Settings → System → Updates
  - **Recommendation:** Stay on stable releases (avoid beta unless needed)

- [ ] **Supervisor Version (if HA OS):** __________

#### 1.4 Current Disk Usage
**Location:** Home Assistant → **Settings → System → Storage**

- [ ] **Total Disk:** __________ GB
- [ ] **Used:** __________ GB (____%)
- [ ] **Free:** __________ GB

**Breakdown by component:**
- [ ] **Database (recorder):** __________ GB
- [ ] **Backups:** __________ GB
- [ ] **Add-ons:** __________ GB
- [ ] **Other:** __________ GB

---

### Step 2: Performance Baseline

#### 2.1 Current Response Times
Test HomeKit control response (from Home app):

- [ ] **Test 1:** Toggle "Bedroom Vanity Light" (Hue via HA Bridge)
  - **Response time:** __________ seconds
  - **Expected:** <2 seconds

- [ ] **Test 2:** Toggle "Kitchen Camera" privacy mode (Scrypted via HKSV)
  - **Response time:** __________ seconds
  - **Expected:** <3 seconds

- [ ] **Test 3:** Open HA web UI (`http://192.168.1.21:8123`)
  - **Load time:** __________ seconds
  - **Expected:** <5 seconds (first load), <2 seconds (cached)

#### 2.2 VM Uptime & Stability
**Location:** Mac mini Terminal or HA → Developer Tools → System Log

- [ ] **HA Uptime:** __________ (days/hours)
  - Check via: `uptime` in HA Terminal (if available) or System Log
- [ ] **Last Restart Reason:** __________ (manual, crash, update, etc.)
- [ ] **Recent Errors:** Review HA logs for crashes, OOM errors, timeouts

#### 2.3 Integration Health
**Location:** Home Assistant → **Settings → Devices & Services**

- [ ] **Hue Bridge:** [Status] __________ (Expected: Connected)
- [ ] **Aqara Hub M2:** [Status] __________ (Expected: Connected)
- [ ] **Kasa (Plugs):** [Status] __________ (Expected: Connected)
- [ ] **Govee:** [Status] __________ (Expected: Connected)
- [ ] **HomeKit Bridge:** [Status] __________ (Expected: Running)
- [ ] **Tapo (Cameras):** [Status] __________ (if using HA Tapo integration, otherwise N/A)

**If any integration shows "Failed" or "Unavailable":**
1. Click integration → **Reload**
2. Check device reachability (ping from Mac mini's IoT interface if on Router #2)
3. Review integration logs

---

### Step 3: Optimize VM Resources (If Needed)

#### 3.1 CPU Optimization

**Current allocation:** __________ cores
**Recommendation:**
- **2 cores:** If HA response is fast (<2 sec), no performance issues
- **4 cores:** If HA feels sluggish, automation delays >5 seconds
- **DON'T allocate more than 4 cores** (diminishing returns for HA workload)

**To change:**
1. Shut down HA VM: UTM app → Select VM → Stop
2. Edit VM: UTM app → Select VM → Edit → System → CPU Cores
3. Save and start VM
4. Verify HA boots successfully

#### 3.2 RAM Optimization

**Current allocation:** __________ GB
**Recommendation:**
- **4 GB:** Minimum for HA OS with 35 devices + typical integrations
- **8 GB:** Optimal for smooth operation, room for add-ons (AdGuard Home, VSCode Server, etc.)
- **16 GB:** Overkill for this setup (no benefit)

**To change:**
1. Shut down HA VM: UTM app → Select VM → Stop
2. Edit VM: UTM app → Select VM → Edit → System → Memory
3. Save and start VM
4. Verify HA boots successfully
5. Check HA System → Storage (should show increased available RAM)

#### 3.3 Storage Optimization

**Current allocation:** __________ GB
**Current usage:** __________ GB

**If usage > 80%:**
1. **Reduce recorder retention:** HA → **Settings → System → Recorder**
   - Default: 10 days
   - Recommended: 7 days (unless historical data needed)
   - **Action:** Change to __________ days
2. **Clean old backups:** HA → **Settings → System → Backups**
   - Keep last 7 backups locally, delete older ones
   - **Action:** Deleted __________ old backups
3. **Increase VM disk size (if needed):**
   - UTM app → Select VM → Edit → Drives → Resize disk
   - **New size:** __________ GB (64 GB recommended)
   - **Note:** Cannot shrink disk, only grow

---

### Step 4: Discovery & mDNS Validation

#### 4.1 mDNS/Bonjour Test (Critical for HomeKit)

**Goal:** Verify HomeKit Bridge discoverable by iOS devices on Router #1 subnet (192.168.1.x)

**Test from iPhone/iPad (on "EnterAtYourOwnRisk" SSID):**

- [ ] **Test 1:** Open Home app → Add Accessory → Should show "Home Assistant"
  - **Result:** [Pass/Fail] __________
  - **Expected:** Pass (mDNS works, same subnet as HA Bridge)

- [ ] **Test 2:** Control any HA-exposed device (e.g., "Bedroom Vanity Light")
  - **Response time:** __________ seconds
  - **Expected:** <2 seconds

**If mDNS fails (HomeKit devices "No Response"):**
1. **Verify HA Bridge is running:** HA → Settings → Devices & Services → HomeKit Bridge
2. **Restart HomeKit Bridge:** Click integration → **Restart**
3. **Check macOS Firewall (Mac mini host):**
   - System Settings → Network → Firewall → Allow incoming connections
4. **Verify Router #1 settings:**
   - UPnP: Should be DISABLED (per Phase 1)
   - Multicast/IGMP: Should be ENABLED (for mDNS)

#### 4.2 Integration Discovery (IoT Devices Behind Router #2)

**Goal:** Verify HA can reach IoT devices via Mac mini's IoT interface (192.168.2.10)

**From Mac mini Terminal:**

```bash
# Verify Mac mini IoT interface is active
ifconfig | grep 192.168.2
# Expected output: 192.168.2.10

# Test connectivity to Hue Bridge (Router #2 LAN)
ping -c 4 192.168.2.50
# Expected: 4 packets transmitted, 4 received

# Test connectivity to Aqara Hub
ping -c 4 192.168.2.51
# Expected: 4 packets transmitted, 4 received

# Test connectivity to Kitchen Camera
ping -c 4 192.168.2.150
# Expected: 4 packets transmitted, 4 received
```

**Results:**
- [ ] Hue Bridge reachable: [Yes/No] __________
- [ ] Aqara Hub reachable: [Yes/No] __________
- [ ] Kitchen Camera reachable: [Yes/No] __________

**If any device unreachable:**
1. Verify Mac mini's second Ethernet cable connected to Router #2 LAN
2. Check Router #2 DHCP Connections (should show Mac mini at 192.168.2.10)
3. Verify device IP reservations in Router #2

---

### Step 5: Backup & Snapshot Strategy

#### 5.1 Create Immediate Backup (Pre-Phase 2)

**Before making any changes:**

1. **Create HA backup:**
   - HA → **Settings → System → Backups → Create Backup**
   - **Name:** "Pre-Phase2-Baseline-2026-01-22"
   - **Include:** All (full backup)
   - **Wait for completion:** __________ minutes

2. **Download backup to safe location:**
   - Click backup → **Download** → Save to Mac mini Desktop
   - **Move to:** `~/SmartHome-Backups/Phase2/`
   - **Verify file size:** __________ MB (typical: 100-500 MB)

3. **Create UTM VM snapshot:**
   - UTM app → Select HA VM → Right-click → **Snapshot**
   - **Name:** "Pre-Phase2-VM-Snapshot-2026-01-22"
   - **Wait for completion:** __________ seconds

#### 5.2 Automated Backup Plan

**Goal:** Weekly automatic backups + manual backups before major changes

**HA Automatic Backup Setup:**
1. HA → **Settings → System → Backups → Automatic Backups**
2. **Enable automatic backups:** [Toggle ON]
3. **Frequency:** Weekly (every Sunday at 3:00 AM)
4. **Retention:** Keep last 4 backups (1 month)
5. **Location:** Local HA storage (default)

**Offsite Backup Plan:**
- **Method:** Manual download to Mac mini, then sync to PC/cloud
- **Frequency:** Monthly (or after major changes)
- **Destination:** `~/SmartHome-Backups/` on Mac mini → sync to PC via AirDrop/network share
- **Retention:** Keep last 12 monthly backups (1 year)

#### 5.3 UTM VM Snapshot Schedule

**Snapshots:** Before major changes only (not automatic)

**When to create snapshot:**
- ✅ Before Phase 2 changes (done in Step 5.1)
- ✅ Before HA version upgrades
- ✅ Before major integration changes
- ✅ Before Phase 3, 4, etc.

**Snapshot retention:**
- Keep last 3 snapshots
- Delete older snapshots to free disk space (UTM snapshots stored on Mac mini SSD)

---

### Step 6: Performance Validation Tests

#### 6.1 HomeKit Response Time (Post-Optimization)

- [ ] **Test 1:** Toggle "Bedroom Vanity Light"
  - **Response time:** __________ seconds
  - **Baseline (Step 2.1):** __________ seconds
  - **Change:** [Faster/Same/Slower] __________

- [ ] **Test 2:** Toggle "Kitchen Camera" privacy mode
  - **Response time:** __________ seconds
  - **Baseline (Step 2.1):** __________ seconds
  - **Change:** [Faster/Same/Slower] __________

#### 6.2 HA Web UI Load Time

- [ ] **Clear browser cache:** Cmd+Shift+R (Safari/Chrome)
- [ ] **Load HA:** `http://192.168.1.21:8123`
  - **Load time:** __________ seconds
  - **Baseline (Step 2.1):** __________ seconds
  - **Change:** [Faster/Same/Slower] __________

#### 6.3 Integration Response (IoT Devices)

- [ ] **Test Hue light via HA UI:** Toggle "Bedroom Vanity Light" in HA
  - **Response time:** __________ seconds
  - **Expected:** <1 second (local Zigbee, fast)

- [ ] **Test Aqara sensor:** Check "Door Contact Sensor" state in HA
  - **Trigger sensor:** Open/close door
  - **HA update delay:** __________ seconds
  - **Expected:** <2 seconds

- [ ] **Test Kasa plug:** Toggle "Bedroom TV Plug" in HA
  - **Response time:** __________ seconds
  - **Expected:** 2-4 seconds (cloud-based, may be slower)

#### 6.4 VM Stability (72-Hour Test)

**Goal:** Ensure VM remains stable after optimization

- [ ] **Start monitoring:** Record current uptime: __________ (date/time)
- [ ] **Check after 72 hours:** New uptime: __________ (date/time)
- [ ] **Verify no crashes:** HA logs show no OOM errors, crashes, or restarts
- [ ] **Verify no service interruptions:** HomeKit devices remain responsive

---

### Step 7: Update Documentation

#### 7.1 Update ha-baseline.md

**Fill in TBD items with discovered values:**

- [ ] **VM Resources (Line 14-17):**
  - CPU cores: __________
  - RAM: __________
  - Disk: __________

- [ ] **HA Install Type (Line 11):**
  - Type: __________ (HA OS / Supervised / Container / Core)

- [ ] **VM Networking (Line 12):**
  - Mode: Bridged (confirmed in Step 1.2)

- [ ] **Installed Add-ons (Line 118):**
  - List: __________

- [ ] **Last Backup (Line 167):**
  - Date: 2026-01-22 (Pre-Phase2 backup)

**How to update:**
1. Open [ha-baseline.md](ha-baseline.md)
2. Replace TBD values with actual data from Steps 1-6
3. Save file
4. Commit to git: `git add ha-baseline.md && git commit -m "Update HA baseline with Phase 2 discovered values"`

#### 7.2 Create Performance Baseline Document

**File:** `02-HomeAssistant/ha-performance-baseline.md`

**Contents:**
- VM resource allocation (CPU, RAM, disk)
- Response time benchmarks (HomeKit, HA UI, integrations)
- Integration health status
- Backup/snapshot schedule
- Next review date: 2026-04-22 (3 months)

---

## Phase 2 Completion Checklist

### Configuration Documented
- [ ] VM resource allocation verified (CPU, RAM, disk)
- [ ] VM network mode confirmed (bridged)
- [ ] HA version and install type documented
- [ ] Current disk usage recorded

### Performance Optimized
- [ ] VM resources adjusted if needed (CPU, RAM)
- [ ] Recorder retention optimized (if disk usage high)
- [ ] Performance baseline established (response times)

### Discovery Validated
- [ ] mDNS/HomeKit discovery working (<2 sec response)
- [ ] IoT device reachability confirmed (Mac mini dual-homed)
- [ ] All integrations showing "Connected" status

### Backup Strategy Implemented
- [ ] Pre-Phase2 backup created and downloaded
- [ ] UTM VM snapshot created
- [ ] Automatic backup schedule configured (weekly)
- [ ] Offsite backup plan documented (monthly manual)

### Documentation Updated
- [ ] ha-baseline.md updated with actual values
- [ ] Performance baseline document created
- [ ] Phase 2 completion summary created

---

## Rollback Procedure

### If Phase 2 Changes Break HA

**Option 1: Restore HA Backup**
1. HA → **Settings → System → Backups**
2. Select "Pre-Phase2-Baseline-2026-01-22"
3. Click **Restore** → Wait 5-10 minutes
4. Verify HA accessible and integrations working

**Option 2: Revert UTM VM Snapshot**
1. UTM app → Select HA VM → Stop
2. Right-click VM → **Snapshots** → Select "Pre-Phase2-VM-Snapshot-2026-01-22"
3. Click **Restore**
4. Start VM
5. Verify HA accessible

**Option 3: Revert VM Resource Changes**
1. UTM app → Select HA VM → Stop
2. Edit VM → System → Change CPU/RAM back to original values
3. Save and start VM

---

## Validation Results

**Completed:** 2026-01-22

### Network Reliability Tests (All PASS)
- ✅ **HA IP reservation enforced on Router #1:** 192.168.1.21 (confirmed after DHCP drift fix)
- ✅ **Windows validation:** ping 192.168.1.21 PASS, TCP 8123 PASS, HTTP 200 PASS
- ✅ **Host reboot test (Mac mini):** PASS; HA reachable on .21 after reboot
- ✅ **Router #2 reboot resiliency:** PASS; HA stayed/recovered, cameras recovered
- ✅ **Router #1 reboot resiliency:** PASS; HA stayed/recovered on .21, HomeKit cameras recovered, reservation persisted

---

## Success Criteria

✅ **Phase 2 Complete When:**
1. VM resource allocation documented and optimized
2. HA VM stable (72-hour uptime test passed)
3. HomeKit response time <2 seconds (no mDNS issues)
4. All integrations showing "Connected" (no discovery issues)
5. Backup strategy implemented (automatic weekly + UTM snapshots)
6. Performance baseline documented for future comparison
7. ha-baseline.md updated with actual configuration

---

## Next Steps After Phase 2

Once Phase 2 is complete:
1. **Phase 2.5:** Home Model Governance & Normalization (see phase2.5-governance-normalization.md)
2. **Phase 3:** Consolidate HomeKit Bridges (merge multiple bridges into single filtered bridge)
3. **Phase 6:** Automation Library (document and standardize automations)

---

## Troubleshooting

### Issue: HA VM won't start after resource change

**Cause:** Too few resources allocated (e.g., 1 CPU core, 2 GB RAM)

**Solution:**
1. UTM app → Select VM → Edit → System
2. Increase CPU to 2 cores minimum
3. Increase RAM to 4 GB minimum
4. Save and start VM

### Issue: HomeKit devices "No Response" after optimization

**Cause:** mDNS/Bonjour blocked or HA Bridge offline

**Solution:**
1. Restart HA: **Settings → System → Restart**
2. Restart HomeKit Bridge: **Settings → Devices & Services → HomeKit Bridge → Restart**
3. Restart Apple TV hubs: **Settings → System → Restart**
4. Verify macOS Firewall allows incoming connections (Mac mini)

### Issue: IoT integrations "Unavailable" after optimization

**Cause:** Mac mini IoT interface down or misconfigured

**Solution:**
1. Verify Mac mini second Ethernet cable connected to Router #2
2. Check Mac mini network interfaces: `ifconfig | grep 192.168.2`
3. Restart Mac mini if necessary
4. Reload integrations in HA: **Settings → Devices & Services → [Integration] → Reload**

---

**Last Updated:** 2026-01-22
**Status:** ✅ COMPLETE
**Completed:** 2026-01-22
