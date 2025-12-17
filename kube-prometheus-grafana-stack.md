
Prometheus-Grafana setup in kubernetes:
-----------------------------------
1. Install kube-prometheus-stack with helm:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm search repo kube-prometheus-stack

kubectl create namespace monitoring

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```

Expose NodePort For Grafana:
```
kubectl patch svc prometheus-grafana -n monitoring -p '
{
  "spec": {
    "type": "NodePort"
  }
}'
```


Get password:
```
 kubectl get secret -n monitoring prometheus-grafana \
  -o jsonpath='{.data.admin-password}' | base64 -d
```
