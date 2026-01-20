# Home Assistant Baseline (Canonical)

**Last Updated:** 2026-01-19

---

## Host + VM

- **Host:** Mac mini M4
- **Virtualization:** UTM
- **HA Install Type:** TBD (HA OS / Supervised / Container - document after verification)
- **VM Networking:** Bridged (preferred for mDNS/Bonjour discovery)
- **VM IP Address:** 192.168.1.242 (static/reserved)
- **VM Resources:**
  - CPU cores: TBD
  - RAM: TBD
  - Disk: TBD
- **Timezone:** America/New_York
- **Access URL:** `http://192.168.1.242:8123`

---

## Home Stack Requirements (North Star)

**From CLAUDE.md (non-negotiable):**
- ✅ **HomeKit is the primary user experience**
- ✅ **Home Assistant is the central hub / automation brain**
- ✅ **Homebridge is allowed for niche integrations only**
- ✅ **Scrypted is the camera hub (HKSV where possible)**
- ✅ **One source of truth per device** (never double-bridge)
- ✅ **Prefer local control and privacy** (minimize cloud dependencies)

---

## HomeKit Ownership / Discovery Rules

### Primary Bridge Owner: Home Assistant HomeKit Bridge
**Responsible for:** Non-camera devices

**Exposed entities (22 devices as per ownership map):**
- Hue lights (7x) via Hue Bridge
- Aqara devices (11x) via Aqara Hub M2
- Kasa WiFi plugs (2x) direct
- Govee lights/strips (2x) direct

**Why HA Bridge:**
- HA sits on Trusted network, can reach IoT/Cameras
- Single subnet (192.168.1.0/24) means mDNS works across all SSIDs
- No mDNS reflection needed (Phase 1)
- Centralized control and automation logic

### Camera Bridge Owner: Scrypted
**Responsible for:** All cameras (4x Tapo C110)

**Exposed to HomeKit:**
- Kitchen Camera (via Scrypted → HKSV)
- Hallway Camera (via Scrypted → HKSV)
- Entrance Camera (via Scrypted → HKSV)
- Office Camera (via Scrypted → HKSV)

**Why Scrypted:**
- Purpose-built camera hub
- HKSV (HomeKit Secure Video) support
- Local RTSP stream handling
- Better performance than HA camera integrations

### Homebridge: Compatibility Layer Only
**Use cases:**
- Devices that don't work with HA native integrations
- Temporary bridge for testing before HA integration available
- Niche plugins not supported by HA

**CRITICAL RULE:** Never double-bridge same device to HomeKit (HA Bridge + Homebridge). Choose one owner per device.

---

## mDNS / Discovery Risk Note

**Current setup (Phase 1):**
- Single LAN subnet: 192.168.1.0/24
- mDNS/Bonjour: Works across all SSIDs (Trusted, IoT, Cameras)
- No special router config needed

**Phase 2 consideration:**
- If VLANs are added (separate subnets per SSID), mDNS will NOT work across VLANs by default
- Will require mDNS reflector/repeater (Avahi, router feature, or dedicated service)
- HA Bridge and Scrypted must remain reachable by HomeKit controllers (iPhones, Apple TVs)

---

## Integrations (Current)

Based on device inventory, the following integrations are required:

### Required Integrations
- **Hue:** REQUIRED (7 Hue lights via Hue Bridge at 192.168.1.169)
- **Aqara:** REQUIRED (11 Aqara devices via Hub M2 at 192.168.1.165)
- **Tapo:** OPTIONAL (4 Tapo C110 cameras - may be via Scrypted instead)
  - If using Scrypted for cameras, Tapo integration may not be needed in HA
- **Kasa:** REQUIRED (2 Kasa WiFi plugs)
- **Govee:** REQUIRED (2 Govee lights/strips)
- **HomeKit Bridge:** REQUIRED (exposes HA entities to HomeKit)
  - Current count: 22 devices (10 WiFi/sensor + sensors exposed)
- **HomeKit Controller:** Optional (if using native Hue/Aqara HomeKit without bridge)
- **Matter:** Status TBD (document if/when used)
- **Thread:** Status TBD (Apple TV hubs support Thread; document if Thread devices added)

### Integration Notes
- **Hue Bridge:** Prefer HA Hue integration over HomeKit Controller for Hue devices (more control, faster)
- **Aqara Hub:** Prefer HA Aqara integration over HomeKit Controller (local control, no cloud)
- **Cameras:** Use Scrypted for HomeKit exposure (not HA Tapo integration)

---

## Add-ons / Services (Current)

### Installed Add-ons
- TBD (document after verification)

### Recommended Add-ons (Phase 1)
- **File Editor** or **VS Code Server:** Configuration file editing
- **AdGuard Home:** DNS-based ad blocking (if using local DNS server)
- **Matter Server:** If using Matter devices (check inventory)
- **MQTT Broker:** If using MQTT devices (check integrations)

### Services to Verify
- **HomeKit Bridge:** Status (enabled/disabled)
- **Backup strategy:** Auto-backup add-on or manual backups

