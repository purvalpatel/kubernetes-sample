Basics of kubernetes cluster:
----------------------------
1. Create kubernetes cluster
2. Deploy an app
3. Explore your app
4. Expose your app publicly
5. Scale up your app
6. Update your app

Project deployment links - https://github.com/purvalpatel/Sample-mlops-project

Kubernetes cluster components:
------------------------------

Cluster contains two type of resources.
1. Control plane		- cordinates the cluster ( responsible for managing cluster)
   - Cloud-Controller-manager
   - Etcd
   - Api-server
   - Scheduler
   - Controller-manager

2. Nodes				- workers that run the application	(VM or physical machine )
    - Kubelet - each node as agent to managing the node.
    - Containerd - tool for managing the containers on worker node.
  
Installation:
--------------

1.Swap disable

`swapoff -a`

2.Install tools: (on both master and worker)
```bash
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

3.Enable IPv4 packet forwarding:
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.confnet.ipv4.ip_forward = 1
EOF

sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-kubernetes-ip-forward.conf

sudo sysctl --system

sysctl net.ipv4.ip_forward
```

Set container runtime for kubernetes:

Ref: https://v1-31.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/

Install containerd:(on both master and worker)
```bash
sudo apt install -y containerd
```

Create default config ( on master and worker )
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Use systemd as cgroup driver
```
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

Restart service
```
sudo systemctl restart containerd
```

4.Initialize master node:
```BASH
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

5.Install CNI plugin:
```BASH
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

6.Join worker nodes:
```
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

7.Testing cluster:
```
kubectl get nodes
kubectl get pods -A
```

Cluster management:
------------------

**For Single node cluster:**
```BASH
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

**Verify cluster is healthy or not:**
```
kubectl get componentstatuses
```
**Check which CIDR used at the time of kubeadm initialization:**
```
kubectl get configmap -n kube-system -o yaml
```

**Reset cluster or worker:**
```
kubeadm reset
```

**How to check which runtime kubernetes is using currently:**
```
kubectl get nodes -o wide
```
Kuberenetes can use below runtimes:

1. Containerd
2. CRI-O

#### How to get the Kubernetes token for joining nodes (if lost):

Command for joining nodes:
```
kubeadm join 10.10.110.22:6443 --token 8k5elt.852433dupe8bp53o \ --discovery-token-ca-cert-hash sha256:2ca59e3405dec39af108c05015630e27d478c454fd1b7d7c8d38573f76b4b356 
```

#### List existing token:
```
kubeadm token list
```

If no token found then create new,
```
kubeadm token create
```

To get with full join command,
```
kubeadm token create --print-join-command
```

Get the ca-cert-hash:
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | \
openssl rsa -pubin -outform DER 2>/dev/null | \
sha256sum | awk '{print $1}'
```
This will show the token.


Get the kubeadm server URL:
```
kubectl config view -o yaml | grep -i server
```

So the final command for join cluster is:
```
kubeadm join 10.10.20.37:6443 --token estw9q.qryflwiss25nnxxm \ --discovery-token-ca-cert-hash sha256:2ca59e3405dec39af108c05015630e27d478c454fd1b7d7c8d38573f76b4b356 
```

List all resources:
```
kubectl api-resources
```

Contexts:
---------

Contexts are connection profiles of your kubectl.

Context tells kubectl:
- which cluster to connect.
- which user credentials to use.
- which namespace to use.

1. Set default namespace.
```
kubectl config set-context my-context --namespace=mystuff
```

2. List the contexts.
```
kubectl config get-contexts
```

### Users in kubernetes:

Kubernetes does not store users inside the cluster. Like, there is no users like other service.

There is two main types of users.
1. Human users  - Admins, developers which is created on server. And they can use kubectl.
2. Service accounts - this are the actual objects inside kubernetes.
```
kubectl create serviceaccount my-app
```

