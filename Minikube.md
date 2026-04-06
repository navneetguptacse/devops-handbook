# Minikube Single-Node Kubernetes Cluster Setup

![Kubernetes](https://img.shields.io/badge/Kubernetes-Single--Node-blue)
![Tool](https://img.shields.io/badge/Tool-Minikube-green)
![Runtime](https://img.shields.io/badge/Runtime-Docker%20%7C%20VM-orange)
![CLI](https://img.shields.io/badge/CLI-kubectl-yellow)
![Platform](https://img.shields.io/badge/Platform-Local-lightgrey)
![Level](https://img.shields.io/badge/Level-Beginner--Friendly-success)
![License](https://img.shields.io/badge/License-MIT-brightgreen)

This guide explains how to set up a **single-node Kubernetes cluster locally using Minikube**, and how to interact with it using `kubectl`.

---

## Table of Contents

- [Overview](#overview)
- [What is Minikube](#what-is-minikube)
- [Prerequisites](#prerequisites)
- [Start Minikube Cluster](#start-minikube-cluster)
- [Cluster Verification](#cluster-verification)
- [Understanding kubectl](#understanding-kubectl)
- [Kubernetes Communication Model](#kubernetes-communication-model)
- [Minikube IP and Accessing Applications](#minikube-ip-and-accessing-applications)
- [Access Minikube Node](#access-minikube-node)
- [Minikube Dashboard](#minikube-dashboard)
- [Stopping the Cluster](#stopping-the-cluster)
- [Conclusion](#conclusion)

---

## Overview

Minikube allows you to run a **local Kubernetes cluster** on your machine for:

- Learning Kubernetes
- Development and testing
- Experimentation without cloud infrastructure

---

## What is Minikube

Minikube is an open-source tool that:

- Runs a single-node Kubernetes cluster locally
- Works on Windows, macOS, and Linux
- Uses Docker or VM drivers

It simplifies Kubernetes setup for beginners and developers.

---

## Prerequisites

Ensure the following are installed:

- Docker (recommended) or a VM driver (VirtualBox, VMware, etc.)
- kubectl CLI

Verify installations:

```bash
docker --version
kubectl version --client
```

> If you don't have Docker, install it from [Docker's official site](https://docs.docker.com/engine/install/). For kubectl, follow the instructions at [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/).

## Start Minikube Cluster

### Start with Default Driver

```bash
minikube start
```

---

### Start with Docker Driver

```bash
minikube start --driver=docker
```

After a few minutes, the cluster will be ready.

---

## Cluster Verification

### Check Nodes

```bash
kubectl get nodes
```

Example:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   30m   v1.xx.x
```

---

### Check Pods

```bash
kubectl get pods
```

---

## Understanding kubectl

`kubectl` is the command-line tool used to interact with Kubernetes.

Example:

```bash
kubectl get pods
```

### What happens internally

- kubectl sends a request to the Kubernetes API server
- API server processes the request
- Response is returned and displayed

---

## Kubernetes Communication Model

All communication with a Kubernetes cluster happens via the **API Server**.

### Methods of Interaction

1. **kubectl (CLI)**
2. **REST API**
3. **Client Libraries (Go, Python, etc.)**
4. **Web UI (Dashboard)**
5. **CI/CD Tools (ArgoCD, Flux)**

Example REST API call:

```http
GET /api/v1/namespaces/default/pods
```

Key idea:

- Every tool communicates through the same Kubernetes API

---

## Minikube IP and Accessing Applications

### Get Minikube IP

```bash
minikube ip
```

Example:

```
192.168.49.2
```

---

### Expose Application

```bash
kubectl expose deployment my-app --type=NodePort --port=80
```

Access application:

```
http://<minikube-ip>:<node-port>
```

---

## Access Minikube Node

### Recommended Method

```bash
minikube ssh
```

---

### Using SSH (VM Driver Only)

```bash
ssh -i ~/.minikube/machines/minikube/id_rsa docker@$(minikube ip)
```

---

### Notes

- Docker driver does not support direct SSH
- Use `minikube ssh` instead
- VM drivers support key-based SSH

---

## Minikube Dashboard

### Start Dashboard

```bash
minikube dashboard
```

---

### Background Mode

```bash
minikube dashboard --url &
```

---

### Features

- View pods, deployments, services
- Monitor cluster state
- Inspect logs
- Manage resources visually

---

## Stopping the Cluster

```bash
minikube stop
```

---

## Conclusion

You now have a working **single-node Kubernetes cluster** using Minikube.

### Key Takeaways

- Minikube is ideal for local development
- kubectl is used to manage the cluster
- Kubernetes API server is the core communication layer

## References

- [Docker Engine Installation](https://docs.docker.com/engine/install/)
- [Kubectl Installation](https://kubernetes.io/docs/tasks/tools/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/start/)
