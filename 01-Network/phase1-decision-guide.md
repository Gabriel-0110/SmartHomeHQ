# Phase 1: Network Segmentation — Decision Guide

**Date:** 2026-01-19
**Status:** ✅ DECISIONS FINALIZED (INTENTIONAL DOUBLE NAT DESIGN)

---

## 🎯 Your Goals

- ✅ Router-level AdBlock (block tracking, ads, malware domains)
- ✅ Privacy: stop devices from tracking/leaking data
- ✅ Security: isolate IoT devices from trusted devices
- ✅ Stability: prevent devices from randomly switching networks
- ✅ **HomeKit stability is TOP PRIORITY**

---

## ✅ FINAL ARCHITECTURE (NON-NEGOTIABLE)

### Topology Overview

```
ISP → Router #1 (Trusted / Control Plane) → Router #2 WAN (IoT / Edge Network)
      192.168.1.0/24                         192.168.2.0/24
```

### Core Design Principles

- **Two IDENTICAL Verizon CR1000A routers**
- **Router #2 REMAINS in ROUTER MODE** (NOT access point mode)
- **Intentional DOUBLE NAT is REQUIRED**
- **Two separate subnets** (not one)
- **Router #1 = Trusted / Control Plane** (192.168.1.0/24)
- **Router #2 = IoT / Edge Network** (192.168.2.0/24)

### Why This Design?

**CRITICAL CONSTRAINT**: Verizon CR1000A routers **cannot** do:
- True VLANs with inter-VLAN firewall rules
- Per-SSID subnet assignment
- mDNS reflection across networks

**INTENTIONAL DOUBLE NAT RATIONALE**:

✅ **Pros**:
1. **Two separate subnets**: Trusted (192.168.1.x) and IoT (192.168.2.x) are organizationally separated
2. **Simplicity**: No mDNS reflection needed — HomeKit devices stay on Router #1
3. **Works with existing hardware**: No new equipment purchase required
4. **Architecture enables control**: Dual-homed Mac mini allows HA/Scrypted to reach both subnets
5. **Reduced exposure**: Separation provides visibility and organizational control

❌ **Acceptable Trade-offs**:
1. **Double NAT exists**: This is INTENTIONAL, not a bug
2. **mDNS/Bonjour does NOT traverse NAT**: HomeKit devices behind Router #2 won't work via Router #2's Wi-Fi — this is by design (HomeKit devices stay on Trusted network)
3. **Mac mini MUST be dual-homed**: HA/Scrypted reach IoT via second interface (not via routing)
4. **NAT is NOT a security boundary**: It only reduces unsolicited inbound traffic; endpoint firewalls provide enforcement

### Device Placement Rules

| Device Type | Router | Subnet | Reason |
|-------------|--------|--------|--------|
| Home Assistant (VM) | Router #1 | 192.168.1.x | Central hub on Trusted network |
| Scrypted (Mac mini host) | Router #1 | 192.168.1.x | Camera hub on Trusted network |
| Mac mini M4 (host) | DUAL-HOMED | 192.168.1.x + 192.168.2.x | Interface on BOTH routers for HA/Scrypted IoT access |
| HomeKit hubs (Apple TV, iPad) | Router #1 | 192.168.1.x | Must communicate with HA Bridge |
| Phones, PCs, tablets | Router #1 | 192.168.1.x | Trusted user devices |
| **All IoT devices** | Router #2 | 192.168.2.x | Separated on IoT subnet |
| **All cameras** | Router #2 | 192.168.2.x | Separated on IoT subnet |
| Hubs (Hue, Aqara) | Router #2 | 192.168.2.x | Separated on IoT subnet |

**CRITICAL**: Mac mini MUST be dual-homed (one interface on each router) so that Home Assistant and Scrypted can reach IoT devices on 192.168.2.x.

---

## 🔴 FINAL DECISIONS (Selected)

### Decision 1: Network Strategy
**✅ SELECTED: Intentional Double NAT (Two Routers, Two Subnets)**

