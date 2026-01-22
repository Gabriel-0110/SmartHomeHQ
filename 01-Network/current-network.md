# Current Network State (Canonical)

**Last Updated:** 2026-01-22
**Status:** ✅ **OPERATIONAL** (Phase 1 COMPLETED)
**Architecture:** DUAL-ROUTER DESIGN (Two routers, two subnets, Mac mini dual-homed for HA/Scrypted IoT access)

**Phase 1 Completion:** This document reflects the **final, validated network architecture**. Phase 1 (Network Segmentation) is complete and stable. See [PHASE1-COMPLETION-SUMMARY.md](PHASE1-COMPLETION-SUMMARY.md) for full completion report.

---

## 🎯 Network Topology (Intentional Double NAT Design)

### Physical Architecture

```
ISP Modem (WAN)
    |
    v
Router #1 (Verizon CR1000A) — TRUSTED / CONTROL PLANE
  IP: 192.168.1.1 (gateway for 192.168.1.0/24)
  Role: NAT + DHCP + Firewall + Wi-Fi (Trusted devices)
  |
  +-- WAN Port: Connected to ISP Modem
  +-- LAN Port 1: Apple TV 4K (Ethernet)
  +-- LAN Port 2: Mac mini M4 Interface 1 (Trusted) ← HA + Scrypted
  +-- LAN Port 3: Router #2 WAN port (intentional cascade)
      |
      v
Router #2 (Verizon CR1000A) — IOT / EDGE NETWORK
  IP: 192.168.2.1 (gateway for 192.168.2.0/24)
  Role: NAT + DHCP + Firewall + Wi-Fi (IoT/Camera devices)
  |
  +-- WAN Port: Connected to Router #1 LAN Port 3 (intentional double NAT)
  +-- LAN Port 1: Aqara Hub M2 (Ethernet)
  +-- LAN Port 2: Mac mini M4 Interface 2 (IoT) ← DUAL-HOMED
  +-- Wi-Fi: Philips Hue Bridge (via extender if needed)
```

