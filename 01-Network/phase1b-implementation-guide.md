# Phase 1B: Endpoint and Platform Hardening

**Date:** 2026-01-19
**Estimated Time:** 1-1.5 hours
**Prerequisites:** Phase 1A complete

---

## Scope

This phase hardens Home Assistant and trusted endpoints. Endpoint firewalls are the primary isolation control in this architecture.

This phase does NOT reconfigure routers, move devices, or change IP plans.

---

## Architecture Context (Fixed)

- Two Verizon CR1000A routers in intentional double NAT.
- Router #2 stays in router mode; WAN connected to Router #1 LAN.
- Trusted subnet: 192.168.1.0/24 (Router #1).
- IoT/Cameras subnet: 192.168.2.0/24 (Router #2).
- **NAT is NOT a security boundary; it only reduces unsolicited inbound traffic.**
- **Endpoint firewalls are the primary isolation control.**
- There is NO routing between subnets; the Mac mini is dual-homed to reach both.
- Home Assistant + Scrypted on Mac mini (dual-homed: 192.168.1.20 + 192.168.2.10).
- HomeKit hubs on Router #1.

---

## Step 1: Home Assistant Hardening (20 minutes)

### 1.1 Enable Multi-Factor Authentication (MFA)
Access Home Assistant: `http://192.168.1.21:8123`

1. Settings -> People
2. Select your admin user
3. Multi-factor Authentication -> Add -> Authenticator app
4. Scan QR code, enter verification code
5. Save backup codes in a safe place
6. Finish

Validation: Log out and log back in; MFA is required.

### 1.2 Create Non-Admin Daily User
1. Settings -> People -> Add Person
2. Name and username (for daily use)
3. Admin: unchecked
4. Create

Validation: User appears without Admin badge.

### 1.3 Review HomeKit Bridge Exposed Entities
1. Settings -> Devices & Services -> HomeKit Bridge -> Configure/Entities
2. Remove anything not needed in HomeKit (diagnostics, helpers, unused devices)

Validation: Only essential entities are exposed.

### 1.4 Verify Backup Strategy
1. Settings -> System -> Backups
2. Create a backup if none exists this week
3. Download backup to a safe location
4. Confirm a recurring backup method is in place (manual weekly or add-on)

Validation: Recent backup exists and is downloaded.

---

## Step 2: macOS Firewall and Host Security (15 minutes)

### 2.1 Enable macOS Firewall
1. System Settings -> Network -> Firewall (or Security & Privacy -> Firewall)
2. Turn Firewall on
3. Firewall Options:
   - Ensure "Block all incoming connections" is OFF
   - Allow UTM and Scrypted if prompted
   - Enable Stealth Mode

Validation:
- HA is reachable at `http://192.168.1.21:8123`
- Scrypted is reachable at `https://192.168.1.20:10443`

### 2.2 Disable Unnecessary Sharing Services
1. System Settings -> General -> Sharing
2. Disable services you do not use (File Sharing, Screen Sharing, Remote Management, etc.)
3. Keep only the services you actively need

Validation: Only required sharing services remain enabled.

---

## Step 3: Windows Firewall (15 minutes, if applicable)

### 3.1 Verify Firewall Enabled
1. Settings -> Privacy & Security -> Windows Security
2. Firewall & network protection
3. Ensure Firewall is on for Domain, Private, and Public profiles

### 3.2 Ensure Network Profile is Private
1. Settings -> Network & Internet -> Wi-Fi/Ethernet
2. Select your connection
3. Network profile: Private

### 3.3 Optional: Explicit Inbound Block from IoT Subnet
This rule is optional defense-in-depth. **NAT alone is insufficient for isolation**—endpoint firewalls provide actual enforcement.

Open PowerShell as Administrator:

```powershell
New-NetFirewallRule -DisplayName "Block IoT Subnet" `
  -Direction Inbound `
  -RemoteAddress 192.168.2.0/24 `
  -Action Block `
  -Enabled True
```

Validation:
- Firewall enabled
- Network profile Private
- Optional rule present (if created)

---

## Defense-in-Depth Note

- **NAT is NOT a security boundary.** It reduces unsolicited inbound traffic but does not enforce isolation.
- **Endpoint firewalls are the primary isolation control** for trusted devices.
- The dual-router architecture enables visibility and control, not magic isolation.
- HA/Scrypted reach IoT devices because the Mac mini is dual-homed—not via routing.
- Do not change router mode as part of Phase 1B.

---

## Step 4: Validation Tests (Required)

### 4.1 IoT -> Trusted Behavior (OPTIONAL - Informational Only)

**Note:** NAT does NOT guarantee isolation. This test is informational only.

From a device on Router #2 (ByteMe or Cameras SSID), attempt:

```bash
ping 192.168.1.1
ping 192.168.1.20
```

Expected behavior varies:
- Pings may succeed or fail depending on router and endpoint firewall configuration.
- **Do not rely on this test to confirm security.**
- True isolation requires endpoint firewall rules (configured in this phase).

### 4.2 Mac mini Dual-Homing Validation (REQUIRED)

From the Mac mini (which is dual-homed on both subnets):

```bash
ping 192.168.2.50   # Hue Bridge via IoT interface
ping 192.168.2.150  # Kitchen Camera via IoT interface
```

Expected: both succeed.

**Note:** The Mac mini reaches 192.168.2.x devices via its IoT interface (192.168.2.10), NOT via routing. Other trusted devices on Router #1 cannot reach 192.168.2.x directly.

### 4.3 Home Assistant and Scrypted

- Home Assistant loads and IoT integrations show available.
- Scrypted shows all camera streams.

### 4.4 HomeKit Check
- Toggle a HomeKit accessory in the Home app.
- Expected: response in 1-2 seconds.

### 4.5 Firewall Status Checks

macOS (Terminal):
```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```
Expected: Firewall is enabled.

Windows (PowerShell):
```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled
```
Expected: all profiles Enabled = True.

---

## Rollback Plan

### Home Assistant
- Remove MFA if needed: Settings -> People -> your user -> MFA -> Remove
- Delete non-admin user if it causes confusion
- Restore from the latest backup if required

### macOS Firewall
```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode off
```

### Windows Firewall (optional rule)
```powershell
Remove-NetFirewallRule -DisplayName "Block IoT Subnet"
```

---

## Phase 1B Completion Checklist

- [ ] MFA enabled for Home Assistant admin user
- [ ] Non-admin daily user created
- [ ] HomeKit Bridge entities reviewed and minimized
- [ ] Recent Home Assistant backup created and downloaded
- [ ] macOS firewall enabled and sharing services trimmed
- [ ] Windows Firewall enabled (if applicable)
- [ ] Optional Windows IoT inbound block rule created (if desired)
- [ ] Mac mini can reach IoT devices via dual-homed interface (192.168.2.10)
- [ ] Endpoint firewalls configured on trusted devices
- [ ] Home Assistant and Scrypted access verified
- [ ] HomeKit control verified

---

**Phase 1B complete.**
