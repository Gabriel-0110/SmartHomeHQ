# Phase 1 Network Topology

**Architecture:** Dual-Router Design with Intentional Double NAT

## Network Diagram

```mermaid
flowchart LR
    ISP[ISP Modem]
    
    subgraph R1["Router #1 (Trusted / Control Plane)<br>192.168.1.0/24"]
        R1_GW["192.168.1.1<br>Gateway"]
        AppleTV["Apple TV 4K<br>192.168.1.50<br>HomeKit Hub"]
        HA["Home Assistant<br>192.168.1.21"]
        Mac_Trusted["Mac mini NIC A<br>192.168.1.20<br>Trusted Interface"]
        Trusted["Trusted Clients<br>Phones, PCs, iPads"]
    end
    
    subgraph R2["Router #2 (IoT / Edge)<br>192.168.2.0/24"]
        R2_GW["192.168.2.1<br>Gateway"]
        Mac_IoT["Mac mini NIC B<br>192.168.2.10<br>IoT Interface"]
        Hue["Hue Bridge<br>192.168.2.50"]
        Aqara["Aqara Hub<br>192.168.2.51"]
        IoT["IoT Devices<br>Echo, Plugs, etc."]
        Cameras["Cameras<br>192.168.2.150-153<br>Internet Blocked"]
    end
    
    Mac_Trusted -.Dual-Homed.-> Mac_IoT
    
    ISP -->|WAN| R1_GW
    R1_GW -->|"LAN → WAN<br>NAT Boundary"| R2_GW
    
    R1_GW --- AppleTV
    R1_GW --- Mac_Trusted
    R1_GW --- HA
    R1_GW --- Trusted
    
    R2_GW --- Mac_IoT
    R2_GW --- Hue
    R2_GW --- Aqara
    R2_GW --- IoT
    R2_GW --- Cameras
    
    AppleTV -->|mDNS/HomeKit| HA
    Mac_Trusted -->|HA UI Access| HA
    Mac_IoT -->|"HA/Scrypted→IoT"| Hue
    Mac_IoT -->|"HA/Scrypted→IoT"| Aqara
    Mac_IoT -->|"Scrypted→Cameras"| Cameras
```

## Key Design Elements

- **Dual-Router Architecture**: Two Verizon CR1000A routers in intentional double NAT configuration
- **Router #2 in ROUTER MODE**: WAN port connected to Router #1 LAN (LAN → WAN)
- **Two Subnets**: Trusted (192.168.1.0/24) and IoT (192.168.2.0/24)
- **Mac mini Dual-Homed**: One interface on each router enables HA/Scrypted to reach IoT devices
- **NAT Boundary**: Provides subnet separation but is NOT a security boundary
- **Endpoint Enforcement**: macOS/Windows firewalls provide actual security enforcement
- **HomeKit on Trusted**: All HomeKit hubs stay on Router #1 for mDNS compatibility
- **Camera Isolation**: Cameras on Router #2 with internet access blocked via Access Control

## Traffic Flows

- **HomeKit/mDNS**: Remains within Trusted subnet (Router #1)
- **HA/Scrypted → IoT**: Via Mac mini's IoT interface (192.168.2.10)
- **Trusted → IoT**: Not possible for generic clients (no routing between subnets)
- **Mac mini only**: Can reach both subnets due to dual-homing

---

**Last Updated:** 2026-01-19
