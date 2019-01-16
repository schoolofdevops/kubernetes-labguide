# Kubernetes Cluster Administration


## Defining Quotas

```
kubectl create namespace staging

```


`file: staging-quota.yaml`
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging
  namespace: staging
spec:
  hard:
    requests.cpu: "0.5"
    requests.memory: 500Mi
    limits.cpu: "2"
    limits.memory: 2Gi
    count/deployments.apps: 1
```


```
kubectl apply -f quota.yaml
kubectl get quota -n staging
kubectl describe  quota -n staging
```


`file: nginx-deploy.yaml`


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: staging
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: nginx
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "500Mi"
            cpu: "500m"
          requests:
            memory: "200Mi"
            cpu: "200m"
```


```
kubectl apply -f nginx-deploy.yaml
kubectl describe  quota -n staging

kubectl run dep2 --image=nginx -n staging

```


## Nodes  Maintenance


Cordon

```
$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          5h        10.233.74.74   node4
redis-5bf748dbcf-ckn65    1/1       Running   0          42m       10.233.71.26   node3
redis-5bf748dbcf-vxppx    1/1       Running   0          1h        10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          5h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          1h        10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          50m       10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          50m       10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          50m       10.233.74.81   node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   0          39m       10.233.75.15   node2



$ kubectl cordon node4
node/node4 cordoned


$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          5h        10.233.74.74   node4
redis-5bf748dbcf-ckn65    1/1       Running   0          43m       10.233.71.26   node3
redis-5bf748dbcf-vxppx    1/1       Running   0          1h        10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          5h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          1h        10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          51m       10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          51m       10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          51m       10.233.74.81   node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   0          40m       10.233.75.15   node2


$ kubectl get nodes -o wide
NAME      STATUS                     ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready                      node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready,SchedulingDisabled   node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2



$ kubectl uncordon node4
node/node4 uncordoned


$ kubectl get nodes -o wide
NAME      STATUS    ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
```




### Drain a Node


```
$ kubectl drain node3
node/node3 cordoned
error: unable to drain node "node3", aborting command...

There are pending nodes to be drained:
 node3
error: pods with local storage (use --delete-local-data to override): kubernetes-dashboard-55fdfd74b4-jdgch; DaemonSet-managed pods (use --ignore-daemonsets to ignore): calico-node-4f8xc



$ kubectl get nodes -o wide
NAME      STATUS                     ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready,SchedulingDisabled   node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready                      node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2



$ kubectl uncordon node3

$ kubectl get nodes -o wide
NAME      STATUS    ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2


```

Drain with options

```
kubectl drain node4 --ignore-daemonsets=true
node/node4 cordoned
WARNING: Ignoring DaemonSet-managed pods: calico-node-hnw87
pod/nginx-65899c769f-lphtq evicted
pod/vote-56bf599b9c-22lpw evicted
pod/vote-56bf599b9c-bqsrq evicted
pod/vote-56bf599b9c-xw7zc evicted
pod/nginx-65899c769f-kq9qb evicted
pod/nginx-65899c769f-b59jq evicted
pod/vote-56bf599b9c-4l6bc evicted
pod/redis-5bf748dbcf-vxppx evicted
pod/db-66496667c9-qggzd evicted
pod/result-5c7569bcb7-s4rdx evicted
```


