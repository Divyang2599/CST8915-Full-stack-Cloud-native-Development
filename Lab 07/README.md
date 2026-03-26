# Lab 7 - CST8915: Introduction to Kubernetes Basics

## Demo Video
🎥 [YouTube Demo Video](https://youtube.com/your-link-here)


---

## Deployment

The Algonquin Pet Store application was deployed to Azure Kubernetes Service (AKS) using the `algonquin-pet-store-all-in-one.yaml` file.

```bash
kubectl apply -f algonquin-pet-store-all-in-one.yaml
```

All four components were successfully deployed:

```
NAME                                  READY   STATUS    RESTARTS   AGE
order-service-576b69964f-2m5rm        1/1     Running   0          13h
product-service-6d677dbbcb-dbvmf      1/1     Running   0          13h
rabbitmq-67c4c8c64f-ctf9v             1/1     Running   0          13h
store-front-86d4bf757c-h79rp          1/1     Running   1          13h
```

The Store Front was accessible via the LoadBalancer external IP at `http://20.104.6.203`.

---

## RabbitMQ Configuration Analysis

### Is RabbitMQ a Stateless or Stateful Application?

**RabbitMQ is a stateful application.**

A stateless application holds no critical data internally — you can kill it and restart it with zero data loss because the data lives elsewhere (a database, a file system, etc.). RabbitMQ is fundamentally different. It *is* the data store for messages. It maintains queues, exchanges, bindings, and unprocessed messages in its internal storage. When RabbitMQ goes down, everything it was holding goes with it.

A real-world analogy: RabbitMQ is like a post office that holds undelivered packages inside its building. If the building is destroyed, every package waiting for delivery is permanently lost.

---

### Implications of Running RabbitMQ Without Persistent Storage

The YAML configuration in this lab deploys RabbitMQ as a standard `Deployment` with no `PersistentVolumeClaim` attached:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq
spec:
  replicas: 1
  ...
```

Without a PersistentVolume, RabbitMQ stores all its data in the container's **ephemeral filesystem**. This means:

- All queued messages exist only in memory/ephemeral disk inside the container
- No data is written to durable external storage
- The moment the container is destroyed, all data is permanently gone

In production, this is catastrophic. Orders placed by customers, payment confirmations, notification events — anything sitting in the queue between being published and being consumed is wiped on every pod restart.

---

### What Happens When the RabbitMQ Pod is Deleted or Restarted?

This was demonstrated live during the lab:

**Before restart:** An order was placed through the Store Front (4x Dog Food, Total: $79.96). The RabbitMQ dashboard confirmed **Queues: 1, Ready: 1** — the order message was sitting in the queue waiting to be processed.

**After pod deletion/restart:** The RabbitMQ pod was deleted. Kubernetes automatically restarted it (because it is managed by a Deployment controller). However, the new pod started with a completely fresh ephemeral filesystem. Upon logging back into the RabbitMQ management UI, the dashboard showed **Queues: 0** — the order was permanently lost.

This was also observed when the AKS cluster was stopped overnight and restarted the next day. All previously queued messages were gone, confirming that no data survived the pod lifecycle.

**Sequence of events:**
1. Order placed → message published to RabbitMQ queue
2. RabbitMQ pod deleted (or node restarted)
3. Kubernetes starts a new RabbitMQ pod
4. New pod has empty ephemeral storage
5. All queued messages are permanently lost
6. Order Service never processes the lost orders — customers never receive their purchases

---

### Potential Solutions

**Solution 1: PersistentVolumeClaim + StatefulSet (Kubernetes-native)**

The correct Kubernetes-native fix is two-part. First, replace the `Deployment` with a `StatefulSet`, which is designed specifically for stateful workloads and provides stable network identities and ordered pod management. Second, attach a `PersistentVolumeClaim` that mounts Azure Disk storage at `/var/lib/rabbitmq`.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: rabbitmq
spec:
  serviceName: rabbitmq
  replicas: 1
  ...
  volumeClaimTemplates:
  - metadata:
      name: rabbitmq-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

With this setup, RabbitMQ's data directory is backed by an Azure Managed Disk that persists independently of the pod lifecycle. Pod restarts, node failures, and rolling updates no longer cause data loss.

**Solution 2: Replace RabbitMQ with Azure Service Bus (cloud-native)**

Instead of self-managing a message broker inside Kubernetes, delegate the responsibility entirely to Azure Service Bus — a fully managed enterprise messaging service. The Order Service would publish messages to a Service Bus Queue instead of RabbitMQ, and a consumer would read from it.

---

### Does Azure Service Bus Solve the Issues Identified with RabbitMQ?

**Yes — and it solves more problems than just persistence.**

| Problem | RabbitMQ on Kubernetes | Azure Service Bus |
|---|---|---|
| Data lost on pod restart | Yes (without PVC) | Never — Azure manages durability |
| You manage availability | Yes — your responsibility | No — Microsoft SLA of 99.9% |
| Scaling under load | Manual — you resize pods | Automatic |
| Hardcoded credentials in YAML | Yes (`myuser`/`mypassword`) | Use Azure Managed Identity — no secrets needed |
| Monitoring and alerting | Manual setup required | Built-in Azure Monitor integration |
| Cost | Free (but compute costs apply) | Pay per million operations |

Azure Service Bus stores all messages in Azure-managed, geo-redundant storage. Messages are never lost due to infrastructure failures. It supports the AMQP 1.0 protocol, which means the Order Service's connection string is the primary change required — the application logic may remain the same.

From a production standpoint, Azure Service Bus is the recommended approach for cloud-native applications on Azure. It eliminates the operational burden of managing a stateful workload in Kubernetes, removes the need for PersistentVolumes, and integrates natively with Azure Monitor, Azure Active Directory, and Managed Identity for zero-credential authentication.

---

### Additional Security Issue Observed

The lab YAML contains hardcoded credentials in plaintext:

```yaml
env:
- name: RABBITMQ_DEFAULT_USER
  value: "myuser"
- name: RABBITMQ_DEFAULT_PASS
  value: "mypassword"
```

In production, credentials must never be stored in YAML files. The correct approach is to use **Kubernetes Secrets** or **Azure Key Vault** with the Secrets Store CSI Driver, so sensitive values are never committed to version control.

---

## Setup Notes

- AKS cluster was created with 1 node (`Standard_DS2_v2`) in `canadacentral` region
- Approximate cost: ~$3 of Azure for Students credit
- Cloud Shell session is ephemeral — `kubectl` credentials must be re-fetched with `az aks get-credentials` after each session
- All resources were cleaned up after lab completion using `kubectl delete -f algonquin-pet-store-all-in-one.yaml` and `az group delete --name lab7-rg`
