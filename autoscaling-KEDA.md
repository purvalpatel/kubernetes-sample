
KEDA:
------

The Default kubernetes metrices for scaling is CPU, RAM. <br>
if we wanted to go above this then we have to use Advance kubernetes functionality use on top of this. i.e. **KEDA**  **(Kubernetes Event driven Autoscaling)** <br>

**Best For:** <br>
✔ Workloads triggered by queues (RabbitMQ, Kafka, SQS) <br>
✔ Cron jobs / timers <br>
✔ Scaling based on external events <br>
✔ Cloud-native microservices <br>
✔ Bursty traffic <br>

**Scales to zero (HPA cannot)** <br>

KEDA makes scaling very easy because it creates the HPA for you. <br>

**Verdict:** <br>

Best for queue/event-driven workloads and cost savings (scale to zero).

KEDA Work along side with the existing kubernetes component like HPA and can extend functionality without duplication or overwriding. <br>

**Deployment document**: https://keda.sh/docs/2.18/deploy/

**Install KEDA with Helm.<br>**

Now, we will understand **how we use it**. <br>

We, have grafana and prometheus setup. in Grafana there is metrics called, Latency is showing.<br>

**What is latency(95% percentile)?<br>**
95th percentile latency is a statistical metric used to measure the performance of a system, particulary in latency-sensitive applications like web service, APIs, or databases.<br>

Means, <br>
95% requests are faster<br>
5% Requests are slower.<br>

Now we will take this data from** prometheus for scaling**.<br>

For that KEDA gives us configuration called **scaledObject**.<br>
If you are directly dealing with kubernetes then you would use **hpa**.<br>
in this case we are using **scaledobject**, which is a wrapper on top of that.<br>

## Deployment:
deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-sentiment
  labels:
    app: fastapi-sentiment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fastapi-sentiment
  template:
    metadata:
      labels:
        app: fastapi-sentiment
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: fastapi-app
        image: docker.merai.app/harshal/hf-model:0.2
        ports:
        - containerPort: 8000
        env:
        - name: PYTHONUNBUFFERED
          value: "1"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /docs
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /docs
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: fastapi-sentiment-service
  labels:
    app: fastapi-sentiment
spec:
  selector:
    app: fastapi-sentiment
  ports:
  - name: http
    port: 8000
    targetPort: 8000
    protocol: TCP
  type: ClusterIP

```
Create deployment:
```
kubectl apply -f deployment.yaml
```

### Create servicemonitor:
This will scrape data from application. <br>
servicemonitor.yaml<br>
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: fastapi-sentiment-monitor
  namespace: monitoring
  labels:
    release: prometheus
spec:
  namespaceSelector:
    matchNames:
    - default
  selector:
    matchLabels:
      app: fastapi-sentiment
  endpoints:
    - port: http        # MUST match the service port name
      path: /metrics
      interval: 15s

```
Create:
```
kubectl apply -f servicemonitor.yaml
```

Below is the example of http requests.
```YAML
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
   name: http-app-scaledobject
spec:
   scaleTargetRef:
      name: http-app
   minReplicaCount: 1
   maxReplicaCount: 10
   triggers:
      - type: prometheus
        metadata:
           serverAddress: http://prometheus-server.default.svc.cluster.local:9090
           metricName: http_requests_total
           threshold: '5'
           query: sum(rate(http_requests_total[1m])) 
```
Check created scaledobject.
```
kubectl get scaledobject
```

Note: ScaledObject will create HPA Automatically, unlinke  manual hpa creation. <br>

Check created hpa:<br>
KEDA will create linked Autoscaler automatically with scaledobject.
```
kubectl get hpa
```
Verify KEDA pods are running properly or not: `kubectget pods -n keda` <br>

You can add **multiple triggers in scaledobject.**<br>
Just add one more block.<br>

### Now we will do Load Test:

```
while true; do   curl -s http://10.98.63.182:8000/predict -X POST -H "Content-Type: application/json"      -d '{"text":"hello"}' > /dev/null; done
```
