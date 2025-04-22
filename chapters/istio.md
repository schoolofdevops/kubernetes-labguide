# Istio 


**Install Istio CLI**:
	
```
curl -L https://istio.io/downloadIstio | sh -
cd istio-*/bin
export PATH=$PWD:$PATH
```

**Install Istio with a minimal profile** (to keep it light for KIND):

```
istioctl install --set profile=demo -y

```


**Enable sidecar injection**:

```
kubectl config set-context --current --namespace=default
kubectl label namespace default istio-injection=enabled

```

Install Gateway CRDs 

```
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.2.1" | kubectl apply -f -; }

```


**Deploy sample apps** (e.g., Bookinfo):
```
cd ..
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

```

Expose App with Kuberneted Gateway 

```
kubectl apply -f samples/bookinfo/gateway-api/bookinfo-gateway.yaml

```

Change Service to NodePort 

```
kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=NodePort -n default --overwrite
```

Validate 

```
 kubectl get gateway
```

Access the endpoint using port-forward as 

```
kubectl patch svc bookinfo-gateway-istio -n default \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 80, "nodePort": 30300, "protocol": "TCP", "name": "http"}]}}'
```


Open your browser and navigate to [http://localhost:30300/productpage](http://localhost:30300/productpage)  to view the Bookinfo application.


### Observability with Kiali, Prometheus and Grafana 


Setup observability  addons including Prometheus, Grafana, Jaeger 
```
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system

```


Patch the Observability services with NodePort Configs 

**✅ Grafana → NodePort 30500**

```
kubectl -n istio-system patch svc grafana \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 3000, "targetPort": 3000, "nodePort": 30500}]}}'

```

✅ Kiali → NodePort 30600
```
kubectl -n istio-system patch svc kiali \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 20001, "targetPort": 20001, "nodePort": 30600}]}}'

```

**✅ Zipkin → NodePort 30700**
```
kubectl -n istio-system patch svc zipkin \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 9411, "targetPort": 9411, "nodePort": 30700}]}}'

```

**✅ Loki → NodePort 30800**
```
kubectl -n istio-system patch svc loki \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 3100, "targetPort": 3100, "nodePort": 30800}]}}'
```


Access the observability dashboards at 

* Grafana 
* Kiali 
* Zipkin
* Loki

### Load Test 

```
export GATEWAY_URL=165.22.218.103:30300

for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done

```

## Reference 

Istio Setup : [Getting Started](https://istio.io/latest/docs/setup/getting-started/)
Grafana : [Visualizing Metrics with Grafana](https://istio.io/latest/docs/tasks/observability/metrics/using-istio-dashboard/)
Kiali: [Visualizing Your Mesh](https://istio.io/latest/docs/tasks/observability/kiali/)


