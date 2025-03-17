# Lab 802  - Working with Kubernetes Control Plane

In this lab, you will learn about the components of the control plane and how to manage them. Specifically, you will learn how to examine the API server, the controller manager, the scheduler, and the etcd database.  You would learn how you could extend kubernetes using agregator extenstion such as metrics server. 


##  Examine Key Components of the Control Plane

List all the pods in the control plane namespace as 

```
kubectl get pods -n kube-system
```

Check the logs of the API server with 

```
kubectl logs kube-apiserver-kind-control-plane -n kube-system
```

Examine the API Server configurations as, 

```
kubectl describe pod kube-apiserver-kind-control-plane -n kube-system
```

You could refer to [this resource to understand API Server Configurations Better](https://gamma.app/docs/Understanding-Kubernetes-API-Server-6s6fo4yidmgahp2)


```
# Watching API requests in real-time
kubectl get events --watch

# Exploring the API with kubectl proxy
nohup kubectl proxy --port 8011 &
curl http://localhost:8001/api/v1/namespaces

```

```
# Viewing controller-manager configuration
kubectl -n kube-system describe pod kube-controller-manager

# Viewing scheduler configuration
kubectl -n kube-system describe pod kube-scheduler

# Viewing etcd configuration
kubectl -n kube-system describe pod etcd-kind-control-plane

```

### Common Troubleshooting: Control Plane**

```
# Check control plane pods
kubectl get pods -n kube-system

# Check control plane component health
kubectl get componentstatuses

```

**Why is componentstatuses Deprecated?** 

* It **does not work reliably** in all cluster configurations.  
* It is being replaced with **better health-checking mechanisms** such as:  
  * kubectl get --raw '/readyz' → Checks API Server readiness.  
  * kubectl get --raw '/healthz' → Checks API Server health.  
  * kubectl get events -n kube-system → Checks recent cluster events.
  * kubectl logs -n kube-system <component-pod> → Checks logs of critical components.  

⠀
**Check API Server Readiness:**

```
kubectl get --raw '/readyz'

```

**retrieves the readiness status of the Kubernetes API server** directly from its internal health check endpoints.

  *🛠 Breaking it Down* 

  * 1 kubectl get --raw
	* kubectl get is generally used to fetch Kubernetes resources (like pods, nodes, etc.).
	* --raw allows you to send raw HTTP requests **directly to the Kubernetes API server** instead of retrieving Kubernetes objects.
  * 2 '/readyz'
	* This is an **internal health-check endpoint** exposed by the API server.
	* The /readyz endpoint **confirms whether the API server is ready to serve traffic**.


**Check API Server Health:**

```
kubectl get --raw '/healthz'
```


### Inspect etcd health

A simple command to check the status of etcd could have been  
```

kubectl exec -it etcd-kind-control-plane -n kube-system -- etcdctl endpoint health

```

however this does not work. To know why, check the logs to diagnose the issue 

```
 kubectl logs etcd-kind-control-plane -n kube-system --tail=50
```

[sample output snippet] 
```
{"level":"warn","ts":"2025-03-17T06:02:36.176263Z","caller":"embed/config_logging.go:170","msg":"rejected connection on client endpoint","remote-addr":"127.0.0.1:49154","server-name":"","error":"tls: first record does not look like a TLS handshake"}
```

Right way is to pass on the  TLS certs as,  

```
kubectl exec -it etcd-kind-control-plane -n kube-system -- \
    etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    endpoint health
```

now you should see the output as,

```
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 9.67447ms
```


### Common Troubleshooting: Worker Nodes**

Node Status 

```
# Check node status
kubectl get nodes
kubectl describe node kind-worker

```

**Kube-Proxy Status**

```
# Check kube-proxy logs
kubectl logs -n kube-system -l "k8s-app=kube-proxy"
    
```

**Examine Kubelet**

```
# Connect to one of the nodes 
docker exec -it kind-worker bash

# Check kubelet logs
journalctl -u kubelet
    
# Check Service Status 
systemctl status kubelet

exit
```



## Setup Metrics Server as Aggregator Extenstion for Kubernetes Cluster

```
kubectl top nodes 
kubectl top pods 
```

[sample output]
```
error: Metrics API not available
```

This is because we have not setup the metrics server yet, which is a aggregator extenstion for Kubernetes Cluster.  To set it up, run the following commands

```
cd ~
git clone https://github.com/schoolofdevops/metrics-server.git
kubectl apply -k metrics-server/manifests/overlays/release
```

validate

```
kubectl get deploy,pods -n kube-system --selector='k8s-app=metrics-server'

```
Once metrics server is setup and ready, you can run the following commands to check the metrics again, 

```
kubectl top nodes 
kubectl top pods 
```

This time it will give you the output as,

```
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/metrics-server   1/1     1            1           77s

NAME                 CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
kind-control-plane   111m         5%       769Mi           19%
kind-worker          26m          1%       167Mi           4%
kind-worker2         24m          1%       238Mi           6%
```

You just extended the Kubernetes Cluster with a metrics server aggregator.

#KUBERNETES/adv