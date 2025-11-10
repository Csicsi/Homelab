# Network Setup Guide

## Overview

**Goal**: Configure LTE modem → router → switch topology with static IP assignments for all homelab devices.

## Hardware Chain

```
[Internet/LTE] → [Netgear LM1200] → [GL.iNet SF1200] → [Netgear GS308EP Switch] → [Devices]
```

**Network**: `192.168.8.0/24`  
**Router/Gateway**: `192.168.8.1` (GL.iNet default - no change needed)  
**Infrastructure**: `192.168.8.2-99` (static IPs for servers, switch, Pis)  
**DHCP Pool**: `192.168.8.100-200` (guests, temporary devices)

## Setup Steps

### 1. LM1200 Modem Setup

1. Insert SIM card
2. Connect to power
3. Connect Ethernet cable from LM1200 LAN port to GL.iNet WAN port
4. Wait for LTE connection (check indicator lights)
5. Connect to Modem on 192.168.5.1
6. Set Settings->Mobile->Auto Connect to "Always" to prevent it from dropping the connection

**Note**: LM1200 should be in bridge/passthrough mode (default) - router handles DHCP/routing.

### 2. GL.iNet SF1200 Router Setup

**For detailed setup instructions, see `docs/glinet_setup.md`**

Access router web interface:

- Stock firmware: `http://192.168.8.1` or `http://gl-sf1200.lan`

**Basic Settings**:

- Set admin password
- Configure WiFi SSID and password (or disable WiFi if using wired-only)
- Verify WAN connection is active (should get IP from LM1200)
- Enable SSH access (System → Advanced Settings)

**DHCP Settings** (Network → LAN → DHCP Server):

- Set DHCP range: `192.168.8.100 - 192.168.8.200` (leaves `.2-.99` for infrastructure)
- Gateway: `192.168.8.1` (router itself - default)
- DNS: Use router's DNS or public DNS (1.1.1.1, 8.8.8.8)

**Static IP Reservations** (Network → LAN → DHCP Reservations or use Ansible - see `ansible/playbooks/setup_glinet_openwrt.yml`):
| Device | MAC Address | Reserved IP |
|-------------------|---------------------|---------------|
| Netgear GS308EP | (from switch label) | 192.168.8.2 |
| ThinkPad T440 | (from `ip link`) | 192.168.8.10 |
| Asus X550C | (from `ip link`) | 192.168.8.11 |
| Raspberry Pi 4 #1 | (from `ip link`) | 192.168.8.20 |
| Raspberry Pi 4 #2 | (from `ip link`) | 192.168.8.21 |
| Raspberry Pi 3B+ | (from `ip link`) | 192.168.8.22 |

### 3. Netgear GS308EP Switch Setup

1. Connect switch to router (any switch port → router LAN port)
2. Power on switch
3. Access switch web interface at `http://192.168.8.2` (after DHCP reservation takes effect)

**PoE Configuration**:

- Enable PoE on ports 4, 5, 6 (for Raspberry Pis with Evodata hats)
- Set PoE priority if needed (High for critical Pis)

**Port Assignment** (update `network_inventory.md`):

- Port 1: Uplink to GL.iNet router
- Port 2: ThinkPad T440
- Port 3: Asus X550C
- Port 4-6: Raspberry Pis (with PoE enabled)

### 4. Connect Devices

1. Connect servers and Pis to switch via Ethernet
2. Boot each device
3. Verify they receive correct static IPs from DHCP reservations
4. Test connectivity: `ping 192.168.8.1` (router), `ping 8.8.8.8` (internet)

## Verification Checklist

- [ ] LM1200 has LTE connection (check lights)
- [ ] GL.iNet router has WAN IP from modem
- [ ] Router accessible at 192.168.8.1 (default)
- [ ] SSH enabled on router
- [ ] DHCP reservations configured for all devices
- [ ] Switch accessible at 192.168.8.2
- [ ] PoE enabled on Pi ports (4, 5, 6)
- [ ] All devices get correct static IPs
- [ ] All devices can ping router (192.168.8.1)
- [ ] All devices can reach internet
- [ ] Update `docs/network_inventory.md` with real MAC addresses

## Finding MAC Addresses

**On Linux devices** (servers/Pis after initial boot):

```bash
ip link show
# Look for "link/ether XX:XX:XX:XX:XX:XX"
```

**From GL.iNet router**:

- Go to Clients page in web UI
- View connected devices with MAC addresses
- Or via SSH: `cat /tmp/dhcp.leases`

## Troubleshooting

**No internet**: Check LM1200 LTE connection, verify GL.iNet WAN settings

**Device not getting static IP**: Check MAC address matches DHCP reservation exactly

**Can't access router**: Try default IP `192.168.8.1` if LAN IP not changed yet, or factory reset

**Can't access switch**: Verify switch got 192.168.8.2 from router, try factory reset if needed

