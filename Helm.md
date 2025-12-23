Introduction:
--------------
Helm is kubernetes package manager. <br>
Helm is like apt or yum for kubernetes. <br>

Helps in Streamlining the application management by using “Charts” to package the kubernetes resources. <br>

It facilitates simplifying the deployment, upgrades, and dependency resolution with in kubernetes cluster. <br>

Installation:
-----------
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
```
Reference link for latest helm versions: https://helm.sh/docs/intro/install/ <br>

Verify version:
```
helm version
```

Create Helm chart for Simple application:
----------------------------------------
### Create New helm chart
```
helm create myapp

## Create namespace
kubectl create ns test
```
This will create directory structure like:
```
myapp/
	Chart.yaml
	Values.yaml
	Chats/
	Templates/
```

### Customize `values.yaml`
```
 image:
   repository: your-docker-repo/your-app
   tag: "latest"
 service:
   type: ClusterIP
   port: 80
```
- The `values.yaml` file holds the default configuration values for your chart. Modify it to reflect the settings for your application.
- set Docker image details ( `repository` and `tag`)
- Define `service ports`, `replicas`, and other configurations

### Edit Kubernetes Manifests in `templates` Directory
- `deployment.yaml` for deployment configurations.
- `service.yaml` for exposing your applications.
- Additional YAML files if you need specific resources like `ConfigMaps`, `Ingress` or `secrets`.


### Package and install helm chart
Create Package:
```
helm package myapp
```

Install:
```
helm install myapp ./myapp -n test
```

Verify the deployment:
```
kubectl get pods
kubectl get pvc
```

Upgrade it:
```
helm upgrade myapp ./myapp
```

Uninstall (For Information purpose only.)
```
helm uninstall myapp
```

`values.yaml`: Default values for your chart. <br>
`templates/`: Directory containing Kubernetes manifests for deployment, service, and other resources. <br>
`Chart.yaml`: Metadata about the chart (name, version, etc.). <br>
