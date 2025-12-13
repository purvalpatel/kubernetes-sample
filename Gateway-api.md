Introduction:
--------------
Kubernetes originally used **Ingress** to manage L7 (HTTP/HTTPS) traffic into the cluster. <br>
But **Ingress had limitations** → confusing annotations, no standard features across vendors, weak TCP/UDP support. <br>

So Kubernetes introduced **Gateway API** → a more powerful, modular, and extensible alternative. <br>

**Ingress** is old. <br>
**Gateway API** is New, modern, improved way to manage traffic. <br>
- More expressive
- More flexible
- More standard
- Supports HTTP, TCP, UDP, TLS routing
- Designed for multi-tenant, multi-team clusters

Resources required in Gateway API. <br>

| Resource                | Purpose                                       |
| ----------------------- | --------------------------------------------- |
| **GatewayClass**        | Defines type of Gateway (like IngressClass)   |
| **Gateway**             | Configures actual load balancer / entry point |
| **HTTPRoute**           | Rules for HTTP routing                        |
| **TCPRoute / UDPRoute** | Routing for layer 4 traffic                   |
| **TLSRoute**            | Routing for raw TLS                           |

Here, only **GatewayClass** and **Gateway** are required once. <br>
The rest (**HTTPRoute**, **TCPRoute**, **TLSRoute**, **UDPRoute**) are **optional**, depending on what traffic you want to route. <br>

Reference:
http://docs.nginx.com/nginx-gateway-fabric/install/manifests/ <br>


```
Install CRD's -> Install Gateway Fabric -> Create Gateway Class -> Create Gateway -> Create HTTPRoute
```
### How it works ?
```
Internet Traffic
       ↓
   Gateway (nginx-gateway)  ← Listens on port 80
       ↓
   HTTPRoute (attu-route)   ← Routing rules
       ↓
   Service (attu)           ← Your backend service
       ↓
   Pods                     ← Your application
```
### STEP 1 - Install CRDs.
Install Gateway API Resources:
```
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.2.2" | kubectl apply -f -
```

Install Gateway fabric CRD's:
```
kubectl apply --server-side -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.2.2/deploy/crds.yaml
```

### STEP 2 - Deploy Nginx Fabric Gateway:
```
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.2.2/deploy/default/deploy.yaml
```

Verify it is installed or not:
```
kubectl get crds | grep -i gateway
```

Verify the nginx-gateway pod is running or not:
```
kubectl get all -n nginx-gateway
```
<img width="833" height="270" alt="image" src="https://github.com/user-attachments/assets/0902c92e-b2f2-4d4b-8698-2cd7bdfbe611" />

### Step 3  - Create Gateway class (If not created already):
Ensure NGF Controller is running:
```
kubectl get pods -n nginx-gateway
```
Verify gateway class:
```
kubectl get gatewayclass
```
If not found create it manually:
`gateway-class.yaml`
```
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
```
Apply:
```
kubectl apply -f gateway-class.yaml
```

### Step 3 - Create Gateway
Create `gateway.yaml`
```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    hostname: "*.xxxxx.xx"              ## Here provide the domain name
    allowedRoutes:
      namespaces:
        from: All

```

Apply:
```
kubectl apply -f gateway.yaml
```

Verify:
```
kubectl get gateway -n default
```
### STEP 4 - Deploy your application with service.
If your application and service is already deployed then ignore this.

### STEP 5 - Create HTTPRoute.
httproute.yaml
- Please note to create **HTTPRoute inside the namespace** where your appliction service is running.
- Here, Namespace is "milvus"
- We want to route the traffic on 3000 port for attu service which is running inside the milvus namespace.
```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: attu-route
  namespace: milvus  # Must match your service namespace
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: default
  hostnames:
  - "milvus.xxxx.xx"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: attu
      port: 3000


```
Apply:
```
kubectl apply -f httproute.yaml
```
Test the routing:
```
kubectl get gateway my-gateway -n default
```
### Test your gateway is working or not:

Get port first `kubectl get svc -A | grep nginx-gateway` <br>
You can provide fix port to this service (optional) testing purpose only:
```
kubectl patch svc nginx-gateway-nginx -n default -p '{"spec":{"ports":[{"port":80,"nodePort":30506,"protocol":"TCP"}]}}'
```

```
curl http://localhost:30506 -H "Host: milvus.xxx.xx"
```
In browser: `http://milvus.xxxxx.xx:30030/#/connect` <br>

<img width="1919" height="1068" alt="image" src="https://github.com/user-attachments/assets/c15170f9-1c49-430b-bee7-386d42d99778" />


Note:
- Now if want to setup for another domain then need to create only HTTPRoute for another domain.



## issue:
- In production i dont want to open application with custom port. i want that it will open directly with the domain. for that need to use MetalLB Load balancer so the IP of load balancer will be fixed and you have to redirect domain to the external IP of load balancer.
