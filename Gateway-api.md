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
  name: my-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
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
```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
  namespace: default
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: demo
          port: 80
```
Apply:
```
kubectl apply -f httproute.yaml
```
Test the routing:
```
kubectl get gateway my-gateway -n default
```
