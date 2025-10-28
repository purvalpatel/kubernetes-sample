Each pod have IP address assigned. When new pod is created then new ip is assigned 	to them.

So here service comes into the picture,

Like, 

assign permenant IP address to Pod.

Also helps in load balancing.

Internet ---> Service ---> Pod1-Pod2-Pod3

Each service has its own end points from where they can be accessed from internet.


Get list of services:
kubectl get svc


Types of services:
1.ClusterIP
2.NodePort
3.LoadBalancer
4.ExternalName
5.Headless

1.ClusterIP 		
- Expose an application of cluster to other parts of cluster.
Allows internal Applications communication with each other.
It Assigns a uniq range of IP address from the clusters address 	range.


Not Externally Accessible.

Internal load balancer between pods.

ClusterIP is default service. If we did not define Type in yaml 	then it will take default service.

YAML file: clusterip.yaml

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


Create service:
Kubectl apply -f clusterip.yaml

2.NodePort :

Node IP + Port

- External service
- Expose app running in kubernetes cluster to outside world, using static port on each node.


How it works ?
Pod - where your app runs
ClusterIP - load balancer between pods
NodePort - opens port on each node so you can access the app from outside.

Sample: deployment.yaml
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


Apply the yamls
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml


List services from all namespaces:
kubectl get svc --all-namespaces
