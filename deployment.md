## Description:

- Deployment is a controller of kubernetes that manages your application’s pod for you.
- Manages to ensure:
1. Right number of Pods are running.
2. Pods are healthy and up-to-date.
3. When you update your app, it does  a rolling update automatically. Now downtime.

### Create deployment:
```
Kubectl apply -f deployment.yaml
```
- Comunication of containers with in the same pod will be done by localhost.
- Outside pod communication will be done by exposing port.

### Delete deployment	:
```
kubectl delete -f deployment.yaml
```

### Restart pod:
#### Method 1 : Restart by deleting pod

#### Create pod:
```
kubectl create deployment nginx-deployment --image nginx --replicas 1

kubectl delete pod test-pod

kubectl get pods
```

#### Method 2 : Rollout restart
```
kubectl rollout restart deployment nginx-deployment
```

##### Method 3: Restart by Deployment scale method:
```
Kubectl scale deployment/nginx-deployment --replicas=0
```

Again scale the deployment,
```
Kubectl scale deployment/nginx-deployment --replicas=1
```

#### Method 4: Restart by updating image
```
kubectl create deployment nginx-deployment --image=nginx --replicas=1

kubectl set image deployment/nginx-deployment nginx=nginx:latest
```

#### Method 5 : Restart by replacing specific pod.
```
kubectl create deployment nginx-deployment --image=nginx --replicas=1

Kubectl get  pod nginx-deployment -o yaml | kubectl replace -f -
```

### List deployment:
```
Kubectl get deployment
```

### Delete the complete deployment
```
Kubectl delete deployment nginx-deployment
```

- This will delete all allocated resources. I.e. pods, namespaces


Expose deployment as service: [manually exposing ports]
```
Kubectl expose deployment rabbitmq-deplyment --type=NodePort --port=80
```

Create yaml of the deployment:
```
kubectl get deployment -o yaml > nginx.yaml
```

You can create deployment from the YAML too,
```
Kubectl apply -f nginx.yaml
```
