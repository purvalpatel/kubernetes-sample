### Purpose: 
ArgoCD is a declarative, GitOps continuous delivery tool for kubernetes. <br>

ArgoCD follows gitops patterns using Git repositories as a source. <br>

Kubernetes manifests defined in several ways: <br>
1. kustomize applications <br>
2. Helm <br>
3. Any custom configuration management tool <br>


ArgoCD Automates the deployment of the desired application state for the targetted environment. <br>
Any changes modificated done in a source Git will be automatically deployed on kubernetes environment. <br>

Features: <br>
1. Automated deployment of applications to specified target environments. <br>
2. Support multiple config management tools. (kustomize, helm) <br>
3. Ability to manage multiple clusters. <br>
4. SSO integration. <br>
5. Health status analysis of application resources. <br>
6. Automated configuration drift detection and visualization. <br>
7. WebUI which provides real-time view of application activity. <br>
8. webhook integration. <br>

 

Installation steps: 
-------------------
Create namespace:
```
kubectl create namespace argocd 
```
Create Resources:
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml 
```
Expose port with services: 
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}' 
```
 
List created resources: 
```
kubectl get all –n argocd 
```
 
### Login ArgoCD in brwoser: 

Use port of argocd-server service. <br>

http://<ip>:<port> 

 

#### How to get the default password of argocd login. 

Default username is : admin <br>

**Follow below steps to get password:**

1. Connect Argocd-server pod: 
```
kubectl -n argocd1 exec -it argocd-server-78484d9d9d-q7ghl – bash 
```
2. Run below command to get the password of admin user from argocd CLI: 
```
argocd admin initial-password 
```
3. Use this password to login in to browser. 

 Deployment Architecture:
 --------
 ### 1. If Project have 5 Manifests files.<br>
k8s/<br>
 ├── namespace.yaml<br>
 ├── configmap.yaml<br>
 ├── service.yaml<br>
 └── deployment.yaml<br>

Argo will apply in rough order:<br>

```
kubectl apply -f k8s/
```
it knows the dependencies order. like namesapces before deployments.<br>
It knows CRDs dependencies (CRDs firsts, CRs later) <br>

### 2. With Helm or kustomize

 If your repo uses Kutomize or helm, then Argo doesnt apply manifests manually. it will apply in the same order which you have provided in helm/kustomize.

