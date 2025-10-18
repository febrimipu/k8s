# CloudNativePG PostgreSQL Deployment with PgBouncer (VPS Compatible)

## Overview

This repository contains Kubernetes manifests to deploy a **PostgreSQL cluster** using **CloudNativePG (CNPG)** and **PgBouncer** for connection pooling.  

The deployment includes:

- PostgreSQL cluster with 2 replicas  
- Persistent storage: 10Gi per instance  
- Superuser `postgres` with password `SynapsisChall2025`  
- Max connections: 1000  
- `pgaudit` extension enabled  
- NodePort service for external access  
- PgBouncer for connection pooling with TLS from client → pooler  

> ⚠️ Note: Full TLS end-to-end (client → PostgreSQL) is **not available** due to operator version limitations.

---
# Persiapan
1. Instal CloudNative PG Operator
```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update
helm install cnpg-operator cnpg/cloudnative-pg --namespace cnpg-system
```

Cek pod operator:
```
kubectl get pods -n cnpg-system
```


# File manifest baru (otomatis TLS + PgBouncer)

Kita bisa buat 2 file manifest:

1. postgres-cluster.yaml → namespace + password + cluster + service PostgreSQL

2. pgbouncer-tls.yaml → TLS self-signed otomatis + PgBouncer deployment + service

TLS akan dibuat otomatis oleh Kubernetes dan PgBouncer akan langsung menggunakan TLS tanpa encode manual.


## Kubernetes Manifests

### PostgreSQL Cluster (`postgres-cluster.yaml`)

- Namespace: `cnpg-system`  
- Secret for PostgreSQL password  
- PostgreSQL Cluster CRD (`Cluster`)  
- NodePort service

### PgBouncer (`pgbouncer.yaml`)

- ConfigMap `pgbouncer-config` with database & pooler settings  
- Secret `pgbouncer-users` for authentication  
- Deployment of PgBouncer  
- NodePort service (port 6432)  
- TLS enforced from client → PgBouncer using `sslmode=require`  

---

## Deployment Steps

1. **Ensure operator is running**

```bash
kubectl get pods -n cnpg-system

cloudnative-pg should be READY 1/1

2. **Apply PostgreSQL cluster**
```bash
kubectl apply -f postgres-cluster.yaml

3. **Deploy PgBounce r**
```bash
kubectl apply -f pgbouncer-tls.yaml

4. **Check all pods **
```bash
kubectl get pods -n cnpg-system

5. ** Test connection **


Why Deployment May Still Fail

1. Webhook Timeout (InternalError)

CloudNativePG operator v1.27.0 / chart 0.26.0 has a known issue:

https://github.com/cloudnative-pg/cloudnative-pg/issues/6271

Even if operator pods are healthy, the mutating webhook can timeout due to:

Operator initialization timing
VPS resource limitations
Network latency in cluster

2. TLS End-to-End Not Supported
spec.tls field in the Cluster CRD is not recognized in this operator version
Full TLS (client → PostgreSQL) is only available in v5+, which is not yet available in Helm chart repository

3. 
