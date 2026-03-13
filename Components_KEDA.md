## Key Components of KEDA

**KEDA (Kubernetes Event-Driven Autoscaling)** extends **Kubernetes** by enabling **event-driven autoscaling**. When KEDA is installed in a cluster, it introduces **Custom Resource Definitions (CRDs)** that allow Kubernetes workloads to scale based on external event sources such as message queues, databases, or schedules.

These CRDs behave like native Kubernetes resources (e.g., Pods or Deployments) but are specifically designed to manage event-based scaling.

KEDA installs **four main Custom Resource Definitions (CRDs):**

* ScaledObject
* ScaledJob
* TriggerAuthentication
* ClusterTriggerAuthentication

Together, these components enable dynamic autoscaling for applications running in Kubernetes.

---

# 1. ScaledObject

A **ScaledObject** defines how KEDA scales an existing Kubernetes workload such as a Deployment or StatefulSet.

### Key Fields

* **scaleTargetRef** – The Kubernetes workload to scale.
* **triggers** – Event sources that trigger autoscaling.
* **minReplicaCount** – Minimum number of pods.
* **maxReplicaCount** – Maximum number of pods.

### Example

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: my-scaledobject
spec:
  scaleTargetRef:
    name: my-deployment
  triggers:
  - type: azure-queue
    metadata:
      queueName: my-queue
      connection: azure-secret
      queueLength: "5"
  minReplicaCount: 1
  maxReplicaCount: 10
```

In this example, the deployment scales when the **Azure queue length reaches 5 messages**.

---

# 2. ScaledJob

A **ScaledJob** enables autoscaling for Kubernetes **Jobs**. Instead of scaling long-running services, it dynamically creates jobs based on event triggers.

### Key Use Cases

* Batch processing
* Event-driven background tasks
* Queue-based workloads

### Example

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: my-scaledjob
spec:
  scaleTargetRef:
    name: my-job
  triggers:
  - type: job-completion
```

This resource allows jobs to scale depending on completion events or external triggers.

---

# 3. TriggerAuthentication

A **TriggerAuthentication** resource is used to **secure communication between KEDA and external event sources**.

It stores authentication details such as:

* API keys
* Secrets
* Connection credentials

### Example

```yaml
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: my-triggerauth
spec:
  secretTargetRef:
    - parameter: apiKey
      name: my-secret
      key: api-key
```

This configuration allows KEDA to authenticate securely when accessing an external service.

---

# 4. ClusterTriggerAuthentication

A **ClusterTriggerAuthentication** is similar to TriggerAuthentication but operates **at the cluster level**.

This allows multiple ScaledObjects across namespaces to reuse the same authentication configuration.

### Example

```yaml
apiVersion: keda.sh/v1alpha1
kind: ClusterTriggerAuthentication
metadata:
  name: my-cluster-triggerauth
spec:
  secretTargetRef:
    - parameter: apiKey
      name: my-secret
      key: api-key
```

This approach improves **security, reuse, and centralized credential management**.

---

# Summary

KEDA enhances Kubernetes autoscaling by introducing CRDs that enable **event-driven scaling**.

| Component                        | Purpose                                   |
| -------------------------------- | ----------------------------------------- |
| **ScaledObject**                 | Scales Deployments or StatefulSets        |
| **ScaledJob**                    | Scales Kubernetes Jobs                    |
| **TriggerAuthentication**        | Provides authentication for event sources |
| **ClusterTriggerAuthentication** | Cluster-wide authentication sharing       |

These resources allow applications running in Kubernetes to **scale automatically based on real-time events instead of only CPU or memory metrics**.


