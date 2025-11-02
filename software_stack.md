# Software Stack

## Purpose

This document describes the operating systems, tools, and services planned for the homelab. The stack is designed to support reproducible automation, container-based deployments, and infrastructure-as-code workflows — all focused on learning DevOps and platform engineering.

---

## Operating Systems

| Host                | OS              | Version    | Role                                                   |
| ------------------- | --------------- | ---------- | ------------------------------------------------------ |
| ThinkPad T440       | Rocky Linux     | 9.x        | Main server — Ansible control, Docker host, Jenkins CI |
| Asus X550C          | Rocky Linux     | 9.x        | Staging / reserve node                                 |
| Raspberry Pi 4 (x2) | Raspberry Pi OS | 64-bit     | Critical services: k3s nodes, VPN, monitoring          |
| Raspberry Pi 3B+    | Raspberry Pi OS | 64-bit     | Low-priority utilities: Pi-hole, experiments           |
| Workstation         | Ubuntu / Debian | Latest LTS | Development and testing                                |

Notes:

- Rocky Linux chosen for RHEL compatibility and stability
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
- **Docker Compose**: Multi-container application definitions
- **Kubernetes** (planned Phase 5): Orchestration strategy to be determined
  - Options under consideration:
    - Single k3s cluster spanning both Pi4s and x86 nodes (mixed architecture)
    - Separate k3s cluster on Pi4s (ARM) + bare-metal services on laptops
    - k3s on Pi4s only, laptops remain Docker-based for heavier workloads
  - Decision factors: resource constraints, learning goals, workload requirements
  - Pi4s have limited RAM (4GB each) but persistent SSD storage
  - Laptops have more compute (8-12GB RAM) but may serve better as dedicated CI/monitoring hosts
  - Initial direction: Start with k3s on Pi4s as a learning cluster, run production-like services on laptops via Docker/Compose

### CI/CD

- **Jenkins**: Build automation and deployment pipelines
  - Initial pipelines: build Docker images, push to registry, deploy via Compose or k8s
  - Webhook triggers from GitHub
  - Runs on ThinkPad T440 as primary CI server

### Networking and Security

- **Nginx**: Reverse proxy and TLS termination
  - HTTP/HTTPS routing for containerized services
  - Let's Encrypt integration for SSL certificates
  - Load balancing for multi-instance deployments
- **WireGuard**: VPN server (hosted on Pi4 for remote access)
- **Pi-hole**: DNS filtering and ad blocking (on Pi3)
- **Firewall**: firewalld on Rocky Linux, iptables on Pis

### Monitoring and Observability (planned Phase 4)

- **Prometheus**: Metrics collection from exporters on all hosts
- **Grafana**: Dashboards and visualization
- **Node Exporter**: Host-level metrics

### Infrastructure as Code (planned Phase 5)

- **Terraform**: Reproducible infrastructure provisioning
  - Start with local resources (libvirt VMs, network config)
  - Extend to cloud providers post-k8s (AWS, GCP, or DigitalOcean)

---

## Service Deployment Path

1. Docker Compose on main server (Phase 3)

   - First app: Vite + Three.js static site
   - Jenkins, Nginx, simple APIs
   - All services on ThinkPad T440 initially

2. Kubernetes exploration (Phase 5)

   - Experiment with k3s on Pi4 cluster for learning
   - Deploy non-critical or stateless workloads to k3s
   - Keep resource-intensive services (Jenkins, databases, build agents) on x86 Docker hosts
   - Evaluate mixed-architecture cluster vs. separate ARM/x86 deployment models

3. Cloud migration (Phase 6)
   - Use Terraform and Kubernetes manifests to deploy workloads to cloud
   - Maintain on-prem for experimentation, development, and latency-sensitive services
   - Cloud-first for scalability experiments and public-facing production apps

---

## Network Architecture (current)

- Flat LAN: `192.168.1.0/24`
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
