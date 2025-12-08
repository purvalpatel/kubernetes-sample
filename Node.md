Join Node in existing kubernetes cluster:
-------------------------------------
1. Install all kubernetes packages. ( On Worker Node)
 
2. Create New token: ( On Master node )
```
kubeadm token create --print-join-command
```
- This token will be expired after **24 hours**.
- Creating new token will be **safe**.
- If you want to add new node then you have to **generate new token.**
- Only used for **onboarding new workers**
- Recommended practice is to **generate a new token each time**

3. Paste the join command on Worker node.


Remove worker node from cluster:
------------------------
1. Drain the node first.
```
# List nodes
kubectl get nodes

# Drain nodes
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```
2. Remove/Delete node from the cluster.
```
kubectl drain worker-node-1 --ignore-daemonsets --delete-emptydir-data
```

⚠️ This does NOT touch the node’s OS. <br>
It only removes the Kubernetes registration. <br>

3. Reset Kubernetes on the Worker Node (cleanup)
**Run below command on worker node only.** <br>

```
sudo kubeadm reset -f
```
This removes: <br>
✔ kubelet <br>
✔ container runtime configs <br>
✔ certificates <br>
✔ cluster configuration <br>


If want to join node again then must create token again to follow above steps. <br>


