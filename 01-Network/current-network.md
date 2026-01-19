# Current Network Baseline (Canonical)

## Router / Internet
- ISP router: Verizon CR1000A
- Wi-Fi bands: 2.4 / 5 / 6 GHz (tri-band capable)
- UPnP: TBD (goal: OFF)
- Port forwards: TBD (goal: none unless justified)
- DNS: TBD
- Admin access: Verizon app or router web interface

## SSIDs (Current)
- **Primary SSID**: "EnterAtYourOwnRisk" (2.4 / 5 / 6 GHz)
  - Current usage: Trusted devices (phones, PCs, Apple TV, Mac mini, iPad)
- **Guest SSID**: "HotNeighborsCanConnect" (2.4 GHz)
  - Current usage: Guest network
- **IoT SSID**: "ByteMe" (2.4 GHz)
  - Current usage: IoT devices (plugs, lights, cameras, hubs)

**Notes**:
- 3 SSIDs already configured (good starting point)
- Need to verify device placement across SSIDs
- Need to add firewall rules for proper isolation
- Camera SSID may need to be created OR cameras moved to dedicated VLAN

## IP Plan (Current)
- LAN range: 192.168.1.0/24 (confirmed)
- Router gateway: 192.168.1.1 (confirmed)
- DHCP range: TBD (need to check router config)
- Reserved/static IPs:
  - **Home Assistant VM**: 192.168.1.242
  - **Scrypted host**: 192.168.1.224 (primary), also seen: .209, .171 (investigate multiple IPs)
  - **Homebridge host**: TBD (if used)
  - **Kitchen Camera (Tapo C110)**: 192.168.1.210
  - **Hallway Camera (Tapo C110)**: 192.168.1.236
  - **Entrance Camera (Tapo C110)**: 192.168.1.155
  - **Office Camera (Tapo C110)**: 192.168.1.204
  - **Hue Bridge**: 192.168.1.169
  - **Aqara Hub M2**: 192.168.1.165

**Note**: Scrypted has multiple IPs listed (.224, .209, .171) - need to verify which is primary and consolidate.

## Segmentation Capabilities (Current)
- **VLAN support**: TBD (CR1000A capabilities to be researched)
- **Guest/IoT isolation**: Partial (3 SSIDs exist, but firewall rules unknown)
- **Client isolation toggle**: TBD
- **mDNS/Bonjour across networks**: TBD (critical for HomeKit functionality)
- **Firewall rules**: TBD (need to audit current configuration)

**Research Needed**:
- CR1000A VLAN capabilities (via Verizon app or web interface)
- Current firewall rules between SSIDs
- Whether cameras are isolated from Trusted network
- mDNS reflector/repeater support for HomeKit across networks

## Known Issues / Constraints
- Discovery issues (mDNS/Bonjour): TBD
- Camera stream stability: TBD
- Anything else: TBD

## Target Direction (1-paragraph)
We want 3 logical zones:
1) Trusted (phones/PCs/Apple TV/Mac mini/HA)
2) IoT (plugs/lights/panels/assistants)
3) Cameras (cameras only)
Policy: IoT/Cameras cannot initiate traffic to Trusted; HA can initiate to both; preserve HomeKit discovery.