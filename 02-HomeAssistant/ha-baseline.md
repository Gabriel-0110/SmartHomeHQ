# Home Assistant Baseline (Canonical)

## Host + VM
- Host: Mac mini M4
- Virtualization: UTM
- HA install type: TBD (HA OS / Supervised / Container)
- VM networking: TBD (Bridged is preferred for discovery; note what you use)
- VM resources:
  - CPU cores: TBD
  - RAM: TBD
  - Disk: TBD
- Timezone: America/New_York

## Home Stack Requirements (Fixed)
- HomeKit is the user experience
- Home Assistant is the automation brain
- Homebridge is allowed for niche integrations only
- Scrypted is the camera hub (HKSV)

## Integrations (Current)
Based on device inventory, the following integrations are required:
- Hue: REQUIRED (7 Hue lights via Hue Bridge)
- Aqara: REQUIRED (11 Aqara devices via Hub M2)
- Tapo: REQUIRED (4 Tapo C110 cameras - may be via Scrypted instead)
- Kasa: REQUIRED (2 Kasa WiFi plugs)
- Govee: REQUIRED (2 Govee lights/strips)
- HomeKit Bridge: REQUIRED (exposes 10 WiFi/sensor devices to HomeKit)
- HomeKit Controller: Optional (if using native Hue/Aqara HomeKit)
- Matter: Status TBD
- Thread: Status TBD (Apple TV hubs support Thread)

## Add-ons / Services (Current)
- File editor / VS Code Server: TBD
- Matter Server: TBD
- MQTT: TBD
- Any others: TBD

## Backups / Recovery
- HA backups: TBD (local + offsite preferred)
- UTM snapshots: TBD

## Issues / Pain Points
- List current problems (discovery, duplicates, camera streams, delays, etc.)
- Example: iOS app connectivity, domain/HTTPS, RTSP rebroadcast logs, etc.