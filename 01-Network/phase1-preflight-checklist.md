# Phase 1 Pre-Flight Checklist

**Date:** 2026-01-19
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
  - URL: `http://192.168.1.1` or via My Fios app
  - Have admin credentials ready
- [ ] Can you access Home Assistant?
  - URL: `http://192.168.1.242:8123`
  - Have login credentials ready

**Why:** Need admin access to make network changes.

---

### 2. Backup Current Router Configuration
**Action:**
- [ ] Log into Verizon CR1000A
- [ ] Look for "Backup Settings" or "Export Configuration" option
  - If available: Export and save file to safe location
  - If NOT available: Use screenshots as backup (see step 3)

**Why:** Rollback capability if network changes go wrong.

**Note:** If CR1000A does not provide config export, screenshots are acceptable fallback.

---

### 3. Screenshots to Capture (Verizon UI)
**Router pages to document BEFORE making changes:**

**Network Settings:**
- [ ] **Advanced → Network Settings → DNS Server**
  - Capture current DNS configuration
- [ ] **Advanced → Network Settings → IPv4 Address Distribution**
  - Summary page showing DHCP range
- [ ] **Advanced → Network Settings → IPv4 Address Distribution → DHCP Settings**
  - DHCP lease time, any existing DNS fields (if present)
- [ ] **Advanced → Network Settings → IPv4 Address Distribution → DHCP Connections**
  - Full list of connected devices with IPs and MAC addresses
- [ ] **Advanced → Network Settings → Network Connections**
  - Show "Network (Home/Office)" entry
- [ ] **Advanced → Network Settings → Network Connections → Edit Network (Home/Office)** (if available)
  - LAN settings, subnet mask, gateway

**Wi-Fi Settings:**
- [ ] **Basic → Wi-Fi → Primary Network** (EnterAtYourOwnRisk)
  - Current bands enabled, security mode, password
- [ ] **Basic → Wi-Fi → Guest Network** (HotNeighborsCanConnect)
  - Current bands enabled, security mode, password
- [ ] **Basic → Wi-Fi → IoT Network** (ByteMe, if separate from above)
  - Current bands enabled, security mode, password
- [ ] **Basic → Wi-Fi → Radio Management** (or wherever Band Steering/SON toggle is)
  - Current state of Band Steering/SON/Smart Connect

**Wi-Fi Security:**
- [ ] **Basic → Wi-Fi → Wi-Fi Protected Setup (WPS)**
  - Current WPS status (enabled/disabled)

**Firewall/Security:**
- [ ] **Advanced → Security & Firewall → Port Forwarding**
  - List any existing port forward rules
- [ ] **Advanced → Security & Firewall → Port Forwarding Rules** (if separate page)
- [ ] **Advanced → Security & Firewall → DMZ Host**
  - Verify DMZ is off (or document if on)
- [ ] **Advanced → Security & Firewall → UPnP** (or check Advanced settings)
  - Current UPnP status
- [ ] **Basic → Devices → Parental Controls** (or **Access Control**)
  - Current profiles and rules (should be empty before Phase 1)

**IPv6:**
- [ ] **Advanced → Network Settings → IPv6**
  - Current IPv6 status (enabled/disabled)

**Save location:** `01-Network/screenshots/preflight/`

---

### 4. Document Home Assistant HomeKit Entities
**Action:**
- [ ] Open Home Assistant: `http://192.168.1.242:8123`
- [ ] Go to **Settings → Devices & Services**
- [ ] Find **HomeKit Bridge** integration
- [ ] Click **Entities** tab (or **Configure** button)
- [ ] Take screenshot showing:
  - Which entities are currently exposed to HomeKit
  - Total count of exposed entities
- [ ] Also capture Home app on iPhone/iPad:
  - Open Home app → Home Settings → Hubs & Bridges
  - Take screenshot of current bridges

**Why:** Phase 1 may affect mDNS/Bonjour discovery; need to know current working state.

**Save location:** `02-HomeAssistant/screenshots/preflight/`

---

