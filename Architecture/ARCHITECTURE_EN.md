# On-Premises Architecture: Order Management System

**Version:** 1.0  
**Author:** Ubiratan Fernando Prestes  
**Last Updated:** February 2026 

---

## Overview

This document describes the **high-level** architecture of an on-premises solution for an **Order Management System (OMS)**, designed with a focus on **High Availability** and **Security** across all layers.

**Objective:** Process e-commerce orders with high availability, low latency, and robust security.

**Key Features:**
- Layered architecture with network isolation
- High availability across all critical components
- Secure communication (external TLS, internal mTLS)
- Asynchronous event processing

## Out of Scope (High-Level)

This document does **not** detail:
- Hardware specifications and sizing
- Operational procedures (deploy, backup, restore)
- Component-specific configurations
- Detailed monitoring and alerting

## Architecture Components

###  Nginx - API Gateway & Load Balancer

| Aspect | Description |
|--------|-------------|
| **Function** | Request routing, load balancing, TLS termination |
| **HA Configuration** | 2 active instances with Keepalived (Virtual IP) |
| **Security** | Integrated WAF, rate limiting, TLS 1.3 |

**Responsibilities:**
- Load balancing across application services
- SSL/TLS termination for external communication
- Protection against attacks (DDoS, SQL Injection, XSS)
- Path-based routing to different microservices

---

###  Redis - Read Cache

| Aspect | Description |
|--------|-------------|
| **Function** | Session cache, product catalog, frequent data |
| **HA Configuration** | Redis Sentinel (1 Master + 2 Replicas) |
| **Security** | ACL authentication, encrypted communication |

**Use Cases:**
- Product catalog caching (reduces PostgreSQL load)
- User session storage
- Rate limiting and real-time counters
- Frequent query result caching

---

###  Kafka - Asynchronous Messaging

| Aspect | Description |
|--------|-------------|
| **Function** | Asynchronous event and order processing |
| **HA Configuration** | 3-broker cluster + ZooKeeper (3 nodes) |
| **Security** | SASL/SCRAM, topic-level ACLs, TLS between brokers |

**Use Cases:**
- Decouple order processing from order creation
- Publish inventory updates to multiple consumers
- Queue email and SMS notifications
- Enable event-driven architecture across services

---

###  PostgreSQL - Data Persistence

| Aspect | Description |
|--------|-------------|
| **Function** | Persistent storage for orders, customers, products |
| **HA Configuration** | Primary + Standby with Patroni/etcd |
| **Security** | Encryption at rest, SSL connections, Row Level Security |

**Use Cases:**
- Store transactional data (orders, payments, shipments)
- Manage customer information and authentication
- Maintain product catalog with pricing and inventory
- Support complex queries and reporting

---

## High Availability

### Strategy by Layer

All critical components have **active redundancy**:

- **Nginx**: 2 instances with automatic failover via Virtual IP
- **Application Servers**: Multiple stateless instances (horizontal scaling)
- **Redis**: Master-Replica configuration with automatic promotion
- **Kafka**: 3-broker cluster with data replication
- **PostgreSQL**: Primary + Standby with automatic failover

**Goal:** Ensure no component is a single point of failure.

### Automatic Failover

1. **Nginx**: Keepalived detects failure and transfers VIP to backup instance
2. **Redis**: Sentinel automatically promotes replica to master
3. **Kafka**: Partitions are redistributed among available brokers
4. **PostgreSQL**: Patroni promotes standby to primary via etcd

---

## Security

### Layered Architecture

Security is implemented through **network isolation in 3 zones**:

**DMZ (Public Zone)**
- Nginx API Gateway with WAF
- Single entry point for external traffic
- TLS 1.3 termination

**Application Zone**
- Application Servers
- Redis Cache
- Encrypted internal communication (mTLS)

**Data Zone**
- Kafka and PostgreSQL
- Access restricted to applications only
- No direct internet connectivity

### Security Principles

 **Defense in Depth**: 3 firewall layers (Internet→DMZ, DMZ→App, App→Data)  
 **Encryption**: External TLS, internal mTLS  
 **Least Privilege**: Each component accesses only what's necessary  
 **Audit**: Centralized logging of all operations

---

## Data Flow

### Example: Order Creation

```
1. Client 
       ↓ HTTPS/TLS 1.3
2. Nginx (API Gateway)
       ↓ mTLS
3. Order Service
       ├→ Redis (check product cache)
       ├→ PostgreSQL (persist order)
       └→ Kafka (publish "order.created" event)
              ↓
4. Notification Service (consume event)
       ↓
5. External Email (confirmation to customer)
```

**Flow:**
1. Client sends order via HTTPS
2. Nginx routes to Order Service
3. Order Service validates product (Redis), saves order (PostgreSQL), and publishes event (Kafka)
4. Notification Service consumes event and sends confirmation email

---
