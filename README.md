# 🚀 WordPress & MySQL High-Availability Deployment on AWS

A production-grade Kubernetes deployment featuring shared persistent storage via Amazon EFS, high-performance database blocks with Amazon EBS, and robust traffic management.

---

## 🏗️ Architecture Visualization
Understanding the flow between components is crucial for a stable deployment.

```mermaid
graph TD
    Client[🌐 Internet] --> LB[⚖️ AWS Load Balancer]
    subgraph "Kubernetes Cluster"
        LB --> Service[🔌 WordPress Service]
        Service --> Pod1[📦 wordpress-pod-1]
        Service --> Pod2[📦 wordpress-pod-2]
        
        subgraph "Storage Layer"
            Pod1 --- EFS[(📂 EFS: WP Content)]
            Pod2 --- EFS[(📂 EFS: WP Content)]
            Pod1 --- DB_SVC[🔌 MySQL Service]
            Pod2 --- DB_SVC[🔌 MySQL Service]
            DB_SVC --- MySQL[🗄️ MySQL Pod]
            MySQL --- EBS[(💾 EBS: Database)]
        end
    end
```

---

## 📸 Component deep dive

### 1. Storage Hybrid Strategy
We separate static content from database data to maximize throughput and reliability.

| Component | Storage Type | Purpose |
| :--- | :--- | :--- |
| **WordPress Files** | **Amazon EFS** | Allows multiple pods to read/write the same media & plugins simultaneously. |
| **MySQL Data** | **Amazon EBS** | Guaranteed low-latency block storage for lightning-fast database queries. |

![EFS Creation](./Screenshot-project/create-awsEFS.png)
> [!NOTE]
> **EFS Setup**: We use EFS for `/var/www/html` to ensure that when you upload an image to one WordPress pod, it is instantly available to all other pods.

### 2. Traffic Management
![Kubeview](./Screenshot-project/kubeview.png)
> [!TIP]
> **Kubeview Visualization**: This view shows how Kubernetes balances our frontend pods across different nodes, ensuring that if one node fails, the site stays online.

---

## 🚨 Stability: Resolving "Noisy Neighbor" Issues
A common issue in Kubernetes is when one container steals all the CPU/RAM, crashing others. We solve this using **Resources**.

![Resources Overview](./Screenshot-project/resources-all.png)

> [!IMPORTANT]
> **Crucial Best Practice**: Every container should have `requests` and `limits`. 
> - **Requests**: Guaranteed minimum resources.
> - **Limits**: The hard ceiling to prevent a container from impacting "neighbors".

---

## ☸️ Quick Start Guide

### 1️⃣ Initial Infrastructure
```bash
# Create the project workspace
kubectl apply -f project-namespace.yaml

# Secure your database credentials
kubectl apply -f mysql-secret-password.yaml
```

### 2️⃣ Provision Storage
```bash
# Set up StorageClasses and Volumes
kubectl apply -f mysql-sc.yaml
kubectl apply -f wordpress-pv.yaml
kubectl apply -f wordpress-pvc.yaml
```

### 3️⃣ Launch Core Services
```bash
# Deploy Database first
kubectl apply -f mysql-deployment.yaml

# Deploy WordPress Frontend
kubectl apply -f wordpress-deployment.yaml

# Expose to the internet
kubectl apply -f wordpress-service.yaml
```

---

## 🌟 The Final Result
Once deployed, your high-availability WordPress site is fully operational on AWS.

![Final Website](./Screenshot-project/wordPress-website.png)

---
*Developed with a focus on Cloud-Native Reliability & DevOps Excellence.*