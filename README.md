# Production-Grade Multi-Tier App Cluster on Kubernetes

A resilient, scalable, and highly available multi-tier platform deployed on a multi-node Kubernetes cluster (`Kind`). This project demonstrates platform engineering fundamentals, moving past basic container orchestration into stateful application streaming replication, traffic routing optimization, horizontal autoscaler tuning, and enterprise Role-Based Access Control (RBAC).

---

## Architecture Blueprint

The platform separates layers into logical architectural boundaries to guarantee stateless agility and stateful durability:

- **Traffic Management & Routing:** Handled natively via an NGINX Ingress Controller.
- **Stateless App Tier:** A horizontally autoscaling microservice pool managed by a Custom Horizontal Pod Autoscaler (HPA).
- **Stateful Database Tier:** A PostgreSQL primary-replica streaming architecture utilizing a stateful topology with independent storage provisioning (`StatefulSet` with `volumeClaimTemplates`).
- **Identity & Access Security:** Zero-trust namespace isolation driven by distinct Kubernetes User Groups mapped via cluster-wide RBAC policies.

---

##  Tech Stack & Cluster Requirements

- **Local Infrastructure Engine:** Kubernetes in Docker (`Kind`) configured with 1 Control Plane and 2 Worker Nodes.
- **Data Layer Container Base:** `bitnamilegacy/postgresql:15` (Production-hardened image optimized for multi-node master-slave synchronization).
- **Telemetry Engine:** Kubernetes `metrics-server` with custom local-node TLS bypass flags.
- **Package Management:** `Helm v3` (Optional baseline validation).

---

##  Step-by-Step Deployment Guide

### Phase 1: Cluster Provisioning & Telemetry Configuration
1. Initialize the multi-node infrastructure cluster:
   
   kind create cluster --config=kind-config.yaml --name my-cluster

2. Inject the Kubernetes Metrics Server to allow real-time cluster resource tracking:
     kubectl apply -f [https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)

3. Apply the insecure TLS node patch to permit local telemetry scraping across Kind nodes:
     kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

### Phase 2: Deploying the Resilient Database Topology
Apply the master-replica manifest. This creates a dedicated read-write service (postgres-primary) and a distinct read-only load-balanced service (postgres-read), alongside independent storage layers per pod:

      kubectl apply -f postgres-cluster.yaml

### Phase 3: Launching the Auto-Scaling Stateless Tier
Deploy the application workloads along with their Horizontal Pod Autoscaler and structural routing boundaries:

   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f frontend-hpa.yaml

### Enterprise Security Layout (RBAC)
This cluster implements strict access containment by isolating cluster-management engineers from internal application code developers:
   
   kubectl apply -f rbac-setup.yaml
