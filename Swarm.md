# Docker Swarm Cluster Setup and Management Guide

![Docker](https://img.shields.io/badge/Docker-Swarm-blue)
![Cluster](https://img.shields.io/badge/Cluster-Multi--Node-green)
![Mode](https://img.shields.io/badge/Mode-Replicated%20%7C%20Global-orange)
![Beginner Friendly](https://img.shields.io/badge/Level-Beginner%20Friendly-success)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

This repository provides a complete guide to:

- Setting up a Docker Swarm cluster
- Managing nodes (manager and workers)
- Creating and scaling services
- Using constraints and labels
- Understanding Swarm scheduling behavior

---

## Table of Contents

- [Overview](#overview)
- [Cluster Architecture](#cluster-architecture)
- [Prerequisites](#prerequisites)
- [Initialize Swarm](#initialize-swarm)
- [Join Nodes](#join-nodes)
- [Manage Nodes](#manage-nodes)
- [Promote and Demote Nodes](#promote-and-demote-nodes)
- [Service Management](#service-management)
- [Scaling Services](#scaling-services)
- [Global vs Replicated Mode](#global-vs-replicated-mode)
- [Constraints and Labels](#constraints-and-labels)
- [Failure Handling](#failure-handling)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)

---

## Overview

Docker Swarm is Docker’s native orchestration tool used to manage clusters of Docker nodes.

This guide demonstrates:

- Cluster setup with 1 manager and multiple workers
- Service orchestration across nodes
- Automatic scheduling and recovery

---

## Cluster Architecture

```
Cluster
 ├── ubuntu-master (Manager / Leader)
 ├── ubuntu-worker01 (Worker)
 └── ubuntu-worker02 (Worker)
```

---

## Prerequisites

- 3 or more Linux machines (VMs or EC2 instances)
- Docker installed on all nodes
- Network connectivity between nodes

Verify Docker:

```bash
docker --version
```

---

## Initialize Swarm

Run on the manager node:

```bash
docker swarm init
```

This generates a join command for workers:

```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

---

## Join Nodes

Run the join command on each worker node:

```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

---

## Retrieve Join Tokens

If you lose the join command:

### Worker Token

```bash
docker swarm join-token worker
```

### Manager Token

```bash
docker swarm join-token manager
```

---

## Manage Nodes

### List Nodes

```bash
docker node ls
```

---

### Remove a Worker Node

On the worker:

```bash
docker swarm leave
```

On the manager:

```bash
docker node rm <node-name>
```

---

### Force Remove Node (if unreachable)

```bash
docker node rm -f <node-name>
```

---

### Verify Node Status

```bash
docker info
```

---

## Promote and Demote Nodes

### Promote Worker to Manager

```bash
docker node promote <node-name>
```

---

### Demote Manager to Worker

```bash
docker node demote <node-name>
```

---

### Recommendation

Use an odd number of managers:

```
1, 3, 5, 7...
```

---

## Service Management

### Create a Service

```bash
docker service create -d --replicas 4 alpine ping 192.168.64.10
```

---

### List Services

```bash
docker service ls
```

---

### View Tasks

```bash
docker service ps <service-id>
```

---

### Inspect Service

```bash
docker service inspect <service-id>
```

---

### View Logs

```bash
docker service logs <service-id>
```

---

### Remove Service

```bash
docker service rm <service-id>
```

---

## Scaling Services

### Scale a Service

```bash
docker service scale <service-id>=<replicas>
```

Example:

```bash
docker service scale myservice=6
```

---

### Scale Multiple Services

```bash
docker service scale service1=5 service2=3
```

---

### Scale Down

```bash
docker service scale service1=2
```

---

## Global vs Replicated Mode

### Replicated Mode

You define the number of replicas:

```bash
docker service create --replicas 3 alpine ping 8.8.8.8
```

---

### Global Mode

Runs one container per node:

```bash
docker service create --mode global alpine ping 8.8.8.8
```

---

### Use Cases for Global Mode

- Monitoring agents
- Log collectors
- Security tools
- Node-level utilities

---

## Constraints and Labels

### Node Role Constraint

```bash
docker service create \
  --replicas 3 \
  --constraint "node.role==manager" \
  alpine ping 192.168.64.10
```

---

### Add Label to Node

```bash
docker node update --label-add ssd=true <node-name>
```

---

### Use Label Constraint

```bash
docker service create \
  --constraint "node.labels.ssd==true" \
  --replicas 3 \
  alpine ping 192.168.64.10
```

---

### Engine-Level Labels

On worker node:

```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "labels": ["user=example"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Use in service:

```bash
docker service create \
  --constraint "engine.labels.user==example" \
  --replicas 2 \
  alpine ping 192.168.64.8
```

---

## Failure Handling

If a container is removed manually:

```bash
docker container rm -f <container-id>
```

Docker Swarm will automatically:

- Detect failure
- Recreate container
- Maintain desired state

---

## Troubleshooting

### Node Shows DOWN

```bash
docker node ls
```

Fix:

```bash
docker node rm <node-name>
```

---

### Worker Still Active After Removal

Run on worker:

```bash
docker swarm leave
```

---

### Service Not Scaling

Check:

```bash
docker service ps <service-id>
```

---

### Debug Node

```bash
docker node inspect <node-name>
```

---

## Conclusion

You now have a working Docker Swarm cluster with:

- Multi-node orchestration
- Automatic scheduling
- Self-healing services
- Flexible scaling
