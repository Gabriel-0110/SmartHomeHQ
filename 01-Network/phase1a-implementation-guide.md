# Phase 1A: Network Foundation and Migration (Intentional Double NAT)

**Date:** 2026-01-19
**Estimated Time:** 2-3 hours
**Prerequisites:** `01-Network/phase1-preflight-checklist.md` complete; maintenance window scheduled

---

## Scope

This phase configures the physical topology, router settings, SSIDs and bands, DHCP scopes and reservations, device migration, camera internet blocking, and validation of the intentional double NAT design.

This phase does NOT include endpoint OS hardening or Home Assistant security changes (Phase 1B).

---

## Architecture Contract (Non-Negotiable)

- Two identical Verizon CR1000A routers.
- Router #2 stays in router mode (never AP mode).
- Router #2 WAN connects to Router #1 LAN (LAN -> WAN).
- Two subnets only:
  - Trusted: 192.168.1.0/24 (Router #1)
  - IoT/Cameras: 192.168.2.0/24 (Router #2)
- **NAT does NOT provide isolation by itself; it only reduces unsolicited inbound traffic.**
- Endpoint enforcement (host firewalls) is the primary isolation control.
- This architecture enables control and visibility, not magic isolation.
- Trusted can initiate outbound to IoT/Cameras; IoT can respond.
- Home Assistant + Scrypted live on the Mac mini (must be dual-homed; see below).
- HomeKit hubs stay on Router #1 (mDNS dependency).
- No VLANs, no AP mode, no mDNS reflection in Phase 1.

---

## CRITICAL: Mac mini Dual-Homing Requirement

**The Mac mini (hosting Home Assistant + Scrypted) MUST be dual-homed:**

- **Interface 1 (Trusted):** Connected to Router #1 (192.168.1.x)
  - Provides HomeKit hub access via mDNS
  - Provides HA UI access for trusted devices
- **Interface 2 (IoT):** Connected to Router #2 (192.168.2.x)
  - Enables HA/Scrypted to directly reach IoT devices and cameras on 192.168.2.x

**Without dual-homing, Home Assistant and Scrypted CANNOT reach devices on 192.168.2.x.**

Implementation options:
1. **USB-to-Ethernet adapter:** Plug into Mac mini, connect to Router #2 LAN port.
2. **Thunderbolt/USB-C dock with Ethernet:** Use the dock's Ethernet for the second interface.
3. **Wi-Fi as second interface:** Connect Mac mini Wi-Fi to ByteMe SSID (less reliable than wired).

Validation:
- Mac mini shows two active network interfaces with IPs on both subnets.
- From Mac mini: `ping 192.168.1.1` succeeds (Router #1).
- From Mac mini: `ping 192.168.2.1` succeeds (Router #2).

---

## Target Topology (Locked)

```
ISP
  -> Router #1 (Trusted / Control Plane) 192.168.1.1/24
     - LAN ports: Apple TV 4K, Mac mini Interface 1 (Trusted), Router #2 WAN
  -> Router #2 (IoT / Edge) WAN on 192.168.1.x
     - LAN: 192.168.2.1/24 for IoT and cameras
     - LAN ports: Mac mini Interface 2 (IoT), Hue Bridge, Aqara Hub
```

**Note:** Mac mini is dual-homed—one interface on each router.

---

## Target IP Plan (Locked)

### Router #1 (Trusted) - 192.168.1.0/24

- Gateway: 192.168.1.1
- DHCP dynamic pool: 192.168.1.100-192.168.1.254
- Reserved ranges:
  - 192.168.1.2-19: network infrastructure
  - 192.168.1.20-49: servers and core services
  - 192.168.1.50-99: trusted static devices

**Router #1 DHCP Reservations (set exactly):**

| Device | IP Address | Connection | Notes |
| --- | --- | --- | --- |
| Router #1 | 192.168.1.1 | - | Gateway |
| Router #2 WAN | 192.168.1.10 | DHCP reservation | Use 192.168.1.2-19; change only if 1.10 is in use |
| Mac mini M4 (Trusted Interface) | 192.168.1.20 | Ethernet | Interface 1 for HA + Scrypted (Trusted side) |
| Home Assistant VM | 192.168.1.21 | Virtual bridge | HA UI at `http://192.168.1.21:8123` |
| Apple TV 4K (Control Room) | 192.168.1.50 | Ethernet | HomeKit hub + Thread border |
| Apple TV (Bedroom) | 192.168.1.51 | Wi-Fi | HomeKit hub |
| iPad 5th Gen | 192.168.1.52 | Wi-Fi | HomeKit hub / wall panel |

### Router #2 (IoT/Cameras) - 192.168.2.0/24

- Gateway: 192.168.2.1
- DHCP dynamic pool: 192.168.2.200-192.168.2.254
- Reserved ranges:
  - 192.168.2.2-49: infrastructure
  - 192.168.2.50-149: IoT devices
  - 192.168.2.150-199: cameras

**Router #2 DHCP Reservations (set exactly):**

| Device | IP Address | Connection | Notes |
| --- | --- | --- | --- |
| Router #2 | 192.168.2.1 | - | Gateway |
| Mac mini M4 (IoT Interface) | 192.168.2.10 | Ethernet (USB adapter) | Interface 2 for HA + Scrypted (IoT side) |
| Philips Hue Bridge | 192.168.2.50 | Ethernet or Wi-Fi | IoT hub |
| Aqara Hub M2 | 192.168.2.51 | Ethernet | IoT hub |
| Echo Show 8 (Bedroom) | 192.168.2.52 | Wi-Fi | IoT |
| Echo Dot (Office) | 192.168.2.53 | Wi-Fi | IoT |
| Nest Mini (Bathroom) | 192.168.2.54 | Wi-Fi | IoT |
| HP Printer | 192.168.2.55 | Wi-Fi | IoT |
| Toshiba Microwave | 192.168.2.56 | Wi-Fi | IoT |
| Kitchen Camera (Tapo C110) | 192.168.2.150 | Wi-Fi (Cameras SSID) | Internet blocked |
| Hallway Camera (Tapo C110) | 192.168.2.151 | Wi-Fi (Cameras SSID) | Internet blocked |
| Entrance Camera (Tapo C110) | 192.168.2.152 | Wi-Fi (Cameras SSID) | Internet blocked |
| Office Camera (Tapo C110) | 192.168.2.153 | Wi-Fi (Cameras SSID) | Internet blocked |

---

## Step 0: Pre-Change Snapshots (Required)

Capture screenshots for rollback. Save to `01-Network/screenshots/preflight/`.

Router #1 pages:
- Advanced -> Network Settings -> IPv4 Address Distribution (summary and DHCP Settings)
- Advanced -> Network Settings -> DHCP Connections
- Basic -> Wi-Fi -> Primary Network (EnterAtYourOwnRisk)
- Basic -> Wi-Fi -> Guest/IoT networks (if present)
- Basic -> Wi-Fi -> Radio Management (Band Steering/SON)
- Basic -> Wi-Fi -> WPS
- Advanced -> Security & Firewall -> Port Forwarding
- Advanced -> Security & Firewall -> DMZ Host
- Advanced -> Security & Firewall -> UPnP
- Advanced -> Network Settings -> IPv6

Router #2 pages (access from Router #2 LAN or ByteMe):
- Advanced -> Network Settings -> WAN Connection
- Advanced -> Network Settings -> LAN Settings
- Advanced -> Network Settings -> IPv4 Address Distribution (summary and DHCP Settings)
- Advanced -> Network Settings -> DHCP Connections
- Basic -> Wi-Fi -> SSIDs (ByteMe, Cameras)
- Basic -> Wi-Fi -> Radio Management (Band Steering/SON)
- Basic -> Wi-Fi -> WPS
- Advanced -> Security & Firewall -> UPnP
- Advanced -> Security & Firewall -> Port Forwarding
- Advanced -> Security & Firewall -> DMZ Host

Validation:
- Screenshots exist and are readable.
- You can still log into both routers.

Rollback:
- Use these screenshots to restore any setting.

---

## Step 1: Confirm Physical Wiring (LAN -> WAN)

Goal: Router #2 WAN is connected to Router #1 LAN.

- Router #1 WAN -> ISP
- Router #1 LAN -> Router #2 WAN
- Router #2 LAN -> IoT hubs (Aqara, Hue) and any IoT Ethernet devices

Validation:
- Router #2 WAN receives a 192.168.1.x address from Router #1.
- Router #2 stays in router mode (not AP mode).

Rollback:
- Reconnect cables to prior state and reboot Router #2.

---

## Step 2: Configure Router #1 (Trusted)

Access: `http://192.168.1.1`

### 2.1 LAN and DHCP

- LAN IP: `192.168.1.1`
- Subnet mask: `255.255.255.0`
- DHCP enabled
- DHCP pool: `192.168.1.100-192.168.1.254`

Validation:
- Trusted device renews DHCP and gets 192.168.1.x with gateway 192.168.1.1.

Rollback:
- Restore previous DHCP range from snapshots.

### 2.2 Wi-Fi (Trusted SSID)

SSID: `EnterAtYourOwnRisk`

- Bands: 5 GHz and 6 GHz enabled
- 2.4 GHz: disabled unless a trusted device requires it
- Band Steering/SON: disabled
- Security: WPA2/WPA3 or WPA2

Validation:
- Trusted device connects to EnterAtYourOwnRisk and receives 192.168.1.x.

Rollback:
- Restore prior Wi-Fi settings from snapshots.

### 2.3 Router Hardening (Trusted)

- WPS: disabled
- UPnP: disabled
- Port forwarding: empty
- DMZ: disabled
- Remote admin: disabled
- IPv6: set intentionally (leave enabled only if you use it)

Validation:
- Settings reflect the above.

Rollback:
- Restore prior settings from snapshots.

---

## Step 3: Configure Router #2 (IoT/Cameras)

Access: `http://192.168.2.1` (must be on Router #2 LAN or ByteMe SSID)

### 3.1 LAN and DHCP

- Router mode: required
- LAN IP: `192.168.2.1`
- Subnet mask: `255.255.255.0`
- DHCP enabled
- DHCP pool: `192.168.2.200-192.168.2.254`

Validation:
- Device on Router #2 gets 192.168.2.x with gateway 192.168.2.1.

Rollback:
- Restore prior LAN IP and DHCP range from snapshots.

### 3.2 Wi-Fi (IoT + Cameras)

SSID: `ByteMe` (IoT)
- 2.4 GHz only
- Band Steering/SON disabled
- Security: WPA2

SSID: `Cameras`
- 2.4 GHz only
- Band Steering/SON disabled
- Security: WPA2

Validation:
- IoT device connects to ByteMe and receives 192.168.2.x.
- Camera connects to Cameras and receives 192.168.2.x.

Rollback:
- Restore prior SSID settings from snapshots.

### 3.3 Router Hardening (IoT)

- WPS: disabled
- UPnP: disabled
- Port forwarding: empty
- DMZ: disabled
- Remote admin: disabled
- IPv6: set intentionally (leave enabled only if you use it)

Validation:
- Settings reflect the above.

Rollback:
- Restore prior settings from snapshots.

---

## Step 4: Create DHCP Reservations on Router #1 (Trusted)

Access: `http://192.168.1.1` -> DHCP Connections

- Reserve each device to match the Router #1 table above.
- Reserve Router #2 WAN to 192.168.1.10 (or another unused IP in 192.168.1.2-19).

Validation:
- Reserved devices show correct IPs.
- Router #2 WAN shows the reserved 192.168.1.10.

Rollback:
- Remove reservations to return to dynamic DHCP.

---

## Step 5: Create DHCP Reservations on Router #2 (IoT/Cameras)

Access: `http://192.168.2.1` -> DHCP Connections

- Reserve each IoT and camera device to match the Router #2 table above.

Validation:
- Reserved devices show correct IPs.
- Cameras show 192.168.2.150-153.

Rollback:
- Remove reservations to return to dynamic DHCP.

---

## Step 6: Migrate Devices to Correct Subnets

### 6.1 Trusted devices -> Router #1 (Trusted)

Keep these on Router #1:
- Mac mini (HA + Scrypted host)
- Home Assistant VM
- HomeKit hubs (Apple TVs, iPad)
- Phones, PCs, tablets

Validation:
- All trusted devices are on 192.168.1.x.
- Home Assistant loads at `http://192.168.1.21:8123`.

### 6.2 IoT devices -> Router #2 (ByteMe)

For each IoT device:
1. Move device Wi-Fi to `ByteMe` (or wire it to Router #2 LAN).
2. Verify it appears in Router #2 DHCP Connections.
3. Create reservation from the Router #2 table.
4. Reboot device or renew DHCP.
5. Confirm it returns to the reserved IP.
6. Verify it works in its native app and in Home Assistant.

Validation:
- Device is on 192.168.2.x with gateway 192.168.2.1.
- HA controls the device successfully.

### 6.3 Cameras -> Router #2 (Cameras SSID)

For each camera:
1. Move camera Wi-Fi to `Cameras` SSID.
2. Create reservation from the Router #2 table (192.168.2.150-153).
3. Update camera IPs in Scrypted if required.
4. Verify live stream in Scrypted.

Validation:
- Camera is on 192.168.2.150-153.
- Scrypted streams the camera locally.

Rollback (device migration):
- Move device back to Router #1 SSID or previous wiring.
- Remove its Router #2 reservation.
- Restore IPs in HA/Scrypted if changed.

---

## Step 7: Block Camera Internet Access (Router #2)

Goal: block WAN access for cameras while keeping LAN access for Scrypted.

1. Router #2 -> Parental Controls or Access Control
2. Create profile: `Cameras`
3. Set Internet/WAN access: blocked (always on)
4. Assign all camera devices (by MAC/IP) to the `Cameras` profile

Validation:
- Cameras still stream in Scrypted.
- Camera cloud access fails (expected).

Rollback:
- Remove cameras from the profile or delete the profile.

---

## Step 8: Validation Tests (Required)

### 8.1 Double NAT and Gateway Verification

From a trusted device (Router #1 LAN):
```bash
netstat -nr | grep default
```
Expected: default gateway is 192.168.1.1.

From an IoT device on Router #2 (any device with a shell, or a laptop on ByteMe):
```bash
netstat -nr | grep default
```
Expected: default gateway is 192.168.2.1.

### 8.2 NAT Behavior Test (OPTIONAL - Informational Only)

**Note:** NAT does NOT guarantee isolation. This test is informational only.

From an IoT device on Router #2:
```bash
ping 192.168.1.1
ping 192.168.1.20
```
Expected behavior varies:
- Pings to 192.168.1.1 (Router #1) may succeed or fail depending on router config.
- Pings to 192.168.1.20 (Mac mini Trusted interface) may succeed if the device can route.

**True isolation requires endpoint firewall rules on trusted devices (see Phase 1B).**
Do not rely on this test to confirm security.

### 8.3 Mac mini Dual-Homing Validation (REQUIRED)

From the Mac mini (which must be dual-homed):
```bash
# Verify both interfaces are active
ifconfig | grep "inet "
# Should show both 192.168.1.20 and 192.168.2.10

# Test connectivity to both subnets
ping 192.168.1.1    # Router #1 via Trusted interface
ping 192.168.2.1    # Router #2 via IoT interface
ping 192.168.2.50   # Hue Bridge via IoT interface
ping 192.168.2.150  # Kitchen Camera via IoT interface
```
Expected: all succeed.

**If pings to 192.168.2.x fail:** The Mac mini IoT interface is not properly connected or configured.

### 8.4 HA + Scrypted Checks

- Home Assistant: `http://192.168.1.21:8123` loads and IoT integrations show available.
- Scrypted: `https://192.168.1.20:10443` shows all cameras streaming.

### 8.5 HomeKit Check

- Open the Home app and toggle a HomeKit accessory.
- Expected: response in 1-2 seconds.

### 8.6 Camera Internet Block

- Camera cloud access should fail.
- Local Scrypted streams remain available.

---

## Rollback Plan (Use If Critical Devices Fail)

### Rollback: Router #2 LAN/DHCP
- Restore LAN IP and DHCP range from snapshots.
- Reboot Router #2.

### Rollback: Router #1 DHCP
- Restore DHCP range from snapshots.

### Rollback: SSIDs and Bands
- Restore SSID names, bands, and passwords from snapshots.

### Rollback: DHCP Reservations
- Remove reservations on the affected router.

### Rollback: Device Migration
- Move device back to previous SSID or wiring.
- Restore previous IP settings in HA/Scrypted if needed.

### Rollback: Camera Internet Blocking
- Remove cameras from Access Control profile.

---

## Phase 1A Completion Checklist

- [ ] Pre-change screenshots saved to `01-Network/screenshots/preflight/`
- [ ] Router #2 wired LAN -> WAN (Router #1 LAN -> Router #2 WAN)
- [ ] Router #1 LAN and DHCP set to 192.168.1.0/24 with pool 192.168.1.100-254
- [ ] Router #2 LAN and DHCP set to 192.168.2.0/24 with pool 192.168.2.200-254
- [ ] **Mac mini dual-homed: Interface on Router #1 (192.168.1.20) AND Router #2 (192.168.2.10)**
- [ ] Router #1 reservations created (Mac mini Trusted interface, HA, hubs, Router #2 WAN)
- [ ] Router #2 reservations created (Mac mini IoT interface, IoT devices, cameras)
- [ ] SSIDs set: EnterAtYourOwnRisk (trusted), ByteMe (IoT 2.4 GHz), Cameras (2.4 GHz)
- [ ] WPS, UPnP, remote admin disabled on both routers
- [ ] Cameras moved to Cameras SSID and streaming in Scrypted
- [ ] Cameras internet blocked via Access Control
- [ ] IoT devices moved to ByteMe or Router #2 LAN
- [ ] Mac mini can reach both 192.168.1.x and 192.168.2.x devices
- [ ] HA and Scrypted reachable from Trusted network
- [ ] HA/Scrypted can communicate with IoT devices on 192.168.2.x

---

**Next:** Proceed to `01-Network/phase1b-implementation-guide.md`.
