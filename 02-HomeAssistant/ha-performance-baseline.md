# Home Assistant Performance Baseline

**Last Updated:** 2026-01-22
**Phase:** Phase 2 (HA VM Reliability & Optimization)
**Next Review:** 2026-04-22 (3 months)

---

## VM Resource Allocation

### UTM VM Configuration

| Resource | Allocated | Notes |
|----------|-----------|-------|
| **CPU Cores** | __________ | Recommendation: 2-4 cores for 35 devices |
| **RAM** | __________ GB | Recommendation: 4 GB min, 8 GB optimal |
| **Disk Size** | __________ GB | Recommendation: 32 GB min, 64 GB optimal |
| **Disk Used** | __________ GB (____%) | Target: <80% usage |
| **Network Mode** | Bridged | VM on Router #1 LAN (192.168.1.21) |

### Mac mini M4 Host

| Resource | Total | HA VM Allocation | % Used by HA |
|----------|-------|------------------|--------------|
| **CPU Cores** | 10 | __________ | ____% |
| **RAM** | __________ GB | __________ GB | ____% |
| **Storage** | __________ GB SSD | __________ GB | ____% |

**Notes:**
- Mac mini also hosts Scrypted (native app, not in VM)
- Leave resources for host macOS + Scrypted + other VMs (if any)

---

## Home Assistant Configuration

### Version Info

| Component | Version | Last Updated |
|-----------|---------|--------------|
| **HA Core** | __________ | __________ |
| **Supervisor** | __________ | __________ (if HA OS) |
| **Operating System** | __________ | __________ (if HA OS) |
| **Install Type** | __________ | HA OS / Supervised / Container / Core |

### Recorder Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| **Database Type** | SQLite (default) | Or MariaDB/PostgreSQL if configured |
| **Retention Period** | __________ days | Default: 10 days; Recommended: 7 days |
| **Database Size** | __________ GB | Target: <5 GB for 7-day retention |
| **Purge Interval** | Daily (3:00 AM) | Automatic cleanup |

**Optimization:**
- If DB >5 GB: Reduce retention to 5-7 days
- If DB >10 GB: Consider excluding high-frequency sensors

---

## Performance Benchmarks

### HomeKit Response Times (via Home App)

| Device | Protocol | Baseline | Target | Status |
|--------|----------|----------|--------|--------|
| Bedroom Vanity Light | Hue (Zigbee) → HA Bridge | ______ sec | <2 sec | [Pass/Fail] |
| Bedroom TV Plug | Kasa (WiFi) → HA Bridge | ______ sec | <2 sec | [Pass/Fail] |
| Kitchen Camera (Privacy Mode) | Scrypted HKSV | ______ sec | <3 sec | [Pass/Fail] |
| Door Contact Sensor | Aqara (Zigbee) → HA Bridge | ______ sec | <2 sec | [Pass/Fail] |