**KEY DESIGN NOTES:**
- **Router #2 WAN port IS USED** (connected to Router #1 LAN)
- **Router #2 remains in ROUTER MODE** (NOT access point mode)
- **Intentional double NAT** (Router #1 NAT → Router #2 NAT)
- **Two separate subnets**:
  - Router #1 LAN: 192.168.1.0/24 (Trusted)
  - Router #2 LAN: 192.168.2.0/24 (IoT/Cameras)
- **Mac mini is DUAL-HOMED**: One interface on each router (192.168.1.20 + 192.168.2.10)
- **NAT does NOT provide isolation by itself**; it only reduces unsolicited inbound traffic
- **Endpoint firewalls + architecture are the primary isolation controls**
- **HA/Scrypted reach IoT devices via Mac mini's IoT interface** (NOT via routing)

---

## 🌐 Network Configuration

### Router #1 (Trusted / Control Plane)

- **WAN:** ISP modem
- **LAN Subnet:** 192.168.1.0/24
- **LAN Gateway:** 192.168.1.1
- **Subnet Mask:** 255.255.255.0
- **DHCP Server:** Enabled
- **DHCP Pool:** 192.168.1.100–192.168.1.254 (155 addresses for dynamic devices)
- **Static Reservations:** Yes (HA, Mac mini, Apple TVs, iPad)

### Router #2 (IoT / Edge Network)

- **WAN:** Connected to Router #1 LAN port (intentional cascade)
- **WAN IP:** Assigned by Router #1 DHCP (e.g., 192.168.1.10 or similar)
- **LAN Subnet:** 192.168.2.0/24
- **LAN Gateway:** 192.168.2.1
- **Subnet Mask:** 255.255.255.0
- **DHCP Server:** Enabled
- **DHCP Pool:** 192.168.2.200–192.168.2.254 (55 addresses for dynamic IoT devices)
- **Static Reservations:** Yes (cameras, Hue Bridge, Aqara Hub, IoT devices)

### DNS Strategy

- **Router Capability:** CR1000A cannot push custom DNS via LAN DHCP Option 6
- **Current Approach:** Per-device DNS configuration for critical trusted devices
- **Recommended DNS Servers:**
  - Primary: 94.140.14.14 (AdGuard DNS)
  - Fallback: 94.140.15.15 (AdGuard DNS)
  - OR: 1.1.1.1 (Cloudflare) / 8.8.8.8 (Google)
- **Optional:** Run AdGuard Home locally (HA add-on) and point trusted devices to 192.168.1.21 (HA IP)

---

## 📡 SSIDs (Wi-Fi Networks)

### Router #1 SSIDs (Trusted Network)

#### Primary SSID: "EnterAtYourOwnRisk"
- **Purpose:** Trusted devices (phones, PCs, tablets, Mac mini, Apple TVs, iPad)
- **Subnet:** 192.168.1.0/24
- **Bands:** 5 GHz / 6 GHz preferred (2.4 GHz optional if needed)
- **Security:** WPA2/WPA3 (or WPA2)
- **Preferred Bands:** 5/6 GHz for performance

### Router #2 SSIDs (IoT Network)

#### IoT SSID: "ByteMe"
- **Purpose:** IoT devices (plugs, lights, Echo devices, Govee, HP Printer, Microwave, Hue Bridge)
- **Subnet:** 192.168.2.0/24
- **Bands:** 2.4 GHz ONLY
- **Security:** WPA2
- **Note:** SSID is organizational, NOT a security boundary

#### Cameras SSID: "Cameras" (repurposed from "HotNeighborsCanConnect")
- **Purpose:** Tapo C110 cameras
- **Subnet:** 192.168.2.0/24
- **Bands:** 2.4 GHz ONLY
- **Security:** WPA2
- **Internet Access:** BLOCKED via Access Control (WAN blocked)
- **LAN Access:** Allowed (Scrypted reaches cameras via Mac mini's IoT interface)

---

## 🔢 IP Address Allocation (Organized Ranges)

### Router #1 (Trusted Network) - 192.168.1.0/24

| Range | Purpose | Devices |
|-------|---------|---------|
| 192.168.1.1 | Router #1 gateway | - |
| 192.168.1.2-.19 | Network Infrastructure | Router #2 WAN IP (assigned by DHCP), future APs/switches |
| 192.168.1.20-.49 | Servers & Core Services | Mac mini M4 (.20), Home Assistant VM (.21), reserved (.22-.49) |
| 192.168.1.50-.99 | Trusted Static Devices | Apple TV 4K (.50), Apple TV Bedroom (.51), iPad 5th Gen (.52), reserved (.53-.99) |
| 192.168.1.100-.254 | DHCP Dynamic Pool | Phones, tablets, PCs, guest devices (auto-assigned) |

#### Specific Static Reservations (Router #1)

| Device | IP Address | Connection Type | Notes |
|--------|-----------|-----------------|-------|
| Router #1 | 192.168.1.1 | - | Primary gateway (NAT + DHCP + Firewall) |
| Mac mini M4 (Trusted Interface) | 192.168.1.20 | Ethernet (Router #1 LAN) | Interface 1 for HA + Scrypted |
| Home Assistant VM | 192.168.1.21 | Virtual (UTM bridge) | HA hub (URL: `http://192.168.1.21:8123`) |
| Apple TV 4K (Control Room) | 192.168.1.50 | Ethernet (Router #1 LAN) | HomeKit hub + Thread border router |
| Apple TV (Bedroom) | 192.168.1.51 | Wi-Fi (Primary SSID) | HomeKit hub |
| iPad 5th Gen (wall panel) | 192.168.1.52 | Wi-Fi (Primary SSID) | Dashboard display |

**Note:** Mac mini is DUAL-HOMED with a second interface on Router #2 (192.168.2.10). Scrypted runs on Mac mini host, accessible at `https://192.168.1.20:10443` (or port 11080 for HTTP).

### Router #2 (IoT Network) - 192.168.2.0/24

| Range | Purpose | Devices |
|-------|---------|---------|
| 192.168.2.1 | Router #2 gateway | - |
| 192.168.2.2-.49 | Reserved for Infrastructure | Future expansion |
| 192.168.2.50-.149 | IoT Devices | Hue Bridge, Aqara Hub, Echo devices, smart plugs, printer, microwave |
| 192.168.2.150-.199 | Cameras | Kitchen, Hallway, Entrance, Office cameras |
| 192.168.2.200-.254 | DHCP Dynamic Pool | IoT devices (auto-assigned if not static) |

#### Specific Static Reservations (Router #2)

| Device | IP Address | Connection Type | Notes |
|--------|-----------|-----------------|-------|
| Router #2 | 192.168.2.1 | - | IoT gateway (NAT + DHCP + Firewall) |
| Mac mini M4 (IoT Interface) | 192.168.2.10 | Ethernet (Router #2 LAN) | Interface 2 for HA + Scrypted IoT access |
| Philips Hue Bridge | 192.168.2.50 | Wi-Fi or Ethernet (Router #2 LAN / Extender) | Zigbee hub for Hue lights |
| Aqara Hub M2 | 192.168.2.51 | Ethernet (Router #2 LAN) | Zigbee hub for Aqara devices |
| Echo Show 8 (Bedroom) | 192.168.2.52 | Wi-Fi (ByteMe SSID) | Smart display |
| Echo Dot (Office) | 192.168.2.53 | Wi-Fi (ByteMe SSID) | Voice assistant |
| Nest Mini (Bathroom) | 192.168.2.54 | Wi-Fi (ByteMe SSID) | Voice assistant |
| HP Printer | 192.168.2.55 | Wi-Fi (ByteMe SSID) | Printer |
| Toshiba Microwave | 192.168.2.56 | Wi-Fi (ByteMe SSID) | Smart appliance |
| Kitchen Camera (Tapo C110) | 192.168.2.150 | Wi-Fi (Cameras SSID) | Internet BLOCKED via Access Control |
| Hallway Camera (Tapo C110) | 192.168.2.151 | Wi-Fi (Cameras SSID) | Internet BLOCKED via Access Control |
| Entrance Camera (Tapo C110) | 192.168.2.152 | Wi-Fi (Cameras SSID) | Internet BLOCKED via Access Control |
| Office Camera (Tapo C110) | 192.168.2.153 | Wi-Fi (Cameras SSID) | Internet BLOCKED via Access Control |

**All IPs assigned via DHCP reservations (static lease) on Router #2.**

---

## 🔒 Security Posture

### Router Hardening (Both Routers)

| Setting | Status | Notes |
|---------|--------|-------|
| Band Steering/SON | **Disabled** | Prevents random band switching (both routers) |
| WPS (Wi-Fi Protected Setup) | **Disabled** | Security vulnerability (both routers) |
| UPnP (Universal Plug and Play) | **Disabled** | If exposed in UI (both routers) |
| Port Forwarding | **Empty** | No inbound ports open (both routers) |
| DMZ (Demilitarized Zone) | **Disabled** | No exposed device (both routers) |
| Remote Admin | **Disabled** | Admin access from LAN only (both routers) |
| IPv6 | **Intentional** | Enabled if used, disabled if not (both routers) |
| Firmware | **Updated** | Both routers on latest firmware |

### Camera Internet Blocking

- **Method:** Access Control / Parental Controls (Router #2)
- **Profile Name:** "Cameras" (or per-device blocking by MAC address)
- **Action:** Internet/WAN paused 24/7 (blocked from ISP access)
- **LAN Access:** Allowed (Scrypted reaches cameras via Mac mini's IoT interface 192.168.2.10)

### Endpoint Firewalls

- **Mac mini M4:** macOS Firewall enabled, stealth mode on
- **Windows PCs (if any):** Windows Firewall enabled, network profile = Private

### Home Assistant Hardening

- **MFA:** Multi-factor authentication enabled (admin account)
- **Least Privilege:** Non-admin daily user created
- **Backup Strategy:** Weekly backups, downloaded to safe location
- **HomeKit Exposure:** Minimized (only essential devices exposed to HomeKit)

---

## 🚫 Known Constraints & Limitations

### Intentional Double NAT Design

✅ **Benefits**:
1. **Two separate subnets**: Trusted (192.168.1.x) and IoT (192.168.2.x) are organizationally separated
2. **HomeKit stability**: HomeKit hubs (Apple TVs, iPad) and HA Bridge all on same subnet (192.168.1.x) — mDNS works perfectly
3. **Camera privacy**: Cameras behind Router #2 + internet blocking = no cloud access
4. **No new hardware required**: Works with existing Verizon CR1000A routers
5. **Architecture enables control**: Dual-homed Mac mini allows HA/Scrypted to reach both subnets

❌ **Trade-offs**:
1. **Double NAT exists**: This is INTENTIONAL, not a bug (acceptable for this use case)
2. **mDNS does NOT traverse NAT**: HomeKit devices behind Router #2 won't work — by design (keep HomeKit devices on Router #1)
3. **Mac mini MUST be dual-homed**: HA/Scrypted reach IoT via second interface, not via routing
4. **NAT is NOT a security boundary**: It only reduces unsolicited inbound traffic; endpoint firewalls provide actual enforcement
5. **Port forwarding complexity**: If needed, must forward through both routers (not recommended)

### Verizon CR1000A Hardware Limitations

- **❌ No true VLAN support**: Cannot create VLANs with inter-VLAN firewall rules
- **❌ No per-SSID firewall rules**: Cannot block traffic between SSIDs on same router
- **❌ No custom LAN DHCP DNS**: Cannot push custom DNS via DHCP Option 6
- **❌ NAT is NOT a firewall**: NAT only reduces unsolicited inbound traffic
- **✅ Dual-router design works**: Provides subnet separation with Mac mini dual-homing

### Phase 2 Upgrade Path (Optional)

If double NAT proves insufficient or more granular control is needed:
- Replace CR1000A routers with:
  - Firewalla Gold / Gold Plus (~$400-600)
  - UniFi Dream Machine / UDM Pro (~$200-500)
  - OPNsense on dedicated hardware (~$200-400)
  - pfSense on dedicated hardware (~$200-400)

Phase 2 would add:
- True VLAN per SSID (not just double NAT)
- Layer 3 firewall rules (granular inter-VLAN control)
- mDNS reflection (HomeKit across VLANs)
- Network-wide DNS enforcement
- Better logging and visibility

---

## 🏠 HomeKit Architecture

### HomeKit Bridges

- **Primary Bridge:** Home Assistant HomeKit Bridge
  - Located on Router #1 (192.168.1.21)
  - Exposes selected IoT entities from Router #2 to HomeKit
  - HA reaches IoT devices via Mac mini's IoT interface (192.168.2.10)
- **Camera Hub:** Scrypted
  - Located on Router #1 (192.168.1.20)
  - Pulls camera streams via Mac mini's IoT interface
  - 4 cameras with HKSV (HomeKit Secure Video)
- **Niche Integrations:** Homebridge (only for devices not supported by HA/Scrypted)

**Rule:** ONE source of truth per device. Never double-bridge the same device to HomeKit.

### HomeKit Hubs

| Device | Role | Location | Subnet |
|--------|------|----------|--------|
| Apple TV 4K (Control Room) | Primary hub + Thread border router | Control Room (Ethernet) | 192.168.1.50 |
| Apple TV (Bedroom) | Secondary hub | Bedroom (Wi-Fi) | 192.168.1.51 |
| iPad 5th Gen | Tertiary hub (wall panel) | Hallway (Wi-Fi) | 192.168.1.52 |

### mDNS/Bonjour Discovery

- **Current:** mDNS works within Router #1 subnet (192.168.1.0/24)
- **HomeKit devices:** All on Router #1 (Trusted network) — same subnet as HA Bridge
- **No mDNS reflection needed:** IoT devices behind Router #2 are controlled via HA integrations (not mDNS)
- **Phase 2 consideration:** If VLANs added, will need mDNS reflector

---

## 🛠️ Access URLs (Quick Reference)

### Admin Interfaces

- **Router #1 (Trusted):** `http://192.168.1.1`
- **Router #2 (IoT):** `http://192.168.2.1` (accessible from Router #2 LAN devices)
  - From generic Trusted devices (phones, laptops on 192.168.1.x only): Not directly accessible
  - From dual-homed Mac mini: Accessible via its IoT interface (192.168.2.10)
  - Alternative: Connect phone/laptop to ByteMe (Router #2) SSID, then access `http://192.168.2.1`

### Core Services

- **Home Assistant:** `http://192.168.1.21:8123` (or .20 if bridged)
- **Scrypted:** `https://192.168.1.20:10443` (or port 11080 for HTTP)

### Smart Hubs (Behind Router #2)

- **Philips Hue Bridge:** `http://192.168.2.50` (accessible via Mac mini's IoT interface or from Router #2 LAN devices)
- **Aqara Hub M2:** Via Aqara app (no web UI)

### Cameras (Behind Router #2, Local Admin)

- **Kitchen Camera:** `http://192.168.2.150` (Tapo web UI, if available)
- **Hallway Camera:** `http://192.168.2.151`
- **Entrance Camera:** `http://192.168.2.152`
- **Office Camera:** `http://192.168.2.153`

**Note:** Cameras are on Router #2 subnet (192.168.2.x) with internet blocked. Accessible via Mac mini's IoT interface (for Scrypted) or from Router #2 LAN devices directly.

---

## 🧪 Validation Tests (Health Checks)

### Test 1: Verify Intentional Double NAT

**From Mac mini Terminal (Router #1 LAN):**
```bash
netstat -nr | grep default
# Should show ONLY 192.168.1.1 as gateway
```

**From IoT device on Router #2 LAN (e.g., SSH to Raspberry Pi if available):**
```bash
netstat -nr | grep default
# Should show ONLY 192.168.2.1 as gateway
```

**Expected:** Two separate gateways confirm double NAT is working.

### Test 2: NAT Behavior (OPTIONAL - Informational Only)

**Note:** NAT does NOT guarantee isolation. This test is informational only.

**From IoT device on Router #2 LAN:**
```bash
# Try to ping Mac mini (Router #1 LAN)
ping 192.168.1.20
# May succeed or fail depending on configuration

# Try to ping Router #1 gateway
ping 192.168.1.1
# May succeed or fail depending on configuration
```

**Note:** Do not rely on this test to confirm security. Endpoint firewall rules provide enforcement (Phase 1B).

### Test 3: Mac mini Dual-Homing Validation (REQUIRED)

**From Home Assistant (Router #1 LAN):**
1. Access HA: `http://192.168.1.21:8123`
2. Go to **Settings → Devices & Services**
3. Check IoT integrations (Hue Bridge, Aqara Hub, cameras)
4. **Expected:** All integrations show "Connected" or "Available"

**From Mac mini Terminal (which is dual-homed):**
```bash
# Verify both interfaces are active
ifconfig | grep "inet "
# Should show both 192.168.1.20 and 192.168.2.10

# Ping IoT devices via IoT interface
ping 192.168.2.50  # Hue Bridge
ping 192.168.2.51  # Aqara Hub
ping 192.168.2.150 # Kitchen Camera
# All should SUCCEED (via Mac mini's IoT interface)
```

**Expected:** Mac mini reaches IoT devices via its IoT interface (192.168.2.10), NOT via routing.

### Test 4: HomeKit Response Time

1. Open Home app on iPhone/iPad
2. Toggle Bedroom Vanity Light (via HA Bridge)
3. **Expected:** Instant or 1-2 second response (no mDNS issues, all hubs on same subnet)

### Test 5: Camera Streams

1. Access Scrypted: `https://192.168.1.20:10443`
2. View all 4 cameras
3. **Expected:** All cameras streaming locally (Scrypted reaches cameras via Mac mini's IoT interface)

### Test 6: Camera Internet Block

1. Try accessing Tapo cloud service (via Tapo app or web)
2. **Expected:** Cameras show "offline" or "unreachable" (WAN blocked via Access Control)
3. But camera streams work locally in Scrypted (LAN access allowed)

### Test 7: IP Address Stability

1. Access Router #1: `http://192.168.1.1` → **DHCP Connections**
2. Verify trusted devices show correct static IPs (192.168.1.x range)
3. Access Router #2: `http://192.168.2.1` → **DHCP Connections**
4. Verify IoT/camera devices show correct static IPs (192.168.2.x range)
5. **Expected:** No IP changes after reboot

---

## 🚨 Troubleshooting Guide

### Issue: Can't access Home Assistant from trusted devices

**Possible causes:**
- Mac mini Ethernet cable unplugged
- HA VM stopped or crashed
- IP address changed (DHCP reservation failed on Router #1)

**Troubleshooting steps:**
1. Check Mac mini physical Ethernet connection to Router #1 LAN
2. Access Mac mini (SSH or physical): check UTM VM status
3. Restart HA VM: UTM app → HA VM → Start
4. Verify HA IP in Router #1 DHCP Connections: should be 192.168.1.21 (or .20)
5. Try alternate IP: `http://192.168.1.20:8123` (if bridged networking)

### Issue: Can't access Router #2 admin interface from trusted network

**Context:** Router #2 admin (`http://192.168.2.1`) is on the IoT subnet (192.168.2.x). Generic Trusted devices (phones, laptops on 192.168.1.x only) cannot reach it directly.

**Solutions:**
1. **From Mac mini (dual-homed):** Access `http://192.168.2.1` directly via the Mac mini's IoT interface
2. **From other devices:** Connect phone/laptop to ByteMe or Cameras SSID (Router #2 Wi-Fi), then access `http://192.168.2.1`
3. **Wired option:** Connect Ethernet device directly to Router #2 LAN port
4. Make configuration changes, then reconnect to trusted network when done

### Issue: IoT devices not responding in HA

**Possible causes:**
- IoT device offline (lost Wi-Fi connection)
- IoT device has wrong IP (DHCP reservation failed on Router #2)
- Mac mini IoT interface down or misconfigured

**Troubleshooting steps:**
1. From Mac mini, verify IoT interface is active: `ifconfig | grep 192.168.2`
2. From Mac mini, ping IoT device via IoT interface: `ping 192.168.2.50` (Hue Bridge)
3. If ping fails from Mac mini, check Mac mini's IoT interface connection to Router #2 LAN
4. Connect to Router #2 admin and check DHCP Connections
5. Verify IoT device has correct IP reservation on Router #2

### Issue: Cameras offline in Scrypted

**Possible causes:**
- Camera lost Wi-Fi connection (Router #2 Cameras SSID)
- Camera IP changed (DHCP reservation failed on Router #2)
- Mac mini IoT interface down or misconfigured

**Troubleshooting steps:**
1. From Mac mini, verify IoT interface is active: `ifconfig | grep 192.168.2`
2. From Mac mini, ping cameras via IoT interface: `ping 192.168.2.150` (Kitchen Camera)
3. If ping fails, check Mac mini's IoT interface connection to Router #2 LAN
4. Connect to Router #2 admin and check DHCP Connections
5. Verify cameras have correct IP reservations on Router #2
6. Update camera IPs in Scrypted if changed: Edit camera → change IP → Save

### Issue: HomeKit devices not responding

**Possible causes:**
- mDNS/Bonjour blocked (should not happen, all hubs on same subnet)
- HA HomeKit Bridge offline
- Apple TV hub offline

**Troubleshooting steps:**
1. Check HA is accessible: `http://192.168.1.21:8123`
2. Restart HomeKit Bridge: HA → **Settings → Devices & Services → HomeKit Bridge → Restart**
3. Restart Apple TV hubs: **Settings → System → Restart**
4. Verify all HomeKit hubs are on Router #1 subnet (192.168.1.x)

### Issue: Slow Wi-Fi on trusted devices

**Possible causes:**
- Device stuck on 2.4 GHz band
- Wi-Fi congestion
- Band Steering re-enabled

**Troubleshooting steps:**
1. Verify Band Steering disabled: Router #1 → Wi-Fi → Radio Management
2. Check device connection: iPhone → Settings → Wi-Fi → (i) → should show 5 or 6 GHz
3. Forget network and reconnect (may force 5/6 GHz selection)
4. Check for interference: Use Wi-Fi analyzer app

---

## 📋 Related Documentation

- **Pre-Flight Checklist:** [phase1-preflight-checklist.md](phase1-preflight-checklist.md)
- **Phase 1A Implementation:** [phase1a-implementation-guide.md](phase1a-implementation-guide.md)
- **Phase 1B Implementation:** [phase1b-implementation-guide.md](phase1b-implementation-guide.md)
- **Phase 1 Decision Guide:** [phase1-decision-guide.md](phase1-decision-guide.md)
- **Device Inventory:** [../00-Inventory/device-inventory.csv](../00-Inventory/device-inventory.csv)
- **Home Assistant Baseline:** [../02-HomeAssistant/ha-baseline.md](../02-HomeAssistant/ha-baseline.md)

---

**Last Updated:** 2026-01-22
**Network Design:** Dual-Router Design (Router #1 = Trusted 192.168.1.x, Router #2 = IoT 192.168.2.x)
**IP Scheme:** Organized ranges (Router #1: .20-49 servers, .50-99 trusted, .100-254 dynamic | Router #2: .2-49 infra, .50-149 IoT, .150-199 cameras, .200-254 dynamic)
**Connectivity:** Mac mini dual-homed (192.168.1.20 + 192.168.2.10) enables HA/Scrypted to reach IoT devices

---

## 🎯 Phase 1 Complete

This network architecture is **operational and validated** as of 2026-01-22.

For complete Phase 1 summary including:
- Key lessons learned (Verizon CR1000A behavior, Scrypted conflicts, HomeKit mDNS)
- Architecture decisions and justifications
- Exit criteria validation
- Known limitations and trade-offs
- Maintenance recommendations

**See:** [PHASE1-COMPLETION-SUMMARY.md](PHASE1-COMPLETION-SUMMARY.md)
