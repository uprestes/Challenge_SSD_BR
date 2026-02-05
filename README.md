# Challenge_SSD_BR

This repository contains the solution for a technical  challenge focused on **on-premises infrastructure architecture**, **high availability**, **security**, and **automation**.

The project demonstrates design decisions, architectural reasoning, and Day 2 operational practices using common enterprise technologies.

---
##  Architecture

The **Architecture** folder contains the high-level design for an **On-Premises Order Management System**.

### Files
- **architecture_diagram.jpg**
  - High-level visual diagram illustrating network zones, services, and data flow.
- **ARCHITECTURE_EN.md**
  - Detailed architecture documentation covering:
    - System components
    - High availability strategies
    - Security controls
    - Network segmentation
    - Design rationale

### Key Components
- Nginx (API Gateway / Load Balancer with TLS and WAF)
- Stateless application services
- Redis Cluster (Sentinel)
- Kafka Cluster (event-driven communication)
- PostgreSQL Cluster (Primary / Standby with Patroni)
- Clear separation of:
  - DMZ
  - Application Zone
  - Data & Messaging Zone

---

##  Automation (Ansible)

The **Project-ansible** folder contains an Ansible playbook focused on **Day 2 operations**.

### Scope
- PostgreSQL operational tasks
- Service checks and management
- Backup and restore operations
- Secure credential handling using Ansible Vault

### Documentation
A detailed README is available inside the folder: Project-ansible/README.md

---

##  Design Principles

- High Availability across all critical components
- Defense-in-depth security model
- Secure communication (TLS / mTLS)
- Stateless application layer
- Clear separation of concerns
- Operational simplicity and maintainability

---

##  Author

**Ubiratan Fernando Prestes**

---

## Version

- Architecture version: **2.0**
- Last update: **February 2026**