**Configuration:**
```
Router #1 (Trusted / Control Plane):
- WAN: ISP modem
- LAN subnet: 192.168.1.0/24
- Devices: HA, Scrypted, Mac mini, phones, PCs, Apple TVs, iPad
- SSIDs: Primary (trusted devices only)

Router #2 (IoT / Edge Network):
- WAN: Router #1 LAN port (intentional cascade)
- LAN subnet: 192.168.2.0/24
- Devices: All IoT, all cameras, Hue, Aqara
- SSIDs: IoT (ByteMe), Cameras (dedicated)
```

**Why This Decision:**
- **Subnet separation**: Router #2 provides a separate 192.168.2.x subnet for IoT
- **Mac mini dual-homing**: HA/Scrypted reach IoT via second interface (not via routing)
- **No mDNS needed**: HomeKit devices stay on Router #1 (same subnet as HA Bridge)
- **Works with CR1000A limitations**: No VLAN support required
- **NAT is NOT a firewall**: Endpoint enforcement is primary isolation control

---

### Decision 2: DNS Strategy
**✅ SELECTED: Per-Device DNS for Trusted, Router DNS for IoT**

**Configuration:**
- **Router #1 (Trusted)**: Configure per-device DNS (AdGuard DNS IPs) on critical devices:
  - iPhones/iPads: Wi-Fi settings → Configure DNS → Manual
  - Apple TVs: Network → Configure DNS → Manual
  - macOS (Mac mini): System Settings → Network → DNS
  - Windows PCs: Network adapter → DNS settings
- **Router #2 (IoT)**: Let router assign default DNS (or set manually if UI allows)
  - IoT devices don't need ad blocking (less maintenance)
  - Cameras are internet-blocked anyway

**Optional**: Run AdGuard Home locally (HA add-on) and point devices to 192.168.1.21 (HA IP).

**Why This Decision:**
- CR1000A does not support custom LAN DHCP DNS push
- Per-device DNS ensures trusted devices use AdGuard
- IoT devices are isolated and don't need strict DNS control

---

### Decision 3: Wi-Fi Stability
**✅ SELECTED: Disable Band Steering/SON + Band Assignment per SSID**

**Configuration:**
- **Router #1 (Trusted)**:
  - Disable Band Steering/SON
  - Primary SSID: Enable 5 GHz + 6 GHz (disable 2.4 GHz if possible)
  - Security: WPA2/WPA3
- **Router #2 (IoT)**:
  - Disable Band Steering/SON
  - IoT SSID (ByteMe): 2.4 GHz ONLY
  - Cameras SSID: 2.4 GHz ONLY
  - Security: WPA2

**Why This Decision:**
- Prevents devices from randomly switching bands/SSIDs
- 2.4 GHz for IoT/cameras (better range, wall penetration)
- 5/6 GHz for trusted devices (faster speeds, less congestion)

---

### Decision 4: HomeKit Discovery Strategy
**✅ SELECTED: Home Assistant HomeKit Bridge Owns Discovery (Trusted Network Only)**

**Configuration:**
- **HA HomeKit Bridge**: Primary bridge for all non-camera devices
  - HA on Router #1 (Trusted network)
  - Exposes selected IoT entities from Router #2 to HomeKit
  - HA reaches IoT devices via Mac mini's IoT interface (dual-homed)
- **Scrypted**: Camera hub (HKSV where possible)
  - Scrypted on Router #1 (Trusted network)
  - Pulls camera streams from Router #2 via Mac mini's IoT interface
- **Homebridge**: Compatibility layer only (when absolutely needed)
  - Avoid double-bridging same devices

