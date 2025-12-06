# Homelab Services Dashboard

## Quick Access URLs

| Service             | URL                              | Credentials      | Status         |
| ------------------- | -------------------------------- | ---------------- | -------------- |
| **Homer Dashboard** | **http://192.168.8.22:8081**     | **N/A**          | **✅ Running** |
| Jenkins             | http://192.168.8.10:9080         | admin/configured | ✅ Running     |
| Portfolio Site      | http://192.168.8.10 (HTTP/HTTPS) | N/A              | ✅ Running     |
| Pi-hole Admin       | http://192.168.8.22:8080/admin   | admin/admin      | ✅ Running     |
| Prometheus          | http://192.168.8.20:30090        | N/A              | ✅ Running     |
| Grafana             | http://192.168.8.20:30030        | admin/admin      | ✅ Running     |
| GL.iNet Router      | http://192.168.8.1               | root/configured  | ✅ Running     |

---

## Service Breakdown by Host

### homelab-main (192.168.8.10) - x86 ThinkPad T440

**Docker Containers:**

- **Jenkins** (jenkins/jenkins:lts)
  - Port: 9080 (HTTP)
  - Purpose: CI/CD automation, build pipelines
  - Uptime: 6 days
- **Portfolio Site** (portfolio-portfolio)
  - Ports: 80 (HTTP), 443 (HTTPS)
  - Purpose: Personal portfolio/website
  - Uptime: 5 hours

**System Services:**

- Docker Engine
- Node Exporter (port 9100) - metrics for Prometheus
- SSH (port 22)

---

### pi4-node1 (192.168.8.20) - k3s Server Node

**k3s Workloads:**

- **Prometheus** (NodePort 30090)

  - Metrics collection and storage
  - Scrapes all node exporters (15s interval)
  - 7-day retention

- **Grafana** (NodePort 30030)
  - Visualization dashboards
  - Pre-configured data sources: Prometheus, Loki
- **Loki** (internal)
  - Log aggregation
  - Accessible via Grafana

**System Services:**

- k3s server (control plane)
- Node Exporter (port 9100)
- SSH (port 22)

---

### pi4-node2 (192.168.8.21) - k3s Agent Node

**k3s Role:**

- Worker node (agent)
- Available for pod scheduling
- Part of monitoring cluster

**System Services:**

- k3s agent
- Node Exporter (port 9100)
- SSH (port 22)

---

### pi3-utils (192.168.8.22) - Raspberry Pi 3B+

**Docker Containers:**

- **Homer Dashboard** (b4bz/homer:latest)

  - Port: 8081 (HTTP)
  - Purpose: Unified dashboard for all homelab services
  - Status: Healthy
  - Features: Quick links to all services, infrastructure overview

- **Pi-hole** (pihole/pihole:latest)
  - Ports: 53 (DNS TCP/UDP), 8080 (Web UI)
  - Purpose: Network-wide DNS filtering and ad blocking
  - Status: Healthy
  - Uptime: ~35 minutes

**System Services:**

- Docker Engine
- SSH (port 22)

---

### glinet-router (192.168.8.1) - GL.iNet SF1200

**Services:**

- **WireGuard VPN Server**
  - Port: 51820 (UDP)
  - Purpose: Remote access to homelab
- **DuckDNS DDNS**
  - Purpose: Dynamic DNS for changing public IP
- **DHCP Server**

  - Static reservations for all hosts
  - Range: 192.168.8.0/24

- **OpenWrt UCI Management**
  - Firewall (fw4)
  - Network configuration

---

## Monitoring Coverage

All hosts report metrics to Prometheus via Node Exporter:

- ✅ homelab-main (192.168.8.10:9100)
- ✅ pi4-node1 (192.168.8.20:9100)
- ✅ pi4-node2 (192.168.8.21:9100)
- ❓ pi3-utils (192.168.8.22:9100) - needs verification
- ❓ glinet-router (192.168.8.1) - router metrics not yet configured

**Web Applications:**

- Homer Dashboard (port 8081) - **START HERE**
- Portfolio site (port 80/443)
- Jenkins (port 9080)
- Pi-hole Admin (port 8080)
- Grafana (port 30030)
- Prometheus (port 30090)
  **Web Applications:**
- Portfolio site (port 80/443)
- Jenkins (port 9080)
- Pi-hole Admin (port 8080)
- Grafana (port 30030)
- Prometheus (port 30090)

**Infrastructure:**

- DNS: Pi-hole on 192.168.8.22:53
- VPN: WireGuard on router (port 51820)
- DHCP: Router (192.168.8.1)
- Metrics: Prometheus scraping node exporters
- Logs: Loki on k3s cluster

**Orchestration:**

- k3s: 2-node cluster (pi4-node1 + pi4-node2)
- Docker: homelab-main, pi3-utils
- Docker Compose: Pi-hole

---

## Next Steps / TODO

- [ ] Configure router DNS to point to Pi-hole (192.168.8.22)
- [ ] Install Node Exporter on pi3-utils for monitoring
- [ ] Add router metrics to Prometheus (if supported)
- [ ] Change Pi-hole admin password from default
- [ ] Create Grafana dashboards for homelab metrics
- [ ] Document Jenkins pipeline configurations
- [ ] Set up automated backups for persistent volumes

---

## Maintenance Notes

**Last Updated:** 2025-12-05

**Recent Changes:**

- Pi-hole deployed on pi3-utils (docker-compose)
- Fixed systemd-resolved conflict on Raspberry Pi OS
- Replaced community.docker module with shell for reliability

**Known Issues:**

- None currently

**Backup Status:**

- Jenkins config: Not automated
- Grafana dashboards: Not backed up
- Pi-hole config: Persistent volumes in /opt/pihole
