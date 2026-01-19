# SmartHomeHQ — Claude Code Instructions

You are helping manage a HomeKit-first smart home repository.
This repo is documentation + configuration, NOT an app.

## North Star (non-negotiables)
- HomeKit is the primary user experience.
- Home Assistant (HA) is the central hub/automation brain (UTM VM on Mac mini M4).
- Homebridge is allowed only for niche/compatibility integrations.
- Scrypted is the camera hub and should provide HKSV where possible.
- One source of truth per device. Never double-bridge the same device to HomeKit.
- Prefer local control and privacy. Minimize cloud dependencies.

## Repo workflow rules
1) Treat repo files as the source of truth. Never invent device details.
2) When asked to produce changes, output them as:
   - file path(s)
   - full file contents (ready to paste)
   - a short validation checklist
3) Keep HomeKit clean: expose only the entities that humans use daily.
4) For risky network changes, always include rollback steps and validation tests.
5) Never include secrets in commits. Redact IPs if requested; otherwise use placeholders.

## Canonical “baseline” files (always read first)
- 00-Inventory/device-inventory.csv
- 01-Network/current-network.md
- 02-HomeAssistant/ha-baseline.md
- roadmap-prompts.md

## Naming conventions
- Device display names: "<Room> <Category> <Descriptor>"
- HA entity IDs: "<domain>.<room>_<function>[_<descriptor>]"
- Use consistent Areas, Labels, Zones (see Phase 4).

## Output style
- Be concise.
- Prefer checklists and step-by-step instructions.
- Include validation checks after each step.
