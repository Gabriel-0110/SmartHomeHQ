# Decisions Summary - Ready to Proceed

**Date:** 2026-01-18
**Phase 0 Status:** ✅ COMPLETE with all decisions finalized

---

## ✅ All Decisions Made

| # | Decision Area | Choice | Status |
|---|---------------|--------|--------|
| 1 | Aqara Sensor Exposure | Via HA HomeKit Bridge (Option B) | ✅ Documented |
| 2 | Adaptive Lighting | HA Component (HA-first) | ✅ Documented |
| 3 | Google Assistant | Keep (Nest Mini in use) | ✅ Documented |
| 4 | Phase 4 Naming | Proceed with standardization | ✅ Documented |
| 5 | Network Rebuild | Proceed from scratch | ✅ Documented |

---

## 📊 Updated Inventory

### New Devices Added
- **Nest Mini (1st Gen)** - Bathroom (Voice assistant + speaker)
- **Toshiba Microwave** (ML-SEM23P(BS)) - Kitchen (Smart control, exposed to HomeKit)
- **HP Deskjet 3630** - Office (Wireless printing, HA-managed)

### Updated Totals
- **Total devices**: 38 (was 35)
- **HomeKit exposed**: 21 (was 20) - added Microwave
- **Voice Assistants**: 3 (was 2) - added Nest Mini

---

## 🌐 Network Information Captured

### Router
- **Model**: Verizon CR1000A
- **Bands**: 2.4 / 5 / 6 GHz (tri-band)

### Current SSIDs
1. **Primary**: "EnterAtYourOwnRisk" (2.4/5/6 GHz) - Trusted devices
2. **Guest**: "HotNeighborsCanConnect" (2.4 GHz) - Guest network
3. **IoT**: "ByteMe" (2.4 GHz) - IoT devices

### IP Addresses (192.168.1.x)
- Home Assistant VM: `.242`
- Scrypted: `.224` (primary), also `.209`, `.171` (needs investigation)
- Hue Bridge: `.169`
- Aqara Hub M2: `.165`
- Cameras: Office `.204`, Kitchen `.210`, Entrance `.155`, Hallway `.236`

---

## 🎯 Architectural Direction

### HomeKit-First with Multi-Ecosystem
- **Primary**: HomeKit/Siri (user experience)
- **Secondary**: Google Assistant (Nest Mini) + Alexa (Echo devices)
- **Brain**: Home Assistant (all automation logic)

### Device Exposure Strategy
- **Hue lights** (5): Native Hue Bridge → HomeKit
- **WiFi devices** (10): HA → HomeKit Bridge
- **Cameras** (4): Scrypted → HKSV
- **Aqara sensors** (5): HA → HomeKit Bridge

### Adaptive Lighting
- **All lights managed by HA Adaptive Lighting component**
- Consistent approach across Hue and Govee
- HA controls via respective integrations (Hue Bridge, Govee Cloud)

---

## 🚀 Ready for Phase 1

### Prerequisites Met
- ✅ Complete device inventory (38 devices)
- ✅ All architectural decisions documented
- ✅ Network baseline captured (CR1000A + SSIDs + IPs)
- ✅ Risk acceptance for Phase 4 naming standardization

### Next Phase: Network Segmentation
**Objective**: Build secure, isolated network with 3 zones:
1. **Trusted**: Phones, PCs, Apple TV, Mac mini, HA, iPad
2. **IoT**: Plugs, lights, panels, hubs, Alexa/Google devices
3. **Cameras**: 4x Tapo C110 cameras (isolated)

**Approach**: Leverage existing 3-SSID structure, add firewall rules, verify HomeKit mDNS

---

## 📋 Outstanding Items (Non-Blocking)

### To Investigate
1. Scrypted multiple IPs (`.224`, `.209`, `.171`) - which is primary?
2. CR1000A VLAN capabilities (via Verizon app research)
3. Current firewall rules between SSIDs
4. Control Room physical location (for cable planning)

### Phase 1 Research Tasks
- CR1000A mDNS/Bonjour support for HomeKit across SSIDs
- Firewall rule matrix design
- Device SSID placement audit
- Static IP reservation plan

---

## 📁 Updated Files

### Core Documents
- [device-inventory.csv](00-Inventory/device-inventory.csv) - 38 devices
- [phase0-decisions.md](00-Inventory/phase0-decisions.md) - Full decision rationale
- [current-network.md](01-Network/current-network.md) - CR1000A + SSIDs + IPs
- [roadmap.md](roadmap.md) - Phase 4 marked "PROCEED"

### Summary Files
- [phase0-summary.md](00-Inventory/phase0-summary.md)
- [ownership-map.md](00-Inventory/ownership-map.md)
- [homekit-exposure-list.md](00-Inventory/homekit-exposure-list.md)

---

## ✅ Validation Checklist

- [x] All 5 architectural decisions documented
- [x] Network information captured (router, SSIDs, IPs)
- [x] New devices added to inventory (Nest Mini, Microwave, Printer)
- [x] Phase 4 status updated to "PROCEED"
- [x] Decisions cross-referenced in relevant docs
- [x] Ready to start Phase 1 planning

---

## 🎓 Key Takeaways

1. **HA-First Architecture**: All logic in Home Assistant, HomeKit is UI
2. **Single Source of Truth**: No double-bridging, clear ownership per device
3. **Organization Over Convenience**: Willing to rebuild for long-term maintainability
4. **Security-Focused**: Network rebuild with proper isolation
5. **Multi-Ecosystem**: HomeKit primary, but keeping Google/Alexa for specific use cases

---

**Status**: Phase 0 finalized. Proceed to Phase 1: Network Segmentation.

**Command to start Phase 1**: `/phase1-network` or manual planning per [roadmap.md](roadmap.md)
