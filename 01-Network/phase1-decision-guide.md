# Phase 1: Network Segmentation — Decision Guide

**Date:** 2026-01-18
**Your Goals:**
- ✅ Router-level AdBlock (block tracking, ads, malware domains)
- ✅ Privacy: stop devices from tracking/leaking data
- ✅ Security: isolate IoT devices from trusted devices
- ✅ Stability: prevent devices from randomly switching networks
- ✅ Speed: maintain or improve network performance

---

## 🔴 Critical Decisions You Must Make

### Decision 1: DNS Strategy (AdBlock + Privacy)

**Question:** How do you want to handle DNS-based ad blocking and privacy?

**Options:**

#### Option A: Pi-hole on Separate Device (RECOMMENDED)
- **What:** Install Pi-hole on a Raspberry Pi, spare PC, or Docker container
- **Pros:**
  - Best performance and reliability
  - Full control over blocklists (ads, trackers, telemetry, adult content, etc.)
  - Detailed query logs (see what each device is accessing)
  - Can use it as DHCP server (more control)
  - Can handle 100+ devices easily
- **Cons:**
  - Requires additional hardware ($35 for Pi Zero 2 W, or free if using existing PC/Mac mini)
  - 1-2 hours setup time
- **Your setup:** Could run Pi-hole in Docker on Mac mini M4 (same host as HA)

#### Option B: Router-Based DNS Filtering (NextDNS or Cloudflare for Families)
- **What:** Set CR1000A DNS servers to NextDNS or Cloudflare 1.1.1.2
- **Pros:**
  - Zero additional hardware
  - 5 minutes setup
  - Free tier available (NextDNS: 300k queries/month)
- **Cons:**
  - Less control than Pi-hole
  - Depends on external service (privacy trade-off)
  - Can't see per-device query logs easily
  - Verizon router may not support custom DNS (needs testing)

#### Option C: Hybrid (Pi-hole + Upstream Privacy DNS)
- **What:** Pi-hole on Mac mini → upstream to NextDNS or Cloudflare DoH/DoT
- **Pros:**
  - Best of both worlds (local filtering + encrypted DNS)
  - Maximum privacy and ad blocking
- **Cons:**
  - Most complex setup (30-45 minutes)

**❓ YOUR CHOICE:**
[ ] Option A: Pi-hole (Docker on Mac mini)
[ ] Option B: Router DNS (NextDNS or Cloudflare)
[ ] Option C: Hybrid (Pi-hole + encrypted upstream)

**Recommendation:** **Option A** — Run Pi-hole in Docker on your Mac mini M4. You already have the hardware, and it gives you full visibility into what your IoT devices are calling home to.

---

### Decision 2: Network Segmentation Architecture

**Question:** How many logical networks do you want?

**Current State:** You have 3 SSIDs but no firewall rules (devices can still talk to each other).

**Options:**

#### Option A: 3 Networks (Simple, RECOMMENDED for CR1000A)
```
1. Trusted (EnterAtYourOwnRisk)
   - Phones, PCs, iPad, Apple TV, Mac mini
   - Can access everything

2. IoT (ByteMe)
   - Plugs, lights, Hue Bridge, Aqara Hub, Echo devices
   - Can initiate to internet
   - Cannot initiate to Trusted
   - HA can reach IoT

3. Cameras (NEW or rename HotNeighborsCanConnect)
   - 4x Tapo cameras
   - Can reach Scrypted only
   - Cannot reach internet (local-only)
   - HA can reach cameras
```

#### Option B: 4 Networks (Advanced, requires VLAN support)
```
1. Trusted (as above)
2. IoT (as above)
3. Cameras (as above)
4. Guest (isolated, internet-only)
```

#### Option C: 2 Networks (Minimal, not recommended for your goals)
```
1. Trusted (everything you trust)
2. Untrusted (IoT + cameras + guests)
```

