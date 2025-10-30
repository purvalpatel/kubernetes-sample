Kustomize:
-----------
Kustomize is a **Kubernetes-native configuration management tool** that lets you customize YAML manifests — without needing to modify the original files.<br>
it's like docker-compose for kubernetes.<br>

**Why kustomize exists:**
When you manage multiple environments (like dev, staging, prod), you often have almost the same Kubernetes manifests, differing only in a few fields — like:<br>
- image tag
- replica count
- config values
- resource limits

Instead of copying and editing YAMLs for each environment, Kustomize lets you define a base configuration and overlay environment-specific changes.

For example <br>

app/ <br>
├── base/ <br>
│   ├── deployment.yaml <br>
│   ├── service.yaml <br>
│   └── kustomization.yaml <br>
└── overlays/ <br>
    ├── dev/ <br> 
    │   └── kustomization.yaml <br>
    ├── staging/ <br> 
    │   └── kustomization.yaml  <br>
    └── prod/ <br>
        └── kustomization.yaml <br>

base kustomization.yaml. <br>
```
resources:
  - deployment.yaml
  - service.yaml
```

Here is the configuration of dev/prod/test environment.<br>
overlays/dev/kustomization.yaml
```
resources:
  - ../../base

namePrefix: dev-

commonLabels:
  env: dev

images:
  - name: nginx
    newTag: 1.27.1

```
