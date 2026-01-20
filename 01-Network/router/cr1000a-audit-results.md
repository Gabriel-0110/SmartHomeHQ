# Verizon CR1000A Router Audit Results

**Date:** 2026-01-19
**Firmware Version:** 3.6.0.2_BD
**Hardware Version:** 0.0.7
**Model:** CR1000A

---

## 📊 Audit Summary

The Verizon CR1000A is a **consumer-grade tri-band router** with basic security features. It **CANNOT** implement advanced firewall rules needed for full network segmentation. 

**Key Finding:** NAT does NOT provide isolation by itself. The dual-router architecture provides subnet separation, but **endpoint firewalls are required for actual security enforcement**.

---

## ✅ Capabilities (What It CAN Do)

### 1. Multiple SSIDs
- **Count:** 3 SSIDs total
  - Primary Network (EnterAtYourOwnRisk)
  - Guest Network (HotNeighborsCanConnect)
  - IoT Network (ByteMe)
- **Bands:** Each SSID can be 2.4 GHz, 5 GHz, or 6 GHz
- **Security:** WPA2 (WPA3 option available)
- **Status:** ✅ MEETS REQUIREMENTS

### 2. Custom DNS Configuration
- **Per-Device DNS:** Can assign DNS servers to individual devices
- **Network-Wide DNS:** Can set default DNS for all devices
- **Dynamic DNS:** Supports custom hostnames
- **Status:** ✅ PERFECT FOR PI-HOLE

### 3. DHCP Reservations
- **Static IP Assignment:** Via DHCP Option 60 (Vendor Class Identifier)
- **Current Reservations:** Already has some (MSFT, Philips devices)
- **Capacity:** Can add more reservations as needed
- **Status:** ✅ WORKS

### 4. WPS (Wi-Fi Protected Setup)
- **Status:** Enabled (Screenshot 25)
- **Recommendation:** DISABLE (security risk)

### 5. MAC Address Filtering
- **Location:** Basic → Wi-Fi → Primary Network → MAC Authentication
- **Status:** Can limit devices that can connect to router
- **Use Case:** Extra security layer (optional)

### 6. Device Management
- **Visibility:** Can see all 19 connected devices (10 online)
- **Organization:** Devices categorized by network (Primary, Guest, IoT)
- **Status:** ✅ GOOD VISIBILITY

### 7. Port Forwarding
- **IPv4 & IPv6:** Supports both
- **Current Rules:** Port forwards to HA (192.168.1.242)
- **Status:** ✅ WORKS

---

## ❌ Limitations (What It CANNOT Do)

### 1. **VLAN Support** ❌ CRITICAL LIMITATION
- **No VLAN tagging:** Cannot create separate subnets (192.168.2.x, 192.168.3.x)
- **Single subnet only:** All devices on 192.168.1.0/24
- **Impact:** Cannot implement true Layer 3 network segmentation

### 2. **Advanced Firewall Rules** ❌ CRITICAL LIMITATION
- **Access Control:** Only blocks devices from **internet**, not from **each other**
- **No IP-based rules:** Cannot create rules like:
  - ❌ "Block 192.168.1.100-199 → 192.168.1.2-99"
  - ❌ "Allow cameras → Scrypted only"
  - ❌ "Block cameras → internet"
- **Parental Controls:** Content filtering, time limits, schedules (NOT firewall rules)
- **Impact:** IoT devices **CAN** access Trusted devices (security risk)

### 3. **Per-SSID Firewall Rules** ❌ LIMITATION
- **No client isolation per SSID:** Cannot isolate IoT SSID from Primary SSID at router level
- **Guest Network:** Likely has isolation (need to test), but not configurable for other SSIDs
- **Impact:** SSIDs are **NOT** isolated from each other

### 4. **mDNS Reflection** ❌ LIMITATION
- **No Avahi/mDNS repeater:** Cannot forward mDNS/Bonjour across SSIDs
- **Impact:** HomeKit discovery may break if devices are on different SSIDs
- **Workaround:** Use Home Assistant HomeKit Bridge (already planned)

### 5. **Band Steering Control** ⚠️ UNCLEAR
- **Not visible in screenshots:** No explicit "Band Steering" toggle found
- **Possible location:** May be hidden in advanced settings or mobile app
- **Impact:** Devices may still randomly switch between 2.4/5/6 GHz

### 6. **UPnP Status** ⚠️ UNKNOWN
- **Not visible in screenshots:** Need to check if UPnP is enabled
- **Recommendation:** DISABLE (security risk)

---

## 🔐 Security Hardening Checklist

Based on audit, these settings should be changed:

- [ ] **Disable WPS** (currently enabled, security risk)
- [ ] **Check UPnP status** (should be disabled)
- [ ] **Remove unnecessary port forwards** (currently has some to HA)
- [ ] **Change default passwords** (Wi-Fi and admin)
- [ ] **Update firmware** (currently 3.6.0.2_BD, check for updates)

