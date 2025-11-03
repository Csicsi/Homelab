# Network Setup Guide

## Overview

**Goal**: Configure LTE modem → router → switch topology with static IP assignments for all homelab devices.

## Hardware Chain

```
[Internet/LTE] → [Netgear LM1200] → [TP-Link Router] → [Netgear GS308EP Switch] → [Devices]
```

## Setup Steps

### 1. LM1200 Modem Setup

1. Insert SIM card
2. Connect to power
3. Connect Ethernet cable from LM1200 LAN port to TP-Link WAN port
4. Wait for LTE connection (check indicator lights)
5. Connect to Modem on 192.168.5.1
6. Set Settings->Mobile->Auto Connect to "Always" to prevent it from dropping the connection

**Note**: LM1200 should be in bridge/passthrough mode (default) - router handles DHCP/routing.

### 2. TP-Link Router Configuration

Access router web interface (http://tplinkwifi.net/):

**Basic Settings**:

- Set admin password
- Configure WiFi SSID and password (unless VPN is already configured, then turn off)
- Verify WAN connection is active (should get IP from LM1200)

**DHCP Settings**:

- Set DHCP range: `192.168.1.100 - 192.168.1.200` (leaves `.2-.99` for infrastructure)
- Gateway: `192.168.1.1` (router itself)
- DNS: Use router's DNS or public DNS (1.1.1.1, 8.8.8.8)

**Static IP Reservations** (DHCP Reservation/Address Reservation):
| Device | MAC Address | Reserved IP |
|-------------------|---------------------|---------------|
| Netgear GS308EP | (from switch label) | 192.168.1.2 |
| ThinkPad T440 | (from `ip link`) | 192.168.1.10 |
| Asus X550C | (from `ip link`) | 192.168.1.11 |
| Raspberry Pi 4 #1 | (from `ip link`) | 192.168.1.20 |
| Raspberry Pi 4 #2 | (from `ip link`) | 192.168.1.21 |
| Raspberry Pi 3B+ | (from `ip link`) | 192.168.1.22 |

### 3. Netgear GS308EP Switch Setup

1. Connect switch to router (any switch port → router LAN port)
2. Power on switch
3. Access switch web interface at `http://192.168.1.2` (after DHCP reservation takes effect)

**PoE Configuration**:

- Enable PoE on ports 4, 5, 6 (for Raspberry Pis with Evodata hats)
- Set PoE priority if needed (High for critical Pis)

**Port Assignment** (update `network_inventory.md`):

- Port 1: Uplink to TP-Link router
- Port 2: ThinkPad T440
- Port 3: Asus X550C
- Port 4-6: Raspberry Pis (with PoE enabled)

### 4. Connect Devices

1. Connect servers and Pis to switch via Ethernet
2. Boot each device
3. Verify they receive correct static IPs from DHCP reservations
4. Test connectivity: `ping 192.168.1.1` (router), `ping 8.8.8.8` (internet)

## Verification Checklist

- [ ] LM1200 has LTE connection (check lights)
- [ ] TP-Link router has WAN IP from modem
- [ ] DHCP reservations configured for all devices
- [ ] Switch accessible at 192.168.1.2
- [ ] PoE enabled on Pi ports (4, 5, 6)
- [ ] All devices get correct static IPs
- [ ] All devices can ping router (192.168.1.1)
- [ ] All devices can reach internet
- [ ] Update `docs/network_inventory.md` with real MAC addresses

## Finding MAC Addresses

**On Linux devices** (servers/Pis after initial boot):

```bash
ip link show
# Look for "link/ether XX:XX:XX:XX:XX:XX"
```

**From TP-Link router**:

- Check "DHCP Client List" or "Connected Devices"
- Note MAC and current IP for each device

## Troubleshooting

**No internet**: Check LM1200 LTE connection, verify TP-Link WAN settings

**Device not getting static IP**: Check MAC address matches DHCP reservation exactly

**Can't access switch**: Verify switch got 192.168.1.2 from router, try factory reset if needed

**PoE not working**: Check PoE budget (GS308EP has 83W total), verify Evodata hat compatibility
