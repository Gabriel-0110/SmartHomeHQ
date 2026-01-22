# Phase 1: Network Segmentation — COMPLETION SUMMARY

**Status:** ✅ COMPLETED
**Completion Date:** 2026-01-22
**Duration:** Phase 1A + Phase 1B + Stabilization
**Final Architecture:** Dual-Router Design with Intentional Double NAT

---

## Executive Summary

Phase 1 successfully implemented network segmentation for the SmartHomeHQ smart home infrastructure. The final architecture establishes a dual-router topology with two isolated subnets (Trusted and IoT), HomeKit-first design principles, and local-control enforcement through network segmentation and endpoint hardening.

**Key Achievement:** Zero-trust network separation between user devices and untrusted IoT/camera endpoints, while maintaining full HomeKit functionality and HKSV camera integration.

---

## Final Network Architecture

### Topology Overview

```
ISP Modem
    ↓
Router #1 (Verizon CR1000A) — TRUSTED / CONTROL PLANE
  Subnet: 192.168.1.0/24
  Role: Primary LAN, HomeKit hubs, HA/Scrypted host, user devices
    ↓ (LAN Port 3 → Router #2 WAN)
Router #2 (Verizon CR1000A) — IOT / EDGE NETWORK
  Subnet: 192.168.2.0/24
  Role: IoT devices and cameras (intentional double NAT)
```

### Design Rationale

**Why Dual-Router Instead of VLANs?**
- Verizon CR1000A routers **do not support true VLANs** with inter-VLAN firewall rules
- SSIDs on CR1000A **cannot enforce isolation** (no per-SSID firewall policies)
- Dual-router topology provides **physical subnet separation** without requiring new hardware
- Intentional double NAT reduces unsolicited inbound traffic from IoT subnet to Trusted subnet

**Why Dual-Homed Mac Mini?**
- HA and Scrypted must reach IoT devices (Hue Bridge, Aqara Hub, cameras) without routing between subnets
- Mac mini has **two physical Ethernet interfaces**:
  - Interface 1: 192.168.1.20 (Trusted subnet) → HA/Scrypted accessible to HomeKit hubs
  - Interface 2: 192.168.2.10 (IoT subnet) → HA/Scrypted can query IoT devices directly
- This design avoids router-level routing rules (which CR1000A cannot enforce granularly)

---

## Critical Lessons Learned

### 1. Verizon CR1000A Router Behavior Under Load

**Discovery:** Router #1 exhibited service instability when UPnP or port forwarding was enabled:
- Random service shutdowns
- HomeKit bridge disconnections
- mDNS discovery failures

**Resolution:**
- **Disabled UPnP on both routers** (no automatic port mapping)
- **Removed ALL port forwarding rules** (zero inbound exposure)
- **Treated Router #1 strictly as ISP gateway + primary HomeKit LAN**
- Result: Stability restored, no service interruptions

**Takeaway:** Verizon CR1000A is reliable as a basic router/firewall but cannot handle advanced features under load. Simplicity wins.

---

### 2. Scrypted Multi-Instance Conflict

**Discovery:** Multiple Scrypted runners caused port conflicts and HomeKit bridge instability:
- Docker container running Scrypted
- Scrypted CLI service running as daemon
- Scrypted Desktop App running simultaneously
- All three attempted to bind HomeKit bridge ports → collision

**Resolution:**
- **Removed Docker container** (stopped and deleted)
- **Removed CLI service** (disabled systemd/launchd service)
- **Scrypted Desktop App is now the ONLY runner**
- Configured Desktop App to:
  - Use **Ciao mDNS responder** (not Bonjour/Avahi)
  - Use **random HomeKit bridge ports** (avoid hardcoded ports)

**Takeaway:** One Scrypted instance per host. Desktop App is preferred for macOS due to better mDNS integration.

---

### 3. HomeKit mDNS Sensitivity

**Discovery:** HomeKit hubs (Apple TV, iPad) require **same-subnet mDNS (_hap._tcp) discovery** for bridge visibility:
- HA HomeKit Bridge on 192.168.1.21 → visible to hubs on 192.168.1.x ✅
- Scrypted on 192.168.1.20 → visible to hubs on 192.168.1.x ✅
- Any bridge behind Router #2 (192.168.2.x) → **invisible to hubs** ❌ (mDNS does not traverse NAT)

**Resolution:**
- **All HomeKit bridges MUST live on Router #1 subnet** (Trusted 192.168.1.x)
- HA and Scrypted pull data from IoT devices via Mac mini's IoT interface (192.168.2.10)
- IoT devices themselves are NOT exposed to HomeKit directly (HA/Scrypted act as proxies)

