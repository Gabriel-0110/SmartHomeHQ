# SmartHomeHQ — TASKS

This file is the single source of truth for current work.
Update statuses as you progress.

Legend:
- [ ] Not started
- [~] In progress
- [x] Done
- [!] Blocked

---

## Now (Phase 0) ✅ COMPLETED
- [x] Confirm `00-Inventory/device-inventory.csv` is complete (devices, hubs, IPs if known)
- [x] Add screenshot bundle: HA → Settings → Devices & services (integrations list)
- [x] Generate `00-Inventory/ownership-map.md`
- [x] Generate `00-Inventory/homekit-exposure-list.md`
- [x] Update `roadmap.md` with refined cutover/rollback plan

**Phase 0 Outputs:**
- [device-inventory.csv](00-Inventory/device-inventory.csv) - **34 devices catalogued** (updated)
- [ownership-map.md](00-Inventory/ownership-map.md) - Platform ownership defined
- [homekit-exposure-list.md](00-Inventory/homekit-exposure-list.md) - **20 devices to expose** (updated)
- [phase0-finalized-inventory.md](00-Inventory/phase0-finalized-inventory.md) - Complete analysis
- [phase0-summary.md](00-Inventory/phase0-summary.md) - **Updated summary with all devices**
- [phased-migration-roadmap.md](00-Inventory/phased-migration-roadmap.md) - Detailed migration guide
- [screenshots/](00-Inventory/screenshots/) - HA integrations reference images

**Phase 0 Key Findings:**
- **34 total devices** (was 19) - discovered 15 additional devices
- **20 devices exposed to HomeKit** (was 13)
- **7 Hue lights, 4 cameras, 11 Aqara sensors, 2 Echo devices, 2 Apple TVs, iPad dashboard**
- **Missing info**: 2 Hue light locations, most IP addresses, Aqara M2 hub count (1 or 2?)

## Next (Phase 1 — Network) ✅ COMPLETED
- [x] Confirm Verizon router model + capabilities (CR1000A: no VLANs, no per-SSID firewall)
- [x] Decide segmentation approach:
  - [x] Dual-router topology (intentional double NAT)
  - [x] Mac mini dual-homed (192.168.1.20 + 192.168.2.10)
- [x] Produce firewall rule matrix (endpoint firewalls + Access Control)
- [x] Plan mDNS strategy (all HomeKit bridges on Router #1 subnet)
- [x] Define rollback steps (documented in phase1-preflight-checklist.md)

**Phase 1 Outputs:**
- [PHASE1-COMPLETION-SUMMARY.md](01-Network/PHASE1-COMPLETION-SUMMARY.md) - **Complete architecture & lessons learned**
- [current-network.md](01-Network/current-network.md) - Final network topology (dual-router)
- [phase1a-implementation-guide.md](01-Network/phase1a-implementation-guide.md) - Network foundation guide
- [phase1b-implementation-guide.md](01-Network/phase1b-implementation-guide.md) - Endpoint hardening guide
- [phase1-decision-guide.md](01-Network/phase1-decision-guide.md) - Architecture decisions
- [cr1000a-audit-results.md](01-Network/router/cr1000a-audit-results.md) - Router capability assessment

**Phase 1 Key Achievements:**
- **Dual-router architecture operational** (Router #1: Trusted 192.168.1.x, Router #2: IoT 192.168.2.x)
- **Mac mini dual-homed** (HA/Scrypted reach IoT devices without routing)
- **Cameras isolated with HKSV** (internet blocked, local RTSP, 10-day recording active)
- **Scrypted consolidated** (Desktop App only, Ciao mDNS, random ports)
- **HomeKit stability restored** (mDNS discovery working, <2 sec response time)
- **UPnP/port forwarding removed** (Router #1 stability restored)
- **72-hour uptime validation passed** (zero service interruptions)

**Phase 1 Exit Criteria:** ✅ All met (see [PHASE1-COMPLETION-SUMMARY.md](01-Network/PHASE1-COMPLETION-SUMMARY.md#network-phase-exit-criteria--met))

## Later (HA + UX)
- [ ] Phase 4: Areas/Zones/Labels/Naming
- [ ] Phase 5: Dashboard v1 spec + layout
- [ ] Phase 6: Automations pack (daily + security + maintenance)
- [ ] Phase 7: Hardening + monitoring cadence