**Firewall Rules (for Option A):**
```
✅ Trusted → IoT (allowed, for control)
✅ Trusted → Cameras (allowed, for viewing)
✅ IoT → Internet (allowed, for updates/cloud)
✅ Cameras → Scrypted only (local RTSP)
❌ IoT → Trusted (blocked)
❌ Cameras → Internet (blocked, privacy)
❌ Cameras → Trusted (blocked, except Scrypted)
```

**❓ YOUR CHOICE:**
[ ] Option A: 3 Networks (Trusted, IoT, Cameras)
[ ] Option B: 4 Networks (add dedicated Guest)
[ ] Option C: 2 Networks (simpler but less secure)

**Recommendation:** **Option A** — 3 networks is the sweet spot for your setup. Guest network can be added later if needed.

---

### Decision 3: Prevent Devices from Switching Networks

**Problem:** Your devices are randomly switching between SSIDs.

**Root Cause:** Likely due to:
- Same SSID name broadcast on multiple bands (2.4/5/6 GHz)
- Band steering enabled (router auto-moves devices)
- Devices roaming to "stronger" signal

**Solutions:**

#### Fix 1: Disable Band Steering
- **Action:** In CR1000A settings, turn OFF "Band Steering" or "Smart Connect"
- **Result:** Devices stay on the SSID you assign them to

#### Fix 2: Use Different SSID Names per Band
- **Current:** "EnterAtYourOwnRisk" (2.4/5/6 GHz all same name)
- **Change to:**
  - `Trusted` (5/6 GHz only, for fast devices)
  - `Trusted-Legacy` (2.4 GHz only, for slow IoT devices that need it)
  - `IoT` (2.4 GHz only)
  - `Cameras` (2.4 GHz only)

#### Fix 3: Set Static SSID per Device
- **Action:** Manually configure each device to connect to specific SSID
- **How:** Forget all networks on device, reconnect to only the correct one

#### Fix 4: Use MAC Address Filtering (Optional, Advanced)
- **Action:** Bind specific devices to specific SSIDs via MAC filtering
- **Pros:** Devices can't "roam" even if they try
- **Cons:** Maintenance overhead (need to update router config for every new device)

**❓ YOUR CHOICES:**
[ ] Fix 1: Disable Band Steering (REQUIRED)
[ ] Fix 2: Rename SSIDs per band (RECOMMENDED)
[ ] Fix 3: Manually set SSID on each device (REQUIRED)
[ ] Fix 4: MAC filtering (OPTIONAL, for extra control)

**Recommendation:** Do Fix 1 + Fix 2 + Fix 3. Skip Fix 4 unless you want maximum control.

---

### Decision 4: HomeKit mDNS/Bonjour Strategy

**Problem:** HomeKit uses mDNS/Bonjour for discovery. If you isolate networks, HomeKit may break.

**Question:** How do you want to preserve HomeKit functionality across networks?

**Options:**

#### Option A: Allow mDNS Reflection on Router
- **What:** Enable "mDNS Repeater" or "Avahi Reflector" on CR1000A
- **Pros:** HomeKit works seamlessly across all networks
- **Cons:** CR1000A may not support this (need to check)

#### Option B: Let Home Assistant Bridge Everything
- **What:** All devices go through HA → HomeKit Bridge → Your devices
- **Pros:** HA is on Trusted network, can reach IoT/Cameras, no mDNS issues
- **Cons:** Slightly slower response time (extra hop)

#### Option C: Selective mDNS Rules
- **What:** Allow mDNS (UDP 5353) from IoT → Trusted (one-way)
- **Pros:** HomeKit works, but IoT still can't initiate TCP connections
- **Cons:** Requires advanced firewall rules (CR1000A may not support)

#### Option D: Keep HomeKit Hubs on IoT Network
- **What:** Move Apple TV, Aqara Hub, Hue Bridge to IoT network
- **Pros:** All HomeKit devices are on same network (no mDNS issues)
- **Cons:** Your Apple TV is now on less-trusted network (not ideal)

