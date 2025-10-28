Kubernetes services:
------------------

When you create pod, it get its own IP address - but when it restarted or moved to another node it gets changed. <br>
➡️ That's problem if other pods needs to connects to it.

✅ So, a Service gives a **stable name and IP address** that always points to the right Pod(s), even if they restart or move. <br>
also, helps in load balancing.<br>

Internet ---> Service ---> Pod1-Pod2-Pod3 <br>

Each service has its own end points from where they can be accessed from internet.


Get list of services:
```
kubectl get svc
```

Types of services:
1. ClusterIP
2. NodePort
3. LoadBalancer
4. ExternalName
5. Headless

## 1.ClusterIP 		

- Expose an application of cluster to other parts of cluster.<br>
- Allows internal Applications communication with each other.<br>
- It Assigns a uniq range of IP address from the clusters address 	range.<br>

**Not Externally Accessible.<br>**

- Internal load balancer between pods.<br>

ClusterIP is **default service**. If we did not define type in yaml, then it will take default service.

cluster_ip.yaml

```YAML
apiVersion: v1
kind: Service
metadata: 
name: nginxsvc
spec:
  selector: 
    app: nginx
  type: ClusterIP
    ports: 
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
```

Create service:
```BASH
Kubectl apply -f clusterip.yaml
```

## 2.NodePort :

Node IP + Port <br>

- External service
- Expose app running in kubernetes cluster to outside world, using static port on each node.


**How it works ?<br>**
Pod - where your app runs<br>
ClusterIP - load balancer between pods<br>
NodePort - opens port on each node so you can access the app from outside.<br>


deployment.yaml
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx  # example app
          ports:
            - containerPort: 80
```
nodeport-service.yaml
```
Service : Nodeportservice.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80         # service port (internal)
      targetPort: 80   # pod port
      nodePort: 30080  # external port (between 30000–32767)
```

Apply:
```BASH
kubectl apply -f deployment.yaml
kubectl apply -f nodeport-service.yaml
```

List services from all namespaces:
```
kubectl get svc --all-namespaces
```
<img width="554" height="233" alt="image" src="https://github.com/user-attachments/assets/93fbf9a7-e573-47bb-9212-1a02b6716b8b" />

Here, 31000 is external NodePort which is accessible from internet.<br>

## 3.Loadbalancer:

- Exposes your application externally using a cloud providers external load balancer like. ELB.<br>
- It automatically assigns public IP and routes traffic to your apps backend pod.<br>

- internally it uses nodeport under the hood.	

loadbalancer.yaml:
```YAML
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
    - port: 80        # External port
      targetPort: 8080  # Pod's internal port
```

Apply:
```
Kubectl apply -f loadbalancer.yaml
```

Get external ip:
```
kubectl get svc my-app-lb
```

Get the IP of service:
```
kubectl get svc <service-name>  --namespace minio-chennai
```

Get all the service with loadbalancer service type:
```
kubectl get svc --all-namespaces --field-selector spec.type=LoadBalancer
```

## 4.ExternalName:

Maps the services to external DNS name.<br>

Externalname.yaml
```YAML
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: example.com
```

## 5.Headless service:

Service that does not assign clusterip.<br>
Instead of load balancing it simply returns the IP address of individual pods behind it.<br>

This is usedful for stateful applications, direct pod-to-pod communication.<br>
Statefull like - databases (mysql, cassandra, etc. )<br>

Headless.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None
  selector:
    app: my-db
  ports:
    - port: 5432
```
Apply
```
kubectl apply -f headless.yaml
```
