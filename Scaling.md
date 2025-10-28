Autoscaling:
-------------
Up or Down the resources according to usage.

Think of it like:<br><br>

When your website gets more visitors → Kubernetes automatically adds more servers (Pods).<br>
When traffic goes down → It removes extra ones to save cost.<br>

Methods: <br>
| Type                                | What it Scales          | Example Use                         |
| ----------------------------------- | ----------------------- | ----------------------------------- |
| **HPA** – Horizontal Pod Autoscaler | Number of Pods          | More traffic → more Pods            |
| **VPA** – Vertical Pod Autoscaler   | CPU/memory for each Pod | Same Pods → give them more power    |
| **CA** – Cluster Autoscaler         | Number of Nodes         | Adds/removes Nodes from the cluster |


### 🧩 1. Horizontal Pod Autoscaler (HPA)
- **Increases or decreases Pod replicas automatically based on CPU or memory usage.**

You already have a Deployment:<br>
```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

Now enable autoscaling<br>
```
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=70
```
- Minimum 2 Pods
- Maximum 10 Pods
- If CPU usage goes above 70%, add more Pods
- If CPU usage drops below, remove Pods

Check status:
```
kubectl get hpa
```

### 🧩 2. Vertical Pod Autoscaler (VPA)
Adjusts CPU and memory limits for Pods automatically.<br>

```YAML
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: nginx
  updatePolicy:
    updateMode: "Auto"  # Can be "Off", "Initial", or "Auto"
```
Apply it:
```
kubectl apply -f vpa.yaml
```

### 🧩 3. Cluster Autoscaler (CA)
if Pods can’t start because there aren’t enough Nodes, Cluster Autoscaler adds new Nodes (VMs). <br>
When load decreases, it removes unused Nodes. <br>

This usually works in cloud environments like AWS, GCP, Azure, etc.<br>


KEDA:
------

The Default kubernetes metrices for scaling is CPU, RAM. <br>
if we wanted to go above this then we have to use Advance kubernetes functionality use on top of this. i.e. **KEDA**  **(Kubernetes Event driven Autoscaling)** <br>

KEDA Work along side with the existing kubernetes component like HPA and can extend functionality without duplication or overwriding. <br>

Deployment document: https://keda.sh/docs/2.18/deploy/