**❓ YOUR CHOICE:**
[ ] Option A: mDNS reflection (check if CR1000A supports it)
[ ] Option B: HomeKit Bridge via HA (SAFEST, RECOMMENDED)
[ ] Option C: Selective mDNS firewall rules
[ ] Option D: Move hubs to IoT network (not recommended)

**Recommendation:** **Option B** — Let HA handle all HomeKit exposure. This is already your strategy per CLAUDE.md.

---

### Decision 5: Camera Isolation Level

**Question:** How isolated should your cameras be?

**Your 4 Tapo C110 cameras currently on IoT network (ByteMe).

**Options:**

#### Option A: Full Isolation (RECOMMENDED)
- **Config:**
  - Cameras on dedicated SSID ("Cameras" network)
  - Firewall: Block cameras from internet entirely
  - Firewall: Allow cameras → Scrypted only (192.168.1.224)
  - Scrypted pulls RTSP streams locally
- **Pros:**
  - Maximum privacy (cameras can't phone home to Tapo cloud)
  - Maximum security (cameras can't be hijacked to attack other devices)
- **Cons:**
  - Can't use Tapo mobile app to configure cameras (must use web UI via Scrypted IP access)
  - Can't receive camera firmware updates (must do manually via SD card or allow temporarily)

#### Option B: Internet Access for Updates Only
- **Config:**
  - Cameras on dedicated SSID
  - Firewall: Allow cameras → Tapo cloud for updates
  - Firewall: Block cameras → Trusted network
- **Pros:**
  - Cameras can update automatically
  - Still protected from attacking your trusted devices
- **Cons:**
  - Cameras can still phone home (privacy leak)

#### Option C: Keep Cameras on IoT Network
- **Config:**
  - Cameras stay on ByteMe (same as other IoT devices)
  - Share firewall rules with smart plugs, lights, etc.
- **Pros:**
  - Simplest config (no extra network)
- **Cons:**
  - Cameras are same threat level as light bulbs (not ideal for security cameras)

**❓ YOUR CHOICE:**
[ ] Option A: Full isolation, no internet (RECOMMENDED)
[ ] Option B: Allow Tapo cloud for updates
[ ] Option C: Keep on IoT network with other devices

**Recommendation:** **Option A** — Cameras should be your most locked-down devices. Use Scrypted for all camera access, block Tapo cloud entirely.

---

### Decision 6: Guest Network Strategy

**Question:** What do you want to do with "HotNeighborsCanConnect" SSID?

**Options:**

#### Option A: Repurpose as Cameras Network (RECOMMENDED)
- **Action:** Rename "HotNeighborsCanConnect" → "Cameras"
- **Why:** You have 3 SSIDs, need Trusted/IoT/Cameras

#### Option B: Keep as Guest, Create 4th SSID for Cameras
- **Action:** Keep guest network, add new "Cameras" SSID
- **Why:** You want a true guest network for visitors
- **Limitation:** CR1000A may only support 3 SSIDs (need to verify)

#### Option C: Disable Guest Network Entirely
- **Action:** Turn off "HotNeighborsCanConnect"
- **Why:** Don't need guest network right now

**❓ YOUR CHOICE:**
[ ] Option A: Repurpose Guest → Cameras (RECOMMENDED)
[ ] Option B: Keep Guest, add 4th SSID
[ ] Option C: Disable Guest network

**Recommendation:** **Option A** — You need camera isolation more than a guest network right now. Can always re-add guest later.

---

### Decision 7: IP Address Management

**Question:** How do you want to manage IP addresses?

**Options:**

#### Option A: Subnet per Network (Requires VLAN Support)
```
Trusted:  192.168.1.0/24  (current)
IoT:      192.168.2.0/24  (new)
Cameras:  192.168.3.0/24  (new)
```
- **Pros:** Cleanest segmentation, easy to write firewall rules
- **Cons:** Requires VLAN support (CR1000A may not have this)

#### Option B: Single Subnet with Firewall Rules (RECOMMENDED for CR1000A)
```
All devices: 192.168.1.0/24
- Trusted: .1-.99
- IoT:     .100-.199
- Cameras: .200-.249
- HA/Hubs: .240-.254
```
- **Pros:** Works with basic router (no VLAN needed)
- **Cons:** Firewall rules must use IP ranges instead of subnets

**Current Critical IPs to Preserve:**
```
192.168.1.242 → Home Assistant VM (keep as-is)
192.168.1.224 → Scrypted (verify this is active)
192.168.1.169 → Hue Bridge
192.168.1.165 → Aqara Hub M2
192.168.1.210 → Kitchen Camera
192.168.1.236 → Hallway Camera
192.168.1.155 → Entrance Camera
192.168.1.204 → Office Camera
```

**❓ YOUR CHOICE:**
[ ] Option A: Separate subnets (192.168.1.x, 192.168.2.x, 192.168.3.x)
[ ] Option B: Single subnet with IP ranges (RECOMMENDED)

**Recommendation:** **Option B** — Start with IP ranges on single subnet. If CR1000A supports VLANs, you can migrate to Option A later.

---

### Decision 8: DHCP vs Static IPs

**Question:** How should devices get IP addresses?

**Options:**

#### Option A: Static IPs for Critical Devices Only (RECOMMENDED)
- **Static (never change):**
  - Home Assistant: 192.168.1.242
  - Scrypted: 192.168.1.224
  - Hue Bridge: 192.168.1.169
  - Aqara Hub M2: 192.168.1.165
  - All 4 cameras: .210, .236, .155, .204
- **DHCP (dynamic):**
  - Phones, tablets, PCs, smart plugs, lights

#### Option B: Static IPs for Everything
- **What:** Assign fixed IP to every device via DHCP reservation
- **Pros:** Predictable IP addressing, easier firewall rules
- **Cons:** High maintenance (38+ devices)

#### Option C: DHCP for Everything
- **What:** Let router assign IPs dynamically
- **Pros:** Zero maintenance
- **Cons:** Device IPs can change (breaks firewall rules, HA configs)

**❓ YOUR CHOICE:**
[ ] Option A: Static for critical devices, DHCP for rest (RECOMMENDED)
[ ] Option B: Static for all devices
[ ] Option C: DHCP for everything

**Recommendation:** **Option A** — Static for HA, Scrypted, hubs, cameras. DHCP for everything else.

---

## 📋 Your Next Steps

### Before we proceed with Phase 1 implementation:

1. **Complete the Pre-Flight Checklist:**
   - See [phase1-preflight-checklist.md](phase1-preflight-checklist.md)
   - Especially: Verify CR1000A access, backup router config

2. **Make the 8 decisions above:**
   - Check the boxes for your choices
   - If unsure, I'll use the RECOMMENDED options

3. **Research CR1000A Capabilities:**
   - Does it support VLAN tagging?
   - Does it support mDNS repeater/reflection?
   - Does it support per-SSID firewall rules?
   - Can you set custom DNS servers?
   - (I can help you find this info via Verizon support docs or app)

4. **Tell me:**
   - "I've made my decisions" (share which options you picked)
   - OR "Use all RECOMMENDED options" (I'll proceed with safest defaults)
   - OR "I need help deciding X" (we'll discuss)

---

## ⚡ Quick Start (If You Trust My Recommendations)

If you want to proceed with my recommended config:

- **DNS:** Pi-hole in Docker on Mac mini
- **Networks:** 3 networks (Trusted, IoT, Cameras)
- **SSIDs:** Rename Guest → Cameras, disable band steering
- **HomeKit:** All via HA HomeKit Bridge (already your strategy)
- **Cameras:** Full isolation, no internet, Scrypted-only access
- **IPs:** Single subnet with IP ranges, static for critical devices

**To start Phase 1 with this config, reply:** "Go with recommended setup"

---

**Last Updated:** 2026-01-18
