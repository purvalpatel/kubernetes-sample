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
 
### Login ArgoCD in browser: 

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

### This process can be done by two ways:
#### 1. Manual way ( ArgoCD WebUI )
   
#### 2. Automated (Gitops)

Sample of nginx deployment with helm using ArgoCD:
-------------------------------------------------

##### 1. Generate helm chart for nginx.
```
helm create nginx
```

##### 2. Make changes in the configrations.
Edit chart.yaml <br>
```YAML
apiVersion: v2
name: myapp
version: 0.1.0
description: A simple Helm chart for my app
```

Edit values.yaml <br>
```YAML
replicaCount: 1

image:
  repository: nginx
  tag: "1.27"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```
##### 3. Test the chart locally.
```
helm template nginx
```

##### 4. Push code to git repository.
```BASH
git add .
git commit -m "pushing code"
git branch -m master main
git fetch origin
git branch --set-upstream-to=origin/main main
git pull --rebase
git push origin main
```
Now your repo will looks like this <br>

https://github.com/your-org/myapp-helm <br>
└── nginx/ <br>
    ├── Chart.yaml <br>
    ├── values.yaml <br>
    └── templates/ <br>

##### 5.Now Create deployment in ArgoCD.
- Login ArgoCD. <br>
- Connect repository with ArgoCD with credentials.
     **Settings** -> **Repositories**.<br>
     **Connect Repo** -> Select "**VIA HTTP/HTTPS**" <br>
     Enter below details: <br>
     **Name**, **Project**, **Repository URL**, **Username** and **password**. <br>
     Or you can provide **Gitlab token** instead of username and password.<br>
     Now it is showing **Connected**. <br>

- Naviagate to **applications** ->**New App**. <br>
  **Application Name** <br>
  **Project Name** - (select project) <br>
  **Sync policy** (manual/Automatic) <br>
  **Source** - Git project Repository URL. <br>
  **Revision** - Select branch name <br>
  **Path** - Provide path where YAML files are located. <br>
  **Cluster URL** - Select Cluster URL ( mostly same as where argocd deployed ) <br>
  **Namespace** - Namespace where it should be deployed. <br>

##### 6. Create.
- Now it will start deploying application.

Sample of nginx deployment using ArgoCD:
-----------------------------------------
1. Process will be same just keep the deployment, namespaces, services YAML manifests files seperately.

Automated (GitOps)
------------------

- GitOps way.
- The real power of argoCD where everything is declarative inside Git, including ArgoCD Applicaiton definations.

### Steps:
1. Create Simple helm chart.
2. Push it into Git.
3. Create ArgoCD application YAML that Automatically deploy a chart.

First two steps are completed, so we are ignoring as of now. <br>

Step 3 : Create ArgoCD application YAML. <br> 
You can keep it in another Git repo or apply it directly. <br>
nginx-argocd.yaml <br>

```YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd          # Argo CD’s namespace
spec:
  project: default
  source:
    repoURL: 'https://git.example.com/devops/myapp-helm.git'  # Your repo
    targetRevision: main                                      # Branch name
    path: '.'                                                 # Path of chart inside repo
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: myapp
  syncPolicy:
    automated:                      # Enables full GitOps automation
      prune: true                   # Delete resources removed from Git
      selfHeal: true                # Auto-fix drift
    syncOptions:
      - CreateNamespace=true        # Auto-create namespace if missing

```
Apply:
```
kubectl apply -f myapp-argo.yaml
```

Verify application status:
```
kubectl get applications -n argocd
```
