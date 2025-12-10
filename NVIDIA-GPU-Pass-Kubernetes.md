## install Kubernetes
1. Swap disable

2. Install tools: (on both master and worker)
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

3. Enable IPv4 packet forwarding:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.confnet.ipv4.ip_forward = 1
EOF

sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-kubernetes-ip-forward.conf

sudo sysctl --system

sysctl net.ipv4.ip_forward
```

### Set container runtime for kubernetes:
Ref: https://v1-31.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/ <br>

Install containerd:(on both master and worker) <br>
```
sudo apt install -y containerd
```

### Create default config ( on master and worker )
```
#sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

Use systemd as cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

### Restart service
```
sudo systemctl restart containerd
```

4. Initialize master node:
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

5. Install CNI plugin:
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

6. Join worker nodes:
```
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

7. Testing cluster:
```
kubectl get nodes
```
### Get all pods from all namespaces:
```
kubectl get pods -A
```
## Taint master node to schedule pods on master node
```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
## Install containerd and configure for Nvidia GPUs.
1.Required packages ( on worker node which contains GPU )
```
apt-get install nvidia-container-toolkit nvidia-container-toolkit-base containerd
```

2.Default runtime check.
```
kubectl get nodes -o wide
```

3.Set the label on GPU node.
```
kubectl get nodes
kubectl label node <node-name> gpu=true
```

4.Check the GPU is allocated to the node or not. [ on master ]
```
kubectl describe node <node-name> | grep -A10 Allocatable
```
Configure the nvidia runtime: [ on worker ] <br>
```
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Then add nvidia plugins undet the runtime confiuration: [ on worker ]
`nano /etc/containerd/config.toml`
```
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
	  base_runtime_spec = ""
	  cni_conf_dir = ""
	  cni_max_conf_num = 0
	  container_annotations = []
          pod_annotations = []
          privileged_without_host_devices_all_devices_allowed = false
          runtime_path = ""
          sandbox_mode = "podsandbox"
          snapshotter = ""


          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            systemdCgroup = true
	    CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""

## Modify below line.

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"
```
OR You can apply below command to set nvidia-container-runtime as a default runtime for kubernetes.
```
sudo nvidia-ctk runtime configure --runtime=containerd --set-as-default
```

Restart the containerd and kubelet service: on worker
```
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

## Nvidia-device-plugin install

Deploy nvidia device plugin [ on master node ]
```
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.5/nvidia-device-plugin.yml
```
Check all nodes status are running or not.
```
kubectl get all -n kube-system
```

Now check the gpu is showing on node or not.
```
kubectl describe node <node-name>  | grep -A10 Allocatable
```
<img width="449" height="178" alt="image" src="https://github.com/user-attachments/assets/10c083db-3d2e-4d07-a038-492037a93db2" />