---

## Security / Access Control

### Authentication (Phase 1B Target)
- **MFA (Multi-Factor Authentication):** REQUIRED (enable in Phase 1B)
  - Admin user: MFA enabled
  - Daily user: Non-admin, MFA optional
- **Password Policy:** Strong passwords (min 12 characters, mix of upper/lower/numbers/symbols)

### User Accounts (Phase 1B Target)
- **Admin user:** Full access, MFA required, use only for configuration changes
- **Daily user:** Non-admin, for daily dashboard access and control
- **Service accounts:** TBD (if integrations require separate users)

### HomeKit Bridge Entity Exposure
- **Principle:** Expose only what humans interact with daily
- **Avoid exposing:**
  - Diagnostic sensors
  - Helper entities
  - System monitors
  - Rarely-used devices
- **Current exposed:** 22 devices (review and minimize in Phase 1B)

### Network Access
- **LAN only:** HA accessible from 192.168.1.0/24
- **No external access:** No port forwards, no Nabu Casa (unless explicitly added)
- **HTTPS:** TBD (document if SSL/TLS configured)

---

## Backups / Recovery

### Backup Strategy (Phase 1B Target)
- **Frequency:** Weekly automatic backups (or after major changes)
- **Location:** Local (HA storage) + offsite (PC, cloud storage)
- **Retention:** Keep last 7 days locally, last 4 weeks offsite
- **Last backup:** TBD (check in Phase 1B)

### UTM VM Snapshots
- **Frequency:** Before Phase 1 changes, before major HA updates
- **Location:** Mac mini local storage
- **Last snapshot:** TBD (verify in pre-flight)

### Recovery Plan
1. **HA restore:** Settings → System → Backups → Restore
2. **UTM VM restore:** Revert to snapshot in UTM app
3. **Full reinstall:** Rebuild HA VM from scratch (last resort)

---

## Integrations / Devices Reachability

### Network Zones
**Phase 1 setup:**
- **HA VM:** On Trusted network (192.168.1.242)
- **Hue Bridge:** Wired to router (192.168.1.169, reachable from HA)
- **Aqara Hub M2:** Wired to router (192.168.1.165, reachable from HA)
- **Kasa plugs:** On IoT SSID (ByteMe, reachable from HA)
- **Govee devices:** On IoT SSID (ByteMe, reachable from HA)
- **Tapo cameras:** On Cameras SSID (reachable from HA for integration, if needed)

**HA can reach all networks:** Single subnet (192.168.1.0/24) means HA can communicate with devices on any SSID.

### Discovery Dependencies
- **mDNS/Bonjour:** Required for HomeKit Bridge discovery by iOS devices
- **UPnP:** NOT required (disabled in Phase 1A for security)
- **SSDP:** May be required for some integrations (Govee, Kasa)

---

## Issues / Pain Points

### Current Known Issues (Pre-Phase 1)
- TBD (document after preflight checks)

### Potential Issues to Watch For
- **HomeKit discovery:** After Phase 1A SSID changes, verify HomeKit controllers can still discover HA Bridge
- **Integration timeouts:** If devices move SSIDs, integrations may temporarily lose connection
- **Camera streams:** Tapo integration may break after cameras move to isolated Cameras SSID (use Scrypted instead)

### Phase 1 Mitigations
- **Single subnet:** mDNS works across all SSIDs (no discovery issues expected)
- **HA on Trusted:** Can reach IoT and Cameras SSIDs for control
- **Static IPs:** Critical devices (Hue, Aqara, cameras) won't change IPs

---

## Performance / Monitoring

### Metrics to Track
- **Response time:** HomeKit toggle to device response (target: <1 second)
- **Uptime:** HA VM uptime (target: 99%+)
- **Integration health:** All integrations "Connected" (Settings → Devices & Services)
- **Disk usage:** Monitor HA VM disk space

### Baseline (Pre-Phase 1)
- TBD (document current response times and uptime in pre-flight)

### Post-Phase 1 Validation
- Compare response times to baseline
- Verify no degradation in performance

---

## Automation Strategy

### Current Automations
- TBD (document in pre-flight checklist, optional step 9)

### Automation Best Practices
- **Test automations:** After Phase 1, verify time-sensitive automations still work
- **Fallback plans:** If network-dependent automation fails, have manual control available
- **Logging:** Enable automation tracing for troubleshooting

---

## Next Steps

### Phase 1 Pre-Flight
- Document current HA integrations (Settings → Devices & Services)
- Screenshot HomeKit Bridge exposed entities
- Verify HA reachable and all integrations online

### Phase 1A (Network Layer)
- No direct HA changes (HA continues running on same IP)
- Verify HA remains accessible after SSID changes
- Test HomeKit Bridge still works after network changes

### Phase 1B (HA Hardening)
- Enable MFA for admin user
- Create non-admin daily user
- Review and minimize HomeKit Bridge exposed entities
- Create and download backup

---

**Last Updated:** 2026-01-19
