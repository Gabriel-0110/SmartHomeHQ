# SmartHomeHQ — Requirements

## Hard requirements
- HomeKit as primary UX
- Home Assistant as hub (UTM VM on Mac mini M4)
- Homebridge for niche integrations only
- Scrypted for cameras + HKSV
- Strong organization: Areas, Labels, Zones
- Modern centralized dashboard

## Non-goals
- This is not an “app repo”
- No secrets committed
- Avoid double-bridging devices

## Success criteria
- HomeKit shows only clean, stable, human-friendly devices
- HA owns automation logic and exposes only what matters
- Cameras are isolated and stable in Scrypted/HKSV
- Network segmentation exists (VLANs or equivalent)
- Repeatable onboarding process for new devices