**Notes:**
- All tests from iPhone on "EnterAtYourOwnRisk" SSID (Router #1, 192.168.1.x)
- Tests include end-to-end HomeKit → HA → Device response
- mDNS discovery working (same subnet as HomeKit hubs)

### HA Web UI Load Times

| Test | Baseline | Target | Status |
|------|----------|--------|--------|
| **Initial Load** (cold cache) | ______ sec | <5 sec | [Pass/Fail] |
| **Cached Load** | ______ sec | <2 sec | [Pass/Fail] |
| **Dashboard Switch** (between tabs) | ______ sec | <1 sec | [Pass/Fail] |

**Test URL:** `http://192.168.1.21:8123`
**Browser:** Safari/Chrome (specify: __________)

### Integration Response Times (HA UI)

| Integration | Device | Action | Baseline | Target | Status |
|-------------|--------|--------|----------|--------|--------|
| **Hue Bridge** | Bedroom Vanity Light | Toggle | ______ sec | <1 sec | [Pass/Fail] |
| **Aqara Hub** | Door Contact Sensor | State update | ______ sec | <2 sec | [Pass/Fail] |
| **Kasa** | Bedroom TV Plug | Toggle | ______ sec | <4 sec | [Pass/Fail] |
| **Govee** | Hex Panels | Toggle | ______ sec | <4 sec | [Pass/Fail] |

**Notes:**
- Hue/Aqara: Local Zigbee (fast)
- Kasa/Govee: Cloud-based (slower, acceptable)
- All devices reachable via Mac mini's dual interfaces (Router #1 + Router #2)

---

## System Health

### Uptime & Stability

| Metric | Current Value | Target | Status |
|--------|---------------|--------|--------|
| **HA Uptime** | __________ days | >30 days | [Pass/Fail] |
| **Last Restart Reason** | __________ | Manual/Planned | [Pass/Fail] |
| **Unplanned Restarts (Last 30 Days)** | __________ | 0 | [Pass/Fail] |
| **OOM Errors (Last 30 Days)** | __________ | 0 | [Pass/Fail] |

**Check via:**
- HA → **Settings → System → Logs** (filter for "error", "crash", "OOM")
- HA → **Developer Tools → System Log**

### Integration Health

| Integration | Status | Devices | Last Poll | Issues |
|-------------|--------|---------|-----------|--------|
| **Hue Bridge** | __________ | 7 lights | __________ | __________ |
| **Aqara Hub M2** | __________ | 11 devices | __________ | __________ |
| **Kasa** | __________ | 2 plugs | __________ | __________ |
| **Govee** | __________ | 2 devices | __________ | __________ |
| **HomeKit Bridge** | __________ | 22 exposed | N/A | __________ |
| **Tapo (Cameras)** | __________ | 0-4 | __________ | __________ |

**Status Key:**
- ✅ Connected: Integration online, all devices reachable
- ⚠️ Partial: Some devices unavailable
- ❌ Failed: Integration offline or error

**Check via:** HA → **Settings → Devices & Services**

---

## Network & Discovery

### mDNS/Bonjour Discovery

| Test | Status | Notes |
|------|--------|-------|
| **HomeKit Bridge discoverable** | [Pass/Fail] | From iPhone on Router #1 SSID |
| **HA accessible from Router #1 LAN** | [Pass/Fail] | `http://192.168.1.21:8123` |
| **Multicast working** | [Pass/Fail] | No mDNS issues, <2 sec response |

**Phase 1 Architecture:**
- All HomeKit bridges (HA, Scrypted) on Router #1 subnet (192.168.1.x)
- All HomeKit hubs (Apple TVs, iPad) on Router #1 subnet
- mDNS works within Router #1 subnet (no reflection needed)

### IoT Device Reachability (Router #2)

| Device | IP Address | Reachable from Mac mini | Reachable from HA | Notes |
|--------|------------|-------------------------|-------------------|-------|
| **Hue Bridge** | 192.168.2.50 | [Yes/No] | [Yes/No] | Via Mac mini IoT interface |
| **Aqara Hub** | 192.168.2.51 | [Yes/No] | [Yes/No] | Via Mac mini IoT interface |
| **Kitchen Camera** | 192.168.2.150 | [Yes/No] | [Yes/No] | Via Mac mini IoT interface |
| **Hallway Camera** | 192.168.2.151 | [Yes/No] | [Yes/No] | Via Mac mini IoT interface |

**Test from Mac mini Terminal:**
```bash
# Verify IoT interface active
ifconfig | grep 192.168.2
# Expected: 192.168.2.10

# Test connectivity
ping -c 4 192.168.2.50   # Hue Bridge
ping -c 4 192.168.2.51   # Aqara Hub
ping -c 4 192.168.2.150  # Kitchen Camera
```

---

## Backup Strategy

### HA Backups (Internal)

| Setting | Configuration | Status |
|---------|---------------|--------|
| **Automatic Backups** | [Enabled/Disabled] | Target: Enabled |
| **Frequency** | Weekly (Sunday 3:00 AM) | Target: Weekly |
| **Retention** | __________ backups | Target: 4 backups (1 month) |
| **Last Backup** | __________ (date) | Should be recent |
| **Backup Size** | __________ MB | Typical: 100-500 MB |

**Location:** HA internal storage (`/backup`)

**Check via:** HA → **Settings → System → Backups**

### Offsite Backups (Manual)

| Frequency | Destination | Last Backup | Status |
|-----------|-------------|-------------|--------|
| **Monthly** | Mac mini `~/SmartHome-Backups/` | __________ | [Current/Overdue] |
| **Before Major Changes** | Mac mini `~/SmartHome-Backups/` | __________ | [Current/Overdue] |

**Process:**
1. HA → **Settings → System → Backups → [Backup] → Download**
2. Save to Mac mini: `~/SmartHome-Backups/YYYY-MM-DD-description.tar`
3. Sync to PC/cloud: AirDrop, network share, or cloud storage

**Retention:**
- Keep last 12 monthly backups (1 year)
- Delete older backups to free space

### UTM VM Snapshots

| Snapshot Name | Date Created | Purpose | Status |
|---------------|--------------|---------|--------|
| Pre-Phase2-VM-Snapshot | 2026-01-22 | Phase 2 rollback | Current |
| __________ | __________ | __________ | __________ |

**Snapshot Schedule:**
- Before Phase 2, 3, 4, etc. (major changes)
- Before HA version upgrades
- NOT automatic (manual only)

**Retention:**
- Keep last 3 snapshots
- Delete older snapshots (UTM snapshots stored on Mac mini SSD)

---

## Add-ons & Services

### Installed Add-ons

| Add-on | Version | Purpose | Status |
|--------|---------|---------|--------|
| __________ | __________ | __________ | [Running/Stopped] |

**Check via:** HA → **Settings → Add-ons**

**Recommended Add-ons (Optional):**
- **File Editor** or **VS Code Server**: Configuration editing
- **AdGuard Home**: DNS-based ad blocking (if using local DNS)
- **MQTT Broker**: If using MQTT devices (check integrations)
- **Matter Server**: If using Matter devices (Bedroom TV Light Strip is Matter)

---

## Optimization History

### Phase 2 Changes (2026-01-22)

**Before Optimization:**
- CPU: __________ cores
- RAM: __________ GB
- Disk Used: __________ GB
- HomeKit Response: __________ sec

**After Optimization:**
- CPU: __________ cores (changed: [Yes/No])
- RAM: __________ GB (changed: [Yes/No])
- Disk Used: __________ GB (changed: [Yes/No])
- HomeKit Response: __________ sec

**Changes Made:**
- [ ] Increased CPU from __________ to __________ cores
- [ ] Increased RAM from __________ to __________ GB
- [ ] Reduced recorder retention from __________ to __________ days
- [ ] Cleaned __________ old backups (freed __________ GB)
- [ ] Other: __________

**Result:**
- Performance: [Improved/Same/Degraded]
- Stability: [Improved/Same/Degraded]
- Response time change: __________ sec faster/slower

---

## Known Issues & Limitations

### Current Issues
- [ ] __________ (describe issue, impact, workaround)

### Phase 1 Architecture Limitations
- **Double NAT:** Intentional design (Router #1 → Router #2)
- **mDNS scope:** Works within Router #1 subnet only (acceptable, all HomeKit devices on Router #1)
- **IoT access:** Via Mac mini dual-homing (not routing)
- **Camera internet:** Blocked via Access Control (intentional privacy design)

---

## Next Review Date

**Last Review:** 2026-01-22 (Phase 2 baseline)
**Next Review:** 2026-04-22 (3 months)

**Review Checklist:**
- [ ] Update VM resource allocation if changed
- [ ] Record new performance benchmarks
- [ ] Check for new HA version and upgrade if stable
- [ ] Verify backup strategy still working (automatic + offsite)
- [ ] Review integration health (any new failures?)
- [ ] Check disk usage (if >80%, optimize recorder retention)
- [ ] Update known issues list

---

**Last Updated:** 2026-01-22
**Status:** 🔄 Template ready (fill in actual values during Phase 2 execution)
