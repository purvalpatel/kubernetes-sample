Before diving to ArgoCD, you must have hands-on knowledge of kubernetes, Docker, Helm<br>

ArgoCD is a declarative, GitOps continuous delivery tool for kubernetes. <br>


Core concepts of ArgoCD:
---------------------
**Application** A group of Kubernetes resources as defined by a manifest. This is a Custom Resource Definition (CRD).<br>

**Application source type** Which Tool is used to build the application.<br>

**Target state** The desired state of an application, as represented by files in a Git repository.<br>

**Live state** The live state of that application. What pods etc are deployed.<br>

**Sync status** Whether or not the live state matches the target state. Is the deployed application the same as Git says it should be?<br>

**Sync** The process of making an application move to its target state. E.g. by applying changes to a Kubernetes cluster.<br>

**Sync operation status** Whether or not a sync succeeded.<br>

**Refresh** Compare the latest code in Git with the live state. Figure out what is different.<br>

**Health** The health of the application, is it running correctly? Can it serve requests?<br>

**Tool** A tool to create manifests from a directory of files. E.g. Kustomize. See Application Source Type.<br>

**Configuration management tool** See Tool.<br>

**Configuration management plugin** A custom tool.<br>

## installation:
1. Installation.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Expose port with service type load balancer.
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
