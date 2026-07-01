# Production HA Monitoring Platform on Docker Swarm
# Deployments-Docker_Swarm

## Overview

This project deploys a highly available monitoring platform using Docker Swarm on Ubuntu 22.04 servers.

The solution provides:

* Docker Swarm High Availability Cluster
* Grafana High Availability
* Prometheus High Availability
* Alertmanager High Availability
* Node Exporter Monitoring
* cAdvisor Container Monitoring
* HAProxy Load Balancing
* Keepalived Virtual IP Failover
* Shared NFS Storage
* Production-ready Monitoring Architecture

---

# Infrastructure Layout

| Hostname           | IP Address    | Role                        |
| ------------------ | ------------- | --------------------------- |
| node1              | 192.18.29.136 | Docker Swarm Manager        |
| node2              | 192.18.29.137 | Docker Swarm Manager        |
| node3              | 192.18.29.138 | Docker Swarm Manager        |
| storage-nfs-node1  | 192.18.29.139 | NFS Storage                 |
| loadbalancer-node1 | 192.18.29.140 | HAProxy + Keepalived MASTER |
| loadbalancer-node2 | 192.18.29.141 | HAProxy + Keepalived BACKUP |
| Virtual IP         | 192.18.29.150 | Monitoring Endpoint         |

---

# Architecture

```text
                              VIP
                        192.18.29.150
                               |
                +--------------+--------------+
                |                             |
      loadbalancer-node1            loadbalancer-node2
          192.18.29.140               192.18.29.141
             HAProxy                     HAProxy
       Keepalived MASTER           Keepalived BACKUP
                |                             |
                +--------------+--------------+
                               |
                      Docker Swarm Cluster
                               |
         +---------------------+---------------------+
         |                     |                     |
      node1                 node2                 node3
   192.18.29.136       192.18.29.137       192.18.29.138
      Manager             Manager             Manager
      Grafana             Grafana             Grafana
    Prometheus          Prometheus          Prometheus
   Alertmanager        Alertmanager        Alertmanager
         |                  |                  |
         +------------------+------------------+
                            |
                    Shared NFS Storage
                            |
                 storage-nfs-node1
                    192.18.29.139
```

---

# High Availability Design

## Server HA

### Docker Swarm Managers

Three manager nodes are configured.

Benefits:

* Manager quorum maintained
* Automatic leader election
* Cluster survives single-node failure

### Load Balancer HA

Keepalived provides:

* Virtual IP (VIP)
* Automatic failover
* Active/Passive Load Balancer design

VIP Address:

```text
192.18.29.150
```

---

# Service HA

## Grafana

Replicas: 3

Features:

* Dashboard redundancy
* Session persistence through shared storage
* Load balanced access

## Prometheus

Replicas: 3

Features:

* Redundant metric collection
* Multiple scrape targets
* Improved monitoring availability

## Alertmanager

Replicas: 3

Features:

* Alert redundancy
* Alert clustering
* No single alerting point of failure

## Node Exporter

Deployment Mode:

```text
Global
```

Runs on every Swarm node.

## cAdvisor

Deployment Mode:

```text
Global
```

Provides:

* Container metrics
* CPU monitoring
* Memory monitoring
* Filesystem monitoring

---

# Network Ports

## Docker Swarm

| Port | Protocol | Purpose            |
| ---- | -------- | ------------------ |
| 2377 | TCP      | Cluster Management |
| 7946 | TCP/UDP  | Node Communication |
| 4789 | UDP      | Overlay Network    |

## Monitoring Services

| Service       | Port |
| ------------- | ---- |
| Grafana       | 3000 |
| Prometheus    | 9090 |
| Alertmanager  | 9093 |
| Node Exporter | 9100 |
| cAdvisor      | 8080 |

## Load Balancer

| Service | Port |
| ------- | ---- |
| HTTP    | 80   |
| HTTPS   | 443  |

---

# Storage Design

Shared storage is provided through NFS.

Exports:

```text
/exports/grafana
/exports/prometheus
/exports/alertmanager
```

Mounted on:

```text
/mnt/grafana
/mnt/prometheus
/mnt/alertmanager
```

---

# Deployment Workflow

## Phase 1

Prepare Operating System

* Ubuntu 22.04 Updates
* Package Installation
* Time Synchronization

## Phase 2

Configure Hostnames

## Phase 3

Configure DNS Resolution

* /etc/hosts

## Phase 4

Configure NFS Server

## Phase 5

Mount Shared Storage

## Phase 6

Install Docker

## Phase 7

Create Docker Swarm Cluster

## Phase 8

Create Overlay Network

## Phase 9

Deploy Monitoring Stack

Services:

* Node Exporter
* cAdvisor
* Prometheus
* Alertmanager
* Grafana

## Phase 10

Configure HAProxy

## Phase 11

Configure Keepalived

## Phase 12

Validate High Availability

---

# Validation Checklist

## Docker Swarm

```bash
docker node ls
```

Expected:

```text
Leader
Reachable
Reachable
```

## Services

```bash
docker service ls
```

Verify:

* Grafana
* Prometheus
* Alertmanager
* Node Exporter
* cAdvisor

Running successfully.

## NFS

```bash
showmount -e 192.18.29.139
```

## VIP

```bash
ip addr
```

Expected:

```text
192.18.29.150
```

Available on active load balancer.

---

# Access URLs

## Grafana

```text
http://192.18.29.150/grafana
```

## Prometheus

```text
http://192.18.29.150/prometheus
```

## Alertmanager

```text
http://192.18.29.150/alertmanager
```

---

# Failover Testing

## Test 1

Stop HAProxy on MASTER.

Expected:

* VIP moves to BACKUP node.
* Services remain available.

## Test 2

Power off one Docker Manager.

Expected:

* Cluster remains operational.
* Services continue running.

## Test 3

Stop one Grafana container.

Expected:

* Remaining replicas continue serving traffic.

---

# Backup Strategy

## Grafana

Backup:

```text
/exports/grafana
```

## Prometheus

Backup:

```text
/exports/prometheus
```

## Alertmanager

Backup:

```text
/exports/alertmanager
```

Recommended:

* Daily incremental backup
* Weekly full backup
* Offsite backup retention

---

# Project Outcome

This deployment provides:

* Highly Available Monitoring Platform
* High Availability Load Balancing
* Docker Swarm Cluster Resilience
* Shared Persistent Storage
* Centralized Monitoring
* Production-grade Architecture
* Fault Tolerance Across Infrastructure and Services
