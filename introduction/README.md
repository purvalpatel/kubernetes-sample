Kubernetes cluster contains two types of resources:
--------------------------------------------------
#### 1.Control plane		
cordinates the cluster ( responsible for managing cluster)

a)Cloud-Controller-manager
b)Etcd
c)Kube-api-server
d)Kube-Scheduler
e)Kube-controller-manager

#### 2.Nodes				
workers that run the application	(VM or physical machine )

a)Kubelet - each node as agent to managing the node.
b)Containerd - tool for managing the containers on worker node.


##### Key concepts:

Pod					-  Smallest deployment unit, 1 or more container
Node				-  worker machine VM or physical machine
Cluster				-  A set of nodes managed by kubernetes
Deployment			-  Managing pods, handling updates and scaling
Service				- Exposing pods inside/outside the container
Namespace			- Virtual cluster inside the kubernetes cluster