**Additional Fix:**
- macOS Firewall **must allow mDNS** (UDP 5353)
- Scrypted must use **Ciao mDNS** (not Bonjour) for reliable _hap._tcp advertising
- HomeKit bridge ports must be **dynamic/random** (hardcoded ports caused binding failures)

**Takeaway:** mDNS is non-negotiable for HomeKit. Do not place bridges across NAT boundaries.

---

### 4. Camera Isolation and HKSV Integration

**Discovery:** Tapo C110 cameras default to cloud-first operation (privacy concern).

**Resolution:**
- **Moved all cameras to Router #2 subnet** (192.168.2.150-153)
- **Blocked camera internet access via Access Control** (WAN paused 24/7)
- **Scrypted pulls RTSP streams locally** via Mac mini's IoT interface (192.168.2.10)
- **HKSV (HomeKit Secure Video) enabled** for all 4 cameras:
  - Live view: Instant
  - Motion detection: Working
  - 10-day recording: Active (via iCloud+)

**Validation:**
- Cameras show "offline" in Tapo cloud app ✅ (internet blocked)
- Cameras stream perfectly in Home app ✅ (local RTSP via Scrypted)
- HKSV recordings visible in Home app Activity tab ✅

**Takeaway:** Local RTSP + internet blocking = privacy-first camera operation with full HKSV functionality.

---

### 5. Home Assistant UTM VM Disk Exhaustion

**Discovery:** HA VM (UTM) ran out of disk space during updates:
- Initial disk: 32GB (too small for HA OS + backups + updates)
- VM became unresponsive during OS upgrade

**Resolution:**
- **Resized UTM disk to 64GB** (provides headroom for future updates)
- **Fresh HAOS install** (clean slate, no legacy cruft)
- **Restored backup** after disk resize

**Takeaway:** Allocate 64GB minimum for HA VM. 32GB is insufficient for production use.

---

## Key Decisions and Justifications

### Decision 1: Intentional Double NAT

**Choice:** Router #2 WAN port connects to Router #1 LAN (cascade topology).

**Justification:**
- CR1000A does not support VLANs or inter-VLAN firewalls
- Double NAT reduces unsolicited traffic from IoT subnet to Trusted subnet
- **NAT is NOT a security boundary** (endpoint firewalls provide enforcement)
- Dual-homing (Mac mini) eliminates routing complexity

**Trade-offs Accepted:**
- mDNS does not traverse NAT → HomeKit bridges must stay on Router #1
- Port forwarding complexity (must forward through two NAT layers) → **not used, acceptable**
- Slightly increased latency for IoT → Mac mini queries → negligible in practice

---

### Decision 2: Scrypted Desktop App as Single Source of Truth

**Choice:** Run Scrypted Desktop App only (no Docker, no CLI service).

**Justification:**
- Desktop App integrates best with macOS mDNS/Bonjour
- Single instance eliminates port conflicts
- Ciao mDNS responder provides reliable _hap._tcp advertising
- Random ports avoid hardcoded binding failures

**Alternative Rejected:** Docker (port binding issues, worse mDNS support)

---

### Decision 3: Zero Port Forwarding Policy

**Choice:** No inbound port forwarding on either router.

**Justification:**
- Router #1 exhibited instability with UPnP/port forwarding enabled
- No use case requiring external access (HomeKit works via iCloud relay)
- HA accessible via Nabu Casa Cloud (if needed) or Tailscale VPN (not port forwarding)
- Reduces attack surface

---

### Decision 4: Camera Internet Blocking via Access Control

**Choice:** Use Router #2 Parental Controls / Access Control to block camera WAN access.

**Justification:**
- Cameras on isolated subnet (192.168.2.x)
- Access Control blocks internet (WAN) but allows LAN communication
- Scrypted reaches cameras locally via Mac mini's IoT interface
- Result: Privacy-first operation, no cloud leakage

**Alternative Rejected:** Firewall rules (CR1000A does not support granular per-device firewall policies)

---

## Network Phase Exit Criteria — ✅ MET

### Infrastructure

- [x] **Router #1 (Trusted subnet)**: 192.168.1.0/24, gateway 192.168.1.1
- [x] **Router #2 (IoT subnet)**: 192.168.2.0/24, gateway 192.168.2.1
- [x] **Mac mini dual-homed**: 192.168.1.20 + 192.168.2.10
- [x] **UPnP disabled** on both routers
- [x] **Port forwarding removed** on both routers
- [x] **Band Steering disabled** on both routers (prevents random band switching)
- [x] **WPS disabled** on both routers (security risk)

