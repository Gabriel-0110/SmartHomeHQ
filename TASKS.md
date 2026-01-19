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
- [device-inventory.csv](00-Inventory/device-inventory.csv) - 19 devices catalogued
- [ownership-map.md](00-Inventory/ownership-map.md) - Platform ownership defined
- [homekit-exposure-list.md](00-Inventory/homekit-exposure-list.md) - 13 devices to expose
- [phase0-finalized-inventory.md](00-Inventory/phase0-finalized-inventory.md) - Complete analysis
- [phased-migration-roadmap.md](00-Inventory/phased-migration-roadmap.md) - Detailed migration guide
- [screenshots/](00-Inventory/screenshots/) - HA integrations reference images

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