**Why This Decision:**
- Mac mini dual-homing enables HA/Scrypted to reach all IoT devices
- HomeKit hubs (Apple TVs, iPad) are on same subnet as HA (192.168.1.x) — mDNS works
- No mDNS reflection needed (HomeKit devices not behind Router #2)
- Matches CLAUDE.md north star: HA is central hub

---

### Decision 5: Camera Isolation Level
**✅ SELECTED: Camera Separation (Router #2 Subnet + No Internet)**

**Configuration:**
- Cameras on Router #2 (IoT network) subnet 192.168.2.x
- Dedicated Cameras SSID (2.4 GHz only)
- Block internet access using Router #2 Access Control / Parental Controls
- Cameras can ONLY:
  - Reach Scrypted via Mac mini's IoT interface (192.168.2.10)
  - Cannot reach internet (blocked by Access Control)

**Why This Decision:**
- **Maximum privacy**: Cameras can't phone home to Tapo cloud
- **Local access only**: Scrypted pulls streams via Mac mini's dual-homed IoT interface
- **No NAT dependency**: Mac mini reaches cameras directly on same subnet (192.168.2.x)

**Trade-offs:**
- Can't use Tapo mobile app for camera config (use Scrypted UI instead)
- Can't auto-update firmware (must allow temporarily or manually update)

---

### Decision 6: IP Address Management
**✅ SELECTED: Dual Subnet with Organized IP Ranges**

**Configuration:**

**Router #1 (Trusted) - 192.168.1.0/24:**
```
192.168.1.1           Router #1 gateway
192.168.1.2-19        Reserved for network infrastructure
192.168.1.20-49       Servers & Core Services
  .20                 Mac mini M4 (host for HA + Scrypted)
  .21                 Home Assistant (VM)
  .22-49              Reserved for future services
192.168.1.50-99       Trusted Static Devices
  .50                 Apple TV 4K (Control Room hub)
  .51                 Apple TV (Bedroom)
  .52                 iPad 5th Gen (wall panel)
  .53-99              Reserved for PCs/laptops
192.168.1.100-254     DHCP Dynamic Pool (phones, tablets, PCs)
```

**Router #2 (IoT) - 192.168.2.0/24:**
```
192.168.2.1           Router #2 gateway (WAN from Router #1)
192.168.2.2-49        Reserved for infrastructure
192.168.2.50-149      IoT Devices (DHCP or static)
  .50                 Philips Hue Bridge
  .51                 Aqara Hub M2
  .52                 Echo Show 8
  .53                 Echo Dot
  .54                 Nest Mini
  .55                 HP Printer
  .56                 Toshiba Microwave
  .57-149             Reserved for future IoT
192.168.2.150-199     Cameras (Static Reservations REQUIRED)
  .150                Kitchen Camera
  .151                Hallway Camera
  .152                Entrance Camera
  .153                Office Camera
  .154-199            Reserved for future cameras
192.168.2.200-254     DHCP Dynamic Pool (IoT devices if needed)
```

**Why This Decision:**
- **Clean separation**: Trusted vs IoT subnets
- **Organized ranges**: Easy to identify device types by IP
- **Static for critical**: HA, Scrypted, cameras get fixed IPs
- **DHCP for flexibility**: Consumer devices auto-configure

---

### Decision 7: Router Security Hardening
**✅ SELECTED: Full Hardening (Both Routers)**

**Configuration:**
- **Both routers**:
  - Disable WPS (Wi-Fi Protected Setup)
  - Disable UPnP (Universal Plug and Play)
  - Disable Band Steering/SON
  - No port forwards (except if explicitly justified)
  - DMZ off
  - IPv6: Review and set intentionally (disable if not using)
  - Remote admin: Disabled
- **Router #2 specific**:
  - Block camera internet access via Access Control (by MAC address)
  - Firewall: Default deny all inbound (from WAN/internet)

**Why This Decision:**
- WPS is a known security vulnerability
- UPnP allows devices to open ports automatically (security risk)
- Port forwards create inbound exposure
- Best practice: deny by default

---

## 📋 Implementation Path

### Phase 1A: Network Foundation + IP Reorganization
**Target:** Configure dual-router topology, migrate devices, organize IPs

**Steps:**
1. Pre-change snapshots (screenshots of both routers)
2. Configure Router #2 WAN to connect to Router #1 LAN (already done, verify)
3. Configure Router #2 LAN subnet to 192.168.2.0/24 (change from 192.168.1.x or 192.168.0.x)
4. Disable Band Steering on both routers
5. Configure SSID band assignments (2.4 vs 5/6 GHz)
6. Disable WPS, UPnP on both routers
7. Create static reservations for HA, Scrypted, cameras, hubs
8. Migrate devices to correct routers/SSIDs
9. Block camera internet access (Access Control on Router #2)
10. Validation tests

**Estimated time:** 2-3 hours

**See:** [phase1a-implementation-guide.md](phase1a-implementation-guide.md)

---

### Phase 1B: Endpoint Hardening + Access Control Validation
**Target:** Protect trusted endpoints, configure endpoint firewalls

**Steps:**
1. Home Assistant hardening (MFA, least privilege, backup)
2. macOS Firewall (Mac mini host)
3. Windows Firewall (if applicable)
4. Router security review (DMZ off, port forwards empty, IPv6 intentional)
5. Verify Mac mini dual-homing (can reach both 192.168.1.x and 192.168.2.x)
6. Test HA/Scrypted can reach IoT devices
7. Test camera streams work in Scrypted

**Estimated time:** 1-1.5 hours

**See:** [phase1b-implementation-guide.md](phase1b-implementation-guide.md)

---

### Phase 2: Hardware Upgrade (Future, Optional)
**Target:** Replace CR1000A with true VLAN-capable router/firewall

**Options:**
- Firewalla Gold / Gold Plus (~$400-600)
- UniFi Dream Machine / UDM Pro (~$200-500)
- OPNsense on dedicated hardware (~$200-400)
- pfSense on dedicated hardware (~$200-400)

**Phase 2 would add:**
- True VLAN per SSID (not just double NAT)
- Layer 3 firewall rules (granular control)
- mDNS reflection (HomeKit across VLANs)
- Network-wide DNS enforcement
- Better logging and visibility

**Status:** Not required for Phase 1; document as future option if double NAT proves insufficient.

---

## 🚨 Critical Constraints Summary

| Feature | CR1000A Support | Phase 1 Approach | Phase 2 Path |
|---------|-----------------|------------------|--------------|
| Dual subnet separation | ✅ YES (via double NAT) | Router #2 in router mode | New router with VLANs |
| Mac mini dual-homing | ✅ YES | Second interface on Router #2 | Same |
| HA/Scrypted IoT access | ✅ YES | Via Mac mini IoT interface | Same |
| Endpoint isolation | ✅ YES (host firewalls) | macOS/Windows firewalls | Layer 3 firewall rules |
| mDNS across subnets | ❌ NO | Not needed (HomeKit on Router #1) | mDNS reflector |
| LAN DHCP DNS push | ❌ NO | Per-device DNS for trusted | Same |
| Camera internet block | ✅ YES | Access Control (Router #2) | Same |
| Static reservations | ✅ YES | Use for HA, cameras, hubs | Same |
| Band control per SSID | ✅ YES | 2.4 only for IoT/Cameras | Same |
| WPS disable | ✅ YES | Disable in Phase 1A | N/A |
| UPnP disable | ✅ YES | Disable in Phase 1A | N/A |

---

## ⚡ Quick Start

**If you accept the FINAL ARCHITECTURE (intentional double NAT, two subnets):**

Reply: **"Proceed with Phase 1A"**

I will guide you through:
1. Pre-change screenshots (rollback capability)
2. Router configuration changes (both routers)
3. Device migration to correct routers/SSIDs/IPs
4. Validation testing (verify isolation works)
5. Troubleshooting if needed

**Estimated time:** 2-3 hours (includes testing)

**Rollback plan:** Included in Phase 1A guide (can revert if critical issues occur)

---

**Last Updated:** 2026-01-19
**Architecture:** DUAL-ROUTER DESIGN (two routers, two subnets, Mac mini dual-homed for IoT access)
