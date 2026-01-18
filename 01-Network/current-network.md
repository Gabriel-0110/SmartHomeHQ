# Current Network Baseline (Canonical)

## Router / Internet
- ISP router: Verizon router (model: TBD)
- Wi-Fi bands: 2.4 / 5 / 6 GHz
- UPnP: TBD (goal: OFF)
- Port forwards: TBD (goal: none unless justified)
- DNS: TBD

## SSIDs
- Trusted SSID: TBD
- IoT SSID: TBD
- Camera SSID: TBD
Notes:
- If only one SSID exists today, list it above as Trusted and mark IoT/Camera as “Not yet”.

## IP Plan (Current)
- LAN range: 192.168.1.0/24 (assumed; confirm)
- Router gateway: 192.168.1.1 (assumed; confirm)
- DHCP range: TBD
- Reserved/static IPs:
  - Home Assistant VM: TBD
  - Scrypted host: TBD
  - Homebridge host: TBD
  - Cameras (Tapo C110): 192.168.1.205 (known), others TBD

## Segmentation Capabilities (Current)
- VLAN support: TBD
- Guest/IoT isolation: TBD
- Client isolation toggle: TBD
- mDNS/Bonjour across networks: TBD

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