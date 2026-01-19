# Phase 1 Pre-Flight Checklist

**Date:** 2026-01-18
**Purpose:** Tasks YOU need to complete before starting Phase 1 (Network Segmentation)

---

## ✅ Already Complete

- [x] Router model documented (Verizon CR1000A)
- [x] Existing SSIDs documented (EnterAtYourOwnRisk, HotNeighborsCanConnect, ByteMe)
- [x] Critical device IPs documented (HA, Scrypted, cameras, hubs)
- [x] Device inventory complete (38 devices)
- [x] HomeKit exposure strategy finalized (22 devices)

---

## ⏳ Pending User Actions (Before Phase 1)

### 1. Verify Current Network Access
**What to check:**
- [ ] Can you access Verizon CR1000A admin interface?
  - URL: Usually `192.168.1.1` or via My Fios app
  - Have admin credentials ready
- [ ] Can you access Home Assistant?
  - URL: `http://192.168.1.242:8123`
  - Have login credentials ready

**Why:** Need admin access to make network changes.

---

### 2. Backup Current Router Configuration
**Action:**
- [ ] Log into Verizon CR1000A
- [ ] Export/save current router configuration
  - Look for "Backup Settings" or "Export Configuration" option
  - Save file to safe location

**Why:** Rollback capability if network changes go wrong.

---

### 3. Document Current HA Integrations
**Action:**
- [ ] Open Home Assistant → Settings → Devices & Services
- [ ] Take screenshots of:
  - [ ] All integrations currently installed
  - [ ] HomeKit Bridge configuration (if exists)
  - [ ] Any entities currently exposed to HomeKit
- [ ] Save screenshots to `02-HomeAssistant/screenshots/`

**Why:** Phase 1 may affect mDNS/Bonjour discovery, need to know what's currently working.

---

### 4. Verify Scrypted Access
**Action:**
- [ ] Access Scrypted web interface
  - Try all known IPs: `192.168.1.224`, `192.168.1.209`, `192.168.1.171`
  - Document which IP actually works
- [ ] Verify all 4 cameras are online in Scrypted

**Why:** Need to confirm which Scrypted IP is active before network changes.

---

### 5. Control Room Location Clarification
**Question to answer:**
- [ ] Is "Control Room" a dedicated equipment room/closet?
- [ ] Or is it the same physical location as Office/Bedroom?
- [ ] Where is the Mac mini M4 (HA host) physically located?
- [ ] Where is the Scrypted host physically located?

**Why:** Helps with network planning (Ethernet runs, WiFi coverage zones).

---

### 6. Understand Current Network Behavior
**Test to run:**
- [ ] From your iPhone/iPad, which SSID are you currently connected to?
  - Primary: EnterAtYourOwnRisk
  - Guest: HotNeighborsCanConnect
  - IoT: ByteMe
- [ ] Test HomeKit response time right now (turn on a light, check camera)
- [ ] Note current performance as "baseline"

**Why:** After Phase 1, you'll compare to ensure no degradation.

---

### 7. Identify Critical "Must Work" Devices
**List your top 3-5 devices that MUST work immediately after Phase 1:**

Example:
1. [ ] Home Assistant (must stay accessible)
2. [ ] Kitchen Camera (security priority)
3. [ ] Bedroom Vanity Light (daily use)
4. [ ] [Your choice]
5. [ ] [Your choice]

**Why:** If Phase 1 goes wrong, these are priority devices to fix first during rollback.

---

### 8. Schedule Maintenance Window
**Plan your timing:**
- [ ] Phase 1 network changes will cause brief disconnections
- [ ] Estimate 1-2 hours for network reconfiguration
- [ ] Choose a time when:
  - [ ] No critical automations running (e.g., not during sleep hours)
  - [ ] You can troubleshoot if needed
  - [ ] You have physical access to router and Mac mini

**Recommended:** Weekend afternoon when you're home and awake.

---

## 📋 Optional (Nice to Have)

### 9. Document Current Automations
- [ ] List any HA automations that depend on network connectivity
- [ ] Note any time-sensitive automations (e.g., morning alarms, security alerts)

**Why:** Know what might break during network changes.

---

### 10. Prepare Rollback Tools
- [ ] Have router admin credentials written down (not just in password manager)
- [ ] Have your phone ready with cellular data (in case WiFi breaks)
- [ ] Know how to factory reset CR1000A if needed (check manual)

**Why:** If Phase 1 fails badly, you need quick recovery.

---

## ✅ When All Checked...

Once you've completed the items above (especially #1-6), you're ready to start Phase 1!

**Reply with:** "Pre-flight complete" or let me know which items need help.

---

## Quick Reference: Current Network

| Device | IP Address | SSID | Notes |
|--------|-----------|------|-------|
| Home Assistant VM | 192.168.1.242 | - | Mac mini M4 host |
| Scrypted | 192.168.1.224 (?) | - | Need to verify active IP |
| Kitchen Camera | 192.168.1.210 | ByteMe (IoT) | Tapo C110 |
| Hallway Camera | 192.168.1.236 | ByteMe (IoT) | Tapo C110 |
| Entrance Camera | 192.168.1.155 | ByteMe (IoT) | Tapo C110 |
| Office Camera | 192.168.1.204 | ByteMe (IoT) | Tapo C110 |
| Hue Bridge | 192.168.1.169 | - | - |
| Aqara Hub M2 | 192.168.1.165 | - | - |

---

**Last Updated:** 2026-01-18
