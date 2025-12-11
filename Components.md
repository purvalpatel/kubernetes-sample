# Kubernetes Resources Reference

Complete reference guide for all Kubernetes resources and their usage.

## Workload Resources

| Resource | Usage | status |
|----------|-------|--------|
| Pod | Smallest deployable unit; represents a single instance of a running process with one or more containers sharing storage and network | |
| ReplicaSet | Ensures a specified number of Pod replicas are running; typically managed by Deployments | |
| Deployment | Provides declarative updates for Pods and ReplicaSets; manages rollouts, rollbacks, and scaling | |
| StatefulSet | Manages stateful applications with stable network identities, persistent storage, and ordered deployment | |
| DaemonSet | Ensures all or selected nodes run a copy of a Pod; used for cluster-wide services like logging or monitoring | |
| Job | Creates Pods to complete a specific task; ensures specified number of successful completions | |
| CronJob | Creates Jobs on a time-based schedule using cron format | |

---

## Service Resources

| Resource | Usage | status |
|----------|-------|--------|
| Service | Exposes applications as network services; types include ClusterIP, NodePort, LoadBalancer, ExternalName | |
| Endpoints | Represents IP addresses and ports of Pods backing a Service; automatically managed | |
| EndpointSlice | Scalable alternative to Endpoints for tracking network endpoints | |
| Ingress | Manages external HTTP/HTTPS access to services; provides load balancing, SSL termination, name-based virtual hosting | |
| IngressClass | Defines different types of Ingress controllers and their configurations | |

---

## Configuration Resources

| Resource | Usage | status |
|----------|-------|--------|
| ConfigMap | Stores non-confidential configuration data in key-value pairs; consumed as environment variables, command-line args, or config files | |
| Secret | Stores sensitive information like passwords, tokens, and keys with base64 encoding | |

---

## Storage Resources

| Resource | Usage | status |
|----------|-------|--------|
| PersistentVolume (PV) | Cluster storage resource provisioned by admin or dynamically; independent of Pod lifecycle | |
| PersistentVolumeClaim (PVC) | User request for storage that consumes PV resources | |
| StorageClass | Describes different storage types/classes; enables dynamic PV provisioning | |
| VolumeAttachment | Represents volume attachment to a node; used by CSI drivers | | 
| CSIDriver | Defines a Container Storage Interface driver available in the cluster | |
| CSINode | Contains information about CSI drivers running on a specific node | |

---

## Access Control Resources

| Resource | Usage | Status |
|----------|-------|--------|
| ServiceAccount | Provides identity for processes running in Pods for API server authentication | |
| Role | Defines permissions within a specific namespace using rules | |
| ClusterRole | Cluster-wide permissions; can grant access to cluster-scoped resources or across all namespaces | |
| RoleBinding | Grants Role permissions to users or groups within a namespace | |
| ClusterRoleBinding | Grants ClusterRole permissions cluster-wide | |

---

## Policy Resources

| Resource | Usage | Status |
|----------|-------|--------|
| NetworkPolicy | Defines communication rules between Pod groups and network endpoints | |
| PodDisruptionBudget (PDB) | Limits Pods that can be down simultaneously during voluntary disruptions | |
| LimitRange | Enforces min/max resource usage per Pod or Container in a namespace | |
| ResourceQuota | Provides constraints on aggregate resource consumption per namespace | |

---

## Cluster Resources

| Resource | Usage | Status |
|----------|-------|--------|
| Node | Represents a worker machine (physical or virtual) in the cluster | |
| Namespace | Isolates groups of resources within a single cluster | |
| Event | Reports state changes and errors for debugging and monitoring | |
| APIService | Represents an API service used by the API aggregation layer | |
| Lease | Provides distributed locking mechanism; used for node heartbeats and leader election | |
| RuntimeClass | Defines different runtime classes available for running containers | |
| PriorityClass | Defines priority classes for Pods to determine scheduling order | |

---

## Metadata Resources

| Resource | Usage | Status |
|----------|-------|--------|
| CustomResourceDefinition (CRD) | Extends Kubernetes API by defining custom resources and API objects | |
| MutatingWebhookConfiguration | Configures admission webhooks that modify objects before storage | |
| ValidatingWebhookConfiguration | Configures admission webhooks that validate objects before storage | |

---

## Autoscaling Resources

| Resource | Usage | Status |
|----------|-------|--------|
| HorizontalPodAutoscaler (HPA) | Automatically scales Pod count based on CPU utilization or custom metrics | |
| VerticalPodAutoscaler (VPA) | Automatically adjusts CPU and memory requests/limits for containers | |

---

## Certificate Resources

| Resource | Usage | Status |
|----------|-------|-------|
| CertificateSigningRequest (CSR) | Requests a signed certificate from a certificate authority | |

---

## Additional Resources

| Resource | Usage | Status |
|----------|-------|---------|
| PodTemplate | Describes a template for creating Pods; used by workload controllers | |
| ControllerRevision | Immutable snapshot of state data for StatefulSets and DaemonSets rollbacks | |
| ValidatingAdmissionPolicy | Validates resources using CEL (Common Expression Language) without webhooks | |
| ResourceClaim | Manages specialized hardware resources with Dynamic Resource Allocation | |

---

## Quick Reference

### Resource Shortcuts (kubectl)

- `po` - Pods
- `deploy` - Deployments
- `rs` - ReplicaSets
- `sts` - StatefulSets
- `ds` - DaemonSets
- `svc` - Services
- `ing` - Ingress
- `cm` - ConfigMaps
- `pv` - PersistentVolumes
- `pvc` - PersistentVolumeClaims
- `sc` - StorageClass
- `sa` - ServiceAccounts
- `ns` - Namespaces
- `no` - Nodes
- `hpa` - HorizontalPodAutoscaler

---

*Last Updated: December 2025*
