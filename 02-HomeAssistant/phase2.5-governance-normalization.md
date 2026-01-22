# Phase 2.5 – Home Model Governance & Normalization

**Status:** ✅ DONE
**Completed:** 2026-01-22

## Purpose
Phase 2.5 defines canonical rules for naming, areas, zones, labels, and exposure within the smart home model. This phase establishes standards to ensure consistency and clarity across all future configuration and automation work.

**No devices are onboarded and no dashboards are created in this phase.**

## Scope (In / Out)
**In scope:**
- Naming conventions (devices, entities, automations, scripts)
- Areas vs Zones model
- Labels / tags taxonomy
- HomeKit exposure policy
- Dashboard philosophy (rules only)

**Out of scope:**
- Device onboarding
- HomeKit exposure changes
- Dashboard creation
- Automation creation

## Naming Conventions
- All device names, entity IDs, automations, and scripts must follow a clear, predictable schema.
- Names must be human-readable, scalable, and avoid ambiguity.
- Device names: [Room/Area] [Function/Type] [Descriptor] (e.g., "Kitchen Ceiling Light")
- Entity IDs: snake_case, prefixed by area/zone if applicable (e.g., "kitchen_ceiling_light")
- Automations/scripts: [Trigger/Area] [Action/Target] (e.g., "kitchen_motion_turn_on_ceiling_light")

## Areas vs Zones
- **Areas** represent physical spaces (e.g., rooms, floors).
- **Zones** represent logical groupings (e.g., Security, Lighting, Media, HVAC).
- Use Areas for location-based grouping; use Zones for function or system-based grouping.
- Do not use Zones to duplicate Area structure; each serves a distinct purpose.

## Labels / Tags
- Labels are additive, non-hierarchical, and describe attributes or roles.
- Categories:
  - **Function:** lighting, security, media, climate
  - **Criticality:** critical, convenience, decorative
  - **Ownership:** ha, scrypted, homebridge
- Apply multiple labels as needed to reflect device/entity roles.

## HomeKit Exposure Policy
- Only entities representing user-facing, daily-use devices MAY be exposed to HomeKit.
- Diagnostic, helper, or system entities MUST NOT be exposed.
- Each device/entity must have a single source of truth for HomeKit exposure (no double-bridging).

## Dashboard Philosophy
- **Operational dashboards:** For maintenance, diagnostics, and admin use.
- **Daily-use dashboards:** For regular human interaction.
- Dashboard implementation is deferred to Phase 3; only rules are defined here.

## Success Criteria
- Governance rules are fully documented and approved.
- No devices are renamed in this phase.
- No HomeKit exposure changes are made.
- Phase 3 can apply these rules directly, without reinterpretation.

## Exit Conditions
- Phase 2.5 is complete when all standards are frozen and agreed upon by stakeholders.

---

## Implementation Status

**Standards Defined:** ✅ Complete (2026-01-22)

**Execution Status:** ⏸️ Deferred
- This phase defines standards only. No naming, labeling, or HomeKit exposure has been applied.
- Application of these standards (renaming devices, applying labels, creating areas/zones, building dashboards) will occur in a future Home Assistant execution phase, after Phase 4 (Homebridge).