### HomeKit Stability

- [x] **HA HomeKit Bridge reachable** from Apple TV/iPad hubs (same subnet)
- [x] **Scrypted HomeKit Bridge reachable** from Apple TV/iPad hubs (same subnet)
- [x] **mDNS discovery working** (_hap._tcp visible on 192.168.1.x)
- [x] **HomeKit response time ≤2 seconds** (lights, switches)
- [x] **No bridge disconnections** (72-hour stability test passed)

### Camera Integration

- [x] **All 4 cameras on Router #2 subnet** (192.168.2.150-153)
- [x] **Camera internet access blocked** (WAN paused via Access Control)
- [x] **Scrypted pulling RTSP streams locally** via 192.168.2.10
- [x] **HKSV enabled and recording** (10-day history visible)
- [x] **Motion detection working** (notifications in Home app)
- [x] **Live view latency <2 seconds**

### Home Assistant

- [x] **HA VM accessible at 192.168.1.21:8123**
- [x] **HA disk resized to 64GB** (prevents future exhaustion)
- [x] **HA MFA enabled** (admin account)
- [x] **Least-privilege user created** (non-admin daily account)
- [x] **IoT integrations online** (Hue Bridge, Aqara Hub, cameras)
- [x] **HA can reach IoT devices** via Mac mini's IoT interface (192.168.2.10)

### Security Hardening

- [x] **macOS Firewall enabled** (Mac mini) with stealth mode
- [x] **mDNS allowed in macOS Firewall** (UDP 5353)
- [x] **Scrypted using Ciao mDNS** (not Bonjour/Avahi)
- [x] **No Docker or CLI Scrypted services running** (Desktop App only)
- [x] **Camera cloud access blocked** (verified via Tapo app: cameras offline)

### Operational Validation

- [x] **72-hour uptime test passed** (no service interruptions)
- [x] **HomeKit commands <2 seconds response time**
- [x] **Camera streams <2 seconds load time**
- [x] **HKSV recordings verified** (motion events captured)
- [x] **HA automations functional** (tested bedroom vanity light automation)
- [x] **IP addresses stable** (DHCP reservations working)

---

## What Changed From Initial Plan

### Phase 1A (Network Foundation)

**Initial Plan:**
- Single-router VLAN segmentation with mDNS reflector

**Actual Implementation:**
- Dual-router topology (VLANs not supported by CR1000A)
- Intentional double NAT (acceptable trade-off)
- Mac mini dual-homed (avoids routing complexity)

### Phase 1B (Endpoint Hardening)

**Initial Plan:**
- Standard endpoint firewall enablement

**Actual Implementation:**
- Endpoint firewall enablement ✅
- **Bonus:** Scrypted runner consolidation (Desktop App only)
- **Bonus:** Camera internet blocking via Access Control
- **Bonus:** HA VM disk resize (64GB)

### Post-Phase 1 Stabilization (Not Originally Scoped)

**Additional Work Required:**
- Resolved Scrypted multi-instance conflicts (Docker vs CLI vs Desktop App)
- Resolved HomeKit mDNS discovery issues (Ciao + random ports)
- Validated HKSV recording functionality
- Verified Router #1 stability (UPnP/port forwarding removal)

---

## Phase 2 Upgrade Path (Optional Future Work)

Phase 1 is **production-ready and stable**. However, if more granular control is desired in the future, consider:

### Recommended Hardware Upgrades

Replace Verizon CR1000A routers with:
- **Firewalla Gold / Gold Plus** (~$400-600) — Easiest, GUI-driven
- **UniFi Dream Machine / UDM Pro** (~$200-500) — Enterprise-grade, requires learning curve
- **OPNsense / pfSense on dedicated hardware** (~$200-400) — Maximum control, steepest learning curve

### Phase 2 Would Enable

- **True VLAN segmentation** (not just double NAT)
- **Layer 3 firewall rules** (granular inter-VLAN policies)
- **mDNS reflection** (HomeKit bridges can live on any VLAN)
- **Network-wide DNS enforcement** (per-VLAN DNS servers)
- **Advanced logging and visibility** (traffic analysis, threat detection)

**Current Recommendation:** **Do not upgrade unless a specific limitation is encountered.** Phase 1 architecture is sufficient for the current use case.

---

## Validation Checklist (Performed 2026-01-22)

### Network Connectivity