---

## 🎯 What Phase 1 CAN Achieve (Revised Goals)

### ✅ Achievable with CR1000A:

1. **DNS-Level Ad/Tracker Blocking**
   - Setup Pi-hole on Mac mini Docker
   - Configure router DNS to point to Pi-hole
   - Block ads, trackers, telemetry domains

2. **Network Organization (Two Subnets)**
   - Router #1: Trusted (192.168.1.0/24)
   - Router #2: IoT (192.168.2.0/24)
   - Mac mini dual-homed for HA/Scrypted IoT access

3. **Camera Internet Blocking**
   - Use Access Control to block cameras from internet
   - Cameras reachable via Mac mini's IoT interface
   - Privacy improvement (no Tapo cloud)

4. **SSID Organization**
   - Trusted SSID on Router #1
   - IoT/Cameras SSIDs on Router #2
   - SSIDs are organizational, NOT security boundaries

5. **Static IP Management**
   - Reserve IPs for critical devices (HA, Scrypted, hubs, cameras)
   - Organize IP ranges by device type

6. **Endpoint Firewall Enforcement**
   - macOS firewall on Mac mini
   - Windows firewall on trusted PCs
   - This is the PRIMARY isolation control

### ❌ NOT Achievable with CR1000A:

1. **NAT-Based Isolation**
   - NAT does NOT block traffic by itself
   - NAT only reduces unsolicited inbound traffic
   - Do NOT rely on NAT for security

2. **VLAN Segmentation**
   - Cannot create true VLANs
   - Dual-router design is the alternative

3. **Per-SSID Firewall Policies**
   - Cannot apply different firewall rules to different SSIDs
   - SSIDs provide organization, not isolation

---

## 💡 Recommendations

### Short-Term (Phase 1 with CR1000A):
1. Proceed with **Dual-Router Phase 1**:
   - ✅ Pi-hole for DNS blocking (privacy + visibility)
   - ✅ Two subnets: Trusted (Router #1) + IoT (Router #2)
   - ✅ Mac mini dual-homed (interface on each router)
   - ✅ Access Control to block cameras from internet
   - ✅ Static IP reservations on both routers
   - ✅ Endpoint firewalls (macOS/Windows) for actual isolation
   - ⚠️ **Accept:** NAT is NOT a security boundary

2. **Benefits:** Achieves organizational separation + endpoint enforcement
   - DNS-level blocking (ads, trackers, telemetry)
   - Better network organization
   - Camera privacy (no cloud access)
   - Improved visibility
   - Endpoint firewall isolation

3. **Key Understanding:**
   - NAT only reduces unsolicited inbound traffic
   - Endpoint firewalls are the PRIMARY isolation control
   - Architecture enables control, not magic isolation

### Long-Term (Future Phase):
1. **Upgrade to Advanced Router** (6-12 months):
   - Keep CR1000A as modem/ONT
   - Add VLAN-capable router:
     - **Ubiquiti Dream Machine** (~$299) [Recommended]
     - **Firewalla Gold** (~$468) [Best security]
     - **pfSense box** (~$200-400) [Most flexible]
   - Full VLAN + firewall capabilities

2. **Alternative:** Ask Verizon if CR1000A has "Advanced Mode" firmware with more features

---

## 📝 Current Network State

From screenshots (Devices page):

**Online Devices (10):**
- Aqara-Hub-M2-7437 (Ethernet, CR1000A) → 192.168.1.165
- Bedroom (Ethernet, CR1000A)
- Google-Home-Mini (2.4 GHz IoT, CR1000A)
- HPXXXX (2.4 GHz IoT, CR1000A)
- HS103 x2 (2.4 GHz IoT, CR1000A) → Kasa smart plugs
- iPad (5 GHz, CR1000A) → 192.168.1.212
- iPhone Gabriel (6 GHz, CR1000A)
- Mac (6 GHz, CR1000A) → Mac mini M4
- Samsung (5 GHz, CR1000A) → Samsung TV
- toshiba_b0_0457 (2.4 GHz IoT, CR1000A) → Microwave

**Offline Devices:**
- Apple-TV (offline)

**Total Devices:** 19 (Primary: 13, Guest: 0, IoT: 6)

---

## 🔍 Unanswered Questions

Need to verify during implementation:

1. **Band Steering:** Is it enabled? Can it be disabled?
2. **Guest Network Isolation:** Does Guest SSID isolate clients?
3. **UPnP Status:** Is it enabled? (should disable)
4. **Scrypted Active IP:** Is it 192.168.1.224 or 192.168.1.171?
5. **Firmware Updates:** Is 3.6.0.2_BD the latest?

---

**Last Updated:** 2026-01-19
**Next Steps:** Create revised Phase 1 implementation plan based on these findings.
