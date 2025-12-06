# Software Stack

## Purpose

This document describes the operating systems, tools, and services planned for the homelab. The stack is designed to support reproducible automation, container-based deployments, and infrastructure-as-code workflows — all focused on learning DevOps and platform engineering.

---

## Operating Systems

| Host                | OS              | Version    | Role                                                   |
| ------------------- | --------------- | ---------- | ------------------------------------------------------ |
| ThinkPad T440       | Ubuntu Server   | 24.04 LTS  | Main server — Ansible control, Docker host, Jenkins CI |
| Asus X550C          | Ubuntu Server   | 24.04 LTS  | Staging / reserve node                                 |
| Raspberry Pi 4 (x2) | Raspberry Pi OS | 64-bit     | Critical services: k3s nodes, VPN, monitoring          |
| Raspberry Pi 3B+    | Raspberry Pi OS | 64-bit     | Low-priority utilities: Pi-hole, experiments           |
| Workstation         | Ubuntu / Debian | Latest LTS | Development and testing                                |

Notes:

- Ubuntu Server 24.04 LTS chosen for long-term support, Debian compatibility, and wide community adoption
- Raspberry Pi OS provides good ARM support and community documentation
- All systems use 64-bit where possible

---

## Core Tooling

### Automation and Provisioning

- **Ansible**: Centralized configuration management from the ThinkPad control node
  - Playbooks for system updates, package installation, Docker setup, user management
  - Inventory organized by role (servers, pis, staging)

### Containers and Orchestration

- **Docker**: Initial container runtime for single-host deployments
- **Docker Compose**: Multi-container application definitions and Pi-hole deployment
- **k3s Kubernetes**: 2-node cluster on Pi4s for monitoring stack and learning
  - **Architecture**: Pi4 node 1 (server) + Pi4 node 2 (agent)
  - **Deployment**: Prometheus, Grafana, and Loki run as k3s pods
  - **Design rationale**: Keep cluster simple and focused on observability
  - **Disabled components**: Traefik (use NodePort), ServiceLB (not needed)
  - **Heavy workloads**: Jenkins, databases, build agents remain on x86 Docker hosts

### CI/CD

- **Jenkins**: Build automation and deployment pipelines
  - Initial pipelines: build Docker images, push to registry, deploy via Compose or k8s
  - Webhook triggers from GitHub
  - Runs on ThinkPad T440 as primary CI server

### Networking and Security

- **Nginx**: Reverse proxy and TLS termination (planned)
  - HTTP/HTTPS routing for containerized services
  - Let's Encrypt integration for SSL certificates
  - Load balancing for multi-instance deployments
- **WireGuard**: VPN server on GL.iNet router for remote access
- **Pi-hole**: DNS filtering and ad blocking via Docker on Pi3
- **Firewall**: UFW on Ubuntu Server, UFW on Raspberry Pis

### Monitoring and Observability

- **Prometheus**: Metrics collection deployed on k3s cluster
  - Scrapes node exporters from all hosts (Pis + laptops)
  - 7-day retention, 15-second scrape interval
  - NodePort access: http://192.168.8.20:30090
- **Grafana**: Dashboards and visualization on k3s
  - Pre-configured with Prometheus and Loki data sources
  - NodePort access: http://192.168.8.20:30030
  - Default credentials: admin/admin
- **Loki**: Log aggregation on k3s cluster
  - Lightweight alternative to ELK stack
  - Integrated with Grafana for unified logs + metrics
- **Node Exporter**: Deployed on all hosts via systemd service
  - Exposes host metrics on port 9100

### Infrastructure as Code (planned Phase 5)

- **Terraform**: Reproducible infrastructure provisioning
  - Start with local resources (libvirt VMs, network config)
  - Extend to cloud providers post-k8s (AWS, GCP, or DigitalOcean)

---

## Service Deployment Path

1. **k3s Cluster on Pi4s** (Completed)

   - 2-node cluster (pi4-node1 as server, pi4-node2 as agent)
   - Running monitoring stack: Prometheus, Grafana, Loki
   - Node exporters on all hosts for metrics collection
   - Simple architecture: no Traefik, using NodePort services

2. **Pi-hole on Pi3** (Completed)

   - DNS filtering via Docker Compose
   - Web interface on port 8080
   - Network-wide ad blocking (after router DNS config)

3. **Docker Compose on main server** (Completed)

   - Jenkins CI/CD server (running on port 9080)
   - Portfolio application (running on ports 80/443)
   - Resource-intensive workloads

4. **Expand k3s workloads** (Future)

   - Deploy lightweight stateless apps to Pi4 cluster
   - Experiment with persistent volumes
   - Learn Kubernetes patterns (deployments, services, ingress)

5. **Cloud migration** (Future)
   - Use Terraform and Kubernetes manifests for cloud deployments
   - Maintain homelab for experimentation and local development

---

## Network Architecture (current)

- Flat LAN: `192.168.8.0/24`
- TP-Link router handles DHCP and routing
- Static IP assignments for all hosts

VLANs are a future possibility but not currently planned. If/when a VLAN-capable router is introduced, segmentation will be considered.

---

## Development Workflow

1. Write playbook or Dockerfile on workstation
2. Commit to GitHub
3. Ansible provisions hosts or Jenkins builds container
4. Deploy to Docker Compose on x86 hosts or k3s on Pi cluster
5. Monitor with Prometheus/Grafana
6. Document in `docs/` with devlog entry

---

## Summary

The software stack prioritizes open-source, widely-adopted DevOps tools. The progression is: automation (Ansible) → containers (Docker) → orchestration (k3s) → IaC (Terraform) → cloud. Each phase builds on the previous, ensuring reproducibility and cloud portability.
