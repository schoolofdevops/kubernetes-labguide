# Event Driven Auto Scaling with KEDA 



### Configure Prometheus 

Install  Prometheus with Grafana with helm 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm upgrade --install prom -n monitoring \
  prometheus-community/kube-prometheus-stack \
  --create-namespace \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=30200 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30300 \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false


```

To connect to Grafana on port 30200 use the following creds   
  * user: admin   
  * pass: prom-operator  

What this does is starts collecting metrics from services from all the namespaces not just the one in which prometheus is deployed. And it reads the annotations from the pods to figure out where to collect the metrics from.  

Setup Nginx Ingress with Metrics

Delete existing nginx ingress controller setup with KIND  

```
kubectl delete -f  https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Re deploy nginx ingress controller with helm, this time enabling the exposing the metrics which can then be scraped/collected by prometheus.

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.metrics.enabled=true \
  --set controller.metrics.serviceMonitor.enabled=true --set \ controller.metrics.serviceMonitor.additionalLabels.release="prometheus" \
  --set controller.hostPort.enabled=true \
  --set controller.hostPort.ports.http=80 \
  --set controller.hostPort.ports.https=443 \
  --set-string controller.nodeSelector."kubernetes\.io/os"=linux \
  --set-string controller.nodeSelector.ingress-ready="true"

```

## Setup Nginx Dashboard

Now, login to grafana and import custom dashboard for Nginx Ingress as

* Left menu (hover over +) -> Dashboard
* Click "Import"
* Enter the copy pasted json from [https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/grafana/dashboards/nginx.json](https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/grafana/dashboards/nginx.json)
* Click Import JSON
* Select the Prometheus data source
* Click "Import"

In a few minutes, you should see data trickling in to Nginx graphs. 


## Setup KEDA (for event-driven scale on Prometheus)


Install KEDA as 

```
helm repo add kedacore https://kedacore.github.io/charts
helm repo update

helm install keda kedacore/keda \
  --namespace keda \
  --create-namespace
```

validate 
```
kubectl get all -n keda
```





### KEDA ScaledObject (Prometheus trigger)

Scale **Chat API** when **RPS > 1 per replica** (same signal as HPA external metric).
Point KEDA to Prometheus from kube-prometheus-stack.

`keda-scaledobject-vote.yaml`

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: vote
  namespace: default
spec:
  scaleTargetRef:
    name: vote
  minReplicaCount: 1
  maxReplicaCount: 8
  cooldownPeriod: 60
  triggers:
    - type: prometheus
      metadata:
        # Use full cluster DNS for Prometheus service (adjust namespace/name if different)
        serverAddress: http://prom-kube-prometheus-stack-prometheus.monitoring.svc:9090
        metricName: nginx_rps_vote
        threshold: "400"
        query: |
          round(
            sum(
              irate(nginx_ingress_controller_requests{
                controller_pod=~".*",
                controller_class=~".*",
                controller_namespace=~".*",
                exported_namespace=~".*",
                ingress="vote"
              }[2m])
            ),
            0.001
          )
```

Apply:

```
kubectl apply -f keda-scaledobject-vote.yaml
kubectl  get scaledobject
```



## Generate Load Test 


now create a new load test config

File:  `loadtest-ing-job.yaml`
```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: loadtest-ing
spec:
  template:
    spec:
      containers:
      - name: siege
        image: schoolofdevops/loadtest:v1
        command: ["siege", "--concurrent=2", "--benchmark", "--time=4m", "http://vote.example.com"]
      restartPolicy: Never
      hostAliases:
      - ip: "ww.xx.yy.zz"
        hostnames:
        - "vote.example.com"
  backoffLimit: 4
```

where, replace `ww.xx.yy.zz` with the INTERNAL-IP address of the node which runs ingress which you can find using

```
kubectl get nodes -o wide
```

```
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
kube-control-plane   Ready    control-plane   47h   v1.29.2   172.18.0.2    <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-31-generic   containerd://1.7.13
kube-worker          Ready    <none>          47h   v1.29.2   172.18.0.3    <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-31-generic   containerd://1.7.13
kube-worker2         Ready    <none>          47h   v1.29.2   172.18.0.4    <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-31-generic   containerd://1.7.13
```

in the above example, `kube-worker` is the node where ingress is running which has and ip address of `172.18.0.3`. Thats what should be used in the load test config.

Finally create an instance of this job as,

```
kubectl create -f loadtest-ing-job.yaml
```

and now watch the scaling
```
watch 'kubectl top pods; kubectl get hpa,scaledobject'
```