### 5. Verify Scrypted Access and Host IP
**Action:**
- [ ] Open Verizon CR1000A: `http://192.168.1.1`
- [ ] Go to **Advanced → Network Settings → IPv4 Address Distribution → DHCP Connections**
- [ ] Find the Mac mini entry (likely "Gabriels-Mini" or similar)
- [ ] Note the IP address assigned to Mac mini
- [ ] Try accessing Scrypted at:
  - `https://<mac-mini-ip>:10443`
  - `http://<mac-mini-ip>:11080` (if port 10443 doesn't work)
- [ ] Verify all 4 cameras are online in Scrypted
- [ ] Take screenshot of Scrypted dashboard showing cameras

**Document:**
- Mac mini hostname: _________________
- Mac mini IP: _________________
- Scrypted URL that works: _________________

**Why:** Need to confirm Scrypted host IP before network changes. Mac mini will be dual-homed so Scrypted can reach cameras on the IoT subnet.

---

### 6. Understand Current Network Behavior (Baseline)
**Test to run:**
- [ ] From your iPhone/iPad, which SSID are you currently connected to?
  - Primary: EnterAtYourOwnRisk
  - Guest: HotNeighborsCanConnect
  - IoT: ByteMe
  - Document: _________________
- [ ] Test HomeKit response time (current baseline):
  - Toggle Bedroom Vanity Light in Home app
  - Note response time: _________________ (instant / 1-2 sec / slow)
- [ ] Test camera stream:
  - Open Kitchen Camera in Home app
  - Note load time: _________________ (instant / 1-3 sec / 5+ sec)
- [ ] Test Home Assistant dashboard:
  - Open HA dashboard in browser
  - Verify all devices show "available" status

**Why:** After Phase 1, you'll compare to ensure no degradation.

---

### 7. Identify Critical "Must Work" Devices
**List your top 5 devices that MUST work immediately after Phase 1:**

1. [ ] Home Assistant (must stay accessible)
2. [ ] _________________
3. [ ] _________________
4. [ ] _________________
5. [ ] _________________

**Suggested priorities:**
- Kitchen Camera (security)
- Bedroom Vanity Light (daily use)
- Entrance Contact Sensor (security)
- Living Room Aqara Switch (daily use)

**Why:** If Phase 1 goes wrong, these are priority devices to fix first during rollback.

---

### 8. Schedule Maintenance Window
**Plan your timing:**
- [ ] Phase 1 network changes will cause brief disconnections (5-10 minutes per SSID rename)
- [ ] Estimate 1.5-2 hours for full Phase 1A implementation
- [ ] Choose a time when:
  - [ ] No critical automations running (e.g., not during sleep hours)
  - [ ] You can troubleshoot if needed
  - [ ] You have physical access to router and Mac mini
  - [ ] You have cellular data available (in case Wi-Fi breaks)

**Recommended:** Weekend afternoon when you're home and awake.

**Your chosen window:** _________________

---

## 📋 Optional (Nice to Have)

### 9. Document Current Automations
- [ ] List any HA automations that depend on network connectivity
- [ ] Note any time-sensitive automations (e.g., morning alarms, security alerts)

**Examples:**
- "Turn on Entrance Light at sunset" (uses Hue Bridge)
- "Send notification if Entrance Door opens" (uses Aqara contact sensor)

**Why:** Know what might break during network changes.

---

### 10. Prepare Rollback Tools
- [ ] Have router admin credentials written down (not just in password manager)
- [ ] Have your phone ready with cellular data (in case Wi-Fi breaks)
- [ ] Know how to factory reset CR1000A if needed:
  - **Method:** Hold reset button on back of router for 10+ seconds
  - **Warning:** This will erase ALL settings (use only as last resort)
- [ ] Have SSID passwords written down for quick reconnection

**Why:** If Phase 1 fails badly, you need quick recovery.

---

## ✅ When All Checked

Once you've completed items 1-8 above (especially screenshots and access verification), you're ready to start Phase 1A!

**Reply with:** "Pre-flight complete" or let me know which items need help.

---

## Quick Reference: Current Network (Pre-Phase 1)

| Device | Current IP | Current SSID | Notes |
|--------|-----------|--------------|-------|
| Home Assistant VM | 192.168.1.242 | - | Mac mini M4 host (UTM) |
| Mac mini (Scrypted host) | TBD from DHCP | - | Find as "Gabriels-Mini" |
| Kitchen Camera | 192.168.1.210 | ByteMe (IoT) | Tapo C110 |
| Hallway Camera | 192.168.1.236 | ByteMe (IoT) | Tapo C110 |
| Entrance Camera | 192.168.1.155 | ByteMe (IoT) | Tapo C110 |
| Office Camera | 192.168.1.204 | ByteMe (IoT) | Tapo C110 |
| Hue Bridge | 192.168.1.169 | - | Wired to router |
| Aqara Hub M2 | 192.168.1.165 | - | Wired to router |

---

**Last Updated:** 2026-01-19
