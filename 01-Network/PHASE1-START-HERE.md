# Phase 1: Network Segmentation - START HERE

**Status:** ✅ **PHASE 1 COMPLETED** (2026-01-22)
**Last Updated:** 2026-01-22
**Completion Report:** See [PHASE1-COMPLETION-SUMMARY.md](PHASE1-COMPLETION-SUMMARY.md)

---

## 🎉 Phase 1 Complete

Phase 1 (Network Segmentation) has been successfully completed and validated. The dual-router architecture is operational with:
- Router #1 (Trusted subnet: 192.168.1.x) + Router #2 (IoT subnet: 192.168.2.x)
- Mac mini dual-homed (192.168.1.20 + 192.168.2.10)
- Cameras isolated with HKSV active (internet blocked, local RTSP)
- HomeKit mDNS discovery stable (<2 sec response time)
- 72-hour uptime validation passed

**Next Phase:** Phase 2 (HomeKit Bridge Cleanup) or Phase 4 (Areas/Zones/Labels)

**This document is retained for historical reference.**

---

## Original Implementation Guide (Historical)

## Documentation Overview

Phase 1 is split into two parts: network foundation (1A) and endpoint hardening (1B).

Phase 1 implements a dual-router, dual-subnet design. Router #2 remains in router mode as the IoT edge, and the Mac mini is dual-homed as the enforcement point for HA/Scrypted access.

### Pre-Implementation Documents

1. **[cr1000a-audit-results.md](cr1000a-audit-results.md)** - READ FIRST
   - Router capabilities assessment
   - Explains why Phase 1 is split into A+B

2. **[phase1-decision-guide.md](phase1-decision-guide.md)** - DECISIONS FINALIZED
   - Dual-router, dual-subnet architecture (Trusted + IoT)
   - Router #2 remains in router mode as the IoT edge
   - Mac mini dual-homed requirement
   - Phase 2 upgrade path documented

3. **[phase1-preflight-checklist.md](phase1-preflight-checklist.md)** - DO BEFORE STARTING
   - Screenshots to capture
   - Access verification
   - Backup procedures
   - Maintenance window planning

### Implementation Guides

4. **[phase1a-implementation-guide.md](phase1a-implementation-guide.md)** - START HERE (Phase 1A)
   - **Time:** 2-3 hours
   - **Focus:** Network foundation (routers, subnets, DHCP, wiring)
   - **Outcomes:**
     - Router #1 (Trusted) + Router #2 (IoT) with separate subnets
     - DHCP scopes and reservations in place
     - IoT devices migrated behind Router #2

5. **[phase1b-implementation-guide.md](phase1b-implementation-guide.md)** - DO AFTER 1A
   - **Time:** 1-1.5 hours
   - **Focus:** Endpoint hardening
   - **Outcomes:**
     - Home Assistant authentication hardening (MFA, least privilege)
     - OS firewalls enabled and reviewed
     - Router security review completed

### Baseline Documents (Updated with Phase 1 changes)

6. **[current-network.md](current-network.md)** - REFERENCE
   - Current network state (pre and post Phase 1)
   - Router specifications (CR1000A)
   - IP address plan
   - CR1000A limitations documented

7. **[../02-HomeAssistant/ha-baseline.md](../02-HomeAssistant/ha-baseline.md)** - REFERENCE
   - Home Assistant configuration
   - HomeKit ownership rules (HA Bridge vs Scrypted)
   - Integration requirements
   - mDNS/discovery strategy

---

## Quick Start: What to Do Next

### Step 1: Pre-Flight Checks (If Not Done)
If you haven't completed the pre-flight checklist:
1. Open [phase1-preflight-checklist.md](phase1-preflight-checklist.md)
2. Complete items 1-8 (especially screenshots and access verification)
3. Return here when done

### Step 2: Start Phase 1A (Network Foundation)
When ready to begin:
1. Open [phase1a-implementation-guide.md](phase1a-implementation-guide.md)
2. Follow the guide end-to-end
3. Allow 2-3 hours for completion
4. Validate connectivity before proceeding to Phase 1B