- [x] Router #1 reachable at 192.168.1.1
- [x] Router #2 reachable at 192.168.2.1 (from Mac mini or Router #2 SSID devices)
- [x] Mac mini dual-homed: `ifconfig` shows 192.168.1.20 + 192.168.2.10
- [x] HA reachable at http://192.168.1.21:8123
- [x] Scrypted reachable at https://192.168.1.20:10443

### HomeKit Discovery

- [x] `dns-sd -B _hap._tcp` on Router #1 subnet shows HA Bridge + Scrypted Bridge
- [x] Home app shows HA Bridge as "Connected"
- [x] Home app shows Scrypted cameras as "Connected"
- [x] HomeKit commands respond in <2 seconds

### Camera Validation

- [x] Cameras show as "offline" in Tapo cloud app (internet blocked) ✅
- [x] Cameras stream in Home app (local RTSP via Scrypted) ✅
- [x] HKSV recordings visible in Home app Activity tab ✅
- [x] Motion detection triggers notifications ✅

### Security Posture

- [x] UPnP disabled (both routers)
- [x] Port forwarding empty (both routers)
- [x] macOS Firewall enabled (Mac mini)
- [x] HA MFA enabled (admin account)
- [x] Camera internet blocked (Access Control paused WAN)

---

## Known Limitations (Acceptable)

1. **mDNS does not traverse NAT**
   - Impact: HomeKit bridges must stay on Router #1 subnet
   - Workaround: HA/Scrypted proxy IoT devices via dual-homed Mac mini
   - Status: Working as designed ✅

2. **CR1000A does not support VLANs**
   - Impact: No true Layer 3 segmentation with inter-VLAN firewall rules
   - Workaround: Dual-router topology with endpoint firewalls
   - Status: Acceptable for current use case ✅

3. **Intentional double NAT**
   - Impact: Port forwarding complexity (not used, not needed)
   - Workaround: Avoid port forwarding; use iCloud relay or Tailscale VPN
   - Status: Acceptable ✅

4. **No centralized firewall logging**
   - Impact: Limited visibility into inter-subnet traffic
   - Workaround: Endpoint monitoring (HA device availability, macOS logs)
   - Status: Acceptable for residential use case ✅

---

## Post-Completion Notes

### What Works Well

- **HomeKit response time:** Instant to 1-2 seconds (excellent)
- **Camera latency:** <2 seconds live view load time (excellent)
- **HKSV reliability:** 100% motion event capture over 72-hour test (excellent)
- **Network stability:** Zero service interruptions since UPnP/port forwarding removal (excellent)
- **Dual-homing:** Mac mini reaches IoT devices without routing complexity (excellent)

### What to Monitor

- **Router #1 uptime:** Watch for service instability if UPnP is accidentally re-enabled
- **Scrypted updates:** Ensure Desktop App is updated regularly (auto-update enabled)
- **HA disk usage:** 64GB provides headroom, but monitor quarterly (`df -h` in HA Terminal)
- **Camera RTSP streams:** Verify cameras remain reachable after firmware updates

### Maintenance Cadence

- **Monthly:** Verify HA backup success (Settings → Backups)
- **Quarterly:** Check HA disk usage and clean old backups
- **Quarterly:** Verify camera firmware updates (via Tapo app, then re-test RTSP)
- **Biannually:** Review router security settings (UPnP, port forwarding, WPS)

---

## Files Updated During Phase 1

### Created
- `01-Network/PHASE1-COMPLETION-SUMMARY.md` (this file)
- `01-Network/PHASE1-START-HERE.md`
- `01-Network/phase1a-implementation-guide.md`
- `01-Network/phase1b-implementation-guide.md`
- `01-Network/phase1-decision-guide.md`
- `01-Network/phase1-preflight-checklist.md`
- `01-Network/router/cr1000a-audit-results.md`
- `01-Network/network-topology-diagram.md`

### Updated
- `01-Network/current-network.md` — Reflects final dual-router topology
- `TASKS.md` — Phase 1 marked as COMPLETED
- `roadmap.md` — Phase 1 status updated
- `CLAUDE.md` — North Star principles validated (no changes needed)

---

## Next Phase: Ready to Proceed

**Phase 1 Network Segmentation:** ✅ **COMPLETED**

The project is now ready to proceed to **Phase 2: HomeKit Bridge Cleanup** or **Phase 4: Areas/Zones/Labels** (per user priority).

Network foundation is stable, secure, and production-ready.

---

**Completion Date:** 2026-01-22
**Validated By:** SmartHomeHQ Repository (operational testing)
**Final Status:** ✅ **NETWORK PHASE COMPLETE**
