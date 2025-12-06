# Hardware Overview

## Core Systems

| Device        | CPU                           | RAM   | Storage  | OS                  | Role                                              |
| ------------- | ----------------------------- | ----- | -------- | ------------------- | ------------------------------------------------- |
| ThinkPad T440 | Intel i5-4300U                | 8 GB  | NVMe/SSD | Ubuntu Server 24.04 | Main server — Ansible control, Docker, Jenkins CI |
| Asus X550C    | Intel i3-3217U + GeForce 720M | 12 GB | SSD      | Ubuntu Server 24.04 | Staging / reserve node                            |

---

## Raspberry Pi Cluster

| Device            | Model                     | RAM  | Storage                        | Expansion          | Network         | Role                                                     |
| ----------------- | ------------------------- | ---- | ------------------------------ | ------------------ | --------------- | -------------------------------------------------------- |
| Raspberry Pi 4 #1 | BCM2711 (Quad-core ARM)   | 4 GB | Intenso Internal 128GB M.2 SSD | Geekworm X862 V2.0 | Evodata PoE hat | Critical services: k3s node, monitoring                  |
| Raspberry Pi 4 #2 | BCM2711 (Quad-core ARM)   | 4 GB | Intenso Internal 128GB M.2 SSD | Geekworm X862 V2.0 | Evodata PoE hat | Critical services: k3s node, backup agent, log collector |
| Raspberry Pi 3 B+ | BCM2837B0 (Quad-core ARM) | 1 GB | microSD                        | None               | Evodata PoE hat | Low-priority utilities: Pi-hole, experiments, GPIO       |

Notes:

- Geekworm X862 V2.0: SATA M.2 SSD expansion board with power management
- Evodata PoE hats: IEEE 802.3af/at compliant PoE+ power delivery
- Pi4 SSDs are used for services requiring storage durability (k3s, logs, VPN configs)
- Pi3 microSD is acceptable for ephemeral or read-heavy services only

---

## Networking Equipment

| Device    | Model           | Role                     | Notes                                                             |
| --------- | --------------- | ------------------------ | ----------------------------------------------------------------- |
| LTE Modem | Netgear LM1200  | WAN uplink               | LTE passthrough mode (optional/failover)                          |
| Router    | GL.iNet SF1200  | LAN gateway / DHCP / VPN | OpenWrt-based with VLAN support, SSH access, WireGuard VPN server |
| Switch    | Netgear GS308EP | Managed PoE+ switch      | Powers all Pis; supports VLAN tagging (not currently used)        |

---

## Physical Infrastructure

| Component       | Description                                                                    |
| --------------- | ------------------------------------------------------------------------------ |
| Rack            | Custom frame built from 2020 and 2040 aluminum extrusions with 10-inch shelves |
| Future upgrades | 3D-printed mounts and cable management parts                                   |
| Power           | PoE for all Pis via GS308EP; AC for servers                                    |
| UPS             | Not yet installed; planned for later                                           |

Notes:

- Rack design allows modular expansion and airflow management
- 3D-printed parts will improve cable routing and device mounting

---

## Summary

The homelab uses a mix of x86 servers and ARM-based Pis. Pi4s with SSD storage are designated for critical, write-heavy workloads. The Pi3 handles lightweight or experimental tasks. The custom rack provides flexible mounting and will be enhanced with 3D-printed improvements over time.
