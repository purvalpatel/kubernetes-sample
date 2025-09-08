Kubernetes cluster contains two types of resources:
--------------------------------------------------
#### 1.Control plane		
cordinates the cluster ( responsible for managing cluster)

- Cloud-Controller-manager
- Etcd
- Kube-api-server
- Kube-Scheduler
- Kube-controller-manager

#### 2.Nodes				
workers that run the application	(VM or physical machine )

- Kubelet - each node as agent to managing the node.
- Containerd - tool for managing the containers on worker node.


##### Key concepts:

- Pod					-  Smallest deployment unit, 1 or more container
- Node				-  worker machine VM or physical machine
- Cluster				-  A set of nodes managed by kubernetes
- Deployment			-  Managing pods, handling updates and scaling
- Service				- Exposing pods inside/outside the container
- Namespace			- Virtual cluster inside the kubernetes cluster

<img width="1402" height="882" alt="image" src="https://github.com/user-attachments/assets/27fc321d-490a-4ea1-93df-3c9e227beeff" />
