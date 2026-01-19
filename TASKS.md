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

## Next (Phase 1 — Network)
- [ ] Confirm Verizon router model + capabilities (VLAN? guest isolation? mDNS?)
- [ ] Decide segmentation approach:
  - [ ] VLANs (ideal) OR
  - [ ] SSID split + client isolation (good enough)
- [ ] Produce firewall rule matrix (Trusted/IoT/Cameras)
- [ ] Plan mDNS strategy (reflector/allow rules)
- [ ] Define rollback steps

## Later (HA + UX)
- [ ] Phase 4: Areas/Zones/Labels/Naming
- [ ] Phase 5: Dashboard v1 spec + layout
- [ ] Phase 6: Automations pack (daily + security + maintenance)
- [ ] Phase 7: Hardening + monitoring cadence