```
$ kubectl get pods  -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
db-66496667c9-wvrrg       1/1       Running   0          1m        10.233.75.18     node2
redis-5bf748dbcf-ckn65    1/1       Running   0          52m       10.233.71.26     node3
redis-5bf748dbcf-qbx2t    1/1       Running   0          1m        10.233.75.17     node2
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18     node3
result-5c7569bcb7-h5222   1/1       Running   0          1m        10.233.102.142   node1
vote-56bf599b9c-fvcqt     1/1       Running   0          1m        10.233.71.31     node3
vote-56bf599b9c-k6s7q     1/1       Running   0          1m        10.233.71.30     node3
vote-56bf599b9c-kv9qp     1/1       Running   0          1m        10.233.71.29     node3
vote-56bf599b9c-zz746     1/1       Running   0          1m        10.233.71.32     node3
worker-6cc8dbd4f8-6bkfg   1/1       Running   1          49m       10.233.75.15     node2


$ kubectl get pods -n kube-system -o wide
NAME                                    READY     STATUS    RESTARTS   AGE       IP                NODE
calico-node-4f8xc                       1/1       Running   2          1d        128.199.249.122   node3
calico-node-gbgxs                       1/1       Running   2          1d        128.199.224.141   node1
calico-node-hnw87                       1/1       Running   4          1d        128.199.248.156   node4
calico-node-tq46l                       1/1       Running   0          39m       128.199.248.240   node2
kube-apiserver-node1                    1/1       Running   3          23h       128.199.224.141   node1
kube-apiserver-node2                    1/1       Running   2          1d        128.199.248.240   node2
kube-controller-manager-node1           1/1       Running   3          23h       128.199.224.141   node1
kube-controller-manager-node2           1/1       Running   1          1d        128.199.248.240   node2
kube-dns-6d6674c7c6-2gqhv               3/3       Running   0          22h       10.233.71.15      node3
kube-dns-6d6674c7c6-9d2zg               3/3       Running   0          22h       10.233.102.131    node1
kube-proxy-node1                        1/1       Running   2          23h       128.199.224.141   node1
kube-proxy-node2                        1/1       Running   2          1d        128.199.248.240   node2
kube-proxy-node3                        1/1       Running   3          1d        128.199.249.122   node3
kube-proxy-node4                        1/1       Running   2          1d        128.199.248.156   node4
kube-scheduler-node1                    1/1       Running   3          23h       128.199.224.141   node1
kube-scheduler-node2                    1/1       Running   1          1d        128.199.248.240   node2
kubedns-autoscaler-679b8b455-tkntk      1/1       Running   2          1d        10.233.71.14      node3
kubernetes-dashboard-55fdfd74b4-jdgch   1/1       Running   4          1d        10.233.71.12      node3
nginx-proxy-node3                       1/1       Running   3          1d        128.199.249.122   node3
nginx-proxy-node4                       1/1       Running   2          1d        128.199.248.156   node4
tiller-deploy-5c688d5f9b-8hbpv          1/1       Running   0          22h       10.233.71.16      node3


$ kubectl get nodes -o wide
NAME      STATUS                     ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready                      node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready,SchedulingDisabled   node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2


$ kubectl uncordon node4
node/node4 uncordoned


$ kubectl get pods  -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
db-66496667c9-wvrrg       1/1       Running   0          2m        10.233.75.18     node2
redis-5bf748dbcf-ckn65    1/1       Running   0          53m       10.233.71.26     node3
redis-5bf748dbcf-qbx2t    1/1       Running   0          2m        10.233.75.17     node2
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18     node3
result-5c7569bcb7-h5222   1/1       Running   0          2m        10.233.102.142   node1
vote-56bf599b9c-fvcqt     1/1       Running   0          2m        10.233.71.31     node3
vote-56bf599b9c-k6s7q     1/1       Running   0          2m        10.233.71.30     node3
vote-56bf599b9c-kv9qp     1/1       Running   0          2m        10.233.71.29     node3
vote-56bf599b9c-zz746     1/1       Running   0          2m        10.233.71.32     node3
worker-6cc8dbd4f8-6bkfg   1/1       Running   1          50m       10.233.75.15     node2



$ kubectl delete pods vote-56bf599b9c-k6s7q vote-56bf599b9c-k6s7q vote-56bf599b9c-zz746
pod "vote-56bf599b9c-k6s7q" deleted
pod "vote-56bf599b9c-k6s7q" deleted
pod "vote-56bf599b9c-zz746" deleted


$ kubectl get pods  -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
db-66496667c9-wvrrg       1/1       Running   0          3m        10.233.75.18     node2
redis-5bf748dbcf-ckn65    1/1       Running   0          54m       10.233.71.26     node3
redis-5bf748dbcf-qbx2t    1/1       Running   0          3m        10.233.75.17     node2
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18     node3
result-5c7569bcb7-h5222   1/1       Running   0          3m        10.233.102.142   node1
vote-56bf599b9c-dzgsf     1/1       Running   0          17s       10.233.74.86     node4
vote-56bf599b9c-fvcqt     1/1       Running   0          3m        10.233.71.31     node3
vote-56bf599b9c-kv9qp     1/1       Running   0          3m        10.233.71.29     node3
vote-56bf599b9c-ptd29     1/1       Running   0          17s       10.233.74.85     node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   1          51m       10.233.75.15     node2
```
