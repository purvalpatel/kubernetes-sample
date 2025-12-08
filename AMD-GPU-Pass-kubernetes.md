Assign AMD GPU to Kubernetes node:
------------------------------------
## Install Kubernetes if not installed:

### Prerequisites.
1. ROCM is installed on server.
2. rocm-smi should list the GPU's.

### Install kubernetes:

#### 1.Swap disable
```
swapoff -a
```

#### 2.Install tools: (on both master and worker)
```
sudo apt-get update# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
```

#### 3.Enable IPv4 packet forwarding:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.confnet.ipv4.ip_forward = 1
EOF

sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-kubernetes-ip-forward.conf

sudo sysctl --system

sysctl net.ipv4.ip_forward
```

#### 4. Set container runtime for kubernetes:

Ref: https://v1-31.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/

##### Install containerd:(on both master and worker)
```
#sudo apt install -y containerd
```

##### Create default config ( on master and worker )
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

##### Use systemd as cgroup driver
```
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

##### Restart service
```
sudo systemctl restart containerd
```

#### 4.Initialize master node:
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 5.Install CNI plugin:
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### Allow pods to schedule on master node ( Optional ):

```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### label-node.yaml
```
# This is a configuration example for the node labeller which includes
# ClusterRole and ClusterRoleBinding definitions, as well as the
# DeamonSet configuration that deploys the actual node labeller
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-node-labeller
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["watch", "get", "list", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: labeller
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-node-labeller
subjects:
- kind: ServiceAccount
  name: node-labeller-sa
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: node-labeller-sa
  namespace: kube-system
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: amdgpu-labeller-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: amdgpu-lr-ds
  template:
    metadata:
      labels:
        name: amdgpu-lr-ds
    spec:
      serviceAccountName: node-labeller-sa
      nodeSelector:
        kubernetes.io/arch: amd64
      priorityClassName: system-node-critical
      tolerations:
      - key: CriticalAddonsOnly
        operator: Exists
      containers:
      - image: rocm/k8s-device-plugin:labeller-latest
        name: amdgpu-lr-cntr
        imagePullPolicy: Always
        workingDir: /root
        command: ["./k8s-node-labeller"]
        args: ["-vram", "-cu-count", "-simd-count", "-device-id", "-family", "-product-name"]
        env:
          - name: DS_NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
        securityContext:
          privileged: true #Needed for /dev
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: sys
            mountPath: /sys
            readOnly: true
          - name: dev
            mountPath: /dev
        resources: {}
          # limits:
          #   memory: 20Mi
          #   cpu: 100m
          # requests:
          #   memory: 30Mi
          #   cpu: 150m
      volumes:
        - name: sys
          hostPath:
            path: /sys
            type: Directory
        - name: dev
          hostPath:
            path: /dev
            type: Directory
```
Apply changes:
```
kubectl apply -f label-node.yaml
```
### amd-device-plugin-daemon.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    deprecated.daemonset.template.generation: "1"
  creationTimestamp: "2025-09-04T10:42:18Z"
  generation: 1
  name: amdgpu-device-plugin-daemonset
  namespace: kube-system
  resourceVersion: "847"
  uid: 99dac911-d815-4c18-9df5-cf1500b33c3f
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: amdgpu-dp-ds
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: amdgpu-dp-ds
    spec:
      containers:
      - image: rocm/k8s-device-plugin
        imagePullPolicy: Always
        name: amdgpu-dp-cntr
        resources: {}
        securityContext:
          capabilities:
            drop:
            - ALL
          privileged: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/kubelet/device-plugins
          name: dp
        - mountPath: /sys
          name: sys
        - name: dev-kfd
          mountPath: /dev/kfd
        - name: dev-dri
          mountPath: /dev/dri
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/arch: amd64
      priorityClassName: system-node-critical
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
      - key: CriticalAddonsOnly
        operator: Exists
      volumes:
      - hostPath:
          path: /var/lib/kubelet/device-plugins
          type: ""
        name: dp
      - hostPath:
          path: /sys
          type: ""
        name: sys
      - name: dev-kfd
        hostPath:
          path: /dev/kfd
      - name: dev-dri
        hostPath:
          path: /dev/dri
  updateStrategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
    type: RollingUpdate
status:
  currentNumberScheduled: 0
  desiredNumberScheduled: 0
  numberMisscheduled: 0
  numberReady: 0
  observedGeneration: 1
```
Apply changes:
```
kubectl apply -f amd-device-plugin-daemon.yaml
```

Now verify GPU is assigned to node or not.
```
kubectl get nodes

kubectl describe node <node-name>
```
<img width="447" height="354" alt="image" src="https://github.com/user-attachments/assets/e8de08bc-926b-4993-96b7-6c8c95327689" />
