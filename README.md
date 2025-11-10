# Homelab Infrastructure Project

A production-like DevOps learning environment built on real hardware, focusing on automation, containerization, and infrastructure as code.

---

## Overview

This homelab serves as a practical platform for learning and demonstrating DevOps and platform engineering skills. The project emphasizes reproducible infrastructure, configuration as code, and production-ready workflows on self-hosted hardware.

**Key focus areas:**

- Configuration management and automation (Ansible)
- Container orchestration (Docker, Kubernetes)
- CI/CD pipeline development (Jenkins)
- Infrastructure as code (Terraform)
- Monitoring and observability (Prometheus, Grafana)
- Network services (VPN, reverse proxy, DNS filtering)

---

## Current Infrastructure

### Hardware

**Compute:**

- 2x x86 servers (ThinkPad T440, Asus X550C) running Rocky Linux
- 2x Raspberry Pi 4 (4GB RAM) with M.2 SSD storage via Geekworm X862 expansion boards
- 1x Raspberry Pi 3B+ for lightweight utilities

**Networking:**

- Netgear GS308EP managed PoE+ switch
- GL.iNet SF1200 router (OpenWrt-based, VLAN-capable)
- Evodata PoE hats powering all Raspberry Pis

**Physical Infrastructure:**

- Custom rack built from 2020/2040 aluminum extrusions
- Planned 3D-printed cable management and mounting solutions

### Software Stack

**Core Technologies:**

- **OS:** Rocky Linux 10 (servers), Raspberry Pi OS 64-bit (ARM nodes)
- **Automation:** Ansible for configuration management and provisioning
- **Containers:** Docker and Docker Compose
- **CI/CD:** Jenkins with GitHub webhook integration
- **Reverse Proxy:** Nginx with Let's Encrypt SSL
- **Monitoring:** Prometheus + Grafana (planned)
- **VPN:** WireGuard for secure remote access
- **DNS:** Pi-hole for network-wide ad blocking

**Infrastructure as Code:**

- Ansible playbooks for all system configuration
- Dockerfiles and Compose manifests for service deployment
- Terraform for future cloud and hybrid deployments

---

## Architecture Decisions

### Kubernetes Strategy

The project will introduce Kubernetes through k3s (lightweight k8s) on the Raspberry Pi 4 cluster:

- Pi4s provide persistent SSD storage suitable for etcd and workload data
- Separation of concerns: k3s for orchestration learning, x86 hosts for resource-intensive services
- Progression path: standalone k3s cluster → optional mixed-architecture cluster → cloud migration

**Rationale:** This approach balances learning objectives with operational stability. Resource-heavy tasks (Jenkins builds, databases) remain on more capable x86 hardware while Kubernetes workloads run on dedicated ARM nodes.

### Network Design

**Current:** Flat LAN (192.168.8.0/24) with static IP assignments

**Future:** VLAN segmentation is a possibility but not an immediate priority. Focus remains on automation, service reliability, and monitoring before introducing network complexity.

---

## Project Phases

### Phase 1: Foundation (Current)

- Hardware assembly and network connectivity
- Operating system installation and baseline configuration
- Static IP assignment and documentation

### Phase 2: Automation

- Ansible control node setup
- SSH key-based authentication across all hosts
- Base playbooks: system updates, user management, Docker installation

### Phase 3: Core Services

- Jenkins CI/CD server
- Nginx reverse proxy with SSL
- First containerized application deployment
- GitHub webhook integration

### Phase 4: Observability

- Prometheus metrics collection
- Grafana dashboards
- Node exporters on all hosts
- Backup and recovery procedures

### Phase 5: Orchestration

- k3s deployment on Raspberry Pi cluster
- Kubernetes workload migration
- Terraform for infrastructure provisioning
- GitOps workflows

### Phase 6: Cloud Integration

- Hybrid cloud architecture using Terraform
- Workload portability between on-prem and cloud
- Comparative performance and cost analysis

---

## Repository Structure

```
homelab/
├── README.md                    # This file
├── hardware.md                  # Complete hardware inventory
├── software_stack.md            # Software tools and deployment paths
├── docs/                        # Additional documentation
│   └── devlog/                  # Session notes and progress logs
├── ansible/                     # Configuration management playbooks (planned)
├── docker/                      # Container definitions and compose files (planned)
└── terraform/                   # Infrastructure as code (planned)
```

---

## Key Learning Outcomes

This project demonstrates practical experience with:

**Infrastructure & Automation:**

- Bare-metal server provisioning and lifecycle management
- Configuration as code using Ansible
- Idempotent, reproducible infrastructure patterns

**Containerization & Orchestration:**

- Docker multi-stage builds and optimization
- Docker Compose for multi-service applications
- Kubernetes cluster administration (k3s)
- Multi-architecture container deployments (ARM/x86)

**CI/CD & DevOps Practices:**

- Jenkins pipeline development
- Automated build, test, and deployment workflows
- Git-based infrastructure workflows
- Integration between CI and container orchestration

**Networking & Security:**

- Reverse proxy configuration and TLS management
- VPN setup for secure remote access
- Network segmentation planning (future)
- Firewall rule management

**Monitoring & Operations:**

- Metrics collection and visualization
- Service health monitoring
- Backup and disaster recovery planning

---

## Why a Homelab?

Cloud platforms are excellent for production workloads, but a physical homelab offers unique learning advantages:

1. **Full control:** Complete access to the infrastructure stack, from hardware to application layer
2. **Real constraints:** Working with limited resources teaches optimization and efficiency
3. **Failure learning:** Safe environment to break things, troubleshoot, and rebuild
4. **Cost-effective:** Using mostly hardware thats already laying around or cheap vs. ongoing cloud costs for experimentation
5. **Hybrid skills:** Experience managing both on-premises and cloud infrastructure

The ultimate goal is cloud competency, with the homelab serving as a proving ground for infrastructure patterns that will be replicated in cloud environments.

---

## Current Status

**Completed:**

- Hardware assembly and rack construction
- Network configuration with static IPs
- Operating system installation on all nodes
- Documentation structure and initial technical specifications

**In Progress:**

- Ansible playbook development
- SSH key distribution and access control
- Jenkins installation planning

**Next Steps:**

- Execute base Ansible playbooks across all hosts
- Deploy Jenkins on primary server
- Create first CI/CD pipeline for containerized application
- Implement WireGuard VPN on Raspberry Pi 4

---

## Documentation

All infrastructure decisions, configurations, and procedures are documented in Markdown and version-controlled in this repository. Each significant change includes a devlog entry with context, implementation details, and lessons learned.

---

## Contact

This is a personal learning project demonstrating DevOps and platform engineering capabilities. For questions or collaboration opportunities, please reach out via GitHub.

---

## License

Documentation and configuration code in this repository are available under the MIT License. See LICENSE file for details.
