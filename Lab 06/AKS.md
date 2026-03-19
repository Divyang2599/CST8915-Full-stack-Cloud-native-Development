# Lab 6 - Deploy Algonquin Pet Store to Azure Kubernetes Service (AKS)
**Course:** CST8915 Full-Stack Cloud-Native Development  
**Student:** Divya  
**College:** Algonquin College  

---

## 📹 Demo Video

[![Lab 6 Demo Video](https://img.shields.io/badge/YouTube-Watch%20Demo-red?style=for-the-badge&logo=youtube)](https://www.youtube.com/watch?v=REPLACE_WITH_YOUR_LINK)

> 🔗 **Video Link:** https://www.youtube.com/watch?v=REPLACE_WITH_YOUR_LINK

---

## 📋 Lab Overview

In this lab, I deployed the **Algonquin Pet Store** — a microservices-based cloud-native application — to **Azure Kubernetes Service (AKS)**. The lab demonstrates how to provision a managed Kubernetes cluster on Azure, deploy containerized microservices using a Kubernetes manifest, and expose services to the internet.

---

## 🏗️ Application Architecture

The Algonquin Pet Store consists of 4 microservices, each running as an independent container (pod) inside the AKS cluster:

| Service | Technology | Port | Access |
|---|---|---|---|
| `store-front` | Vue.js + nginx | 80 | Public (LoadBalancer) |
| `order-service` | Node.js | 3000 | Internal (ClusterIP) |
| `product-service` | Node.js | 3030 | Internal (ClusterIP) |
| `rabbitmq` | RabbitMQ 3 Management | 5672 / 15672 | Internal (ClusterIP) |

Only `store-front` is exposed to the internet via an Azure Load Balancer. All other services are internal to the cluster.

---

## 🔑 Key Concept — How One IP Serves Multiple Services

All endpoints (`/`, `/products`, `/orders`, `/rabbitmq`) are accessible through a **single public IP address**. This is achieved through **nginx acting as a reverse proxy** inside the `store-front` container.

### nginx.conf Routing Logic

```nginx
server {
    listen 80;

    # Serves the Vue.js frontend
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    # Proxies /products → product-service:3030
    location /products {
        proxy_pass http://product-service:3030/products;
    }

    # Proxies /orders → order-service:3000
    location /orders {
        proxy_pass http://order-service:3000/orders;
    }

    # Proxies /rabbitmq/ → rabbitmq:15672
    location /rabbitmq/ {
        proxy_pass http://rabbitmq:15672;
    }
}
```

### Why Service Names Work (Kubernetes Internal DNS)

nginx uses service names like `order-service` and `product-service` as hostnames — not IP addresses. This works because Kubernetes runs an internal DNS server called **CoreDNS** that automatically creates a DNS record for every Service created in the cluster. When a pod makes a request to `http://order-service:3000`, CoreDNS resolves `order-service` to its stable ClusterIP automatically.

This means no hardcoded IPs anywhere — if a pod restarts and gets a new IP, the Service ClusterIP stays the same, and the DNS name always points to it.

---

## 🛠️ Tools & Prerequisites

- **Azure CLI** v2.84.0
- **kubectl** v1.35.2
- **kubelogin** v0.2.16
- **Azure for Students** subscription

---

## 🚀 Deployment Steps

### Step 1 — Login to Azure

```bash
az login
az account set --subscription "<your-subscription-id>"
```

### Step 2 — Create Resource Group

```bash
az group create --name AlgonquinPetStoreRG --location canadacentral
```

### Step 3 — Create AKS Cluster

Created via Azure Portal with the following configuration:
- **Cluster name:** `AlgonquinPetStoreCluster`
- **Node pools:** `masterpool` + `workerspool` (D2as_v4, 1 node each)
- **Pricing tier:** Free
- **Authentication:** Local accounts with Kubernetes RBAC

### Step 4 — Connect kubectl to the Cluster

```bash
az aks get-credentials --resource-group AlgonquinPetStoreRG --name AlgonquinPetStoreCluster
kubectl get nodes
```

**Output:**
```
NAME                                  STATUS   ROLES    AGE   VERSION
aks-agentpool-16485496-vmss000000     Ready    <none>   4m    v1.33.7
aks-workerspool-16485496-vmss000000   Ready    <none>   4m    v1.33.7
```

### Step 5 — Deploy the Application

```bash
kubectl apply -f algonquin-pet-store-all-in-one.yaml
```

**Output:**
```
deployment.apps/rabbitmq created
service/rabbitmq created
deployment.apps/order-service created
service/order-service created
deployment.apps/product-service created
service/product-service created
deployment.apps/store-front created
service/store-front created
```

### Step 6 — Verify Pods and Services

```bash
kubectl get pods
kubectl get services
```

**Pods:**
```
NAME                               READY   STATUS    RESTARTS   AGE
order-service-576b69964f-vtdsc     1/1     Running   0          50s
product-service-6d677dbbcb-9hpbn   1/1     Running   0          49s
rabbitmq-76fb68cc9d-gtdp9          1/1     Running   0          50s
store-front-86d4bf757c-r8c7m       1/1     Running   0          49s
```

**Services:**
```
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
order-service     ClusterIP      10.0.167.171   <none>          3000/TCP
product-service   ClusterIP      10.0.163.113   <none>          3030/TCP
rabbitmq          ClusterIP      10.0.234.217   <none>          5672/TCP,15672/TCP
store-front       LoadBalancer   10.0.85.11     20.116.167.61   80:32342/TCP
```

---

## 🌐 Live Endpoints

All endpoints served through a single public IP via nginx reverse proxy:

| Endpoint | URL |
|---|---|
| Store Front | `http://20.116.167.61` |
| Products API | `http://20.116.167.61/products` |
| Orders API | `http://20.116.167.61/orders` |
| RabbitMQ Dashboard | `http://20.116.167.61/rabbitmq` |

---

## 🧹 Cleanup

```bash
# Delete all Kubernetes resources
kubectl delete -f algonquin-pet-store-all-in-one.yaml

# Delete Azure resource groups
az group delete --name AlgonquinPetStoreRG --yes --no-wait
```

Also manually deleted from Azure Portal:
- `MC_AlgonquinPetStoreRG_AlgonquinPetStoreCluster_canadacentral`
- `NetworkWatcherRG`
- Any auto-created monitoring resource groups

---

## 💡 Key Learnings

**1. Why only store-front has a LoadBalancer**
Only the frontend needs to be publicly accessible. Exposing backend services directly would be a security risk. All internal communication stays within the cluster using ClusterIP services.

**2. Why service names work as hostnames**
Kubernetes CoreDNS automatically maps every service name to its ClusterIP. No hardcoded IPs needed anywhere in the app configuration.

**3. The nginx reverse proxy pattern**
A single public IP can serve multiple backend services by routing based on URL path. This is a standard production pattern used before dedicated Ingress controllers.

**4. Resource requests vs limits**
`requests` tells the scheduler the minimum resources needed to place a pod on a node. `limits` caps maximum usage. This is how Kubernetes manages resource fairness across pods on the same node.

**5. Credentials in YAML — a real-world flaw**
The RabbitMQ credentials (`myuser`/`mypassword`) are hardcoded in the YAML. In production, these must be stored in Kubernetes Secrets and referenced as environment variables — never hardcoded in manifests.

---

## 📁 Files

| File | Description |
|---|---|
| `algonquin-pet-store-all-in-one.yaml` | Kubernetes manifest — all 8 objects (4 Deployments + 4 Services) |
| `README.md` | This file |