### Step 3: Start Phase 1B (Endpoint Hardening)
After Phase 1A is stable (same day or next day):
1. Open [phase1b-implementation-guide.md](phase1b-implementation-guide.md)
2. Follow the guide end-to-end
3. Allow 1-1.5 hours for completion

---

## Time Breakdown (Phase 1)

| Phase | Focus | Time |
|------|------|------|
| Phase 1A | Dual-router foundation: wiring, subnets, DHCP scopes, reservations, device migration | 2-3 hours |
| Phase 1B | Endpoint hardening: HA auth, OS firewalls, router security review | 1-1.5 hours |
| **TOTAL** | **Phase 1** | **~3-4.5 hours** |

---

## What Phase 1 Will Achieve

### Achievable in Phase 1A + 1B:

1. **Network-level separation via routers**
   - Two routers, two subnets: Trusted (Router #1) and IoT (Router #2)
   - Router #2 stays in router mode as the IoT edge
   - Mac mini is dual-homed so HA/Scrypted can reach IoT without routing changes

2. **Clear organization (SSIDs are not security boundaries)**
   - SSIDs are used for labeling and device placement only
   - Security separation comes from the router topology and endpoint controls

3. **Endpoint enforcement and hardening**
   - Endpoint firewalls on Mac mini (and Windows, if applicable)
   - Home Assistant MFA and least-privilege accounts
   - Router security review completed

4. **Stable addressing**
   - DHCP scopes per subnet
   - Reservations for critical devices and hubs

### Not in scope for Phase 1 (Phase 2 targets)

- Centralized inter-zone firewall policy with granular rules
- VLAN-based segmentation with mDNS reflection
- Network-wide DNS enforcement and advanced logging

---

## Critical Success Factors

After Phase 1 implementation, verify:

- Router #1 LAN -> Router #2 WAN wiring is correct
- Trusted subnet is 192.168.1.x and IoT subnet is 192.168.2.x
- Mac mini is dual-homed with both interfaces active
- HA and Scrypted are reachable on the Trusted subnet
- HA and Scrypted can reach IoT devices via the IoT interface
- Endpoint firewalls enabled (macOS, Windows if applicable)
- Critical devices have DHCP reservations
- Core integrations are online

---

## Emergency Rollback

**If something breaks:**

### Phase 1A Rollback:
1. Revert router wiring to the previous upstream configuration
2. Restore Router #1 and Router #2 LAN/DHCP scopes from snapshots
3. Remove new DHCP reservations if needed
4. Move devices back to their previous network

### Phase 1B Rollback:
1. Disable HA MFA (if causing issues)
2. Disable macOS Firewall: `sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off`
3. Remove Windows firewall rules (if applicable)

### Nuclear Option:
- Router backup file location: `01-Network/CR1000A_V3.6.0.2_BD_backup.cfg` (if captured)
- Factory reset CR1000A (last resort)

---

## What to Tell Me When Starting

**"Starting Phase 1A now"** - I'll guide you through each step
**"Completed Phase 1A, ready for Phase 1B"** - I'll walk you through endpoint hardening
**"Phase 1 complete, here are results"** - I'll help validate and document
**"I'm on Step X of Phase 1A, need help"** - I'll troubleshoot that specific step

---

## What Happens After Phase 1?

Once Phase 1 is stable (24-48 hours):

- **Phase 2:** Hardware upgrade evaluation (Firewalla, UniFi, OPNsense)
- **Phase 3:** Home Assistant entity organization
- **Phase 4:** Areas/Zones/Labels (HomeKit room structure)
- **Phase 5:** Dashboard creation (iPad wall panel)
- **Phase 6:** Automation library

**But first:** Let's complete Phase 1 network foundation and endpoint hardening.

---

## Ready to Start?

Reply with one of these:

1. **"Start Phase 1A"** - Begin network foundation (dual-router)
2. **"I have questions about X"** - Ask before starting
3. **"Show me the decision summary"** - Review final decisions document

---

**Last Updated:** 2026-01-19
**Next Review:** After Phase 1A + 1B implementation complete
